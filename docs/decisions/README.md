# Architecture Decision Records

This directory contains the project's Architecture Decision Records (ADRs).

## Status values
- **Proposed**: under discussion; not binding.
- **Accepted**: approved and binding until superseded.
- **Rejected**: considered and not selected.
- **Superseded**: replaced by a later ADR.

An ADR records context, options, the decision, and consequences. Do not rewrite history to hide a previous decision; create a superseding ADR when a material decision changes.

## Index

| ADR | Subject | Status |
|---|---|---|
| [0001](0001-project-goals.md) | Project goals and boundaries | Accepted |
| [0002](0002-local-mcp.md) | Local MCP architecture | Accepted |
| [0003](0003-write-approval.md) | Per-operation write approval | Accepted |
| [0004](0004-implementation-language.md) | Go implementation language | Proposed |
| [0005](0005-stdio-transport.md) | STDIO transport | Proposed |
| [0006](0006-public-repository.md) | Public project repository | Accepted |
| [0007](0007-client-compatibility.md) | ChatGPT Desktop and Codex client scope | Accepted |
| [0008](0008-project-allowlist.md) | Explicit project allowlist | Accepted |
| [0009](0009-working-branch-and-merge-request.md) | Working branch and merge request workflow | Accepted |
| [0010](0010-merge-with-separate-approval.md) | MCP merge with separate approval | Accepted |
| [0011](0011-human-only-merge-request-approval.md) | Human-only merge request approval | Accepted |
| [0012](0012-read-only-ci-and-deferred-logs.md) | Read-only CI/CD and deferred logs/artifacts | Accepted |
| [0013](0013-gitlab-18x-compatibility.md) | GitLab Self-Managed 18.x compatibility target | Accepted |
| [0014](0014-free-tier-human-approval-merge-gate.md) | Free-tier human approval requirement before MCP merge | Proposed |
