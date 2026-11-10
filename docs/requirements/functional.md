# Functional Requirements

## Status
Initial draft. Items below define the intended scope and require review through the capability matrix.

## GitLab discovery and reading
- List and inspect accessible GitLab projects.
- Read project metadata and repository tree.
- Read files at a specified branch, tag, or commit.
- Search repository code and text.
- List branches and tags.
- Inspect commits and compare refs.
- Inspect merge requests, diffs, discussions, and changed files.
- Inspect issues, labels, milestones, and comments.
- Inspect pipeline and job status where supported by the GitLab API and permissions.

## GitLab mutations
Write support is in scope from the first release. Candidate operations include:
- Create and delete branches.
- Create or update repository files and commits.
- Create and update merge requests.
- Add comments and discussions.
- Create and update issues, labels, and milestones.
- Trigger or retry supported pipeline/job actions only if explicitly included in the approved capability matrix.

## Approval
- Every mutation requires explicit user approval for that individual operation.
- The approval preview must identify the target project and all relevant parameters.
- Approval must not be reusable for a different operation or modified parameters.
- The result must include the GitLab response and resulting object identifiers.

## Configuration
- Configure the GitLab base URL locally.
- Configure credentials without committing them.
- Support an explicit allowlist of projects or groups if selected by the user.
- Provide a way to inspect effective permissions and enabled capabilities.

## Error handling
- Return actionable errors for authentication, authorization, rate limiting, missing resources, conflicts, and network failures.
- Do not silently retry non-idempotent write operations.
