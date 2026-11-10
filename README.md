# GitLab MCP

A local Model Context Protocol (MCP) server that enables ChatGPT Desktop and Codex to work with a self-managed GitLab instance reachable from the user's workstation.

The project is documentation-first. No implementation is approved until the requirements and architecture baseline are complete and the user explicitly authorizes implementation.

## Goals

- Connect AI-assisted development workflows to a self-managed GitLab instance without making GitLab publicly reachable.
- Support repository discovery, code reading, search, and approved development workflows.
- Support write operations with explicit approval required for every state-changing operation.
- Restrict all MCP access to projects explicitly included in a local allowlist.
- Require repository code changes to use a dedicated working branch and merge request.
- Permit MR merge only through a separate, explicit approval; keep MR review approval human-only.
- Limit first-release CI/CD access to pipeline and job metadata/status.

## Non-goals

- Exposing the company's GitLab instance to the public internet.
- Accessing every project visible to the GitLab identity without an explicit allowlist entry.
- Writing directly to the default or protected branch.
- Approving or unapproving merge requests on behalf of a human reviewer.
- Triggering, retrying, or cancelling CI/CD jobs/pipelines in the first release.
- Fetching CI/CD job logs/traces or artifacts in the first release.
- Storing GitLab credentials in source control.

## Accepted design baseline

| Area | Decision |
|---|---|
| Git provider | GitLab Self-Managed |
| Target version | GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier |
| Network | Private/internal; workstation already has access |
| MCP location | Local workstation |
| Clients | ChatGPT Desktop first; Codex is an acceptable fallback |
| Project access | Explicit local project allowlist |
| Repository code changes | Dedicated working branch followed by MR |
| Merge | Requires a verified human GitLab approval plus separate, operation-specific MCP approval |
| MR approval | Human-only; MCP may read approval state |
| CI/CD | Read pipeline/job metadata and status only |
| Job logs/artifacts | Deferred, excluded from first release |
| Write safety | Explicit approval for every mutation |
| GitLab authentication | User's Personal Access Token; actions are attributed to the user's human GitLab account |
| Token scope | api for the combined read/write REST API workflow |
| Token resource reach | All projects/groups accessible to the PAT owner; the MCP allowlist is enforced separately server-side |

The selected PAT is not project-scoped. Its effective reach follows the user's GitLab memberships and permissions; therefore, the MCP allowlist is a mandatory server-side control, not a restriction on the token itself. GitLab audit history attributes MCP operations to the PAT owner's human account rather than a bot identity. See [ADR-0015](docs/decisions/0015-gitlab-token-and-execution-identity.md) and the [authentication model](docs/security/authentication-model.md).

## GitLab Enterprise Edition 18.8.4-ee with Free subscription constraints

- Basic code search is limited to a project; advanced/exact and global/group code search are not part of the Free-tier baseline.
- MR approvals are optional on Free; required approval rules are not included. The accepted MCP policy requires one verified human GitLab approval plus a separate MCP merge confirmation.
- Job logs and artifacts remain excluded from the first release.

GitLab 18.8 is outside the maintained release lines listed by GitLab as of 2026-11-10. Production use should be accompanied by an upgrade plan or explicit risk acceptance.

## Proposed technical choices

The following choices remain Proposed and are not implementation commitments yet:

- Implementation language: Go.
- Primary local transport: STDIO.
- GitLab integration: REST API v4.

Existing GitLab MCP implementations are under a documentation-only security evaluation (DES-012). No upstream code has been selected or adopted; see [the evaluation](docs/research/existing-gitlab-mcp-evaluation.md).

See [Authentication Model](docs/security/authentication-model.md), [Architecture Decision Records](docs/decisions/README.md), [Capability Matrix](docs/gitlab/capability-matrix.md), [Client Compatibility](docs/architecture/client-compatibility.md), [GitLab Environment Profile](docs/environment/gitlab-target.md), [Detailed Task Backlog](planning/tasks.md), [Roadmap](planning/roadmap.md), and [Decisions Log](planning/decisions-log.md).

## Important data-handling note

GitLab being private does not mean repository content stays local to the workstation. Content returned by MCP tools may be sent to the AI provider for model processing. Use only projects and data authorized for this workflow.

## Repository hygiene

This repository is public. Never commit internal GitLab URLs, hostnames, IP addresses, organization-specific project names, customer information, private source code, credentials, tokens, private keys, or unredacted operational logs.

## Status

Design and requirements phase. No application implementation code is included or authorized yet.
