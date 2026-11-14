# LFR-K1-D01 | Opt-in Generalized Mechanisms: Cross-Domain Architecture

**Document class:** Source-grounded detailed design proposal and mechanism-promotion framework  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve, LFR-C02  
**Foundation dependency:** `LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md` (published; design-only)  
**Implementation language:** Gerbil Scheme for new reference mechanics; other-language clients only through separately qualified adapters  
**Status:** `CROSS_DOMAIN_DESIGN_PROPOSED / OPEN_MECHANISM_REGISTRY / NO_RUNTIME_PROOF / UNIMPLEMENTED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Scope:** K1 design only. Published under `docs/design/` on `audit/lisp-foundation-reserve` with owner approval; no changes to K0, P01–P07, preserved sources, original games, external libraries, operating systems, or runtime.  

> **Decision:** K1 is an **open-ended, opt-in portfolio of qualified mechanisms** that can serve different classes of systems: games, reinforcement learning, multi-agent coordination, procedural worlds, simulation, control, scientific computing, quantitative risk, information systems, and applications not yet identified. The seven specialist editions are **research inputs, not the boundary of the design space**. A named similarity does not qualify a shared mechanism.

## 0. Correction to the design scope

The previous narrow framing around eight categories in LFR-C02 is **an initial candidate inventory**, not a closed taxonomy, mandatory module list, application roadmap, or upper bound on K1. It misses independently established abstractions from game engines, RL environment interfaces, multi-agent systems, physical co-simulation, control loops and numerical research. Their omission would falsely turn an extensible meta-foundation into a summary of seven earlier drafts.

K1 must accomplish two complementary things:

1. Provide **small, separately selectable** reusable mechanism profiles with exact semantics, exclusions, evidence and versioned interfaces.
2. Provide an **extensible mechanism admission and composition procedure** so new domains can contribute candidates without changing K0 or assuming a single execution ontology.

No candidate is declared production-ready simply because an upstream program demonstrates a similar technique. Design acceptance, mock-model qualification, real-adapter qualification and release qualification are different claims.

### 0.1 Application-space coverage, intentionally non-exhaustive

- **Interactive games / creative systems:** entities, scene hierarchy, resource ownership, scripted behaviors, event timing, graphics/audio bindings, procedural content, physics boundaries and live authoring.
- **Reinforcement learning:** observation/action spaces, reward or cost signals, policy evaluation, episode boundaries, exploration, off-policy datasets, partial observability and train/evaluate separation.
- **Multi-agent environments:** turn-taking, simultaneous action, dynamic membership, asymmetric information, policy sharing, conflict, coordination and adversarial interaction.
- **Control / robotics:** supervisory authority, state estimation, time bases, safety constraints, external actuation and hardware/OS boundaries; real-world safety is not inherited from a simulator.
- **Scientific and numerical computing:** typed models, coupled discrete/continuous dynamics, units/precision, solver ownership, experiment provenance and reproducibility limits.
- **Quantitative risk / decision systems:** scenario graphs, historical/synthetic evidence, stress propagation, uncertainty, data revisions, constraints and audit-friendly decisions. No trading or risk model is executed by K1.
- **Workflow / infrastructure / applications:** dependency graphs, process/service transitions, databases, UI contracts, retrieval/memory, external APIs, failure reconciliation, bounded resources and authorization.

These are **targets for capability coverage**, not mandatory runtime dependencies or a promise that one implementation can replace their native tooling.

## 1. Evidence ledger: two initial game-oriented references, projects, Gerbil docs, and new comparators

**Evidence tiers:** `SOURCE` directly inspected pinned code; `PRESENTATION` primary vendor/public design material without source; `DOC` official language/framework/standard documentation; `DESIGN` LFR proposal not implementation; `OPEN` application hypothesis or capability gap. Evidence about an API's existence is not evidence that K1 reproduces its behavior.

### 1.1 Foundational reference systems

**Alien-Works** (`borodust/alien-works@86381f8bce44a396347cbe8de70349be5b43ece1`, `SOURCE`): Common Lisp-native integration, macros, resource scopes, callbacks, rendering queues and domain-specific material/shader definitions. **Reusable questions:** lifetime and affinity, generated-code contracts, resource budgets, staged domain DSLs. **Non-transfer:** Alien-Works is a framework, not a released game; its coupled native dependency graph, unqualified runtime, unresolved dependency closure and documented source defects are not imported.

**Uncharted 2 / Naughty Dog** (GDC 2008/2009 public talks, `PRESENTATION`): Scheme-authored gameplay/data semantics, state scripting, directors, bounded native hooks and temporal coordination. **Reusable questions:** authored transitions vs native execution, bounded domain authority, traceable timing and scripted orchestration. **Non-transfer:** no access to proprietary production code, no invented project source tree, no proprietary VM port and no assumption that the exposed talks specify a universal runtime.

### 1.2 Associated LFR research corpus

- **Native/graphics:** Trial, CEPL, Varjo, Gerbil SDL2, Gambit Raylib and comparative resource studies. Their mechanisms serve as portability counterexamples as much as inspiration.
- **Declarative systems:** GNU Guix/Guile and GNU Shepherd: graphs, staged descriptions, generations, services and process life cycles. These are *research* sources, not runtime dependencies or an automatic Guile bridge.
- **Semantic programming:** Glow compiler/IR and Uncharted public evidence, plus P03-D01; source-level liveness/fixed-point concerns are retained as negative test material.
- **Governed effects:** Gerbil Coding Harness and P02-D02; typed authorization decisions are not OS-level containment.
- **Memory/evidence:** AutoContext, Personal AI CLI and P04-D01; authoritative durable storage is P04-D01-specific, never a universal guarantee of K1.
- **Search and causal histories:** Chess AI, Guix generations, Git graph and P03-D02; scoped undo, historical branch heads and real-world causal identification remain distinct.
- **UI/interop:** GloUI and P01-D02; entity identity, task-truth vs UI-observation, typed messages and effect permissions.
- **K0:** six published contract families (identity/version, schema/validation, effects, authority records, ownership/lease, results/receipts) define the only proposed mandatory contract dependency. They do not execute anything.

The seven P01–P07 designs have **not been implemented or independently runtime-qualified** as a group; K1 does not count them as executed proofs.

### 1.3 Additional external primary and official references

- **Gerbil language and macro architecture (`DOC`):** [Introduction, macros, phases, interfaces and custom preludes](https://gerbil.scheme.org/guide/intro.html), [interfaces](https://gerbil.scheme.org/reference/std/interface.html), [FFI guide](https://gerbil.scheme.org/guide/ffi.html), [standard test](https://gerbil.scheme.org/reference/std/test.html), [build tooling](https://gerbil.scheme.org/reference/dev/build). Phase separation does not sandbox macros; unchecked interface methods bypass some checks.
- **Bevy ECS (`DOC`):** [Entities, Components and Systems](https://bevy.org/learn/quick-start/getting-started/ecs/). This is an **organizational/processing model**, not universal state semantics or a reason to place an ECS into K0.
- **Godot (`DOC`):** [SceneTree](https://docs.godotengine.org/en/stable/classes/class_scenetree.html) and [idle vs physics processing](https://docs.godotengine.org/en/stable/tutorials/scripting/idle_and_physics_processing.html). Game frames and fixed physics ticks are not interchangeable with RL episodes or external control deadlines.
- **Gymnasium (`DOC`):** [Env API](https://gymnasium.farama.org/api/env/). `step()` has observation, reward, `terminated`, `truncated`, information; termination and external truncation cannot be silently collapsed into one flag.
- **PettingZoo (`DOC`):** [AEC API](https://pettingzoo.farama.org/api/aec/) and [Parallel API](https://pettingzoo.farama.org/api/parallel/). Agent turns and simultaneous actions have different admission/order rules, and not all AEC games can be losslessly converted to parallel execution.
- **RLlib (`DOC`):** [multi-agent environments and agent-to-policy mapping](https://docs.ray.io/en/master/rllib/multi-agent-envs.html). Agent identity is not policy identity; policy sharing and policy updates need distinct versioning.
- **FMI 3 (`DOC`):** [Functional Mock-up Interface specification](https://fmi-standard.org/docs/main/). Model Exchange, Co-Simulation and Scheduled Execution impose different solver/time responsibilities; a generic `step` interface cannot silently guarantee equivalence.
- **ROS 2 lifecycle (`DOC`):** [Managed nodes](https://design.ros2.org/articles/node_lifecycle.html). Controlled state changes, failures and externally supervised transitions are useful comparators, not license to implement a general robot controller in K1.

**Version and research discipline:** official links are discovery/semantic sources; implementation must pin source tags, revisions, runtime versions and dependency closures separately. No claims of benchmarked speed, machine learning performance, physical safety, deterministic GPUs or cross-platform binary compatibility are made here.

## 2. Architecture: open portfolio, not a new monolith

```text
          APPLICATIONS / EXPERIMENTS / PRODUCTS
          games | RL | multi-agent | control | science | quant | tools | ...
                     | select profiles explicitly
                     v
          K1 MECHANISM PORTFOLIO (OPT-IN)
       graph | staged IR | transition | observation | time | search
       interaction | objective | leases | bounded effects | ...
                     | contracts / independent conformance
                     v
                 K0 CONTRACT KERNEL
        identity | schema | effects | decisions | lease | result

 P01–P07 AND FUTURE SPECIALISTS: optional peers/consumers of K0/K1.
 Source vault and public reference material: research evidence ONLY.
 No runtime import from K1 into any source specimen or vice versa.
