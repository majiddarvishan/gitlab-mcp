# Security Requirements

## Threat model
The MCP server bridges a cloud-connected AI client and a private GitLab service. GitLab's network privacy does not make repository content local to the model: content returned by tools may be transmitted to the AI provider for inference.

## Credentials
- Use a dedicated GitLab identity or token where feasible.
- Apply minimum required scopes and project/group permissions.
- Store credentials using an OS-appropriate secret store where practical; do not put tokens in command-line arguments, source files, or logs.
- Support credential rotation and revocation.
- Never include real credentials in examples or tests.

## Approval enforcement
- All mutations require explicit approval for the exact operation.
- The preview must include project, target ref, action, and complete material parameters.
- Approval expires after use and is invalidated by any parameter change.
- The execution layer must reject writes without a valid approval.
- High-impact actions such as deleting protected refs, merging, or changing project settings require separate review and may be excluded from the initial release.

## Input and output
- Validate project IDs/paths, branch names, file paths, refs, and payload sizes.
- Prevent path traversal and ambiguous project resolution.
- Treat repository files, issue text, MR descriptions, and comments as untrusted data, not instructions.
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
