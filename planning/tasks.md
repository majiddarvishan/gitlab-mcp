# Project Task Backlog

- **Backlog status:** Initial planning baseline
- **Last updated:** 2026-11-10
- **Project phase:** Design and requirements
- **Implementation authorization:** Not granted
- **Source of truth:** Accepted ADRs, approved requirements, and the capability matrix

This file is the central task register for the project. Tasks are grouped by delivery phase and have stable IDs so they can later be moved into GitHub Issues or another tracker without losing traceability.

## Task conventions

### Priority
- **P0 — Must:** required before the first useful release or before a dependent task can proceed.
- **P1 — Should:** important for a secure, maintainable release.
- **P2 — Could:** useful enhancement, not a first-release blocker.
  
### Status
- **OPEN:** planned and not yet completed.
- **BLOCKED:** cannot start until a stated dependency, external input, or approval is satisfied.
- **DONE:** acceptance criteria satisfied and evidence recorded.
- **DEFERRED:** intentionally outside the first release.
  
### Task completion
A task is complete only when its acceptance criteria are satisfied and the relevant documentation, tests, or review evidence is committed. Checking a task does not authorize work that is separately marked as not authorized.

---

# Phase 0 — Design closure and environment discovery

These tasks must be completed before implementation can be considered. They are documentation, investigation, and decision tasks; they do not authorize coding.

## DES-001 — Identify the exact GitLab environment

- **Priority:** P0
- **Status:** DONE
- **Depends on:** User-provided environment facts
- **Owner:** User provides instance facts; project records sanitized result
- **Reference:** [GitLab target environment profile](../docs/environment/gitlab-target.md); [validation runbook](../docs/environment/validation-runbook.md)
- **Evidence:** User supplied the exact Help-page version line and the matching public GitLab release tag.

