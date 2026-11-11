# Existing GitLab MCP Implementation Evaluation

- **Status:** Source-level review completed for the current snapshots; DES-012 remains OPEN pending target-instance compatibility and final architecture review
- **Review date:** 2026-11-12
- **Target:** GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier
- **Project decision:** No product selected or adopted; no fork authorized
- **Scope:** Public documentation and pinned upstream source review. No candidate was built, run against the target GitLab, or given a PAT.

## 1. Executive summary

GitLab MCP products exist, including an official GitLab server and independent open-source implementations. None can be adopted unchanged under this project's accepted security decisions.

The current findings favor **zereight/gitlab-mcp as the first fork-and-harden candidate**. Its built-in project allowlist, strict-scope mode, download-token project binding, and regression tests are closer to our required project boundary. The latest 2.x release also records fixes for past cross-project scope defects. Important gaps remain: model-generated confirmation, approval/unapproval tools, a Raw REST escape hatch that does not inspect project IDs in body/query parameters, no mandatory default-branch write guard, and no direct Secret Service integration.

**jmrplens/gitlab-mcp-server remains a serious alternative if Go is selected**, and has a more extensive documented assurance case. It also already implements MCP Elicitation and the newer multi-round-trip flow supported by Codex. Its signed request state is bound to the tool call and principal, but it can be replayed for the same call until expiry; the current prompt is also too generic to show the exact MR identity and merge parameters. The reviewed configuration has no fixed MCP project allowlist; per-action `project_id` parameters are not an authorization boundary. It permits a model-supplied `confirm: true` to bypass Elicitation, has a YOLO bypass setting, exposes MR approval mutations, and does not integrate Linux Secret Service.

The most important project rule is unchanged: an MCP tool argument such as `confirm=true` or `_confirmed=true` is not user approval. The server must receive and verify a separate approval that the model cannot invent, and that approval must be bound to the exact operation and consumed once.

## 2. Mandatory target constraints

Any adopted or forked candidate must preserve:

- GitLab Self-Managed EE v18.8.4-ee with the Free subscription tier. API compatibility and license-tier availability must be checked separately.
- One Personal Access Token owned by the user's regular human GitLab account, initially with `api` scope (ADR-0015).
- A server-side canonical project allowlist across every read/write route, including nested resources and cross-project parameters. The PAT itself is not project-scoped.
- Repository code changes only on a dedicated working branch followed by an MR. The MCP must reject writes to the default/protected branch even if GitLab branch protection is misconfigured.
- MCP must never approve, unapprove, or reset GitLab MR review approvals on behalf of a human.
- Before MCP merge: at least one independently verified regular human GitLab approver distinct from both PAT owner and MR author, plus a separate exact-operation, single-use MCP user approval enforced server-side (ADR-0014).
- First-release CI/CD is read-only pipeline/job metadata and status. No trigger/retry/cancel, logs/traces, artifacts, or secret variables.
- PAT retrieval directly from Linux Secret Service in the same interactive desktop D-Bus session. No plaintext file, process argument, environment variable, model-visible prompt, or log fallback.
- No upstream code import, fork, source modification, PAT creation, or deployment as part of DES-012.

## 3. Official GitLab MCP server