```

**Import rules:** `application -> chosen K1 profiles -> K0`, with optional specialist packages imported by the application or opt-in adapters. `K0 -> K1` is forbidden; `K1 general-purpose core -> P01–P07` is forbidden. An independently qualified K1-to-specialist integration adapter may import both public interfaces. K1 must also be usable without *any* specialist installed.

**No universal scheduler, engine, trainer, world model, policy authority, actor runtime, RPC bus, persistence provider, machine-learning framework or generic native execution dispatcher.** Different mechanisms can be composed by an application with an explicit integration contract rather than hidden orchestration.

### 2.1 Three tiers within the K1 design space

1. **Reusable semantic mechanism:** stable meaning with two unlike domain witnesses, a bounded reference model and a falsifiable oracle; eligible for a thin opt-in K1 profile.
2. **Domain-semantic specialization:** useful framework/adapter contract (such as RL episode closure, physics integrator or ECS query) that cannot be reduced to a universal law; may live in an optional K1 extension or later specialist reserve after a real use case.
3. **Concrete backend/algorithm:** renderer, solver, PPO/DQN trainer, database, OS process driver, RLlib/PettingZoo/Gymnasium wrapper, C FFI or exchange API; never automatically promoted to shared foundation.

Both (2) and (3) remain valuable. **Being outside K1's generalized core is not a rejection of the application domain.**

### 2.2 Open versioned mechanism registry (not a privileged plugin loader)

```text
MechanismManifest {
  mechanism_ref, semantic_revision, profile_kind,
  k0_contract_refs[], optional_dependency_refs[],
  source_evidence_refs[], independent_domain_witnesses[],
  input_schema_ref, output_schema_ref, effect_ceiling,
  invariants[], forbidden_equivalences[],
  resource_limits, failure_modes[], qualification_gates[],
  declared_adapters[], evidence_receipt_ref_optional
}

AdmissionStatus = INVENTORIED | DESIGN_ADMISSIBLE | MOCK_QUALIFIED
                | ADAPTER_QUALIFIED | RELEASE_QUALIFIED
                | DEFERRED | REJECTED
