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
| Target version | 18.x; exact minor/patch and edition to be confirmed |
| Network | Private/internal; workstation already has access |
| MCP location | Local workstation |
| Clients | ChatGPT Desktop first; Codex is an acceptable fallback |
| Project access | Explicit local project allowlist |
| Repository code changes | Dedicated working branch followed by MR |
| Merge | Supported only with separate, operation-specific approval |
| MR approval | Human-only; MCP may read approval state |
| CI/CD | Read pipeline/job metadata and status only |
| Job logs/artifacts | Deferred, excluded from first release |
| Write safety | Explicit approval for every mutation |

## Proposed technical choices

The following choices remain Proposed and are not implementation commitments yet:

- Implementation language: Go.
- Primary local transport: STDIO.
- GitLab integration: REST API v4.

See [Architecture Decision Records](docs/decisions/README.md), [Capability Matrix](docs/gitlab/capability-matrix.md), [Detailed Task Backlog](planning/tasks.md), [Roadmap](planning/roadmap.md), and [Decisions Log](planning/decisions-log.md).

## Important data-handling note

GitLab being private does not mean repository content stays local to the workstation. Content returned by MCP tools may be sent to the AI provider for model processing. Use only projects and data authorized for this workflow.

## Repository hygiene

This repository is public. Never commit internal GitLab URLs, hostnames, IP addresses, organization-specific project names, customer information, private source code, credentials, tokens, private keys, or unredacted operational logs.

## Status

Design and requirements phase. No application implementation code is included or authorized yet.
