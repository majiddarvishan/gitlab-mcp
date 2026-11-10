# Decisions Log

Last updated: 2026-11-10

| ID | Decision | Status | Notes |
|---|---|---|---|
| D-001 | Target self-managed GitLab | Accepted | GitLab is internal-only. |
| D-002 | Use a local MCP process | Accepted | Workstation already reaches GitLab. |
| D-003 | ChatGPT Desktop is primary; Codex is fallback | Accepted | Verify exact client support before implementation. |
| D-004 | Include read and write capabilities | Accepted | Both are in first-release scope. |
| D-005 | Require approval for every write | Accepted | Approval bound to exact operation and parameters. |
| D-006 | Keep project repository public | Accepted | No internal details or secrets may be published. |
| D-007 | Use Go | Proposed | Awaiting explicit approval. |
| D-008 | Use STDIO | Proposed | Awaiting compatibility verification and approval. |
| D-009 | Use GitLab REST API | Proposed | Expected integration approach; formal approval pending. |

## Decision process
A proposal becomes Accepted only after explicit user agreement. Material changes should be recorded as a new ADR rather than silently changing an accepted decision.