```

The manifest is *data*, not executable code; accepting a manifest cannot register a privileged handler. A `profile_kind` may be `PURE`, `MOCK_RUNTIME`, `EFFECT_ADAPTER`, `NUMERIC`, `STOCHASTIC`, `DISTRIBUTED` or another explicitly versioned extension. These are **assurance/operating profiles**, not a single type hierarchy or exclusive categories.

**Promotion procedure:** (i) independently inspect at least two unlike domain witnesses; (ii) state one shared semantic law and at least one counterexample to a tempting false equivalence; (iii) define a profile-specific minimal API, dependency closure and exact capability claims; (iv) build pure/mock reference oracles; (v) run cross-domain and negative tests; (vi) qualify target-specific adapters only when separately requested. A paper-level `DESIGN_ADMISSIBLE` must never be rendered `QUALIFIED`.

**Mechanism growth:** adding an M19 or an entirely new family does not revise K0 merely because the portfolio grows. A new K0 primitive requires its own explicit minimality proof and compatibility review. Mechanism version changes retain original evidence and do not retroactively validate consumers.

## 3. Candidate mechanism portfolio: not a final list, not 18 mandatory libraries

Each candidate below is a **design proposition**. None is `MOCK_QUALIFIED`, `ADAPTER_QUALIFIED` or `RELEASE_QUALIFIED` here. Some belong ultimately in specialist packages rather than the reusable K1 layer; promotion must be earned by proof and actual consumer demand. The `Mxx` numbers organize the **initial inventory only** and are not a hard maximum.

### M01 — Typed dependency and relation graphs

**Disposition:** `PURE candidate`. **Witnesses:** P02-D01 Guix/service graphs; P03-D01 pass CFGs; P03-D02 causal incidence; procedural-generation dependency graphs.  
**Transferable contract:** Graph node/edge identities, labeled relation types, cycle/closure/topological algorithms with graph-specific assumptions; never unify all graphs' semantics.  
**Explicit non-equivalence:** A service dependency cycle, a temporal cycle and a causal ancestry cycle are not the same failure.

### M02 — Staged transformation and normalized IR

**Disposition:** `PURE candidate`. **Witnesses:** Alien-Works material DSL; Uncharted Scheme authorship; Glow P03-D01 passes; GloUI P01-D02 generated bindings; Gerbil macro docs.  
**Transferable contract:** A validated data-only IR, phase locks, pass contracts, provenance and code/data separation; allow trusted macro authoring without treating serialized data as executable code.  
**Explicit non-equivalence:** Macro expansion is not authorization or proof of behavior; no imported proprietary VM.

### M03 — State transition and finite decision traces

**Disposition:** `PURE/STOCHASTIC profiles`. **Witnesses:** Uncharted state scripting; Chess AI/P03-D02; P03-D01 machine; Gymnasium; ROS lifecycle.  
**Transferable contract:** Transition contract with before/action/input/model/after, explicit nondeterministic witnesses and typed invalid steps.  
**Explicit non-equivalence:** Discrete game turn is not continuous integration; state equality is not historical identity.

### M04 — Observation and projection

**Disposition:** `PURE profile`. **Witnesses:** P01-D02 interface projection; P04-D01 retrieval observables; Gymnasium/RL partial observation; control state estimators.  
**Transferable contract:** Separate latent/model state, observable view, observer identity, source locks, staleness, missingness and admissible equivalence.  
**Explicit non-equivalence:** Observation is not full state or proof of causality; reward is not an observation by default.

### M05 — Logical/physical/simulation time contracts

**Disposition:** `PURE + adapter profiles`. **Witnesses:** Uncharted directors; P03-D01 timers; Godot variable render/fixed physics ticks; FMI; ROS lifecycle.  
**Transferable contract:** Clock domain IDs, event ordering, simulation stepping, timestamp uncertainty and explicit synchronization/adaptation.  
**Explicit non-equivalence:** No conversion from logical step to wall-clock deadline or between solver integrators without a profile.

### M06 — Bounded exploration and candidate generation

**Disposition:** `PURE candidate`. **Witnesses:** Chess AI/P03-D02 search; procedural worlds; RL exploration; risk scenario generation; P03-D01 analysis.  
**Transferable contract:** Frontiers, budgets, completeness flags, candidate witnesses and objective-specific search oracle.  
**Explicit non-equivalence:** Search is not an optimizer, PPO trainer, proof of optimality, or complete enumeration by default.

### M07 — Multi-agent interaction contracts

**Disposition:** `DOMAIN-SEMANTIC candidate`. **Witnesses:** PettingZoo AEC/Parallel; RLlib policy mapping; Uncharted multi-object directors; adversarial games.  
**Transferable contract:** Agent membership, turn scheduling vs simultaneous batches, legal action masks, individualized observations and results.  
**Explicit non-equivalence:** Agent identity != policy identity; AEC-to-parallel conversion may change semantics.

### M08 — Objectives, costs and feedback contracts

**Disposition:** `DOMAIN-SEMANTIC candidate`. **Witnesses:** Gymnasium rewards; optimization/control constraints; game scoring; quant-risk objectives.  
**Transferable contract:** Named objective components, units, discount/horizon policy, reward/cost separation and evidence-bound evaluations.  
**Explicit non-equivalence:** Reward != utility != physical safety constraint != authorization decision.

### M09 — Resource scope and lease operations

**Disposition:** `MOCK + backend profiles`. **Witnesses:** Alien-Works/Trial; P01-D01 foreign handles; P02-D01 process handles; FMI model instances.  
**Transferable contract:** Acquisition/ownership/generation/release intent, scoped cleanup ordering, stale handle and partial-init records.  
**Explicit non-equivalence:** Mock cleanup != real native destructor/ABI safety or GPU affinity.

### M10 — Bounded operation and effects interface

**Disposition:** `MOCK + backend profiles`. **Witnesses:** P02-D02 command executor; Alien-Works render queue; P02-D01 supervised child; robotics actuators.  
**Transferable contract:** Declared budgets, explicit admission, capability checks, async status and receipts through injected executor contracts.  
**Explicit non-equivalence:** Accepted != started != completed != reversible; no universal handler or OS sandbox.

### M11 — Commit, compensation and reconciliation contracts

**Disposition:** `MOCK + backend profiles`. **Witnesses:** P04-D01 durable commit; P02-D02 staged edit; P03-D02 branch commit; Guix rollback limitation.  
**Transferable contract:** Separate pure transaction, durable atomic commit, compensating effect and unknown external outcome; scope guarantees individually.  
**Explicit non-equivalence:** Undoing a simulation step != reversing a financial trade or physical actuator command.

### M12 — Checkpoint, snapshot and history reconstruction

**Disposition:** `PURE + durable-adapter profiles`. **Witnesses:** Guix generations; P03-D02 causal histories; P04-D01 durability; RL rollouts.  
**Transferable contract:** Explicit snapshot closure, revision locks, stream checkpoints, lineage, replayability grade and durable anchor contract.  
**Explicit non-equivalence:** State snapshot != environment reset, exact replay, empirical counterfactual or crash durability.

### M13 — Structured interop and semantic adapters

**Disposition:** `PURE + client-adapter profiles`. **Witnesses:** GloUI/P01-D02; P03-D01 generated artifacts; Gymnasium/PettingZoo; FMI interfaces.  
**Transferable contract:** Typed messages, schema migration, parity oracles, ownership and version negotiation between chosen implementations.  
**Explicit non-equivalence:** A protocol shape match does not ensure semantic equivalence or effect permission.

### M14 — Experiment/policy lifecycle

**Disposition:** `DOMAIN-SEMANTIC candidate`. **Witnesses:** RL policies and rollouts; quantitative backtests; simulation experiments; P03-D02 rule revisions.  
**Transferable contract:** Separate policy model, environment revision, evaluation/learning data, RNG streams, dataset partitions and claim receipts.  
**Explicit non-equivalence:** Training success != validation success; seeds don't certify deterministic hardware.

### M15 — Constraints, supervisors and admissibility

**Disposition:** `PURE + qualified-domain profiles`. **Witnesses:** P02-D02 authorization; P03-D02 synthesis constraints; control viability; RL action masks.  
**Transferable contract:** Named invariant, admissibility verdict, fallback/refusal and independent policy/authority boundaries.  
**Explicit non-equivalence:** Constraint satisfiability, action legality, security permission and real-world safety are distinct claims.

### M16 — Observability and provenance traces

**Disposition:** `PURE candidate`. **Witnesses:** P02-D02/P04-D01 receipts; P01-D02 async task truth; P03-D02 ancestry; RL trajectories; game traces.  
**Transferable contract:** Typed events, correlation/cause IDs, model/data versions, partial unknowns, redaction and evidence scope.  
**Explicit non-equivalence:** Trace links are not automatic causal identification; observation is not a durable log.

### M17 — Numeric domains and data validity

**Disposition:** `DOMAIN-SEMANTIC candidate`. **Witnesses:** GloUI token decimals; scientific/physics units; RL reward precision; financial risk data.  
**Transferable contract:** Units/dimensions, exact decimal vs float profiles, missingness, staleness, NaN/Inf and conversion witnesses.  
**Explicit non-equivalence:** No universal floating-point determinism, dimensional inference, or financial valuation assumptions.

### M18 — Coupled simulation and model boundaries

**Disposition:** `DEFERRED advanced profile`. **Witnesses:** FMI Model Exchange/Co-Simulation/Scheduled Execution; game physics; control numerical plants.  
**Transferable contract:** Typed coupling points, solver ownership, time-step negotiation, rollback capability and error bounds where qualified.  
**Explicit non-equivalence:** Do not force physical ODE/DAE, discrete event, and episode stepping into a single integrator.

## 4. Compositional semantics for games, RL, multi-agent, control and scientific systems

### 4.1 Do not define one universal `step()`

```text
Pure transition:    T(model, state, action, sealed_input) -> StepResult
Observation:        O(observer, model, state, view_scope) -> ObservationResult
Feedback:           F(objective, step_receipt, context) -> FeedbackResult
Temporal admission: A(time_profile, pending_events) -> OrderDecision
Effect dispatch:    E(driver, qualified_intent, authority) -> EffectReceipt

