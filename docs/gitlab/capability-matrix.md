# GitLab Capability Matrix

- **Document status:** Partially accepted
- **Last reviewed:** 2026-11-10
- **Compatibility target:** GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier
- **API family:** GitLab REST API v4
- **Scope:** First-release capabilities and explicitly deferred operations.

The access boundary, repository code workflow, merge/approval policy, CI/CD policy, and GitLab major-version target have been accepted. Candidate features outside those decisions remain subject to detailed capability and tool-contract review.

Availability may depend on the exact GitLab 18.x minor/patch version, edition, instance configuration, authenticated user's role, and token scopes. The implementation must detect and report unsupported features rather than assume identical behavior across installations.

## Priority and risk model

### Delivery priority
- **P0 — Core:** required for useful repository understanding and safe code changes.
- **P1 — Workflow:** important for normal development collaboration.
- **P2 — Advanced:** higher-impact or instance-dependent capabilities.
- **Deferred:** excluded from the first release unless separately approved.

### Mutation risk
- **Low:** bounded and generally reversible changes, such as adding a comment.
- **Medium:** repository content, issue/MR metadata, or working-branch changes.
- **High:** merge, protected refs, CI/CD execution, access control, or project configuration.

Every mutation requires explicit approval for that exact operation. High-risk operations additionally require preflight checks and may be disabled when GitLab policy or permissions do not permit them.

## Accepted access boundary

| Rule | Decision |
|---|---|
| Project access | Explicit local allowlist only |
| Visible but unlisted projects | Denied |
| Group membership | Does not implicitly authorize every group project |
| Project path | Resolved to canonical project identity before authorization |
| Allowlist check | Enforced server-side on every project-scoped tool call |
| Caller-supplied project identifier | Never bypasses allowlist validation |

The first release uses explicit project entries. Recursive group allowlisting or automatic authorization of newly created projects requires a separate decision.

## Capability inventory

| Domain | Capability | Access | Approval | Priority | Risk | First-release decision |
|---|---|---:|---:|---:|---:|---|
| Project discovery | List explicitly allowlisted projects | Read | No | P0 | — | Accepted |
| Project discovery | Read metadata for an allowlisted project | Read | No | P0 | — | Accepted |
| Project discovery | List groups/subgroups and discover unallowlisted projects | Read | No | — | — | Excluded |
| Project discovery | Inspect effective user access where API permits | Read | No | P1 | — | Candidate |
| Repository | Read repository tree at a ref/path | Read | No | P0 | — | Accepted |
| Repository | Read file content at a branch, tag, or commit | Read | No | P0 | — | Accepted |
| Repository | Read file metadata and last-commit information | Read | No | P1 | — | Candidate |
| Repository | Basic code search within one allowlisted project | Read | No | P0 | — | Accepted subject to 18.8 Free API validation; project-level only |
| Repository | Advanced/exact/regex code search and group/global code search | Read | No | — | — | Not available in Free; excluded from first release |
| Repository | List branches and tags, including default/protected status | Read | No | P0 | — | Accepted |
| Repository | Read blame for a file range | Read | No | P2 | — | Candidate |
| Repository | Download repository archive | Read | No | P2 | — | Deferred pending response-size/data review |
| Repository writes | Create a dedicated working branch from an explicit base ref | Write | Required | P0 | Low | Accepted |
| Repository writes | Create/update files on the dedicated working branch | Write | Required | P0 | Medium | Accepted |
| Repository writes | Create a commit containing approved file actions on the working branch | Write | Required | P0 | Medium | Accepted |
| Repository writes | Move/delete files or change executable mode on the working branch | Write | Required | P1 | Medium | Candidate |
| Repository writes | Delete a working branch | Write | Required | P2 | High | Candidate; protected/default branches must be blocked |
| Repository writes | Write directly to the default branch | Write | Required | — | High | Prohibited |
| Repository writes | Write directly to any protected branch | Write | Required | — | High | Prohibited |
| Commits | List and inspect commits | Read | No | P0 | — | Accepted |
| Commits | Read commit diff and changed-file summary | Read | No | P0 | — | Accepted |
| Commits | Compare two refs | Read | No | P1 | — | Candidate |
| Merge requests | List and inspect merge requests | Read | No | P0 | — | Accepted |
| Merge requests | Read changes, diffs, commits, and changed-file details | Read | No | P0 | — | Accepted |
| Merge requests | Read notes, discussions, reviewers, and approval state | Read | No | P1 | — | Accepted |
| Merge requests | Create an MR from a working branch | Write | Required | P0 | Medium | Accepted |
| Merge requests | Update MR title/description/target/labels/assignees/reviewers | Write | Required | P1 | Medium | Candidate |
| Merge requests | Add general or diff discussion comments | Write | Required | P1 | Low | Candidate |
| Merge requests | Edit or resolve discussion threads | Write | Required | P2 | Medium | Candidate |
| Merge requests | Approve or unapprove an MR | Write | — | — | High | Prohibited; human-only |
| Merge requests | Merge an MR | Write | Separate approval required | P2 | High | Accepted, subject to GitLab merge checks |
| Merge requests | Close or reopen an MR | Write | Required | P1 | Medium | Candidate |
| Issues | List/search/inspect issues | Read | No | P1 | — | Candidate |
| Issues | Read notes, discussions, labels, milestones, and relationships | Read | No | P1 | — | Candidate |
| Issues | Create/update/close issues or add comments | Write | Required | P1 | Low–Medium | Candidate |
| Issues | Delete an issue | Write | Required | Deferred | High | Excluded pending separate review |
| CI/CD | Read pipeline status and metadata | Read | No | P1 | — | Accepted |
| CI/CD | Read job status and metadata | Read | No | P1 | — | Accepted |
| CI/CD | Read job logs/traces | Read | No | — | — | Deferred; excluded from first release |
| CI/CD | Download job artifacts | Read | No | — | — | Deferred; excluded from first release |
| CI/CD | Trigger a pipeline | Write | — | — | High | Excluded from first release |
| CI/CD | Retry/cancel a pipeline or job | Write | — | — | Medium | Excluded from first release |
| CI/CD | Read or change CI/CD variables | Read/Write | — | — | High | Excluded from first release |
| Project governance | Read branch protection and merge-readiness information needed for safe operations | Read | No | P1 | — | Accepted where available |
| Project governance | Change project settings, visibility, membership, hooks, or access controls | Write | Required | Deferred | High | Excluded pending separate review |
| Releases | List and inspect releases | Read | No | P2 | — | Candidate |
| Releases | Create/update releases or tags | Write | Required | P2 | High | Candidate |
| Wiki | Read/search project wiki | Read | No | P2 | — | Candidate |
| Wiki | Create/update wiki pages | Write | Required | P2 | Medium | Candidate |

