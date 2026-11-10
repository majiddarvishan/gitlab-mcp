# GitLab Authentication and Identity Model

- **Document status:** Proposed for review
- **Target:** GitLab Self-Managed Free 18.8
- **Last reviewed:** 2026-11-10
- **Related task:** SEC-001
- **Related ADR:** ADR-0015 (Proposed)

## Purpose

This document compares authentication choices for a local MCP process that uses GitLab REST API v4. The target instance is GitLab Self-Managed Free 18.8.

The selected credential must support:
- Reading project and repository content.
- Creating working branches, commits, and merge requests.
- Reading merge request approval state.
- Merging only after a verified human GitLab approval and a separate MCP operation approval.
- Reading pipeline and job metadata/status only.
- Operating only on projects in the MCP's explicit local allowlist.

## Important GitLab 18.8 constraints

- Fine-grained personal access tokens were introduced as Beta in GitLab 18.10. They are not available in the 18.8 target.
- Service accounts on GitLab Free were introduced in 18.10 behind a feature flag and became generally available in 18.11. Do not design the 18.8 Free setup around Free-tier service accounts.
- Project access tokens are available on GitLab Self-Managed with any license, including Free.
- Project access tokens require a Maintainer or Owner to create them in the project settings.
- The project access token's role cannot exceed the creating user's role.

## Credential type comparison

| Credential type | Effective resource scope | Identity/audit attribution | 18.8 Free fit | Assessment |
|---|---|---|---|---|
| Personal access token (PAT) | All projects and groups accessible to the token owner | Human user who owns the PAT | Available | Simple, but broad; use only if other options are impractical |
| Project access token | Associated project; verify internal-visibility edge cases on the target instance | Project bot/internal user | Available on Self-Managed Free | Recommended candidate for per-project isolation |
| Group access token | Subgroups and projects in the group | Group bot/internal user | Available | Too broad for an explicit per-project allowlist unless group scope exactly matches policy |
| OAuth 2.0 access token | User's accessible resources, within granted scopes | Authorizing user | Available | Supports delegated user identity but adds OAuth application, redirect, refresh, and lifecycle complexity |
| GitLab Free service account | Service-account user and memberships | Service account | Not available as a standard Free feature in 18.8 | Not an option for this target baseline |

## Project access token behavior

Project access tokens are supported on GitLab Self-Managed Free. A project access token is intended to access its associated project and is represented by a project bot identity.

Important setup facts:
- A Maintainer or Owner is required to create a project access token.
- The creator cannot grant the token a role higher than their own.
- The token role can be selected, subject to GitLab's available roles and the creator's authority.
- A project access token can be created with an expiration date and one or more scopes.
- The token value is shown only once; it must be saved securely.
- Project access tokens can be rotated or revoked from GitLab.

**Internal-visibility caveat:** GitLab's project access token documentation includes a note about tokens created by internal users accessing projects with Internal visibility. The exact applicability must be validated against the company's 18.8 CE/EE distribution and visibility settings. Do not assume a token's project scope alone is the only isolation control. The MCP project allowlist remains mandatory on every call, and SEC-010 must verify cross-project behavior using a non-allowlisted test project.

## Scope analysis

GitLab access-token scopes relevant to this project:

| Scope | Meaning | Suitability |
|---|---|---|
| `api` | Complete read/write API access within the token's resource scope | Required candidate for the combined REST API read/write workflow |
| `read_api` | Read-only API access within the token's resource scope | Insufficient for first-release write operations |
| `read_repository` | Repository read access through supported repository access methods | Not enough for MR/issue/commit/merge API writes |
| `write_repository` | Repository pull/push through Git-over-HTTP | Does not authenticate API requests; not a substitute for `api` |

The `api` scope is broad within the token's resource scope. GitLab 18.8 does not provide fine-grained PAT permissions to narrow it to only a list of REST endpoints. The token's role and resource scope still apply, but the server must separately restrict the exposed MCP operations.

## Role and merge implications

