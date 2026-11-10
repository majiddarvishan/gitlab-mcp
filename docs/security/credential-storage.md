# Local Credential Storage Design

- **Status:** Proposed; user approval pending
- **Initial platform assumption:** Linux desktop session running the local MCP process
- **Related task:** SEC-002
- **Related ADR:** [ADR-0016 — Linux Secret Service credential storage](../decisions/0016-linux-secret-service-credential-storage.md)
- **Target GitLab:** Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier
- **Last reviewed:** 2026-11-10

## 1. Purpose

Define how the local MCP process stores and retrieves the user's GitLab Personal Access Token (PAT) without putting the token in model-visible content, source control, ordinary configuration, process arguments, or logs.

No PAT currently exists. Do not create one until this proposal is accepted and SEC-002's acceptance criteria are satisfied.

## 2. Proposed Linux backend

Use the Freedesktop Secret Service D-Bus API, backed by a desktop keyring provider such as GNOME Keyring. KDE environments may provide a compatible Secret Service endpoint through their wallet integration; this must be verified on the actual workstation rather than assumed.

The Secret Service is designed for applications to access secrets in the user's login session. Secret values are usually encrypted at rest, and a collection/item can be locked until the user unlocks it. See the [Secret Service specification](https://specifications.freedesktop.org/secret-service/latest/description.html).

### Runtime access

- The MCP process runs as the normal logged-in desktop user, never as root.
- The MCP process connects to the user's existing D-Bus session and reads the secret through the Secret Service API.
- The eventual Go implementation should use a D-Bus Secret Service client directly. It must not spawn `secret-tool lookup` at runtime because that prints the PAT to a child process's stdout.
- Retrieve the PAT only when needed, keep its in-memory lifetime short, attach it to the GitLab `PRIVATE-TOKEN` HTTP header, and never include the header in logs or errors.
- The PAT must never be returned as an MCP tool result or passed to the AI client/model.
- The MCP process must not read the PAT from environment variables, command-line arguments, project files, workspace files, or normal configuration.

### Secret identity and metadata

Use a generic label and non-sensitive lookup attributes, for example:

| Field | Proposed value |
|---|---|
| Item label | `GitLab MCP PAT` |
| `application` | `gitlab-mcp` |
| `credential` | `gitlab-personal-access-token` |
| `profile` | `primary` |

The Secret Service specification warns that lookup attributes are not secret material and may be stored unencrypted. Therefore, do not put the private GitLab hostname/base URL, username, email address, project ID/path, token prefix, or other sensitive identifiers in the label or attributes. A locally chosen opaque profile alias is enough to select the credential.

Use the configured default collection only after confirming where the workstation's `default` alias points. Do not assume that it is named `login`.

## 3. Non-secret workstation preflight

Run this only in the same desktop user session that will launch the MCP process. These checks do not need a GitLab PAT.

### 3.1 Check local tools and Secret Service

On Debian 12, `secret-tool` is supplied by `libsecret-tools`. Check whether it is already installed:

```bash
command -v secret-tool
command -v busctl
```

Check the current user-session service owner:

```bash
busctl --user status org.freedesktop.secrets
```

Read the default collection alias:

```bash
busctl --user call \
  org.freedesktop.secrets \
  /org/freedesktop/secrets \
  org.freedesktop.Secret.Service \
  ReadAlias s default
```

Do not assume that the default collection is named `login`. A result of object path `/` means that no default collection is assigned. Do not publish D-Bus environment details or local usernames.

### 3.2 Store and retrieve a disposable canary

The following commands use only the non-secret value `mcp-storage-canary`. They do not touch a GitLab token.

Store the value in an item identified by generic attributes:

```bash
secret-tool store \
  --label='GitLab MCP storage canary' \
  application gitlab-mcp \
  credential storage-canary \
  profile primary
```

When prompted, enter exactly `mcp-storage-canary`. The prompt is interactive and does not echo the value.

Retrieve the canary and verify that it prints exactly the test string:

```bash
secret-tool lookup \
  application gitlab-mcp \
  credential storage-canary \
  profile primary
```

Remove the canary item:

```bash
secret-tool clear \
  application gitlab-mcp \
  credential storage-canary \
  profile primary
```

Run lookup once more; it should return no item. Do not substitute a real PAT into any of these commands. For a real PAT, do not use `secret-tool lookup` because it prints the value to stdout; the eventual MCP process must retrieve it through direct D-Bus access.

### 3.3 What to report

Return only sanitized results:

- `secret-tool` installed: YES/NO.
- Secret Service D-Bus service reachable: YES/NO.
- Default collection alias assigned: YES/NO.
- Canary store: PASS/FAIL.
- Canary lookup: PASS/FAIL.
- Canary clear: PASS/FAIL.
- Keyring unlock interaction: works / prompts / unavailable.
- Desktop environment family: GNOME / KDE / other / unknown.

Do not send usernames, D-Bus environment variables, object paths, local home paths, keyring contents, or screenshots. Do not paste the canary command output if it contains anything other than the known test value.

If the canary test fails, stop. Do not create the PAT and do not add a plaintext fallback. First determine whether a Secret Service provider exists and is unlocked in the MCP host's desktop session.