Each operation can be absent; each has separate semantics and authority.
```

A *gameplay update*, *physics integration*, *RL episode step*, *multi-agent simultaneous turn*, *event-driven director*, *OS process lifecycle* and *hardware control tick* have overlapping data descriptions but **different transition laws**. Their data may flow through common K0 contracts and selected K1 profiles; that does not license a shared runtime scheduler or equivalent event order.

### 4.2 RL environment contract: optional extension, not mandatory K1 vocabulary

```text
EnvironmentProfile {
  environment_ref, model_revision, observation_schema,
  action_schema, stochasticity_profile, reset_contract,
  termination_contract, truncation_contract, safety_constraints_optional
}
EpisodeStep {
  episode_ref, step_index, observation_ref,
  action_ref, action_admissibility, next_observation_ref,
  reward_optional, costs_optional, terminated, truncated,
  environment_revision, rng_stream_ref_optional, witness, status
}
```

- `terminated` is a task-defined terminal condition; `truncated` is an externally imposed or profile-defined cut. **Never** convert both to one scalar `done` where bootstrapping/evaluation needs the distinction.
- `state` can be latent/inaccessible. `observation` is relative to a declared actor and observer; do not require a fully observable Markov state.
- A reward is not automatically utility, risk, fairness, viability or permission. Cost vectors and hard constraints are different objects with different admission criteria.
- Environment revision, policy revision, learner update revision and evaluation corpus revision must be distinguished. A policy is not an agent instance.
- `reset` may resample initial state; it is not causal rollback of external effects or a proof of deterministic replay.
- A rollout can be synthetic, simulated, offline, shadow-mode or deployed. Evidence and effect boundaries follow the profile, not an algorithm's marketing label.
- Seed locks alone do not guarantee reproducibility under nondeterministic GPU kernels, asynchronous workers, external data or floating-point differences.

### 4.3 Multi-agent action and information semantics

```text
InteractionProfile {
  active_agent_ids[], agent_generation_map,
  policy_mapping_ref, observation_scope_per_agent,
  turn_mode: AEC | SIMULTANEOUS | DECLARED_CUSTOM,
  legal_action_contract, join_leave_rules,
  global_step_boundaries, ordering_witness
}
```

AEC updates and simultaneous batches are **not interchangeable** merely because both have dictionaries of agent actions. Require a conversion witness specifying atomicity and observation timing, or reject conversion. Explicitly test disappearing agents, agents sharing one policy, partial observations, invalid action masks, agent identity reuse, contested resources and order-sensitive conflicts.

### 4.4 Games and procedural worlds without one mandated ECS

A game may use an ECS, a scene tree, object hierarchies, pure functional world snapshots, state-script directors or a mixture. K1 may expose identity/projection/graph/transition profiles to any of them, but **does not require an ECS**. Entity, component, actor, scene node, renderer object, simulated particle and RL agent are distinct roles. Ownership, scheduling, visibility and mutation capabilities vary by system.

Procedural world generation may use a staged generator description, immutable seeds, partial materialization, dependency graphs, bounded search and rollback-capable local state. K1 must not import a terrain renderer, 3D engine, shader compiler or graphics API to support these semantic mechanisms.

### 4.5 Continuous-time, hybrid and external control

- Distinguish **simulation time**, **logical event order**, **monotonic wall interval**, **render frame time**, **physics tick time** and **hardware time**.
- Solver-owned continuous integration (FMI Model Exchange), simulator-owned stepping (Co-Simulation) and externally triggered partitions (Scheduled Execution) have different obligations. A model-coupling adapter must declare which it supports.
- A supervisory decision may constrain a proposed action but does **not** prove a real actuator is isolated or a plant remains safe. Real-world control requires domain safety models, calibrated observations, deadlines, isolation and independent hardware-specific evidence.
- Stochastic model transitions, nondeterministic execution, unknown disturbances and uncertain sensors are distinct from the ability to replay a sealed deterministic computational trace.

### 4.6 Quantitative risk, research and other computational applications

Scenario engines and quantitative research can reuse graph/IR, data revisions, stochastic seeds, objective/constraint descriptions, evidence receipts, bounded exploration and model comparison. A scenario lineage graph proves **computational provenance**, not financial causation, market-price correctness or a tradable strategy. Historical claims need empirical identification and independent data qualification; simulated/stress scenarios are labeled separately.

Likewise, scientific experiments can reuse numerical validity, model coupling and observational provenance without importing RL trainers or game schedulers. New application classes may propose additional K1 mechanisms after repeating the same admission process.

## 5. Code–data duality and metaprogramming: open-ended but governed

Lisp metaprogramming matters here because profiles, transformation passes, constraints and adapters can be **represented and inspected as data before generating or executing code**. This does not mean K1 should become a privileged macro system or an unrestricted code interpreter.

```text
Trusted Gerbil authoring / inert data descriptor
          -> schema validation
          -> canonical, typed MechanismIR
          -> cross-domain invariant and effect analysis
          -> versioned transformation passes
          -> candidate adapter/spec output
          -> independent conformance + authority qualification
          -> opt-in execution through qualified domain backend
