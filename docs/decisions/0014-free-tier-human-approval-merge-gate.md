# ADR-0014: Require Human MR Approval Before MCP Merge on GitLab Free

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User

## Context

The target is GitLab Self-Managed Free 18.8. GitLab Free allows eligible users to approve merge requests, but approvals are optional and do not prevent a merge when no approval exists. Required approval rules are a Premium/Ultimate capability.

The project already accepts:
- MR review approval is a human-only action; MCP must not approve or unapprove an MR.
- MCP may merge an MR only after a separate, exact-operation user approval.

Those two approvals serve different purposes:
1. A human reviewer records a code-review approval in GitLab.
2. The user separately authorizes MCP to execute the merge operation.

## Decision

**MCP must not merge a merge request unless both of the following conditions are satisfied:**

1. At least one verified human GitLab reviewer, distinct from both the MCP PAT owner and the MR author, has an approval recorded for the current merge request.
2. The user explicitly approves the exact merge operation through the MCP host.

These are independent requirements. Neither substitutes for the other.

This is an application-level policy gate because GitLab Free does not enforce required approval rules itself.

## Human approval verification

The required target-instance validation procedure is defined in [SEC-009 reviewer verification](../security/reviewer-verification.md).

The first-release implementation must:
- Read the GitLab Free-compatible approval state through `GET /projects/:id/merge_requests/:merge_request_iid/approvals`.
- Inspect the `approved_by` list; do not rely only on the boolean `approved`, because its semantics and available response fields differ by CE/EE distribution.
- Resolve each candidate approver through the GitLab Users API and verify the account is a regular human account (for example, `bot=false` on `GET /users/:id` or an equivalent supported human-user filter).
- Reject bot/internal accounts and fail closed if the approver identity cannot be reliably classified.
- Exclude the MCP PAT owner and the MR author from qualifying approvers, even when either account is otherwise classified as human.
- Require at least one verified human approval from a qualifying independent reviewer. A missing, empty, unavailable, or ambiguous approval record is not sufficient.
- Never create, modify, or remove an MR approval. The user must record the review approval in GitLab through a human-operated workflow.

The GitLab approval API's `approved_by` array includes all approvers, regardless of whether those approvals satisfy approval rules. GitLab Free does not provide the Premium/Ultimate `approval_state` endpoint, so MCP must not depend on that endpoint or on required approval-rule fields.

## Approval freshness and merge preflight

Before requesting the MCP merge approval:
- Wait until GitLab has finished processing the current MR diff.
- Read the current MR state, source and target refs, source HEAD SHA, merge readiness, pipeline status, and approval state.
- Verify at least one qualifying independent human approver (not the PAT owner or MR author) is recorded for the current MR.
- Present the qualifying approver identity, current source HEAD SHA, target branch, and merge options in the preview.

Immediately before executing the merge:
- Re-read the MR and approval state.
- Verify the source HEAD SHA, target branch, project, MR IID, and merge options still match the approved preview.
- Verify the same qualifying independent human approval is still present.
- Reject and request a new review/approval if the MR changed, the approval disappeared, the approver is the PAT owner or MR author, the identity state is ambiguous, or GitLab reports the MR is not mergeable.

GitLab documentation states that approvals are normally removed when new changes are added after approval; the Premium/Ultimate setting that changes approval-removal behavior is not part of the Free-tier baseline. The implementation must still re-read the state rather than assume an approval remains valid.

## Separate MCP merge authorization

After the human GitLab approval has been verified, MCP must request a separate, explicit approval for the merge operation.

The MCP approval must be:
- Bound to the exact project, MR IID, source and target refs, source HEAD SHA, merge method/options, and relevant preflight state.
- Single-use and time-limited.
- Invalidated by any material change to the MR or merge parameters.
- Checked by the execution layer; a model-generated statement must never count as user approval.

## Fail-closed behavior

MCP must refuse to merge if:
- There is no recorded qualifying human approval.
- All recorded approvers are bots or internal accounts.
- The only human approvers are the PAT owner and/or MR author.
- The approver identity cannot be checked.
- The approval state cannot be read from GitLab.
- The MR head or relevant parameters changed after approval.
- GitLab policy, permission, protection, CI, or merge-readiness checks reject the merge.

GitLab remains authoritative for user permissions, protected branch rules, CI requirements, merge checks, and instance-level policies. MCP must never bypass or work around a GitLab rejection.

## Consequences

- GitLab Free's optional approval mechanism is supplemented by an application-level MCP gate.
- Every MCP merge requires two independent approvals: a recorded human GitLab review approval and a separate MCP operation approval.
- MCP may read approval state but has no approval mutation tools.
- The merge preflight must verify reviewer independence from both the PAT owner and MR author.
- If the deployed CE/EE distribution does not expose enough information to verify a qualifying independent human approver safely, merge remains disabled until a safe verification method is established.
- GitLab 18.8 Free remains a supported compatibility target, but exact patch and CE/EE distribution must be confirmed.

## References

- [GitLab merge request approvals](https://docs.gitlab.com/user/project/merge_requests/approvals/)
- [GitLab merge request approvals API](https://docs.gitlab.com/api/merge_request_approvals/)
- [GitLab Users API](https://docs.gitlab.com/api/users/)
- [GitLab merge request approval settings](https://docs.gitlab.com/user/project/merge_requests/approvals/settings/)
