# LFR-P02-D02 | Declarative Scheme Systems & Governed Execution / Agent Authority & Controlled Execution

**Document class:** Detailed Design Proposal, one specialized edition only  
**Date:** 12 November 2026  
**Programme:** Lisp Foundation Reserve, current concept LFR-C02  
**Edition:** P02 — Declarative Scheme Systems & Governed Execution  
**Design slice:** D02 — Agent Authority & Controlled Execution  
**Language:** Gerbil Scheme (new independent implementation, not a source port)  
**Main source:** Mark Watson, Gerbil Coding Harness, pinned revision `1335cb2708926213ba62afd2998754358f655db1` (11 November 2026)  
**Research context:** Existing Coding Harness source audit; LFR-C02; P01-D01–P03-D01 are independent sibling designs  
**Status:** `DETAILED_DESIGN_PROPOSED / SOURCE_LINEAGE_EXPLICIT / SECURITY_BOUNDARIES_SPECIFIED / RUNTIME_UNQUALIFIED`  
**Publication:** Owner-approved documentation publication on 12 November 2026 under `docs/design/` in `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`. This publication does not authorize source code changes, implementation, deployment, runtime tool execution, or a new repository.  
**Scope isolation:** Does not modify LFR-C02, P01-D01–P03-D01, existing reviews, Alien-Works source, or historical isolated concept 04.

---

## 0. Decision and product purpose

Design **one independent Gerbil specialized reserve** that converts potentially untrusted proposals from an LLM or another caller into **explicitly scoped, policy-checked, inspectable, bounded tool operations**. This is a useful software capability even with no LLM: a deterministic caller can request the same tools through the same contracts.

Two separately qualifying profiles:

- **P02-D02-A — Authority & Policy (primary, pure):** tool contracts, schema validation, effect classification, contextual policy evaluation, binding approvals to exact requests, and structured decisions. No filesystem or subprocess operations.
- **P02-D02-B — Controlled Local Execution (optional, effectful):** narrowly registered read operations, staged edits, checkers, subprocess invocation, operational receipts, and recovery. A feature-specific driver must qualify its guarantees. P02-D02-B consumes decisions from P02-D02-A rather than inheriting a global `auto-approve` flag.

Optional *outside* adapters may provide model proposals or UI approval, but P02-D02 has no mandatory OpenAI, Gemini, cloud, browser, MCP, database, GUI, Guix/Guile, P01-D01, P02-D01, P03-D01, or Meta-Foundation dependency.

The **goal is not to make arbitrary model code safe merely by putting it inside Gerbil**. The local tool runtime executes with the permissions and kernel protections of its process. For hostile code, operating-system sandboxing and privilege separation are separate acceptance requirements.

### Product use cases

1. An unattended **read-only project analyst** allowed to inspect selected files without execution.
2. An **approval-gated coding assistant** that proposes and validates file changes, then commits exactly an approved revision.
3. A **bounded local maintenance runner** that invokes explicitly declared commands and captures real outcomes.
4. A consumer application exposing a few **purpose-built operations**, without needing an AI conversation or universal shell.

### Non-goals

Not an autonomous agent platform, LLM framework, general-purpose system policy authority, OS security kernel, container runtime, process supervisor, universal filesystem, Git implementation, editor, package manager, or generic transactional database. No arbitrary `eval`, `sh -c`, or direct write tool is enabled by default. A custom VM is not warranted.

## 1. Threat, trust, and control boundaries

The threat model distinguishes five parties; treating them as equivalent would erase the entire purpose of P02-D02.

| Party | Trust assumption | Allowed role |
|---|---|---|
| **Application owner** | Supplies bounded configuration and approvals | Select tool set, root, policy, budgets, and external isolation strategy |
| **P02-D02-A policy evaluator** | Trusted local implementation and configuration | Validate and decide; deterministic audit records |
| **P02-D02-B executor/driver** | Trusted implementation, but operating inside host OS permissions | Execute authorized operations; report what occurred |
| **Agent/model caller** | Untrusted proposal producer, even when helpful | Submit typed requests; never self-grant rights |
| **Repository content, search output, tool text** | Untrusted data | Supply context only; not authority, policy or developer instructions |

Assumptions and limits:

1. **Policy is not containment.** A same-process library cannot defend against malicious arbitrary native code with equivalent privileges. OS-level enforcement is separate.
2. **Approval is not confinement.** A valid approval proves a particular decision was recorded, not that the kernel prevents other effects.
3. **Filesystem scope is not a security boundary unless enforced on actual opened objects.** Lexical path checking, string-prefix checks, and one-time `realpath` inspection alone are inadequate against concurrent symlink swaps.
4. **Commands are not pure just because they are called `check` or `test`.** A build/checker can write files, access secrets, or launch descendants.
5. **Action completion is not external-state recovery.** A failed effect may leave partial changes or an unknown result.
6. **Secrets are never inserted into full-text receipts.** Redact and reference secret identities, not secret values.
7. **Trust is local to explicitly configured capabilities.** A model response, tool output, injected instruction, or repository file cannot change the policy.

A `policy-decision=ALLOW` is sufficient only within the *configured* trust setting. A security-sensitive deployment must additionally qualify OS isolation (e.g., dedicated UID, restricted filesystem view, namespace/container/cgroup policy where appropriate).

## 2. Source-lineage and defect-resolution register

