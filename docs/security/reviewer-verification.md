# SEC-009 Test Protocol: Independent Human Reviewer Verification

- **Status:** Test procedure defined; target-instance validation pending
- **Target:** GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier
- **Related tasks:** SEC-009, DES-001
- **Related decisions:** [ADR-0014](../decisions/0014-free-tier-human-approval-merge-gate.md), [ADR-0015](../decisions/0015-gitlab-token-and-execution-identity.md)
- **Last reviewed:** 2026-11-10

## 1. Purpose and security invariant

Before MCP may merge a merge request, it must establish that at least one account in the MR's current `approved_by` list is:

1. A regular human GitLab account, not a bot or internal/system account.
2. Different from the human account that owns the MCP Personal Access Token.
3. Different from the merge request author.

The reviewer-independence rule is an accepted user decision (D-020). The operator procedure is also available in the [target-instance validation runbook](../environment/validation-runbook.md). This document defines how to validate that the target GitLab Self-Managed EE v18.8.4 instance (Free subscription tier) exposes enough API data to enforce it.

**This is a test plan, not evidence that the target instance has passed.** MCP merge remains disabled until the tests below pass on the exact installed patch and CE/EE distribution.

## 2. API surfaces to validate

Use the PAT selected in ADR-0015. Run tests only against a disposable or explicitly approved test project. Never put a PAT value, internal URL, project name, user ID, username, or raw API response in this public repository.

### 2.1 Identify the PAT owner

Request:

```http
GET /api/v4/user
```

Record the returned numeric `id` locally as the expected MCP execution identity. Do not publish it. This value is used only for numeric identity comparison.

### 2.2 Identify the merge request author

Request:

```http
GET /api/v4/projects/:id/merge_requests/:merge_request_iid
```

Read the author numeric user ID from the MR response. Compare IDs, not display names or usernames.

### 2.3 Read the recorded MR approvals

Request:

```http
GET /api/v4/projects/:id/merge_requests/:merge_request_iid/approvals
```

Read the `approved_by[].user.id` values. Do not rely on the `approved` boolean as a substitute for inspecting the approver list. The response shape differs between Community Edition and Enterprise Edition; the CE response includes `approved`, `approved_by`, `user_can_approve`, and `user_has_approved`. The `approved_by` list includes recorded approvers regardless of whether they satisfy an approval rule.

An empty, absent, malformed, or inaccessible `approved_by` list does not satisfy the gate.

### 2.4 Classify each candidate approver

For every distinct numeric approver ID:

1. Request `GET /api/v4/users/:id` using the regular authenticated PAT. The regular-user form of this endpoint is documented for any signed-in user; it is not the administrator-only response. Verify that the returned `id` exactly matches the candidate ID and inspect `bot`, `state`, and `locked`.
2. Require `bot=false`, `state=active`, and `locked=false`. Missing, conflicting, or inaccessible fields fail classification.
3. Request `GET /api/v4/users?username=:username&humans=true&active=true`. Verify that the response contains exactly the candidate numeric ID. `humans=true` filters out bot and internal accounts; `active=true` limits results to active users.
4. The REST `humans` filter was added in GitLab 17.3 (Merge Request [!159200](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/159200), milestone 17.3), so it predates the v18.8.4-ee target. The target-instance check is still required to confirm deployed behavior and access.
5. Do not use `exclude_internal=true` by itself as proof of a human account: it does not exclude project/group bot users. `without_project_bots=true` alone does not establish that an account is not an internal user.
6. If the filter is rejected, ignored, or inconsistent with the single-user response, classification fails closed; do not silently fall back to `bot=false` alone.

The `username` lookup is a lookup aid only. Authorization decisions and identity comparisons must use numeric GitLab user IDs. The regular-user `GET /users/:id` representation is documented for signed-in non-admin users and includes `state`, `locked`, and `bot`. The `humans` REST filter was added in GitLab 17.3 (Merge Request [!159200](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/159200), milestone 17.3), so it predates v18.8.4-ee.

### 2.5 Apply the accepted independence rule