## Accepted repository code workflow

All repository code changes must follow this sequence:

1. Resolve the project and confirm it is in the local allowlist.
2. Read the target project, current branch, protected-branch rules, and relevant files.
3. Create or use a dedicated working branch based on an explicitly identified base ref.
4. Prepare the complete change set and present a diff/preview.
5. Obtain explicit approval for the branch creation or selection mutation, if a new branch is created.
6. Obtain explicit approval for each commit operation, including its complete file actions and commit message.
7. Create a merge request from the working branch, with an explicit target branch and a separate approval.
8. Do not write directly to the default branch or any protected branch.

Approval of a branch, commit, or merge request creation does not authorize any subsequent operation.

## Accepted merge policy

**Two independent approvals are mandatory:** at least one verified human approval recorded in GitLab and a separate, exact-operation approval from the user through MCP. This is an MCP-side requirement because GitLab Free approvals are optional.

MCP may merge a merge request, but only as a distinct high-impact operation with its own explicit approval.

Before requesting approval, the MCP must:
- Confirm the project is allowlisted.
- Confirm the merge request belongs to the allowlisted project.
- Read the current MR state, source branch/head, target branch, and merge readiness.
- Read the Free-tier-compatible approval state and verify at least one approver is a regular human account using the Users API.
- Reject bot/internal approvers and fail closed when an approver cannot be classified.
- Read relevant pipeline status where available.
- Identify whether the MR is draft, conflicted, blocked, or otherwise not mergeable.
- Display the exact project, MR IID, source/target refs, current source HEAD SHA, and merge options.
- Clearly state that GitLab's own authorization, branch protection, approval rules, CI requirements, and merge checks remain authoritative.

The merge operation must have a separate, single-use user approval in addition to the verified human GitLab approval. Any material change to the MR or merge parameters invalidates the approval and requires a new preview. The MCP must not attempt to bypass a GitLab rejection or policy gate.

## Accepted human-only MR approval policy