Evidence statuses: **S** = directly inspected static source; **R** = existing bounded review; **D** = documentation-level official behavior; **P** = proposed P02-D02 requirement, **not implemented**.

| ID | Source: original problem and mechanism | Verified strength | Known weakness or boundary | Proposed P02-D02 correction | Falsification evidence |
|---|---|---|---|---|---|
| **CH-01** | Coding Harness `tools.ss`: centralized `define-tool` registry | Tool inventory and schemas are explicit (S) | Registry schema alone grants no authority or effect constraints | Versioned `ToolSpec`; separate `EffectSpec`, `PolicyDecision`, `ExecutorBinding` | Unknown/malformed/mismatched tool requests fail closed |
| **CH-02** | `tools.ss:134–150`, `checker.ss:131–132`: file tools / project inclusion | Useful file and diagnostic tools (S) | Inadequate root containment; string-prefix sibling confusion (S) | Root-handle scoped, race-resistant file driver; deny absolute/escape paths | `../`, sibling, symlink, rename race, `/proc` magic-link tests |
| **CH-03** | `approval.ss`: render diff and ask user | Human confirmation and noninteractive EOF refusal (S) | A displayed diff is not a cryptographic/exact binding to future execution | Approval record binds canonical intent, tool/policy version, root and base revisions | Modify any bound field after approval; execution must refuse |
| **CH-04** | `tools.ss:288–294`: manual vs `auto-approve` for eval | Flexible operator workflow (S) | Auto approval intentionally bypasses prompt and is not a sandbox (S) | No global auto-authority; only a narrowly scoped owner policy with explicit risk tier | High-risk tool denied even when low-risk edits allowed |
| **CH-05** | `tools.ss:299–354`: stale-base edit + apply/check | Detects simple stale text and checks syntax (S) | Writes before checks; failed check may leave dirty state (S) | Separate prepare/validate/approve/commit; report irreversible checker effects | Forced failure leaves target unchanged in atomic single-file profile |
| **CH-06** | `checker.ss`: language profiles, delimiter and syntax diagnostics | Reusable diagnostics and per-language checks (S) | Delimiter checks are not proof of full syntax or semantic correctness | Checkers have declared type, side effects, exit semantics and trust | Malformed syntax, misleading output, checker crash, forced write during check |
| **CH-07** | `util.ss:32–73`: argv-backed `open-process` | No implicit shell-string interpolation in documented call (S) | Input-port inactivity timeout != hard monotonic wall deadline; PID kill != descendant cleanup (S/R) | Driver capability negotiation for deadline, process identity, group cleanup | Continuous-output timeout and detached-child tests |
| **CH-08** | `harness.ss:372–395`: one-shot status from transcript strings | Attempt to return exit status to automation (S) | Previous `FAILED` text can contaminate final code (S) | Typed current-run outcomes with explicit precedence, not log scanning | `fail → repair → pass`, and failure text in harmless output |
| **CH-09** | `llm.ss:170–220`: iteration and repetition cap | Limits simple repeated tool-call loops (S) | Identical polling can be legitimate; loops can vary wording | Resource budgets + operation-specific progress and idempotency semantics | Polling with new state allowed; nonprogress bounded |
| **CH-10** | `lang.ss`: per-language dispatch and syntax checker | Multi-language tool behavior is inspectable (S) | Language selection cannot substitute for scoped execution and authority | Language profiles are *data* and checker descriptors; not permission grants | Changing file extension cannot elevate capabilities |
| **PA-01** | Personal AI CLI review: API errors can be treated as text answers (R) | Clear example of typed-result need | Failure values can be misclassified as valid model output | Tagged `OperationResult`, `Failure`, `Uncertain` separate from human prose | API error, validator error and refused action cannot become `SUCCESS` |
| **P02-D01-X** | P02-D01 optional process-driver design (concept only) | Distinguishes OS supervision from actor lifecycle | P02-D01 remains unbuilt, and P02-D02 cannot claim its capabilities | Own minimal `CommandDriver`; optional independent future adapter, **no P02-D01 dependency** | Dependency audit and real-driver capability tests |

**Evidence boundary:** The source findings are static; P02-D02 remedies are proposals. Source evidence does not establish successful execution of any P02-D02 design. The Coding Harness repository headers identify AGPL-3.0; preserve attribution and assess license obligations before any verbatim source reuse. Studying algorithms/contracts is not automatic authorization to port code.

## 3. Internal architecture and dependency direction

```text
OUTSIDE P02-D02
  application / CLI / optional LLM caller / optional approval UI
                      |
                typed ToolRequest
                      v
P02-D02-A [PURE, MAIN PROFILE]
  registry -> normalize -> schema-validate -> effect-classify
             -> policy-evaluate -> prepare approval commitment
             -> authorize exact request or refuse
                      |
            immutable authority decision
                      v
P02-D02-B [OPTIONAL, EFFECTFUL]
  executor gate / decision verification
    +-- scoped read adapter
    +-- staged-edit adapter -> isolated validator -> commit gate
    +-- command driver (mock first; real OS driver only when qualified)
    +-- observation + recovery
                      |
                 RunReceipt
                      |
OUTSIDE P02-D02
  application interprets final outcome and manages larger workflow
```

**Dependency rules:**