A candidate qualifies only when all checks are successful:

- The account is classified as a regular human by a validated target-instance method.
- The account is active and not locked.
- Its numeric ID is not the PAT owner's ID from `GET /user`.
- Its numeric ID is not the MR author's ID.
- The approval record belongs to the current MR state/head being evaluated.

At least one candidate must qualify. Multiple non-qualifying approvers do not compensate for the absence of a qualifying approver.

## 3. Test matrix

| Case | Expected result |
|---|---|
| Active regular human, distinct from PAT owner and MR author, appears in `approved_by` | Qualifies |
| PAT owner appears as an approver | Does not qualify |
| MR author appears as an approver | Does not qualify |
| Project or group bot appears as an approver | Does not qualify |
| Internal/system bot appears as an approver | Does not qualify |
| Candidate account is blocked, locked, or not active | Does not qualify |
| User lookup returns 403, 404, a different ID, `bot` is missing/true, or state is not active | Does not qualify |
| Candidate is absent from the `humans=true&active=true` result or result does not match the exact numeric ID | Does not qualify |
| Candidate is returned with `locked=true` | Does not qualify |
| `humans=true` is rejected, ignored, or conflicts with the single-user response | Does not qualify; merge is blocked |
| `approved_by` is empty, missing, or cannot be read | Does not qualify |
| One qualifying human and additional non-qualifying approvers exist | Qualifies based on the one verified human |
| Approval disappears or MR head changes before merge | Merge is blocked and the preflight must be repeated |

## 4. Approval freshness

The identity check does not replace the merge preflight in ADR-0014.

Before presenting the MCP merge approval, capture the current MR head SHA and approval state. Immediately before merge, re-read both and verify that the same qualifying independent reviewer remains recorded for that MR state. Any relevant change invalidates the pending MCP merge approval.

## EE response interpretation for v18.8.4-ee

Because the target uses the EE distribution, the `GET .../approvals` endpoint is documented to return the full EE response schema even without a paid license. Useful fields can include `approvals_required`, `approvals_left`, `approval_rules_left`, `state`, and `merge_status`, in addition to `approved_by`.

Do not use `approved=true` as proof of a human approval. On EE, that flag can be true when no approval rules apply even if `approved_by` is empty. For D-020, the gate is satisfied only by at least one account in `approved_by` that passes the regular-human classification and is distinct from both the PAT owner and MR author.

The separate `GET .../approval_state` endpoint for detailed approval-rule information remains Premium/Ultimate and must not be required by the Free-tier workflow.

## 5. What this API check does and does not prove

The approvals API records the GitLab account to which an approval is attributed. It does not prove whether the approval was submitted through the web UI or by an API client, nor does it prove which physical person was operating the account. This project uses the accepted account-independence rule; it must not claim stronger proof of interaction channel or personal presence than GitLab exposes.

The test verifies account classification and separation from the PAT owner and MR author. It does not validate organizational reviewer eligibility, code-owner rules, or paid-tier approval-rule enforcement.

## 6. Evidence and completion criteria

Keep only sanitized evidence in the public repository:

- Exact GitLab version and CE/EE distribution, without hostname or internal identifiers.
- HTTP status and whether each required field/filter was present.
- Whether regular human, bot, internal, and inaccessible-user cases produced the expected classification.
- A pass/fail result for every test-matrix row.
- The chosen classification method and any known limitation.

Do not publish raw responses, user IDs, usernames, project IDs, project paths, internal URLs, token values, or screenshots containing those values.

SEC-009 may be marked DONE only when the target-instance test demonstrates reliable classification of regular human accounts and all negative cases fail closed. Otherwise it remains OPEN and MCP merge remains disabled.

## References

- [GitLab merge request approvals API](https://docs.gitlab.com/api/merge_request_approvals/)
- [GitLab Users API](https://docs.gitlab.com/api/users/) (regular-user `GET /users/:id` fields and `humans` filter)
- [GitLab internal users](https://docs.gitlab.com/administration/internal_users/)
- [GitLab !159200 — REST human-user filter added in milestone 17.3](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/159200)