**Source:** [Official GitLab documentation](https://docs.gitlab.com/user/model_context_protocol/mcp_server/).

GitLab's published feature history says the MCP server:
- Was introduced as an experiment in GitLab 18.3.
- Became Beta in 18.6.
- Moved from Premium to Free in 19.2.
- Became generally available in 19.5.

**Assessment:** The project's target is 18.8.4-ee with a Free subscription, before the move to Free. The official server is therefore not a usable baseline on the exact target without a separate GitLab upgrade/tier decision. Its status on a future upgraded instance should be reevaluated then.

## 4. Candidate A — zereight/gitlab-mcp

### 4.1 Snapshot and distribution

- **Repository:** https://github.com/zereight/gitlab-mcp
- **Reviewed main commit:** `b525d1e8b2c156f4c2af1d56b0e5656da56bdcd9` (2026-11-11)
- **Latest published release observed:** [v2.3.25](https://github.com/zereight/gitlab-mcp/releases/tag/v2.3.25), published 2026-11-11
- **License/runtime:** MIT; TypeScript/Node.js; Node.js >=20
- **Default surface at the reviewed snapshot:** approximately 40 tools; 288 total catalog tools across toolsets.
- **Security support statement:** the latest 2.x release is supported; older releases are not.

Primary pinned sources:
- [README](https://github.com/zereight/gitlab-mcp/blob/b525d1e8b2c156f4c2af1d56b0e5656da56bdcd9/README.md)
- [Security policy](https://github.com/zereight/gitlab-mcp/blob/b525d1e8b2c156f4c2af1d56b0e5656da56bdcd9/SECURITY.md)
- [Project context and auth](https://github.com/zereight/gitlab-mcp/blob/b525d1e8b2c156f4c2af1d56b0e5656da56bdcd9/server/gitlab-request-context.ts)
- [Raw API handler](https://github.com/zereight/gitlab-mcp/blob/b525d1e8b2c156f4c2af1d56b0e5656da56bdcd9/tools/gitlab-api-request.ts)
- [Download proxy](https://github.com/zereight/gitlab-mcp/blob/b525d1e8b2c156f4c2af1d56b0e5656da56bdcd9/downloads/proxy.ts)
- [Cross-project allowlist regression tests](https://github.com/zereight/gitlab-mcp/blob/b525d1e8b2c156f4c2af1d56b0e5656da56bdcd9/test/allowed-project-target-scope.test.ts)

### 4.2 Project-scope controls verified in source

The current code contains a real process-level `GITLAB_ALLOWED_PROJECT_IDS` allowlist and a request-local effective scope:

- `getEffectiveProjectId` rejects a project value not present in the effective allowlist before the project-scoped API operation proceeds.
- One allowlisted project can be used as a locked default; multiple allowed projects require the caller to identify one.
- `ENABLE_STRICT_PROJECT_SCOPE=true` filters project listings/search results and rejects group-wide or instance-wide tools that cannot be constrained to the allowed projects.
- The public GraphQL execution tool is rejected when a project-scoped deployment is active. This is conservative, but means GraphQL features cannot be used in that configuration.
- Download URLs use short-lived encrypted download tokens carrying the effective project allowlist and resource identity; the download proxy checks the bound project again.
- The allowlist compares supplied project identifiers against configured values. An ID/path mismatch may fail closed even when both refer to the same project; canonicalization policy should be standardized before adoption.

The current test `allowed-project-target-scope.test.ts` checks that an allowlisted source project cannot create an issue link or MR targeting an unlisted project, and that work-item note/reaction operations cannot be redirected to an unrelated note ID. The changelog records the corresponding previous defect as GHSA-ffr3-5f57-gf93 and says the fix is included in v2.3.25.

This is meaningful evidence of project-boundary engineering, but not proof that every tool path has been audited against this project's full SEC-010 matrix.

### 4.3 Blocking security gaps

**Raw REST escape hatch.** `gitlab_api_request` is explicitly opt-in and must not be enabled. Its handler constrains request paths to an allowed `/projects/:id/...` route when a project allowlist is active, but it does not inspect project IDs in request body/query parameters (for example `target_project_id` or `from_project_id`). It also bypasses other per-tool safety checks. A path-only check is not sufficient for this project. The safe proposal is to remove this tool from the fork, not merely hide it.

**Model-generated write confirmation.** `GITLAB_TOOL_POLICY_APPROVE` requires the tool call to carry `_confirmed: true`. The source reads that value directly from tool arguments; the model creates those arguments. Therefore the setting is a model-controlled flag, not independent user approval. It does not satisfy ADR-0003/ADR-0014. MCP client approval behavior cannot replace the server-side approval gate.

**MR review mutation tools.** The merge-request toolset includes `approve_merge_request` and `unapprove_merge_request`. They must be denied by server enforcement or removed from the fork. Hiding them from `tools/list` alone is not enough. Prefer deleting the handlers/registry entries and adding tests that direct invocation fails before any GitLab request.

**Default-branch write policy.** Repository write tools accept branch values from the caller. The reviewed code does not establish the mandatory project rule that the server itself rejects default/protected-branch writes. GitLab branch protection is an additional control, not a replacement for the accepted MCP branch policy.

**Credential backend mismatch.** The token loader accepts a CLI token, environment variable, or token file. The reviewed source does not retrieve the PAT directly from Linux Secret Service. The CLI argument, environment, and plaintext-file paths must not be used under ADR-0016; a direct D-Bus provider and fail-closed behavior are required.

**Visibility versus denial.** `GITLAB_TOOL_POLICY_HIDDEN` is a visibility mechanism; it is not the security boundary for prohibited operations. The changelog records GHSA-3gfg-wh5g-cf74, where hidden tools could be called even when their toolsets were disabled, fixed in v2.3.25. For prohibited operations, remove the tool or enforce a server-side deny rule and test direct calls. Do not rely on hiding alone.

### 4.4 Maintenance and security history

The repository has active releases and documents that only the latest 2.x line receives security fixes. The v2.3.25 changelog includes fixes for multiple security advisories, including the cross-project `target_project_id` issue and hidden-tool exposure behavior. This history is not by itself a reason to reject the project; it means the fork must start from a supported, patched snapshot and retain upstream security updates.

The deployment must pin a release/commit rather than use `latest`. Before adopting a binary, verify that its source commit includes the fixes relied on by this assessment and verify release provenance/checksums.

### 4.5 Preliminary fit

**Best current starting point for reusing project-scope controls.** A fork would still need a real server-side approval system, protected/default branch guard, human-only MR review surface, and D-Bus credential provider. The API escape hatch should be removed. The project is TypeScript/Node.js, so choosing it would require revisiting Proposed ADR-0004 (Go).

## 5. Candidate B — jmrplens/gitlab-mcp-server

### 5.1 Snapshot and distribution

- **Repository:** https://github.com/jmrplens/gitlab-mcp-server
- **Reviewed main commit:** `257c58f0b376983d98d177ce10f45c6f011ef92b` (2026-11-11)
- **Latest published release observed:** [v3.4.0](https://github.com/jmrplens/gitlab-mcp-server/releases/tag/v3.4.0), published 2026-10-31; the reviewed main snapshot is newer than that release.
- **License/runtime:** MIT; Go; single-binary distribution.
- **Tool surface:** tier-aware catalog with a default dynamic find/execute interface and a much larger action catalog (over 1,100 actions across the published surfaces).
- **Governance note:** the project's own assurance case records a bus factor of one maintainer.

Primary pinned sources:
- [Configuration](https://github.com/jmrplens/gitlab-mcp-server/blob/257c58f0b376983d98d177ce10f45c6f011ef92b/internal/config/config.go)
- [Confirmation gate](https://github.com/jmrplens/gitlab-mcp-server/blob/257c58f0b376983d98d177ce10f45c6f011ef92b/internal/toolutil/confirm.go)
- [MR action registration](https://github.com/jmrplens/gitlab-mcp-server/blob/257c58f0b376983d98d177ce10f45c6f011ef92b/internal/tools/mergerequests/action_specs.go)
- [MR approval actions](https://github.com/jmrplens/gitlab-mcp-server/blob/257c58f0b376983d98d177ce10f45c6f011ef92b/internal/tools/mrapprovals/action_specs.go)
- [Assurance case](https://github.com/jmrplens/gitlab-mcp-server/blob/257c58f0b376983d98d177ce10f45c6f011ef92b/site/src/content/docs/operations/assurance-case.mdx)

### 5.2 Existing Elicitation implementation

The repository already implements a direct user-interaction mechanism through MCP Elicitation:

- Legacy MCP sessions use a synchronous server-initiated Elicitation request.
- Sessions using protocol `2026-07-28` or later use multi-round-trip elicitation (MRTR), returning `inputRequests` and a signed `RequestState` that is echoed on retry.
- The request state is HMAC-protected, bound to a digest of the tool name plus canonical arguments, bound to the caller principal where the server has one, and expires after 10 minutes.
- OpenAI's current Codex App Server documentation exposes `mcpServer/elicitation/request`; the ChatGPT desktop app, Codex CLI, and IDE extension share the same Codex MCP host/configuration. This makes Elicitation a plausible interaction layer for the planned local deployment. See [Elicitation feasibility](#7-elicitation-feasibility-with-chatgpt-desktop-and-codex).

This is a useful foundation, but it is not yet the project's approval system. The existing `confirm: true` and YOLO bypasses remain, RequestState is reusable for the same call until expiry rather than single-use, and the current destructive-action prompt identifies the tool/action but not the exact MR, HEAD SHA, or merge options.

### 5.3 Project-scope controls

The reviewed `Config` supports GitLab URL/token, tier, tool surface, read-only/safe modes, tool exclusion by name, and other server settings. The inspected configuration does **not** define a fixed project-ID allowlist or project-scope policy. `ExcludeTools` is a tool-name filter, not a project authorization boundary.

Project-scoped actions require a `project_id` parameter, but a parameter is only a resource selector. The server needs a central authorization layer that checks every requested project against a canonical allowlist, including target IDs, nested resources, GraphQL-backed operations, and any cross-project relation. No such fixed MCP project allowlist was identified in the reviewed configuration/source.

The catalog includes global and group-wide operations such as global MR listing; excluding a few global tools would still not restrict ordinary project-scoped actions to the allowlist.

### 5.4 Blocking security gaps

**Model-controlled confirmation.** `ConfirmDestructiveAction` first checks `GITLAB_MCP_YOLO_MODE` / `AUTOPILOT`, then accepts `confirm: true` from the tool arguments, and only otherwise attempts MCP Elicitation. Elicitation can present a real prompt when the client supports it, but the model-supplied boolean bypasses that prompt. The project's assurance case explicitly recognizes that the model can write the confirmation argument. Therefore the current behavior is not a mandatory independent user approval. A hardened fork must remove the boolean bypass, reject YOLO for protected writes, and make server-issued Elicitation a required gate rather than an optional path.

**RequestState is bound but not single-use.** `internal/elicitation/flow.go` and `internal/elicitation/state.go` bind an accepted response to the tool name, canonicalized arguments, principal, and a 10-minute expiry. The state is signed, but `decodeState` returns the accepted answer again whenever the same call presents the same valid state before expiry. The repository's `state_test.go` explicitly verifies that an accepted answer is still returned to the same call one minute later. No server-side consumed-nonce/one-time registry was identified. This fails the project's one-use approval requirement.

**The current confirmation prompt does not show the exact Merge.** The individual surface prompts with the tool name; the meta surface prompts with the tool and action names. Neither adds the canonical project ID, MR IID, current source HEAD SHA, target branch, or merge options. A hardened Merge confirmation must show these exact values, bind the user's response to them, and re-read the MR/approval state immediately before consuming the approval and sending the merge.

**The existing gate is destructive-only.** The shared confirmation wrapper is invoked for actions classified as destructive. Ordinary create/update actions do not all pass through this gate, whereas this project requires explicit approval for every state-changing operation. The fork must apply the approval check to all mutation routes, with additional human GitLab approval preconditions for Merge.

**MR approval mutations.** The action catalog registers `merge_request.approve` and `merge_request.unapprove`. The approval package also registers approval-reset behavior. The first two can perform review approval mutations with the calling user's identity, contrary to ADR-0011. These operations must be removed or denied by the server, not merely hidden.

**No PAT-to-Secret-Service integration.** Standard stdio configuration reads `GITLAB_TOKEN` from the process environment. The project deliberately does not accept a token command-line flag, which avoids one exposure path, but there is no direct Linux Secret Service backend in the reviewed code. A direct D-Bus provider would still need to be added.

**Default-branch policy.** The reviewed architecture relies heavily on GitLab permissions and action behavior. No project-specific server-side guard was identified that universally forbids writes to a configured default/protected branch. This needs a separate source audit and, if absent, a server-side policy implementation.

**Large surface.** The action catalog contains many domains beyond this project's first-release scope. A restrictive catalog and server-side deny-by-default policy would be required across dynamic, meta, and individual surfaces. Tool-name filtering alone is not enough if a route remains executable through another surface.

### 5.5 Useful Free-tier approval read path

The source distinguishes Premium/Ultimate per-MR approval-state/rules actions from the basic approval configuration endpoint. `merge_request.approval_config` is documented as available on all tiers and reads the MR approvals data; the Premium/Ultimate approval-state endpoints are marked as such. This gives a possible starting point for reading `approved_by` on the target EE Free tier, but it does not implement our reviewer identity protocol. The server would still need to resolve each approver using the target-instance Users API checks in SEC-009 and fail closed on any ambiguous/incomplete identity.

### 5.6 Preliminary fit

**Best language fit if Go is retained**, with a detailed assurance case, extensive automated tests, and a single-binary distribution. However, adding a complete project-authorization layer, a mandatory exact-operation approval system, human-only MR review restrictions, branch protection policy, and Secret Service provider is substantial security engineering. The project currently has no proven project allowlist, so it is not the nearer drop-in candidate.

## 6. Comparison against project requirements

| Requirement | GitLab official (18.8 Free) | Zereight | JMRPlens |
|---|---|---|---|
| Available on exact target Free tier | No; Free from 19.2 | Self-managed API model; 18.8 target test pending | Self-managed/Free catalog; 18.8 target test pending |
| Local STDIO / Codex documented | Not suitable on target tier | Yes | Yes |
| Fixed project allowlist | No target-tier baseline | Yes, plus strict-scope mode and tests | No fixed MCP project allowlist identified |
| Cross-project parameter checks | Not established | Several known cases have regression tests; full audit pending | No central allowlist to enforce |
| Raw REST escape hatch | Not assessed for target | Explicit opt-in; body/query project fields are not checked; must be removed | No fixed project boundary to protect |
| Generic GraphQL path under project scope | Not established | Blocked when project scope is enabled | Must be covered by new central project authorization |
| Independent user-interaction channel | Not established | No Elicitation gate identified; `_confirmed` is model input | Direct MCP Elicitation exists and Codex-host support is documented; current bypass, generic prompt, replayable state and destructive-only coverage prevent compliance |
| Human-only MR approval | Not established | Approve/unapprove tools exist; must be removed/denied | Approve/unapprove/reset actions exist; must be removed/denied |
| Direct Secret Service | Not established | No; env/CLI/file token inputs | No; env token input |
| Go ecosystem | N/A | No; TypeScript/Node.js | Yes |
| Main adoption blocker | Target Free availability | Approval gate, raw API, branch gate, Secret Service | Project allowlist, approval gate, review controls, Secret Service |

## 7. Elicitation feasibility with ChatGPT Desktop and Codex

OpenAI's current Codex App Server documentation lists `mcpServer/elicitation/request` as a server request. The request carries a form or URL elicitation, and the client responds with `accept`, `decline`, or `cancel`. OpenAI's MCP connection guide states that ChatGPT desktop, Codex CLI, and the IDE extension share the same Codex MCP host and configuration. This makes MCP Elicitation a viable user-interaction candidate for our planned local STDIO setup.

Sources:
- [Codex App Server — MCP server elicitation requests](https://learn.chatgpt.com/docs/app-server)
- [ChatGPT Learn — supported MCP hosts and shared desktop/Codex configuration](https://learn.chatgpt.com/docs/extend/mcp)
- [OpenAI developer guidance — annotations do not replace server authorization or confirmation](https://developers.openai.com/plugins/build/mcp-server)

Design implications:
1. Treat Elicitation as the user-facing interaction channel, not as the authorization store. The server must fail closed if the client does not advertise support, does not return a valid accepted response, or cannot complete the request. Never fall back to `confirm=true`, `_confirmed=true`, YOLO, or a model instruction to retry.
2. Require approval on **every mutation**, not just destructive operations. Merge additionally requires the independent GitLab human approval from ADR-0014.
3. The Merge confirmation must display and bind the exact canonical project ID, MR IID, current source HEAD SHA, target branch, and all merge options that affect the result.
4. The approval must be atomically consumed once on the server. The JMRPlens MRTR RequestState is HMAC-signed, call/principal-bound, and expires in 10 minutes, but the current state remains reusable for the same call until expiry. A separate consumed-nonce or pending-approval registry is needed.
5. Immediately before Merge, re-read the MR, verify that its HEAD SHA still matches the approved SHA, and re-check the independent human GitLab approval. Any change invalidates the approval and requires a new Elicitation.
6. Actual behavior on the user's installed ChatGPT Desktop and Codex versions remains a TST-009 workstation test. Published protocol support is not a substitute for testing the exact local host UX and fail-closed behavior.

**Effect on candidate comparison:** JMRPlens has a meaningful advantage in existing Elicitation plumbing; Zereight has the stronger existing project-scope baseline. Neither currently implements the complete approval policy required by this project.

## 8. Current recommendation

Do not adopt either implementation unchanged.

Keep **zereight/gitlab-mcp as the first fork-and-harden candidate** because it already has an explicit allowlist, strict scope mode, download-scope binding, and project-boundary regression tests. Start from a supported patched snapshot (at least v2.3.25 for the reviewed security fixes) and pin an exact source commit. Disable/remove the Raw API tool, remove or deny MR approval mutations, implement the independent approval gate and default-branch guard, and integrate Secret Service. This path requires reconsidering the Proposed Go language ADR.

Keep **jmrplens/gitlab-mcp-server as the Go alternative**. It has a useful Elicitation implementation that may reduce the amount of client-interaction plumbing, but the current flow is not one-use and does not show an exact Merge summary. A canonical project authorization layer remains a prerequisite, and the confirmation bypass, destructive-only coverage, and approval mutation surfaces must also be changed.

A narrow wrapper that merely filters tool names is not sufficient for either candidate. If a policy boundary can be bypassed through a raw REST/GraphQL route, another tool surface, a nested resource, or a target-project parameter, the wrapper is not a security boundary.

This is a source-review priority only, not a product selection or ADR. The final recommendation remains conditional on a complete route-by-route authorization audit and target GitLab 18.8.4-ee validation.

## 9. Remaining DES-012 work

1. Complete route inventory for Zereight: all project-scoped reads/writes, nested IDs, cross-project parameters, aliases, branch writes, redirect/download routes, and every alternate tool/CLI surface. Verify denied tools cannot be called directly.
2. Complete route inventory for JMRPlens if retained as a Go challenger; identify every action route that requires project authorization and every global/group/GraphQL surface.
3. Define the exact hardened design for a model-independent, server-enforced approval flow. It must bind a one-time approval to canonical project ID, MR IID, current source HEAD SHA, merge options, and operation; it must be unusable for any other call or after state changes.
4. Define a server-side branch policy that rejects writes to default/protected branches regardless of GitLab configuration.
5. Design direct Secret Service D-Bus token retrieval for the selected candidate; do not use an environment, argument, plaintext-file, or chat fallback.
6. Test the exact required API operations against GitLab Self-Managed EE v18.8.4-ee with the Free subscription. This remains blocked until SEC-002 is accepted, the non-secret canary passes, a real PAT is securely provisioned, and the tests are explicitly authorized.
7. Review the upstream release contents, dependency graph, security advisories, signed artifacts/provenance, maintenance policy, and update strategy for the exact fork point.

DES-012 remains OPEN until the outstanding technical checks are completed and the user accepts a final adopt/fork/build recommendation.

No upstream repository is to be cloned into this project, no dependency is to be added, no source code is to be modified, and no PAT is to be created as part of DES-012.
