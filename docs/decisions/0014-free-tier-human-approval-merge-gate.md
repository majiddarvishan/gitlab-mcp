# ADR-0014: Enforce a Human MR Approval Before MCP Merge on GitLab Free

- **Status:** Proposed
- **Date:** 2026-11-10
- **Decision owner:** Pending user decision

## Context

The target is GitLab Self-Managed Free 18.8. GitLab Free allows eligible users to approve merge requests, but approvals are optional and do not prevent a merge when no approval exists. Required approval rules are a Premium/Ultimate capability.

The project already accepts:
- Human-only MR approval: MCP must not approve or unapprove an MR.
- MCP merge capability with a separate, exact-operation user approval.

Those decisions do not specify whether the MCP must additionally require at least one human MR approval before it executes a merge.

## Options

1. **Require at least one recorded human approval in MCP before merge.** The server reads the Free-tier-compatible MR approval state and refuses to prepare/execute a merge if no eligible human approval is present.
2. **Do not require an MR approval.** The user's separate MCP approval is sufficient, subject to all GitLab merge checks and permissions.
3. **Make the requirement configurable per project.** This adds configuration complexity and requires a per-project policy model.

## Proposal

For a safe default on the Free tier, require at least one human approval recorded in GitLab before MCP is allowed to execute a merge.

The MCP server would:
- Read the MR approval state immediately before preparing the merge approval.
- Verify that at least one approver is a human reviewer and is not the MCP execution identity.
- Include the approver identity and current source HEAD SHA in the merge preview.
- Re-read approval state and the MR head immediately before execution.
- Reject merge if the approval is missing, has been removed, or no longer applies to the current MR version.
- Never create an approval itself.

## Important implementation caveat

The Free-tier approval API reports `approved_by`, but human-versus-bot classification and response details must be validated against the installed CE/EE distribution. If the API cannot reliably distinguish a human reviewer from a bot/service identity, the product must not silently treat any approval as human; a safer alternative or explicit user policy is required.

## Consequences if accepted

- GitLab Free's optional approval mechanism is supplemented by an application-level MCP gate.
- Merge operations may be refused even when GitLab itself would allow the merge.
- The human approval must be separate from the ChatGPT/MCP merge confirmation.
- A change to the MR head after approval requires renewed human review and a new MCP merge approval.

## Consequences if rejected

- MCP may merge without any recorded MR approval if the user separately approves the exact merge and GitLab's own merge checks permit it.
- The UI must clearly distinguish GitLab human review approval from the MCP authorization to execute a merge.

## References

- [GitLab merge request approvals](https://docs.gitlab.com/user/project/merge_requests/approvals/)
- [GitLab merge request approvals API](https://docs.gitlab.com/api/merge_request_approvals/)
