# GitLab MCP

A local Model Context Protocol (MCP) server that enables ChatGPT Desktop and Codex to work with a self-managed GitLab instance that is reachable from the user's workstation.

The project is being designed documentation-first. No implementation is considered approved until the relevant requirements and architecture decisions are reviewed.

## Goals

- Connect AI-assisted development workflows to a self-managed GitLab instance without making GitLab publicly reachable.
- Support repository discovery, code reading, search, and project-management context.
- Support write operations from the outset, with explicit approval required for every state-changing operation.
- Keep the MCP server local to the workstation and use the workstation's existing network/VPN access to GitLab.
- Keep the public project repository free of company-specific details, credentials, private source code, and internal network identifiers.

## Non-goals

- Exposing the company's GitLab instance to the public internet.
- Storing GitLab credentials in source control or in tool descriptions.
- Automatically performing writes without per-operation user approval.
- Implementing the server before the open design decisions are resolved.

## Current design direction

| Area | Current position | Status |
|---|---|---|
| Git provider | GitLab Self-Managed | Accepted |
| GitLab reachability | Workstation can reach GitLab on the internal network | Confirmed environment constraint |
| MCP location | Local workstation | Accepted |
| Clients | ChatGPT Desktop first; Codex is an acceptable fallback | Accepted |
| Access model | Read and write capabilities | Accepted |
| Write safety | Explicit approval for every state-changing operation | Accepted |
| Implementation language | Go | Proposed; not yet ratified |
| MCP transport | STDIO | Proposed; not yet ratified |
| GitLab API | REST API | Proposed; not yet ratified |
| Project repository visibility | Public documentation repository | Accepted |

## Important client distinction

This project targets the local MCP host supported by ChatGPT Desktop and Codex. The current OpenAI MCP documentation says the ChatGPT desktop app, Codex CLI, and IDE extension share the Codex MCP configuration, and that local STDIO servers can be configured there.

This is distinct from ChatGPT web's Developer Mode custom MCP apps, which connect to remote MCP servers. A local STDIO server should not be assumed to appear as a web-hosted custom app.

Official documentation:
- [MCP in ChatGPT and Codex](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)
- [Developer mode and MCP apps in ChatGPT](https://help.openai.com/en/articles/12584461-developer-mode-and-full-mcp-connectors-in-chatgpt)

## Repository structure

- `docs/requirements/`: functional and non-functional requirements.
- `docs/architecture/`: architecture and client integration notes.
- `docs/security/`: credentials, approval, and data-handling requirements.
- `docs/gitlab/`: capability inventory and operation status.
- `docs/decisions/`: Architecture Decision Records (ADRs).
- `planning/`: roadmap and decision tracking.

## Status

Design and requirements phase. No application code is included yet.

See [the decision log](planning/decisions-log.md) and [roadmap](planning/roadmap.md) for current progress.
