# LFR-P03-D01 | Symbolic State Machines & Causal Histories / Semantic Compilation & Temporal State DSL

**Document class:** Detailed Design Proposal, one specialized edition only  
**Date:** 12 November 2026  
**Status:** `DETAILED_DESIGN_PROPOSED / STATIC_SOURCE_GROUNDED / RUNTIME_UNQUALIFIED`  
**Programme:** Lisp Foundation Reserve, current concept LFR-C02  
**Edition:** P03 — Symbolic State Machines & Causal Histories  
**Design slice:** D01 — Semantic Compilation & Temporal State DSL  
**Target implementation:** Gerbil Scheme, standalone and independently testable  
**Research baseline:** Glow source at `47753e6777ebe933be319f117c00401745eaeb2f`, public Naughty Dog/Uncharted GDC 2008/2009 materials, Alien-Works at `86381f8bce44a396347cbe8de70349be5b43ece1` as secondary comparison  
**Publication:** Owner-approved documentation publication on 12 November 2026 under `docs/design/` in `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`. This publication does not authorize source implementation, source imports, deployment, or a custom VM.  
**Isolation:** Does not revise P01-D01, P02-D01, LFR-C02, existing audits, or the independently parked historical concept 04.

## 0. Decision and intended product

Design one Gerbil-native **semantic compilation and temporal orchestration reserve** with two independently usable profiles, sharing one typed behavioral representation:

- **P03-D01-A — Semantic Compiler (primary, pure):** validate small domain descriptions, normalize them through inspectable passes, construct control-flow/state graphs, compute correct fixed-point facts, and produce immutable verified `BehaviorSpec` artifacts. A consumer can use P03-D01-A without starting the runtime or loading foreign libraries.
- **P03-D01-B — Temporal State Runtime (opt-in):** interpret validated `BehaviorSpec` objects with deterministic, bounded state transitions, events, logical timers, suspension/resumption, and coordinator/director instances. It emits effect *intents*, not unchecked native operations. External effect executors belong to a consumer or separately qualified adapter.

P03-D01 is *not* the Gerbil Meta-Foundation, a general build system, a game engine, a distributed workflow engine, a blockchain participant runtime, an actor capability-security framework, or a re-creation of Naughty Dog's proprietary VM. It does not use Glow itself as a runtime dependency.

**Product purpose:** allow applications to express what happens, when it can happen, what an event changes, and what external effect is requested, without scattering untestable callbacks and clocks throughout implementation code. This should work for small interactive simulations, task coordinators, device-independent scenarios and other domains, while making the boundary between pure behavior and external consequences explicit.

**Core hypothesis:** source-authored declarative behavior is useful only when its static analysis and runtime semantics are precise enough to permit deterministic tests, rejection of invalid programs, bounded execution, and replay that never silently duplicates real-world side effects.

## 1. Source lineage: exactly what P03-D01 learns and what it must repair

