# Existing GitLab MCP Implementation Evaluation

- **Status:** Preliminary source review; DES-012 remains OPEN
- **Review date:** 2026-11-10
- **Target:** GitLab Self-Managed Enterprise Edition v18.8.4-ee, Free subscription tier
- **Project decision:** No product selected or adopted
- **Scope:** Public documentation and pinned upstream source snapshots; no code was built or executed

## 1. Executive summary

Existing GitLab MCP products do exist. GitLab ships an official server, and multiple independent open-source servers expose GitLab REST/GraphQL operations through MCP.

The official GitLab server is not a viable baseline for this project's exact target: its feature history says it moved from Premium to Free in GitLab 19.2. The target is 18.8.4-ee with a Free subscription.

Two independent projects merit further source-level evaluation:

1. **zereight/gitlab-mcp** — TypeScript/Node.js, MIT, with a built-in project allowlist and strict-scope mode. It is the more promising starting point for the project's mandatory project-boundary control, but its optional raw API tool has documented bypass behavior and its confirmation parameter is not independent user approval.
2. **jmrplens/gitlab-mcp-server** — Go, MIT, single-binary distribution, large tier-aware action catalog. It aligns with the currently Proposed Go language choice and publishes a detailed assurance case, but the reviewed material did not identify a first-class fixed MCP project allowlist. Its own assurance case explicitly says that `confirm: true` is written by the model and is a request to the model, not a guarantee of user approval.

Neither server is safe to adopt unchanged under the accepted ADRs. The merge gate in ADR-0014 is the decisive blocker for both. An operation-specific server-side approval primitive must not be replaced by a tool argument that the model can generate.

## 2. Target constraints that govern this evaluation

The candidate must preserve all of the following:

- GitLab Self-Managed EE v18.8.4-ee with Free subscription tier.
- One Personal Access Token owned by the user's regular human GitLab account, initially with `api` scope (ADR-0015).
- A server-side canonical project allowlist across every read and write path, including nested and cross-project references. The PAT itself is not project-scoped.
- No write to a default/protected branch; repository code changes use a working branch and MR.
- MCP must never approve or unapprove a GitLab MR on behalf of a human.
- Before merge: at least one independently verified, regular human GitLab approver who is distinct from both PAT owner and MR author, plus a separate exact-operation, single-use MCP user approval enforced on the server (ADR-0014).
- First-release CI/CD is read-only pipeline/job metadata and status; no trigger/retry/cancel, job logs, artifacts, or secret variables.
- PAT retrieval directly from Linux Secret Service in the same interactive desktop D-Bus session. No plaintext file, environment variable, argument, model-visible prompt, or log fallback.
- No implementation or fork is authorized during this evaluation.

## 3. Official GitLab MCP server