```

- **Phase and trust separation:** trusted macros may run at compile time; untrusted serialized profiles are inert and cannot invoke `eval`, load new code, gain capabilities or choose unrestricted FFI handlers.
- **IR/code identity:** language/toolchain lock, macro/expander version, schema revisions, source inputs and generated artifacts must be distinguished. A source-text hash is not a semantic equivalence proof.
- **Meta-profiles can evolve:** a profile may propose a new version of its own constraints, adapter or qualification rules. It cannot approve its own privileged capability, elevate K0 authority records or retroactively validate evidence.
- **One small trust boundary:** K0 records do not run policy; K1 registration does not run code; a domain owner/adapter verifies policy and exact effects independently.
- **No universal law of composition:** joining histories, combining two policies, unioning entity graphs, coupling numerical solvers and merging durable transactions have different algebraic properties. Associativity, commutativity, monotonicity, reversibility and idempotency are **profile-specific hypotheses**.

**Illustrative DSL, not compiled Gerbil syntax:**

```scheme
(define-mechanism-profile adaptive-simulation
  (requires (k0 identity schema result))
  (composes (state-transition observation objective-signal))
  (time-domain simulated)
  (stochastic-inputs explicitly-sealed)
  (authority (no-external-actuation))
  (qualification (cross-domain-fixtures negative-cases)))
```

This is declarative authoring intent only. It does not claim a ready-to-run macro, a `define-mechanism-profile` implementation or a runtime security guarantee.

## 6. Composition, contracts and capability ceiling

### 6.1 Profile composition is conditional and typed

```text
Compose(M_i, M_j, bridge, context)
  -> ADMITTED(composed_profile, witness)
   | CONDITIONAL(missing_obligations)
   | REJECTED(counterexample)
   | UNVERIFIED(evidence_gap)
```

Composition checks semantic profile, input/output schemas, identity/version, time domain, effect ceilings, authority source, numeric units, state ownership and evidence level. The result is **not unique** unless the bridge profile proves uniqueness. Different admissible composition orders may produce different results; no default associativity.

Candidate adapters may compose the same mechanisms differently for a turn-based game, robotic test harness and offline RL environment. No global controller automatically decides the universally correct composition.

### 6.2 Five different directions of dependency

1. **LFR source influence** is references/evidence only: research sources never load into K1 by provenance association.
2. **K0 record dependency** is minimal and monotonic: a K1 profile uses existing K0 public contracts only.
3. **K1-to-K1 composition** is explicit and opt-in; transitive imports are qualified and manifest-visible.
4. **Domain adapters** can import selected K1 and specialist interfaces; they do not grant new K1 capabilities automatically.
5. **Application authority and physical effects** remain external to profile admission and require separate execution gates.

### 6.3 Concrete failure semantics

- `UNSUPPORTED_PROFILE`: required semantic or execution profile absent, no silent fallback that changes meaning.
- `SCHEMA_MISMATCH`: incompatible payload or revision, not coerced to success.
- `TIME_DOMAIN_CONFLICT`: physics tick or simulation clock wrongly used as a deadline authority.
- `OBSERVABILITY_GAP`: requested state/equality claim exceeds observation available.
- `AGENT_ORDER_CONFLICT`: attempted conversion between turn-taking/simultaneous order without witness.
- `UNVERIFIED_STOCHASTIC_REPLAY`: incomplete RNG/model/external input closure.
- `BUDGET_EXHAUSTED`: bounded enumeration incomplete, not proof of nonexistence.
- `AUTHORITY_NOT_GRANTED`: valid request or legal RL action is **not** permission for external effect.
- `UNKNOWN_EXTERNAL_EFFECT`: no unconditional retry of non-idempotent action.
- `DURABILITY_UNQUALIFIED`: in-memory snapshot not a committed durable checkpoint.
- `NUMERIC_DOMAIN_MISMATCH`: units/precision/NaN/Inf incompatibility.

## 7. Example cross-domain contracts, not software commitments

### 7.1 Same graph algorithm, different graph ontology

P02-D01 dependencies, P03-D01 compiler graphs, P03-D02 causal events and procedural-generation dependency graphs can reuse a **parameterized graph traversal kernel** where they share an actual law. Their edge meanings and cycle acceptance policies remain independent. Tests must demonstrate that a cycle admissible in a control-flow graph is refused where a causal-history DAG forbids cycles, without modifying the graph algorithm.

### 7.2 Game director, RL agent and robotic controller

All three may consume an observation and propose an action, but an Uncharted-style director can coordinate scripted encounters, a learned RL policy can choose actions based on observations, and a robotics supervisor can forbid unsafe actuator commands. A compatible observation/action envelope is reusable; rewards, scheduler ownership, safety authority and physical actuation are **not interchangeable**.

### 7.3 Procedural world and scenario analysis

A procedural terrain generator and a financial scenario generator can both use staged transformations, versioned stochastic seeds and bounded exploration. Visual terrain consistency is not a risk-model calibration metric; a shared stage/receipt implementation qualifies only its specified semantics, not either model's predictive validity.

### 7.4 Multi-agent and concurrent workflows

A multi-agent game's simultaneous update and a P02-D02 concurrent tool workflow both need ordering, actor identity and conflict handling. But game transition atomicity does not imply a transaction spanning two files, network services or physical actuators. The shared primitive can report a conflict and retain evidence; specialized adapters decide effect commit/compensation.

## 8. Implementation packaging if separately authorized

**No K1 source has been written under this D01.** The following is a candidate workspace layout, not a list of required imports:

```text
foundation/
  k0/                         # existing proposed minimum public contracts
  k1/
    registry/                 # inert manifests, compatibility, admission metadata
    pure/
      graph/                  # parameterized graph algorithms
      transform/              # staged typed IR, no arbitrary evaluation
      transition/             # pure step and explicit witnesses
      observation/            # actor-relative observations
      time/                   # typed time domain decisions
      evidence/               # in-memory provenance and uncertainty
    optional/
      explore/                # bounded search and candidate enumeration
      interaction/            # agent order, mask and membership profiles
      objectives/             # reward/cost/constraint descriptions
      scope/                  # mock lifetime and optional native adapters
      bounded-effects/        # narrow qualified executor protocols
      recovery/               # compensation/reconstruction contracts
      protocols/              # adapter conformance and codegen bridges
      adaptive/               # policy experiment lifecycle
      numeric/                # unit/precision profiles
      coupling/               # time/solver-specific adapters
    fixtures/
      game/
      rl-single-agent/
      rl-multi-agent/
      procedural-simulation/
      control/
      scientific-coupling/
      quant-risk/
      tools-and-data/
    qualification/
      conformance/
      counterexamples/
      executed-evidence/      # empty until tests run