**Confirmed**
- Offering: GitLab Self-Managed.
- Distribution: Enterprise Edition (EE).
- Subscription tier: Free.
- Exact version: 18.8.4-ee.
- Public tag: [v18.8.4-ee](https://gitlab.com/gitlab-org/gitlab/-/tags/v18.8.4-ee).

**Important distinction**
- EE is the installed distribution; Free is the active subscription tier.
- The EE approval response schema includes edition-specific fields even without a paid license, but paid-tier-only endpoints and approval-rule features remain out of scope.

**Acceptance criteria**
- Exact patch, distribution, and subscription tier are recorded in sanitized form.
- The profile contains no private network identifiers or credentials.
- Version-sensitive behavior remains subject to target-instance API validation; environment identification alone does not complete SEC-009, DES-008, or DES-009.

## DES-002 — Verify ChatGPT Desktop and Codex local MCP compatibility

- **Priority:** P0
- **Status:** DONE
- **Depends on:** None
- **Owner:** Project research
- **Evidence:** [Client compatibility research](../docs/architecture/client-compatibility.md)
- **Completion scope:** Official-documentation review only; target-workstation smoke testing remains TST-009.

**Work**
- Verify current official support for local MCP servers in the new ChatGPT Desktop app.
- Verify current Codex support and configuration for the same local server.
- Identify host-specific differences in tool discovery, approval prompts, transport, lifecycle, and configuration.
- Distinguish local desktop MCP from ChatGPT web remote MCP.

**Acceptance criteria**
- A client compatibility matrix cites current official documentation.
- The design identifies which client supports local MCP, in which mode, and with which limitations.
- No assumption is made that ChatGPT web custom MCP and local desktop MCP are equivalent.

**Verified findings (2026-11-10)**
- The new ChatGPT desktop app, Codex CLI, and Codex IDE extension support MCP and share MCP configuration for the same Codex host.
- ChatGPT desktop settings include **MCP servers**, with STDIO and Streamable HTTP options.
- The desktop composer can display connected MCP servers via `/mcp`.
- Codex configuration supports server-level and per-tool approval modes; `writes` prompts for tools not marked read-only.
- Local desktop/Codex MCP configuration is not read by ChatGPT web.
- Linux desktop preview availability is limited to the distributions documented by OpenAI; target-workstation OS confirmation and runtime smoke test remain pending.

## DES-003 — Decide the implementation language

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-002 (complete)
- **Related ADR:** ADR-0004

**Work**
- Evaluate Go against TypeScript/Node.js and other viable options for the selected MCP host and SDK.
- Assess SDK maturity, protocol support, maintenance, packaging, and the user's development workflow.

**Acceptance criteria**
- A decision record is updated to Accepted or Superseded.
- The selected SDK ecosystem is viable for the chosen clients and transport.
- Trade-offs and rejected options are documented.

## DES-004 — Decide the MCP transport

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-002 (complete)
- **Related ADR:** ADR-0005

**Work**
- Validate STDIO against the selected desktop and Codex hosts.
- Compare with any locally supported alternative only if required by host compatibility.
- Document process launch, environment, working directory, protocol output, diagnostics, and shutdown behavior.

**Acceptance criteria**
- A transport decision is explicitly accepted.
- The transport works in the primary supported host and the fallback host, or the documented client limitation is accepted.
- The protocol channel and diagnostic channel are unambiguously separated.

## DES-005 — Select MCP protocol version and SDK

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** DES-003, DES-004
- **Related ADRs:** ADR-0004, ADR-0005

**Work**
- Review maintained SDKs for the chosen language.
- Confirm protocol version, tool/resource/prompt support required by this project.
- Review SDK release activity, license, compatibility, and security posture.

**Acceptance criteria**
- The chosen SDK and protocol version are documented.
- The choice is compatible with both supported clients or client limitations are documented.
- No deprecated or unsupported feature is required for core behavior.

## DES-006 — Finalize the first-release capability inventory

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-001
- **Related document:** docs/gitlab/capability-matrix.md

**Work**
- Resolve remaining Candidate items for Issues, comments/discussions, file move/delete, compare refs, blame, wiki, releases, and advanced repository reads.
- Explicitly identify P0 first-release tools versus later additions.
- Confirm that MR approval mutations, CI/CD writes, job logs, artifacts, secret variables, direct default-branch writes, and protected-branch writes remain excluded.

**Acceptance criteria**
- Every capability has one of: First release, Later release, Deferred, or Prohibited.
- Every write capability has a risk classification and explicit approval requirement.
- No candidate operation silently enters first-release scope.

## DES-007 — Define MCP tool contracts

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** DES-005, DES-006
- **Related documents:** docs/gitlab/capability-matrix.md, docs/requirements/functional.md

**Work**
- Define stable tool names, descriptions, required/optional inputs, output schemas, and error contracts.
- Define project identifiers, branch/ref inputs, pagination, and response-size behavior.
- Separate read tools, prepare/preview operations, and execute/mutation operations.
- Do not expose approve/unapprove MR tools or first-release CI/CD mutation/log/artifact tools.

**Acceptance criteria**
- Every first-release capability maps to a documented tool contract.
- Inputs and outputs are unambiguous and bounded.
- Each mutation contract identifies its approval preview fields.
- Tool contracts are reviewed before implementation.

## DES-008 — Map capabilities to GitLab 18.x REST endpoints

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** DES-001, DES-006
- **Related document:** docs/gitlab/capability-matrix.md

**Work**
- Map every first-release capability to its GitLab REST API v4 endpoint and required permission.
- Identify version, edition, role, token-scope, and instance-setting dependencies.
- Identify the endpoints needed for protected-branch checks, MR approval-state reads, merge preflight, and pipeline/job metadata.

**Acceptance criteria**
- Each first-release capability has a documented endpoint, method, required permission, and expected response fields.
- Unsupported or conditional features are explicitly marked.
- The mapping is checked against the exact target GitLab 18.x environment, not only GitLab.com.

---


## SEC-009 — Verify independent human approver identity on GitLab Free

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** SEC-001 (DES-001 complete)
- **Reference:** [SEC-009 reviewer verification test protocol](../docs/security/reviewer-verification.md); [target-instance validation runbook](../docs/environment/validation-runbook.md)
- **Related decisions:** [ADR-0014](../docs/decisions/0014-free-tier-human-approval-merge-gate.md), [ADR-0015](../docs/decisions/0015-gitlab-token-and-execution-identity.md)

**Work**
- Run the protocol against the exact installed GitLab 18.8 patch and CE/EE distribution using a disposable or explicitly approved test project.
- Resolve the PAT owner through GET /user and the MR author through the MR API; compare numeric GitLab user IDs.
- Read approved_by from the MR approvals endpoint and resolve every candidate approver through the Users API.
- Validate `GET /users/:id` regular-user fields (`bot`, `state`) and `GET /users?username=:username&humans=true&active=true`; verify the exact numeric ID is returned and that the filter excludes bot and internal users on this target instance. Do not treat `bot=false` or `exclude_internal=true` alone as sufficient proof.
- Test regular human, PAT owner, MR author, project/group bot, internal user, inactive/locked user, inaccessible user, and ambiguous/missing metadata cases.
- Verify approval freshness by re-reading the MR head SHA and approval list immediately before merge.
- Do not infer the interaction channel from an approval record; GitLab account metadata does not prove whether approval was submitted through UI or API.
- Keep MCP merge disabled unless every required identity check is reliable and all negative cases fail closed.

**Acceptance criteria**
- The exact target patch and CE/EE distribution are recorded in sanitized form.
- At least one qualifying regular human approver can be distinguished from bots/internal users and is numerically distinct from both PAT owner and MR author.
- The human-account classification method is verified on the target instance; unsupported filters or incomplete metadata fail closed.
- Missing, inaccessible, inactive, ambiguous, stale, or changed approval state blocks merge.
- Sanitized evidence is recorded without publishing user IDs, usernames, project identifiers, internal URLs, or credentials.
- SEC-009 remains OPEN until target-instance evidence is reviewed; defining this protocol alone does not complete the test.
## SEC-001 — Select the GitLab authentication and identity model

- **Priority:** P0
- **Status:** DONE
- **Depends on:** None (the user decision has been received)
- **Reference:** [Authentication model](../docs/security/authentication-model.md)
- **Related ADR:** [ADR-0015 (Accepted)](../docs/decisions/0015-gitlab-token-and-execution-identity.md)

**Decision recorded**
- The user selected a Personal Access Token belonging to their existing human GitLab account.
- GitLab actions performed by MCP are attributed to that human account; no project/group bot identity is used.
- The selected initial scope is api for read/write REST API operations.
- The PAT inherits the owner's effective permissions and can reach all projects/groups available to that account; it is not project-scoped.
- The user's per-project roles are inherited by the PAT; there is no independent role setting on the token.
- The explicit MCP project allowlist remains mandatory and must be enforced server-side.

**Acceptance criteria**
- ADR-0015 is Accepted and records the user's choice.
- Token type, owner identity, effective resource reach, initial scope, and permission model are documented.
- Credential handling and the broader PAT blast radius are recorded in the security requirements.
- No real credential is included in the repository or design artifacts; runtime secret isolation remains a pre-release validation requirement.

**Follow-up validation**
- SEC-010 validates PAT reach against the MCP allowlist.
- SEC-011 validates the user's actual per-project permissions and protected-branch behavior.
- SEC-009 validates independent human-review identity before MCP merge can be enabled.

## SEC-010 — Validate PAT reach against the MCP project allowlist

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** SEC-001, DES-001
- **Reference:** [Authentication model](../docs/security/authentication-model.md); [target-instance validation runbook](../docs/environment/validation-runbook.md)
- **Execution gate:** This is an MCP integration test; do not mark DONE until the allowlist implementation exists and is authorized for testing.

**Work**
- On a disposable test setup, identify one project accessible to the PAT owner that is explicitly allowlisted and one project accessible to the same user that is not allowlisted. This validates the selected personal PAT; do not describe or test it as project-scoped.
- Verify that MCP can perform permitted reads/writes on the allowlisted project subject to the user's GitLab role and operation approval.
- Verify that MCP rejects reads and writes to the accessible-but-unlisted project before returning project data or issuing a mutation.
- Test nested resources, cross-project references, alternate project paths/aliases, and renamed/ambiguous project resolution.
- Confirm that the PAT itself remains capable of accessing the unlisted project directly; document this as the known credential blast radius rather than describing the PAT as project-scoped.
- Record only sanitized outcomes; do not publish project names, IDs, URLs, or credentials.

**Acceptance criteria**
- The MCP allowlist is demonstrated to deny an unlisted project even when the PAT owner can access it.
- All project-scoped read and write paths enforce canonical project IDs.
- Tests and residual risk are documented without internal identifiers or secrets.

## SEC-011 — Validate the PAT owner's permissions against protected-branch operations

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** SEC-001, DES-001
- **Reference:** [Authentication model](../docs/security/authentication-model.md)

**Work**
- Record the PAT owner's effective role on each sanitized test project; the PAT has no independent Developer/Maintainer role.
- Test repository reads, working-branch creation, file updates, commit creation, MR creation, approval-state reads, and merge using the actual account permissions.
- Test merge against the project's actual protected default branch and current branch rules.
- Identify any operation that the account cannot perform and keep that MCP capability unavailable for the project unless GitLab permissions are independently changed through normal administrative governance.
- Review whether the account has broader memberships/roles than this workflow requires.
- Do not elevate the user's role or change protected-branch rules merely to make MCP work.

**Acceptance criteria**
- Effective permissions are recorded per operation and test project without publishing internal identifiers.
- MCP capabilities are limited to operations GitLab authorizes for the user's existing role.
- Any missing permission or excessive account privilege is documented for explicit review.
- No protected-branch policy or account role has been changed by this task.

## SEC-002 — Define local secret storage and credential lifecycle

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** SEC-001

**Work**
- Select an OS-appropriate credential store for the user's workstation.
- Define initial credential setup, replacement, rotation, revocation, and missing/expired token behavior.
- Define safe handling for non-interactive MCP process startup.

**Acceptance criteria**
- Credentials are not stored in source control, plaintext configuration, process arguments, or ordinary logs.
- Setup and rotation procedures are documented.
- Missing, invalid, expired, and revoked credentials produce clear non-secret errors.

## SEC-012 — Validate personal access token expiration and rotation behavior

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** SEC-001, SEC-002
- **Reference:** [Target-instance validation runbook](../docs/environment/validation-runbook.md)

**Current documented baseline (must be verified on target)**
- GitLab currently documents a default expiry date 365 days from token creation when no date is supplied, unless the instance setting is changed.
- Token expiry takes effect at midnight UTC on the expiry date.
- Maximum lifetime enforcement can depend on edition and instance configuration; do not assume Ultimate-only administrator settings exist on a Free target.

**Work**
- Confirm whether the `buffered_token_expiration_limit` feature flag is enabled. The v18.8.4-ee PAT documentation states a 365-day default/max lifetime for Free, with a 400-day limit available from 17.6 via that flag; token expiry occurs at midnight UTC.
- Define renewal, replacement, and immediate revocation for the single user PAT.
- Verify the MCP can distinguish expired, revoked, rotated, and under-scoped tokens without exposing token values.
- Document that rotation/revocation affects MCP access to all configured projects because one PAT is used.

**Acceptance criteria**
- The user PAT lifecycle is documented.
- Rotation does not cause uncontrolled or ambiguous credential state.
- Revoked/expired tokens fail with clear, non-secret diagnostics.

## SEC-003 — Define project allowlist configuration and administration

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-006

**Work**
- Choose the local allowlist representation and canonical project identity format.
- Define how a user adds, reviews, and removes a project.
- Define behavior when a project is renamed, transferred, deleted, or recreated.
- Define whether project paths may be used as human-friendly aliases while authorization uses canonical IDs.

**Acceptance criteria**
- The allowlist format and lifecycle are documented.
- Runtime authorization uses canonical project identity.
- Unknown, ambiguous, and unlisted projects are denied.
- Group membership never implicitly grants access.

## SEC-004 — Define a trustworthy per-operation approval mechanism

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-002, DES-005, DES-007

**Work**
- Verify which client-host confirmation mechanisms are available for local MCP writes in ChatGPT Desktop and Codex.
- Distinguish host-level user confirmation from text generated by the model.
- Define a safe prepare/preview/execute flow if host confirmation alone cannot enforce the policy.
- Ensure a model cannot self-approve by emitting a natural-language confirmation.

**Acceptance criteria**
- The user-facing approval interaction is identified for each supported client.
- The execution layer can prove that an explicit user approval occurred.
- Writes are rejected when approval is absent, ambiguous, stale, or invalid.
- If a client cannot provide a trustworthy approval path, the affected write capability is disabled in that client.

## SEC-005 — Specify the approval state machine and binding

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** SEC-004, DES-007

**Work**
- Define prepare, preview, approve, execute, expire, reject, and cancel states.
- Bind approval to the exact operation, canonical project, target resource, and all material parameters.
- Define expiry, single-use semantics, replay prevention, and concurrent-request behavior.
- Define how stale repository/MR state invalidates an approval.

**Acceptance criteria**
- A documented state machine covers success, rejection, expiry, replay, parameter mutation, and concurrency.
- Any material parameter change requires a new approval.
- An approval cannot be reused for another operation or target.
- Execution revalidates live GitLab preconditions before mutation.

## SEC-006 — Specify merge preflight and merge safety

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** SEC-004, DES-008, DES-011 (complete), SEC-009

**Work**
- Define the fields shown before merge approval: project, MR IID, source/target refs, current source HEAD SHA, merge method/options, readiness, approvals, and pipeline state.
- Define how the server handles draft, closed, conflicted, blocked, stale, or non-mergeable MRs.
- Define race handling if the MR changes after preview or approval.
- Ensure GitLab protected branch, approval, CI, and merge policies remain authoritative.

**Acceptance criteria**
- Merge requires an approval separate from all prior operations.
- At least one verified GitLab reviewer distinct from the PAT owner and MR author has approved the current MR.
- The MCP user approval is separate and bound to the displayed MR state and parameters.
- Stale or changed MR state invalidates the approval.
- Merge is disabled if reviewer independence cannot be verified.
- The server never bypasses a GitLab policy rejection.

## SEC-007 — Define audit logging and diagnostics

- **Priority:** P1
- **Status:** OPEN
- **Depends on:** SEC-001, SEC-004

**Work**
- Define which security-relevant events are recorded: tool call, project authorization result, approval decision, mutation result, and GitLab request correlation.
- Define redaction rules and log retention location.
- Keep MCP protocol output separate from diagnostics if STDIO is selected.

**Acceptance criteria**
- Logs never contain tokens, private keys, session cookies, secret variables, or unnecessary source content.
- Approved and rejected mutations can be correlated without logging sensitive payloads.
- Diagnostics cannot corrupt the MCP protocol stream.

## SEC-008 — Define untrusted-content and data-exposure safeguards

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-006

**Work**
- Treat repository files, comments, issue/MR descriptions, commit messages, and metadata as untrusted content.
- Define prompt-injection handling and ensure retrieved content cannot alter tool authorization or policy.
- Define output redaction, response-size caps, and sensitive-data handling.
- Document that fetched private repository content may be sent to the AI provider for inference.

**Acceptance criteria**
- Retrieved content is clearly treated as data, not tool instructions.
- Secrets are not intentionally surfaced in tool responses.
- Response limits and truncation behavior are documented.
- User-facing documentation explains the cloud inference/data-handling boundary.

## ARC-001 — Finalize component boundaries and trust boundaries

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** DES-005, SEC-001, SEC-004

**Work**
- Define MCP host adapter, GitLab API client, authorization boundary, capability layer, approval manager, configuration/secrets, and diagnostics.
- Document which component is trusted to authorize project access and writes.
- Define dependency direction and prohibit bypass paths around the authorization/approval layers.

**Acceptance criteria**
- An updated architecture diagram and component responsibility document are committed.
- Every project-scoped tool passes through the allowlist guard.
- Every mutation passes through approval enforcement.
- No capability can call GitLab mutation endpoints directly outside the approved execution path.

## ARC-002 — Define configuration schema and defaults

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** SEC-002, SEC-003, DES-004

**Work**
- Define local configuration for GitLab instance alias/base URL, project allowlist, safe defaults, response limits, and diagnostics.
- Separate non-secret settings from credentials.
- Define validation, schema versioning, and configuration migration behavior.

**Acceptance criteria**
- A documented sample configuration contains placeholders only.
- Unsafe defaults are disabled.
- Invalid or ambiguous configuration fails closed with actionable errors.

## ARC-003 — Define GitLab API resilience and concurrency behavior

- **Priority:** P1
- **Status:** OPEN
- **Depends on:** DES-008

**Work**
- Define pagination, timeouts, rate-limit handling, retries, and error normalization.
- Define idempotency and duplicate-write protection when a request times out after GitLab may have accepted it.
- Define optimistic concurrency for file updates and MR merge preflight.

**Acceptance criteria**
- Read and write failure behavior is documented separately.
- Non-idempotent writes are not blindly retried.
- Conflict and stale-state errors are clear and actionable.

## ARC-004 — Define file, diff, and response size policy

- **Priority:** P1
- **Status:** OPEN
- **Depends on:** DES-007

**Work**
- Set maximum file, tree, diff, and tool response sizes.
- Define truncation and pagination behavior.
- Define how binary files, generated files, and unsupported encodings are reported.

**Acceptance criteria**
- Each content-returning tool has documented size limits.
- Truncated results are explicitly marked and cannot be mistaken for complete content.
- Binary or unsupported data is handled safely.

---

# Phase 2 — Implementation

**All tasks in this phase are BLOCKED until the user explicitly authorizes implementation and the design baseline is accepted.** Task registration is not implementation authorization.

## IMP-001 — Create the application repository skeleton

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** DES-003, DES-004, DES-005, ARC-001

**Acceptance criteria**
- The project builds reproducibly.
- Package boundaries match the accepted architecture.
- No secrets or internal environment details are committed.

## IMP-002 — Implement the local MCP host process

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-001, DES-002, DES-004

**Acceptance criteria**
- The selected primary host can launch and communicate with the server.
- The fallback host is supported or its limitation is documented.
- Startup, shutdown, cancellation, and diagnostics behave correctly.

## IMP-003 — Implement configuration validation

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-001, ARC-002

**Acceptance criteria**
- Configuration schema and defaults match the approved design.
- Invalid configuration fails closed.
- No credential is required in ordinary configuration files.

## IMP-004 — Implement secure credential retrieval

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-003, SEC-001, SEC-002

**Acceptance criteria**
- Credentials are retrieved from the approved local secret store.
- Credentials are never returned in tool output or diagnostics.
- Rotation, revocation, and invalid-token behavior are tested.

## IMP-005 — Implement GitLab 18.x API client

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-001, DES-008, SEC-001, ARC-003

**Acceptance criteria**
- API v4 requests use the configured GitLab instance and secure TLS validation.
- Pagination, timeouts, rate limits, and normalized errors are handled.
- Integration tests run against the target GitLab 18.x instance.

## IMP-006 — Implement project identity resolution and allowlist guard

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-003, IMP-005, SEC-003, ARC-001

**Acceptance criteria**
- Every project-scoped read and write enforces canonical project allowlist membership.
- Unlisted, ambiguous, renamed, and unknown projects fail closed.
- No tool argument or nested resource reference bypasses the guard.

## IMP-007 — Implement core project and repository read tools

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-005, IMP-006, DES-007

**Scope**
- List allowlisted projects.
- Read project metadata.
- Read repository tree.
- Read files at explicit refs.
- List branches and tags, including protection metadata where available.

**Acceptance criteria**
- All returned project data is allowlist-scoped.
- Ref and path inputs are validated and URL-encoded.
- Large responses follow ARC-004 limits.

## IMP-008 — Implement repository search and commit read tools

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-005, IMP-006, DES-007, DES-008

**Scope**
- Search supported repository content.
- List and inspect commits.
- Read commit diffs.
- Compare refs if included in the accepted first-release matrix.

**Acceptance criteria**
- Search behavior and limitations match the target instance.
- Commit and diff responses are bounded and correctly scoped.
- Unsupported search features are reported clearly.

## IMP-009 — Implement merge request read tools

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-005, IMP-006, DES-007, DES-008

**Scope**
- List and inspect MRs.
- Read changes, diffs, commits, and discussions.
- Read MR approval state and merge-readiness metadata where supported.

**Acceptance criteria**
- Project allowlist is enforced.
- Approval state is read-only.
- No approve/unapprove/reset-approval mutation is exposed.

## IMP-010 — Implement read-only pipeline and job metadata tools

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** IMP-005, IMP-006, DES-007

**Acceptance criteria**
- Pipeline and job metadata/status are readable for allowlisted projects.
- No trigger, retry, cancel, log/trace, artifact, or variable-value endpoint is exposed or called.

## IMP-011 — Implement the approval and mutation execution boundary

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** SEC-004, SEC-005, IMP-006

**Acceptance criteria**
- Every mutation is rejected unless the exact operation has a valid approval.
- Approval is single-use, expires, and is invalidated by parameter or state changes.
- Model-generated text cannot be interpreted as approval.
- Negative tests cover missing, stale, replayed, and modified approvals.

## IMP-012 — Implement working-branch and code-commit operations

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-007, IMP-011, SEC-006

**Scope**
- Create working branch.
- Prepare and preview file changes.
- Commit approved file actions to the working branch.

**Acceptance criteria**
- Branch creation and every commit require separate approval.
- Writes to default/protected branches are rejected regardless of tool arguments.
- Commit author/committer attribution matches the accepted identity decision.
- Concurrency conflicts are surfaced without blind retry.

## IMP-013 — Implement MR creation and approved updates

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-009, IMP-012, IMP-011

**Acceptance criteria**
- MR source and target branches are explicit.
- MR create/update operations require their own approval.
- An MR cannot be created from an unapproved or unauthorized project/branch context.
- MR approval mutations are not exposed.

## IMP-014 — Implement separately approved merge operation

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** IMP-009, IMP-011, SEC-006

**Acceptance criteria**
- Merge preview contains all required MR identity, ref, SHA, method, and readiness data.
- A fresh preflight runs immediately before execution.
- Separate merge approval is mandatory and single-use.
- GitLab policy failures are returned without bypass attempts.

## IMP-015 — Implement only the additional project-management tools approved in DES-006

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** DES-006, DES-007, IMP-005, IMP-006, IMP-011

**Scope**
- Only issue/comment/discussion capabilities explicitly included in the approved first-release matrix.

**Acceptance criteria**
- No Candidate feature is implemented without explicit inclusion in the approved matrix.
- Every mutation follows the approval contract.
- Delete/high-risk actions remain excluded unless separately approved.

## IMP-016 — Implement safe diagnostics and audit events

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** SEC-007, IMP-001

**Acceptance criteria**
- Security-relevant events are auditable without storing secrets or excessive source content.
- STDIO protocol output, if selected, is never mixed with logs.
- Error details are useful but do not disclose credentials or internal network configuration unnecessarily.

---

# Phase 3 — Verification and security testing

**These tasks are also BLOCKED until implementation is authorized and available.**

## TST-001 — Unit-test configuration, validation, and API client

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-003, IMP-005

**Acceptance criteria**
- Valid/invalid configuration, path/ref validation, pagination, timeout, and error mapping are covered.
- Tests do not require real company credentials.

## TST-002 — Test project allowlist isolation

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-006

**Acceptance criteria**
- Allowed projects work.
- Visible but unlisted projects are denied for reads and writes.
- Cross-project and nested-resource references cannot bypass the allowlist.
- Rename/transfer/recreation cases are covered.

## TST-003 — Test approval integrity and replay resistance

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-011

**Acceptance criteria**
- Missing, invalid, expired, reused, replayed, and parameter-modified approvals are rejected.
- Approval for one operation cannot authorize another.
- Concurrent execution cannot consume the same approval twice.

## TST-004 — Test working-branch and commit restrictions

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-012

**Acceptance criteria**
- Writes to default and protected branches are rejected.
- Working-branch writes succeed only after exact approval.
- Stale base/head state and file conflicts are handled safely.

## TST-005 — Test merge preflight and GitLab policy enforcement

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-014

**Acceptance criteria**
- Draft, closed, conflicted, blocked, stale, and non-mergeable MRs are handled safely.
- Missing human GitLab approval or separate MCP merge approval always blocks merge.
- Bot/internal approvals do not satisfy the human-approval gate.
- GitLab policy requirements are never bypassed.
- Merge approval cannot be reused after MR state changes.
- GitLab rejection is treated as final for that attempt.

## TST-006 — Test CI/CD read-only boundary

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** IMP-010

**Acceptance criteria**
- Pipeline/job metadata and status can be read.
- No job logs, artifacts, secret variables, or mutation endpoints are exposed or invoked.

## TST-007 — Security test content handling and secret leakage

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-007, IMP-009, IMP-016

**Acceptance criteria**
- Prompt-injection-like content in repository files and GitLab metadata cannot change tool policy.
- Secret-like test values are not leaked through logs or diagnostics.
- Binary/oversized/hostile content is handled within documented limits.

## TST-008 — Test failure, retry, and concurrency behavior

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** IMP-005, IMP-012, IMP-014

**Acceptance criteria**
- Network timeout after a possible write does not cause an unsafe blind retry.
- Rate limits and server errors are handled consistently.
- Concurrent update and stale-state errors are explicit.

## TST-009 — End-to-end client compatibility tests

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** IMP-002, IMP-007, IMP-011

**Acceptance criteria**
- The primary supported ChatGPT Desktop workflow is tested.
- The Codex fallback is tested where supported.
- Tool discovery, read calls, approval prompts, rejected writes, and successful approved writes are verified.
- Any client-specific restriction is documented.

---

# Phase 4 — Packaging, release, and operations

## REL-001 — Define local installation and configuration procedure

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** IMP-002, IMP-003, IMP-004

**Acceptance criteria**
- Installation, host configuration, credential setup, allowlist management, update, and uninstall procedures are documented.
- No internal URL or credential appears in public examples.

## REL-002 — Add build and release packaging

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** IMP-001, IMP-002, TST-001

**Acceptance criteria**
- Reproducible release artifacts are produced for the supported workstation OS/architectures.
- Version and checksum information is published.
- Upgrade and rollback behavior is documented.

## REL-003 — Complete security and public-repository review

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** TST-002, TST-003, TST-005, TST-007

**Acceptance criteria**
- No credentials, private source, internal network details, or customer data are present.
- Security requirements and actual behavior are reconciled.
- High-impact operations and residual risks are documented.

## REL-004 — Publish user, operator, and troubleshooting documentation

- **Priority:** P1
- **Status:** BLOCKED
- **Depends on:** REL-001, REL-002, TST-009

**Acceptance criteria**
- Users can install, configure, operate, troubleshoot, rotate credentials, and revoke access using the documentation.
- Client-specific setup and limitations are clear.
- The first-release capability boundary is documented.

## REL-005 — Release readiness review

- **Priority:** P0
- **Status:** BLOCKED
- **Depends on:** All P0 tasks in Phases 0–4

**Acceptance criteria**
- All P0 acceptance criteria are satisfied.
- No known critical/high security issue remains unresolved.
- GitLab 18.x integration tests pass against the target environment.
- Client compatibility is demonstrated.
- User explicitly approves the release.

---

# Current next-action queue

The next design and validation tasks, in order, are:

1. **DES-001** — Confirm the exact GitLab 18.8 patch and CE/EE distribution.
2. **SEC-009** — Validate independent human reviewer classification on the exact target instance using the [SEC-009 test protocol](../docs/security/reviewer-verification.md). Merge remains disabled until this passes.
3. **SEC-010 / SEC-011** — Validate the selected personal PAT's allowlist boundary and the user's actual per-project permissions. These are not Project Access Token tests.
4. **SEC-002 / SEC-003** — Finalize OS-native PAT storage and local canonical project allowlist administration.
5. **SEC-004 / SEC-005** — Finalize trusted user approval interaction and server-side enforcement.
6. **DES-003 / DES-004 / DES-005** — Decide Go, STDIO, and the MCP SDK/protocol version.
7. **DES-006 / DES-007 / DES-008 / DES-009** — Freeze the first-release tool list, tool contracts, API mapping, and Free-tier code-search behavior.
8. **DES-010** — Record the GitLab 18.8 upgrade plan or explicit risk acceptance.
9. **TST-009** — Run the client smoke test on the target workstation; this is not implementation authorization.

**SEC-001 is DONE:** the user selected a Personal Access Token (PAT) belonging to their existing human GitLab account; see ADR-0015.

No implementation task is authorized by this backlog. Implementation remains blocked until the design baseline is reviewed and explicitly approved by the user.


## DES-009 — Validate Free-tier project code search

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-001 (partially confirmed), DES-008
- **Reference:** [GitLab target environment profile](../docs/environment/gitlab-target.md)

**Work**
- Verify that project-scoped basic code search is available through the GitLab 18.8 Free instance's REST API.
- Confirm supported search modes, branches/refs, filters, pagination, and result shape.
- Keep group/global advanced and exact code search out of scope for Free.
- If the API behavior is insufficient, compare targeted file retrieval, repository archive, and local Git clone/index as alternatives. Do not choose a fallback without documenting storage, freshness, and data-exposure implications.

**Acceptance criteria**
- A tested endpoint and request shape are documented for project-level basic code search, or the capability is marked unavailable.
- Search results remain scoped to one allowlisted project.
- The final search strategy does not depend on Premium/Ultimate, Elasticsearch, Zoekt, or GitLab Duo.

## DES-010 — Review GitLab 18.8 maintenance and upgrade plan

- **Priority:** P0
- **Status:** OPEN
- **Depends on:** DES-001
- **Reference:** [GitLab target environment profile](../docs/environment/gitlab-target.md)

**Work**
- Confirm the exact installed patch.
- Compare the installed release with GitLab's current maintained release lines and patch policy.
- Ask the responsible GitLab administrator to confirm the planned upgrade path or document the organization's accepted risk.
- Do not change the MCP compatibility target without an explicit user decision.

**Acceptance criteria**
- The backlog records whether an upgrade is planned, the responsible owner/role, and a target window, or records an explicit risk acceptance.
- No private hostname or operational detail is published.
- The MCP can continue to target 18.8 for compatibility testing while release readiness tracks the maintenance risk separately.

## DES-011 — Decide whether MCP must require human MR approval before merge

- **Priority:** P0
- **Status:** DONE
- **Depends on:** User decision
- **Evidence:** [ADR-0014 — Accepted](../docs/decisions/0014-free-tier-human-approval-merge-gate.md)

**Decision recorded**
- Require at least one verified human approval recorded in GitLab.
- Require a separate, exact-operation user approval through MCP before executing the merge.
- Never let MCP create or modify the human review approval.

**Acceptance criteria**
- ADR-0014 is Accepted.
- Merge preflight and test requirements reflect both independent approval conditions.

**Follow-up technical validation**
- Verify that the deployed Free 18.8 CE/EE distribution exposes enough user metadata to identify bot/internal accounts. Tracked separately by SEC-009.