- P02-D02-A may not import filesystem mutation, process, network or model modules through runtime dependencies.
- P02-D02-B imports P02-D02-A's *public contracts*, not its private policy internals.
- The optional model-facing adapter imports the public request schema, never the executive driver directly.
- No source-level dependency on original Coding Harness, AGPL code, Guix, Guile, Shepherd, P01-D01–P03-D01, or Meta-Foundation is required.
- Event orchestration and temporal directors remain P03-D01's specialty; persistent OS supervision remains P02-D01's specialty. P02-D02 owns a *bounded invocation*, not a daemon or scheduler.

### Proposed Gerbil module layout (not yet created)

```text
editions/agent-authority/
  gerbil.pkg
  README.md
  lineage/
    source-lock.md
    known-findings.md
  src/
    authority/
      identity.ss
      tool-spec.ss
      schema.ss
      effects.ss
      policy.ss
      decisions.ss
      approval.ss
      budgets.ss
      canonical.ss
    execution/
      gate.ss
      outcome.ss
      receipts.ss
      scope.ss
      read-file.ss
      edit-proposal.ss
      staging.ss
      checker.ss
      commit.ss
      process-driver.ss
      command.ss
      recovery.ss
      logging.ss
    adapters/
      mock-fs.ss
      mock-command.ss
      linux-fs.ss            # candidate; only after qualification
      linux-process.ss       # candidate; only after qualification
    optional/
      llm-request-translator.ss
      cli-approval-renderer.ss
  tests/
    authority/
    filesystem/
    edit-staging/
    process/
    negative/
    replay/
  qualification/
    drivers.md
    findings.md
    receipts/
```

Module names and folders are **proposed**, not compiled Gerbil source. `src/authority/` and `src/execution/` define separate packages/profiles in the first implementation.

## 4. Core data model and ownership of contracts

All records have an explicit version; unknown required fields fail closed; public serialized data is validated before trust.

| Contract | Essential fields | Critical invariant |
|---|---|---|
| `ToolSpec` | `tool_id`, version, input/output schemas, effect set, driver requirements, max risk tier | An unregistered tool cannot execute |
| `CallerContext` | caller identity, origin (`MODEL`, `HUMAN`, `PROGRAM`), session, owner, trust label | `MODEL` origin is never policy owner |
| `WorkspaceScope` | root identifier, root-handle generation, allowed operations/relative patterns, mount/symlink policy | Paths resolve against the approved *opened root*, not string prefixes |
| `ToolRequest` | request ID, tool ID/version, canonical args, target identities, base revisions, caller, max budget | Request fields become immutable after normalization |
| `EffectSpec` | classes, target set, reversibility declaration, checker effects, external targets | Effect classification cannot be weakened by caller text |
| `PolicySpec` | policy revision, authorized identities, allowed tools/effects, scope, max bounds, confirmation rules | Deny unless a rule explicitly allows it |
| `PolicyDecision` | `ALLOW`, `DENY`, `REQUIRE_APPROVAL`, `UNSUPPORTED`, reason, digest of request+policy | Approval required cannot be mistaken for allow |
| `ApprovalCommitment` | exact canonical operation digest, scope/base, expiry, approver, interaction evidence, single-use nonce | Only the exact approved request can proceed |
| `ExecutionEnvelope` | verified decision, driver ID/capability set, deadline, quotas, correlations | Driver must satisfy requested guarantees or refuse |
| `CheckResult` | validator identity/revision, input digest, observed status, diagnostics, declared effects | `PASS` is not inferred from absence of error text |
| `OperationResult` | tagged status, exit status/signal, committed effects, unverified effects, error data | Partial or unknown effects are visible |
| `RunReceipt` | attempt ID, exact request/decision/driver hashes, monotonic duration, observations, redaction status | Results and provenance cannot be silently overwritten |

The `ApprovalCommitment` is **logical application authorization**, not a cryptographic hardware security token. An in-process attacker able to change trusted state may bypass it. For remote approvals, authentication and secure channel design are a separate future adapter qualification.

### Proposed result taxonomy

```text
REQUEST_INVALID | POLICY_DENIED | APPROVAL_REQUIRED | APPROVAL_REJECTED
APPROVAL_EXPIRED | APPROVAL_STALE | UNSUPPORTED_CAPABILITY
PREPARED | VALIDATION_FAILED | READY_TO_COMMIT
COMMITTED | EXECUTION_FAILED | TIMED_OUT | CANCELLED
PARTIAL_EFFECT | UNKNOWN_EFFECT | RECOVERY_REQUIRED
CLEANUP_UNVERIFIED | INTERNAL_ERROR
```

Never collapse `TIMED_OUT`, `UNKNOWN_EFFECT`, or `CLEANUP_UNVERIFIED` into a generic success. Every result distinguishes **request acceptance**, **execution outcome**, and **observed postcondition**.

## 5. Policy evaluation, delegation and approvals

### 5.1 Closed-world tool registry

Every tool carries immutable identity and version, validated argument schema, effect tags, target selectors, driver capability requirements, max request size, quota class, and default-deny status.

Tool names are not permissions. A tool labeled `read_only` can still leak credentials if its filesystem scope includes secrets; a tool labeled `check_syntax` can launch arbitrary code if its check command is supplied by an untrusted repository.

### 5.2 Effect classes

