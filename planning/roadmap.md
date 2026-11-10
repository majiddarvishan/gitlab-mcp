# Roadmap

## Phase 0 — Design baseline (current)
- [x] Confirm target: self-managed GitLab reachable from workstation.
- [x] Prefer local MCP architecture; no public GitLab exposure.
- [x] Prioritize ChatGPT Desktop; allow Codex fallback.
- [x] Require explicit approval for every write operation.
- [x] Keep this project repository public with strict disclosure hygiene.
- [ ] Review and accept implementation language.
- [ ] Review and accept MCP transport.
- [ ] Finalize GitLab capability matrix.
- [ ] Finalize security and approval model.
- [ ] Verify current ChatGPT Desktop and Codex client compatibility.

## Phase 1 — Technical design
- [ ] Select and document MCP SDK.
- [ ] Define tool naming, input schemas, and response contracts.
- [ ] Define configuration and credential storage.
- [ ] Define approval lifecycle and failure behavior.
- [ ] Define GitLab version support and API compatibility.

## Phase 2 — Implementation (not authorized yet)
- [ ] Implement local MCP host integration.
- [ ] Implement GitLab API client.
- [ ] Implement read capabilities.
- [ ] Implement approval-gated write capabilities.
- [ ] Add unit, integration, and security tests.

## Phase 3 — Validation
- [ ] Validate with a test GitLab project.
- [ ] Test permission boundaries and approval bypass attempts.
- [ ] Test network failures, rate limits, and large responses.
- [ ] Document installation, configuration, and troubleshooting.

Implementation must not begin until the user explicitly approves the design baseline.
