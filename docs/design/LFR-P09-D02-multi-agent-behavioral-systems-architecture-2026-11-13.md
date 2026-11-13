# LFR-P09-D02 | Multi-Agent & Behavioral Systems Architecture

**Document class:** Source-grounded specialist detailed design proposal  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `ADAPTIVE SYSTEMS` — Adaptive Intelligence & Autonomous Systems  
**Edition:** P09 — Adaptive Intelligence & Autonomous Systems  
**Design slice:** D02 — Multi-Agent & Behavioral Systems  
**Primary design language:** Gerbil Scheme for bounded reference behavior, contracts and orchestration; qualified computational adapters only when justified  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P09-D02-multi-agent-behavioral-systems-architecture-2026-11-13.md`  
**Status:** `SOURCE_GROUNDED_DESIGN_PROPOSED / 80_FUTURE_TESTS / 12_FUTURE_GATES / NO_IMPLEMENTATION / NO_RUNTIME_QUALIFICATION / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. Publication does not authorize code installation, implementation, data access, autonomous external actions or deployment.

> **Design decision:** P09-D02 defines a **versioned, bounded, observable multi-agent behavior and interaction system**. It supports deterministic and learned behaviors, sequential and simultaneous actions, cooperation and competition, heterogeneous agents, limited knowledge, shared resources and adversarial opponents. **Single-agent planning is not multi-agent coordination; multi-agent behavior is not automatically multi-agent reinforcement learning (MARL).** The first useful artifact is a small Gerbil-native multi-agent fixture with independent exact oracles and traceable action conflict decisions. No game engine, neural network, external service or distributed runtime is mandatory.

## 0. Objectives, product, and boundaries

**Research question:** Can independently selected behavioral and multi-agent algorithms form a usable, auditable Gerbil-first architecture for games, simulations, robotics-like mock environments and non-game decision tasks, without hiding the difference between observation, intention, negotiation, authorized effects and observed consequences? Can it scale by *measured need* instead of calling all coordination a MARL problem?

**Useful first deliverable:** a deterministic `Crossroads` micro-simulator with **two or three role-distinct agents**, shared and private observations, explicit turn/simultaneity modes, bounded task allocation, resource collision handling, an oracle for small instances, and an independent trace/evaluation CLI. Demonstrate a fixed policy, a behavior tree or task planner, and one coordination method under the *same* fixture semantics. A renderer, P08 world engine, RL learner and network are optional adapters, not prerequisites.

**In scope:** actor/agent identity; role and group membership; heterogeneous observation and action spaces; per-agent memory; FSM, behavior trees, utility, GOAP, HTN, small exact search, team allocation, communication contracts; sequential, simultaneous, mixed and event-driven schedules; action masks and resource conflicts; cooperation, adversarial and mixed incentives; local/public/private knowledge; policy binding; multi-agent evaluation; MARL integration ports and qualified backend choices.

**Out of scope:** operating a game engine; generating worlds (P08-D03); choosing P08-D01 causal consequences; storing authoritative P08-D02 world state; owning a deep RL training system (P09-D01); supervisory control/physical actuation (future P09-D03); distributed transport/consensus/storage semantics (future P09-D04); a universal theorem prover; invented general-game equilibria; unrestricted autonomous tool execution; making a test fixture into production AI.

**Four design prohibitions:** no *implicit* truth-sharing among agents; no implicit promotion from proposal to effect; no conversion of simultaneous turns into sequential turns without semantic proof; no claim that increasing a training reward proves coordination or exploitability resistance.

### 0.1 Claims that must remain separate

1. `BEHAVIOR_CONTRACT_VALID`: a behavior graph/plan obeys its declared input, timing, outcome and resource semantics.
2. `INTERACTION_CONTRACT_VALID`: agent eligibility, turn progression, communication and conflict resolution are reproducible and authorized.
3. `TASK_EFFECTIVE`: a specific candidate beats declared baselines under an independently scored task, uncertainty and budget profile.
4. `DEPLOYMENT_ADMISSIBLE`: an external host independently authorizes the proposed agent action system for a specific environment and effect scope. The first three claims do **not** establish the fourth.

## 1. Source and implementation evidence ledger

Evidence classes: `DOC` = official/maintainer API description; `SOURCE` = publicly inspectable implementation; `COMPARATOR` = reusable technique or counterexample; `LFR_DESIGN` = earlier design document; `PROPOSAL` = not implemented or tested locally. A GitHub README is not evidence of a tested Gerbil port, performance or production suitability.

