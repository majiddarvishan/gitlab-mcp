# GitLab Target Environment Profile

- **Profile status:** Partially confirmed; patch and distribution details pending
- **Last updated:** 2026-11-10
- **Offering:** GitLab Self-Managed
- **Subscription tier:** Free (user-confirmed)
- **Version line:** 18.8 (user-confirmed)
- **Exact patch:** Not yet provided (for example, 18.8.0 or 18.8.6)
- **Distribution / edition:** Community Edition (CE) or Enterprise Edition (EE) not yet confirmed
- **Internal base URL / hostname:** Intentionally not recorded in this public repository
- **Credentials / tokens:** Never record here

## Purpose

The MCP design targets the company's GitLab Self-Managed Free tier, version 18.8. The exact patch and CE/EE distribution are still needed because the deployed build can affect security fixes, endpoint behavior, and response fields. The subscription tier and the CE/EE distribution are related but are not interchangeable labels.

## Confirmed environment facts

| Field | Value | Status |
|---|---|---|
| Offering | GitLab Self-Managed | Confirmed |
| Subscription tier | Free | Confirmed by user |
| Major version | 18 | Confirmed |
| Minor version | 8 | Confirmed by user |
| Patch version | Not yet supplied | Pending |
| Distribution | CE or EE not yet supplied | Pending |
| Internal network reachability | Workstation can reach GitLab directly | Confirmed |
| Relevant feature limitations | Free-tier limitations documented below; instance validation pending | Partially confirmed |

## How to identify the exact patch and distribution safely

1. Sign in to the company GitLab using the normal browser.
2. In the left sidebar, go to **Help > Help**.
3. Read the version/distribution string displayed at the top of the Help page.
4. Send only the text of that version/distribution line, for example `GitLab Enterprise Edition 18.8.6` or `GitLab Community Edition 18.8.6`.
5. Do not send or publish the private URL, hostname, IP, project names, account identifiers, or any token.

If the page shows only `18.8`, the administrator can also provide the exact package version from the server's package information. No shell output containing hostnames, IPs, or configuration secrets is needed.

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
- The MR approval-state API is available on Free, but response attributes depend on CE/EE distribution. On Community Edition, the documented response is limited to `approved`, `approved_by`, `user_can_approve`, and `user_has_approved`.
- The richer `approval_state` endpoint and approval-rule details are Premium/Ultimate. Do not assume they are available on this target.
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
