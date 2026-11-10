# Functional Requirements

## Status
Approved scope baseline for design. Detailed tool schemas and endpoint mapping remain pending.

## Project scope and authorization
- The MCP server must operate only on projects explicitly included in a local allowlist.
- Projects visible to the GitLab identity but absent from the allowlist must be inaccessible through every MCP tool.
- The allowlist must be checked server-side for every project-scoped read and write operation.
- Group membership or project visibility must not implicitly expand the allowlist.
- The initial allowlist model is project-specific. Group-wide recursive access requires a separate decision.

## GitLab discovery and reading
- List configured/allowlisted projects.
- Read project metadata and repository tree.
- Read files at a specified branch, tag, or commit.
- Search code within one allowlisted project using Free-tier basic project search, subject to validation against the installed 18.8 API. Advanced/exact and global/group code search are outside the Free-tier baseline.
- List branches and tags, including default/protected status where available.
- Inspect commits and compare refs.
- Inspect merge requests, diffs, discussions, changed files, and approval state.
- Inspect issues, labels, milestones, and comments.
- Read pipeline and job metadata/status only.

## Repository code changes
- Repository code changes must be made on a dedicated working branch.
- The MCP server must not write directly to the default branch or a protected branch.
- A code change must be proposed through a merge request.
- Branch creation, file changes, commits, and merge request creation/update are separate mutations and each requires its own explicit approval.
- The merge request target branch must be explicit in the preview and approval.

## Merge requests
- MCP may create and update merge requests with explicit approval.
- MCP may read merge request approval status and applicable merge readiness metadata.
- MCP must not approve or unapprove a merge request.
- MCP may merge a merge request only after a separate, explicit approval for that exact merge operation.
- A merge approval must not be inferred from approval to create/update the MR or from approval of a commit.
- GitLab protected-branch permissions, approval rules, CI requirements, merge checks, and instance policies remain authoritative and must not be bypassed.

## CI/CD
- The first release is read-only for CI/CD.
- MCP may read pipeline and job metadata/status.
- MCP must not trigger, retry, cancel, or otherwise mutate pipelines/jobs in the first release.
- Job logs/traces and job artifacts are excluded from the first release and deferred for a separate data-handling review.
- CI/CD variable values and secret values are out of scope.

## Other GitLab mutations
- Create/update issues and add comments/discussions where included in the final capability matrix.
- Every mutation requires explicit user approval for that individual operation.

## Approval
- Every mutation requires explicit user approval for that individual operation.
- The approval preview must identify the target project and all relevant parameters.
- Approval must not be reusable for a different operation or modified parameters.
- The result must include the GitLab response and resulting object identifiers.

## Compatibility
- Target GitLab Self-Managed 18.x using REST API v4.
- GitLab Free 18.8 is confirmed. Exact patch, CE/EE distribution, enabled features, and effective user permissions must be checked during environment validation.
- Unsupported features must be reported clearly and must not silently fall back to a broader operation.

## Configuration
- Configure the GitLab base URL locally.
- Configure credentials without committing them.
- Maintain an explicit local project allowlist.
- Provide a way to inspect effective permissions and enabled capabilities.

## Error handling
- Return actionable errors for authentication, authorization, rate limiting, missing resources, conflicts, and network failures.
- Do not silently retry non-idempotent write operations.