| Source | Directly inspectable engineering evidence | Design consequence | What it does **not** prove |
|---|---|---|---|
| [PettingZoo AEC](https://pettingzoo.farama.org/main/api/aec/) (`DOC`) | One selected agent acts per AEC step; agent membership and dead-agent cleanup have explicit semantics | `TURN_AEC` must retain order and terminal bookkeeping | Arbitrary AEC to simultaneous conversion is safe |
| [PettingZoo Parallel](https://pettingzoo.farama.org/main/api/parallel/) (`DOC`) | Joint submissions and simultaneous transitions with distinct observations/actions | `TURN_BARRIER` consumes one coherent pre-step snapshot | Every multi-agent problem permits parallel stepping |
| [RLlib Multi-Agent Environments](https://docs.ray.io/en/latest/rllib/multi-agent-envs.html) (`DOC`) | Agent-to-policy mapping, dynamic active agents and per-agent termination/rewards | Agent identity, role, policy identity and policy revision are disjoint | A shared policy implies shared agent information |
| [OpenSpiel](https://openspiel.readthedocs.io/en/latest/intro.html) (`SOURCE`, `DOC`) | Sequential/simultaneous, stochastic, imperfect-information and mixed-payoff games with game-analysis tools | Small exact strategy/oracle fixtures and information-state validation | Arbitrary many-agent general-sum equilibria are efficiently computable |
| [BehaviorTree.CPP](https://github.com/BehaviorTree/BehaviorTree.CPP) (`SOURCE`) | Sequence/fallback nodes, nonblocking actions, typed ports, logging and halting | Explicit tick/interrupt status and blackboard access contracts | C++ implementation has been ported or timed in Gerbil |
| [py_trees](https://py-trees.readthedocs.io/en/release-2.2.x/modules.html) (`DOC`, `SOURCE`) | Memoryful selectors/sequences and interruption behavior differ | Tick scheduling and reset semantics must be type-checked | All behavior-tree dialects are behaviorally identical |
| [ReGoap](https://github.com/luxkun/ReGoap) (`SOURCE`) | Fact/precondition/effect GOAP with bounded action sequences and replanning | Candidate planner with independent plan simulation | A plan's predicted effects will be admitted by its environment |
| [Fluid HTN](https://github.com/ptrefall/fluid-hierarchical-task-network) (`SOURCE`) | Forward task decomposition, method/domain slots, partial plans, logging | HTN candidate for hierarchical task structure | HTN is an optimal or guaranteed complete planner |
| [MPE2](https://github.com/Farama-Foundation/MPE2) (`SOURCE`) | Cooperative and competitive particle scenarios and observation rules | Cross-check of interaction/reward distinctions after baseline | MPE2 scores certify task effectiveness here |
| [BenchMARL](https://github.com/facebookresearch/BenchMARL) (`SOURCE`) | Standardized MARL comparisons incl. MAPPO/IPPO/QMIX/VDN and mixed tasks | MARL adapter candidates + equal-budget comparison discipline | MARL is necessary or superior for all coordination |
| [EPyMARL](https://github.com/uoe-agents/epymarl) (`SOURCE`) | Explicit common-reward vs individual-reward algorithm limitations | Reject incompatible payoff/algorithm pairings | QMIX/VDN apply unchanged to arbitrary general-sum rewards |
| [Gerbil interface reference](https://gerbil.scheme.org/reference/std/interface.html) and [Gambit](https://gambitscheme.org/latest/gambit.pdf) (`DOC`) | Module interfaces, runtime/foreign bindings and toolchain constraints | Contracts first; compile and FFI probes before port claims | Interfaces are a security sandbox or foreign ABI proof |
| LFR [K0-D01](LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md), [K1-D01](LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md) (`LFR_DESIGN`) | Identity, authority, result, observation, multi-agent interaction candidates | Reuse only applicable exact contract slices | Any K1 mechanism has passed runtime gates |
| LFR [P09-D01](LFR-P09-D01-reinforcement-learning-adaptive-intelligence-architecture-2026-11-13.md), [P08-D01](LFR-P08-D01-spacetime-decision-causal-consequence-design-2026-11-13.md), [P08-D02](LFR-P08-D02-game-world-scene-architecture-2026-11-13.md), [P08-D03](LFR-P08-D03-game-world-procedural-generation-architecture-2026-11-13.md) (`LFR_DESIGN`) | RL learner scope and game causality/world/generation separation | Optional typed integration, no hidden cross-edition edits | These designs or runtime adapters are already implemented |

**Licensing/provenance:** candidate algorithms are independently specified first; reuse of third-party source requires a later file-level copyright/license inspection. The LFR design is not a claim to possess proprietary Naughty Dog engine code. Pin a source revision and dependency closure before any backend comparison; published docs may drift.

## 2. Architecture: five independently governed roles

```text
Application / test host (only authoritative action/world executor)
                 | observation + eligibility
                 v
           Observation projector
                 | per-agent local/public/private scopes
                 v
          +------------------------+
          | P09-D02 behavior layer |
          | FSM / BT / utility     |
          | GOAP / HTN / strategy  |
          | optional learned policy|
          +-----------+------------+
                      | ActionProposal + provenance
                      v
           Turn + coordination kernel
           - AEC / simultaneous / event
           - allocation / claims / conflict
           - stale-policy / version checks
                      | Admissible JointActionProposal
                      v
              Host admission port
                      | accepts / rejects / partial
                      v
               Host transition
                      | receipts + observations
                      v
          Trace / independent evaluator

P09-D01: optional learner, policy artifacts, dataset/evaluation port
P08: optional game/world adapter; retains world rules and truth
P09-D03: future control/supervisory adapter; retains actuator authority
P09-D04: future distributed transport/service adapter; retains delivery semantics
K0: identity, validation, authority-record and receipt vocabulary
K1: separately qualified optional behavior/interaction mechanisms
```

The **behavior layer** selects proposals from available information. The **coordination kernel** determines which proposals can be presented as a coherent joint candidate under an explicit turn model. The **host** remains responsible for admission and real effects. An **evaluator** observes independently qualified state/reward evidence rather than treating the agent's self-report as truth.

### 2.1 Identity, role and policy are different dimensions

`AgentId` denotes an active entity in a scenario. `AgentRole` describes privileges and intended participation. `PolicyRef` denotes a frozen decision function, possibly shared by multiple agents; `PolicyRevision` is not an `AgentId`. `TeamRef` groups agents under a versioned membership/coalition rule; teams may change without rewriting past records. `InstanceGeneration` disambiguates a respawned/rejoined agent from a previous instance using the same display name.

A policy may be deterministic, randomized, planning-based, symbolic or learned. Policy sharing never entails shared private observations, shared mutable memory, equal action spaces, synchronized episode endpoints or a common reward. A team can coordinate under distinct policies; competitive agents may temporarily share a communication protocol.

### 2.2 Dependency as an interface

- **K0:** small, exact, versioned identity/effect/authority/result envelopes. K0 cannot schedule turns or enforce permissions by itself.
- **K1:** opt-in M03/M04/M05/M06/M07/M08/M12/M14/M15/M16 when independently qualified. None required as one bundled runtime.
- **P09-D01:** optional policy-inference and learner/evaluation adapter. D02 does not implement PPO/QMIX, train neural networks or overwrite D01's algorithm registry.
- **P08-D01/D02/D03:** optional environment connector. D01 admits causal interventions, D02 owns world truth, D03 generates representations. D02 agents propose only.
- **P03/P04/P05/P06:** optional semantic state/directors, authority-bound effects, durable memory/evidence and history; D02 does not replace their hosts.
- **P09-D03/D04:** future design-only ports for supervisory control and distributed execution. D02 defines intention/interaction semantics but not actuator safety, transactions, consensus or network retries.

## 3. Typed scenario, observation and membership contracts

The following schemas are **normative field semantics**.

```text
MultiAgentScenarioSpec {
  scenario_ref, scenario_revision, environment_ref, environment_revision,
  episode_profile, turn_model: AEC | BARRIER | EVENT | MIXED,
  agent_kind_registry, role_registry, team_registry,
  observation_projection_ref, action_schema_ref, legality_ref,
  goal_objective_ref, payoff_profile_ref, resource_claim_profile_ref,
  conflict_profile_ref, communication_profile_ref_optional,
  termination_profile_ref, truncation_profile_ref, time_profile_ref,
  numeric_profile_ref, rng_stream_ref, host_admission_port_ref,
  budget_profile_ref, evaluation_protocol_ref, adapter_locks[]
}
AgentInstance {
  agent_id, instance_generation, role_ref, team_ref_optional,
  policy_ref, policy_revision, policy_kind,
  private_memory_ref_optional, last_observation_ref_optional,
  action_capability_set, alive_status, ready_status,
  observation_access_scope, knowledge_revision, lease_ref_optional
}
GroupMembership {
  team_ref, revision, membership_intervals[], cooperative_objective_ref_optional,
  shared_resources[], shared_blackboard_scope_optional,
  joining_leaving_policy_ref, evidence_ref
}
ObservationEnvelope {
  scenario_ref, episode_id, logical_step, agent_id, instance_generation,
  observation_revision, visible_facts[], local_events[],
  optional_communication_inbox[], action_mask_ref, time_context,
  provenance_refs[], staleness, privileged_scope: NONE | EVALUATOR_ONLY
}
```

**Knowledge boundaries:** `global_state`, `environment_truth`, `agent_observation`, `agent_private_memory`, `team_blackboard` and `evaluator_state` are different objects. An observation projection cannot secretly inject hidden adversary goals or invisible terrain through a shared team cache. A trained centralized critic may read privileged global features *only within a separately declared training/evaluation profile*; inference policy reads its allowed deployment inputs.

**Membership:** agent disappearance, death, disconnect, respawn, team swap and policy update have explicit generation/episode records. A terminated agent cannot submit a new action by reusing an old agent ID, and an episode may continue when one agent terminates. Distinguish per-agent `terminated/truncated` from collective episode completion, preserving P09-D01 target semantics.

## 4. Behavior engine contract: from stimulus to proposal

```text
BehaviorSpec {
  behavior_ref, revision, kind: FSM | BT | UTILITY | GOAP | HTN | SEARCH | LEARNED,
  input_observation_schema_ref, private_memory_schema_ref_optional,
  action_candidate_schema_ref, goal_ref_optional, plan_cost_profile_ref_optional,
  tick_trigger_profile, replanning_condition_ref, rng_profile_ref_optional,
  deadline, max_search_nodes, max_memory_bytes, max_output_bytes,
  interruption_policy_ref, effect_policy: PROPOSAL_ONLY, provenance_refs[]
}
BehaviorDecision {
  scenario_ref, episode_id, agent_id, instance_generation,
  behavior_ref, behavior_revision, observation_ref, policy_ref,
  tick_or_decision_id, previous_memory_revision, next_memory_candidate_ref,
  action_candidates[], assumptions[], plan_trace_ref_optional,
  claimed_resource_refs[], estimated_cost_vector[],
  objective_evidence_ref_optional, rng_draw_refs[],
  result: PROPOSED | NO_ACTION | DEFERRED | FAILED | INDETERMINATE,
  elapsed_budget, diagnostics[]
}
```

**FSM:** baseline for small stable modes. A transition table must define priority/order when multiple guards are true, unreachable states, re-entry and reset behavior. Good when finite state graphs are small and authored transitions remain legible; poor when combinatorial goals demand search.

**Behavior tree:** tick-based selectors/sequences/decorators with `SUCCESS`, `FAILURE`, `RUNNING` and explicit `CANCELLED/HALTED`. Specify whether the selector/sequence has memory; preemption must cancel or reconcile in-flight action proposals. Blackboard reads and writes are typed by scope, not unrestricted global strings. BT is strong for reactive prioritization and interruption; it is not automatically a goal planner or a theorem prover. BehaviorTree.CPP and py_trees are *comparators*, not forced runtime dependencies.

**Utility-based selection:** evaluate normalized, explicitly calibrated preference scores under a declared context and tie policy; preserve separate hard constraints and safety filters. Include hysteresis/commitment to prevent oscillation, with testable bounds. A weighted score is not an authorization or a globally optimal team allocation.

**GOAP:** actions have explicit preconditions/effects, costs and an exact planning-state model. A* or bounded best-first search is a candidate only if heuristic properties and node/time budgets are declared. Predicted effects are *planning hypotheses*; host receipts determine actual effects. Replan on observed invalidation and record failures. A GOAP planner cannot freely inspect inaccessible world truth.

**HTN:** compound tasks decompose through versioned methods into primitive action intentions; ordered and partially ordered domains are distinct variants and need explicit scheduling semantics. Partial planning, failure recovery and method priorities are independently tested. HTN's authored decomposition can outperform blind search on structured tasks, but is not automatically complete/optimal.

**Search/game-theoretic policy:** exhaustive/minimax/alpha-beta or tiny matrix-game oracle applies only under its declared rules, players and observability. Counterfactual-regret-minimization families and exploitability metrics may be qualified for particular finite extensive-form zero-sum settings. Do not claim a Nash equilibrium or general exploitability score for arbitrary stochastic general-sum, changing-population scenarios.

**Learned policy:** D01 owns parameter/optimizer and policy provenance; D02 owns binding policy revision to a specific agent instance, observation projection, decision deadline, and action candidate. A stale policy or an unavailable learner must not mutate the role registry or silently fallback to a different algorithm.

### 4.1 Planner reuse, failure and memory lifecycle

Planning cache keys include domain/method/goal revisions, observation or belief revision, relevant resource locks, agent role, permitted effects and time profile. Cache reuse requires a valid witness that inputs still match. Invalidation is not just a time-to-live guess. Plans can fail despite a valid predictive model because the host admits different joint actions. Every execution must return a receipt linking attempted step, observed result, replanning reason and bounded search evidence.

Agent private memory is owned by the agent instance. Team blackboard is a separately scoped resource with read/write grants; it is **not** world truth or a distributed consistency database. D02 may record logical claims on shared resources, but durable cross-process messages and storage guarantees belong to later integrations.

## 5. Turn, temporal and simultaneous-action semantics

```text
TurnFrame {
  scenario_ref, scenario_revision, episode_id, frame_id,
  turn_model, pre_state_ref, pre_state_revision,
  eligible_agents[], observation_refs[], action_deadlines[],
  barrier_id_optional, event_order_profile_ref,
  rng_stream_locks[], expected_agent_generations[],
  frame_budget, host_transaction_ref_optional
}
ActionProposal {
  agent_id, instance_generation, policy_ref, policy_revision,
  observation_ref, frame_id, action_value, action_schema_ref,
  action_mask_witness_optional, preconditions[],
  claimed_resources[], intention_hash, evidence_refs[]
}
JointActionProposal {
  frame_id, member_proposals[], missing_action_disposition[],
  joint_validation_ref, resource_conflict_decisions[],
  arbitration_rule_revision, admitted_candidate_hash,
  expected_host_revision, status, evidence_refs[]
}
```

| Mode | Ready-agent semantics | Required ordering evidence | Failure it must detect |
|---|---|---|---|
| `TURN_AEC` | One selected eligible actor at each environment step | selector position, next-agent transition, dead-step handling | An attempted action from the wrong actor; lost dead-agent cleanup |
| `TURN_BARRIER` | Named actors choose from one pinned pre-state and act as a single joint proposal | same frame/revision, deterministic member order only for serialization | Later actor peeking at earlier actor's action before barrier closes |
| `TURN_EVENT` | Explicit event queue with causal/logical order and optional asynchronous emissions | event source, tie-break, effect/admission receipts | Late event treated as current; double application |
| `TURN_MIXED` | Explicit subphases, e.g. several simultaneous actors then one referee turn | phase transition and epoch/step locks | Implicit AEC↔barrier equivalence |

**Barrier semantics:** collect proposals from a frozen observation epoch. Validate membership and legality; apply policy-specified missing-action handling (`WAIT`, `NO_OP`, `FORFEIT`, `REJECT_FRAME`) with exact deadlines; resolve joint resource conflicts by a versioned arbitration rule; submit one `JointActionProposal` to the host; consume one authoritative transition receipt. Deterministic serialization order is not a sequential game transition.

**AEC semantics:** the environment chooses the next active actor. Actions from others are rejected even if their policy can compute early. A finished/terminated actor may require a final bookkeeping step under an adapter's declared contract; D02 retains `terminated` vs `truncated`. AEC→Parallel conversion requires a proof that intermediate actions do not change exposed state or rewards until the barrier; otherwise reject the adapter.

**Asynchrony:** worker completion order may vary, but eligibility, generation locks and expected revisions remain stable. Late replies cannot overwrite a newer frame. Cancellations and deadline misses are explicit traces. If reordering changes semantics, no deterministic replay claim is issued.

### 5.1 Communication and information

`MessageIntent {sender, recipient_or_team, schema, content_ref, visibility_scope, send_frame, receive_phase, ttl, cost, authority_label}` is a proposed communication artifact. Delivered messages become observations only when the environment/host confirms delivery under its stated semantics. Sender-controlled data cannot impersonate environment truth. Distinguish communication actions (`send`, `signal`) from environment actions (`move`, `reserve`) and from a transport-level delivery guarantee.

Use private, public, team and evaluator information partitions. If private beliefs differ despite the same physical state, strategies may legitimately diverge. Test that messages cannot leak secrets to adversaries or upgrade authority claims. If a real network is later used, D04 will separately own retries, ordering guarantees, idempotency and Byzantine/failure assumptions.

## 6. Resource contention, coordination and group decision

**The coordination kernel must not pretend conflicts are solved by serializing agents in an arbitrary order.** Every shared claim has an identity, claim type (exclusive/shared/capacity), lease or frame scope, amount/capacity and deterministic arbitration rule, or is rejected as under-specified.

```text
ResourceClaim {resource_ref, claimant, claim_kind, amount, frame_scope,
               expected_resource_revision, priority_evidence_ref_optional}
ConflictDecision {joint_frame, competing_claim_refs[], rule_ref, rule_revision,
                  accepted_claim_refs[], rejected_claim_refs[], tie_break_ref,
                  fairness_observation_ref, result, evidence_ref}
TeamTaskAllocation {team_ref, goal_revision, task_refs[], candidate_agent_refs[],
                    feasible_assignment_refs[], score_or_cost_matrix_ref,
                    constraints[], allocation_rule_ref, assignment_receipt_ref}
```

**Coordination profiles (independent alternatives):**

- **C0 — Fixed role/rule assignment:** reference fallback, good for authored small scenarios; fails when roles/resources change beyond its table.
- **C1 — Greedy/priority allocation:** low overhead, explicit order and fairness diagnostics; may be suboptimal, starving or order-sensitive.
- **C2 — Exact matching/assignment:** small finite bipartite matching, Hungarian-style cost minimization or enumerative oracle *only if* costs/constraints satisfy the selected algorithm; useful as independent quality witness for task allocation.
- **C3 — Auction/contract-net-inspired proposals:** bounded bidding/award acceptance with bid validity and latency; not a blockchain, consensus or autonomous contracting authority. For regulated/real effects, host approval remains decisive.
- **C4 — Central joint planning:** bounded search over joint actions, exact for tiny states and complexity-limited by combinatorial branching; good oracle, not automatic at scale.
- **C5 — Distributed intention exchange / local negotiation:** only in-memory logical protocol first; partial knowledge, message loss and inconsistent beliefs are expected. No implicit global consensus.
- **C6 — Learned coordination/MARL:** separately qualified task-specific policy profiles via D01, not a substitute for legal action and conflict validation.

**Fairness vs optimality:** minimizing aggregate cost can systematically starve one role. Report objective score, worst-agent deprivation, allocation disparity and budget usage separately; do not collapse them into one scalar labeled `fair`. A priority rule may be intentionally asymmetric when roles/authority require it; this is an explicit task policy, not algorithm failure by itself.

**Deadlock and livelock:** keep a wait-for graph only when resources truly admit hold/wait semantics; test cycles and bounded recovery. Hysteresis/lease expiry and fairness controls are selected based on the workload. A negotiation can time out; it must not claim successful allocation simply because the last message was sent.

## 7. Task-to-algorithm selection and bounded research tours

**Decision order:** fix task and oracle/evaluator -> record observation/action/payoff assumptions -> pick a simple valid baseline -> inspect candidate implementation and source/provenance -> run small falsification fixtures -> compare under equal budgets -> record `qualified`, `rejected`, `unresolved` or `spec-blocked` *for the task*. A paper's citation count and an implementation's popularity are not qualification criteria.

| Task/job | Baseline | Candidate families to investigate | Discriminating tests and reason to reject |
|---|---|---|---|
| Reactive NPC or UI actor | Explicit FSM | BT (reactive/memoryful), utility selector | Tick latency, interrupts, state coverage, score oscillation and unauthorized actions |
| Bounded individual goal planning | Fixed plan/greedy | GOAP/A*, HTN decomposition | Precondition/effect accuracy, replanning budget, unreachable goal, search exhaustion |
| Coordinating several workers | Fixed roles + greedy | Bipartite matching, bounded auction, joint search | Assignment cost vs exact oracle, fairness, resource feasibility, stale bids |
| Finite adversarial turns | Fixed strategy, random | Minimax/alpha-beta, small normal-form solver | Legal action, exploitability only when applicable, hidden info constraints |
| Partial-information cooperation | Independent fixed policies | Shared memory/communication, team task allocation | Forbidden information leakage, communication cost, message delay and missing partner |
| Continuous crowd steering | Rule-based steering | Local avoidance/planner or qualified native vector kernel | Collision rate, frame budget, numeric stability, dynamics validity; future P10 simulation scope |
| Repeated nonstationary games | Frozen pool, independent heuristic | Fictitious-play/self-play or MARL via D01 | Population shift, opponent overfitting, held-out opponents, training/eval separation |
| Learned cooperative discrete actions | Independent policy baseline | IPPO/MAPPO, VDN/QMIX under **compatible shared payoff** | Credit assignment, team vs individual reward semantics, opponent shifts |
| Learned heterogeneous continuous actions | Fixed decentralized control | MAPPO/MASAC/MADDPG under supported spaces | Continuous action bounds, centralized training info, shared-policy compatibility |
| External autonomous systems | Pure mock only | D03 authority/supervision adapter later | No live actuator or tool authority inherited from planner successes |

For each tour, define a task-specific short list, the feasible algorithm versions and stop criteria before running expensive benchmarks. Accept algorithm-family rejection when observed mismatch is real; do not reject a working algorithm for an unrelated academic criterion. A missing oracle yields `UNRESOLVED`, not retroactive success or blanket rejection.

### 7.1 MARL boundary

P09-D01 owns learner identity, frozen policy artifacts, trajectories, update and evaluation lifecycle. D02 adds `AgentPolicyBinding`, team metadata, joint-action records, opponent pools, communication observations and interaction metrics. D02 may request D01 to train a policy or import an independently qualified learned artifact. It cannot silently modify D01 policy weights during inference, turn an individual reward into a team reward, use a centralized critic at decentralized deployment, or infer a game's equilibrium from an episode reward curve.

**Candidate compatibility:** independent learners (IQL/IPPO) may be applicable to distinct-payoff settings but must address nonstationary co-learners. Value-decomposition methods such as VDN/QMIX require the matching cooperative/common-reward assumptions in the selected implementation; the EPyMARL implementation explicitly distinguishes these from general-sum profiles. MAPPO/MASAC/MADDPG depend on their actual action-space and information-profile implementations, not just shared algorithm names. Each variant must be inspected and pinned before proposing implementation.

**CTDE:** centralized training may consume privileged global state under a training-only allowlist; decentralized execution may only consume permitted local observations/history. Evaluate leakage with a test fixture where privileged state predicts outcomes but is intentionally hidden at deployment.

**Opponent evaluation:** separate self-play training pool, held-out policy families, evaluation seeds and adversarial specialists. Report exploitability only for compatible games and a validated best-response solver; use tournament win-rate with confidence intervals and cross-play matrices otherwise. No universally valid `coordination score` exists.

## 8. Gerbil architecture and backend selection

Three valid implementation profiles are alternatives selected **by job**.

| Profile | Runtime ownership | Suitable initial task | Must prove |
|---|---|---|---|
| **A — Pure Gerbil reference** | Gerbil owns typed contracts, small FSM/BT/GOAP/HTN, turn kernel, fixtures, search and trace | Deterministic 2–3 actor scenarios, oracle comparison, small authored NPCs | Source compiles in pinned Gerbil/Gambit, correct replay, bounded resource behavior |
| **B — Gerbil + qualified native accelerator** | Gerbil remains semantic owner; native kernel computes measured hot path (search, batch utility scores, geometric neighbors) | Profiled high-density agent or large search workload | ABI/lifetime validation, exact semantics at boundary, numeric/ordering parity, end-to-end win |
| **C — Gerbil + external execution/learning service** | Gerbil owns request/admission/evidence; service owns separately qualified compute/training | Measured remote MARL training, distributed scenario sweep or specialized solver | Identity/version binding, retry/idempotency, authenticated transport, failure reconciliation and replay ceiling |

Profile A is a useful reference because pure finite-state/graph and small matching algorithms do not require a tensor framework. Profile B must beat A on **end-to-end wall time and correctness** for a representative workload, including serialization, copies, locks and failure overhead. Profile C needs a separately approved service security and data policy. Prefer an optional P09-D01 MARL backend over duplicating a learner in D02.

### 8.1 Proposed Gerbil namespace, descriptive only

```text
lfr/p09/multiagent/
  contract/{identity,scenario,agent,turn,observation,action,receipt}.ss
  behavior/{fsm,bt,utility,goap,htn,search,validation}.ss
  coordination/{membership,claims,allocation,negotiation,arbitration}.ss
  dynamics/{aec,barrier,event,mixed,episode}.ss
  information/{projection,scoped-memory,communication}.ss
  learning/{policy-binding,d01-adapter,opponent-pool}.ss
  evaluation/{oracle,metrics,trace,qualification}.ss
  adapters/{host-world,pettingzoo,rllib,openspiel,native,service}.ss
  tests/{fixtures,behavior,ordering,coordination,adversarial,parity}/
  qualification/{matrix,evidence}/  # empty until execution
```

These are candidate module partitions. Use plain validated data at untrusted boundaries; macros may provide trusted authoring sugar but must not evaluate untrusted Scheme. Verify actual Gerbil standard-library forms and Gambit FFI against a pinned toolchain; do not present pseudocode as built code.

### 8.2 Minimal reference API and trace

```text
create_scenario(spec, authority_ref) -> ScenarioHandle | Error
reset_scenario(handle, seed, initial_profile) -> EpisodeRef + ReadySet + Receipt
observe_agent(episode, agent_ref) -> ScopedObservation | Denied
propose_action(episode, frame, agent_ref, frozen_policy_ref) -> Proposal | NoAction | Error
submit_joint_action(episode, frame, proposals[]) -> JointProposal | Rejected
host_admit_and_step(joint_proposal, host_token) -> HostStepReceipt | Rejected | Unknown
advance_turn(receipt) -> ReadySet + NewFrame | EndEpisode | Unknown
evaluate_trace(trace_ref, protocol_ref) -> MetricBundle + EvidenceStatus
```

A synchronous reference adapter may host all operations in one process, while preserving these logical boundaries. In tests, an injected fake host returns scripted success/reject/unknown outcomes; no real external effects are performed.

## 9. Test worlds and independent oracles

**F01 — Crossroads (required first):** 4×4 finite grid, two agents with different goals, one capacity-one bridge tile and a shared movable token. Include fixed obstacles and a known local observation radius. Run both barrier and AEC *as deliberately different environments*. Enumerate tiny reachable joint states and legal joint actions; derive exact shortest-cost/feasible assignments for selected subcases. Verify the source of a movement conflict and priority decision against an independent transition table.

**F02 — Three-worker allocation:** three heterogeneous workers and three jobs with pinned integer cost matrix; independent brute-force permutations serve as oracle. Include a forbidden assignment, zero-cost tie, missing worker and a fairness-specific evaluation.

**F03 — Interrupted sentry BT:** tree with patrol, pursue, evade and threat-priority guards. Change threat observations while a low-priority task returns `RUNNING`; verify memoryless and memoryful interruption semantics independently.

**F04 — GOAP dynamic-world invalidation:** plan to obtain a resource, then remove the resource after planning but before execution. Check that plan prediction is not a host receipt, stale plan is detected and bounded replan ends with honest failure or alternative.

**F05 — HTN partially unavailable primitive:** decompose a craft/deliver task, forbid one primitive, and test method fallback/partial plan under time budget. Compare declared method priority with actual chosen decomposition.

**F06 — Imperfect-information duel:** finite 2-player game in which two hidden states produce identical observation for one player. Use small OpenSpiel-compatible information-state fixtures, but require independent expected-payoff calculations; no leaked opponent hand.

**F07 — Mixed-motive team:** shared resource and individual payoffs differ. Verify raw individual reward vectors vs team aggregate and reject common-reward-only MARL claims.

**F08 — Heterogeneous turn schedule:** one agent acts at t0, two jointly at t1, one rejoins with new generation at t2; stale messages, policy revision changes and missing deadlines exercise mixed semantics.

**F09 — Adversarial blackboard/message:** one member sends false `resource_available` and forges a privileged `host_receipt`. The team may treat the message as uncertain claim, but may not overwrite environment truth or fabricate admission.

**F10 — Frozen-opponent cross-play:** deterministic, random, fixed-heuristic and optionally learned policies play against held-out opponents across independent seeds. Report a per-opponent matrix, not only mean reward.

**F11 — P08 optional world adapter:** project a P08 world view into per-agent observations, pass intentions to P08-D01 host rules, and use only admitted D02 state receipts. Do not equate a generated terrain descriptor from D03 with a canonical scene mutation.

**F12 — Non-game team scheduling:** workers are abstract software agents with no physics. Propose task allocation under deadlines; evaluate via a pure fake executor. Neither real scheduler permissions nor distributed consensus is assumed.

## 10. Independent evaluation and performance accounting

**Scoring dimensions are not interchangeable:** per-agent completion, team completion, individual reward, common reward, conflicts, collision violations, legal-action failures, plan expansions, replan counts, fairness/privation, communication cost, strategic regret when applicable, inference latency, peak memory, search failures, and evidence gaps. Each metric has an explicit unit, direction, scope and sampling design.

**Compare fairly:** same scenario and adapter revision; same observation scopes; identical action legality and horizon; pinned baselines; disjoint held-out seeds/starts and opponents; equal wall/step/memory budget or a documented normalized comparison; both train and inference cost when relevant. Use paired comparisons where meaningful, uncertainty intervals over independent seeds and per-scenario failures; do not hide unsuccessful episodes in aggregate success rates.

**Small-state oracles:** enumerate joint actions and evaluate exact state transitions for bounded cases; use independent brute-force assignment for C2; validate game-theoretic payoffs with an independent payoff table. For bigger problems, return lower/upper bounds or credible reference comparators and label exact optimality `UNESTABLISHED`.

**Latency:** include observer projection, planner inference, serialization, arbitration and host validation in end-to-end time. A fast planner with expensive copies or invalid outputs is not qualified. Queue/worker timing must distinguish simulation time, host wall time and logical turn index.

**Evidence outcomes:**

- `CONTRACT_QUALIFIED`: fixture conformance and semantic guards passed for a pinned profile.
- `TASK_QUALIFIED`: against task-specific oracle/baseline, independent held-out evaluation and budgets passed with claim-specific intervals.
- `ADAPTER_QUALIFIED`: interface/conversion parity and failure handling passed for a pinned adapter.
- `NOT_ESTABLISHED`: missing evidence, unsuitable evaluator, underpowered comparison or untested runtime.
- `REJECTED_FOR_TASK`: a candidate fails necessary semantics or explicitly defined task objective; does not invalidate a family globally.
- `SPEC_BLOCKED`: assumptions/observation/membership/authority contract insufficient to run a meaningful test.
- `BUDGET_EXHAUSTED`: experiment stopped at declared bound; retains partial evidence, no inferred pass.

A meaningful negative result is retained. A task may succeed under an FSM or greedy algorithm and need no MARL, GPU or distributed runtime.

## 11. Security, authority and failure handling

**Trust domains:** user-authored trusted scenario configuration; untrusted perception/messages; learner-produced proposals; privileged host truth/effects; evaluation-only truth; third-party native/service outputs. A schema validator rejects malformed data; an authority service admits effects. These mechanisms are distinct.

**Failures:** invalid schema; stale frame; non-ready agent; dead generation; missing action; contradictory resource claims; unavailable policy; plan timeout; BT interruption failure; corrupted blackboard; communication scope denial; RNG/profile drift; host reject; ambiguous unknown host status; evaluation leakage; nondeterministic backend discrepancy. Each yields a typed status and retained trace, not silent rescheduling.

**Unknown effect:** when an external host response is unknown, D02 must not assume it committed or failed. Ask the authoritative host for reconciliation using an idempotency/correlation token in a separately designed adapter; no action replay until scope is known. D04 may later supply delivery protocol, but D02 keeps the evidence ambiguity explicit.

**Resource bounds:** caps on active agents, frame horizon, proposal bytes, planner expansions, BT ticks, communication bytes, pending waits, recursion/decomposition depth, memory, wall time, service requests and retained trace data. `NO_ACTION` and `BUDGET_EXHAUSTED` are allowed outcomes; an unbounded retry loop is not.

## 12. Invariants and outcome taxonomy

### 12.1 Core invariants I01–I30

1. **I01 — Identity separation:** agent instance, team, role, policy, environment and action are different identifiers.
2. **I02 — Revision closure:** scenario, agent generation, policy, observation and conflict rule revisions bind each proposal.
3. **I03 — Observation isolation:** private/hidden host state never enters standard agent policy inputs without a declared grant.
4. **I04 — No shared-policy information leak:** shared weights do not imply shared mutable observation/private memory.
5. **I05 — Exact eligibility:** only the ready set may submit frame actions.
6. **I06 — Joint snapshot:** simultaneous actors see the same pinned pre-step world epoch, modulo individually allowed observation projections.
7. **I07 — No accidental serialization:** a stable sort of joint proposals is not sequential game semantics.
8. **I08 — Turn preservation:** AEC, barrier, event and mixed schedules are distinct unless an adapter proves semantic equivalence.
9. **I09 — Dead-agent discipline:** terminated/truncated agents and their final observations have typed bookkeeping.
10. **I10 — Generation freshness:** a restarted agent cannot replay an older instance's action/authority.
11. **I11 — Mask compliance:** masked/illegal action proposals are rejected or handled under a named host policy.
12. **I12 — Plan hypothesis:** planner effects are predictions, never canonical host receipts.
13. **I13 — Interruption correctness:** cancel/halting and in-flight state recovery follow the exact BT/plan variant.
14. **I14 — Memory scope:** blackboard reads/writes enforce private/team/public boundaries.
15. **I15 — Message honesty:** an agent message cannot masquerade as host observation or evaluator ground truth.
16. **I16 — Cost/payoff separation:** individual, team, adversarial, reward, cost and safety metrics are not silently scalarized.
17. **I17 — Conflict evidence:** all contended resource claims have typed arbitration, tie-break and a receipt.
18. **I18 — No fairness claim from score alone:** fairness metrics are independently defined and evaluated.
19. **I19 — Bounded behavior:** BT tick loops, GOAP/HTN search and coordination retries have finite budgets.
20. **I20 — Bounded negotiation:** lack of consensus/acceptance yields a failure or timeout, not success.
21. **I21 — Frozen evaluation:** evaluation opponents and policy revisions are pinned and excluded from training leakage.
22. **I22 — Algorithm/job fit:** selection must satisfy observed action space, turn mode, payoff and information assumptions.
23. **I23 — MARL provenance:** D01 training/trajectory/policy evidence is not rewritten by D02 binding.
24. **I24 — CTDE isolation:** training-only global truth is unavailable at decentralized inference.
25. **I25 — Distinct causal authority:** P08 decides world consequences; D02 cannot force a result by proposing it.
26. **I26 — Unknown effect is unknown:** no fake success/undo after ambiguous host response.
27. **I27 — Determinism scope:** declared replay conditions include scheduling/numeric/backend constraints, not just seed.
28. **I28 — Independent oracle:** task acceptance cannot be graded solely by the evaluated agent's self-report.
29. **I29 — Negative evidence retention:** invalid, rejected and budget-exhausted attempts remain in evidence.
30. **I30 — No false deployment claim:** design/test success is not live effect authorization.

### 12.2 Typed failures

```text
INVALID_AGENT_ID | WRONG_AGENT_GENERATION | POLICY_REVISION_MISMATCH
UNREADY_AGENT | ILLEGAL_ACTION | STALE_OBSERVATION | STALE_FRAME
INVALID_TURN_CONVERSION | BARRIER_INCOMPLETE | DEAD_AGENT_STEP
UNAUTHORIZED_BLACKBOARD_READ | MESSAGE_SCOPE_DENIED | MESSAGE_FORGERY
RESOURCE_CONFLICT | ALLOCATION_INFEASIBLE | FAIRNESS_POLICY_UNMET
PLAN_INVALIDATED | SEARCH_BUDGET_EXCEEDED | INTERRUPT_INCONSISTENT
GOAL_UNREACHABLE | OPPONENT_SHIFT | COMMON_REWARD_ASSUMPTION_MISMATCH
CTDE_INFORMATION_LEAK | HOST_ACTION_REJECTED | HOST_RESULT_UNKNOWN
UNQUALIFIED_BACKEND | NUMERIC_INVALID | SEED_OR_REVISION_DRIFT
EVALUATION_LEAKAGE | TEST_ORACLE_MISSING | INSUFFICIENT_EVIDENCE
```

## 13. Qualification plan: 80 proposed tests and 12 gates

**Important:** all tests/gates below are future specifications. None is marked PASS. Every test must record case ID, fixed inputs, expected failure/success oracle, observed outcome, candidate revision, environment/adapter versions, runtime/toolchain, reproducibility hashes, timing and reviewer disposition. Test-suite execution cannot be asserted by making a PDF.

### 13.1 Gates Q0–Q11

| Gate | Qualification objective | Required artifact and failure disposition |
|---|---|---|
| **Q0 — Source/capability mapping** | Source revisions, licenses and algorithm assumptions documented | `source-lock.json`, unsupported claim -> `SPEC_BLOCKED` |
| **Q1 — Contract/toolchain compile** | Gerbil syntax, type/schema checks, negative parsing, pinned runtime | Build/test log; pseudocode alone -> `NOT_ESTABLISHED` |
| **Q2 — Identity and knowledge** | Agent generation, role/policy separation, observation/blackboard ACL | Independent projection fixtures; leakage -> reject |
| **Q3 — Turn correctness** | AEC, barrier, event, mixed transitions, death/truncation and exact ordering | Turn-trace oracle; accidental conversion -> reject |
| **Q4 — Behavior correctness** | FSM/BT/utility/GOAP/HTN tests against exact oracles where available | Tick/replan logs and bounded termination; wrong transition -> reject |
| **Q5 — Coordination correctness** | Joint action, resource claims, assignment and bargaining | Allocation oracle and receipts; conflicting double grant -> reject |
| **Q6 — Game/payoff semantics** | Cooperative, competitive and mixed objective matrices; strategy claim scope | Payoff oracle; unsupported equilibrium -> `UNRESOLVED` |
| **Q7 — MARL adapter fidelity** | D01 policy/learner separation, CTDE, payoff compatibility, freezes | Pin learned artifact and compare; no learner -> `NOT_APPLICABLE` |
| **Q8 — Robustness and adversarial** | Forged messages, unavailable workers, stale data, failures and resource bounds | Fuzz/negative tests; authority bypass -> reject |
| **Q9 — Performance and backend parity** | A/B/C under identical semantics with end-to-end costs | Benchmark manifest; untested backend -> `NOT_ESTABLISHED` |
| **Q10 — Held-out task effectiveness** | Multi-seed, opponent/task partitions, uncertainty and declared baselines | Evaluation report; inconclusive -> `UNRESOLVED` |
| **Q11 — Independent audit and release scope** | Reproducibility, provenance, licensing, failure evidence and claim discipline | Signed checklist/report; no runtime evidence -> `DESIGN_ONLY` |

**Gate ordering:** start Q0→Q4 on tiny fixtures, Q5–Q6 on joint-task oracles, Q7 only when learning is requested, Q8–Q10 for each real deployment profile, then Q11 independent review. An inapplicable optional gate is recorded as `NOT_APPLICABLE`, never silently counted PASS.

### 13.2 Proposed 80-case falsification register

**IDENTITY (T01–T08):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T01** · Distinct agent and policy IDs | Two agents share one frozen policy; instance memories remain disjoint. |
| **P09-D02-T02** · Spawn generation lock | Old instance action cannot execute after respawn. |
| **P09-D02-T03** · Role swap revision | Role membership change invalidates the previous action grant. |
| **P09-D02-T04** · Agent join mid-episode | Joined actor appears only under declared eligibility phase. |
| **P09-D02-T05** · Agent leave mid-frame | Departed actor is removed or handled by pinned missing-action policy. |
| **P09-D02-T06** · Policy revision freeze | Action proposal pins exact policy while trainer updates another revision. |
| **P09-D02-T07** · Mixed observation spaces | Two roles accept different observation schemas without coercion. |
| **P09-D02-T08** · Team reassignment evidence | History retains old team membership when a role switches team. |

**KNOWLEDGE (T09–T16):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T09** · Hidden adversary intention | Visible observation cannot reveal secret opponent goal. |
| **P09-D02-T10** · Identical observation different truth | Agent action may be identical despite evaluator-only hidden difference. |
| **P09-D02-T11** · Private memory separation | Shared-policy agents cannot read each other private memory. |
| **P09-D02-T12** · Team blackboard read denial | Unauthorized role read returns typed denial. |
| **P09-D02-T13** · Blackboard unauthorized mutation | Team writer scope prevents overriding global world truth. |
| **P09-D02-T14** · Message not trusted as observation | Forged teammate fact remains attributed sender claim. |
| **P09-D02-T15** · Training-only privileged critic | Global state available only under explicit CTDE train profile. |
| **P09-D02-T16** · Observation staleness | Old observation hash fails current-frame proposal validation. |

**TURN (T17–T24):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T17** · AEC selected actor | Nonselected actor action rejected. |
| **P09-D02-T18** · Barrier joint snapshot | No agent sees other frame actions before joint completion. |
| **P09-D02-T19** · Barrier deterministic serialization | Permuting proposal receipt ordering leaves simultaneous outcome unchanged. |
| **P09-D02-T20** · AEC not equal parallel | Action-dependent intermediate reward blocks naive conversion. |
| **P09-D02-T21** · Terminated agent cleanup | Final dead step/receipt is preserved per adapter. |
| **P09-D02-T22** · Truncated versus terminated | Per-agent time limit remains distinct from task terminal. |
| **P09-D02-T23** · Mixed event schedule | Phase and event order match independent hand trace. |
| **P09-D02-T24** · Late worker reply | Stale frame result cannot advance new frame. |

**BEHAVIOR (T25–T32):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T25** · FSM guarded transition tie | Two true guards resolve under stated priority. |
| **P09-D02-T26** · FSM unreachable-state proof | Unreachable or invalid transitions detected under bounded enumeration. |
| **P09-D02-T27** · BT memoryless selector preemption | Threat interrupts running patrol and records halt. |
| **P09-D02-T28** · BT memoryful selector semantics | Running child resumes under pinned variant until external preemption. |
| **P09-D02-T29** · BT failed async cancellation | Late action result cannot resurrect halted subtree. |
| **P09-D02-T30** · Utility tie/hysteresis | Equal scores use fixed tie-break; bounded oscillation test. |
| **P09-D02-T31** · GOAP precondition effect trace | Plan simulator matches declared symbolic model, not host truth. |
| **P09-D02-T32** · HTN method fallback | Unavailable primitive triggers allowed alternate decomposition. |

**PLANNING (T33–T40):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T33** · GOAP false initial state | Unobserved required fact does not become assumed true. |
| **P09-D02-T34** · A* small exact oracle | Optimal path cost equals independent enumeration for admissible heuristic. |
| **P09-D02-T35** · Heuristic nonadmissibility label | No optimality claim for unproven heuristic. |
| **P09-D02-T36** · Stale plan invalidation | Changed resource invalidates cached plan key. |
| **P09-D02-T37** · Bounded no-solution | Search exhaustion returns explicit budget status, no infinite loop. |
| **P09-D02-T38** · HTN recursion cap | Self-referential decomposition stops at configured depth. |
| **P09-D02-T39** · GOAP actual-host reject | Host denial does not get reclassified as plan success. |
| **P09-D02-T40** · Agent-specific goal revision | Goal change invalidates prior proposal and utility scores. |

**COORDINATION (T41–T48):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T41** · Exclusive bridge contention | At most one agent receives capacity-one resource per frame. |
| **P09-D02-T42** · Capacity-N shared resource | Admitted total claims do not exceed pinned capacity. |
| **P09-D02-T43** · Exact allocation oracle | Small assignment result matches independent brute force. |
| **P09-D02-T44** · Forbidden job assignment | Planner never matches agent to explicitly barred job. |
| **P09-D02-T45** · Priority starvation detector | A rule may be policy-admissible but reports worst-agent deprivation. |
| **P09-D02-T46** · Auction stale bid | Bid from prior task revision is invalid. |
| **P09-D02-T47** · Negotiation timeout | Unconfirmed award is not an executed agreement. |
| **P09-D02-T48** · Wait-for cycle | Deadlock fixture reports cycle or bounded escape. |

**STRATEGY (T49–T56):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T49** · Finite zero-sum payoff | Two-player payoffs match hand-computed matrix. |
| **P09-D02-T50** · Imperfect-information partition | Equivalent information states not split by hidden truth. |
| **P09-D02-T51** · Simultaneous mixed strategies | One-shot opponent strategies tested under declared action timing. |
| **P09-D02-T52** · General-sum metric refusal | A zero-sum exploitability metric is not asserted for unsupported game. |
| **P09-D02-T53** · Best-response oracle validity | Tiny best-response computation agrees with exhaustive search. |
| **P09-D02-T54** · Opponent pool leakage | Held-out opponent cannot appear in training record. |
| **P09-D02-T55** · Policy cross-play matrix | Win-rate disaggregated by opponent class and confidence. |
| **P09-D02-T56** · Team versus individual payoff | Aggregate team score never erases individual rewards. |

**LEARNING (T57–T64):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T57** · D01 policy artifact lock | D02 inference does not mutate learned policy weights. |
| **P09-D02-T58** · Common-reward incompatibility | QMIX/VDN candidate rejected for mismatched general-sum setup. |
| **P09-D02-T59** · Independent learner nonstationarity | Changing opponent revision is recorded in data provenance. |
| **P09-D02-T60** · CTDE inference isolation | Decentralized policy cannot consume privileged critic features. |
| **P09-D02-T61** · Invalid action mask from learner | Learned proposal outside mask fails host admission. |
| **P09-D02-T62** · Missing learner fallback | A declared nonlearning fallback records exact alternate policy ID. |
| **P09-D02-T63** · Equal training budgets | Two MARL candidates use declared comparable interaction budgets. |
| **P09-D02-T64** · Zero-learning baseline wins | Nonlearning approach is permitted to qualify over learned one. |

**AUTHORITY (T65–T72):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T65** · False host receipt message | Agent-supplied receipt cannot change canonical state. |
| **P09-D02-T66** · Double action correlation | Duplicate accepted intent does not trigger duplicate host effect. |
| **P09-D02-T67** · Unknown host result | Uncertain effect triggers reconciliation, not retry-as-if-failed. |
| **P09-D02-T68** · P08 world-adapter denial | P08-D01 host rejects inadmissible intervention despite high utility. |
| **P09-D02-T69** · Blackboard permission escalation | Team write never grants resource executor permission. |
| **P09-D02-T70** · External tool access denial | Policy action value cannot call filesystem/network directly. |
| **P09-D02-T71** · Malformed native response | Type and revision mismatches reject B backend output. |
| **P09-D02-T72** · Service replay token mismatch | C backend stale correlation ID cannot replace current result. |

**EVALUATION (T73–T80):**

| Test | Fixture / falsification criterion |
|---|---|
| **P09-D02-T73** · Independent held-out seeds | Train and test seed families remain disjoint. |
| **P09-D02-T74** · Oracle on 4x4 Crossroads | Finite enumerated optimum/feasibility checked independently. |
| **P09-D02-T75** · Metrics preserve negative cases | Failures and timeouts included in denominators. |
| **P09-D02-T76** · End-to-end latency measurement | Observation-to-receipt time counts overhead and contention. |
| **P09-D02-T77** · Memory/expansion budget | Solver stops within profile caps with precise outcome. |
| **P09-D02-T78** · Native semantic parity | Same fixtures expose any native/reference outcome mismatch. |
| **P09-D02-T79** · Unknown game-theory claim | Unvalidated equilibrium remains unestablished. |
| **P09-D02-T80** · Release disposition evidence | Audit report states design/contract/task/deployment separately. |

### 13.3 Claim matrix and evidence artifacts

| Claim | Minimum evidence | Explicit nonclaim |
|---|---|---|
| Deterministic AEC/barrier reference | Q1–Q5, Q8, exact trace hashes, platform lock | GPU/remote reproducibility |
| FSM/BT/GOAP/HTN works for one task | Q4, correct oracle, bounded budget, held-out scenario | General intelligence or universal planner optimality |
| Cooperative allocator useful | Q5, per-job result and independent matching baseline | Globally fair allocation across every task |
| Adversarial strategy effective | Q6, valid game class, opponent coverage and intervals | Nash equilibrium absent a valid solver/proof |
| MARL policy qualified for one scenario | Q7, learner evidence from D01, Q10 held-out and constraint checks | Superior to nonlearning algorithms on other scenarios |
| Native/service adapter safe to consume | Q8–Q9, identity/ABI/protocol and failure tests | Correct physical actuation or unrestricted autonomy |

**Evidence files to create after authorization:** `manifests/scenario-locks.json`, `algorithms/candidate-inventory.md`, `fixtures/oracle-table.json`, `runs/trace-log.jsonl`, `qualification/gate-results.md`, `qualification/task-claim-matrix.csv`, `qualification/reviewer-notes.md`. The absence of these files today is expected and must not be recorded as a successful test.

## 14. Implementation sequencing post separate authorization

1. **D02-00 — Contract and reproducibility probe:** pin Gerbil/Gambit source and runtime; implement inert typed inputs, validation, seeded reference PRNG and exact error outcomes. Run Q0–Q2. Preserve an independent fixture oracle.
2. **D02-01 — Two-agent finite environment:** Crossroads AEC and barrier as distinct profiles; reset/observe/propose/joint-admit/step/trace. Run Q3 with deterministic negative actions and source-lock fixtures.
3. **D02-02 — Behavior reference:** small FSM plus one reactive/memoryful BT; separate utility selector where workload requires. Validate cancellations, memory, and action mask semantics. Run first Q4 subset.
4. **D02-03 — Goal planning comparison:** small GOAP/A* and HTN variants against fixed plans, including invalidation, unreachable goals and bounded expansions. Reject misleading planner claims.
5. **D02-04 — Team coordination:** exact small matching oracle, fixed/greedy allocator and one bounded negotiation profile. Validate double-claim prevention and fairness traces. Run Q5.
6. **D02-05 — Strategic and knowledge fixtures:** small matrix game, information-state distinctions, adversary/teammate messages and frozen opponent evaluations. Run Q6 and Q8.
7. **D02-06 — Optional D01 MARL adapter:** only if a declared task requires learning and nonlearning baselines are inadequate or comparisons are specifically requested. Pin D01 policy/learner and train/evaluate isolation. Run Q7.
8. **D02-07 — Optional measured native/service backends:** only after profiling indicates a real deficit, perform A/B/C parity and end-to-end budget experiments; do not introduce distributed infrastructure just to run a two-agent toy.
9. **D02-08 — Independent audit:** held-out tasks/seeds/opponents, failure retention, license/provenance, Q10–Q11, release scope. Publish claims limited to demonstrated tasks and adapters.

**Independent stopping conditions:** if the baseline satisfies the declared use case under budget, implementation may stop without GOAP, HTN or MARL. If no suitable oracle/evaluator exists, mark `SPEC_BLOCKED` or `UNRESOLVED` rather than spending compute to create persuasive-looking curves. Do not mutate P09-D01, P08, K0 or K1 to make the local design appear compatible.

## 15. Open issues and decision register

| Question | Initial design baseline | Evidence required before selecting alternative |
|---|---|---|
| Agent cardinality | 2–3 finite actors | Memory/latency pressure and real use case for larger population |
| Ready-set semantics | AEC and barrier independent | Formal adapter equivalence evidence before any collapse |
| BT dialect | Minimal explicit statuses/memory | Source-specific interruption/tick parity suite |
| GOAP versus HTN | Per-task candidates, both optional | Search complexity, authoring burden and correctness benchmarks |
| Communication truth | Sender-attributed messages, no inherited trust | Authenticated host delivery and scoped disclosure |
| Cooperative fairness | Separate worst-agent and aggregate metrics | Application's actual fairness policy and constraints |
| Game-theory guarantees | Exact only for tiny compatible games | Valid finite-game solver and strategic assumptions |
| Training | No MARL required for reference | D01 algorithm-specific data/action/payoff qualification |
| Native acceleration | None until A is profiled | End-to-end semantic parity and resource win |
| External service | None until scale demands | D04-grade transport/security and recovery evidence |
| P08 game adapter | Optional source-compatible request/receipt | P08 owner admission, world/procedural version locks |
| Durable team memory | Optional persistence adapter, not world truth | P05 storage/provenance and permission qualification |
| Control/robotics | Simulation-only until D03 | Independent supervisory and actuator safety cases |
| Next design | P09-D03 — Control & Autonomous Systems | Separate review; no automatic inheritance from D02 |

## 16. Final design verdict

```text
P09_D02_MULTI_AGENT_BEHAVIORAL_DESIGN_PROPOSED /
GERBIL_REFERENCE_FIRST / ROLE_AGENT_POLICY_DISTINCT /
LOCAL_PUBLIC_PRIVATE_EVALUATOR_KNOWLEDGE_DISTINCT /
AEC_BARRIER_EVENT_MIXED_TURN_CONTRACTS /
FSM_BT_UTILITY_GOAP_HTN_SEARCH_CANDIDATES /
TEAM_ALLOCATION_RESOURCES_CONFLICTS_COMMUNICATION /
COOPERATIVE_COMPETITIVE_MIXED_PAYOFF_DISTINCTIONS /
MARL_OPTIONAL_D01_LEARNING_OWNER / CTDE_SCOPE_ENFORCED /
P08_GAME_ADAPTER_OPTIONAL / D03_CONTROL_SEPARATE /
D04_DISTRIBUTED_TRANSPORT_SEPARATE / NO_AUTO_AUTHORITY /
A_B_C_BACKENDS_SELECTED_PER_JOB / TASK_LEVEL_QUALIFICATION /
80_FUTURE_TESTS / Q0_TO_Q11 / NO_RUNTIME_EVIDENCE /
NO_BENCHMARK_PASS / NO_IMPLEMENTATION / DOCUMENT_PUBLISHED
```

> **Closing principle:** Agents may disagree, coordinate, compete, deceive, or learn, but their intentions are not world facts and their collective confidence is not authority. A credible multi-agent system must explain **who knew what, who could act when, who won a resource conflict, what the host actually did and how the claimed improvement was independently measured**. The smallest working reference outranks an impressive but unverified hierarchy of algorithms.