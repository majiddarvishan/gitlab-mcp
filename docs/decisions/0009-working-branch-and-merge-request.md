# ADR-0009: Working Branch and Merge Request Workflow

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User

## Context
The MCP server will support repository code changes. Direct modification of shared/default branches creates unnecessary risk and bypasses the normal review workflow.

## Options
1. Permit direct code writes to any branch with confirmation.
2. Require all code changes to use a dedicated working branch and be proposed through a merge request.

## Decision
Every repository code change must be made on a dedicated working branch and proposed through a merge request.

The MCP server must not write directly to the project's default branch or any protected branch. The target branch of every merge request must be explicit.

Branch creation, each commit operation, and merge request creation/update are separate mutations. Each requires its own approval under ADR-0003.

## Consequences
- The initial code-writing workflow is branch-based.
- The tool must inspect branch protection and relevant project permissions before offering an operation.
- Approval to create a branch does not approve file changes or commits.
- Approval to create a commit does not approve creating an MR.
- Approval to create/update an MR does not approve merging it.
- Direct write-to-default/protected-branch tools are prohibited in the first release.

## Scope
This ADR governs repository code changes. It does not by itself decide whether issue, comment, wiki, or other project-management mutations are included in the first release.
