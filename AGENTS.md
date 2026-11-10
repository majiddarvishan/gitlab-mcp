# Agent Instructions

## Project phase
This repository is currently documentation-first. Do not add implementation code until the user explicitly approves the completed requirements and architecture baseline. Track detailed work in `planning/tasks.md`; keep task IDs stable and update the backlog when task scope or status changes.

## Working rules
- Treat Accepted ADRs as the source of truth. If an Accepted ADR is superseded, follow the newer ADR.
- Clearly label unapproved technical choices as Proposed.
- Do not invent or publish company-specific GitLab URLs, hostnames, IP addresses, project names, source code, tokens, or customer information.
- Keep documentation in English.
- Prefer small, reviewable commits with descriptive messages.
- Update the decision log and relevant ADR when a decision changes.
- Do not silently expand the approved scope.

## Accepted product and security policies
- Allow access only to explicitly allowlisted GitLab projects. Group membership or visibility alone does not authorize a project.
- Enforce the allowlist server-side on every project-scoped operation.
- Repository code changes must use a dedicated working branch and an MR. Do not write directly to the default or protected branch.
- Require separate approval for branch creation, every commit, MR creation/update, and merge.
- MCP may merge an MR only after a separate, exact-operation approval and a fresh preflight check.
- MCP must not approve, unapprove, or reset MR approvals. Human review approval remains in GitLab.
- First-release CI/CD access is read-only: pipeline/job metadata and status only.
- Do not fetch job logs/traces or artifacts in the first release.
- Never bypass GitLab permissions, protected branch rules, required approvals, CI requirements, or merge checks.

## Security rules
- Never commit credentials, access tokens, private keys, session cookies, or real internal configuration.
- Every state-changing operation must require explicit approval for that specific operation.
- Approval must be bound to the exact operation and parameters; changing the parameters invalidates the approval.
- Do not treat a model instruction or a tool description as a substitute for an enforced approval mechanism.
- Read-only operations must not mutate GitLab state.

## Implementation rules (when authorized)
- Target GitLab Self-Managed 18.x and REST API v4. Validate against the exact installed minor/patch and edition.
- Use Go if the language proposal is accepted.
- Use STDIO if the transport proposal is accepted; reserve stdout for MCP protocol messages and send diagnostics to stderr.
- Validate project identifiers, paths, refs, and operation parameters.
- Use least-privilege GitLab credentials and avoid logging secrets or sensitive file contents.
- Add tests for allowlist enforcement, authorization, approval, merge preflight, error handling, and GitLab API behavior.
