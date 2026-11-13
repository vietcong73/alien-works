# LFR-P08-D01 | Game World Spacetime Decision & Causal Consequence Architecture

**Document class:** Source-grounded, implementable detailed design proposal for one specialist edition  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve, LFR-C02  
**Domain track:** `GAMING` — Gaming & Interactive Worlds  
**Classification:** Primary gaming specialization; cross-domain reuse is permitted through explicit contracts  
**Edition:** P08 — World, Scene & Spacetime Decision Systems  
**Design slice:** D01 — Spacetime Decision & Causal Consequence (forward-first)  
**Primary implementation target:** Independent Gerbil Scheme pure semantic reference core; optional integrations via explicit contracts  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P08-D01-spacetime-decision-causal-consequence-design-2026-11-13.md`  
**Status:** `DESIGN_PROPOSED / SOURCE_CONTRACTS_REVIEWED / 64_FUTURE_TESTS_SPECIFIED / 12_FUTURE_GATES / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. No historical source, game, runtime, backend, or other specialist design is modified. This publication does not authorize implementation or deployment.  

> **Decision:** P08-D01 governs how versioned world rules evaluate interventions, propagate their consequences through space and time, present imperfect evidence to actors, and commit admitted world decisions. A player's choice selects an *intervention*, never a guaranteed *consequence*. The first release is forward-only. Historical intervention, persistent successor timelines and cross-timeline travel are future opt-in semantic profiles, not a DLC design or a requirement for the base system.

## 0. Research question, limits, and intended useful capability

Can an independent Gerbil Scheme package let a host game, creative world, simulation or test environment author *causal decision rules*, run bounded previews, apply admissible decisions, and report consequences without requiring a renderer, physics engine, machine-learning model, database or a complete historical universe?

The intended capability is **rule-governed, observer-relative world decision semantics**. A minimalist world can be a small graph of regions, actors, resources and social dependencies, not a rendered 3D planet. P08-D01 qualifies on symbolic/finite fixtures before any native world generator or game client is attached.

**In scope:** world/revision references; spatial and logical-temporal references; intervention and admissibility; finite causal transition kernels; explicit exogenous inputs; conservative dependency/influence analysis; bounded propagation; incomplete inference; actor-relative observation; preview vs state admission; persistence/convergence/divergence; optional stability/collapse rule semantics; decision receipts; typed interfaces to P06, P08-D02, P09, P03 and K1.

**Not in scope:** rendering, scene storage/streaming, entity-component implementation, planet/terrain generation, voxel/mesh kernels, physics solver, RL training, NPC planner, narrative content, P06 history graph implementation, N-ary historical synthesis, real-world causal identification, untrusted Scheme evaluation, universal global scheduler, distributed transaction, save-game durability or time-travel gameplay design.

### 0.1 Why a separate D01 within P08

- P06 deals with **historical causal topology, recorded branches, reconstruction, synthesis and rule/meta-policy governance**. It does **not** own the spatial world dynamics or choose world consequences.
- P08-D01 owns **intervention -> domain-specific consequences -> observation/admission**, including futures not yet committed.
- P08-D02, designed later, will own **world/scene entities, region ownership, materialization state and accessibility**. No ECS is imposed by D01.
- P09, designed separately, will own **procedural synthesis algorithms and their performance backends**. D01 defines only generation/materialization request contracts.
- K1 M03/M04/M05/M06/M15 provide optional *generic primitives*; P08 owns the game/world-specific invariants and their composition. No K1 profile is assumed qualified or mandatory.

**Semantic separation:** history != world state != observation != predicted future != published successor. These must remain distinct identities even if their serialized contents sometimes match.

## 1. Source-lineage ledger and evidentiary ceiling

The sources below inform boundary choices; none is a claim that the proposed P08 implementation already exists or that fiction models physical time travel.

| Evidence | Directly supported lesson | P08 adaptation | What does *not* transfer |
|---|---|---|---|
| P06-D01, committed LFR specialist design | Historical ancestry, branch CAS, history immutability, policy-relative synthesis, incomplete search | Separate speculative world decisions from historical publication and use version-pinned optional P06 adapter | P08 may not own P06's DAG, merge/synthesis or historical-retention policy |
| P03-D01, committed LFR specialist design | Pure microsteps, bounded event processing, time-domain ordering, effect intents and receipts | Deterministic step boundaries and fake-clock fixtures when a temporal director is selected | P08 is not a replacement director/event scheduler |
| K0-D01, committed LFR foundation | Versioned identity, schema validation, effect descriptions, authority records, resource ownership and typed outcomes | Shared representation for exact refs, validation and receipt evidence | A validation result/authority record does not itself execute or secure anything |
| K1-D01, committed LFR mechanism portfolio | Transition, projection, time, search, constraints, history and numeric profiles | Selection of separately qualified primitives by consumer need | No mandated universal `step`, optimizer, graph ontology or scheduler |
| Naughty Dog / Jason Gregory, GDC 2009, publicly presented Uncharted state scripting | Authored state transitions and bounded game scripting coupled to native engines | An authored decision-rule IR with explicit semantics and limited native effects | No proprietary Uncharted engine code, VM or production algorithm is available for porting |
| Alien-Works source review (Common Lisp framework) | Native resources, macros, callbacks, renderer queues and backend coupling | Keep P08 independent from native lifetime, FFI and renderer threads | Native integration is not proof of safe, causal or deterministic game rules |
| Pearl, *An Introduction to Causal Inference* (2010), and Oxford SCM lecture | Interventions in a structural causal model differ from conditioning on observations; model assumptions matter | Distinguish model-defined intervention from observation and state the causal assumptions explicitly | A fictional causal model is not empirical causal identification or a proof about reality |
| Godot official SceneTree/physics-processing docs | Scene activation, frame-driven processing and fixed-rate physics updates are distinct | P08 semantic time independent of render/physics cadence | D01 is neither a scene tree nor a real-time physics loop |
| Gerbil official introduction, macro/interface/test/FFI docs | Gerbil modules, checked interfaces, phase-separated macros, Gambit interop and test harnesses | Prefer plain data/structs and pure functions; macros only trusted authoring sugar | Interfaces do not by themselves enforce authority; macro expansion is not a sandbox |

