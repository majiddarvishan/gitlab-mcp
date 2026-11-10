# GitLab Target Instance Validation Runbook

- **Status:** Procedure defined; target-instance results pending
- **Target:** GitLab Self-Managed Free 18.8
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

## 2. Identify the exact GitLab build (DES-001)

1. Sign in using the normal browser.
2. Open **Help > Help**.
3. Copy only the product/version line shown on that page.
4. Determine whether the distribution is Community Edition (CE) or Enterprise Edition (EE).
5. Keep the subscription tier separate from the distribution label. An EE distribution can be running without a paid license and still expose edition-specific response fields.
6. Provide only a sanitized string such as `GitLab Community Edition 18.8.x` or `GitLab Enterprise Edition 18.8.x`.

**Required output:** exact patch plus CE/EE. Do not send the page URL, hostname, screenshots, or package/configuration output.

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

Inspect `approved_by`; for each entry, record locally the nested user ID and username. Do not rely on the top-level `approved` flag alone. The returned fields vary between CE and EE, but the `approved_by` list is the relevant record of accounts that approved.

### 3.4 Classify each candidate approver

For each candidate:

1. Call `GET /api/v4/users/:id` as a regular authenticated user. Confirm the returned numeric ID matches the candidate. Inspect `bot` and `state`. A missing or contradictory field fails classification.
2. Call `GET /api/v4/users?username=:username&humans=true&active=true`. Confirm the result contains exactly the candidate's numeric ID. The `humans` filter is documented to include regular users and exclude bot and internal users.
3. If the returned list exposes `locked`, it must be `false`. If the required identity or state information is absent, ambiguous, or inconsistent, fail closed.
4. Do not use `exclude_internal=true` alone: GitLab documents that it does not exclude project/group bot users.
5. Validate all of the above on the exact 18.8 patch. Current documentation lists `humans`, but does not state its introduction version; do not infer 18.8 support from the current page alone.

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

Before token creation, inspect the target instance's PAT creation page and, with an administrator where necessary, confirm:
- Whether an expiration date is required.
- The maximum permitted expiration date or lifetime.
- The expiration date selected for the eventual PAT.
- The local renewal owner and reminder process.

GitLab's current documentation says new access tokens default to an expiry date 365 days from creation when no date is supplied, unless the instance setting is changed; token expiry takes effect at midnight UTC on the expiry date. Administrator-configured maximum lifetime behavior depends on edition/settings. Validate the actual target instance rather than assuming current defaults apply unchanged to 18.8.

Do not place the PAT in chat, source control, a plain text config file, shell history, process arguments, or logs. Token creation and rotation must follow the accepted local secret-storage decision. Revoke a compromised token immediately; a revoked/expired PAT cannot be reactivated and must be replaced.

## 7. Sanitized result template

Use this template in a private worksheet first. Publish only fields marked safe.

| Check | Sanitized result |
|---|---|
| GitLab version/edition line | Safe to publish |
| Subscription tier | Free / other |
| `approved_by` available | PASS / FAIL |
| `GET /users/:id` available to regular user | PASS / FAIL |
| `bot` and `state` fields present | PASS / FAIL |
| `humans=true` accepted on target instance | PASS / FAIL |
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

- **DES-001** is DONE only after exact patch and CE/EE distribution are recorded.
- **SEC-009** is DONE only after regular human, bot/internal, owner, author, inactive/locked, inaccessible, and stale-approval cases are verified and sanitized evidence is committed.
- **SEC-010** is DONE only after the implemented MCP allowlist passes the integration matrix.
- **SEC-011** is DONE only after actual per-project permissions and protected-branch behavior are recorded.
- **SEC-012** is DONE only after target expiry policy and secure token lifecycle are validated.
- Defining this runbook does not complete any target-instance validation task and does not authorize implementation.
