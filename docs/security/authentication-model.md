# GitLab Authentication and Identity Model

- **Document status:** Selected model documented; validation pending
- **Target:** GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier
- **Last reviewed:** 2026-11-10
- **Related task:** SEC-001
- **Related ADR:** [ADR-0015 (Accepted)](../decisions/0015-gitlab-token-and-execution-identity.md)

## Purpose

This document defines the authentication and execution-identity model for the local MCP process using GitLab REST API v4. The selected model is a Personal Access Token (PAT) belonging to the user's existing human GitLab account.

The design must support:
- Reading project and repository content.
- Creating working branches, commits, and merge requests.
- Reading merge request approval state.
- Merging only after a verified human GitLab approval and a separate MCP operation approval.
- Reading pipeline and job metadata/status only.
- Operating only on projects in the MCP's explicit local allowlist.

## Selected model

| Attribute | Selected value |
|---|---|
| Token type | Personal Access Token (PAT) |
| Token owner | User's existing human GitLab account |
| GitLab execution identity | The PAT owner's human account |
| Token scope | api only for the initial read/write REST API workflow |
| Effective resource reach | All projects/groups the owner can access, subject to GitLab authorization and token scope |
| Project isolation | Enforced by the MCP canonical project allowlist; not provided by PAT scope |
| GitLab role | Inherited from the owner's actual membership/role in each project; no separate role can be assigned to the PAT |
| Credential storage | OS-native secret store where practical |
| Audit attribution | GitLab records the PAT owner's human account as the actor |

This is an Accepted token/identity choice. It does not authorize implementation. Account-specific permissions, allowlist behavior, secret storage, and merge preconditions still require validation.

## Important GitLab v18.8.4-ee constraints

- Fine-grained personal access tokens were introduced as Beta in GitLab 18.10. They are not available in the 18.8 target.
- Service accounts on GitLab Free were introduced in 18.10 behind a feature flag and became generally available in 18.11. Do not design the 18.8 Free setup around Free-tier service accounts.
- Project access tokens are available on GitLab Self-Managed Free, but they were not selected.
- A PAT uses the permissions of its owning user; it is not restricted to a single project by its token type.
- The `api` scope grants complete read/write access across the REST API, including all groups/projects available to the user and related registry/package/dependency-proxy API surfaces. It is deliberately broader than the MCP tool inventory.
- GitLab v18.8.4-ee does not provide fine-grained PAT permissions to restrict the token to only the API endpoints used by MCP.

## Credential type comparison

| Credential type | Effective resource scope | Identity/audit attribution | 18.8 Free fit | Decision |
|---|---|---|---|---|
| Personal access token (PAT) | All resources the owning user can access, subject to scopes and GitLab policy | Human user who owns the PAT | Available | **Selected** |
| Project access token | Associated project, with an internal-visibility caveat to validate | Project bot/internal user | Available on Self-Managed Free | Not selected |
| Group access token | Subgroups and projects in the group | Group bot/internal user | Available | Not selected |
| OAuth 2.0 access token | User's accessible resources, within granted scopes | Authorizing user | Available | Not selected |
| GitLab Free service account | Service-account user and memberships | Service account | Not available as a standard Free feature in 18.8 | Not suitable for this target baseline |

## Scope analysis

GitLab access-token scopes relevant to this project:

| Scope | Meaning | Suitability |
|---|---|---|
| api | Complete read/write API access across the API within the owning user's permissions, including accessible groups/projects and registry/package/dependency-proxy surfaces | Selected for the combined REST API read/write workflow; broad credential blast radius |
| read_api | Read-only API access | Insufficient for first-release write operations |
| read_repository | Repository read access through supported repository access methods | Not enough for MR/issue/commit/merge API writes |
| write_repository | Repository pull/push through Git-over-HTTP | Does not authenticate API requests; not a substitute for api |

Use `api` only for the first-release REST workflow; do not add unrelated scopes. The version-tagged v18.8.4-ee documentation explicitly describes this scope as complete API read/write access, including all groups and projects within the user's access. The scope does not grant permissions the user does not already have, but it can expose the full API surface allowed by that user's GitLab memberships and role. The MCP server must expose only the approved tool surface and enforce its allowlist and approval rules.

## Effective permissions and least privilege

A PAT does not have an independently selectable Developer or Maintainer role. It inherits the token owner's effective GitLab role and permissions on each project.

Before creating the PAT:
- Review the user's project and group memberships, especially Owner, Maintainer, and administrator-level access.
- Prefer the least-privileged practical account and memberships for the intended workflow.
- Do not elevate the account or alter protected-branch rules only to make MCP writes or merges possible.
- Validate, per test project, which operations the actual role permits: repository reads, working-branch creation, commits, MR creation, approval-state reads, and merge.
- If the account lacks permission for an operation, MCP must return a permission failure and must not attempt privilege escalation or a policy workaround.

A protected-branch merge may be rejected by GitLab if the user's effective role or branch rules do not permit it. That rejection is authoritative. A separate MCP approval does not create GitLab permissions.

## Project allowlist is mandatory defense in depth

