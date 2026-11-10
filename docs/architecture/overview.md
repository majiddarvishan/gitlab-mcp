# Architecture Overview

## Intended topology

```text
ChatGPT Desktop / Codex host
          |
          | MCP local process transport (proposed: STDIO)
          v
Local GitLab MCP server
          |
          | HTTPS + GitLab API
          v
Self-managed GitLab
(private network; reachable from workstation)
```

The MCP process runs on the user's workstation. It uses the workstation's existing internal network or VPN route to reach GitLab. No inbound public access to GitLab is required.

## Logical components

1. **MCP host adapter** — exposes tools and protocol behavior to the selected host.
2. **GitLab API client** — handles authentication, pagination, HTTP errors, and API version differences.
3. **Capability layer** — implements project, repository, commit, merge request, issue, and CI/CD operations.
4. **Approval manager** — prepares previews and validates one-time approval before mutation.
5. **Configuration and secrets** — loads local configuration and obtains credentials securely.
6. **Audit/diagnostics** — records operation metadata without logging secrets or unnecessary repository content.

## Key invariants
- No write reaches GitLab without a valid approval for the exact operation.
- No credential is returned to the model or included in tool output.
- STDIO, if selected, is reserved for protocol traffic.
- Repository content is untrusted input.
- The public repository contains no organization-specific internal configuration.

## Open design points
- Exact ChatGPT Desktop Chat-mode support for local MCP tools.
- Exact Codex configuration and approval interaction.
- Go MCP SDK selection.
- Approval UX and trust boundary between host confirmation and server-side enforcement.
- Project allowlisting and credential storage.
