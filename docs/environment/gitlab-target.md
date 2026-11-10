# GitLab Target Environment Profile

- **Profile status:** Incomplete
- **Last updated:** 2026-11-10
- **Target offering:** GitLab Self-Managed
- **Target major version:** 18.x (user-confirmed)
- **Exact minor/patch:** Pending
- **Edition:** Pending
- **Internal base URL / hostname:** Intentionally not recorded in this public repository
- **Credentials / tokens:** Never record here

## Purpose

The MCP design targets the company's GitLab Self-Managed 18.x instance. The exact minor/patch version and edition are needed to validate REST API endpoint behavior and feature availability.

## How to identify the installed version safely

GitLab's official user documentation specifies this method for GitLab Self-Managed:

1. Sign in to the company GitLab using the normal browser.
2. In the left sidebar, go to **Help > Help** (at the bottom of the sidebar).
3. Read the version string displayed at the top of the Help page.
4. Record only the edition and version string, for example a generic form such as `GitLab Enterprise Edition 18.x.y` or `GitLab Community Edition 18.x.y`.
5. Do not copy the private URL, hostname, IP address, project names, or any token into this public project.

## Environment facts to record

| Field | Value | Status |
|---|---|---|
| Offering | GitLab Self-Managed | Confirmed |
| Major version | 18.x | Confirmed |
| Minor version | Not yet supplied | Pending |
| Patch version | Not yet supplied | Pending |
| Edition (Community / Enterprise) | Not yet supplied | Pending |
| Tier / license (if applicable and known) | Not yet supplied | Pending |
| Relevant feature limitations | To be checked after edition/version is known | Pending |

Do not infer the edition from the major version. If the Help page does not show enough detail, ask a GitLab administrator to provide the edition and exact version without disclosing internal host information.

## Why the exact version matters

- Self-Managed instances can lag current GitLab documentation.
- Some capabilities and response fields can vary by release, edition, license, role, or instance configuration.
- Merge approval and merge readiness data must not be assumed available merely because an endpoint exists in current documentation.
- The compatibility target remains GitLab 18.x; the exact minor/patch and edition refine the test matrix.

## Related task

- [DES-001 — Identify the exact GitLab 18.x environment](../../planning/tasks.md#des-001--identify-the-exact-gitlab-18x-environment)