The PAT itself is not project-scoped. A project allowlist therefore remains a mandatory server-side authorization boundary on every call.

The MCP server must:
- Resolve project paths and aliases to canonical GitLab project IDs.
- Check the canonical ID against the explicit local allowlist before reading project data or issuing project-scoped API requests.
- Repeat the check for nested resources and every cross-project reference.
- Reject unknown, ambiguous, removed, or unlisted projects.
- Never infer authorization from the PAT owner's visibility, membership, or group membership.
- Prevent tool arguments, model-generated content, or repository content from changing or bypassing the allowlist.

The allowlist limits what the MCP application will do; it does not make a leaked PAT safe. A person or process that obtains the PAT can call GitLab directly and may reach all resources available to its owner. The blast radius of credential compromise must be assessed against the user's full GitLab access, not merely the MCP allowlist.

## Local credential handling

- Store the PAT in an OS-native secret store where practical.
- Keep the token out of source control, ordinary configuration files, command-line arguments, environment dumps, MCP tool arguments, model-visible output, crash reports, and logs.
- Only the local MCP process should retrieve the PAT and attach it to GitLab requests.
- Never provide the token to the AI client or model as a tool input or tool result.
- Use an expiration date and document renewal, rotation, and immediate revocation.
- Keep a sanitized credential record containing only token type, owner alias, scope, expiry, and status; never record the token value.
- Redact authentication headers and token-like values from all diagnostics.

## Human approval identity separation

The target-instance verification procedure and negative test matrix are maintained in [SEC-009 reviewer verification](reviewer-verification.md).

ADR-0014 remains in force: merge requires a verified human GitLab approval plus a separate, exact-operation MCP user approval. MCP may read approval state but must not approve, unapprove, or reset an MR approval.

The selected PAT belongs to a human user, so GitLab may expose the execution identity as a regular human account. A GitLab approval record identifies the approving account but does not prove whether the approval was entered through the GitLab UI or by an API client. The approval of the PAT owner must therefore not automatically be treated as independent review.

**Accepted reviewer-independence rule:** the qualifying reviewer must be a regular human account distinct from both the PAT owner (the MCP execution identity) and the MR author. The user explicitly approved this condition. SEC-009 must still validate that the target GitLab Self-Managed EE v18.8.4 API exposes sufficient identity metadata. Until this validation is complete and the implementation can enforce the rule, MCP merge remains disabled. Bot/internal users and missing, stale, inaccessible, or ambiguous approval identities never satisfy the gate.

## Credential lifecycle

The v18.8.4-ee PAT documentation states that Free-tier PATs have a 365-day default and maximum lifetime. A 400-day maximum is available in GitLab 17.6 and later through the `buffered_token_expiration_limit` feature flag, disabled by default. PATs expire at midnight UTC on the expiry date. The actual flag state and creation UI must still be checked on the target instance; see [SEC-012](../environment/validation-runbook.md#6-validate-pat-expiry-and-rotation-policy-sec-012).

The design must define:
- Secure initial PAT provisioning without entering the token into model-visible chat.
- Expiration and renewal.
- Planned no-downtime replacement: create a second PAT, store and validate it, switch the MCP client, then revoke the old PAT. Do not use the GitLab Rotate UI for this sequence because it immediately invalidates the old token.
- Incident response: revoke a compromised PAT immediately, then provision a replacement; do not preserve a compromised token for overlap.
- Immediate revocation if the token is exposed or the user no longer wants MCP access.
- Behavior when the token is missing, revoked, expired, or loses required permissions.
- A sanitized credential inventory without token values.

Because this is one user PAT, rotation or revocation affects MCP access to all configured projects.

## Remaining validation

The target-instance test procedure is maintained in the [validation runbook](../environment/validation-runbook.md).

1. Which projects and groups can the PAT owner currently access, and what is the effective role on each intended project?
2. Does the explicit MCP allowlist deny a project that the PAT owner can access but has not allowlisted?
3. Which repository write operations are permitted by the user's actual roles?
4. Does the target EE v18.8.4 API return a qualifying regular human reviewer distinct from the PAT owner and MR author, using the documented `humans=true` filter?
5. What expiry policy and rotation interval are enforced by the target instance?

## Official references

- [REST API authentication](https://docs.gitlab.com/api/rest/authentication/)
- [Access token scopes](https://docs.gitlab.com/security/tokens/access_token_scopes/)
- [Personal access tokens](https://docs.gitlab.com/user/profile/personal_access_tokens/)
- [Project access tokens](https://docs.gitlab.com/user/project/settings/project_access_tokens/)
- [Service accounts](https://docs.gitlab.com/user/profile/service_accounts/)
- [Fine-grained personal access tokens](https://docs.gitlab.com/auth/tokens/fine_grained_access_tokens/)
- [v18.8.4-ee PAT documentation](https://gitlab.com/gitlab-org/gitlab/-/raw/v18.8.4-ee/doc/user/profile/personal_access_tokens.md)
- [Roles and permissions](https://docs.gitlab.com/user/permissions/)
- [Protected branch rules](https://docs.gitlab.com/user/project/repository/branches/protection_rules/)
