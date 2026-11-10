# ADR-0013: GitLab Self-Managed 18.x Compatibility Target

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User

## Context
The target company instance is GitLab Self-Managed 18.x. The MCP server must be designed and validated against this deployment rather than assuming GitLab.com behavior or the latest GitLab release.

## Options
1. Target the current latest GitLab release only.
2. Target the company's GitLab Self-Managed 18.x major release line.
3. Support a broad range of older and newer GitLab releases from the beginning.

## Decision
The compatibility target is GitLab Self-Managed Free 18.8 and the GitLab REST API v4. The exact patch number and CE/EE distribution remain pending confirmation.

The exact minor/patch version and edition of the company instance have not yet been supplied. They must be recorded during environment validation before endpoint-level implementation is finalized.

The server must:
- Avoid relying on endpoints or response fields introduced after the target 18.x version.
- Check edition, feature availability, permissions, and instance settings where these affect behavior.
- Report unavailable or unsupported capabilities explicitly.
- Include integration tests against the actual target 18.x instance before release.

## Consequences
- API design must be checked against version-appropriate GitLab documentation and the company's actual instance.
- Features whose behavior differs by edition or configuration must be capability-detected or documented as conditional.
- The compatibility target does not imply that every candidate capability is available on every 18.x edition.

## References
- [GitLab REST API](https://docs.gitlab.com/api/rest/)
- [GitLab Projects API](https://docs.gitlab.com/api/projects/)
- [GitLab Merge Requests API](https://docs.gitlab.com/api/merge_requests/)
- [GitLab Merge Request Approvals API](https://docs.gitlab.com/api/merge_request_approvals/)
- [GitLab Pipelines API](https://docs.gitlab.com/api/pipelines/)


## Maintenance status

As of 2026-11-10, GitLab's published maintained release lines are 19.5, 19.4, and 19.3. Version 18.8 is outside the currently maintained list. Compatibility work may target 18.8, but production deployment should be accompanied by an administrator-owned upgrade plan or explicit risk acceptance.

## Additional compatibility considerations

- GitLab Free does not include advanced/exact code search or required MR approval rules.
- Basic project-level code search is available in Free, but the exact project-scoped API behavior must be validated on the installed 18.8 instance.
- MR approval state is readable on Free, but fields differ between Community Edition and Enterprise Edition.