| ID | Source / original problem solved | Proven mechanism or public evidence | Boundary / issue | P03-D01 design response | Falsifiable evidence needed |
| --- | --- | --- | --- | --- | --- |
| **GL-01** | [Glow `compiler/passes.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/passes.ss): compile declarative interactions | Registered layers, passes and intermediate outputs from parse/alpha/type/ANF to checkpoints/projection | A pass's presence does not establish invariant preservation; some source comments identify missing source locations / ancillary-data handling | Versioned `PassContract`, explicit input/output layer schemas, validator after each pass, diagnostics with source span | Inject a malformed pass and ensure the pipeline fails at the named boundary |
| **GL-02** | [Glow `compiler/multipass.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/multipass.ss): make compiler steps inspectable and testable | Layer registry, pass registry, ordered strategies, read/write/compare test artifacts | Mutable registries and stage-local comparison alone are not semantic proof; strategy comments describe future DAG work | Immutable compile-session registry, explicit pass DAG validation, per-pass artifact identity and schema | Duplicate outputs, pass cycles, mismatched inputs and nondeterministic canonical output are rejected |
| **GL-03** | [Glow `compiler/checkpointify/checkpointify.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/checkpointify/checkpointify.ss): mark participant/transition boundaries | Checkpoint graph with incoming/outgoing transitions and variables/effects | Glow checkpoints are for multiparty interaction, not universal orchestration; joins, branches and loops complicate analysis | General `BehaviorGraph` with explicit edge identity and `use/def`, no implicit participant or consensus semantics | Diamond, loop, join and unreachable-node fixtures preserve graph invariants |
| **GL-04** | [Glow `compiler/liveness/checkpoint-liveness.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/liveness/checkpoint-liveness.ss): compute values live across checkpoints | Backward propagation of introduced/used variables through transitions | Source-grounded concern: `seen` can suppress predecessor reprocessing after facts grow at a join; an end-to-end reachable failure remains to be proven | Monotone set-lattice worklist; re-enqueue predecessors **whenever** facts change, even if visited | Counterexample and property tests against a simple reference fixed-point implementation |
| **GL-05** | [Glow `compiler/project/project.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/project/project.ss): transform one interaction into participant-specific views | Structural projection guided by checkpoint and participant context | Its consensus/EVM and participant semantics are not generally applicable; correctness not inherited | Keep multi-view/endpoint projection as **deferred optional investigation**, not a core P03-D01 feature | Later: prove view consistency for a chosen restricted semantics before adding a module |
| **UC-01** | [Gregory, GDC 2009](https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf): define repeatable native-game behavior in a domain language | Publicly described states, begin/update/end, events, transitions, native hooks | Proprietary production implementation unavailable; platform- and game-specific | A new finite-state/event DSL with explicit guards, transition atomicity and event ordering | Hand-authored trace fixtures, observational equivalence between IR and reference interpreter |
| **UC-02** | [Gregory, GDC 2009](https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf): orchestrate sequences across objects | Stand-alone directors, waits/signals, state tracks, suspension/resume as public architectural pattern | Director does not equal ordinary event poller or universal async runtime; real-world timing uncertain | `DirectorSpec` is a composition of P03-D01 machines plus explicit signals and logical timers | Deterministic wait/resume tests, timeout and cancellation precedence, no lost wakeups |
| **UC-03** | [Liebgold, GDC 2008](https://media.gdcvault.com/gdc08/slides/S6636i1.pdf): use Scheme between data and native engine | Data definition / compilation boundary | Public slides are **not** open-source implementation or license to reproduce engine scripts | Independent Gerbil source authoring -> canonical IR -> inspected artifact | No proprietary source copied; independent fixture attribution and clean lineage |
| **AW-01** | [Alien-Works materials](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp): macros for specialized graphics authoring | `defmaterial`, shader declarations and builder abstractions | Coupling to Varjo/Filament, runtime registration and native effects | Optional ergonomic Gerbil macros that emit **only** validated P03-D01 data, not privileged backend calls | Macro expansion equals hand-authored normalized representation |
| **AW-02** | [Alien-Works host](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp): native event polling | SDL events handed to callbacks | Event polling is not transition semantics or a director | Untrusted events enter through an explicit envelope/schema validation boundary | Invalid, duplicate, late and out-of-order events follow defined policies |
| **GB-01** | [Gerbil macro guide](https://gerbil.scheme.org/guide/intro.html), [prelude syntax](https://gerbil.scheme.org/reference/gerbil/prelude/macros.html) | Hygienic `defsyntax`, `syntax-case`, phase separation and module imports | Macro-phase authority is still trusted code; documentation does not prove P03-D01 DSL implementation | Small Gerbil syntax front end over a validated serializable AST; data-only loader is preferred for untrusted input | Macro/data parser conformance fixtures and source-location diagnostics |

**Evidence grading:** GL/AW claims are bounded direct source inspection. UC claims are primary public presentations interpreted through the existing review, **not** production-source access. GB is official language documentation. All P03-D01 behavior below is a proposed design, not a passed runtime test.

**Negative-transfer rules:** no Glow blockchain/consensus imports; no Naughty Dog code, binary, VM, game scripts or disguised recreation; no unconditional Varjo/Filament imports; no direct executable authorization from a macro; no inference that using Scheme makes a compiler correct.

## 2. Scope and ownership boundaries

P03-D01 **owns**:
1. Domain syntax and canonical behavior IR; semantic validation and diagnostics.
2. Finite control-flow graphs, pass artifacts, graph analyses and termination diagnostics.
3. Pure step semantics, logical time, timers, explicit event ordering and state transitions.
4. Local director composition, signal/wait coordination and bounded scheduling decisions.
5. Trace/receipt schemas local to behavior, replay of **pure decisions**, and model/test harnesses.

P03-D01 **does not own**:
- native handles, rendering, FFI or memory (P01-D01); OS child/process supervision, Guix or deployment graphs (P02-D01);
- global user authorization, agent tool access, filesystem/network privileges (future consumer or P02-D02);
- distributed messages, consensus, network timing, RPC delivery, durable exactly-once effects, arbitrary coroutine/continuation capture;
- general-purpose bytecode VM, runtime evaluation of untrusted Gerbil source, or unrestricted third-party plugin execution;
- the final Meta-Foundation K0/K1 contract design. P03-D01 exports only a small, versioned **proposed** identity/schema/effect/result projection for future comparison.

A timer in P03-D01 is an instance-local **logical deadline**. It is not an OS process timer, deadline enforcement guarantee, hardware clock or a durable scheduling service.

## 3. Architecture: pure semantics first

```text
           P03-D01-A: COMPILER / ANALYSIS (Gerbil; pure)
  trusted macro syntax  OR  untrusted validated data descriptor
                  |                     |
            source AST / source maps / schema version
                             |
                parse -> resolve -> validate
                             |
              normalize -> BehaviorGraph (CFG)
                             |
             worklist dataflow + semantic checks
                             |
              lower -> immutable BehaviorSpec
                             |
                  digest / diagnostics / tests
                             |
             optional P03-D01-B loading boundary
                             v
              P03-D01-B: TEMPORAL RUNTIME (Gerbil)
       (MachineInstance, DirectorInstance, ClockAdapter)
                             |
      event normalization -> deterministic step selection
                             |
        new state + timers + effect intents + trace
                             |
                optional consumer adapter
           (authorization and real effects OUTSIDE P03-D01)
                             |
                observed effect-result events
```

**Dependency invariants:** P03-D01-A imports no runtime interpreter, native backend, graphics, Guix, Guile, Shepherd, LLM, database or network package. P03-D01-B depends on P03-D01-A's validated artifacts and pure semantic functions. Both run in Gerbil/Gambit with a mock event/clock host; no custom VM is a prerequisite.

## 4. Proposed Gerbil module structure (not created)

```text
editions/semantic-state/
  README.md
  gerbil.pkg
  src/
    schema/
      version.ss            # wire schema and compatibility policy
      ast.ss                # data-only syntax tree and source spans
      ir.ss                 # typed canonical behavior graph/spec
      diagnostics.ss        # structured semantic errors
    compiler/
      parse.ss              # restricted data parser; no eval
      macros.ss             # trusted authoring macros -> AST
      resolve.ss            # names, scopes, symbol IDs
      normalize.ss          # control-flow normalization
      passes.ss             # immutable pass contracts, output validators
      cfg.ss                # graph construction and reachability
      dataflow.ss           # fixed-point worklist engine
      lower.ss              # canonical BehaviorSpec creation
      encode.ss             # stable binary/text encoding and digest inputs
    semantics/
      transition.ss         # reference pure step relation
      clocks.ss             # logical timers, explicit time advances
      events.ss             # envelopes, ordering, bounded queue policies
      effects.ss            # intent types and effect-result contracts
    runtime/
      machine.ss            # state-instance driver
      director.ss           # composition, wait/signal behavior
      scheduler.ss          # bounded microsteps / queue management
      replay.ss             # pure trace replay, never external actions
    adapters/
      mock.ss               # fake clock, events and effects
      sink.ss               # protocol only; no built-in side effects
  tests/
    fixtures/
    compiler/
    analysis/
    runtime/
    model/
  evidence/
    source-lineage.md
    failures.md
    qualification.md
```

There is **no** `native/`, `ffi/`, `vm/`, `guile/`, `guix/`, `engine/` or `network/` module in the required P03-D01 design. No compiler pass runs arbitrary user-provided Scheme as part of descriptor parsing.

## 5. Type contracts and canonical representation

| Type | Required fields | Invariant |
| --- | --- | --- |
| `SourceUnit` | `{source_id, schema_version, span_table, trusted_syntax_flag, digest}` | Source map points to original declarations; digest covers exact source bytes where retained |
| `DomainAST` | `{definitions, machines, directors, declarations, imports}` | Data grammar has explicit allowed tags; no executable `(eval ...)` |
| `PassContract` | `{pass_id, version, input_schema, output_schema, invariants, deterministic_flag}` | Stable pass identity and validated output are mandatory |
| `PassArtifact` | `{layer, value_digest, source_digest, pass_version, diagnostics}` | Does not reuse result across changed inputs or compiler versions |
| `BehaviorGraph` | `{node_ids, edge_ids, entries, terminals, edge_use_def, spans}` | References resolve; graph IDs unique; edges are explicit |
| `BehaviorSpec` | `{spec_id, schema_version, compiler_version, options, initial_state, transitions, timers, event_schemas, effect_contracts, artifact_digest}` | Serializable, immutable, validated at load; no closures or raw foreign pointers |
| `EventEnvelope` | `{event_id, producer_id, machine_id, instance_generation, type, payload, logical_due, insertion_seq}` | Validated schema; ordering is explicit; target generation must match |
| `MachineSnapshot` | `{instance_id, generation, spec_digest, state_id, state_data, timers, logical_now, cursor, queue_state}` | No stale generation; canonical state representation |
| `TransitionRule` | `{source_state, event_kind, guard_id, priority, target_state, actions, source_span}` | Guards are pure; ambiguous precedence rejected, not silently guessed |
| `TimerToken` | `{timer_id, instance_generation, due_logical, scope_id, cancellation_state}` | Cancelled/old-generation timers cannot resume a new instance |
| `EffectIntent` | `{intent_id, effect_kind, payload, required_capability, instance_generation, causal_transition}` | Mere intent never performs a real effect; downstream authorization is separate |
| `StepReceipt` | `{before_digest, event_id, chosen_rule, after_digest, timer_changes, intents, status, diagnostics}` | Pure transition output tied to exact inputs and version |
| `DirectorSpec` | `{director_id, member_specs, subscriptions, wait_rules, join_rules, limits}` | Explicit topology; no default global access to all member states |
| `ReplayTrace` | `{spec_digest, initial_digest, ordered_inputs, recorded_results, expected_final_digest}` | Real effects forbidden during replay; traces record observations, not magically rewind them |

Identifiers are stable **within a versioned artifact**. New compiler or policy versions must not silently reinterpret existing serialized snapshots. Unknown version => `INCOMPATIBLE_SPEC`, not permissive decoding.

### 5.1 Proposed authoring form

The following is **illustrative domain notation**, not asserted valid Gerbil code or a finalized grammar:

```scheme
(define-machine door-controller
  (state closed)
  (state opening)
  (state open)
  (on closed (request-open)
      (guard authorized-request?)
      (to opening)
      (emit (actuate-open)))
  (on opening (observed-open)
      (to open))
  (on opening (timeout open-limit)
      (to closed)
      (emit (report-failure open-timeout))))
```

The names `authorized-request?` and `actuate-open` are **symbolic references resolved against a typed registry**, not arbitrary user-provided source strings or unrestricted functions executed by a data parser. In the pure conformance profile `authorized-request?` is a declared predicate over immutable input data and `actuate-open` becomes an `EffectIntent`. The application is responsible for deciding what authorization and real actuation mean.

A trusted Gerbil macro may make this pleasant to write; a data-only equivalent must compile to the **same canonical IR**. Macro expansion is not automatically a security boundary.

### 5.2 Canonicalization

- Normalize named symbols to scoped, versioned internal IDs without discarding source spans.
- Sort order-independent maps/sets; preserve explicitly ordered transitions and effects. Stable output must not depend on hash-table iteration order.
- Bind `spec_digest` to IR canonical bytes, compiler schema/version, feature flags and declared semantic options. **Hash equality is identity evidence, not a proof of correct behavior.**
- Expose `dump-ir`, `explain-transition`, `explain-pass`, `explain-liveness` and `diff-spec` diagnostics as pure interfaces.
- Unknown guards, effects, event payload fields and symbols fail before loading.

## 6. P03-D01-A compiler pipeline and pass contracts

| Stage | Input -> Output | Mandatory verification |
| --- | --- | --- |
| `S0 parse` | constrained data / trusted macro result -> `DomainAST` | accepted tags; source spans; bounded sizes and nesting |
| `S1 resolve` | `DomainAST` -> scoped names and symbol table | duplicate/unknown names, undefined events/states, cyclic includes |
| `S2 validate` | resolved AST -> typed `SemanticAST` | transition guards are pure; event and effect contract schemas match |
| `S3 normalize` | `SemanticAST` -> canonical state/CFG graph | deterministic order and explicit default/ambiguous-rule semantics |
| `S4 analyze` | CFG -> facts, liveness, reachability, SCCs | monotonic dataflow fixed point; unbounded cycles flagged by policy |
| `S5 lower` | graph+facts -> `BehaviorSpec` | all references resolved; no executable runtime closure in serialized artifact |
| `S6 verify` | `BehaviorSpec` -> `VerifiedBehaviorSpec` or diagnostics | post-lowering validation, canonical digest, source map and artifact version |

Pass runner duties:
1. Each pass has a unique ID and version; it cannot mutate an input artifact in place.
2. The runner validates the input layer, invokes the pass with explicit options, and validates the output layer.
3. A pass's claimed invariants are independently checked where executable checks exist; unproved invariants are labeled assumptions.
4. Failed output is quarantined. A downstream pass must not run on a failed or unknown layer.
5. Incremental reuse requires a key including input digest, pass version, compiler version, feature set and schema. A cache hit does not waive output schema checks.
6. Nondeterministic plugins, external I/O or ambient-host reads are excluded from the core compiler profile.
7. Every diagnostic includes pass, artifact, source span where available, and cause; no bare 'compiler failed' result.

Pass order is a constrained DAG, not a freeform build system. A pass DAG may branch into diagnostics and analysis artifacts, but P03-D01 does not become P02-D01's generic dependency graph framework.

## 7. Correct fixed-point analysis: the Glow lesson made explicit

For the finite graph `G = (V,E)`, a transition edge `e = (u,v)` has `USE[e]` and `DEF[e]`. For *may-liveness* of finite variable sets, define:

```text
L[u] = UNION over every outgoing edge e=(u,v):
       USE[e] UNION (L[v] MINUS DEF[e])
```

Initialize all `L[u] = empty-set`, enqueue all nodes or all relevant edges, and apply transfer repeatedly until **no node's set changes**. If `L[u]` grows, enqueue every predecessor of `u` (deduplicated while queued). `seen` may optimize queue membership but must **not** permanently prevent recomputation. Validate finite variable universe and bounded graph size; otherwise reject or return `ANALYSIS_INCOMPLETE` at a resource budget, never a fake fixed point.

This is a finite-height monotone framework under the proposed may-liveness transfer. It terminates in a finite number of fact additions if the implementation does not introduce new variables during iteration. The solver must permit cycles without terminal nodes: whether an orchestration loop can complete is a *separate runtime/liveness question*.

### 7.1 Critical join counterexample

```text
           +----> B ----[USE x]----> T
           |
           |
           S
           |
           +----> C ----> D ----[USE y]----> T
```

For the edge-based equation above, `L[T] = {}`, `L[B] = {x}`, `L[D] = {y}`, `L[C] = {y}`, and **`L[S] = {x, y}`**. In an unlucky backward traversal, the short `B` branch can cause `S` to be processed with `{x}` before the longer `C -> D` branch has delivered `{y}` to `C`. A permanent `seen` exclusion then prevents `S` from being revisited after `L[C]` grows, producing an incomplete result. A worklist must re-enqueue `S` whenever its successor fact changes. This is a source-grounded **static algorithmic counterexample** to single-visit reasoning, not proof that the published Glow application manifested the error end to end.

### 7.2 Analysis safeguards

- Test output against a deliberately slow reference iterator that scans *all edges* until no changes.
- Check fixed-point consistency: applying transfer once more yields the same result.
- Check least solution for small enumerated graphs or comparison with the bottom-started reference.
- Check graph-order independence, duplicate-edge invariance and alpha-renaming invariance.
- Check cyclic CFG, irreducible joins, isolated nodes, missing outgoing edges and unreachable nodes.
- Record step counts and convergence diagnostics; exhaustion must return `ANALYSIS_INCOMPLETE`, not `VALID`.
- Separate **static variable liveness**, **transition reachability** and **eventual runtime progress**; they answer different questions.

## 8. P03-D01-B operational semantics: exact behavior of one microstep

Define a pure relation:

```text
step(VerifiedBehaviorSpec, MachineSnapshot, EventEnvelope, LogicalClock)
   -> StepDecision | Rejection

StepDecision = {
    next_snapshot,
    timer_mutations,
    effect_intents,
    receipt,
    emitted_internal_events
}
```

`step` does not call a network, device, subprocess, Guix service or arbitrary user function. All domain predicates must be registered pure operations whose inputs and declared semantics can be tested; for data-only untrusted DSLs, the predicate vocabulary is strictly whitelisted.

### 8.1 Sequential microstep contract

1. Check artifact/compiler version and instance generation.
2. Validate event shape and event identity against the machine's bounded dedup policy.
3. Choose a ready event using a **stable total ordering**: `(logical_due, explicit_priority, insertion_seq, event_id)`; reject duplicate insertion sequences from the same queue epoch.
4. Snapshot state for all guard evaluation. Evaluate only guards applicable to current state and event kind, using pure deterministic functions.
5. Resolve rule selection: one eligible rule, or a deterministic documented priority; equal-priority conflicting transitions are `AMBIGUOUS_TRANSITION`. Unhandled event follows explicit `IGNORE/DEFER/REJECT` schema policy.
6. Compute `next_snapshot`, timer cancellations/schedules and effect intents without causing external effects.
7. Validate new state and all outputs; publish the pure decision atomically to the local machine state store under its single-writer model.
8. Put internal emitted events at the next scheduling boundary. An event handler cannot reenter the partially committed transition.
9. Record `StepReceipt`; only a separate authorized consumer may dispatch real effects. Its outcome must return as another validated event, not mutate the machine secretly.

The **pure state decision commit** is not an atomic transaction with external effects or a durable database. If a consumer executes an intent and loses the acknowledgement, its external state is `UNKNOWN` until reconciled. No automatic replay of non-idempotent effects.

### 8.2 Exact transition lifecycle

```text
INSTANCE_NEW -> ACTIVE(state_id, generation)
ACTIVE + event -> CHECKED -> RULE_SELECTED -> STATE_COMMITTED
                                  |                   |
                           REJECT/IGNORE       INTENTS_EMITTED
                                                      |
                                             result event (later)
ACTIVE -> PAUSED | COMPLETED | FAILED | CANCELLED
```

`PAUSED` means waiting for a named event, timer or result; `COMPLETED` requires the completion predicate; `FAILED` preserves the failure; `CANCELLED` invalidates relevant timers and queued events by generation. An externally requested cancel enters as a **control event**, not an invisible change to state.

## 9. Time, event, wait and resumption semantics

### 9.1 Clock model

- The core uses a **logical monotonic time** value represented by a chosen fixed-resolution integer unit; avoid floating-point timer equality.
- A `ClockAdapter` can map wall/monotonic real time to explicit `advance-to(t)` inputs **outside** the pure core. Real-time OS scheduling guarantees are not claimed.
- `advance-to(t')` must satisfy `t' >= t`; backward jumps are rejected. Timers with `due <= t'` become eligible in deterministic `(due, timer_id)` order.
- A paused program may be advanced by a fake clock in tests. Paused/resumed host execution does not imply timestamps can be invented retroactively.

### 9.2 Event ingress and delivery

- Every inbound event is validated, assigned stable local insertion sequence, and bound to a machine generation.
- P03-D01 guarantees **deterministic single-writer consumption** *for events it has accepted into one in-memory queue*. It does not promise distributed exactly-once delivery or durability across crashes.
- Queue capacity, event size, maximum age, and per-source rate limits are explicit. Backpressure returns `QUEUE_FULL` or other typed outcome; events are not silently dropped unless a declared bounded-drop policy permits it and emits a receipt.
- Within one microstep, producer callbacks may enqueue follow-up events, which are processed only at the next boundary.
- Event timestamps from outside are untrusted hints unless normalized by an admitted clock adapter.

### 9.3 Waiting and signals

`WaitSpec` contains `{wait_id, kinds, predicate_id, deadline?, scope_id, resume_target, on_timeout}`. A wait subscribes at a defined state boundary, with registration completed before the next queued event can be consumed. Thus a signal arriving on the same boundary is neither silently lost nor allowed to reenter a half-built wait.

If a signal and its timeout are eligible at the **same logical time**, the default proposed rule is **explicit priority by event category: cancellation > timeout > matching signal**, unless the artifact declares a different validated precedence. This is a design choice requiring tests; it is not claimed to be Uncharted's original behavior. Events processed later are not allowed to undo a terminal cancellation without an explicit new generation.

A wait cancellation invalidates associated timer tokens, guards and subscriptions. Late results from an older generation fail with `STALE_INSTANCE_GENERATION`.

### 9.4 Resource bounds and progress

- `max_microsteps_per_advance`, `max_queue_length`, `max_waits`, `max_director_members`, `max_timer_count`, `max_transition_depth`, and `max_serialized_snapshot_bytes` are configurable with conservative caps.
- A zero-time event loop that exceeds its fuel returns `STEP_BUDGET_EXHAUSTED` with a resumable **pure** snapshot or a policy-declared terminal fault; it must not spin indefinitely.
- Fairness is not automatic. If priorities can starve lower-priority events, the admission policy must declare this. Optional age-based fairness requires its own deterministic semantics.
- Repeating events, re-entrant callbacks and infinite self-transitions must not bypass the fuel counter.

## 10. Stand-alone directors: coordination without engine entanglement

A `DirectorInstance` composes a finite set of registered machines and can issue **typed events** and effect intents. It does not gain direct read/write access to every child variable. The relationship between director and child is an explicit subscription/command contract.

Director responsibilities:
1. Track named member instances and their generation-specific identities.
2. Subscribe to explicit child completion, milestone, failure and signal events.
3. Define `ANY`, `ALL`, or guarded joins with explicit failure and timeout policy.
4. Schedule waits and resume from observed signals under the same logical clock and deterministic event-order rules.
5. Propagate cancellation by named policy (`CANCEL_CHILDREN`, `DETACH`, `WAIT_FOR_EXIT`) only as typed commands; no invisible forced writes.
6. Emit trace entries showing which child event satisfied which condition.

Example: two child machines `A` and `B` must finish before a parent director emits `next-phase`. If `A` completes, `B` times out, and the director fails, the trace must preserve both outcomes rather than retroactively deleting `A`'s completion. If an external adapter performed an action for `A`, the director cannot roll it back merely by reverting a state snapshot.

**No automatic distributed causality or transactional cross-process coordination.** P03-D01 directors are in one logical execution domain; crossing machine/process/network boundaries requires a future separately qualified protocol adapter.

## 11. Effect boundary and authorization

P03-D01 emits **effect descriptors**, never unrestricted foreign calls:

```text
EffectIntent {
  id, kind, target_ref, payload_schema,
  causal_state_transition, generation,
  idempotency_class, required_capability
}
```

- The **trusted application** decides which effect kinds exist and which caller may execute them. A DSL validation pass does not grant OS permissions.
- `PURE`: no external effect. `REQUEST_EFFECT`: produces a descriptor only. `OBSERVE_EFFECT`: consumes an external outcome event; it does not invoke the executor.
- An effect outcome is one of `SUCCEEDED`, `FAILED`, `CANCELLED`, `UNKNOWN`, with observable external-state evidence when available.
- `UNKNOWN` forbids unconditional retry of non-idempotent effects. Idempotency keys are caller/backend contracts, not magic exactly-once guarantees.
- An external executor crash, timeout or lost acknowledgement must leave a traceable pending/uncertain intent, never convert to success by default.
- P03-D01 must not hardcode P01-D01 resource APIs, P02-D01 process APIs or P02-D02 agent tools. Those are optional consumers of normalized descriptors through separately validated integration contracts.

## 12. Replay, observability and persistence boundaries

**Replay modes:**

1. `PURE_REPLAY`: re-run compiler-produced state transitions against the recorded sequence of inputs and **recorded** effect-result events. Never invoke external adapters.
2. `TRACE_CHECK`: compare before/after digests, selected rules, timers, transition source spans and intent identities against expected trace.
3. `SCENARIO_SIMULATION`: fake clock/event source and mock outcomes, including failure injection and permutation testing.

`LIVE_REEXECUTION` is **not** a P03-D01 replay mode. If a consumer needs to reissue actual effects, it must make an independently authorized execution decision with idempotency/reconciliation checks. Persistent crash recovery is deferred: the proposed initial product is in-memory deterministic; a storage adapter is optional and must explicitly state durability and write ordering.

Record versioned field types, bounded trace length, cause IDs, redaction policy, and artifact digest. Snapshotting memory does not prove physical rollback, and restoring a machine snapshot does not undo signals already delivered elsewhere.

## 13. Failure taxonomy

| Error class | Trigger | Required outcome |
| --- | --- | --- |
| `PARSE_REJECTED`, `UNSUPPORTED_SYNTAX` | malformed data grammar or prohibited code | no macro eval of untrusted source |
| `DUPLICATE_ID`, `UNKNOWN_STATE`, `UNKNOWN_EVENT` | semantic definition invalid | block S5 lower |
| `SCHEMA_MISMATCH`, `IMPURE_GUARD` | invalid input type or effectful predicate in pure guard | named compiler error |
| `PASS_PRECONDITION_FAILED`, `PASS_POSTCONDITION_FAILED` | pass layer contract breach | fail closed and preserve diagnostic |
| `ANALYSIS_INCOMPLETE`, `LIVENESS_INVALID` | non-convergent by resource bound or wrong fixed point | never qualify artifact |
| `AMBIGUOUS_TRANSITION`, `UNHANDLED_EVENT` | nondeterministic or unhandled runtime rule | explicit policy-based reject/ignore/defer |
| `CLOCK_REGRESSION`, `TIMER_STALE` | invalid time advance or old timer | reject without machine mutation |
| `QUEUE_FULL`, `STEP_BUDGET_EXHAUSTED` | bounded resources | typed backpressure/fault, no infinite loop |
| `STALE_INSTANCE_GENERATION` | late event or result | reject stale delivery, retain evidence |
| `EFFECT_UNKNOWN`, `EFFECT_REJECTED` | effect result uncertain/refused | no invented success or retry |
| `DIRECTOR_JOIN_FAILED`, `DIRECTOR_CANCELLED` | join/cancellation policy outcome | consistent multi-child trace |
| `INCOMPATIBLE_SPEC`, `TRACE_DIVERGED` | schema version mismatch or non-equivalent replay | report mismatch and halt replay |

## 14. Three proposed vertical slices (design only)

### Slice A — Compiler-only finite-state specification

Input: data-only `DoorMachine`, three states, two events, one timeout, one symbolic effect.  
Expected: validated `BehaviorSpec`, deterministic digest, a source-mapped IR dump, and rejected tests for missing states, duplicate transitions and impure guards.  
Dependencies: Gerbil/Gambit; **no runtime engine, native FFI or clock**.

### Slice B — Fake-clock temporal director

Input: director waiting for two child milestones, one delayed signal, one timeout, and one cancellation event.  
Expected: deterministic total event order; repeatable final state; identical traces across source-map-only changes; cancelled timers never resume a new instance.  
Dependencies: P03-D01-A + P03-D01-B mock clock and queue; **no OS timer guarantees**.

### Slice C — Mock external effect with uncertain result

Input: machine emits `open-actuator` intent; fake consumer records an effect but returns `UNKNOWN` due to lost acknowledgement; later an observed result arrives with the same causal ID.  
Expected: no automatic duplicate actuation; trace records uncertainty and reconciliation; pure replay reproduces decision path **without** calling an actuator.  
Dependencies: in-memory mock only. This validates the semantic boundary, not a real device.

## 15. Proposed API surface (behavioral pseudocode only)

```text
parse_descriptor(data, limits)                -> DomainAST | ParseError
expand_trusted_dsl(syntax)                   -> DomainAST | ExpansionError
compile_behavior(ast, compiler_config)       -> VerifiedBehaviorSpec | DiagnosticSet
inspect_pass(artifact, pass_id)              -> PassArtifact | NotFound
explain_liveness(artifact, graph_node)       -> AnalysisExplanation
validate_spec(serialized_spec)               -> VerifiedBehaviorSpec | Incompatible
create_machine(spec, instance_id, start_at)  -> MachineSnapshot
accept_event(machine, event)                 -> QueuedEvent | RejectReceipt
advance_time(machine, logical_t)             -> StepBatch | RejectReceipt
step(machine, normalized_event)              -> StepDecision | RejectReceipt
create_director(spec, children, clock)       -> DirectorSnapshot
cancel(instance, reason)                     -> ControlEvent
replay_pure(trace, spec)                     -> ReplayVerdict
```

These names specify behavior, not compilable Gerbil signatures. No arbitrary host `eval`, unrestricted procedure closures, remote actor authority, or implicit real-world effect handling is part of this surface.

## 16. Qualification test matrix: 36 exact scenarios

| ID | Scenario | Required result |
| --- | --- | --- |
| **P03-D01-T01** | Build P03-D01-A without P03-D01-B or any native/Guix/LLM dependencies | Import/build succeeds in an isolated Gerbil toolchain |
| **P03-D01-T02** | Unknown DSL tag from data-only input | `UNSUPPORTED_SYNTAX`, no eval |
| **P03-D01-T03** | Trusted macro vs equivalent data AST | Same canonical `BehaviorSpec` |
| **P03-D01-T04** | Duplicate state, event or rule ID | Rejected before S5 |
| **P03-D01-T05** | Missing state or event target | Source-located error |
| **P03-D01-T06** | Impure or unknown guard | Static rejection |
| **P03-D01-T07** | Pass produces wrong schema | Named `PASS_POSTCONDITION_FAILED` |
| **P03-D01-T08** | Compiler pass execution reads ambient input | Strict core mode refuses undeclared dependency |
| **P03-D01-T09** | Same descriptor with randomized map order | Stable canonical digest |
| **P03-D01-T10** | Changed compiler or semantic option version | No reuse of stale artifact |
| **P03-D01-T11** | Diamond CFG; variables introduced/used along different paths | Correct liveness by slow reference solver |
| **P03-D01-T12** | Late-growing fact at a join | Predecessors re-enqueued, complete least fixed point |
| **P03-D01-T13** | Cyclic CFG without terminal node | Analysis converges or explicitly reports bounded failure |
| **P03-D01-T14** | Random graph 1–20 nodes with mixed use/def | Worklist equals all-edges reference solver |
| **P03-D01-T15** | Reverse edge enumeration order | Identical liveness facts |
| **P03-D01-T16** | Artificial low analysis budget | `ANALYSIS_INCOMPLETE`, never qualified |
| **P03-D01-T17** | Two competing equal-priority rules | `AMBIGUOUS_TRANSITION` |
| **P03-D01-T18** | Unhandled event with each declared policy | Exactly `IGNORE`, `DEFER`, or `REJECT` semantics |
| **P03-D01-T19** | Same event log and logical clock in two runs | Same state/receipt trace |
| **P03-D01-T20** | Event enqueued during a handler | Processed after current commit, no reentrancy |
| **P03-D01-T21** | Fake clock advances to exact timer deadline | Timer fires once at declared boundary |
| **P03-D01-T22** | Fake clock moves backwards | `CLOCK_REGRESSION`, unchanged state |
| **P03-D01-T23** | Timeout and signal at identical logical instant | Declared precedence enforced |
| **P03-D01-T24** | Cancellation and timer at same instant | Cancellation wins under default rule; timer invalidated |
| **P03-D01-T25** | Late event for old machine generation | Rejected as stale |
| **P03-D01-T26** | Duplicate event under bounded dedup policy | Identified duplicate never silently executes transition twice |
| **P03-D01-T27** | Queue reaches capacity | Typed backpressure; no silent uncontrolled growth |
| **P03-D01-T28** | Zero-time self-emitting loop | Fuel exhaustion, no infinite run |
| **P03-D01-T29** | Director `ALL` wait, A completes, B completes | One completion after both, causal trace intact |
| **P03-D01-T30** | Director `ALL` wait, A completes, B times out | Failed join retains A completion and B timeout |
| **P03-D01-T31** | Director cancellation with pending child results | No stale child result revives director |
| **P03-D01-T32** | Mock effect succeeds but result acknowledgement lost | Outcome `UNKNOWN`, no automatic unsafe retry |
| **P03-D01-T33** | Reconcile unknown effect from later observation | Correct causal receipt linkage |
| **P03-D01-T34** | Pure trace replay with mock observer instrumented | Zero calls to any external effect sink |
| **P03-D01-T35** | Serialized spec from incompatible schema version | `INCOMPATIBLE_SPEC`, no permissive interpretation |
| **P03-D01-T36** | Mutated recorded rule result or timer order | `TRACE_DIVERGED` with first mismatched step |

`P03-D01-T11`–`T16` are not optional “unit tests”; they protect precisely the Glow fixed-point failure class identified in prior source review. `P03-D01-T19`–`T34` protect temporal semantics. Count: **36 scenarios**, all design requirements rather than executed passes.

## 17. Qualification gates

- **Q0 — Provenance/license:** pinned source lineage; attributed public lectures; permitted reuse; proprietary/source access boundary recorded.
- **Q1 — Pure compilation:** data grammar, syntax macro equivalence, source maps, pass contracts, canonicalization, and dependency isolation pass.
- **Q2 — Graph-analysis correctness:** joins, cycles, transfer laws, fixed-point reference oracle, mutation testing and constrained resource failure pass.
- **Q3 — Reference state semantics:** finite-state step invariants, transition selection, ambiguity policy and trace determinism pass under a fake clock.
- **Q4 — Temporal/director semantics:** explicit time ordering, wait/signal/cancel, join behavior, bounded queue/fuel and stale-generation rules pass.
- **Q5 — Effect/replay boundary:** intent-only output, lost-acknowledgement uncertainty, pure replay isolation, causal evidence and versioned artifact loading pass.
- **Q6 — Independent review and portability:** cold Gerbil build, negative import/dependency audit, property/fuzz tests, documented performance limits and independent failure-path review pass.

**Gate accounting:** P03-D01-A may be released as `COMPILER_QUALIFIED` after Q0–Q2 with its own evidence. P03-D01-B is not qualified without Q3–Q5. Q6 is required before describing any edition as generally reusable. Design acceptance is not execution qualification; an audited source does not count as a passing P03-D01 test.

## 18. Proposed staged implementation, if separately authorized

1. **P03-D01-00:** freeze AST/IR schemas, source maps and source-lineage/negative-transfer register. Write a reference conformance specification, not a broad framework.
2. **P03-D01-01:** implement parser, resolver, canonicalization and compiler pass runner with synthetic fixtures; test without runtime.
3. **P03-D01-02:** implement finite-graph worklist analysis and independent slow reference solver; require T11–T16 before any checkpoint-like optimization.
4. **P03-D01-03:** implement tiny pure reference interpreter and deterministic logical clock; exercise T17–T28.
5. **P03-D01-04:** implement directors, joins, waits and cancellation semantics under mock event source; exercise T29–T31.
6. **P03-D01-05:** implement intent/result envelopes and pure replay; exercise T32–T36.
7. **P03-D01-Q01:** independent test tour, negative-dependency build and evidence-based gate disposition.

No P01-D01/P02-D01 code changes, K0 implementation, custom VM, native engine, OS deployment, or source port is authorized as part of this *design document*. Publication of P03-D01 itself was separately approved by the owner. Performance optimization should follow reference correctness, not precede it.

## 19. Unresolved questions, recorded rather than concealed

1. **Nested/hierarchical states:** default is flat machine states plus director composition. Hierarchical entry/exit order is postponed until a user need and model-checking strategy exist.
2. **Orthogonal parallel states:** default is separate machines plus explicit messages; do not silently introduce simultaneous transitions.
3. **Priority fairness:** explicit total order is deterministic but can starve; bounded fairness/age policy requires independent tests.
4. **Clock granularity:** choose a fixed integer tick unit in the first implementation. Wall-time synchronization, drift and host clock mapping are adapter concerns.
5. **Liveness vs eventual progress:** fixed-point variable liveness is not temporal model checking; a separate temporal property language requires new design and tests.
6. **Guard language:** restrictive declarative predicates are preferred for untrusted specs; trusted Gerbil guard procedures must explicitly opt out of replay/purity claims unless qualified.
7. **Snapshot persistence:** strict in-memory guarantee first; transactional durable storage and crash recovery are not presumed.
8. **Multi-endpoint projection:** Glow inspires a future profile only after a formal compatibility question is demonstrated. No consensus or participant lowering in D01.
9. **Custom VM:** not selected. If direct interpreter throughput is inadequate, benchmark a constrained compiler/VM against the reference semantics before proposing one.
10. **Licensing:** Glow's exact redistribution requirements and any copied source snippets must be checked before source integration; public talks are reference material, not a license to copy Naughty Dog production code.

## 20. Design verdict

`P03_D01_DETAILED_DESIGN_PROPOSED / GLOW_PASS_IR_AND_FIXED_POINT_LINEAGE / UNCHARTED_TEMPORAL_DIRECTOR_LINEAGE / ALIEN_WORKS_MACRO_COMPARATOR / GERBIL_NATIVE / DATA_FIRST / PURE_COMPILER_AND_MOCK_RUNTIME / NO_CUSTOM_VM / NO_PROPRIETARY_PORT / OTHER_EDITIONS_UNTOUCHED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED`

**Conclusion:** P03-D01 should preserve the inspectability of Glow's multi-pass compiler and the expressive temporal state/director ideas reported by Naughty Dog, while explicitly preventing incomplete dataflow convergence, ambiguous event ordering, stale timers, uncontrolled loops and accidental execution of external effects. It remains one specialized reserve, not the new universal foundation.

## Primary source anchors

- Current LFR-C02 concept: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/concept/LFR-C02-source-lineage-gerbil-reserve-concept-2026-11-12.md
- Published P01-D01 and P02-D01: https://github.com/vietcong73/alien-works/tree/audit/lisp-foundation-reserve/docs/design
- Existing Glow source audit: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects/01-glow-source-code-review.md
- Existing Naughty Dog public-evidence review: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/naughty-dog-uncharted-lisp-family-review.md
- Glow compiler passes: https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/passes.ss
- Glow pass registry / layers: https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/multipass.ss
- Glow checkpoint construction: https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/checkpointify/checkpointify.ss
- Glow liveness: https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/liveness/checkpoint-liveness.ss
- Glow projection: https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/project/project.ss
- GDC 2008 Uncharted: https://media.gdcvault.com/gdc08/slides/S6636i1.pdf
- GDC 2009 Uncharted 2 state scripting: https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf
- Gerbil macros: https://gerbil.scheme.org/reference/gerbil/prelude/macros.html
- Gerbil language overview: https://gerbil.scheme.org/guide/intro.html