Initial vocabulary: `READ_SCOPED`, `LIST_SCOPED`, `PROPOSE_EDIT`, `VALIDATE_STAGED`, `COMMIT_EDIT`, `SPAWN_BOUNDED`, `EXEC_CODE`, `EXTERNAL_NETWORK`, `DELETE`, `MUTATE_EXTERNAL`.

An exact tool can declare more than one effect. An operation that invokes a compiler or package manager must declare its possible writes and process execution, even when the user's intent is merely "check."

### 5.3 Decision ordering

```text
Receive request
  -> schema/size/version validation
  -> resolve tool and immutable effect classification
  -> resolve caller, workspace root and target identities
  -> evaluate owner policy and environment constraints
  -> check quotas / driver capabilities / required isolation
  -> DENY, UNSUPPORTED, REQUIRE_APPROVAL or ALLOW
  -> bind approval to exact canonical request (when required)
  -> revalidate target/base/policy at execution time
  -> execute through the named driver only
```

A lower-trust caller cannot manufacture an `ALLOW` record. P02-D02-B accepts only decisions from the configured P02-D02-A evaluator and checks their binding. The approval interface is deliberately decoupled from model chat content.

### 5.4 Approval binding and revocation

Compute a commitment over a deterministic canonical serialization including:
`request_id`, tool/version, typed arguments, workspace root identity, target identities, expected base versions/digests, effects, checker specification, policy revision, target driver/capabilities, allowed deadline/budget and approver scope.

Approval expires after a defined duration (elapsed time measured monotonically in-process; recovery after restart needs an explicit persisted and trustworthy time model). The approval is single-use for effectful operations. If file, target, policy, tool version, environment or risk scope changes, **new approval** is required.

The final commit gate must check the same base identity **again**. Displaying a diff, then changing its contents afterward, must never inherit an earlier approval.

### 5.5 Delegation

No general transitive delegation. A delegated tool credential, if introduced later, must only attenuate (never increase) tool set, scope, budget, and lifetime. A child agent cannot authorize its own escalation. Default P02-D02 supports no agent-created subagents or automatic permission inheritance.

## 6. Scoped filesystem and path-resolution protocol

### 6.1 Defend the actual operation, not only the spelling

`../x`, `/absolute`, `root-copy/foo`, symlink chains, magic links, mount traversal, hard-link aliases, renamed directories and concurrent swaps require distinct tests. A path prefix test must **not** be an authorization check.

P02-D02's *future Linux driver* is proposed to open an owner-selected root directory and operate via directory file descriptors. Linux `openat2()` provides policy flags such as `RESOLVE_BENEATH`, `RESOLVE_NO_MAGICLINKS` and optionally `RESOLVE_NO_SYMLINKS` or `RESOLVE_NO_XDEV`; the exact choice depends on the scope policy. `RESOLVE_BENEATH` alone must not be advertised as a complete filesystem sandbox.

A descriptor-relative strategy still needs to handle **write operations**, concurrent renames, mount conditions, creation semantics, and which filesystem objects the process can access through other APIs. The driver must pass race tests. If adequate primitives are unavailable, **refuse high-assurance scoped write operations** instead of falling back silently to `realpath` + ordinary `open`.

### 6.2 Root identity and lifetime

`WorkspaceScope` binds to a specific opened root/inode identity and driver generation, not merely a directory string. Root replacement, ancestor rename, or changed mount policy invalidates the scope or requires revalidation according to the selected safety model.

Symlink policy is explicit: default strict profile forbids path traversal through symlinks for mutation. If a relaxed mode exists, it needs independent qualification.

### 6.3 File operation classifications

- `READ_SCOPED`: content is data; restrict size and file type; don't follow forbidden links.
- `LIST_SCOPED`: restrict traversal counts, depth and entry bytes; don't leak outside metadata.
- `PROPOSE_EDIT`: calculate candidate bytes and diff in memory/staging; no target write.
- `COMMIT_EDIT`: only through checked base-target operation.
- `DELETE`: absent from default profile, separately authorized if ever introduced.

Canonical text identity is not sufficient to defend against hard-link aliases or race conditions. The OS driver must document which security properties it really enforces.

## 7. Staged edit and validation protocol

### 7.1 State machine

```text
PROPOSED -> NORMALIZED -> BASE_CAPTURED -> STAGED
                                      -> VALIDATED
                                      -> APPROVAL_PENDING -> APPROVED
                                      -> RECHECK_BASE -> COMMITTING
                                      -> COMMITTED -> VERIFIED
                            \-> DENIED / ABORTED / STALE / VALIDATION_FAILED
                                      \-> PARTIAL_EFFECT / UNKNOWN_EFFECT
```

An approval may be obtained after rendering a candidate, but **before commit**. A validator's outcome can invalidate the earlier approval if it changes what will be written.

### 7.2 Procedure (proposed; no code)

1. Open the target relative to the approved workspace root, recording file identity, mode and exact byte digest. Explicitly distinguish absent target from unreadable target.
2. Produce new content in a private **staging workspace**; do not mutate the target. Bind all proposed files to an ordered `ChangeSet`.
3. Validate the candidate using deterministic text/AST checks first. Executable validators require an explicitly declared OS isolation profile and may not have access to the live target.
4. Render exact diff and checker evidence. Request authorization bound to the change-set digest and base identities.
5. Reopen/recheck the target through the scoped driver, not through a stale path string. Reject changed bases.
6. Commit with target-specific atomic replacement if supported and qualified, preserving required permissions and metadata. Record commit intent before effects.
7. Confirm post-commit bytes and file identity. Persist a receipt with `COMMITTED` or a precise partial/unknown state.

