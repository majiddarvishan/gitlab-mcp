# ADR-0010: Allow MCP Merge with Separate Approval

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User

## Context
The user wants the MCP server to be able to complete a merge request workflow, including merging, but merging is a high-impact operation and must not be implied by earlier approvals.

## Options
1. MCP cannot merge; the user must always merge manually in GitLab.
2. MCP may merge only after a separate, explicit approval for the exact merge operation.

## Decision
MCP may merge a merge request, provided that the merge is a distinct high-impact mutation and receives its own explicit, single-use approval.

Before requesting approval, the server must re-read and validate the MR's current state, project, source and target refs, source HEAD SHA, merge readiness, and available approval/pipeline status. The preview must state the exact MR and merge options.

GitLab remains authoritative for user permissions, protected branch rules, required approvals, CI requirements, merge checks, and instance-level policies. MCP must never bypass or work around a GitLab rejection.

Any material change to the MR or merge parameters after approval invalidates the approval and requires a new preview and approval.

## Consequences
- Merge is a separate P2/high-risk capability.
- The MCP host must present an explicit approval request specifically for merge.
- The implementation must handle stale MR state and failed merge responses safely.
- Approval to create/update an MR, commit code, or perform another write does not authorize merging.
- The exact merge methods/options and mandatory preflight checks remain technical-design items.

## Reference
- [GitLab Merge Requests API](https://docs.gitlab.com/api/merge_requests/)


## GitLab Free limitation

On the target GitLab Free tier, MR approvals are optional and do not prevent merging when no approval exists. The richer required approval rules are not available. The approval-state API can report the users who approved, but exact response fields depend on CE/EE distribution.

The accepted decision to allow MCP merge with separate MCP approval therefore does not imply that GitLab Free enforces a human approval. Whether MCP should enforce at least one human approval as an additional precondition is still Proposed in ADR-0014.