- MCP may read merge request approval status using the Free-tier-compatible approval-state API.
- On the Free subscription tier, approvals are optional and do not block a merge by default. This target is EE, so the `/approvals` endpoint returns the full EE response schema even without a paid license; `approved` can be true with an empty `approved_by` list when no approval rules apply.
- The richer approval-rules/details API is Premium/Ultimate; do not depend on it for this target.
- MCP must not expose approve, unapprove, or reset-approval write tools.
- A human reviewer must perform approval in the GitLab UI or another explicitly human-operated GitLab workflow.
- Merge approval in ChatGPT is not an MR review approval and must never be represented as one.

## Accepted CI/CD scope

The first release is read-only for CI/CD:
- Read pipeline metadata/status.
- Read job metadata/status.

The first release must not:
- Trigger, retry, or cancel pipelines/jobs.
- Fetch job logs/traces.
- Download job artifacts.
- Read secret CI/CD variable values or modify CI/CD variables.

Job logs and artifacts are deferred until a separate review defines redaction, size limits, and sensitive-data handling.

## Approval contract for all write capabilities

Approval is required per mutation, not once per session or conversation.

An approval preview should contain, as applicable:
- A safe project identifier and project path.
- Operation name and target resource.
- Source and target branches/refs.
- Full file paths and content diff for repository changes.
- Commit message and complete commit actions.
- MR/issue fields and all labels, assignees, reviewers, or milestones being changed.
- For merge: MR IID, source/target refs, current source HEAD SHA, merge method/options, readiness warnings, and the available human-approval state.

**Accepted Free-tier policy (ADR-0014):** GitLab Free does not enforce required MR approvals, so MCP adds its own gate. At least one verified human account must appear in the Free-tier-compatible `approved_by` list, and the user must separately approve the exact MCP merge operation. If human identity or approval state cannot be verified, MCP refuses to merge.

Approval must be bound to the exact operation and parameters, be single-use, and expire. Any parameter change requires new approval. The execution path must reject a mutation without valid approval. Host-level confirmation UX and server-side enforcement are separate design concerns and both must be evaluated.

## Cross-cutting constraints

- Enforce the local project allowlist server-side for every project-scoped read and write.
- Resolve project paths to canonical IDs before authorization checks.
- Correctly URL-encode namespaced project paths, branch/tag names, and repository file paths.
- Distinguish project-global IDs from project-scoped IIDs.
- Implement pagination and bounded response sizes.
- Respect GitLab rate limits and do not blindly retry non-idempotent writes.
- Use optimistic concurrency checks for file updates where supported.
- Check branch protection, permissions, merge readiness, and approval state before relevant operations.
- Treat repository files, issue descriptions, MR descriptions, comments, and returned metadata as untrusted data, not instructions.
- Do not return secret variables, access tokens, private keys, or session credentials.
- Report version, edition, instance-configuration, or permission limitations explicitly.
- Target GitLab Self-Managed Enterprise Edition v18.8.4-ee with Free subscription tier and REST API v4. Validate version-sensitive behavior against the installed build.

## Remaining design questions

The following are not resolved by the accepted decisions above:
1. Exact GitLab build is confirmed as EE v18.8.4-ee with Free subscription (DES-001 DONE).
2. Whether project allowlist entries will use numeric project IDs only or permit namespaced paths at configuration time (runtime authorization should use canonical IDs).
3. Which candidate issue, discussion, wiki, release, and advanced repository capabilities belong in the first release.
4. Whether direct file deletion/move operations should be included in initial code-writing tools.
5. Whether the project-scoped basic code-search API works as expected on the target 18.8 Free instance; advanced/exact/global/group code search is excluded.
6. Validate the Free-tier human-approver verification against the exact deployed EE v18.8.4 instance (implementation prerequisite; policy is accepted in ADR-0014).
7. Which merge methods/options should be exposed, if any, and which preflight conditions will be mandatory.

## Official GitLab API references

- [REST API overview](https://docs.gitlab.com/api/rest/)
- [Projects API](https://docs.gitlab.com/api/projects/)
- [Repository files API](https://docs.gitlab.com/api/repository_files/)
- [Branches API](https://docs.gitlab.com/api/branches/)
- [Protected branches API](https://docs.gitlab.com/api/protected_branches/)
- [Commits API](https://docs.gitlab.com/api/commits/)
- [Merge requests API](https://docs.gitlab.com/api/merge_requests/)
- [Merge request approvals API](https://docs.gitlab.com/api/merge_request_approvals/)
- [Issues API](https://docs.gitlab.com/api/issues/)
- [Notes API](https://docs.gitlab.com/api/notes/)
- [Pipelines API](https://docs.gitlab.com/api/pipelines/)
