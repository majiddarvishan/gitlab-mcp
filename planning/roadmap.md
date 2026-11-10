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
- [ ] Finalize detailed security and approval model.
- [x] Review official ChatGPT Desktop and Codex local MCP compatibility; see [client compatibility](../docs/architecture/client-compatibility.md).
- [ ] Run local MCP runtime smoke test on the target workstation (TST-009).

## Phase 1 — Technical design
- [ ] Select and document MCP SDK.
- [ ] Define tool naming, input schemas, and response contracts.
- [ ] Define local project allowlist format and administration workflow.
- [ ] Define configuration and credential storage.
- [ ] Define approval lifecycle, merge preflight checks, and failure behavior.
- [ ] Confirm exact GitLab 18.x minor/patch, edition, and enabled features.
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