**Atomicity ceiling:** a same-filesystem rename can make a *single-file replacement* atomic as a namespace operation; it is not a multi-file transaction or a guarantee of crash durability. Correct durability protocols (file and parent-directory syncing, temporary-file placement, error propagation) need OS-level tests. Git objects, compiler caches, config side effects and external commands are not automatically reverted by a file rename.

For **multi-file** edits, D01 does not promise all-or-nothing commit. Options include fail-closed refusal in high-assurance mode or a journaled sequential commit explicitly reporting partial state. A true transactional workspace is a different engineering project.

### 7.3 Validation side effects

Never run an arbitrary project test/build directly in the supposedly pure staging step and call it safe. A checker can execute untrusted project build instructions. A protected profile must use a restricted subprocess runner with declared read/write/network constraints. When the chosen driver cannot enforce those constraints, return `UNSUPPORTED_CAPABILITY`, not `CHECKER_SAFE`.

### 7.4 Recovery

A durable intent journal must distinguish **prepared**, **commit started**, **commit observed**, and **postcondition verified**. After a crash, inspect target and staging identities before deciding whether to retry, compensate, or mark `UNKNOWN_EFFECT`. Automatic replay of a non-idempotent edit or external command is forbidden.

## 8. Bounded subprocess execution

The minimal command contract is argument-array based. Do not implicitly invoke a shell. `exec`-like behavior still executes arbitrary code with available user rights; argv form only addresses one particular injection mechanism.

### `CommandSpec`

```text
executable_ref        pinned or policy-approved executable identity
argv                  exact list of typed arguments
working_scope         approved directory object
environment           explicit allowlist; do not inherit by default
stdin                 bounded stream or closed
stdout/stderr         capped capture, truncation indicated
deadline              monotonic elapsed-time requirement
cpu/memory/pids       declared resource budgets, only if driver can enforce
isolation             selected kernel-enforced profile or NONE
cleanup               process/group/descendant guarantees
idempotency           NONE, SAFE_RETRY, or EXPLICIT_RECONCILIATION
```

Gerbil's documented `:std/misc/process` `run-process` is an argv-based synchronous helper, and by default may inherit environment and working directory. The **documented wrapper is not proof** of hard deadlines, process-tree cleanup, container isolation, or cgroup enforcement. Such guarantees need a separately qualified driver.

### Distinct outcomes

`SPAWN_FAILED`, `EXIT_ZERO`, `EXIT_NONZERO`, `EXIT_SIGNALLED`, `TIMED_OUT`, `CANCELLED`, `OUTPUT_TRUNCATED`, `DESCENDANTS_UNVERIFIED`, `REAP_UNCONFIRMED`, `UNKNOWN_EFFECT`. Output truncation is orthogonal to execution exit status.

### Deadline and cleanup

Use monotonic elapsed time for deadlines. Silence/inactivity timeout is separate from hard wall duration. A continuously printing child must still time out when its absolute budget expires. Sending SIGKILL to a single PID does not establish that all descendants terminated. A process-group or cgroup-based plan must document escape limits, session changes, reaping and identity reuse; refuse guarantees that cannot be verified.

P02-D02 owns a **bounded command**, not a long-lived system service. P02-D01-B remains the separate source-derived design for durable local service supervision.

## 9. Tool execution and request lifecycle

```text
RECEIVED
 -> NORMALIZED
 -> VALIDATED
 -> POLICY_CHECKED
 -> {DENIED | APPROVAL_REQUIRED | AUTHORIZED}
 -> PREPARED
 -> {READY | UNSUPPORTED | STALE}
 -> RUNNING
 -> {SUCCESS | FAILED | CANCELLED | TIMED_OUT | PARTIAL | UNKNOWN}
 -> RECEIPT_RECORDED
```

Rules:

- `AUTHORIZATION_REQUIRED` cannot be skipped by agent reasoning or textual assertions.
- `PREPARED` checks prerequisites; no implied future success.
- `SUCCESS` requires the operation-specific postcondition, not merely no exception.
- Any timeout after a side effect begins may be `PARTIAL_EFFECT` or `UNKNOWN_EFFECT`, not simply `TIMED_OUT` with implied cleanup.
- An operation cannot be duplicated on retry unless its idempotency contract permits it, or reconciliation proves it safe.
- The executor enforces quotas *before* each effect, not only at request submission.

### Repetition and progress

Use a **request fingerprint** plus operation-specific semantics. Repeated `READ_SCOPED` after a version change and explicit `WAIT_POLL` after a scheduled delay may be legitimate. Identical effects with unchanged state and no progress may be stopped after a bounded number of attempts.

Budgets include tool invocations, model iterations (only if adapter uses a model), cumulative output bytes, elapsed time, cost-estimate hooks, simultaneous operations, and mutation count. No model-supplied text may increase budgets.

## 10. Agent inputs, untrusted content and optional LLM adapter

The model adapter is deliberately a *translator*, not a policy authority.