## 4. Initial provisioning

After this proposal is accepted and the Secret Service provider has passed its local preflight:

1. The user creates a legacy PAT through their normal GitLab UI. The user-selected scope is `api`; no other scope is added.
2. The PAT is displayed only once by GitLab. The user must not paste it into ChatGPT or any AI prompt.
3. Store it directly in the user's keyring using an interactive password prompt in a trusted local terminal or password manager. The optional Debian `secret-tool store` utility prompts on a TTY and accepts the secret without placing its value in the command arguments. It is part of `libsecret-tools`; install it only if needed and approved by the workstation owner.
4. Use only generic lookup attributes from Section 2. Do not put the internal GitLab URL or user identity in keyring metadata.
5. Verify that the new item can be retrieved by the MCP process under the same desktop user, without printing the PAT.
6. Only after secure storage has been verified may the PAT be used for the sanitized SEC-009 API validation.

For a non-secret preflight, first store and retrieve a disposable canary value (not a real PAT), verify the matching attributes and collection behavior, and then remove the canary. This tests the Secret Service path without creating a GitLab token.

## 5. Startup, lock, and failure behavior

The first Linux backend assumes an interactive desktop login session.

| Condition | Required behavior |
|---|---|
| Secret Service available and item unlocked | Retrieve the PAT and continue |
| Item or collection locked | Do not attempt an invisible/indefinite unlock prompt; return a clear non-secret error asking the user to unlock the desktop keyring and restart/reconnect MCP |
| Secret item missing | Fail closed; do not create a token or ask for it in model-visible chat |
| D-Bus session or Secret Service unavailable | Fail closed; do not fall back to plaintext files, environment variables, process arguments, or standard input from the AI client |
| Multiple matching items | Treat as ambiguous and fail closed; do not select an arbitrary token |
| Credential read or D-Bus error | Return a sanitized error without D-Bus payloads or secret material |
| Token rejected by GitLab | Return a sanitized authentication error; do not echo the request header |

Headless SSH, cron, system services, and sessions without an unlocked user keyring are not supported by this initial proposal. A separate backend/ADR would be required for those environments; no automatic downgrade is allowed.

## 6. Security boundaries and residual risk

- The keyring protects stored secrets at rest according to the configured provider and unlock policy.
- Secret Service is a user-session service, not a per-application hardware enclave. A compromised process running as the same logged-in user may be able to ask the unlocked Secret Service for the PAT. This design does not claim protection against malware or arbitrary code execution under the user's account.
- The PAT itself has the broad `api` scope and can reach all GitLab resources permitted to its owner. The MCP project allowlist limits MCP behavior; it does not reduce the PAT's direct GitLab permissions.
- Local disk encryption, OS account security, desktop lock policy, and workstation malware controls remain relevant.
- Do not store the PAT in core dumps. Where supported by the eventual runtime, disable core dumps for the MCP process and redact auth headers from HTTP tracing, panic output, telemetry, and diagnostics.
- The application may log only sanitized credential metadata: configured profile alias, credential-present boolean, last successful retrieval time, expiry date if stored separately, and status. It must never log the PAT or a reversible representation of it.

## 7. Replacement and revocation

- Planned no-downtime replacement: create a second PAT, store and validate it, switch the MCP process to the new item, then revoke the old PAT.
- Do not use GitLab's Rotate UI when an overlap window is required; it invalidates the old token immediately.
- If the PAT is compromised, revoke it immediately. Do not preserve it for an overlap window.
- Delete the old Secret Service item only after GitLab revocation has been confirmed.
- Expired/revoked tokens cannot be reactivated. Provision a new PAT after the approved storage path is available.

## 8. SEC-002 acceptance checklist

SEC-002 may be marked DONE only when all of the following are true:

- [ ] User confirms that the MCP server runs on the same Linux desktop session targeted by this design, or a different platform/backend is selected.
- [ ] A Secret Service provider is available to the normal desktop user.
- [ ] The local canary test has passed and the canary item has been deleted.
- [ ] The default collection alias is verified; a canary secret can be stored, retrieved, and removed.
- [ ] The keyring's lock/unlock behavior is understood and the MCP process can access an unlocked item.
- [ ] Missing provider, missing item, locked keyring, ambiguous item, and read errors all fail closed without fallback.
- [ ] Runtime retrieval is designed around direct D-Bus access, not shelling out to print the PAT.
- [ ] The setup, replacement, emergency-revocation, and sanitized diagnostic procedures are accepted.
- [ ] No real PAT is created until the above criteria are met.

## References

- [Freedesktop Secret Service API](https://specifications.freedesktop.org/secret-service/latest/description.html)
- [Secret Service lookup attributes](https://specifications.freedesktop.org/secret-service/latest/lookup-attributes.html)
- [GNOME libsecret](https://gnome.pages.gitlab.gnome.org/libsecret/)
- [Debian 12 `secret-tool` manual](https://manpages.debian.org/bookworm/libsecret-tools/secret-tool.1.en.html)
- [GitLab v18.8.4-ee PAT documentation](https://gitlab.com/gitlab-org/gitlab/-/raw/v18.8.4-ee/doc/user/profile/personal_access_tokens.md)
