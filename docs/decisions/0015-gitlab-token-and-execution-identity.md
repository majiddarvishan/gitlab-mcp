# ADR-0015: GitLab Token Type and Execution Identity

- **Status:** Proposed
- **Date:** 2026-11-10
- **Decision owner:** Pending user decision
- **Target:** GitLab Self-Managed Free 18.8

## Context

The local MCP server needs to read and mutate GitLab through REST API v4. It must access only explicitly allowlisted projects, create working branches/commits/MRs, inspect human MR approvals, and merge only after both a human GitLab approval and a separate MCP operation approval.

GitLab 18.8 Free does not provide fine-grained personal access tokens or generally available Free service accounts. Project access tokens are available on GitLab Self-Managed Free.

## Options

1. **Personal access token of the user's everyday account.** Easy to provision, but inherits all projects and permissions of that account. Not recommended.
2. **Personal access token of a dedicated regular user.** Can be restricted by that user's memberships and roles, but a formal Free service account is not available as a standard 18.8 feature. It also requires lifecycle and account-governance handling.
3. **One project access token per allowlisted project.** Project-scoped bot identity and independent revocation. Requires one token per project and a role decision. Recommended candidate.
4. **Group access token.** Covers the group's projects/subgroups and is broader than the accepted per-project allowlist. Not recommended as the default.
5. **OAuth 2.0 delegated user token.** Supports user-delegated API access but adds OAuth application setup and refresh-token lifecycle. Keep as an alternative if a user-delegated execution identity is preferred.

## Proposal

Use one Project Access Token per allowlisted project. Use the `api` scope for the combined read/write REST API workflow. Store each token securely and map it to the canonical project ID.

Start with the Developer role for branch, commit, and MR work. Do not assume Developer can merge into a protected default branch. Validate the actual branch permissions. If MCP merge requires Maintainer, present the broader Maintainer project-bot permissions for explicit review; do not weaken protected branch rules to accommodate MCP.

The project bot identity must never be accepted as a human reviewer under ADR-0014.

## Consequences if accepted

- Each allowlisted project has a distinct credential and bot actor.
- Removing a project from the allowlist can revoke only that project's token.
- MCP-created changes are attributed to the project bot, not the human workstation user.
- The `api` scope grants broad API permissions within the token's scope; GitLab 18.8 does not provide fine-grained PAT scopes.
- More than one project requires more than one token.
- The target instance's internal-visibility behavior must be tested before relying on project token scope as the only credential-level boundary.

## Consequences if rejected

A PAT or OAuth token can be selected instead, but its resource reach, account identity, and blast radius must be documented and mitigated.

## Required validation

- Verify project token behavior on the exact GitLab Free 18.8 CE/EE build.
- Verify token access against one allowlisted project and one non-allowlisted project.
- Verify Developer vs Maintainer behavior for working branch creation, commits, MR creation, approval-state reads, and MR merge.
- Verify that the bot identity can be distinguished from a human reviewer.
- Verify token expiry, rotation, and revocation behavior.

## References

- [GitLab Project Access Tokens](https://docs.gitlab.com/user/project/settings/project_access_tokens/)
- [GitLab access token scopes](https://docs.gitlab.com/security/tokens/access_token_scopes/)
- [GitLab protected branch rules](https://docs.gitlab.com/user/project/repository/branches/protection_rules/)
