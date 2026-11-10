# ADR-0008: Explicit Project Allowlist

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User

## Context
The GitLab identity used by the local MCP server may be able to access many company projects. The user wants the MCP integration to operate only on projects explicitly authorized for this tool.

## Options
1. Allow every project visible to the GitLab identity.
2. Restrict all operations to an explicit local project allowlist.
3. Allow entire groups and implicitly include all current and future projects in those groups.

## Decision
Use an explicit local allowlist of projects. A project that is visible to the GitLab identity but is not listed is not authorized for MCP access.

The first release uses explicit project entries. Group membership or namespace membership does not automatically authorize all projects in that group. Recursive group allowlisting requires a separate decision.

The server must resolve user-supplied project paths/identifiers to a canonical GitLab project identity and enforce the allowlist server-side on every project-scoped read and write operation.

## Consequences
- Project discovery must not expose unallowlisted projects.
- Every tool path, including cross-project references and nested resources, must be checked.
- Configuration management must provide a deliberate way to add and remove project entries.
- A tool argument must never be able to override the allowlist.
- This policy narrows MCP access; it does not change the underlying GitLab user's permissions.

## Security notes
The allowlist is an application-level boundary, not a replacement for GitLab authorization. The GitLab token must still use least privilege.
