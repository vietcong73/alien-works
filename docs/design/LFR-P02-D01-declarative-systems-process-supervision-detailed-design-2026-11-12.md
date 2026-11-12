# LFR-P02-D01 | Declarative Systems & Process Supervision Reserve

**Document class:** Detailed Design Proposal — specialized edition P02 only  
**Date:** 12 November 2026  
**Status:** `DETAILED_DESIGN_PROPOSED / STATIC_SOURCE_GROUNDED / EXECUTION_NOT_QUALIFIED`  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Target implementation language:** Gerbil Scheme, using Gambit runtime mechanisms only where independently qualified  
**Primary source lineage:** GNU Guix / GNU Guile Scheme, GNU Shepherd  
**Background cross-check:** Alien-Works lifecycle design and Gerbil Coding Harness process-invocation review  
**Document boundary:** P02 alone; **not** P03, the Gerbil Meta-Foundation, Guix itself, or a full system manager  
**Publication:** Owner-approved documentation publication on 12 November 2026 in `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`, at `docs/design/`. This publication does not authorize implementation, source-code changes, runtime commands, or Guix reconfiguration.

## 0. Decision statement

Design **one independently versioned Gerbil specialized edition** with two separable profiles:

- **P02-A, Declarative Systems (primary / pure):** describes objects and dependencies; validates and lowers declarations into normalized plans; models execution stages, effects, versions, evidence and discrepancies without executing arbitrary code. It is not a package manager, general build system, or process scheduler.
- **P02-B, Local Process Supervision (optional / effectful):** manages specifically registered local child processes, service dependencies, start/readiness/stop, bounded restart behavior, shutdown and health observations through an explicitly qualified OS-process backend. It is not PID 1, an actor runtime, a system-wide init replacement, or a Guix/Shepherd bridge.

Both are *new Gerbil designs informed by source behavior*, **not** ports or claims of semantic equivalence. No Guile runtime, Guix daemon, Shepherd daemon, GNU store, Guix Home, GNU package graph, or G-expression interpreter belongs on their mandatory dependency path.

### Product purpose

P02 is useful when an application must first describe what should exist, reason about dependent components, plan an ordered change, and optionally run long-lived local child services. Consumers may use P02-A alone for pure dependency composition or use P02-B for local supervision. Each use has explicit outcomes, rather than optimistic calls that declare success merely because a command was started.

### Problem P02 is solving

Ad hoc applications frequently conflate *configuration declarations*, *execution order*, *process liveness*, *readiness*, *authorization*, *observed state* and *rollback*. That makes partial failure hard to diagnose and small workflows accidentally dependent on one host's environment. Guix and Shepherd demonstrate important but distinct remedies. P02 extracts only the transferable mechanisms and specifies their limits.

## 1. Primary source-lineage / defect-resolution ledger

