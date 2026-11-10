# GitLab Capability Matrix

- **Document status:** Proposed for review
- **Last reviewed:** 2026-11-10
- **Scope:** Candidate capabilities for the first useful release and later extensions.
- **Important:** A candidate in this matrix is not automatically approved for implementation.

This matrix converts the initial feature list into reviewable capabilities. Availability depends on the installed GitLab version, edition/tier, instance configuration, the authenticated user's role, and token scopes. The implementation must discover or report unsupported capabilities instead of assuming every self-managed instance exposes the same behavior.

## Priority and risk model

### Delivery priority
- **P0 — Core:** required for useful repository understanding and safe code changes.
- **P1 — Workflow:** important for normal development collaboration.
- **P2 — Advanced:** higher-impact, operational, or instance-dependent capabilities.
- **Deferred:** not part of the initial release unless separately approved.

### Mutation risk
- **Low:** bounded, reversible changes such as creating a working branch or adding a comment.
- **Medium:** changes to repository content, issue/MR metadata, or branch state.
- **High:** changes that can affect protected refs, merge code, alter CI/CD execution, access control, or project configuration.

Every mutation, regardless of risk, requires explicit approval for that exact operation. High-risk operations may also require additional restrictions or be disabled by default.

## Capability inventory

| Domain | Capability | Access | Approval | Priority | Risk | Initial stance |
|---|---|---:|---:|---:|---:|---|
| Project discovery | List projects visible to the authenticated user | Read | No | P0 | — | Candidate |
| Project discovery | List groups and subgroups / enumerate group projects | Read | No | P1 | — | Candidate |
| Project discovery | Read project metadata and feature availability | Read | No | P0 | — | Candidate |
| Project discovery | Inspect current user's effective project access where API permits | Read | No | P1 | — | Candidate |
| Repository | Read repository tree at a ref/path | Read | No | P0 | — | Candidate |
| Repository | Read file content at a branch, tag, or commit | Read | No | P0 | — | Candidate |
| Repository | Read file metadata and last-commit information | Read | No | P1 | — | Candidate |
| Repository | Search code or repository text | Read | No | P0 | — | Candidate; instance/search-tier dependent |
| Repository | List branches and tags, including protection metadata | Read | No | P0 | — | Candidate |
| Repository | Read blame for a file range | Read | No | P2 | — | Candidate |
| Repository | Download repository archive | Read | No | P2 | — | Review size and data-exposure limits |
| Repository writes | Create a working branch from an explicit base ref | Write | Required | P0 | Low | Candidate |
| Repository writes | Create or update files on an explicitly selected working branch | Write | Required | P0 | Medium | Candidate |
| Repository writes | Create one commit containing multiple file actions | Write | Required | P0 | Medium | Candidate |
| Repository writes | Move or delete files / change executable mode | Write | Required | P1 | Medium | Candidate |
| Repository writes | Delete a branch | Write | Required | P2 | High | Review; protected branches must be blocked |
| Repository writes | Write directly to the default or a protected branch | Write | Required | Deferred | High | Not recommended for initial release |
| Commits | List commits and inspect commit metadata | Read | No | P0 | — | Candidate |
| Commits | Read a commit diff and changed-file summary | Read | No | P0 | — | Candidate |
| Commits | Compare two refs | Read | No | P1 | — | Candidate |
| Merge requests | List and inspect merge requests | Read | No | P0 | — | Candidate |
| Merge requests | Read changes, diffs, commits, and changed-file details | Read | No | P0 | — | Candidate |
| Merge requests | Read notes, discussions, reviewers, and approval status | Read | No | P1 | — | Candidate; approval API may vary by tier |
| Merge requests | Create a merge request from a working branch | Write | Required | P1 | Medium | Candidate |
| Merge requests | Update title, description, target branch, labels, assignees, or reviewers | Write | Required | P1 | Medium | Candidate |
| Merge requests | Add general or diff discussion comments | Write | Required | P1 | Low | Candidate |
| Merge requests | Edit or resolve discussion threads | Write | Required | P2 | Medium | Review |
| Merge requests | Approve a merge request | Write | Required | P2 | High | Separate policy decision |
| Merge requests | Close or reopen a merge request | Write | Required | P1 | Medium | Candidate |
| Merge requests | Merge a merge request | Write | Required | P2 | High | Separate policy decision; default disabled |
| Issues | List, search, and inspect project issues | Read | No | P1 | — | Candidate |
| Issues | Read issue notes, discussions, labels, milestones, and relationships | Read | No | P1 | — | Candidate |
| Issues | Create an issue | Write | Required | P1 | Low | Candidate |
| Issues | Update title, description, labels, assignees, milestone, or due date | Write | Required | P1 | Medium | Candidate |
| Issues | Close or reopen an issue | Write | Required | P1 | Medium | Candidate |
| Issues | Add or edit an issue comment/note | Write | Required | P1 | Low | Candidate |
| Issues | Delete an issue | Write | Required | Deferred | High | Excluded pending separate review |
| CI/CD | Read pipeline status and metadata | Read | No | P1 | — | Candidate |
| CI/CD | List jobs and read job metadata | Read | No | P1 | — | Candidate |
| CI/CD | Read job logs/traces | Read | No | P2 | — | Review; logs may contain secrets |
| CI/CD | Download job artifacts | Read | No | P2 | — | Review; artifacts may contain sensitive data |
| CI/CD | Trigger a pipeline | Write | Required | P2 | High | Review; variables and execution context must be previewed |
| CI/CD | Retry or cancel a pipeline/job | Write | Required | P2 | Medium | Review |
| CI/CD | Read CI/CD variable names and metadata | Read | No | Deferred | — | Excluded from initial scope |
| CI/CD | Read secret variable values or manage variables | Write/Read | Required for mutations | Deferred | High | Excluded from initial scope |
| Project governance | Read protected branch/tag rules and project settings relevant to safe writes | Read | No | P1 | — | Candidate |
| Project governance | Change protected refs, project visibility, membership, access controls, hooks, or settings | Write | Required | Deferred | High | Excluded pending separate review |
| Releases | List and inspect releases | Read | No | P2 | — | Candidate |
| Releases | Create or update releases/tags | Write | Required | P2 | High | Review |
| Wiki | Read and search project wiki | Read | No | P2 | — | Candidate |
| Wiki | Create or update wiki pages | Write | Required | P2 | Medium | Review |