GitLab roles are cumulative. A Developer can push to non-protected branches and create merge requests. By default, only a Maintainer can merge into a protected branch. Protected branch rules can allow Developers to merge, but on Free those rules are role-based; granting Developer merge capability can affect all Developers in the project, not only the MCP project bot.

Therefore:
- A Developer-level project token is the lower-privilege starting point for branch/commit/MR work.
- Whether a Developer token can merge depends on the target branch protection rules.
- A Maintainer-level project token may be required to merge into a protected default branch under the default policy.
- Maintainer is a materially broader project role, so it must not be granted without explicit user/admin review.
- Do not weaken protected branch settings merely to make an MCP token work.
- The exact API permissions must be tested against the company's project and branch settings.

## Recommended direction (Proposed)

Use one project access token for each explicitly allowlisted project, rather than one broad PAT or a group token.

Proposed baseline:
- Token type: Project Access Token.
- Resource scope: one project per token.
- Scope: `api`, because first-release capabilities include REST API writes.
- Initial role candidate: Developer.
- Merge role: unresolved; validate whether the target protected branch permits Developer merge. If it does not, decide whether to grant a Maintainer-level project bot or leave MCP merge unavailable for that project. Do not alter project protection rules implicitly.
- Identity: the project bot is the actor for MCP-created branches, commits, MRs, and merges; human review approvals remain separate.
- Storage: OS-native secret store, keyed by canonical project ID. No token in source control, plain configuration, command line, or logs.

This is a recommendation, not an Accepted decision. The exact role and token type must be confirmed by the user after reviewing the trade-offs.

## Why not a single personal access token by default?

A PAT inherits all permissions of its owner and can access every project/group that the owner can access. This creates a broader credential blast radius. The application allowlist is still required, but it is not a substitute for limiting the credential itself.

A PAT can be an acceptable fallback if project access tokens are operationally unsuitable, but the owner account should then be a dedicated, minimally privileged identity—not a highly privileged everyday account.

## Why not a group access token by default?

A group access token covers the group's subgroups and projects. That can include projects not present in the MCP allowlist. Since this project explicitly chose per-project authorization, a group token is not the recommended default.

## Human approval identity separation

The MCP execution identity must never satisfy ADR-0014's human-review requirement.

- Project access token bot users must not count as human reviewers.
- The merge preflight must exclude the MCP execution identity and reject bot/internal accounts.
- A recorded human approval in GitLab and the user's separate MCP merge confirmation are both mandatory.
- If the installed GitLab API cannot distinguish a human reviewer reliably, MCP merge remains disabled.

## Credential lifecycle requirements

The final design must define:
- Secure initial token provisioning.
- Token expiration and renewal.
- Rotation without an uncontrolled downtime window.
- Immediate revocation when a project is removed from the allowlist.
- Behavior when a token is missing, revoked, expired, or loses role/scope.
- A sanitized credential inventory showing project alias/ID, token type, role, scopes, and expiry—but never token values.

## Open questions

1. Does the user have Maintainer or Owner rights on every project that may be allowlisted, or must a project admin provision each token?
2. Do the protected default branches currently allow only Maintainers to merge?
3. Should MCP merge be disabled for a project if its bot token can only have Developer role?
4. Can the target instance reliably identify project access token bots and regular human approvers in the approvals API/Users API?
5. Does the internal-visibility caveat affect this instance's project access tokens?

## Official references

- [REST API authentication](https://docs.gitlab.com/api/rest/authentication/)
- [Access token scopes](https://docs.gitlab.com/security/tokens/access_token_scopes/)
- [Personal access tokens](https://docs.gitlab.com/user/profile/personal_access_tokens/)
- [Project access tokens](https://docs.gitlab.com/user/project/settings/project_access_tokens/)
- [Project access tokens API](https://docs.gitlab.com/api/project_access_tokens/)
- [Service accounts](https://docs.gitlab.com/user/profile/service_accounts/)
- [Fine-grained personal access tokens](https://docs.gitlab.com/auth/tokens/fine_grained_access_tokens/)
- [Roles and permissions](https://docs.gitlab.com/user/permissions/)
- [Protected branch rules](https://docs.gitlab.com/user/project/repository/branches/protection_rules/)
