# Security Requirements

## Threat model
The MCP server bridges a cloud-connected AI client and a private GitLab service. GitLab's network privacy does not make repository content local to the model: content returned by tools may be transmitted to the AI provider for inference.

## Credentials
- Use a dedicated GitLab identity or token where feasible.
- Apply minimum required scopes and project/group permissions.
- Store credentials using an OS-appropriate secret store where practical; do not put tokens in command-line arguments, source files, or logs.
- Support credential rotation and revocation.
- Never include real credentials in examples or tests.

## Project allowlist enforcement
- Maintain an explicit allowlist of project identities in local configuration.
- Resolve project paths to canonical project IDs and enforce the allowlist against canonical IDs.
- Check the allowlist on every tool call, including nested references and cross-project references.
- Do not treat "visible to the GitLab user" as equivalent to "allowed for MCP".
- Do not implicitly authorize new projects created under an allowed group.
- Reject unknown, ambiguous, or unlisted projects before making project-scoped API calls.
- Never permit a caller to override or bypass the allowlist through tool arguments.

## Approval enforcement
- All mutations require explicit approval for the exact operation.
- The preview must include project, target ref, action, and complete material parameters.
- Approval expires after use and is invalidated by any parameter change.
- The execution layer must reject writes without valid approval.
- Merge operations require a separate approval and a fresh preflight check.
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