```

Folder names express potential ownership only; packages are not built en masse. Prefer first building a small pure reference slice with real cross-domain test oracles, then add one opt-in mechanism at a time. An application must be able to import `K0` or one selected K1 module without importing unrelated domain backends. A future reserve repo requires explicit owner approval; this document does not create one.

## 9. Qualification policy: tests must fail when abstraction overreaches

### 9.1 Promotion and truthfulness

No `PASS` is claimed. A future report must name exact Gerbil/Gambit versions, OS/architecture, source and test-fixture revisions, test counts, backend profiles, redacted trace/receipts, independent oracles and all unsupported claims. A two-domain mock proof establishes reusable **mock semantics only**; a real game engine, training experiment, controls interface, DB or network adapter qualifies separately.

No fixed set of 18 mechanisms is mandated. A candidate may be promoted, deferred, split into several semantic profiles or rejected. K1 is open to new mechanisms discovered after the source register; all must pass the same process without importing application-specific assumptions into K0.

### 9.2 Sixteen proposed gates (Q0–Q15)

| Gate | Subject | Required evidence |
|---|---|---|
| Q0 | Dependency isolation | Core-only Gerbil build; selective imports; no transitive concrete engine/agent/database/native stack |
| Q1 | Manifest admission | Two independent witnesses, falsifiable laws, versioned profile, no code execution on registry entry |
| Q2 | Graph and staged IR | Correct reusable graph/transform primitives and negative ontology-transfer cases |
| Q3 | State transition | Deterministic and stochastic reference models, invalid actions, model-state vs history separation |
| Q4 | Observation, episodes, agents | Termination vs truncation, partial observation, turns vs simultaneous batching, agent vs policy identity |
| Q5 | Time and scheduling | Explicit logical/fixed/variable/continuous/wall time; no silent cross-clock assumptions |
| Q6 | Search and optimization bounds | Correct incomplete/complete statuses, no unearned optimality, quota enforcement |
| Q7 | Scope and bounded effect adapters | Mock lifetime/ownership; authority recheck; real adapter needs its own tests |
| Q8 | Checkpoint, durable commit and compensation | In-memory vs durable guarantees separated, partial failure/unknown replay semantics |
| Q9 | Structured protocols | Schema/version and adapter parity on explicit scope, invalid data denied |
| Q10 | Evidence and observability | Typed, redacted receipts; uncertainty and computational vs empirical causality labels |
| Q11 | Adaptive experiment/policy | Policy/model/data revisions, stochastic streams, train/test isolation, no performance overclaim |
| Q12 | Numeric/scientific/control | Units, precision, solver/time ownership, hard constraints and physical actuation ceiling |
| Q13 | Broad cross-domain validation | At least game + RL + one independent third domain fixtures; multiple pairwise contrasts |
| Q14 | Security/metaprogramming | Untrusted declarations inert, no code execution/authority elevation, version-lock drift checks |
| Q15 | Release qualification | Manifest+test trace completeness, profile-specific status and no unexecuted test presented as passed |

The broader Q13 fixture coverage is a **design acceptance goal for the K1 portfolio**, not an assertion every individual mechanism must be imported by a game, RL, control and quant system simultaneously. Each mechanism must individually establish its own distinct two-domain witness obligation.

### 9.3 Proposed falsification register: 96 tests

IDs `K1-T01` through `K1-T96` are **future tests only**, not executed. Tests are grouped into twelve independently reportable areas, with eight explicit negative/positive fixtures per area.

#### Manifest, scope, dependency and admission

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T01 | Import one pure profile with K0 only | No Pxx/native/LLM/graphics imports |

| K1-T02 | Load K0 without any K1 profiles | No automatic registry or module initialization |

| K1-T03 | Manifest cites one domain only | No cross-domain qualification |

| K1-T04 | Manifest lacks negative counterexample | Admission withheld with typed reason |

| K1-T05 | Version changes while semantic revision label remains unchanged | Different artifact/lock; requalification required |

| K1-T06 | Manifest tries to execute embedded Scheme | Data-only inspection, no eval or effects |

| K1-T07 | New M19 candidate registered | No K0 or preexisting profile mutation |

| K1-T08 | Two competing profile declarations share name | Namespace/revision conflict reported, not silently replaced |



#### Graph and transformation semantics

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T09 | Topological order of valid dependency DAG | Reference oracle matches and graph identity preserved |

| K1-T10 | Cycle in causal history graph | Rejected under ancestry DAG rules |

| K1-T11 | Cycle in an admitted program control-flow graph | Handled by fixed-point/CFG algorithm, not blanket refusal |

| K1-T12 | Uncharted-style script descriptor reaches canonical IR | No native engine executed |

| K1-T13 | Alien-Works-like material descriptor through same staging API | IR retains domain type and backend-specific fields separately |

| K1-T14 | Macro expander version drift changes artifact | Stale qualification is rejected |

| K1-T15 | Untrusted input contains code-shaped forms | Parsed as inert data or rejected |

| K1-T16 | Lossy transform used as invertible migration | Typed refusal or declared loss witness |



#### Pure and stochastic transition profiles

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T17 | Same sealed deterministic inputs and revision | Reference trace equivalent under declared oracle |

| K1-T18 | Missing stochastic seed/external observation | No deterministic replay claim |

| K1-T19 | Game legal action vs control action with same bytes | Different domain schemas prevent accidental equivalence |

| K1-T20 | Invalid state/action pair | No committed transition; typed diagnostic |

| K1-T21 | Two distinct histories reach identical state payload | History identity preserved |

| K1-T22 | Partial observation presented as full state | Refused as exceeding observer scope |

| K1-T23 | Stochastic transition returns multiple candidate outcomes | Multiplicity and qualification retained |

| K1-T24 | Simulation step after terminal episode without declared reset | Rejected according to environment profile |



#### RL and multi-agent semantics

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T25 | Task terminal without external truncation | terminated=true, truncated=false retained |

| K1-T26 | Time limit truncates unfinished episode | terminated=false, truncated=true retained |

| K1-T27 | AEC turn sequence requested as parallel without witness | Conversion refused |

| K1-T28 | Qualified simultaneous action group | Atomic batch semantics/order witness retained |

| K1-T29 | Agent disappears mid-episode | Per-agent completion and membership update valid |

| K1-T30 | Two agents share policy revision | Agent IDs stay distinct from shared policy ID |

| K1-T31 | Illegal action mask value | Rejected or domain-declared penalty, not unconditional execution |

| K1-T32 | Different observations for same latent state | Observer-specific identity and provenance retained |



#### Time and event scheduling

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T33 | Variable render frame vs fixed physics tick | Different clock domains, no implicit equality |

| K1-T34 | Logical timer compared to wall deadline | Explicit adapter required |

| K1-T35 | Simultaneous timestamp but ordered causal events | Ordering witness separate from timestamp |

| K1-T36 | Out-of-order event arrival | Profile-specific buffer/rejection policy observed |

| K1-T37 | Clock wrap or nonmonotonic wall time | No silent negative elapsed duration |

| K1-T38 | FMI Model Exchange vs Co-Simulation owner | Solver ownership negotiated or refused |

| K1-T39 | Missed control deadline | Typed missing-deadline observation, no simulated safety success |

| K1-T40 | Bounded queue saturation | Backpressure receipt and no silent unbounded growth |



#### Exploration and optimization

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T41 | Exhaustive small state graph | Reference completeness and result count correct |

| K1-T42 | Budget cutoff during branch enumeration | PARTIAL, not impossible or optimal |

| K1-T43 | Three-way constraint conflicts despite valid pairs | Joint-admissibility rejection |

| K1-T44 | Heuristic search with unproved admissibility | No optimality certificate |

| K1-T45 | RL exploration trajectory used as model validation | Cannot claim unbiased off-policy/real-world proof by default |

| K1-T46 | Procedural generator reuses same seed with different model | Version mismatch prevents false equivalence |

| K1-T47 | Two valid candidate outcomes discovered | Both retained, no silent winner |

| K1-T48 | Search objective changed without requalifying transposition cache | Context-aware invalidation |



#### Resource lifecycle and bounded effects

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T49 | Resource creation fails after partial acquisition | Mock scope releases exactly eligible owned handles |

| K1-T50 | Borrowed handle attempts destroy | Refused and owner unchanged |

| K1-T51 | Stale generation token after reuse | ABA safety failure reported |

| K1-T52 | Operation accepted but worker not spawned | Acceptance not reported as execution success |

| K1-T53 | Unauthorized effect proposal with valid schema | Refused before adapter call |

| K1-T54 | Cancellation after external dispatch | No automatic undo claim |

| K1-T55 | Budget exhausted in operation queue | Bounded rejection and receipt |

| K1-T56 | Native backend missing required affinity capability | No real destructor executed under unsupported assumption |



#### Commit, checkpoint and recovery

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T57 | In-memory snapshot without durable backend | No durable recovery claim |

| K1-T58 | Durable commit loses acknowledgement | UNKNOWN_ACK, reconcile before retry |

| K1-T59 | Compensation fails after partial effect | Original effect and new failed compensating effect both recorded |

| K1-T60 | P03-D02 branch checkout after live external mutation | No claim of external rollback |

| K1-T61 | Two histories share snapshot but not ancestry | Distinct historical identity retained |

| K1-T62 | Recovery with missing rule/toolchain lock | Replay denied or partial status |

| K1-T63 | Two output heads require atomic publication but backend lacks transaction | Unsupported guarantee, no false atomicity |

| K1-T64 | Old policy revision and new policy revision applied to same snapshot | Historical vs counterfactual re-evaluation distinguished |



#### Interop and semantic adapter parity

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T65 | Valid schema round-trip through two qualified mock adapters | Same declared observable with equal scope |

| K1-T66 | Incompatible enum or numeric unit on wire | Refuse, no silent coercion |

| K1-T67 | Malformed untrusted payload attempts prototype pollution | No cross-instance state leakage or execution |

| K1-T68 | Observation adapter drops failure kind | Parity test fails |

| K1-T69 | Version negotiation fails | Unsupported profile, not downgraded semantics |

| K1-T70 | FMI and Gymnasium step APIs presented as interchangeable | Bridge refused unless explicit semantics qualified |

| K1-T71 | Same event serialized with distinct agent identities | Ownership and visibility preserved |

| K1-T72 | Generated adapter uses unchecked interface method as auth bypass | Negative gate rejects public privileged path |



#### Evidence and claims

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T73 | Synthetic and historical risk evidence combined | Provenance and empirical scope kept separate |

| K1-T74 | Trace contains computational lineage only | No empirical causal claim |

| K1-T75 | Error text resembles success field | Typed result remains failure |

| K1-T76 | Receipt contains secret-bearing input | Redaction performed before persistence/export |

| K1-T77 | Worker completes without domain postcondition | No verified success claim |

| K1-T78 | Missing observation from external operation | UNKNOWN preserved |

| K1-T79 | Truncated log claimed complete | Completeness marker prevents overclaim |

| K1-T80 | Two policies with identical names but different hashes | Evidence binds exact revision and context |



#### Adaptive algorithms and experiments

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T81 | Policy updated mid-rollout | Policy revision boundaries retained |

| K1-T82 | Training data reused as independent validation | Evaluation claim blocked or labeled contaminated |

| K1-T83 | Reward scalar mistaken for hard safety constraint | Constraint profile rejects substitution |

| K1-T84 | RNG seed present but asynchronous GPU execution unpinned | No deterministic output claim |

| K1-T85 | Agent-to-policy mapping changes between episodes | Mapping revision recorded |

| K1-T86 | Nonstationary environment revision ignored by sampler | Dataset compatibility failure |

| K1-T87 | Off-policy trajectory lacks action probabilities when needed | Estimator claim unsupported |

| K1-T88 | Algorithm wins synthetic scenario but fails live-control evidence | Synthetic success not elevated to deployed certification |



#### Scientific, numeric, control and broad domains

| Test | Falsification fixture | Required result |
|---|---|---|

| K1-T89 | Decimal asset amount with wrong scale | Exact unit/scale validation catches error |

| K1-T90 | Finite-only numeric contract receives NaN/Inf | Reject or explicit missingness classification |

| K1-T91 | Solver time step violates declared error or interval bound | Refusal or uncertainty status |

| K1-T92 | Continuous co-simulation used as discrete one-step proof | Incompatible time semantics flagged |

| K1-T93 | Robot control action has valid RL mask but no actuation authorization | Actuation denied |

| K1-T94 | Game rendering calls reuse control safety receipt | Claim scope mismatch rejected |

| K1-T95 | Financial scenario described as empirical causal estimate | Inference scope refused without identification |

| K1-T96 | New unrelated domain offers same invariant and negative case | Open registry accepts review without forcing new K0 primitive |



### 9.4 Qualification report matrix

| Claim | Minimum gate family | Explicit ceiling |
|---|---|---|
| `K1_MANIFEST_DESIGN_ADMISSIBLE` | Q1 design review | Paper-level; no execution |
| `K1_PURE_PROFILE_MOCK_QUALIFIED` | Q0, Q1 and relevant Q2/Q3/Q4/Q5/Q6/Q9/Q10/Q11/Q12, plus Q13/Q14 | Only exact mock input/oracle revision |
| `K1_EFFECT_ADAPTER_QUALIFIED` | Q0, Q1, Q7/Q8/Q9/Q12/Q14 and targeted backend fault tests | Exact OS/toolchain/backend/authority profile |
| `K1_PORTFOLIO_RELEASE_QUALIFIED` | Q0–Q15 relevant to declared modules, negative tests and independent audit | Qualified *module set only*, not every future K1 candidate |

A profile may explicitly report `DEFERRED`, `DESIGN_ONLY`, `SPEC_BLOCKED`, `UNSUPPORTED_BACKEND` or `UNQUALIFIED`. The presence of 96 proposed tests and 16 gates does not make them passed. An arbitrary new domain cannot automatically inherit a gate passed by an unrelated target.

## 10. Recommended sequencing, if separately authorized

1. **K1-00, evidence and version locks:** confirm published K0 contract versions and exact source references. Add a read-only evidence ledger for all P01–P07, Alien-Works, Uncharted, official Gerbil docs, RL/game standards and source-only/missing cases.
2. **K1-01, semantic manifest prototype:** inert manifests and admission validator with negative security/semantic cases. Do not implement privileged dynamic loading.
3. **K1-02, first pure cross-domain slice:** graph + staged IR + state/observation primitives in small Gerbil mock packages. Use one scripted game/director fixture, one single/multi-agent RL fixture and one independent control/scenario fixture.
4. **K1-03, truthfulness and feedback:** separate objective/reward/cost, task episode status, computational state/history, time-domain profiles and limited observability before any adaptive learning executor.
5. **K1-04, bounded exploration and meta-policies:** qualified candidate sets, finite search, constraint witnesses, policy revision governance; refuse global optimality claims without proof.
6. **K1-05, selective adapters:** choose real runtime/transport/solver/native/data adapter only after the application need is explicit. One adapter per qualification gate; refuse implicit broadening.
7. **K1-06, independent portfolio audit:** compare three independent domain mock consumers, attempt rejected compositions, inspect dependency closure, and record exactly which profiles remain design-only.

This order is illustrative and may change after audit evidence. The **first K1 release is not required to implement M01–M18**. The registry is deliberately open to new candidates and may also reject or split existing ones. Module selection is demand- and evidence-led.

## 11. Open decisions and explicit nonclaims

1. **Registry as documentation or runtime metadata:** start data-only; no privileged runtime plugin manager until a qualified use case justifies it.
2. **Composition granularity:** make semantic profiles smaller than a framework. Decide whether a graph algorithm, temporal coordinator or observation projection genuinely needs shared code, or just a shared K0 contract.
3. **RL environment semantics:** preserve state/observation distinction, multi-agent turn rules and termination/truncation; do not assume every task is MDP, discrete-time, scalar-reward or learnable by one algorithm.
4. **Native/game systems:** do not bind K1 to a renderer, ECS library, physics engine, graphics API or proprietary console tooling.
5. **Research/control:** no physical-safety certification, constrained RL proof, scientific-calibration claim, historical-causal estimate or financial alpha proof follows from contract compatibility.
6. **Distributed systems:** no exactly-once delivery, global ordering, consensus or cross-service ACID claims without explicitly scoped independent evidence.
7. **Metaprogramming:** no arbitrary untrusted Scheme evaluation, macro self-authorization or automatic trusted-kernel upgrade.
8. **Durability:** no assumption that an in-memory profile has crash persistence; P04-D01's independently qualified backend remains available through optional explicit composition.
9. **Licensing:** conceptual lineage does not authorize copying Alien-Works/Uncharted/external source; independently review license/compatibility for any reused code.
10. **Portability:** Gerbil/Gambit version lock and qualified target-specific adapters are required before claims of compiled portability or native resource safety.
11. **Negative results:** failed promotions and unsupported domains remain first-class evidence. Never hide failures by renaming a rejected mechanism into a broader abstraction.
12. **Scope control:** this document changes no project code, repository, active deployment, source review, P01–P07 design, K0 contract or K1 runtime.

## 12. Design verdict

`K1_D01_OPEN_ENDED_CROSS_DOMAIN_ARCHITECTURE_PROPOSED / BOTH_ORIGINAL_GAME_REFERENCES_INCLUDED / P01_TO_P07_AND_ASSOCIATED_PROJECTS_REFERENCED / OFFICIAL_GERBIL_DOCS_GROUNDED / GAME_RL_MULTIAGENT_PROCEDURAL_CONTROL_SCIENCE_QUANT_TARGETS / EIGHT_SEED_CATEGORIES_NOT_CLOSED / EIGHTEEN_INITIAL_MECHANISM_CANDIDATES / EXTENSIBLE_MANIFEST_PROMOTION / NO_UNIVERSAL_RUNTIME / K0_INDEPENDENT / 96_FUTURE_TESTS / 16_FUTURE_GATES / NO_IMPLEMENTATION / NO_QUALIFIED_RUNTIME / DOCUMENT_PUBLISHED`

> **K0 defines what a trustworthy boundary means. K1 offers independently admitted, opt-in mechanisms for reasoning and working across those boundaries. The specialist systems and applications retain their own domain semantics.**

K1 grows by demonstrated reuse and counterexample-resistant qualification. It is not constrained to seven specialist source documents, a fixed mechanism inventory, or one model of time, agent, state, feedback or execution.