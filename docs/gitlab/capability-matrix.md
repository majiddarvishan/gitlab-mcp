# GitLab Capability Matrix

This is an initial inventory, not a promise that every operation will ship in the first release. Exact tools, API endpoints, permissions, approval risk, and priority remain to be reviewed.

| Domain | Capability | Access | Approval | Initial priority | Status |
|---|---|---|---|---|---|
| Projects | List accessible projects | Read | No | P0 | Candidate |
| Projects | Get project metadata | Read | No | P0 | Candidate |
| Repository | Get repository tree | Read | No | P0 | Candidate |
| Repository | Read file/raw content | Read | No | P0 | Candidate |
| Repository | Search code | Read | No | P0 | Candidate |
| Repository | List branches and tags | Read | No | P0 | Candidate |
| Repository | Create/delete branch | Write | Yes | P1 | Candidate |
| Repository | Create/update file | Write | Yes | P1 | Candidate |
| Repository | Commit multiple file changes | Write | Yes | P1 | Candidate |
| Commits | List and inspect commits | Read | No | P0 | Candidate |
| Commits | Compare refs | Read | No | P1 | Candidate |
| Merge requests | List/get MR | Read | No | P0 | Candidate |
| Merge requests | Read diff and discussions | Read | No | P0 | Candidate |
| Merge requests | Create/update MR | Write | Yes | P1 | Candidate |
| Merge requests | Add comment/discussion | Write | Yes | P1 | Candidate |
| Merge requests | Merge MR | Write | Yes; high impact | P2 | Review |
| Issues | List/get issues | Read | No | P1 | Candidate |
| Issues | Create/update/close issue | Write | Yes | P1 | Candidate |
| CI/CD | Read pipeline/job status | Read | No | P1 | Candidate |
| CI/CD | Retry/cancel jobs or pipelines | Write | Yes | P2 | Review |
| Project settings | Read settings | Read | No | P2 | Review |
| Project settings | Change settings/protection | Write | Yes; high impact | Out of initial scope | Excluded pending review |

## Priority
- **P0:** essential for useful repository understanding.
- **P1:** important workflow capabilities.
- **P2:** advanced or higher-risk capabilities.

## Review questions
- Should access be restricted to an explicit project/group allowlist?
- Should search use GitLab's native search API only, or may a local index be added later?
- Which GitLab versions must be supported?
- Which write operations should be excluded from the first release?
