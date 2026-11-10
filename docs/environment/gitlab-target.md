# GitLab Target Environment Profile

- **Profile status:** Confirmed (exact build supplied by user)
- **Last updated:** 2026-11-10
- **Validation procedure:** [Target-instance validation runbook](validation-runbook.md)
- **Offering:** GitLab Self-Managed
- **Distribution / edition:** Enterprise Edition (EE), user-confirmed
- **Subscription tier:** Free, user-confirmed (EE distribution does not imply a paid license)
- **Version line:** 18.8
- **Exact patch:** 18.8.4-ee, user-confirmed
- **Version tag:** [v18.8.4-ee](https://gitlab.com/gitlab-org/gitlab/-/tags/v18.8.4-ee)
- **Internal base URL / hostname:** Intentionally not recorded in this public repository
- **Credentials / tokens:** Never record here

## Purpose

The MCP design targets GitLab Self-Managed Enterprise Edition v18.8.4-ee running with the Free subscription tier. The exact patch and distribution were supplied by the user. The subscription tier and the CE/EE distribution are separate properties: EE is the deployed distribution, while Free is the license tier.

## Confirmed environment facts

| Field | Value | Status |
|---|---|---|
| Offering | GitLab Self-Managed | Confirmed |
| Subscription tier | Free | Confirmed by user |
| Major version | 18 | Confirmed |
| Minor version | 8 | Confirmed by user |
| Patch version | 18.8.4 | Confirmed by user |
| Distribution | Enterprise Edition (EE) | Confirmed by user |
| Version tag | v18.8.4-ee | Confirmed by user |
| Internal network reachability | Workstation can reach GitLab directly | Confirmed |
| Relevant feature limitations | Free-tier limitations documented below; instance validation pending | Partially confirmed |

## Version evidence

The user supplied the version line `GitLab Enterprise Edition v18.8.4-ee` and its public GitLab release tag. This is sufficient to complete DES-001. No private instance URL, hostname, IP, project name, account identifier, or token is recorded.

The remaining API behavior checks must still be run against the actual instance, because instance settings, feature flags, and exact runtime configuration can affect behavior.

## Free-tier constraints relevant to this project

### Code search

- GitLab Basic Search is available in Free.
- Basic code search is limited to project-level search; it does not provide group/global code search.
- Basic search uses exact substring matching and has fewer capabilities than advanced or exact code search.
- Advanced Search and Exact Code Search are Premium/Ultimate capabilities.
- The project-scoped Search API exposes a `blobs` scope; the exact request behavior and availability must be verified against this 18.8 Free instance before the server relies on it.
- Do not design the first release around global/group code search, Zoekt, or semantic code search.

### Merge request approvals

- GitLab Free allows eligible users with Developer role or higher to approve merge requests.
- On Free, these approvals are optional; they do not prevent merging when no approval exists.
- Required approval rules and richer approval-policy configuration require Premium or Ultimate.
- The MR approval-state API is available on Free. Because this target is EE, GitLab documents that the approval-state endpoint returns the full EE response schema even without a paid license. That includes attributes such as `approved_by`, `approvals_required`, `approvals_left`, approval-rule metadata, `state`, and `merge_status`; the exact response must still be captured on v18.8.4-ee.
- On EE, `approved=true` means the configured approval rules are satisfied. When no approval rules apply, GitLab can return `approved=true` even when `approved_by` is empty. Therefore `approved` is not evidence that a human reviewer exists; the MCP gate must inspect and classify `approved_by`.
- The separate `GET .../approval_state` endpoint that retrieves approval details/rules remains Premium/Ultimate. The EE response schema for `GET .../approvals` does not make that separate endpoint available on the Free license.
- The REST Users API `humans=true` filter selects regular users that are neither bot nor internal users. It was added in GitLab 17.3 (Merge Request [!159200](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/159200)), before the v18.8.4-ee target. The regular-user `GET /users/:id` endpoint is available to signed-in non-admin users and documents `bot`, `state`, and `locked` fields. Runtime behavior and visibility must still be checked on the actual instance.
- If the product must require at least one human approval before MCP performs a merge, that must be an explicit MCP-side policy; it is not enforced by the Free tier itself.

## Version maintenance and security note

As of 2026-11-10, GitLab's published maintenance policy lists 19.5, 19.4, and 19.3 as the maintained release lines. Version 18.8 is not in the currently maintained list.

GitLab states that bug fixes are backported to the current stable release, while security fixes are generally backported to the previous two monthly releases in addition to the current stable release. Therefore, 18.8 should be treated as an outdated target from a maintenance perspective.

This does not prevent designing or testing MCP compatibility with 18.8. However, production use should be accompanied by an administrator-owned upgrade plan or an explicitly documented risk acceptance. Do not assume routine security fixes are still being backported to 18.8.

## Why version and tier are recorded separately

- The REST API uses version 4 as its stable major version, but new endpoints, fields, and behaviors are introduced over time.
- Minor/patch release differences can affect endpoint availability and bug behavior.
- Tier controls whether certain capabilities are licensed.
- CE/EE distribution can affect response fields even when the subscription tier is Free.
- Role, token scopes, feature flags, and instance settings can still restrict otherwise available endpoints.

## Related tasks

- [DES-001 — Identify the exact GitLab 18.x environment](../../planning/tasks.md#des-001--identify-the-exact-gitlab-18x-environment)
- [DES-009 — Validate Free-tier project code search](../../planning/tasks.md#des-009--validate-free-tier-project-code-search)
- [DES-010 — Review GitLab 18.8 maintenance and upgrade plan](../../planning/tasks.md#des-010--review-gitlab-188-maintenance-and-upgrade-plan)

## Official references

- [GitLab release and maintenance policy](https://docs.gitlab.com/policy/maintenance/)
- [GitLab Search documentation](https://docs.gitlab.com/user/search/)
- [GitLab Search API](https://docs.gitlab.com/api/search/)
- [Merge request approvals](https://docs.gitlab.com/user/project/merge_requests/approvals/)
- [Merge request approvals API](https://docs.gitlab.com/api/merge_request_approvals/)
- [Users API](https://docs.gitlab.com/api/users/)
- [GitLab !159200 — REST human-user filter added in milestone 17.3](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/159200)
- [Internal users](https://docs.gitlab.com/administration/internal_users/)
- [Personal access tokens](https://docs.gitlab.com/user/profile/personal_access_tokens/)
- [Exact GitLab v18.8.4-ee tag](https://gitlab.com/gitlab-org/gitlab/-/tags/v18.8.4-ee)
