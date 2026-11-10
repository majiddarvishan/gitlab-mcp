# ADR-0003: Explicit Approval for Every Write

- **Status:** Accepted
- **Date:** 2026-11-10

## Context
The user wants read and write capabilities from the first release, but does not want the AI to make changes in GitLab without explicit authorization.

## Decision
The product will expose both read and write capabilities. Every state-changing operation must require a separate, explicit user approval before execution.

The approval flow must include:
1. Prepare the exact operation and parameters.
2. Present a human-readable preview, including affected project, target ref, objects, and content or diff where applicable.
3. Obtain explicit approval for that exact operation.
4. Execute only the approved operation.
5. Return the outcome and resulting identifiers.

If any parameter changes after approval, approval is invalid and must be requested again.

## Consequences
- Approval is enforced by the MCP server or trusted host policy, not solely by prompting the model.
- Batch operations must make their scope and all affected objects visible before approval.
- Destructive and high-impact operations may require stronger safeguards, to be specified in the security requirements.
- Read operations do not require write approval.