1. Parse structured tool calls through an allowlisted schema.
2. Treat model-generated descriptions and repository documents as **untrusted evidence**.
3. Never parse tool output as instructions to grant new permissions.
4. Only P02-D02-A's configured owner policy and approved user actions can authorize an effect.
5. Do not automatically forward secrets or sensitive tool output to a model provider; transmission is itself a separately declared external effect.
6. Return typed error/status fields, not solely prose, to the calling agent.
7. The model may propose *why* a tool is needed, but its rationale does not change effect class.
8. A caller wanting code evaluation must request a distinct, high-risk tool with an OS isolation profile. It is **disabled by default**.

An application without an LLM can issue the same `ToolRequest` records. P02-D02 consequently remains useful as a local bounded tool governor.

## 11. Evidence, receipts, audit and privacy

Every execution attempt records:

- `request_id`, owner context, tool and schema versions, canonical args digest;
- `policy_revision`, decision status, approval commitment ID (not secret);
- workspace root identity and target revisions;
- driver ID, declared and realized capabilities;
- start/end monotonic timestamps for durations; wall-clock timestamps only for human timeline;
- observation set, exit status or signal, checker outcome and postcondition;
- set of **known committed** effects and **uncertain** effects;
- cleanup/recovery status, output bytes and truncation flags;
- sanitized diagnostic references, policy decision rationale and escalation reason.

The **receipt itself is not proof** that external side effects have been undone, or that an OS sandbox contained hostile code. Append-only tamper-evident storage, remote attestation and cross-process audit persistence are optional future concerns and must not be claimed in D01.

Do not log full environment variables, authentication tokens, complete model prompts, arbitrary private file contents or unchecked command outputs by default. Redaction is not merely UI masking; sensitive bytes should not be written into an unrestricted log in the first place.

### Final result precedence

A stable final verdict is derived from **current run state and verified postconditions**, not searches for `FAILED` in historical transcript text. Example: `fail → correction → validation PASS → commit PASS` can produce final `SUCCESS`, with the earlier failed attempt still retained as history. Conversely, a successful-looking message with a failed postcondition cannot become `SUCCESS`.

## 12. Security and ownership boundaries

| Boundary | P02-D02 owns | P02-D02 does not claim |
|---|---|---|
| Application-level approval | Exact request/effect/policy binding | Proof of malicious-code containment |
| Scoped file API | Fail-closed, driver-qualified resolution of approved workspace | Global protection against same-UID code accessing other files |
| Staged edits | Check-before-commit and exact-base compare | Cross-file atomicity or rollback of arbitrary external effects |
| Process call | Structured request, deadlines and driver results | Linux isolation without kernel primitives |
| LLM adapter | Parse untrusted proposals | Authority derived from model role or textual instructions |
| Logs/receipts | Accurate typed outcomes and uncertainties | A full forensic or tamper-proof audit system |
| Cross-edition contracts | Local P02-D02 mapping for identity/effect/result | Ownership of Meta-Foundation or P01-D01–P03-D01 internals |

A user-level Gerbil executable running on GNU Guix may be *packaged* by Guix, but P02-D02 requires no Guile Scheme, Guix service definitions, Shepherd process manager or `guix-daemon` at runtime. Any such integration belongs to a future consumer.

## 13. Proposed API surface (behavioral pseudocode, not executable Gerbil)

```text
register_tool(ToolSpec)                            -> RegistryResult
normalize_request(raw_request, caller)              -> RequestResult
validate_request(ToolRequest, Registry)             -> ValidationResult
classify_effects(ToolRequest, ToolSpec)              -> EffectSummary
decide(PolicySpec, CallerContext, ToolRequest)       -> PolicyDecision
prepare_approval(ToolRequest, PolicyDecision)        -> ApprovalCommitment
record_approval(commitment, approver_evidence)       -> ApprovalResult
verify_authorization(request, decision, approval)    -> AuthorizedEnvelope | Refusal

propose_edit(workspace, target, bytes, base_version)  -> ChangeSet
validate_staged(ChangeSet, CheckSpec, driver)          -> CheckResult
commit_approved(ChangeSet, AuthorizedEnvelope, driver)-> OperationResult
read_scoped(workspace, target, envelope, driver)      -> OperationResult
invoke_bounded(CommandSpec, AuthorizedEnvelope, driver)-> OperationResult
reconcile(receipt, observed_state)                    -> ReconcileResult
finalize_run(attempts, verified_postconditions)       -> RunVerdict
```

`verify_authorization` is a real mandatory boundary *inside the library*, but not a privilege barrier against other malicious code running as the same OS identity.

## 14. Three vertical slices to qualify before adding more tools

### Slice A — Deterministic policy engine (pure)

A mock caller requests `read`, `propose_edit`, and `run_command`. A constrained policy allows only selected reads. Assertions check exact refusal reasons, effect tags, canonical digests, stale/expired approvals and no import of effectful libraries.

**Pass:** all pure policy scenarios satisfy deterministic oracles. **Not** evidence of filesystem or OS security.

### Slice B — One-file staged edit (mock filesystem first)

Propose an edit, calculate diff, validate staged bytes, approve exact digest, recheck base, commit through simulated driver. Inject failure after each checkpoint, mutate the base between approval and commit, and test recovery from an ambiguous commit.

**Pass:** a failed validation does not touch the target in the declared atomic single-file profile; partial/unknown outcome is preserved.

### Slice C — Bounded command execution (real OS only after mock)