| ID | Source evidence and original problem solved | What is strong | Boundary / failure mode | P02 design response | Evidence needed later |
|---|---|---|---|---|---|
| **G01** | Guix `package->bag`: transforms symbolic packages into explicit build inputs and outputs [source](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm#L1812-L1855) | Staged description-to-IR lowering | Guix bag is specific to its store and build systems | `Declaration -> NormalizedGraph -> EffectPlan`; typed, serializable, inspectable stages | Deterministic lowering and unchanged input descriptor |
| **G02** | Guix G-expressions / `lower-gexp`: prepares code and referenced dependencies for later execution [source](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/gexp.scm#L1120-L1172) | Explicit host/build staging and reference capture | Escapes and ambient-host values may leak between stages; staged errors are difficult to localize | Data-only **StageSpec** and explicit input manifest; no arbitrary host/target code evaluation | Reject unresolved reference, ambient input, or executable code in descriptors |
| **G03** | Guix `package-closure` walks transitive dependencies [source](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm#L1537-L1557) | Traversal with deduplication | A runtime dependency graph is not a build closure or universal job plan | Separate **build/reference graph** from **service readiness graph** and **effect-plan graph** | Stable topological ordering, cycles, isolated and diamond graph properties |
| **G04** | Guix `<service-type>` extensions, compose and extend [source](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/services.scm#L187-L211) | Composable declarations with targeted configuration changes | Effective configuration can be obscured across many contributing sources | Typed **Contribution** ledger with source locations and deterministic folding | Every output field traceable to contributing declarations |
| **G05** | Guix `service-back-edges` and `fold-services` [source](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/services.scm#L1319-L1459) | Missing/ambiguous targets are explicit errors; extension folding | Complex multi-stage graph semantics; not a daemon supervisor | Reject ambiguous/missing providers, cycles and illegal composition before effects | Counterexamples for missing, ambiguous and cyclic extensions |
| **G06** | Guix profile construction and switching [source](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/package.scm#L140-L179) | Versioned, provenance-rich desired artifacts | Switching a generation does **not** roll back mutable databases, external files or running services | Immutable declaration revisions, explicit before/after observations and `PARTIAL_EFFECT` outcomes | Failure-injection demonstrates no fictitious atomic rollback |
| **S01** | Shepherd service provisions and requirements [manual](https://www.gnu.org/software/shepherd/manual/html_node/) | Managed dependency-aware service start | Service dependency is not the same as artifact dependency | P02-B's separate runtime service registry and readiness conditions | Dependency readiness and reverse shutdown tests |
| **S02** | Shepherd restart delay/limit and disable behavior [manual](https://www.gnu.org/software/shepherd/manual/html_node/) | Restart-loop containment | Crashes and respawn do not repair external state; restart storms can persist without limits | Monotonic backoff and bounded restart window; explicit `DISABLED` state | Crash-loop model, fake clock, real child exits |
| **S03** | Shepherd one-shot vs long-running services [manual](https://www.gnu.org/software/shepherd/manual/html_node/) | Different service completion semantics | One-shot success cannot be inferred from long-running process health | Tagged `ONESHOT` vs `DAEMON` declarations with distinct success rules | Exit/readiness distinctions and failure propagation |
| **L01** | Gerbil `:std/misc/process` `run-process` provides argv-based synchronous child invocation [reference](https://gerbil.scheme.org/reference/std/misc/process.html) | Concrete process integration point | Synchronous helper **does not by itself qualify** persistent service supervision, process-tree death or hard deadlines | Separate `ProcessDriver` interface; OS backend admitted only after spawn/signal/wait/reap/PGID qualification | Real kernel process, child-tree, timeout and PID-reuse tests |
| **L02** | Gerbil actors/threads provide in-process messaging [reference](https://gerbil.scheme.org/reference/std/actor.html) | Useful coordination primitive | Actor monitor ≠ PID supervisor; same-process actor administration ≠ security boundary | P02-B may use actors internally, but owns OS-process lifecycle separately | Death-of-actor vs death-of-child non-equivalence test |
| **A01** | Alien-Works uses scoped application native lifecycle and task queues [source](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/framework/framework.lisp) | Clear acquire/use/release composition | Native renderer tasks are not durable process supervision | Explicit separate lifecycle of declarations, child services, and native resources | No importing graphics packages to build P02-A |
| **H01** | Gerbil Coding Harness `open-process` execution audit [review](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects/06-coding-harness-gerbil-source-code-review.md) | argv dispatch avoids accidental shell composition | Idle timeout ≠ wall-clock deadline; signaling one PID ≠ descendant cleanup | Structured hard deadline with process-group handling when supported; test fallback refusal | Continuous-output and child-spawning adversarial fixtures |

**Source assurance ceiling:** Guix/Gerbil entries are pinned code or official API references, Shepherd rows are official manual behavior rather than a fresh independent source audit, and all target P02 behavior is still proposed. Guix Home and Shepherd implementation internals have **not** been audited to the same depth as Guix package/System in the existing dossier. Do not silently upgrade evidence levels.

### A precision correction to earlier drafts

Guix service *extensions* are about composing configuration/outputs, whereas Shepherd *requirements* determine runtime start dependencies. Identical graph terminology does not make them the same graph. P02 treats them as different graph kinds, with an explicit lowering relation only when an application supplies one. Likewise, Gerbil actor lifecycle cannot be treated as a substitute for OS process supervision.

## 2. Module boundary and dependency design

```text
P02-A: PURE DECLARATIVE PROFILE (mandatory only if this edition is used)
  p02/schema   p02/identity   p02/graph   p02/compose
      |               |            |           |
      +---------------+------------+-----------+
                          |
                     p02/stage
                          |
                   p02/plan / diff
                          |
                 p02/receipt-model
                          |
               pure mock/evidence tests

P02-B: OPTIONAL LOCAL PROCESS PROFILE
  p02/service-spec  --> p02/registry --> p02/supervisor-state
                             |                    |
                       p02/health           p02/restart-policy
                             \                    /
                              p02/process-driver
                                     |
                    mock driver (first) / Linux driver (later)
                                     |
                             owned child processes
```

**Dependency invariants:**

1. P02-A can load, validate, lower, topologically sort, diff and produce dry-run receipts without P02-B, native process execution, Guile, Guix or Shepherd.
2. P02-B imports P02-A's declaration/result types but P02-A never imports P02-B.
3. P02-B may operate through an injected fake process driver and fake monotonic clock. A real Linux driver is an independently qualified optional backend.
4. Neither profile imports P01; future interoperability occurs through agreed descriptors, not cross-edition source dependency.
5. The future LFR Meta-Foundation may later *accept* promoted cross-domain contracts, but P02 does not define global K0 or force other editions to adopt its graph ontology.

### Proposed source layout (not created)

```text
editions/declarative-systems-processes/          # P02 standalone future repository/package
  README.md
  gerbil.pkg
  build.ss
  lineage/
    upstream-revisions.lock
    mechanism-to-requirement.md
    license-and-derivation.md
  src/p02/
    core/{identity,schema,result}.ss
    graph/{model,validate,toposort,provenance}.ss
    compose/{registry,contribution,fold}.ss
    stage/{description,lower,inputs}.ss
    plan/{diff,effect-plan,receipt}.ss
    service/{declaration,registry,health,dependencies}.ss
    supervisor/{state,restart,clock,controller}.ss
    driver/{interface,mock}.ss
    driver/linux/            # independent later qualification
  tests/
    pure/
    property/
    fault/
    integration/            # opt-in kernel subprocess tests
    fixtures/
  docs/
    contract-v1.md
    qualification.md
    known-limitations.md
```

The layout names are **proposed module responsibilities**, not claims about Gerbil module availability, package compilation or specific APIs being exercised. No source code exists under this plan yet.

## 3. Contract model: declaration, dependency and provenance

### 3.1 Principal pure records

| Contract | Required fields | Semantics |
|---|---|---|
| `DeclarationRef` | `{id, schema_version, revision, origin_hash}` | Stable identity of an input declaration; content-addressed digest is an evidence field, not an execution permission |
| `ComponentSpec` | `{ref, kind, inputs, outputs, parameters, effects, constraints}` | One declared component, not a running process by default |
| `DependencyRef` | `{from, to, relation, required, version_constraint}` | Edges tagged `INPUT`, `EXTENDS`, `BEFORE`, `REQUIRES_READY`; kind-check edges before sorting |
| `ContributionSpec` | `{source, target, field_path, operation, value, priority?}` | Typed contribution to a declared target; provenance retained |
| `ResolvedGraph` | `{nodes, typed_edges, order, input_revisions, hash, validation_report}` | Deterministic, immutable output of graph resolution |
| `StageSpec` | `{stage_id, inputs, outputs, executor_type, declared_effects, env_refs}` | Data-only description of a possible later action |
| `EffectPlan` | `{spec_revision, steps, constraints, expected_observations, auth_ref?}` | Describes effectful work without performing it |
| `PlanReceipt` | `{plan_ref, status, observations, completed_steps, pending_steps, uncertainties, causal_links}` | Makes partial execution visible, never fabricates rollback |
| `StageArtifact` | `{type, schema, digest, origin, created_by_stage}` | Output reference with provenance and a declared verification status |

All public formats need stable tagged encoding, explicit type/version discriminators, finite-size limits and canonical serialization rules. **No executable closure, procedure value, raw Scheme expression, foreign pointer or opaque OS handle is accepted in serialized descriptors.** Secrets are references to an external credential mechanism, not values serialized into provenance.

### 3.2 Graph kinds and their differing meaning

- **Reference/input DAG:** A node may depend on another declaration/artifact. It establishes ordering or inclusion only. Edges do not start processes.
- **Extension/composition DAG:** Values contribute to selected targets through declared combiners. Missing/ambiguous target resolution is a validation error.
- **Effect-plan DAG:** Ordered intents and compensability metadata. Approval of one step is not approval of all descendants unless scope states so.
- **Runtime-service graph (P02-B):** Depends on the **readiness/success** of other services, not simply existence of graph nodes.

No implicit conversion between graph kinds. A consumer must declare a mapping and prove its preconditions. Store-derived graph contents are never assumed to be automatically executable.

### 3.3 Determinism and canonicalization

- Stable IDs and input revisions are required; duplicate IDs with conflicting definitions fail before composition.
- Sort independent nodes by a canonical normalized identity, **not** hash-table iteration order.
- Disallow cycles on ordering edges. For explicit cycles in pure references, retain diagnostic cycle path; do not silently topologically order an invalid DAG.
- Extension combiners specify associativity/commutativity/order dependence. A noncommutative combiner must declare and preserve a total order; no arbitrary merge.
- Hash the canonical IR and declared input closure, using a versioned algorithm and encoding. The digest means *description integrity*, **not** build reproducibility, completion or safety.
- Track the provenance chain `declared-field -> contribution(s) -> effective-field -> plan-step -> observation`.
- Reject host-dependent values unless represented as explicit declared inputs. Examples: unresolved `$PATH`, current locale, platform default, implicit user home, ambient network endpoint or clock value.

### 3.4 Pure API sketch (behavioral, not real Gerbil syntax)

```text
parse_declarations(data)             -> ValidatedInput | DecodeError
validate_graph(specs)               -> ValidGraph | GraphErrors
resolve_extensions(valid_graph)     -> ComposedGraph | CompositionErrors
lower_description(composed_graph)   -> NormalizedDescription | LoweringErrors
plan_effects(description, observed) -> EffectPlan | NoChange | PlanError
explain_field(description, path)    -> ProvenanceTrace | UnknownField
compare_versions(old, new)          -> GraphDiff | CompatibilityError
make_dry_run_receipt(plan)          -> Receipt(status=NOT_EXECUTED)
```

All operations here are pure in the architectural sense: no OS process spawn, package installation, filesystem mutation or network request. Reading an external observed-state snapshot is an effectful boundary and must occur **outside** pure planning.

## 4. Staging semantics: the Guix lesson without a Guix interpreter

### 4.1 Explicit stages

```text
AUTHORING INPUT
      |
      v
DECODED -> VALIDATED -> COMPOSED -> LOWERED -> PLANNED
                                              |
                                              v
                                        DRY-RUN REPORT
                                              |
                                  optional external approval
                                              |
                                              v
                                           PREPARED
                                              |
                                      effectful adapter
                                              |
                             +----------------+------------------+
                             v                v                  v
                           APPLIED         PARTIAL_FAILED       UNKNOWN
```

- `VALIDATED` verifies declared schemas, references, graph kinds and purity of the *description representation*.
- `COMPOSED` resolves extension targets and contributions into final values, with source provenance.
- `LOWERED` creates normalized machine-inspectable IR, not generated Gerbil/GNU Guile executable code.
- `PLANNED` records intended steps and their dependencies, declared effects and observable postconditions.
- `PREPARED` checks prerequisites, authority and adapter support. It is not a guarantee of later success.
- `APPLIED` requires observed postconditions, not just a successful command dispatch.
- `PARTIAL_FAILED` means earlier effects occurred and later effects failed. `UNKNOWN` means reconciliation cannot establish what occurred.

### 4.2 The execution boundary

- P02-A is incapable of execution by design; its output is inert data.
- A concrete executor consumes `EffectPlan` **only after** a separate authority decision by the owning application. This design does **not** implement a global authorization framework.
- Every effectful step declares target identity, required owner, preconditions, timeout, retry/idempotency policy, expected output/health checks, and side-effect class.
- An executor must refuse unknown effect types and incompatible adapter versions; it must not fall back to `(eval ...)`, `sh -c`, or textual interpolation.
- A non-idempotent step must not automatically be retried after ambiguous failure or lost acknowledgement.
- `compensable` means *a conditional inverse is known*, not a guaranteed atomic rollback.

### 4.3 Why this is **not** reimplementing Guix

P02 does not contain: derivation hash compatibility, `/gnu/store`, substitute verification, package build isolation, GC roots, system profiles, package naming semantics, Guix channels, bootloader activation, Guix daemon protocol or reproducible-build certification. A future consumer may use Guix externally, but P02 itself neither requires nor controls Guix.

## 5. Composition and extension semantics

### 5.1 Registration

A `CompositionRegistry` binds a type name and schema version to an allowed extension schema and deterministic merge policy. It is explicit and local to a plan; no unconstrained global registry that changes meaning midway through a run.

### 5.2 Composition algorithm

1. Parse components and extension declarations into immutable tagged records.
2. Resolve *exactly one* target for each extension (or documented explicit multi-target semantics). Missing or ambiguous target is a structured error.
3. Check field-path compatibility, required target fields, version constraints and source provenance.
4. Detect cycles and assign a stable ordering where needed.
5. Fold contributions using declared type-aware combiners; reject conflict instead of silently using last writer.
6. Construct an effective specification and `ContributionTrace` for every changed field.
7. Validate the effective graph again; lowering cannot silently bypass a post-compose validation failure.

### 5.3 Conflicts

`CONFLICTING_WRITES`, `AMBIGUOUS_TARGET`, `MISSING_TARGET`, `ILLEGAL_EXTENSION`, `CYCLE`, `UNSTABLE_ORDER`, `VERSION_MISMATCH`, and `UNDECLARED_INPUT` are distinct errors. A contributor's declared priority cannot grant permission to override a field that the owner marked immutable.

### 5.4 Graph complexity and limits

Compute expected complexity of resolution as `O(V+E)` for ordinary traversal/topological checks, plus documented complexity of the chosen combiners. Enforce limits on graph nodes, edges, depth, total descriptor bytes and recursion. Property tests cover graph diamonds, duplicate edges, stable traversal and adversarial cycle graphs. No hard real-time guarantee follows from Big-O alone.

## 6. P02-B: optional local OS-process supervision

### 6.1 Why it exists separately

Shepherd handles *running operating-system services*. Gerbil actor handles and in-process queues have different ownership, failure and restart semantics. P02-B will manage **only processes it starts and owns** (or those explicitly adopted under a separate, narrowly specified ownership protocol). A process PID cannot be treated as an unforgeable, eternal identity. The supervisor must record a process identity/generation and refuse stale handles.

### 6.2 Process and service contracts

| Contract | Main fields | Key rule |
|---|---|---|
| `ServiceSpec` | `{service_id, kind, argv, cwd, env_allowlist, requires, readiness, restart, stop, limits, owner}` | No free-form shell command; `kind` is `ONESHOT` or `DAEMON` |
| `ProcessIdentity` | `{service_id, generation, process_ref, pid?, started_at_monotonic, driver_ref}` | PID alone is not identity; generation binding required |
| `ReadinessSpec` | `{predicate_type, timeout, check_interval, evidence}` | Spawn != ready; supported checks depend on available driver |
| `RestartPolicy` | `{mode, max_attempts, window, delay, backoff_cap, jitter_policy}` | Mandatory bounded restart policy for automatic restarts |
| `StopPolicy` | `{signal_or_request, grace_deadline, escalate?, descendant_policy}` | Child groups must be handled or limitation declared |
| `ServiceReceipt` | `{service_ref, generation, state, exit_kind, stop_reason, health, completed_actions, gaps}` | No success without observed exit/readiness as applicable |
| `ProcessDriver` | `{spawn, poll, signal, reap, close, capabilities}` | Backend capability negotiation is required |

Environment must be an explicit mapping or tightly controlled inheritance policy; ambient PATH lookup is discouraged for executable identity. Credentials must not be logged in argv, env dumps, raw output or receipts. A service may run under the same OS account as the supervisor: **P02 does not claim sandboxing or privilege separation**.

### 6.3 Service lifecycle state machine

```text
DECLARED -> VALIDATED -> WAITING_DEPENDENCIES -> STARTING
                                                    |
                                             +------+------+ 
                                             |             |
                                           READY       START_FAILED
                                             |
                                          RUNNING
                                       /      |      \
                             EXITED_OK   EXITED_ERROR   STOPPING
                               |              |             |
                         (one-shot)      RESTART_WAIT    STOPPED
                           SUCCEEDED          |
                                          STARTING
                                               \
                                            DISABLED
```

`RUNNING` and `READY` are logically distinct observations: an OS child may exist but not be ready for dependent services. In the implementation state machine, readiness is a flag or explicit transition with transition history; double-success and unsolicited restarts are forbidden. For `ONESHOT`, successful exit and verified postconditions establish completion; for `DAEMON`, process alive is necessary but may not be sufficient for health.

### 6.4 Dependency, start and stop contract

1. Validate requirements and reject missing, duplicate/ambiguous service providers and cycles before spawn.
2. Start by topological dependency order, but move a dependent from WAITING to STARTING **only after requirements are READY** or an explicitly permitted one-shot success.
3. Propagate dependency failure by named policy: `WAIT`, `DEGRADE`, or `STOP_DEPENDENTS`. Never invent a successful dependency when readiness cannot be verified.
4. Shutdown in reverse dependency order where possible; allow bounded grace and report non-reaped children/descendants.
5. Child-process state is not the same as the declaration graph's desired state; record them separately and reconcile periodically.
6. A stop or restart request cannot target an unowned PID or a stale generation token.

### 6.5 Restart policy

`ON_FAILURE` is the default candidate for long-running services; `NEVER` and `ALWAYS` are explicit options. A restart budget specifies `(N attempts, T monotonic-time window)`, minimum delay and capped backoff. When exceeded, transition to `DISABLED` requiring an explicit owner action to re-enable. It must not reset merely because the supervisor's actor loops or a heartbeat timer fired. Persistent restart-budget recovery across supervisor crashes is an **open qualification issue**, not automatically solved by writing to a file.

A nominal example is `max_attempts=3 within 60 seconds, delay=2 seconds, backoff_cap=30 seconds`, which is illustrative **only**, not a recommendation or a default to encode prematurely. Use a fake clock for reproducible policy tests. Actual monotonic clock and signal behavior are backend-qualified.

### 6.6 Process-driver capability table

| Required capability | Mock driver | Real Linux candidate | Evidence state |
|---|---|---|---|
| Controlled spawn + argv/cwd/env | Simulated | Gerbil/Gambit process primitives may support base launch | API visible; exact lifecycle unqualified |
| Exit status and owned-child reaping | Simulated | Needs process-handle/OS verification | Unqualified |
| Hard wall-clock deadline | Simulated | Needs monotonic clock + explicit process control | Unqualified |
| Terminate owned process group and descendants | Simulated | May need POSIX PGID/session wrapper or qualified native shim | **Not provided merely by `run-process`** |
| Output backpressure and size bounds | Simulated | Needs bounded reader/pipe policy and deadlock tests | Unqualified |
| Explicit OS sandbox / privilege isolation | Not claimed | Optional future external OS mechanism | **Out of scope** |

**Critical implementation gate:** If the actual Gerbil/Gambit and target Linux APIs cannot fulfill a requirement, the real driver must expose that limitation and refuse configurations that demand stronger guarantees. Do not simulate a hard process deadline by attaching an idle timeout to a port, and do not claim descendant cleanup by signaling only the original PID.

### 6.7 Internal actors are not security boundaries

P02-B may use Gerbil actor threads to serialize commands and collect health reports. Actor-admin checks within one process are not a hostile-code isolation mechanism; all threads in that process are implicitly trusted for some administrative actor actions according to Gerbil's actor reference. Tool authority must come from the owning application's explicit policy and, where needed, OS boundaries, not from an actor name alone.

## 7. Failure and uncertainty semantics

### 7.1 Structured error taxonomy

- `INVALID_DECLARATION`, `UNDECLARED_INPUT`, `GRAPH_CYCLE`, `MISSING_PROVIDER`, `AMBIGUOUS_PROVIDER`, `COMPOSITION_CONFLICT`.
- `UNSUPPORTED_STAGE`, `UNKNOWN_EFFECT`, `AUTHORITY_NOT_GRANTED`, `PREPARE_FAILED`, `EFFECT_PARTIAL`, `OBSERVATION_INCOMPLETE`.
- `SPAWN_FAILED`, `READINESS_TIMEOUT`, `EXIT_NONZERO`, `EXIT_SIGNALLED`, `RESTART_BUDGET_EXHAUSTED`, `STOP_TIMEOUT`, `DESCENDANT_UNVERIFIED`, `REAP_UNCONFIRMED`.
- `STALE_PROCESS_IDENTITY`, `UNSUPPORTED_DRIVER_CAPABILITY`, `UNKNOWN_EXTERNAL_STATE`, `RECEIPT_PERSIST_FAILED`.

### 7.2 Crash consistency

A supervisor crash, host crash, parent process death, lost output or missing receipt may leave state uncertain. Recovery is a distinct reconciliation phase. Preserve evidence about what is known; **do not assume ownership or restart arbitrary processes by PID scan**, and do not claim that process existence or a matching name proves it belongs to an earlier run.

### 7.3 Compensation rather than magic rollback

A completed stage may modify external state before a downstream stage fails. Its receipt must record completed and pending effects, then optionally propose explicitly authorized compensating steps with independently checked postconditions. A compensation may itself fail. A versioned declaration graph is not a transactional OS snapshot.

### 7.4 Output and provenance handling

Record schema version, source revision, plan hash, correlation IDs, step identity, timestamp type (monotonic for elapsed timing; wall-clock for human logs), exit result, truncated-log marker, and health evidence. Bound log size/retention. Do not store environment secrets or unredacted credentials. Require a schema migration policy if receipt formats change.

## 8. Example vertical slices (design examples, not implemented)

### Slice A — Pure dependency composition

```text
Declare: A provides config, B extends A with typed values, C requires B output.
Validate: A,B,C exist; one target per extension; no cycles.
Compose: build effective config + explain every modified field.
Lower: produce normalized graph and declared input closure.
Output: graph digest, dependency order, validation trace, NOT_EXECUTED receipt.
```

Success requires the same canonical result across different declaration orders whenever merge semantics are order-independent. A conflicting extension must fail with source annotations.

### Slice B — One-shot local build-like task (mock first)

```text
Describe: task X consumes artifact A, produces artifact B.
Plan: explicit input reference, effect class EXECUTE, expected output check.
Authorize: owning application approves only this bounded effect.
Simulate: injected driver executes mock and returns structured outcome.
Observe: completed output contract or PARTIAL_FAILED, not just exit 0.
```

This does **not** instantiate Guix derivations or implement a package manager.

### Slice C — Two dependent services (mock then Linux)

```text
Declare: service database (DAEMON) and service worker (DAEMON).
Require: worker may start only after database readiness condition passes.
Inject: database exits repeatedly; restart budget is exhausted.
Expect: database DISABLED; worker receives dependency-failure policy outcome.
Shutdown: attempt reverse-order stop; report any descendants not proven dead.
```

A mock pass proves only policy/state behavior. A later Linux pass must independently qualify real signals, child-tree cleanup, reaping and output handling.

## 9. Module-level design deliverables and acceptance tests

### 9.1 Work packages (for future implementation only)

| Work package | Deliverable | Gate |
|---|---|---|
| `P02-W01` | Source/provenance lock, license analysis, finite-scope contracts | Q0 |
| `P02-W02` | Pure typed declarations and stable serialization | Q1 |
| `P02-W03` | DAG and extension registry, deterministic fold + explainability | Q2 |
| `P02-W04` | Data-only lowering, staged effect-plan + typed result/receipt | Q3 |
| `P02-W05` | Injected fake supervisor, fake clock, restart/readiness rules | Q4 |
| `P02-W06` | Conditional Linux process driver under strict capability tests | Q5 |
| `P02-W07` | Independent review and cross-profile dependency checks | Q6 |

### 9.2 Exact negative/positive test scenarios

| Test | Given / stimulus | Required observation |
|---|---|---|
| **P02-T01** | Load P02-A without supervisor/native dependencies | Pure imports and tests succeed; no OS process starts |
| **P02-T02** | A->B->C reference chain | Deterministic closure and topological order |
| **P02-T03** | Diamond dependency | Shared input appears once, retains both provenance paths |
| **P02-T04** | A->B->A cycle | Structured cycle diagnostic with path |
| **P02-T05** | Duplicate ID, differing definitions | Reject conflicting identity |
| **P02-T06** | Missing extension target | `MISSING_PROVIDER`, no effects |
| **P02-T07** | Two equally matching targets | `AMBIGUOUS_PROVIDER`, no arbitrary selection |
| **P02-T08** | Conflicting contributions on immutable field | Reject, cite both contributors |
| **P02-T09** | Commutative merges in permuted order | Byte-identical canonical representation/digest |
| **P02-T10** | Ordered noncommutative merge | Explicit declared ordering; no hidden dependence on hash iteration |
| **P02-T11** | Source edit changes declared input | Version/digest changes; stale plan refused |
| **P02-T12** | Undeclared host PATH/locale/clock in stage | Rejected rather than ambient capture |
| **P02-T13** | Stage contains executable Scheme form | Data-only decoder rejects code-bearing input |
| **P02-T14** | Dry-run plan | Receipt marked `NOT_EXECUTED`; no mutable side effects |
| **P02-T15** | Third plan action fails after two committed actions | `PARTIAL_FAILED`, completed steps preserved, no fake rollback |
| **P02-T16** | Lost execution acknowledgement | `UNKNOWN` until observation/reconciliation |
| **P02-T17** | Descriptor includes credential literal | Validation rejects or redacts according to explicit field policy |
| **P02-T18** | Fake supervisor: dependency not ready | Dependent process never starts |
| **P02-T19** | Fake supervisor: requirement completes as ONESHOT | Success semantics differ from long-running readiness |
| **P02-T20** | Fake process exits successfully once | `ONESHOT` succeeds only with verified postcondition |
| **P02-T21** | Fake daemon dies with failure, restart allowed | Backoff and attempt count match policy |
| **P02-T22** | Restart budget exceeded | `DISABLED`, never infinite respawn |
| **P02-T23** | Fake dependency disabled mid-run | Policy-specific propagate/wait/stop result |
| **P02-T24** | Fake clock jumps wall time backward | Restart budget uses monotonic logical time |
| **P02-T25** | Stale process generation / PID reused | Cannot stop or signal unrelated process |
| **P02-T26** | Stop request with grace deadline | Required stop/reap/escalation or explicit `UNVERIFIED` |
| **P02-T27** | Real child continuously streams output | True wall-clock deadline unaffected by output chatter |
| **P02-T28** | Real child forks descendants | Process-group cleanup verified or capability refused |
| **P02-T29** | Real child creates output saturation | Bounded memory, drained streams and no deadlock |
| **P02-T30** | Real child exits during readiness probe | Correct race handling and non-success state |
| **P02-T31** | Parent crashes during child's lifetime | No false claim of recovered ownership; observation logs uncertainty |
| **P02-T32** | P02-A packaged/imported independently | No Guix/Guile/Shepherd, P01, SDL, LLM or privileged daemon loaded |

A test marked **design-specified** is not a test passed. Distinguish fake-clock/fake-child model tests, property tests and real Linux integration tests in every future receipt.

### 9.3 Qualification gates

- **Q0 — Preservation:** upstream SHAs, source links, applicable licenses, exact reviewed scope and known limits recorded. Shepherd official docs are not substituted for a code audit.
- **Q1 — Pure contracts:** decoder/type/serialization tests and input-size limits pass with no OS effects.
- **Q2 — Graph correctness:** closure, cycle, ambiguity, extension provenance and deterministic canonicalization tests pass.
- **Q3 — Stage/effect correctness:** data-only staging, declared inputs, dry-run and partial-failure results verified; no arbitrary Scheme evaluation.
- **Q4 — Supervisor model:** state machine, readiness, dependency propagation, fake clock, bounded restarts and stale identity pass under an injected driver.
- **Q5 — Native Linux process:** spawn, polling, deadlines, reaping, descendants, output bounds and crash behavior pass against actual OS processes, or the unsupported guarantees are explicitly excluded.
- **Q6 — Independent review and isolation:** evidence reviewed by a second agent/reviewer; P02-A builds alone; no code/import into P01 or future Meta-Foundation by accident.

**Release positioning:** `P02-A: PURE_CONTRACT_QUALIFIED` and `P02-B: SUPERVISOR_MODEL_QUALIFIED` are separately attainable. `P02-B: NATIVE_PROCESS_QUALIFIED` requires Q5; an unqualified real-process driver must remain opt-in experimental, not be silently enabled by default.

## 10. Security, legality and dependency discipline

1. Preserve upstream versions and audit notes. Guix/Shepherd licensing and reuse requirements must be reviewed component-by-component. **Studying architecture is not permission to copy GPL implementation into a differently licensed project**.
2. Gerbil-only new source; no mandatory Guile, Guix, Shepherd runtime, CLI, daemon or generated Guile scripts.
3. No shell-string interpolation. Commands use argv and validated executable paths, with an explicit environment and cwd policy; still not a sandbox.
4. Readiness probes must be bounded and not expose unapproved network endpoints or privileges; the local process driver must not gain global service-manager authority.
5. Ownership is explicit. Disallow unowned PID control, implicit adoption, signal-by-name, global `pkill`, or unverified process-tree extermination.
6. Keep secrets out of hashes/receipts/logs except non-reversible approved references with controlled disclosure. Hashing a secret does not automatically make it safe to publish.
7. Status, receipts and logs may be reviewed read-only without launching or mutating anything.
8. Mock tests never qualify native kernel behavior. Static source reviews never qualify compiled runtime behavior.
9. GNU Guix deployed on a workstation may later be a *test host*, but its existence says nothing about P02's portability or required dependencies.

## 11. Bounded proposed implementation sequence (no implementation authorization)

**Only P02, one profile at a time:**

1. **P02-00:** freeze exact source-lineage and contract semantics; resolve ambiguity on graph types and stage results.
2. **P02-01:** implement **P02-A only**, pure validators, graphs, composition, stage descriptors and dry-run fixtures.
3. **P02-02:** qualify P02-A with property tests, adversarial graphs, deterministic serialization and input-closure checking.
4. **P02-03:** implement P02-B's **mock driver** and independently test state, dependencies, restart windows, readiness and stop policy.
5. **P02-04:** inspect concrete target Gerbil/Gambit/POSIX process capabilities before writing a Linux driver; explicitly mark anything unsupported.
6. **P02-05:** qualify one Linux child-service vertical slice with controlled failure-injection and resource monitoring, if P02-04 evidence permits.
7. **P02-06:** independent gate review and publication of qualified limits, not automatic promotion to the Meta-Foundation.

**Do not proceed to P03 or detailed K0 foundation design as part of this P02 design artifact.** No deployment, system reconfigure, Shepherd edits, persistent service registration or new code push is authorized.

## 12. Open decisions to carry into P02 qualification (not questions requiring immediate owner reply)

| Question | Provisional design position | Evidence that will decide |
|---|---|---|
| Is P02 one release or two? | One source edition, two independent package profiles and separate qualification | Independent P02-A import/test and optional P02-B isolation |
| How expressive should extension combiners be? | Finite explicit set of typed combiners first; no serialized closures | Real applications needing extensions not expressible with the finite set |
| Do we need executable staged Scheme? | **No**; retain data-only normalized stages | Demonstrated domain requirement + security review, not analogy to Gexps |
| When to add real process supervision? | After fake-driver Q4 and kernel capabilities audit | Spawn/reap/deadline/process-group tests |
| What process platforms? | First candidate: GNU/Linux x86_64, Guix-managed host optional | Real toolchain, target kernel and CI evidence |
| Is service state durable across supervisor crash? | **No guarantee initially**; preserve explicit UNKNOWN and require reconciliation | Verified recovery/re-adoption protocol with process identity |
| Should P02 own global policy? | **No**; local adapter checks scoped permission, broader policy belongs to host | Independent cross-domain requirements |
| Does a future cross-edition shared contract emerge? | **Undecided**; no K0 promotion during P02 design | Two dissimilar edition consumers passing compatibility tests |

## 13. Final disposition

`P02_D01_DETAILED_DESIGN_PROPOSED / GUIX_GUILE_SHEPHERD_MECHANISMS_MAPPED / TWO_PROFILES_SEPARATED / GERBIL_NATIVE / MOCK_FIRST / NATIVE_SUPERVISION_UNQUALIFIED / NO_GUILE_RUNTIME_DEPENDENCY / OTHER_EDITIONS_UNTOUCHED / NO_GITHUB_PUSH`

**One-sentence design conclusion:** From Guix/Guile, inherit the practice of **explicit, inspectable, dependency-aware declarative computation** without recreating a package manager; from Shepherd, inherit **bounded service lifecycle supervision** without pretending that an actor or a PID alone is a trusted service manager; express both as independent, tested Gerbil mechanisms with distinct effect boundaries.

## Primary source references

- GNU Guix pinned source: https://github.com/guix-mirror/guix/tree/18f545c14eb36c7c588d2cf356e387e17e53cd51
- Guix staged expressions: https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/gexp.scm
- Guix package lowering: https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm
- Guix service extensions: https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/services.scm
- Guix profile generation switching: https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/package.scm
- Prior Guix source audit in research fork: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gnu-guix-system-and-package-manager-scheme-source-review.md
- GNU Shepherd documentation: https://www.gnu.org/software/shepherd/manual/html_node/
- Gerbil process utilities: https://gerbil.scheme.org/reference/std/misc/process.html
- Gerbil actor API: https://gerbil.scheme.org/reference/std/actor.html
- Coding Harness source audit: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects/06-coding-harness-gerbil-source-code-review.md
- LFR-C02 accepted conceptual direction: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/concept/LFR-C02-source-lineage-gerbil-reserve-concept-2026-11-12.md
- P01-D01 sibling design (not imported): https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/design/LFR-P01-D01-native-metaprogramming-detailed-design-2026-11-12.md