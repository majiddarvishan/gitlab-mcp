# Decisions Log

Last updated: 2026-11-10

| ID | Decision | Status | Notes |
|---|---|---|---|
| D-001 | Target self-managed GitLab | Accepted | GitLab is internal-only and reachable from the workstation. |
| D-002 | Use a local MCP process | Accepted | Workstation already reaches GitLab. |
| D-003 | ChatGPT Desktop is primary; Codex is fallback | Accepted | Verify exact client support before implementation. |
| D-004 | Include read and write capabilities | Accepted | Both are in first-release scope. |
| D-005 | Require approval for every write | Accepted | Approval is bound to exact operation and parameters. |
| D-006 | Keep project repository public | Accepted | No internal details or secrets may be published. |
| D-007 | Use Go | Proposed | Awaiting explicit approval. |
| D-008 | Use STDIO | Proposed | Awaiting compatibility verification and approval. |
| D-009 | Use GitLab REST API | Proposed | Expected integration approach; formal approval pending. |
| D-010 | Restrict access to an explicit project allowlist | Accepted | No implicit access to all projects visible to the GitLab identity. |
| D-011 | Require working branch and MR for repository code changes | Accepted | No direct writes to the default or protected branch. |
| D-012 | Allow MR merge through MCP with separate approval | Accepted | Merge is a distinct high-impact operation; GitLab policies remain authoritative. |
| D-013 | Keep MR approval human-only | Accepted | MCP may read approval state but must not approve or unapprove. |
| D-014 | Keep CI/CD read-only in the first release | Accepted | Pipeline and job metadata/status only; no triggers, retries, or cancellation. |
| D-015 | Defer job logs and artifacts | Accepted | Excluded from the first release pending data-handling review. |
| D-016 | Target GitLab Self-Managed Free 18.8 | Accepted | REST API v4; exact patch and CE/EE distribution remain pending. |
| D-017 | Require human GitLab approval plus separate MCP approval before merge | Accepted | MCP requires one verified human approval recorded in GitLab and a separate operation-specific user confirmation. |
| D-018 | Use one Project Access Token per allowlisted project | Rejected | Not selected; the user explicitly chose a Personal Access Token tied to their human GitLab account. |
| D-019 | Use the user's Personal Access Token as the MCP execution identity | Accepted | GitLab actions are attributed to the user's human account; the token inherits that account's permissions and is not project-scoped. The MCP allowlist remains mandatory server-side enforcement. |

## Decision process
A proposal becomes Accepted only after explicit user agreement. Material changes should be recorded as a new ADR rather than silently changing an accepted decision.