Use an allowlisted executable with explicit argv/environment, run one success, one nonzero exit, a continuous-output infinite loop, a subprocess that spawns children, and cancellation during cleanup.

**Pass:** measured kernel behavior meets the declared driver capability profile. No claim of descendant cleanup is accepted without corresponding observations.

## 15. Qualification test matrix: 44 concrete scenarios

No tests listed here have been run. IDs are stable design targets; pass/fail is pending.

| ID | Scenario | Expected contract |
|---|---|---|
| P02-D02-T01 | Build P02-D02-A without LLM, network, process or file-write dependencies | Pure profile isolation |
| P02-D02-T02 | Unregistered tool | `REQUEST_INVALID` or `POLICY_DENIED`, never dispatch |
| P02-D02-T03 | Wrong tool-version/schema | Fail before effect |
| P02-D02-T04 | Oversized/deeply nested request | Bounded parse refusal |
| P02-D02-T05 | Tool descriptor mislabels an executable as read-only | Registry rejects/driver refuses mismatch |
| P02-D02-T06 | Caller inserts `approved=true` in args | Ignored as non-authoritative |
| P02-D02-T07 | Model impersonates human in prose | No elevation |
| P02-D02-T08 | Policy deny overrides all model arguments | Deterministic deny |
| P02-D02-T09 | Change policy after approval | Approval stale |
| P02-D02-T10 | Change tool version after approval | Approval stale |
| P02-D02-T11 | Change target content after approval | `APPROVAL_STALE` |
| P02-D02-T12 | Change command args after approval | New approval required |
| P02-D02-T13 | Expire or reuse approval nonce | Refuse |
| P02-D02-T14 | User rejects change | No effect; typed denial |
| P02-D02-T15 | Noninteractive EOF at approval | Fail closed |
| P02-D02-T16 | `../` path traversal | Refuse before opening outside |
| P02-D02-T17 | Absolute path outside workspace | Refuse |
| P02-D02-T18 | Sibling-prefix `/proj` vs `/proj-copy` | Refuse |
| P02-D02-T19 | Symlink to outside | Refuse in strict profile |
| P02-D02-T20 | Parent-dir symlink/rename race | No escaped read/write under qualified OS driver |
| P02-D02-T21 | Magic-link path such as `/proc` indirection | Refuse when policy forbids |
| P02-D02-T22 | Hard-link alias / mount boundary | Explicit documented behavior, not silent security claim |
| P02-D02-T23 | Concurrent reader/writer changing target | Stale-base or bounded conflict result |
| P02-D02-T24 | New file absent vs unreadable target | Distinct errors, no destructive fallback |
| P02-D02-T25 | Pure edit proposal | No target mutation |
| P02-D02-T26 | Checker failure before commit | Target unchanged in atomic profile |
| P02-D02-T27 | Checker writes outside staging sandbox | Blocked by qualified isolation or capability unsupported |
| P02-D02-T28 | Approval digest altered after display | Commit refused |
| P02-D02-T29 | Atomic single-file commit and digest verification | `COMMITTED` with correct observed bytes |
| P02-D02-T30 | Failure during commit | `PARTIAL_EFFECT`/`UNKNOWN_EFFECT` and recovery evidence |
| P02-D02-T31 | Multi-file edit under atomic-only policy | Refused, no false transaction |
| P02-D02-T32 | Crash-restart during journal update | No unsafe automatic replay |
| P02-D02-T33 | Shell metacharacters embedded in argv | Passed as literals; no implicit shell |
| P02-D02-T34 | Executable path changed after admission | Driver detects/refuses stale executable identity |
| P02-D02-T35 | Inherited environment contains secret | Excluded by default allowlist |
| P02-D02-T36 | Nonzero process exit | Structured `EXIT_NONZERO`, not success |
| P02-D02-T37 | Infinite child continuously prints output | Hard deadline still enforced if claimed |
| P02-D02-T38 | Output exceeds capture limit | Truncation indicated independently from exit |
| P02-D02-T39 | Child creates grandchildren | Real-driver cleanup claim verified or refused |
| P02-D02-T40 | Cancel after effect begins | Record known and uncertain effects |
| P02-D02-T41 | `fail → repair → pass` | Final current-run success, earlier failure retained |
| P02-D02-T42 | Harmless output contains word `FAILED` | Does not corrupt final result |
| P02-D02-T43 | Legitimate repeated polling with changing state | Not falsely classified as infinite loop |
| P02-D02-T44 | Repeated non-idempotent invocation after lost ack | Refuse retry or require reconciliation |

## 16. Qualification gates

| Gate | Meaning | Evidence needed |
|---|---|---|
| **Q0 — Provenance** | Legal/source lineage known | Source SHA, license assessment, review findings, no code-derivation ambiguity |
| **Q1 — Pure policy** | Tool/identity/schema/effect/approval contracts hold | T01–T15, property tests and fuzzed malformed inputs |
| **Q2 — Filesystem model** | Path and file identity semantics correct in mock | T16–T26, including races modeled explicitly |
| **Q3 — Staged edits** | No check-after-write ambiguity | T25–T32, crash/fault injection and commit journal assertions |
| **Q4 — Mock execution** | Bounded command protocol and typed outcomes | Fake clock, mock process trees, quotas, cancellation, retry rules |
| **Q5 — Real OS driver** | Actual selected kernel capabilities qualified | T17–T24 and T33–T40 on documented Linux target |
| **Q6 — End-to-end agent use** | Model adapter cannot create authority; final statuses correct | T06–T15 and T41–T44 with deterministic fake model |
| **Q7 — Independent review** | Negative tests and dependency isolation independently verified | Reproducible results, adverse findings, unresolved driver limitations |

