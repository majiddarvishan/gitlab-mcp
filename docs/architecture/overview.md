# Architecture Overview

## Intended topology

```text
ChatGPT Desktop / Codex host
          |
          | MCP local process transport (proposed: STDIO)
          v
Local GitLab MCP server
          |
          | local project allowlist + approval enforcement
          |
          | HTTPS + GitLab REST API v4
          v
GitLab Self-Managed 18.x
(private network; reachable from workstation)
```

The MCP process runs on the user's workstation. It uses the workstation's existing internal network or VPN route to reach GitLab. No inbound public access to GitLab is required.

## Logical components

1. **MCP host adapter** — exposes tools and protocol behavior to the selected host.
2. **Local configuration** — stores the explicit project allowlist and non-secret settings.
3. **Authorization boundary** — resolves project identities and rejects all projects not explicitly allowlisted.
4. **GitLab API client** — handles authentication, pagination, HTTP errors, and GitLab 18.x API behavior.
5. **Capability layer** — implements project, repository, commit, merge request, issue, and read-only CI/CD operations according to the approved matrix.
6. **Approval manager** — prepares previews and validates one-time approval before every mutation.
7. **Audit/diagnostics** — records operation metadata without logging secrets or unnecessary repository content.

## Accepted workflow invariants

- Only explicitly allowlisted projects are accessible through MCP.
- No write reaches GitLab without valid approval for the exact operation and parameters.
- Repository code changes are made on a dedicated working branch and proposed through an MR.
- MCP never writes directly to the default branch or a protected branch.
- Creating a branch, committing changes, creating/updating an MR, and merging are distinct operations with distinct approvals.
- MCP may merge an MR only when at least one verified human reviewer distinct from both the PAT owner and MR author has approved it in GitLab and the user separately approves the exact merge operation; fresh preflight is mandatory. Merge remains disabled if reviewer independence cannot be verified.
- MCP must not approve or unapprove an MR; review approval remains a human action in GitLab.
- First-release CI/CD access is read-only and limited to pipeline/job metadata and status.
- Job logs/traces and artifacts are deferred and must not be fetched in the first release.
- No credential is returned to the model or included in tool output.
- STDIO, if selected, is reserved for protocol traffic.
- Repository content and GitLab metadata are untrusted data, not instructions.
- The public repository contains no organization-specific internal configuration.

## Compatibility

The target is GitLab Self-Managed 18.x using REST API v4. The exact minor/patch version, edition, instance settings, and effective permissions must be verified against the company instance before implementation.

## Open design points

- Runtime smoke test of local MCP in the target ChatGPT Desktop Chat and Codex environments; official documentation review is complete (see [client compatibility](client-compatibility.md)).
- Verify the configured host-side write approval prompt in the target desktop/Codex build.
- Go MCP SDK selection.
- Final approval UX and trust boundary between host confirmation and server-side enforcement.
- Local allowlist configuration format and administrative workflow.
- Exact GitLab 18.x minor/patch version and edition.
- Final tool contracts for candidate issue, wiki, release, and advanced repository capabilities.