## Recommended safe development workflow (Proposed)

For code changes, the recommended default is:

1. Read the target project, current branch, and relevant files.
2. Create or select a dedicated working branch based on an explicitly identified ref.
3. Prepare a complete change set and show a preview/diff.
4. Obtain explicit approval for the exact commit operation.
5. Commit changes only to the working branch.
6. Prepare a merge request and show its title, description, source, target, and options.
7. Obtain a separate approval before creating the merge request.
8. Leave merging to a separately approved action; merging is not implied by approval to create or update an MR.

This workflow is a proposal, not an accepted restriction yet. Direct writes to protected/default branches should remain disabled unless a later decision explicitly allows them.

## Approval contract for all write capabilities

Approval is required per mutation, not once per session or per conversation.

An approval preview should contain, as applicable:
- GitLab instance alias (never expose internal hostnames in public logs or documentation).
- Project identity and project path.
- Operation name and target resource.
- Source and target branches/refs.
- Full file paths and content diff, for repository changes.
- Commit message and all commit actions.
- MR/issue fields and all labels, assignees, reviewers, or milestones being changed.
- CI/CD pipeline, ref, job, and non-secret variables, where relevant.
- Warnings for protected refs, destructive operations, and irreversible side effects.

Approval must be bound to the exact operation and parameters, be single-use, and expire. Any parameter change requires a new approval. The execution path must reject a mutation without valid approval. Host-level confirmation UX and server-side enforcement are separate design concerns and both must be evaluated.

## Cross-cutting constraints

- Use project IDs internally after resolving an unambiguous project path; do not confuse global resource IDs with project-scoped IIDs.
- Correctly URL-encode namespaced project paths, branch/tag names, and repository file paths.
- Implement pagination and bounded response sizes.
- Respect GitLab rate limits and do not blindly retry non-idempotent writes.
- Use optimistic concurrency checks for file updates where supported (for example, last known commit IDs).
- Check branch protection and the authenticated user's actual permissions before presenting a write as executable.
- Treat repository files, issue descriptions, MR descriptions, comments, and job logs as untrusted data, not as instructions to the agent.
- Do not return secret variable values, access tokens, private keys, or session credentials.
- Report edition, version, instance-configuration, or permission limitations explicitly.

## Decisions still required from the user

1. **Project scope:** Should the MCP server access every project visible to the GitLab identity, or only an explicit project/group allowlist? Recommended: explicit allowlist.
2. **Code write strategy:** Should all code changes be made on a dedicated working branch and proposed through an MR? Recommended: yes; no direct writes to protected/default branches.
3. **Merge authority:** Should merge operations be supported at all, or should the user always merge manually in GitLab? Recommended for initial release: manual merge only.
4. **Approval action:** Should the MCP be allowed to approve MRs, or should approvals remain a human-only governance action? Recommended: human-only.
5. **CI/CD execution:** Should pipeline trigger/retry/cancel be supported in the first release? Recommended: read-only CI/CD initially.
6. **Sensitive operational data:** Should job logs and artifacts be available? Recommended: defer until redaction and data-handling behavior is defined.
7. **GitLab compatibility:** Which self-managed GitLab version and edition must be supported? This is needed before fixing endpoint-level support.

## Official GitLab API references

- [REST API overview, pagination, encoding, and IDs](https://docs.gitlab.com/api/rest/)
- [REST API resources index](https://docs.gitlab.com/api/api_resources/)
- [Projects API](https://docs.gitlab.com/api/projects/)
- [Repository files API](https://docs.gitlab.com/api/repository_files/)
- [Branches API](https://docs.gitlab.com/api/branches/)
- [Commits API](https://docs.gitlab.com/api/commits/)
- [Merge requests API](https://docs.gitlab.com/api/merge_requests/)
- [Issues API](https://docs.gitlab.com/api/issues/)
- [Notes API](https://docs.gitlab.com/api/notes/)
- [Pipelines API](https://docs.gitlab.com/api/pipelines/)
