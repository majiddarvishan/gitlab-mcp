# ADR-0011: Human-Only Merge Request Approval

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User

## Context
The MCP server may inspect merge requests and merge them with separate approval, but it must not replace the independent human code-review approval required by the organization's workflow.

GitLab exposes API endpoints that can approve or unapprove a merge request as the currently authenticated user. Exposing such an endpoint to an AI tool could cause an approval to be recorded under the user's GitLab identity.

## Options
1. Allow MCP to approve/unapprove MRs with confirmation.
2. Keep approval as a human-operated GitLab action while allowing MCP to read approval state.

## Decision
Merge request approval is human-only.

MCP may read approval state and, when available, the applicable rules and outstanding requirements. MCP must not expose approve, unapprove, or reset-approval mutation tools.

A ChatGPT/MCP confirmation to merge is not a code-review approval and must never be presented as one. If GitLab requires a human approval, that approval must be performed through GitLab by an eligible human reviewer.

## Consequences
- The tool inventory may include read-only approval-state inspection.
- Approval mutation endpoints are excluded from the MCP tool surface.
- Merge preflight should inspect approval/readiness state where the installed GitLab edition exposes it.
- Missing approval data or an unavailable endpoint must be reported as unknown, not treated as approved.

## References
- [GitLab Merge Request Approvals API](https://docs.gitlab.com/api/merge_request_approvals/)
- [GitLab Merge Request Approval Settings](https://docs.gitlab.com/user/project/merge_requests/approvals/settings/)


## Free-tier limitation confirmed after the decision

GitLab Free permits eligible users to approve MRs, but those approvals are optional and do not block merging. Required approval rules and richer approval-rule configuration require Premium or Ultimate.

The Free-tier approval-state API can report `approved_by`, but response fields differ between Community Edition and Enterprise Edition. The richer `approval_state` and approval-rule details endpoints are not available in Free.

This ADR means MCP must never perform an approval. It does not, by itself, specify whether MCP must require a human approval before it executes a separately approved merge. That separate policy is documented as Proposed in ADR-0014.
