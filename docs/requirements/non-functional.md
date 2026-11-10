# Non-Functional Requirements

## Security
- Use least-privilege GitLab credentials.
- Keep credentials outside the repository and ordinary logs.
- Require explicit per-operation approval for every mutation.
- Validate all identifiers and user-supplied parameters.
- Do not expose a listening network service in the default local deployment.
- Do not log access tokens, private keys, or sensitive file contents.

## Reliability
- Use bounded HTTP timeouts.
- Handle GitLab pagination correctly.
- Respect GitLab rate limits and return clear retry guidance.
- Avoid duplicate mutations when a request times out after the server may have accepted it.
- Make operations deterministic and report partial results clearly.

## Performance
- Avoid fetching entire repositories when a tree or targeted file request is sufficient.
- Bound response sizes and support pagination.
- Make large diffs and files explicit in the tool response.

## Maintainability
- Keep GitLab API integration separate from MCP protocol handling.
- Keep approval enforcement separate from model-facing tool descriptions.
- Test permission boundaries and failure cases.
- Document supported GitLab versions and API limitations.

## Privacy and data handling
- Warn that content read from GitLab and returned to ChatGPT may be sent to the model provider for processing.
- Avoid retaining repository content locally unless a documented feature requires it.
- Do not send private content to third-party services other than the configured GitLab and AI host.
