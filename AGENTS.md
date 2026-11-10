# Agent Instructions

## Project phase
This repository is currently documentation-first. Do not add implementation code until the user explicitly approves the completed requirements and architecture baseline.

## Working rules
- Treat accepted ADRs as the source of truth. If a newer accepted ADR supersedes one, follow the newer decision.
- Clearly label unapproved technical choices as Proposed.
- Do not invent or publish company-specific GitLab URLs, hostnames, IP addresses, project names, source code, tokens, or customer information.
- Keep documentation in English.
- Prefer small, reviewable commits with descriptive messages.
- Update the decision log and relevant ADR when a decision changes.
- Do not silently expand the approved scope.

## Security rules
- Never commit credentials, access tokens, private keys, session cookies, or real internal configuration.
- Every state-changing GitLab operation must require explicit user approval for that specific operation.
- Approval must be bound to the exact operation and its parameters; changing the parameters invalidates the approval.
- Do not treat a model instruction or a tool description as a substitute for an enforced approval mechanism.
- Read-only operations must not mutate GitLab state.

## Implementation rules (when authorized)
- Use Go if the language proposal is accepted.
- Keep MCP protocol messages on STDIO; send diagnostics only to stderr.
- Validate project identifiers, paths, refs, and operation parameters.
- Use least-privilege GitLab credentials and avoid logging secrets or sensitive file contents.
- Add tests for authorization, approval, error handling, and GitLab API behavior.
