# GitLab Target Instance Validation Runbook

- **Status:** Procedure defined; target-instance API validation pending
- **Target:** GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier
- **Last reviewed:** 2026-11-10
- **Related tasks:** DES-001, SEC-009, SEC-010, SEC-011, SEC-012
- **Related profile:** [GitLab target environment](gitlab-target.md)

## 1. Safety and evidence rules

This runbook is for an authorized operator using the actual target GitLab instance. Prefer a disposable test project and test users for approval and permission checks.

Never paste or commit any of the following into this public repository or the chat:
- PAT values, authentication headers, cookies, or raw credential-bearing commands.
- Internal hostnames, URLs, IP addresses, project paths/names, or project IDs.
- User IDs, usernames, names, email addresses, or unredacted API responses.
- Screenshots that expose internal identifiers or credentials.

Keep raw results locally in an approved secure location. Publish only the exact GitLab version/edition line and sanitized PASS/FAIL observations. Do not use production projects for destructive or write tests.

Do not create a PAT solely to complete this checklist until the local credential-storage decision (SEC-002) is settled. Do not relax permissions, branch protection, or instance policy to make a test pass.

## 2. Exact GitLab build (DES-001 — DONE)

The user supplied `GitLab Enterprise Edition v18.8.4-ee`. The project records:
- Distribution: Enterprise Edition (EE).
- Subscription tier: Free.
- Exact version: 18.8.4-ee.
- Public version tag: [v18.8.4-ee](https://gitlab.com/gitlab-org/gitlab/-/tags/v18.8.4-ee).

The subscription tier is not inferred from the EE distribution. If the installed build is upgraded or changed, update the environment profile and repeat version-sensitive API validation.

## 3. Validate independent human reviewer identity (SEC-009)

Use a disposable or explicitly approved project with an open MR and suitable test accounts. The chosen PAT is the user's personal token, so the PAT owner is the MCP execution identity.

### 3.1 Record the PAT owner's identity locally

Call:

```http
GET /api/v4/user
```

Record only the numeric account ID in the local test worksheet. Do not publish the value.

### 3.2 Read the MR author and current head

Call:

```http
GET /api/v4/projects/:id/merge_requests/:merge_request_iid
```

Record the author ID and current source HEAD SHA locally. Do not publish either value.

### 3.3 Read approval records

Call:

```http
GET /api/v4/projects/:id/merge_requests/:merge_request_iid/approvals
```

Inspect `approved_by`; for each entry, record locally the nested user ID and username. Do not rely on the top-level `approved` flag alone. This target is EE, so GitLab documents the full EE response schema even without a paid license. In particular, `approved=true` can be returned when no approval rules apply and `approved_by` is empty. The `approved_by` list remains the relevant record of accounts that approved.

### 3.4 Classify each candidate approver

For each candidate:

1. Call `GET /api/v4/users/:id` as a regular authenticated user (the regular-user endpoint does not require administrator privileges). Confirm the returned numeric ID matches the candidate. Inspect `bot`, `state`, and `locked`; require `bot=false`, `state=active`, and `locked=false`.
2. Call `GET /api/v4/users?username=:username&humans=true&active=true`. Confirm the result contains exactly the candidate's numeric ID. The `humans` filter excludes bot and internal users; `active=true` limits results to active users.
3. The REST `humans` filter was added in GitLab 17.3 (Merge Request [!159200](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/159200), milestone 17.3), so it predates v18.8.4-ee. Still verify its behavior on the actual target instance.
4. Do not use `exclude_internal=true` alone: GitLab documents that it does not exclude project/group bot users. If the filter is rejected, ignored, or inconsistent, fail closed.

Compare numeric user IDs, not names or usernames:
- Candidate reviewer ID must differ from the PAT owner ID.
- Candidate reviewer ID must differ from the MR author ID.

At least one candidate must pass all checks. The MR approval record does not prove whether the action was performed through the UI or API, or who physically operated the account; do not claim that it does.

### 3.5 Check approval freshness

Immediately before any future MCP merge:
- Re-read the MR and approval list.
- Confirm the source HEAD SHA is unchanged from the approved preview.
- Confirm the same qualifying independent human reviewer remains in `approved_by`.
- Reject the merge if the MR, head SHA, approval list, reviewer classification, or merge parameters changed or cannot be verified.


### 3.6 Optional local request helper

For manual checks, use the [one-off API caller](../security/reviewer-verification.md#7-optional-one-off-api-caller-local-terminal-only). It prompts for the PAT without echoing it and sends the header through stdin to curl. Use only an existing PAT while SEC-002 is open; do not create a token before its storage procedure is accepted.

MCP merge remains disabled until SEC-009 passes on the target instance and the merge gate can enforce every negative case.

## 4. Validate the personal PAT boundary (SEC-010)

This is an integration test for the MCP application, not a claim that the PAT itself is project-scoped.

After the allowlist implementation and integration test harness are authorized and available:
1. Select one test project the PAT owner can access and explicitly allowlist it.
2. Select a second project the same account can access but do not allowlist it.
3. Confirm MCP permits only intended operations on the allowlisted project.
4. Confirm MCP rejects every read and write path to the accessible-but-unlisted project before returning its data or sending a mutation.
5. Include nested-resource and cross-project references, aliases, project rename/transfer, ambiguous lookup, and removed/recreated project cases.
6. Confirm the PAT can access the second project directly; record this as the credential blast radius, not as an allowlist bypass in MCP.

Do not perform this test against production data unless explicitly approved.

## 5. Validate effective permissions and branch rules (SEC-011)

For each approved test project, record locally:
- PAT owner's effective project role and relevant group inheritance.
- Whether repository reads, branch creation, file/commit updates, MR creation/update, approval-state reads, and merge are permitted.
- The protected default branch's current allowed-to-merge and allowed-to-push settings.
- Any CI, merge checks, or instance policy that can reject the operation.

GitLab's authorization and branch protections remain authoritative. Do not elevate the account or weaken branch protections to make MCP succeed. A denied operation must remain unavailable to MCP for that project.

## 6. Validate PAT expiry and rotation policy (SEC-012)

The version-tagged v18.8.4-ee PAT documentation states:
- A PAT with no explicitly supplied expiry defaults to 365 days from creation.
- The default maximum lifetime on Free is 365 days.
- A 400-day maximum can be enabled in GitLab 17.6 and later by the `buffered_token_expiration_limit` feature flag; it is disabled by default. Confirm whether the target administrator enabled it.
- A PAT expires at midnight UTC on its expiry date.
- The Rotate UI workflow has been available since GitLab 17.7. Rotation creates fresh credentials, preserves the old token's scope and permissions, and immediately invalidates the old token.

Before provisioning, confirm the actual token-creation UI/instance settings and choose an expiry date. Define the local renewal owner and reminder process.

Do not place the PAT in chat, source control, a plain text config file, shell history, process arguments, or logs. Token creation and rotation must follow the accepted local secret-storage decision. Revoke a compromised token immediately. Use the GitLab Rotate flow for planned renewal only after SEC-002 defines safe credential replacement and storage; a rotated/revoked token cannot continue authenticating.

## 7. Sanitized result template

Use this template in a private worksheet first. Publish only fields marked safe.

| Check | Sanitized result |
|---|---|
| GitLab version/edition line | Safe to publish |
| Subscription tier | Free / other |
| `approved_by` available | PASS / FAIL |
| `GET /users/:id` available to regular user | PASS / FAIL |
| `bot` and `state` fields present | PASS / FAIL |
| `humans=true` filter behaves as documented (introduced in 17.3) | PASS / FAIL |
| Regular human appears in humans filter | PASS / FAIL |
| Project/group/internal bot excluded | PASS / FAIL |
| PAT owner and MR author ID comparisons | PASS / FAIL only |
| Approval freshness check | PASS / FAIL |
| Allowlist integration tests | PASS / FAIL / NOT RUN |
| Effective role/branch tests | PASS / FAIL / NOT RUN |
| Expiry policy confirmed | PASS / FAIL / NOT RUN |
| Outstanding limitation | Sanitized description only |

Do not include IDs, usernames, project identifiers, internal URLs, request headers, tokens, or raw response fragments in the public result.

## 8. Completion gates

- **DES-001 is DONE**: exact patch and EE distribution are recorded as v18.8.4-ee with Free subscription tier.
- **SEC-009** is DONE only after regular human, bot/internal, owner, author, inactive/locked, inaccessible, and stale-approval cases are verified and sanitized evidence is committed.
- **SEC-010** is DONE only after the implemented MCP allowlist passes the integration matrix.
- **SEC-011** is DONE only after actual per-project permissions and protected-branch behavior are recorded.
- **SEC-012** is DONE only after target expiry policy and secure token lifecycle are validated.
- Defining this runbook does not complete any target-instance validation task and does not authorize implementation.