**Sources:** [Official documentation](https://docs.gitlab.com/user/model_context_protocol/mcp_server/).

GitLab's published feature history states:
- Introduced as an experiment in 18.3.
- Changed to beta in 18.6.
- Moved from Premium to Free in 19.2.
- Marked generally available in 19.5.

**Assessment:** The 18.8 target is earlier than the Free-tier availability change. The official server therefore cannot be treated as available on the target Free subscription. It is not the current baseline candidate unless the GitLab instance is upgraded to a version where the feature is Free and enabled, which would be a separate user decision and target change.

## 4. Candidate A — zereight/gitlab-mcp

- **Repository:** https://github.com/zereight/gitlab-mcp
- **Reviewed branch:** `main`
- **Pinned commit:** `9bd6a5eba5a9d99d4615a3377c90bacab6981e1d`
- **Commit date:** 2026-11-10
- **License:** MIT
- **Implementation:** TypeScript / Node.js
- **Documented surface at this snapshot:** 40 default tools; 288 total catalog tools.
- **Primary evidence:** [README at pinned commit](https://github.com/zereight/gitlab-mcp/blob/9bd6a5eba5a9d99d4615a3377c90bacab6981e1d/README.md), [security policy](https://github.com/zereight/gitlab-mcp/blob/9bd6a5eba5a9d99d4615a3377c90bacab6981e1d/SECURITY.md), [raw API tool documentation](https://github.com/zereight/gitlab-mcp/blob/9bd6a5eba5a9d99d4615a3377c90bacab6981e1d/docs/tools/api.md), [merge request tools](https://github.com/zereight/gitlab-mcp/blob/9bd6a5eba5a9d99d4615a3377c90bacab6981e1d/docs/tools/merge-requests.md), and [allowlist tests](https://github.com/zereight/gitlab-mcp/tree/9bd6a5eba5a9d99d4615a3377c90bacab6981e1d/test).

### 4.1 Relevant strengths

- Supports self-managed GitLab API URLs and local STDIO mode; Codex configuration is documented.
- Supports PAT authentication and a token-file option.
- Has `GITLAB_ALLOWED_PROJECT_IDS`, with `ENABLE_STRICT_PROJECT_SCOPE` to close discovery gaps and refuse group-/instance-wide operations that cannot be safely scoped.
- Has automated tests for denied project access and project-target scoping, including nested-resource paths.
- Supports toolset selection and a denylist regex, which can reduce the exposed operation surface.
- Explicitly documents the security risk of its raw API tool instead of presenting that feature as safe by default.

### 4.2 Blocking security findings

**Raw API escape hatch.** The `gitlab_api_request` toolset is opt-in, but its own documentation says it bypasses normal tool allowlists, individual tool safety checks, and some group/project restrictions. With a project scope, the path must be under `/projects/:id`, but project IDs appearing in body/query parameters are not checked. The proposed configuration must not enable the `api` toolset, and the fork review must check for any alternative raw REST or GraphQL path.

**Approval is model-controlled.** `GITLAB_TOOL_POLICY_APPROVE` requires the call to carry `_confirmed: true`. The model supplies tool arguments, so this field is not proof that a person approved the exact operation. It does not meet ADR-0003 or the separate merge confirmation in ADR-0014.

**Human review operations are present.** The MR toolset contains `approve_merge_request` and `unapprove_merge_request`. Both must be denied at the server enforcement layer, not just hidden from `tools/list`.

**Credential backend mismatch.** Documented local authentication uses `GITLAB_PERSONAL_ACCESS_TOKEN` or `GITLAB_PERSONAL_ACCESS_TOKEN_FILE`. There is no documented direct Linux Secret Service integration. Passing a keyring value through an environment variable or command argument would violate the current credential-storage proposal; a direct Secret Service adapter or a different accepted credential design would be needed.

**Permission mode requires an explicit choice.** The documented default is `full`; `modify` allows creates/updates and blocks deletes, but it does not supply the project's independent operation approval. The fork would need a closed, explicit first-release tool registry and server-side enforcement, not a permissive default.

### 4.3 Preliminary fit

**Best current fit for project allowlist reuse**, subject to full audit and a server-side merge/approval redesign. It is TypeScript/Node rather than the currently Proposed Go language. The existing allowlist is useful evidence and reusable work, not proof that every edge case in our SEC-010 matrix is covered.

## 5. Candidate B — jmrplens/gitlab-mcp-server

- **Repository:** https://github.com/jmrplens/gitlab-mcp-server
- **Reviewed branch:** `main`
- **Pinned commit:** `088fd585850fce31961b6ebd4d796017be45eaa0`
- **Commit date:** 2026-11-10
- **License:** MIT
- **Implementation:** Go; documented single-binary distribution
- **Documented surface at this snapshot:** more than 1,100 actions across GitLab REST and GraphQL, with a small dynamic discovery/execution surface.
- **Primary evidence:** [README at pinned commit](https://github.com/jmrplens/gitlab-mcp-server/blob/088fd585850fce31961b6ebd4d796017be45eaa0/README.md), [assurance case](https://github.com/jmrplens/gitlab-mcp-server/blob/088fd585850fce31961b6ebd4d796017be45eaa0/site/src/content/docs/operations/assurance-case.mdx), [security model](https://github.com/jmrplens/gitlab-mcp-server/blob/088fd585850fce31961b6ebd4d796017be45eaa0/site/src/content/docs/operations/security.mdx), [dynamic action dispatcher](https://github.com/jmrplens/gitlab-mcp-server/blob/088fd585850fce31961b6ebd4d796017be45eaa0/internal/tools/dynamic/register.go), and [scope filter](https://github.com/jmrplens/gitlab-mcp-server/blob/088fd585850fce31961b6ebd4d796017be45eaa0/internal/tools/scope_filter.go).

### 5.1 Relevant strengths

- Go implementation aligns with ADR-0004's currently Proposed language.
- Supports self-managed instances and GitLab CE/EE; action catalogs are tier-aware.
- Uses a canonical action catalog projected into dynamic, meta, and individual tool surfaces.
- Has documented read-only and safe-preview modes, a detailed threat model and assurance case, automated tests, and release-integrity controls.
- The documented Free/CE baseline includes common project, issue, MR, pipeline, repository, and user workflows.

### 5.2 Blocking security findings

**No first-class fixed MCP project allowlist identified in reviewed source.** The inspected `internal/tools/scope_filter.go` filters actions by PAT scopes such as `read_api`, `api`, and `admin_mode`; it is not a project-ID allowlist. A `project_id` argument chooses the GitLab resource but is not, by itself, an authorization boundary. A complete configuration and route audit is still required before concluding that no other project restriction exists.

**Approval is explicitly not an independent security boundary.** The project's own assurance case says that `confirm: true` is an argument written by the model, and therefore passes without a user prompt; it calls this a request to the model rather than a guarantee. The default dynamic surface does not prompt, and the other surfaces can also accept a model-supplied confirmation. This does not satisfy the project's exact-operation, server-enforced approval requirement.

**Credential backend mismatch.** The documented stdio setup reads `GITLAB_TOKEN` from the environment or a local environment file. The reviewed security documentation does not provide direct Secret Service access. This would need a credential-provider change before the PAT can be used under the current design.

**Large operation surface.** More than 1,100 catalog actions are available across resource domains, including operations outside this project's first-release scope. The implementation would need a tightly bounded catalog, deny-by-default registration, and regression tests proving that omitted operations remain unreachable on every tool surface.

### 5.3 Preliminary fit

**Best language/ecosystem fit** if Go is retained, with strong documentation and security-review discipline. It is not a drop-in fit for the mandatory project allowlist or approval lifecycle; both would require substantial security engineering and evidence.

## 6. Preliminary comparison

| Criterion | GitLab official (18.8 Free) | zereight | jmrplens |
|---|---|---|---|
| Available on target Free tier | No; moved to Free in 19.2 | Self-managed API compatibility documented; verify target instance | Self-managed and CE/EE Free documented; verify target instance |
| Implementation | Integrated GitLab feature | TypeScript / Node.js | Go |
| Local STDIO / Codex | Not suitable on target tier | Documented | Documented |
| Fixed project allowlist | No usable target-tier path established | Yes; strict scope has documented limits and tests | No first-class fixed project allowlist identified in inspected source |
| Raw REST / GraphQL risk | Not assessed for target | Raw API is explicit opt-in and unsafe with allowlist; must remain disabled | Full action catalog includes REST/GraphQL; needs a server-side project boundary |
| Independent user approval | Does not establish our separate MCP gate | No; `_confirmed` is model input | No; `confirm: true` is model input |
| Human MR approval handling | Not sufficient for target policy | Approve/unapprove tools exist; must be denied | Approval actions exist; must be denied |
| Secret Service integration | Not established | Not documented | Not documented |
| Primary advantage | Native, maintained integration | Project allowlist and narrower documented starting surface | Go, single binary, assurance case and extensive tests |
| Primary blocker | Not Free on v18.8 | Model-controlled confirmation; raw API bypass; credential adapter | Missing proven project allowlist; model-controlled confirmation; credential adapter |

## 7. Required next review steps

DES-012 remains OPEN. Complete these checks before making an adoption recommendation:

1. **Pin and inspect all enforcement paths.** Trace every REST/GraphQL call from public tools through canonical project resolution and authorization to the HTTP client. Include redirects, download URLs, aliases, nested IDs, source/target project IDs, and any tool that accepts arbitrary queries or payloads.
2. **Review approval mechanism feasibility.** Identify whether the candidate can host a separate server-owned approval registry with unpredictable, short-lived, single-use approval handles bound to the canonical project ID, MR IID, source HEAD SHA, exact merge parameters, and operation. A model-generated boolean is never accepted as approval.
3. **Verify the GitLab human reviewer path.** The merge gate must consume the exact protocol in `docs/security/reviewer-verification.md`; the candidate's existing approval summary is not enough until it proves regular human identity and excludes the PAT owner and MR author.
4. **Review credential integration.** Confirm a direct Secret Service D-Bus client can be introduced without shelling out to print the PAT or passing the secret through process arguments, environment variables, configuration files, logs, or MCP outputs.
5. **Build a reduced first-release catalog.** Demonstrate that every out-of-scope operation is denied by the server on all public tool surfaces, including raw REST, GraphQL, CLI/skill routes, and alternate transports.
6. **Check exact GitLab 18.8 compatibility.** Run safe API compatibility tests against a disposable target test project only after SEC-002 is accepted and a PAT is safely provisioned.
7. **Review project governance and supply chain.** Check supported release line, dependency licenses and vulnerabilities, CI/test coverage, signed artifacts, maintainer concentration, response to security reports, and an upstream update/patch strategy.

## 8. Current recommendation

Do not adopt either implementation unchanged.

Keep **zereight/gitlab-mcp as the first candidate for deeper project-scope review**, because its fixed project allowlist is closer to the accepted boundary than what has been identified in the Go candidate. Keep **jmrplens/gitlab-mcp-server as a serious challenger**, especially if Go is accepted under ADR-0004, but its project-authorization gap must be addressed first.

This is a provisional research priority, not an architecture decision. DES-012 should end with one of: fork and harden a candidate; build a small policy-enforcing wrapper only if it cannot be bypassed; or keep the current custom design. A wrapper that merely filters tool names is not sufficient when the underlying server exposes unguarded raw API routes.

No upstream repository is to be cloned into this project, no dependency is to be added, no source code is to be modified, and no PAT is to be created as part of DES-012.
