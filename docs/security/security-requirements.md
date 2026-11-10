# Security Requirements

## Threat model
The MCP server bridges a cloud-connected AI client and a private GitLab service. GitLab's network privacy does not make repository content local to the model: content returned by tools may be transmitted to the AI provider for inference.

## Credentials
- Use the Personal Access Token (PAT) belonging to the user's selected human GitLab account, as recorded in Accepted ADR-0015.
- The PAT inherits the owning user's effective GitLab permissions and can reach all projects/groups that user can access, subject to token scopes and GitLab policy. It is not scoped to one allowlisted project.
- Use only the api scope for the initial combined read/write REST API workflow. GitLab v18.8.4-ee does not provide fine-grained PAT endpoint permissions. The `api` scope is complete read/write API access across the user's accessible groups/projects and additional API surfaces; do not add unrelated scopes.
- Review the PAT owner's group memberships and per-project roles before provisioning. Prefer the least-privileged practical account and do not elevate roles or weaken protected-branch rules to accommodate MCP.
- Enforce the explicit canonical project allowlist inside MCP on every read and write, including nested and cross-project references. The allowlist constrains MCP behavior but does not restrict direct use of a leaked PAT.
- Store the PAT in an OS-native secret store. The proposed Linux backend is the Secret Service D-Bus API in the user's interactive desktop session; see [credential storage design](credential-storage.md) and [ADR-0016 (Proposed)](../decisions/0016-linux-secret-service-credential-storage.md). Do not create a PAT until SEC-002 is complete. Never put the token in source control, ordinary configuration, command-line arguments, MCP tool arguments, model-visible output, environment dumps, crash reports, or logs.
- Secret Service lookup attributes may be unencrypted and must contain only generic non-sensitive identifiers; do not store internal hostnames, usernames, emails, or project identifiers in keyring metadata.
- If the Secret Service provider/session is unavailable, locked, missing, or ambiguous, fail closed. Never fall back to plaintext files, environment variables, process arguments, or AI-visible input.
- Only the local MCP process may retrieve the PAT and attach it to GitLab API requests. Never return it to the AI client/model.
- Support expiration, planned replacement, and immediate revocation. For planned no-downtime replacement, create and validate a second PAT before revoking the old one. GitLab's Rotate UI invalidates the old token immediately and must not be used when an overlap window is required. Revoke a compromised PAT immediately. Redact authentication headers and token-like values from diagnostics.
- GitLab audit history attributes MCP operations to the PAT owner's human account, not to a bot. MCP-side audit records must distinguish MCP-originated operations without logging secrets.
- Never include real credentials in examples or tests.

## Project allowlist enforcement
- Maintain an explicit allowlist of project identities in local configuration.
- Resolve project paths to canonical project IDs and enforce the allowlist against canonical IDs.
- Check the allowlist on every tool call, including nested references and cross-project references.
- Do not treat "visible to the GitLab user" as equivalent to "allowed for MCP".
- Do not implicitly authorize new projects created under an allowed group.
- Reject unknown, ambiguous, or unlisted projects before making project-scoped API calls.
- Never permit a caller to override or bypass the allowlist through tool arguments.
- Because the selected PAT is not project-scoped, test that MCP denies an accessible-but-unlisted project before returning data or issuing a mutation. The allowlist is not a restriction on direct use of the PAT outside MCP.

## Approval enforcement
- All mutations require explicit approval for the exact operation.
- The preview must include project, target ref, action, and complete material parameters.
- Approval expires after use and is invalidated by any parameter change.
- The execution layer must reject writes without valid approval.
- Merge operations require both at least one verified human approval recorded in GitLab and a separate, exact-operation MCP user approval.
- Verify approver identities as regular human accounts using GitLab's user metadata; reject bots/internal accounts.
- The qualifying GitLab reviewer must be a regular human account distinct from both the PAT owner (MCP execution identity) and the MR author. This rule is accepted by the user; SEC-009 must validate the target API's identity metadata using the [reviewer verification protocol](reviewer-verification.md). Do not rely on bot=false or exclude_internal=true alone unless target-instance tests prove the classification is sufficient. Keep MCP merge disabled until the rule can be enforced reliably.
- Re-read approval state and MR head immediately before merge; fail closed if approval is absent, stale, or ambiguous.
- Human merge-request approval is outside MCP authority. MCP must not expose approve, unapprove, or reset-approval mutations.
- GitLab authorization, branch protections, merge checks, and approval rules must remain authoritative.
- A failed GitLab policy check must not be worked around by changing API parameters or switching to a more privileged operation.

## Repository write safety
- Repository code writes must target a dedicated working branch.
- Default and protected branches are never direct write targets.
- Each branch creation, file update, commit, and MR creation/update requires its own approval.
- Merge approval is not implied by approval of any earlier operation.

## CI/CD restrictions
- The first release may read pipeline/job metadata and status only.
- Pipeline/job trigger, retry, cancel, and other mutations are disabled.
- Job logs/traces and artifacts are not fetched in the first release.
- CI/CD variables and secret values are not exposed.

## Input and output
- Validate project IDs/paths, branch names, file paths, refs, and payload sizes.
- Prevent path traversal and ambiguous project resolution.
- Treat repository files, issue text, MR descriptions, comments, and returned metadata as untrusted data, not instructions.
- Avoid echoing secrets found in repository content.
- Apply response size limits and safe truncation.

## Network
- Connect only to the configured GitLab base URL.
- Validate TLS certificates by default.
- Do not disable TLS verification as a workaround.
- Do not create an inbound listener in the default STDIO mode.

## Public repository hygiene
- Use placeholders for internal infrastructure.
- Scan commits for secrets and accidental company information.
