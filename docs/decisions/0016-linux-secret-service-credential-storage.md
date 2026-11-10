# ADR-0016: Local GitLab PAT Credential Storage

- **Status:** Proposed
- **Date:** 2026-11-10
- **Decision owner:** User
- **Related task:** SEC-002
- **Target:** Local Linux desktop session running the MCP process
- **Related design:** [Local Credential Storage Design](../security/credential-storage.md)

## Context

ADR-0015 selects a Personal Access Token belonging to the user's human GitLab account. The token uses the broad `api` scope and must never be exposed to the model, committed to the public repository, or placed in ordinary configuration, process arguments, or logs.

The user currently has no PAT. A secure local storage and retrieval path must therefore be designed and validated before the first token is created.

## Proposed decision

For the initial Linux desktop deployment, use the Freedesktop Secret Service D-Bus API, backed by a compatible login-session keyring provider such as GNOME Keyring.

- Store the PAT in the user's default Secret Service collection after verifying where that alias points.
- Run MCP as the regular desktop user in the same D-Bus session.
- Retrieve the PAT through direct Secret Service D-Bus calls in the eventual Go implementation; do not spawn `secret-tool lookup` to print the token at runtime.
- Use only non-sensitive attributes: `application=gitlab-mcp`, `credential=gitlab-personal-access-token`, and an opaque local `profile=primary`.
- Never store private hostnames, usernames, email addresses, or project identifiers in lookup attributes because Secret Service attributes may be unencrypted.
- Fail closed if the provider is unavailable, the collection/item is locked, no item exists, or multiple matching items exist. No plaintext, environment-variable, command-line, or model-visible fallback is allowed.
- This first backend does not support headless SSH/cron/system-service operation. Such a requirement needs a separate design decision.

## Alternatives considered

1. **Plaintext configuration or environment variable.** Rejected: it increases accidental disclosure through files, process environments, diagnostics, shell history, or host configuration.
2. **`secret-tool lookup` child process at runtime.** Rejected for runtime retrieval because it writes the PAT to stdout, adding an avoidable exposure channel. It may be used only as a local one-off setup/test utility when the secret is entered through its hidden TTY prompt.
3. **Secret Service D-Bus API.** Proposed: standard Linux desktop secret-store interface, designed for the user's login session, with GNOME/KDE ecosystem support. Provider and session behavior must be validated on the target workstation.
4. **Separate headless secret backend.** Deferred: not needed for the initial interactive desktop deployment. Any headless requirement must be designed explicitly rather than triggering a fallback.

## Security considerations

- Secret Service secrets are usually encrypted at rest, but the API's lookup attributes may be stored unencrypted.
- Secret Service is not a per-application hardware enclave. Other compromised processes running as the same user may be able to access secrets while the collection is unlocked.
- A local keyring does not narrow GitLab PAT permissions; the canonical project allowlist remains mandatory within MCP.
- The MCP process must return only sanitized credential errors and never reveal the token through tool outputs or logs.

## Consequences

### Benefits
- No PAT in source control, plaintext project configuration, or model-visible chat.
- No secret value in process arguments or environment variables.
- Keyring unlock is tied to the user's desktop session and provider.

### Trade-offs
- The MCP process depends on a compatible Secret Service provider and an unlocked user session.
- Headless/non-interactive environments are unsupported by this first proposal.
- Same-user malware or a compromised same-user process remains in the threat model.
- The actual D-Bus behavior, default collection, lock behavior, and ChatGPT Desktop process access must be tested on the workstation.

## Acceptance criteria

This ADR must remain Proposed until:
- The user confirms the MCP runtime runs in the same Linux desktop session, or selects another platform/backend.
- SEC-002 tests a non-secret canary item through the actual keyring provider, verifies the user's D-Bus session and lock/unlock behavior, and accepts the fail-closed policy.
- TST-009 later verifies that the actual ChatGPT Desktop/Codex-launched MCP process can read an unlocked item without printing it and fails closed under unavailable/locked/missing/ambiguous cases. The runtime must not use the PAT before this host integration test passes.
- The user explicitly accepts this credential-storage design.

No PAT may be created solely to test this storage proposal.
