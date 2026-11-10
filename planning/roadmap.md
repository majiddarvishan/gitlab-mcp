# Roadmap

The detailed, ID-based task register is maintained in [planning/tasks.md](tasks.md). This roadmap is the high-level phase summary; task-level dependencies and acceptance criteria live in the backlog.

## Phase 0 — Design baseline (current)
- [x] Confirm target: self-managed GitLab reachable from workstation.
- [x] Prefer local MCP architecture; no public GitLab exposure.
- [x] Prioritize ChatGPT Desktop; allow Codex fallback.
- [x] Require explicit approval for every write operation.
- [x] Keep this project repository public with strict disclosure hygiene.
- [x] Restrict MCP access to an explicit project allowlist.
- [x] Require a working branch and merge request for repository code changes.
- [x] Allow MCP merge only with a separate, operation-specific approval.
- [x] Keep merge request approval human-only; MCP may read approval state.
- [x] Limit CI/CD to pipeline and job status/metadata in the first release.
- [x] Defer job logs and artifacts.
- [x] Set GitLab Self-Managed 18.x as the compatibility target.
- [ ] Review and accept implementation language.
- [ ] Review and accept MCP transport.
- [ ] Finalize exact capability inventory and tool contracts.
- [ ] Complete target-instance security validation: PAT permissions, allowlist isolation, and independent human reviewer identity.
- [x] Review official ChatGPT Desktop and Codex local MCP compatibility; see [client compatibility](../docs/architecture/client-compatibility.md).
- [ ] Run local MCP runtime smoke test on the target workstation (TST-009).

## Phase 1 — Technical design
- [ ] Select and document MCP SDK.
- [ ] Define tool naming, input schemas, and response contracts.
- [ ] Define local project allowlist format and administration workflow.
- [x] Select the user's Personal Access Token and human execution identity (SEC-001 / ADR-0015).
- [ ] Validate PAT reach against the MCP project allowlist (SEC-010).
- [ ] Validate the user's actual per-project permissions, including protected-branch merge (SEC-011).
- [ ] Validate PAT expiry and rotation behavior (SEC-012).
- [ ] Define configuration and credential storage.
- [ ] Define approval lifecycle, merge preflight checks, and failure behavior.
- [x] Confirm GitLab Free tier and 18.8 minor version.
- [x] Confirm exact GitLab build: Enterprise Edition v18.8.4-ee with Free subscription tier (DES-001).
- [ ] Review 18.8 maintenance status and document upgrade plan/risk acceptance (DES-010).
- [ ] Validate Free-tier project-level code search API (DES-009).
- [x] Require a verified human GitLab MR approval plus separate MCP merge approval (DES-011 / ADR-0014).
- [ ] Run [SEC-009 reviewer verification protocol](../docs/security/reviewer-verification.md) on the exact Free 18.8 CE/EE instance; reviewer must differ from PAT owner and MR author.
- [ ] Define API compatibility tests against the target GitLab 18.x instance.

## Phase 2 — Implementation (not authorized yet)
- [ ] Implement local MCP host integration.
- [ ] Implement GitLab API client.
- [ ] Implement allowlist enforcement across all project-scoped operations.
- [ ] Implement read capabilities.
- [ ] Implement working-branch code changes and approval-gated commits/MRs.
- [ ] Implement separately approved merge capability without bypassing GitLab policy.
- [ ] Implement read-only pipeline/job metadata.
- [ ] Add unit, integration, and security tests.

## Phase 3 — Validation
- [ ] Validate with an allowlisted test GitLab 18.x project.
- [ ] Test project allowlist bypass attempts.
- [ ] Test approval binding and replay prevention.
- [ ] Test merge preconditions, protected branches, and approval rules.
- [ ] Test network failures, rate limits, and large responses.
- [ ] Document installation, configuration, and troubleshooting.

Implementation must not begin until the user explicitly approves the design baseline.