**Source anchors:**

- P06: `docs/design/LFR-P06-D01-branching-causal-reversibility-design-2026-11-13.md` (same branch).
- P03: `docs/design/LFR-P03-D01-semantic-state-dsl-detailed-design-2026-11-12.md` (same branch).
- K0: `docs/design/LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md` (same branch).
- K1: `docs/design/LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md` (same branch).
- GDC 2009: [GDC Vault presentation](https://www.gdcvault.com/play/1730/State_Based_Scripting_in_UNCHARTED__DRAKE_S_FORTUNE_and_UNCHARTED_2__AMONG_THIEVES); [Naughty Dog at GDC 2009](https://www.naughtydog.com/blog/naughty_dog_at_gdc_2009); [State-Based Scripting slides](https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf).
- Causal inference: [Pearl (2010)](https://pmc.ncbi.nlm.nih.gov/articles/PMC2836213/); [Oxford SCM materials](https://www.stats.ox.ac.uk/~evans/APTS/scm.html); [Stanford Encyclopedia](https://plato.stanford.edu/archives/spr2022/entries/causal-models/).
- Godot: [SceneTree](https://docs.godotengine.org/en/stable/tutorials/scripting/scene_tree.html); [Idle and physics processing](https://docs.godotengine.org/en/stable/tutorials/scripting/idle_and_physics_processing.html).
- Gymnasium: [Environment API](https://gymnasium.farama.org/api/env/).
- Gerbil: [Introduction](https://gerbil.scheme.org/guide/intro.html); [Interfaces](https://gerbil.scheme.org/reference/std/interface.html); [Tests](https://gerbil.scheme.org/reference/std/test.html); [FFI](https://gerbil.scheme.org/guide/ffi.html).

**Evidence status:** Published P06/P03/K0/K1 artifacts are prior *design proposals*, not passing code. Official documentation demonstrates reference API properties, not test results on the planned Gerbil toolchain. The causal formalism is a **modeling inspiration**: users can author a valid artificial world without claiming causal inference from empirical history.

## 2. Architecture and dependency boundary

```text
            APPLICATION / GAME / SIMULATION
                         |
               observation / proposals
                         v
              P08-D01 DECISION SEMANTICS
    +---------------+---------------------------+
    | world model   | intervention admission    |
    | causal rules  | bounded propagation       |
    | consequences  | actor-relative projection |
    | preview       | world-step proposal       |
    +---------------+---------------------------+
                   |             |
          optional |             | optional
                   v             v
       P06 historical          P08-D02 world / scene
       publication adapter     state & materialization
                   |             |
                   |        optional P09 generator
                   v             v
          P05 durability      graphics/physics/game host
          only if selected     never imported by pure core

  P08-D01 -> K0 contract vocabulary and *chosen* qualified K1
  pure mechanisms; no import of P06/P09/P03 internal implementations.
```

Allowed dependency direction is through **public, opt-in ports/adapters**. A base forward-only world uses no P06 installation; a history-aware game explicitly selects a P06 bridge. A small world can run purely in memory with no P08-D02/P09/backend.

Two independent roles exist:

1. **Authoritative world kernel:** versioned model and state, typed interventions, dynamic rules and committed local state.
2. **Observer interface:** a filtered view and evidence for a particular player, NPC, debugger or research agent. It cannot mutate world truth, grant permissions or leak hidden causal rules by default.

An editor/debugger may use a privileged explicit observer scope. It must not silently leak into the player-facing observation adapter.

## 3. Formal world and decision semantics

Let `W_v` be a versioned world rule model, `S_t` a canonical world state at world time `t`, `I_t` a submitted intervention, `U_t` an exogenous-input bundle, and `D_t` a bounded set of causal dependency candidates. A reference transition is *a modeled relation*, not a claim of unique physical dynamics:

```text
C_v(S_t, I_t, U_t, clock_profile, budget) -> ConsequenceSet | Refusal | Unknown
O_v(S_t, observer, scope, evidence_limit) -> Observation
A_v(S_t, I_t, policy) -> AdmissibilityVerdict
```

For a declared deterministic and fully sealed profile, `C_v` can return a unique exact next-state witness. For stochastic/nondeterministic profiles, it returns a **bounded set or a characterized distribution** with probability semantics stated and justified independently. Incomplete exploration retains a typed incompleteness flag. The model cannot pretend an unexamined world has one inevitable outcome.

**SCM-inspired optional profile:** a world author may declare structural functions for selected variables and a model intervention that overrides one assignment. This is `do_model`, *not* a general authorization mechanism and not the only possible game rule semantics. Intervening on a real causal process is different from observing a variable with a given value; causal identification of a physical system requires separate assumptions and evidence. Never use model-generated numerical probabilities as calibrated empirical odds without qualification.

### 3.1 Four non-interchangeable decisions

| Layer | Question | Output | Owned by |
|---|---|---|---|
| Syntax/schema | Is the proposal well formed? | `VALID/INVALID/UNVERIFIED` | K0 + P08 schema |
| Domain admissibility | Is this action possible according to world rules at this state? | `ADMIT/REFUSE/UNRESOLVED` | P08-D01 |
| Runtime authority | Is the caller allowed to cause this effect in this host? | bound `ALLOW/DENY/REVIEW/UNKNOWN` | Host/P04 when selected; K0 describes |
| Causal consequence | What happens if the admitted action is applied? | `ConsequenceSet`, uncertainty, and witness | P08-D01 |

An action can be valid, legal in the fictional world, and still not authorized to mutate a live host. Conversely, an authorized caller can propose a physically impossible in-world action, which P08 must reject.

### 3.2 Canonical outcome distinctions

`UNCHANGED_OBSERVABLE`: the targeted outcome is unchanged under a **named observer/comparison**, despite a different intervention or underlying world state.

`DIVERGED`: the named observer finds a substantive, witnessed difference.

`CONVERGED`: distinct reachable histories or trajectories meet a declared equivalence relation at a later comparison point; **not** identical causal ancestry.

`MULTIPLE_ADMISSIBLE`: several future candidates exist under the given state/rules/input or stochastic profile.

`UNKNOWN`: insufficient state, unresolved dependencies, missing exogenous inputs, unsupported profile or budget exhaustion; no claim that world collapsed or remained stable.

`RULE_CONFLICT`: constraints or rules cannot jointly admit a candidate under a defined checker; rejection may be local or global depending on proof scope.

`VIABILITY_FAILED`: the model has demonstrated that a candidate does not meet a **specific world-viability predicate** over a stated horizon/domain. This does not by itself imply timeline destruction.

`COLLAPSED`: **optional fictional rule**, not a generic error code. Only a fully specified collapse predicate and independently validated transition can authorize a collapse outcome. A program exception, undefined data, cycle, budget exhaustion or failed generator is never equivalent to fictional cosmological collapse.

**The player controls interventions, not consequences.** The host controls neither hidden-world truth nor the result of a supposedly stochastic but intentionally seeded transition by replacing it with a dialogue-scripted success label.

## 4. Typed contracts and world identity

The following shapes are *normative semantics*, not compiled Gerbil definitions.

```text
WorldModelRef {
  world_ref: K0.IdentityRef, ruleset_ref: K0.VersionRef,
  transition_contract_ref, causal_dependency_profile_ref,
  observer_profiles[], permitted_time_domains[],
  canonicalizer_ref, qualification_profile_ref
}
WorldAnchor {
  world_ref, model_ref, state_ref, state_generation,
  history_ref_optional, logical_time, space_scope_ref
}
SpaceScope {
  reference_frame_ref, region_selector, locality_mode,
  spatial_resolution_optional, coordinate_schema_ref,
  boundary_policy_ref
}
TimeScope {
  time_domain_ref, current_instant, proposed_horizon,
  event_order_ref, step_granularity, time_uncertainty_optional
}
InterventionSpec {
  intervention_ref, actor_ref, actor_generation,
  target_ref, target_generation, intended_operation_ref,
  proposed_values, scope: SpaceScope, time: TimeScope,
  precondition_ref, model_ref, request_revision,
  declared_effects[], causal_question_optional
}
CausalRule {
  rule_ref, model_ref, input_schema_ref,
  target_variable_refs[], dependency_refs[],
  preconditions, transfer_semantics_ref, time_lag_profile,
  side_effect_ceiling, termination_or_convergence_profile,
  evidence_ref
}
WorldDecisionContext {
  anchor: WorldAnchor, intervention: InterventionSpec,
  rule_locks[], external_input_refs[], budget_ref,
  required_assurance, observation_scope, authority_context_ref_optional
}
```

- **Identity:** equal apparent terrain/characters do not imply identical world, region, generation or history identity. Generation protects against ABA reuse.
- **Space:** one model may represent a graph, metric grid, tiles, astronomical hierarchy, logical zones or mixed partitions. `SpaceScope` never assumes universal Euclidean coordinates.
- **Time:** distinguish event order, simulation time, host monotonic/wall time, fixed physics ticks and fictional historical coordinates. No conversion is implicit.
- **Causal rule locks:** dependencies and evaluator revision must be pinned. Model revision changes invalidate affected caches and receipts.
- **Actor epistemics:** actor belief, perceived evidence and actual state are stored in separate namespaces; an actor is not an omniscient debugger.
- **Dynamic identity:** reference replacement, movement, death/rebirth or region unload can invalidate stale action targets. Pending requests must not accidentally rebind to a new entity with a reused name.
- **Capabilities:** a world rule describes fictional viability, not execution authority in the host OS.

### 4.1 Result and evidence contracts

```text
AdmissibilityVerdict = ADMIT(rule_witnesses)
                     | REFUSE(violated_world_constraints)
                     | UNRESOLVED(missing_facts_or_budget)

ConsequenceCandidate {
  candidate_ref, base_anchor_ref, result_state_ref_or_delta,
  target_observer_diff, dependency_witnesses[],
  unresolved_influence_scopes[], terminal_viability_status,
  input_and_rule_locks[], confidence_kind, claim_scope
}
ConsequenceSet {
  candidates[], completeness: COMPLETE | PARTIAL | NOT_ESTABLISHED,
  coverage_scope, budget_consumed, probability_profile_optional,
  side_effect_intents[], uncertainty, verification_level
}
DecisionReceipt {
  decision_ref, actor_ref, requested_intervention_ref,
  base_state_ref, world_model_ref, admission_result,
  predicted_candidate_refs[], selected_candidate_optional,
  publication_result, actual_observation_refs[],
  effect_status, evidence_scope, unresolved_dependencies[],
  rule_and_rng_locks[], outcome_class
}
```

`predicted_candidate` is not a committed world state. A candidate may be discarded after conflict/refusal or budget exhaustion. **Publishing** a successor is an explicit step with revalidation; an emitted effect intent is not proof that an external side effect occurred.

## 5. Causal dependency and spatial-temporal propagation

### 5.1 Three kinds of dependency

1. **Declared immediate dependency:** world rule states how a value/region/entity depends on selected parents at a given time/version.
2. **Derived transitive influence:** propagation discovers further reachable variables or events under rules, budget and closure constraints.
3. **Epistemic hypothesis:** a player/agent believes one thing causes another, but the world kernel has not validated that edge. A hypothesis never enters the authoritative graph as a proven dependency without an author-approved rule or evidence-admission process.

Some dependencies are **nonlocal** (trade, migration, information, magical linkage, planetary systems). The spatial neighborhood of an intervention is not guaranteed to bound its consequences. The influence engine must use a validated dependency closure or return a conservative `POSSIBLE_REMOTE_EFFECT / UNKNOWN`, not assert no remote effect simply because a region was not materialized.

### 5.2 Conservative influence contract

```text
impact_envelope(W_v, S_t, I_t, H, B)
    -> { definite_affected[], possible_affected[],
         proven_unaffected[], unknown_regions[],
         dependency_certificate, completeness, budget_consumed }
```

- A `proven_unaffected` region requires a **sound, version-bound dependency closure** for the named horizon and state projection. A bounded traversal without closure proof cannot issue this claim.
- If a rule is dynamic, the graph may change after an intervention; incrementally recompute and invalidate stale influence caches.
- A propagation frontier can stop for bounded horizon, capacity, missing state or admitted stable fixed point. Those reasons are distinct receipts.
- Strongly connected dynamical dependencies may require iteration, fixed-point search or explicit nonconvergence. A graph cycle alone neither proves paradox nor justifies collapse.
- `complete` means complete **for the declared domain/horizon/closure**, not complete for an entire universe.
- External side effects may escape spatial scopes and require consumer-level effect authorization, not just causal closure.

### 5.3 A bounded propagation schedule

1. Freeze the world anchor, ruleset, requested spatial/temporal scope, and input/version locks.
2. Validate the intervention and domain admissibility; never run rules from arbitrary serialized Scheme data.
3. Establish a conservative influence frontier and timestamp/ordering assumptions.
4. Evaluate affected rules over an immutable pre-step image or a policy-declared staged transition order; no hidden in-place partial writes.
5. Seal exogenous values (or report explicitly stochastic/unknown inputs), account for fuel and expansion quotas.
6. Build a bounded candidate set and report uncertain branches/remote effects as such.
7. Validate cross-region constraints and the named viability predicates; do not infer stability from momentary consistency.
8. Construct a **pure proposed decision receipt**. No world mutation or history commit during preview.

Explicit bounded policy parameters include:

- World limits: `max_rules`, `max_regions`, `max_horizon_steps`.
- Exploration limits: `max_frontier_items`, `max_candidates`, `max_fixed_point_iterations`.
- Resource and trace limits: `max_bytes`, `max_trace_bytes`, `max_cpu_work_units`.

Do not invent universal production defaults. Host deadlines are independent from model logical time.

## 6. Decision execution, publication and temporal semantics

### 6.1 Three operations

```text
propose_intervention(context) -> ValidatedProposal | Refusal
preview_consequences(proposal, sealed_world) -> ConsequenceSet | Unknown
admit_world_successor(proposal, selected_candidate, expected_anchor)
    -> PublishedLocalState | Conflict | Unknown
```

A game client may *skip preview* and apply an intervention with a valid world kernel transition, but it must still validate and publish through a defined state boundary; preview is not a license to commit stale results.

### 6.2 Publication invariants

- The initial qualified target is a **single-writer in-memory world snapshot** with expected-version comparison. A durable multi-branch commit is **not** promised.
- A multi-region effect is atomic only if its selected state-store profile provides an atomic commit. Otherwise reject the requested guarantee or publish an explicitly staged/partial result, never silently overwrite regions.
- Recheck rule, state, actor, target, permission and generation versions at publication; stale proposals return `STALE_ANCHOR` and must be recomputed.
- Commit picks *one* admitted world successor or an explicitly modeled set of world outputs; an exploratory candidate is never auto-promoted as the single canon.
- Out-of-world effects (saving to disk, sending network commands, triggering cutscenes, modifying a live shared server) use independently bound K0 effects and qualified drivers. A world state transition is not proof those effects succeeded.
- If result acknowledgment is lost, report `UNKNOWN_AFTER_DISPATCH`; no automatic replay of non-idempotent operations. A subsequent observation/reconciliation is a separate step.

### 6.3 Fictional time vs publication order

A world time coordinate may be a date in fiction; publication order is an immutable operational sequence. A change set to *historical world coordinates* (future opt-in profile) must not insert a new committed event into the operational past as though it had always existed. P06's immutable ancestry and revision-pinned reconstruction provide the right optional storage abstraction. Consequently, **a fictional causal loop does not require a cycle in the storage DAG**.

## 7. Outcomes, persistence, convergence and collapse

### 7.1 A changed intervention with the same visible result

Two outcomes may differ in latent state, hidden dependencies, ancestry or random inputs yet be observationally equivalent under observer `O`, at horizon `H`:

```text
observationally_equivalent(S_a, S_b, O, H, profile)
    -> YES(witness) | NO(witness) | UNKNOWN(reason)
```

For example, stopping the initial leader of a conflict may not stop a later war if structural pressures and replacement mechanisms remain. That is **one modeled scenario**, not a universal historical law. The game must supply rules whose output supports the result; an author cannot arbitrarily relabel failure as inevitability while claiming causal simulation.

### 7.2 Convergence is observer-specific

Different paths can yield the same named result (e.g., `war_occurred=true`) without becoming identical worlds. Do not merge history IDs, resource inventories, actor memories or proof traces merely because a displayed outcome matches. Stronger full-state equivalence requires a separate qualified comparator.

### 7.3 Consistency, viability, stability and collapse

- **Consistency:** current candidate satisfies declared hard constraints.
- **Viability:** a declared finite-horizon or policy-specific reachability condition has a witness or refutation.
- **Stability:** a declared perturbation/dynamics profile meets a specific bounded or asymptotic property under stated assumptions. No universal stability oracle is assumed.
- **Collapse:** optional game-world transition activated by a *qualified collapse predicate*. It may make a timeline inaccessible under a future D02/world profile while P06 retains historical identity/evidence per retention policy.

`UNKNOWN`, `UNSAT_UNDER_MODEL`, `UNSTABLE_UNDER_PROFILE`, `COLLAPSED_IN_FICTION` and `DATA_UNAVAILABLE` are not interchangeable. For an incomplete check: refuse definitive collapse and report the missing work. A known inconsistent candidate may be rejected or repaired by an explicitly governed world rule, not automatically destroyed.

### 7.4 Uncertainty without arbitrary randomness

An actor may be uncertain because of **missing observation, unknown structural rule, hidden actor action, stochastic exogenous input, bounded computation or model mismatch**. Encode these causes separately.

- A model's seeded random draw may be replayable if all relevant inputs and implementation versions are sealed.
- Stochasticity must not be used to hide contradictory rules or mislabeled state commits.
- Probability values are model probabilities only if normalized, generated and validated under a named stochastic model; a heuristic ranking is not probability.
- Expose cues and evidence to the player under a declared observability policy, but preserve secret rules and private agent information.

## 8. Observer-relative information and player reasoning

### 8.1 Separate world truth from accessible evidence

```text
observe_world(anchor, observer_ref, observer_scope, observation_policy)
  -> { visible_facts[], uncertain_signals[], source_refs[],
       stale_fields[], inaccessible_fields[], observer_time_ref,
       evidence_grade, world_version_public_commitment }
```

A player never automatically receives authoritative causal edges, inaccessible regions, private NPC motives, RNG stream state or a complete outcome tree. A spectator/research/debug profile may be more privileged only via explicit configuration and access control.

The world kernel may **record an exact causal witness** internally while the player's journal stores an **uncertain hypothesis** based on partial observations. Observation histories can be stale or contradictory from the player's perspective; that does not retroactively mutate world truth.

### 8.2 Testable gameplay requirement

In a minimal toy world, the same visible choice must admit a logical outcome only when the declared hidden-world state warrants it. A thoughtful player should be able to update hypotheses using consistent observable clues, **without** being promised that all hidden causal variables are identifiable from available evidence. A no-information scenario must be labeled an intentional epistemic limit in the design review, rather than marketed as fair deduction gameplay.

Optional player feedback may include confirmed local action, observed consequences, rumor/testimony, partial causal attribution and unresolved assumptions, each with different provenance. Do not award the player a fake `CAUSE_CONFIRMED` status based solely on temporal correlation.

## 9. Gerbil Scheme design and code/data separation

### 9.1 Proposed package layout

```text
editions/world-spacetime/
  src/
    spec/{world,space,time,intervention,rule,observation}.ss
    semantic/{admission,transition,impact,viability,comparison}.ss
    exploration/{frontier,budget,stochastic-inputs}.ss
    runtime/{single-writer-state,decision-receipt}.ss
    ports/{history,world-store,materialization,host-effects}.ss
    authoring/{normalize-ir,diagnostics,macro-sugar}.ss
  tests/
    fixtures/{tiny-world,hidden-world,nonlocal-world,cyclic-world}/
    contract/  pure-reference/  negative/  adapters/
  qualification/
    claims.md  backend-matrix.md  executed-evidence/
```

`src/` topology is a proposal, not proof that Gerbil APIs exist as spelled or that source has compiled. The base has no mandatory imports of graphics libraries, P09, P06, Python, Guix, game engines, neural networks, web frameworks or external databases.

### 9.2 Authoring path

```text
Trusted Gerbil authoring macro OR bounded inert data specification
  -> parse/normalize into typed immutable WorldRuleIR
  -> validate schema, scopes, effects and static dependency claims
  -> analyze determinism, effect escape, cycles and resource bounds
  -> pin interpreter/compiler/expander/model semantic revisions
  -> generate pure reference world transition artifacts
  -> qualify via independent reference oracles and negative tests
  -> optionally bind a qualified world/game/client adapter
```

Untrusted content must remain inert; never pass player-authored text, mods, retrieved files or model output to unrestricted `eval`/macro expansion. A generated source artifact is not automatically executable authority. Gerbil interfaces, especially unchecked method paths, are not a sandbox. Every native/FFI bridge needs independent ABI/thread/lifetime qualification.

### 9.3 Illustrative DSL

```scheme
(define-world-rule settlement-outcome
  (world-revision world-v1)
  (inputs (regional-resources trade-links social-conflict))
  (decision (block-initial-conflict-leader))
  (depends-on (resource-pressure replacement-institutions))
  (time-domain (logical-steps 1 20))
  (impact-scope (possibly-nonlocal))
  (uncertainty (exogenous-inputs explicit))
  (output (consequence-candidates bounded))
  (authority (world-state-only)))
```

The rule name is illustrative. A game-world ruleset can contain causal mechanisms not expressible as one SCM intervention. The concrete Gerbil macro/record syntax is subject to version-locked compiler probes; do not claim `define-world-rule` is currently installed.

## 10. Interoperability contracts without hidden engine imports

### 10.1 P08-D02 world/scene port

```text
WorldStatePort {
  read_snapshot(anchor, scope),
  validate_delta(base, delta),
  publish_if_generation(expected_anchor, proposed_state),
  observe_publication(receipt_ref),
  capability_manifest
}
```

D02 owns scene/entity storage and region accessibility; D01 owns decision semantics. A pure in-memory mock port suffices for D01 qualification. No mandated ECS, quadtree, octree or renderer.

### 10.2 P09 materialization request

```text
MaterializationRequest {
  world_ref, branch_ref_optional, world_model_ref,
  generator_profile_ref, generator_revision, seed_or_input_ref,
  space_scope, world_time_ref, causal_overlay_revision,
  desired_detail, budget_ref, expected_world_generation
}
MaterializationResult {
  region_ref, generated_artifact_ref, input_locks[],
  semantic_validation_result, world_generation,
  uncertainty_or_failure, provenance_receipt
}
```

- D01 validates *meaning and version* of a request, not the algorithm which turns a seed into terrain.
- P09 independently selects **A: pure Gerbil**, **B: Gerbil + native acceleration**, or **C: Gerbil + external generation service** according to measured correctness/performance. This D01 does not decide a backend, port NMS proprietary code or design terrain kernels.
- No per-voxel FFI requirement, no accidental re-run of a generator after stale world revision, and no claim that the same seed alone guarantees byte-identical output across implementations.
- A returned region is not a causal decision; if generator output changes authoritative world semantics, the change must be proposed/admitted as a separate world intervention.

### 10.3 Optional P06 historical port

```text
HistoryPort {
  read_committed_anchor(history_ref),
  propose_step_from_world_receipt(decision_receipt),
  publish_step_if_head(expected_history_ref, qualified_step),
  reconstruct_under_original_rule_locks(history_ref)
}
```

No unconditional `fork_at` call belongs to an ordinary forward-world step. A game selecting historical branches uses P06's **public interface** and retains its own explicit branch/policy semantics; no copying the P06 DAG into P08. If only P08 is installed, `history_ref_optional` stays absent and a documented local-world snapshot still works.

### 10.4 P03/P04/P07 ports

- P03 may produce an event/intent compatible with P08's admitted world transition when semantic clocks and rule revisions match. It does not become P08's scheduler.
- P04 can provide an authorized effect executor for real file/network/native effects if the host selects it; fictional world action legality is not authority.
- P07 may render observer-scoped outcomes and task status; a UI's stale cached view does not authorize or dictate a world transition.

## 11. Cross-domain fixtures and falsifying counterexamples

**F1 — Political persistence:** Model A's initial belligerent is removed by the player. The world rules include resource scarcity and an admissible successor faction, so war still occurs. Observed war flag unchanged; different causal witness and different actor identity. A modified model lacking successor mechanisms must produce a different outcome rather than scripted persistence.

**F2 — Regional nonlocality:** Intervene on a local river dam. Its immediate environment changes; a distant settlement is affected via an explicit trade graph. A naive radius-only impact engine must fail the conservative-closure oracle. An absent trade-edge closure leaves `UNKNOWN`, not certified unaffected.

**F3 — Partial-knowledge investigation:** Two world states yield the same observation to a player but different outcomes under the same action because of hidden resource stock. A privileged oracle can distinguish them; the player cannot be silently granted hidden information. Observational evidence may permit later disambiguation.

**F4 — Constraint inconsistency without collapse:** Two candidate successors violate a declared local invariant; they are rejected. The base world remains valid, no history deleted, and no cinematic universe destruction inferred. With a separately declared and independently qualified fictional collapse predicate, a *different* fixture may produce collapse.

**F5 — Cyclic dynamics with bounded convergence:** Two regions influence each other under a finite monotone fixed-point profile. One rule set converges and emits a witness; another oscillates past the budget and returns `INCOMPLETE`, not `COLLAPSED` or infinite loop.

**F6 — Version race:** A player previews a future under rules `r1`; before commit, the world switches to `r2`. Publication fails with `STALE_MODEL_REVISION`. Re-preview under `r2` may yield a different outcome; cached `r1` result is never reused as valid.

**F7 — History-aware optionality:** A previewed future changes no P06 branch. A committed step uses optional P06 CAS with original rule locks; a new world-only local snapshot exists without P06 installed. The same apparent future on two historical branches remains distinct.

**F8 — Procedural materialization:** Request a region at seed `s`, revision `g1`, branch `b`, world generation `n`; receive output after world advances to `n+1`. Result is stale and cannot overwrite a currently authoritative region. D01 checks contracts but does not generate terrain.

These are **proposed reference fixtures**. They must be implemented with independent expected-output oracles and deterministic synthetic state tables; descriptive examples are not test passes.

## 12. Invariants and failure taxonomy

### 12.1 Core invariants

- **I01 — Independent identity:** world/state/history/observer/candidate/region/actor identities cannot be silently conflated.
- **I02 — Version closure:** model, rule, evaluator, schema and canonicalizer versions are bound to each state transition and receipt.
- **I03 — Distinct statuses:** candidate, preview, admitted successor, observed result and persisted historical event are never the same state flag.
- **I04 — No consequence selection by actor:** actor chooses intervention, not an unverified desired outcome.
- **I05 — No causal inference from correlation alone:** observed co-occurrence does not authorize a new authoritative causal edge.
- **I06 — Sound influence claims:** `proven_unaffected` requires sound named-horizon dependency closure.
- **I07 — Boundedness:** all exploration, propagation, trace and intermediate sizes are budgeted; overrun yields typed partial/unknown.
- **I08 — No implicit omniscience:** observer-relative projections enforce declared scope and information restrictions.
- **I09 — No automatic collapse:** contradiction, nonconvergence and missing information are distinct from a fictional collapse predicate.
- **I10 — No false rollback:** world preview/restore is not the inverse of external effects or immutable P06 historical events.
- **I11 — CAS publication:** stale world, actor, target, rules or history heads block an attempted commit.
- **I12 — No universal time axis:** no implicit conversion between fictional, logical, render, physics and wall clocks.
- **I13 — Explicit stochasticity:** nondeterminism/input closure and claimed replay conditions must be declared and checked.
- **I14 — Observational equivalence is scoped:** convergence does not erase different histories or hidden states.
- **I15 — Source independence:** pure D01 works without P06, P09, game engine, graphics, cloud or RL library.
- **I16 — Qualified adapter bounds:** pure Gerbil qualification cannot automatically certify native, network, filesystem or physical effects.
- **I17 — Effect neutrality:** successful world simulation does not certify actual external effect completion.
- **I18 — Rule-change governance:** rules cannot self-authorize capability escalation or retroactively reinterpret published results.
- **I19 — Speculative noncommitment:** future candidates are never silently published as a historical fact.
- **I20 — Replay honesty:** output reproducibility claims require exact versions, input streams, and a verified compatible execution profile.

### 12.2 Typed failure families

| Failure / uncertainty | Required behavior |
|---|---|
| `INVALID_INTERVENTION`, `INVALID_SCOPE`, `INVALID_TIME_DOMAIN` | Refuse malformed proposals before rule evaluation |
| `ACTOR_NOT_ADMISSIBLE`, `TARGET_GONE`, `STALE_TARGET_GENERATION` | Do not execute action; preserve specific reason |
| `WORLD_RULE_REFUSED`, `UNSAT_UNDER_MODEL` | Distinguish illegal action from inconsistent candidate |
| `AUTHORITY_UNKNOWN`, `EXTERNAL_EFFECT_UNAUTHORIZED` | No live effect dispatch |
| `STALE_ANCHOR`, `STALE_MODEL_REVISION`, `HISTORY_HEAD_CONFLICT` | No stale publication or implicit retry |
| `INFLUENCE_CLOSURE_UNVERIFIED`, `HIDDEN_INPUT_UNAVAILABLE` | Return incomplete/unknown, not a false no-effect result |
| `TIME_PROFILE_UNSUPPORTED`, `STOCHASTIC_PROFILE_UNQUALIFIED` | Reject unsupported semantic claim |
| `BUDGET_EXHAUSTED`, `FIXPOINT_NOT_ESTABLISHED` | Terminate bounded run with resumability claim only if qualified |
| `VIABILITY_UNKNOWN`, `COLLAPSE_RULE_UNQUALIFIED` | Never interpret as a collapsed timeline |
| `MATERIALIZATION_STALE`, `GENERATOR_REVISION_MISMATCH` | Reject stale region result; maintain world identity |
| `UNKNOWN_AFTER_DISPATCH`, `RECONCILIATION_REQUIRED` | No invented effect success, automatic unsafe retry or false rollback |
| `OBSERVER_SCOPE_VIOLATION`, `SECRET_LEAK_BLOCKED` | Fail closed; preserve redacted trace |

## 13. Qualification plan: 64 proposed tests, 12 gates

All `P08-Txx` tests below are **future cases**, not executed or passed. Test runs must bind actual Gerbil/Gambit toolchain versions, model/rule revisions, fixture IDs, observation scope, budgets and oracle versions. The first qualification profile is pure reference Gerbil with a fake world store.

### 13.1 Gate definitions

| Gate | Requirement | Evidence ceiling |
|---|---|---|
| Q0 — Toolchain and source locks | Compile/test exact Gerbil/Gambit APIs and record dependency closure | Toolchain-specific, no blanket portability |
| Q1 — Schema, identity, time | Validate canonical refs and reject stale/ambiguous scopes | P08 pure contracts |
| Q2 — Intervention admission | Independent world-action admissibility oracle and authority separation | Fictional legality, not OS security |
| Q3 — Transition correctness | Finite deterministic transition truth tables and explicit exogenous inputs | Named model/fixtures only |
| Q4 — Impact closure | Conservative spatial/nonlocal influence with soundness counterexamples | Named horizon/closure proof |
| Q5 — Exploration budgets | Termination under cyclic, nonconvergent or stochastic inputs; honest incompleteness | Bounded algorithm only |
| Q6 — Observation epistemics | No hidden-state or rule leakage, stale observation detection | Selected observer profiles |
| Q7 — Outcome semantics | Persistence, divergence, convergence, viability and collapse distinctions | Model-relative, not physical science |
| Q8 — State publication | CAS/rule-lock correctness and no preview side effects | Single-writer in-memory only |
| Q9 — Adapter independence | No P06/P09/graphics requirement; optional integration fault tests | Exact tested adapter |
| Q10 — Cross-domain extension | One game/story fixture and one independent scenario mock consumer | Chosen cross-domain contracts |
| Q11 — Independent audit | Failure injection, mutation tests, negative transfers, evidence reporting | Only tested revision/profile |

### 13.2 Test register

| ID | Fixture / challenge | Pass oracle |
|---|---|---|
| P08-T01 | Duplicate world identity across namespaces | Distinct typed refs; no collision |
| P08-T02 | Target ABA generation reuse | Stale target rejected |
| P08-T03 | Unknown required intervention field | Strict rejection |
| P08-T04 | Wrong coordinate/time schema | Invalid scope before transition |
| P08-T05 | Same apparent world, different model revision | No shared state-cache identity |
| P08-T06 | Unsupported wall-clock to simulation mapping | Explicit unsupported profile |
| P08-T07 | Deterministic canonicalization and round trip | Same validated representation |
| P08-T08 | Invalid actor/world ownership | Domain validation refusal |
| P08-T09 | Admissible but unauthorized host effect | No dispatch |
| P08-T10 | Authorized but world-illegal action | World refusal |
| P08-T11 | Invalid local preconditions | No candidate or mutation |
| P08-T12 | Rule self-grants capability | Rejected capability escalation |
| P08-T13 | Valid action selects not desired reward | Consequence follows world model, not player wish |
| P08-T14 | No matching rule | Typed unhandled-action policy |
| P08-T15 | Two equal-priority conflicting rules | Explicit ambiguity, no silent selection |
| P08-T16 | Change under immutable pre-step state | Reference state-transition oracle agrees |
| P08-T17 | Sealed exogenous deterministic input | Exact repeated trace under pinned profile |
| P08-T18 | Missing exogenous noise | UNKNOWN, not invented exact future |
| P08-T19 | Stochastic probability normalization failure | Invalid probability profile |
| P08-T20 | One intervention, multiple valid candidates | Bounded candidates with incomplete/pure status |
| P08-T21 | Local change with local certified closure | Sound named-horizon unaffected claim |
| P08-T22 | Nonlocal trade/network dependency | Remote possible/affected scope included |
| P08-T23 | Unknown outgoing dependency from region | No false unaffected proof |
| P08-T24 | Dynamic rule changes influence topology | Incremental cache invalidated |
| P08-T25 | Spatial boundary edge-crossing | Boundary policy respected |
| P08-T26 | Zero-size materialized region, nonlocal effect | Still model possible remote outcome |
| P08-T27 | Cyclic monotone dependency | Fixed point matches independent oracle |
| P08-T28 | Oscillating dependency and finite fuel | Typed incomplete, no collapse |
| P08-T29 | Candidate frontier cap | Completeness marked PARTIAL |
| P08-T30 | Maximum state/trace byte bounds | No unbounded allocation; typed status |
| P08-T31 | Deep rule dependency expansion | Depth/fuel bound enforced |
| P08-T32 | Invalid same-time ordering ambiguity | Explicit clock/order rule or refusal |
| P08-T33 | Same observation from different latent states | Player sees same view, world states remain distinct |
| P08-T34 | Privileged debugger observes hidden factor | Scoped access only, no player leak |
| P08-T35 | Player asks for full causal graph | Denied/redacted under player profile |
| P08-T36 | Stale historical testimony | Observation marked stale/qualified |
| P08-T37 | Player hypothesis of false causal relation | Hypothesis never becomes authoritative rule |
| P08-T38 | Observation projector raises exception | No world mutation or secret exposure |
| P08-T39 | Changed intervention, unchanged war outcome | UNCHANGED_OBSERVABLE plus distinct witness |
| P08-T40 | Changed intervention changes outcome | DIVERGED with observer-specific evidence |
| P08-T41 | Two distinct paths observationally converge | CONVERGED; histories not merged |
| P08-T42 | Observational equality vs state equality | Distinct result types and oracles |
| P08-T43 | Constraint-conflicting candidate | Rejected without fictional collapse |
| P08-T44 | Optional valid collapse predicate | Collapse only when exact qualified rule triggers |
| P08-T45 | Unqualified collapse predicate | Refused, never collapsed |
| P08-T46 | Finite horizon viability failure | Scope-specific VIABILITY_FAILED witness |
| P08-T47 | Time-unbounded stability requested | UNKNOWN unless qualified proof exists |
| P08-T48 | Ambiguous collapse/unknown presentation | UI receives distinguishable statuses |
| P08-T49 | Preview without commit | State and P06 history unchanged |
| P08-T50 | Concurrent model version change | STALE_MODEL_REVISION, no commit |
| P08-T51 | Concurrent actor/target generation change | Commit rejected |
| P08-T52 | Single-writer state CAS | One winner; conflict visible |
| P08-T53 | Multi-region atomicity demanded from weak store | Unsupported guarantee refused |
| P08-T54 | Lost external effect acknowledgment | UNKNOWN_AFTER_DISPATCH, no unsafe retry |
| P08-T55 | Optional P06 adapter unavailable | Pure forward world still works |
| P08-T56 | P06 history request with stale branch head | History conflict preserved; no data rewrite |
| P08-T57 | Late P09 region result from older generation | Stale materialization rejected |
| P08-T58 | Generator revision mismatch | No silent region replacement |
| P08-T59 | Gerbil-only reference consumer | Runs without graphics/FFI/RL/database dependencies |
| P08-T60 | Independent control/scenario mock | Shared contracts without adopting gameplay rules |
| P08-T61 | Fault injection into rule evaluator | Typed safe failure; no partial authoritative mutation |
| P08-T62 | Mutated influence rule drops remote edge | Property oracle catches unsound unaffected result |
| P08-T63 | Replay with incompatible compiler/version lock | Refuse exact replay claim |
| P08-T64 | Independent review of evidence claims | Design/mock/backend claims never overstated |

### 13.3 Test harness and release labels

- `DESIGN_PROPOSED`: architecture, contracts, tests specified only.
- `PURE_REFERENCE_QUALIFIED`: a real Gerbil implementation, actual executed tests, and independent finite oracles passed under a locked toolchain.
- `WORLD_STORE_ADAPTER_QUALIFIED`: selected in-memory/other adapter passed CAS, stale state and fault tests; no blanket durability guarantee.
- `HISTORY_ADAPTER_QUALIFIED`: optional exact P06 boundary and historical-version/CAS tests passed.
- `GENERATOR_ADAPTER_QUALIFIED`: optional exact P09 version and materialization tests passed; no blanket correctness across backends.
- `GAME_RUNTIME_QUALIFIED`: a selected host/game client passed end-to-end tests under exact versions, performance targets and bounded resource budgets.
- `UNQUALIFIED`, `SPEC_BLOCKED`, `DEFERRED`, `REJECTED`: legitimate statuses; never silently reclassify them as PASS.

For negative tests, use property checks, mutation tests and independent reference calculations rather than comparing a compiler-generated function with a second call to itself. A source review cannot provide a passing test receipt.

## 14. Implementation sequencing (only if separately authorized)

1. **P08-00 — Frozen semantic register:** pin P06/P03/K0/K1 design refs, official Gerbil toolchain and initial world-rule schema. Record all non-goals and semantics.
2. **P08-01 — Pure decision fixture:** tiny region/actor/resource world, validated intervention, state transition and independent oracle. No scene/backend.
3. **P08-02 — Influence propagation:** conservative local/nonlocal dependency closure, horizon scoping, cycles and budgets.
4. **P08-03 — Epistemic projection:** distinct player/debug observation profiles, hidden causes and testable clues.
5. **P08-04 — Admission/publication:** separate preview and single-writer in-memory commit; stale-rule CAS, failure injection and receipts.
6. **P08-05 — Outcome semantics:** convergence/persistence, optional named viability predicate and collapse counterexamples. Collapse remains disabled until qualified.
7. **P08-06 — Independent tests/audit:** run Q0–Q11 with recorded failures and exact claims. Require a game fixture and an independent non-game fixture.
8. **Separate design authority:** P08-D02 world/scene architecture, P09-D01 procedural generators, and optional P06 time-travel integration. Other application domains may cite P08-D01 as source material and define their own interfaces. No implementation or DLC design is implied by this D01.

Each slice should remain useful by itself. Do not implement eighteen K1 mechanisms or all P08/P09 subsystems just to qualify a small P08 pure reference transition.

## 15. Open design decisions and final verdict

| Issue | Proposed decision | Remaining evidence requirement |
|---|---|---|
| Base-game temporal profile | **Forward-only default** | Exact world/clock semantics after mock qualification |
| Historical intervention | **Future opt-in integration** | P06 public adapter, successor preservation, explicit travel rules |
| World space ontology | **Abstract space scopes** | P08-D02 decides scene/entity/region storage |
| Causal modeling grammar | **Typed deterministic/explicit stochastic rules first** | Gerbil DSL probes, oracle correctness; optional SCM profile later |
| Causal dependencies | **Versioned, conservative and potentially nonlocal** | Soundness counterexamples and changing-rule tests |
| Collapse | **Disabled without named qualified fictional rule** | Consistency/viability/stability oracle plus player-facing semantics |
| Prediction completeness | **Never promised globally** | Explicit budgets, scopes and completeness witnesses |
| Player uncertainty | **Observer-specific and evidence-grounded** | Hidden-world fixtures and nonleakage tests |
| Persistence | **Local in-memory reference only initially** | P05/P06/D02 adapter selected and separately qualified if needed |
| Procedural generation | **Request contract only** | P09 chooses A/B/C based on qualified benchmarks |
| Other application domains | **Independent consumers of decision/observation semantics** | Separate domain designs determine their own adapters and qualification |
| Real-world causal truth | **Not claimed** | No fictitious physical-time or empirical-causation proof |

**Final disposition:**

```text
P08_D01_SPACETIME_DECISION_DESIGN_PROPOSED /
FORWARD_WORLD_PRIMARY / HISTORICAL_INTERVENTION_OPTIONAL /
PLAYER_INTERVENTIONS_NOT_CONSEQUENCES / NONLOCAL_CAUSAL_PROPAGATION /
HIDDEN_WORLD_OBSERVATION / PERSISTENCE_DIVERGENCE_CONVERGENCE_DISTINGUISHED /
UNKNOWN_NOT_COLLAPSE / P06_HISTORY_SEPARATE / P09_GENERATOR_SEPARATE /
GERBIL_PURE_REFERENCE_FIRST / 64_PROPOSED_TESTS / Q0_TO_Q11 /
NO_RUNTIME_PROOF / DOCUMENT_PUBLISHED
```

> **Design principle:** The world determines the consequences under its declared rules. The player learns those rules through limited observations and can intervene, but cannot compel the desired future. Historical facts, possible futures, and observations of either are distinct. The complexity belongs in the causal model, not in a mandatory backend or a magical success flag.