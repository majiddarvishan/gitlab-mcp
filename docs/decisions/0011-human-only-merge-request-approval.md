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