**Gate separation:** Q1 may pass while Q5 remains `UNQUALIFIED`. Q5 is not established by fake process tests. No gate is passed merely because an interface has been described.

## 17. Proposed implementation sequence, if separately authorized

1. **P02-D02-00:** Lock source evidence, legal provenance and test oracle definitions. Avoid wholesale AGPL source import.
2. **P02-D02-01:** Build P02-D02-A schema, tool registry, request normalization and effect classification.
3. **P02-D02-02:** Implement deterministic policy and exact approval binding with pure tests.
4. **P02-D02-03:** Add mock filesystem, staged-change workflow and failure-injection checks.
5. **P02-D02-04:** Qualify one bounded read/edit path; explicitly distinguish staging from actual target writes.
6. **P02-D02-05:** Add mock command driver, deadlines, retry policy and structured results.
7. **P02-D02-06:** Investigate real Gerbil/Gambit/Linux capabilities. Implement only those OS guarantees that can be independently observed.
8. **P02-D02-07:** Add a deterministic fake-model request translator as an *optional* adapter. A live cloud model is unnecessary for acceptance.
9. **P02-D02-Q01:** Independent review and qualification report. Publish PASS/REJECTED/UNQUALIFIED per feature, not a blanket safe label.

P02-D02 design approval does not authorize P01-D01–P03-D01 changes, P04-D01 design, Meta-Foundation implementation, repository creation, Guix reconfiguration, deployment, or source code changes. Documentation publication has been separately approved.

## 18. Open decisions and engineering uncertainties

1. **Workspace trust class:** trusted owner-only repository vs untrusted checkout; the latter requires stronger kernel isolation for checkers and commands.
2. **Filesystem strictness:** whether the first Linux driver forbids all symlinks and mount crossing or supports a tested relaxed mode.
3. **Multi-file semantics:** default is no atomicity guarantee. Choose between fail-closed atomic-only and journaled partial-commit mode.
4. **Operating-system isolation:** identify minimal kernel primitives, identity/UID boundary, namespace/cgroup assumptions and Guix availability only when building a real driver.
5. **Approval lifetime:** persistent approvals after restart require a trustworthy replay/expiry model; default is nonpersistent, single-use.
6. **Edit checker trust:** pure syntax checking vs executing arbitrary build scripts; the latter is high-risk.
7. **Model adapter:** no provider binding until the pure policy and local executor tests pass.
8. **Logging privacy:** retention, redaction and encryption rules belong to the consuming application's deployment policy.
9. **Process-tree guarantee:** whether a driver can credibly contain detached descendants; fail closed for strict cleanup requirements.
10. **Source licensing:** Coding Harness source is AGPL-3.0 according to file headers. Avoid copying implementation until distribution and derivative obligations are explicitly resolved.
11. **Cross-edition abstraction:** P02-D02's identity/effect/result records may someday inform Meta-Foundation, but promotion requires independent cross-domain evidence, not name matching.
12. **Durability:** atomic rename alone is not power-loss durability; filesystem-specific evidence may remain unresolved.

## 19. Final verdict

`P04_D01_DETAILED_DESIGN_PROPOSED / CODING_HARNESS_LINEAGE_EXPLICIT / PURE_AUTHORITY_AND_OPTIONAL_EXECUTION / GERBIL_NATIVE / MOCK_FIRST / SECURITY_CEILING_EXPLICIT / APPROVAL_BOUND_TO_EXACT_REQUEST / NO_AUTO_EVAL / FILE_AND_PROCESS_DRIVERS_UNQUALIFIED / P01_P02_P03_UNTOUCHED / DOCUMENT_PUBLISHED`

P02-D02 preserves the Coding Harness's strongest practical idea: **an agent can work through a registered, inspectable tool protocol rather than unconstrained opaque edits**. It corrects the boundaries that source inspection exposed: directory scope, authorization binding, check-before-commit, structured outcomes, hard deadlines and nonproductive loops. This is an independently proposed Gerbil specialized edition, not a source copy, production guarantee, or general security sandbox.

---

## Source anchors

1. Gerbil Coding Harness at immutable revision: https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness
2. Coding Harness existing audit: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects/06-coding-harness-gerbil-source-code-review.md
3. Coding Harness tool entry points: https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/tools.ss
4. Coding Harness approval module: https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/approval.ss
5. Coding Harness checker: https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/checker.ss
6. Coding Harness process helper: https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/util.ss
7. Coding Harness exit semantics: https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/harness.ss
8. Coding Harness agent loop: https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/llm.ss
9. Gerbil process utilities: https://gerbil.scheme.org/reference/std/misc/process.html
10. Linux path-resolution interface: https://man7.org/linux/man-pages/man2/openat2.2.html
11. LFR-C02 approved concept: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/concept/LFR-C02-source-lineage-gerbil-reserve-concept-2026-11-12.md
12. Existing P02-D01 sibling design: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/design/LFR-P02-D01-declarative-systems-process-supervision-detailed-design-2026-11-12.md