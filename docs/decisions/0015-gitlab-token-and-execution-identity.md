# ADR-0015: GitLab Token Type and Execution Identity

- **Status:** Accepted
- **Date:** 2026-11-10
- **Decision owner:** User
- **Target:** GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier

## Context

The local MCP server uses GitLab REST API v4 for reads and approved mutations. It must access only explicitly allowlisted projects, create working branches/commits/MRs, inspect human MR approvals, and merge only after both a human GitLab approval and a separate MCP operation approval.

The user selected a Personal Access Token (PAT) belonging to their existing human GitLab account. MCP operations will therefore be attributed in GitLab to that human account, not to a project bot or service identity.

GitLab v18.8.4-ee does not provide fine-grained personal access token permissions. The selected `api` scope grants complete read/write API access across the owning user's accessible groups and projects, as well as additional API surfaces such as registries/packages. A PAT's effective resource reach follows the owning user's GitLab access and the token scopes; the token is not intrinsically restricted to one project.

## Decision

Use one Personal Access Token belonging to the user's chosen human GitLab account for MCP's GitLab REST API authentication.

The initial scope is api, because the first release includes both read and write REST API operations. Do not add unrelated scopes. read_api is insufficient for the approved write workflow, and write_repository does not authenticate REST API requests.

The token inherits the owning account's effective GitLab permissions. There is no separate per-token Developer/Maintainer role selection. The account's memberships and roles on each project, together with branch protections and instance policy, determine which operations GitLab accepts.

The MCP server must independently enforce the accepted canonical project allowlist on every project-scoped operation. The allowlist is an application authorization boundary; it does not narrow the PAT's underlying GitLab permissions.

## Operational and security constraints

- Store the PAT only in an OS-native secret store where practical. Never place it in source control, ordinary configuration, command-line arguments, MCP tool arguments, model-visible output, or logs.
- The local MCP process is the only component that reads the PAT and attaches it to GitLab API requests.
- Use the least-privileged practical human GitLab account. Review its memberships and project roles before provisioning the PAT; do not elevate roles or weaken branch protections merely to make MCP operations succeed.
- Treat the PAT as a credential that can reach every project and group the owning account can access, subject to token scopes and GitLab authorization. The impact of PAT disclosure is therefore broader than any one MCP allowlist entry.
- Keep the MCP allowlist mandatory for reads and writes, including nested resources and cross-project references. Deny unknown, ambiguous, and unlisted projects before returning their data or issuing mutations.
- Use an expiration date and document renewal. For planned replacement, create and validate a second PAT before revoking the old one; the GitLab Rotate UI invalidates the old token immediately. Revoke a compromised PAT immediately.
- GitLab activity will show the PAT owner's human account as the actor. Do not label or represent this activity as a bot identity.
- The PAT owner must not be treated as an independent reviewer merely because GitLab records that account as a human user.

## Merge approval interaction

ADR-0014 remains in force: MCP merge requires both a verified human GitLab approval and a separate, exact-operation MCP user approval. MCP must not expose approve, unapprove, or reset-approval operations.

A GitLab approval record identifies an account; it does not, by itself, prove whether the approval was entered through the GitLab UI or by an API client. The user explicitly accepted the following reviewer-independence rule:

- The qualifying reviewer must be a regular human account.
- The reviewer must be different from the PAT owner (the MCP execution identity).
- The reviewer must be different from the MR author.
- Bot/internal accounts and missing, stale, inaccessible, or ambiguous identities never satisfy the human-review gate.

SEC-009 must validate that the target GitLab EE v18.8.4 API exposes enough identity metadata to enforce this rule. Until that validation is complete and the implementation can fail closed, MCP merge remains unavailable.

## Alternatives considered

1. **Project Access Token per allowlisted project.** Not selected. It provides a narrower bot identity and credential boundary, but the user chose their personal human identity instead.
2. **A dedicated regular-user PAT.** Not selected. It would require a separate GitLab user and account governance; it is not the identity the user selected.
3. **Group Access Token.** Not selected. Its resource reach is broader than the intended per-project authorization model.
4. **OAuth 2.0 delegated token.** Not selected. It introduces OAuth application and token-refresh lifecycle complexity and does not match the selected PAT approach.

## Consequences

The version-tagged v18.8.4-ee PAT documentation is referenced in the [authentication model](../security/authentication-model.md). Its Free-tier expiry baseline is a 365-day maximum by default, with a 400-day option controlled by the `buffered_token_expiration_limit` feature flag.

### Benefits

- MCP actions are attributed to the user's existing GitLab identity.
- Only one PAT must be provisioned and rotated for the initial setup.
- No separate project bot identities need to be created or managed.

### Risks and trade-offs

- The PAT can reach all projects/groups available to its owner within its token scopes; it is not project-scoped.
- Compromise of the PAT or local MCP process may expose or mutate any resource the account can access, not only allowlisted projects.
- The MCP allowlist cannot protect GitLab from use of the PAT outside the MCP process; it only constrains MCP's own authorization path.
- GitLab audit records for MCP operations are attributed to the human PAT owner, so they do not distinguish MCP-originated activity from other activity by that user without additional MCP-side audit records.
- The `api` scope is broad: it includes all API resources available to the owner, including accessible groups/projects and registry/package-related API surfaces. GitLab v18.8.4-ee does not provide fine-grained PAT permissions to narrow it to specific endpoints.
- The user's existing project roles may be too weak for some operations (especially protected-branch merge) or broader than necessary. MCP must not elevate them automatically.
- The PAT is a single credential dependency; expiration or revocation interrupts MCP access to all configured projects.

## Required validation

- Confirm the PAT's effective permissions on the exact GitLab Self-Managed EE v18.8.4 instance.
- Verify that the MCP allowlist denies a project the PAT owner can access but has not explicitly allowlisted.
- Verify branch, commit, MR, approval-read, and merge permissions for the user's actual role on each test project.
- Verify the target API can identify a qualifying human reviewer distinct from the PAT owner and MR author using the SEC-009 procedure (`GET /users/:id` plus the exact-ID match in `GET /users?username=:username&humans=true&active=true`); otherwise keep MCP merge disabled.
- Verify token expiry, rotation, revocation, and secure local storage behavior.
- Confirm that diagnostics and MCP tool outputs never expose the PAT.

## References

- [GitLab REST API authentication](https://docs.gitlab.com/api/rest/authentication/)
- [GitLab access token scopes](https://docs.gitlab.com/security/tokens/access_token_scopes/)
- [GitLab personal access tokens](https://docs.gitlab.com/user/profile/personal_access_tokens/)
- [GitLab roles and permissions](https://docs.gitlab.com/user/permissions/)
- [GitLab protected branch rules](https://docs.gitlab.com/user/project/repository/branches/protection_rules/)
