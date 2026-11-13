# LFR-P09-D01 | Reinforcement Learning & Adaptive Intelligence Architecture

**Document class:** Source-grounded specialist detailed-design proposal  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `REINFORCEMENT LEARNING` — Adaptive Intelligence & Learning Systems  
**Edition:** P09 — Reinforcement Learning & Adaptive Intelligence  
**Design slice:** D01 — Environment, Algorithm & Evaluation Architecture  
**Primary design language:** Gerbil Scheme for semantic contracts, reference environments and lightweight learners; opt-in qualified ML/native/service backends for heavier computation  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P09-D01-reinforcement-learning-adaptive-intelligence-architecture-2026-11-13.md`  
**Status:** `DESIGN_PROPOSED / SOURCE_REVIEWED / TESTS_SPECIFIED / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. Publication does not authorize implementation, installation, training on sensitive data, deployment or an external-action policy.

> **Design decision:** P09 treats environments, observations, actions, rewards/costs, policies, learners, training runs and evaluations as **separately versioned systems**. Its first useful implementation is a tiny, deterministic Gerbil environment with an exact solution oracle and tabular RL learners. Deep RL and offline RL become independently selected and qualified algorithm/backend profiles, not a monolithic dependency or an automatic replacement for the reference system.

## 0. Problem, product, and boundaries

**Research question:** Can Gerbil Scheme provide a practical, auditable RL specialization whose learning algorithms are selected and qualified against **actual task objectives**, rather than endorsed from articles or presumed correct from attractive learning curves? Can this architecture integrate with game worlds, simulation and control when requested without importing their engines or assuming they share RL semantics?

**First useful product:** a self-contained RL laboratory capable of executing a finite environment, computing exact reference values/policies, training a tabular learner, evaluating it on held-out episodes/seeds, and issuing an evidence-scoped report with algorithm identity, time/memory/sample budgets and failures.

**Subsequent capability:** import or implement DQN, PPO, SAC, offline learners or model-based planners through individually qualified adapters and task-relevant evaluations. Nothing in this D01 declares that those implementations already run in Gerbil.

**In scope:** environment/action/observation API; episode/reset/termination semantics; algorithm capability registry; policy and learner lifecycle; trajectory and replay formats; reward/cost/constraint separation; online/offline training; reference learner semantics; on-policy/off-policy freshness; evaluation and statistics; reproducibility; external compute adapters; versioned checkpoints; qualified usage of P08 as a reference environment.

**Out of scope:** P08's world storage and causal simulator; P06's history DAG/synthesis; multi-agent game AI and game theory (separately designed); building a universal differentiable tensor compiler; implementation of every contemporary RL paper; a universal reward function; real-actuator permission; guaranteed optimality; trading profitability; model deployment; unlimited distributed training; hardcoding a specific Python library as the foundation.

### 0.1 Three independent claims

1. `ENV_CONFORMANT`: an environment implements its declared reset/step/observation semantics correctly.
2. `ALGORITHM_QUALIFIED_FOR_TASK`: an actual algorithm implementation passes named correctness, numerical and task-performance gates on exact profiles.
3. `DEPLOYMENT_ADMISSIBLE`: an independently authorized host has accepted a particular frozen policy under a specific effect/safety profile.

**Neither an environment contract nor a successful training experiment implies deployment authority.** Even an algorithm that performs well in simulated games has not demonstrated physical safety, financial correctness or reliable behavior on unseen tasks.

## 1. Source and implementation evidence ledger

Evidence categories: `DIRECT_DOC` official or maintainer documentation; `OPEN_SOURCE` inspectable implementation; `LFR_DESIGN` existing design-only work; `RESEARCH` conceptual algorithm claim; `PROPOSED` unimplemented P09 mechanism. Open-source availability is **not** equivalent to running the code in our target toolchain.

| Source | Verified engineering lesson | Proposed P09 transfer | Nontransfer / counterexample |
|---|---|---|---|
| [Gymnasium Env API](https://gymnasium.farama.org/api/env/) (`DIRECT_DOC`) | `reset`, `step`, typed action/observation spaces; `terminated` differs from `truncated` | Make task terminal and external horizon separate typed results | Never replace both with one `done` before value-target construction |
| [Stable-Baselines3 algorithms](https://stable-baselines3.readthedocs.io/en/master/guide/algos.html) (`DIRECT_DOC`, `OPEN_SOURCE`) | DQN, PPO, SAC and other families have different space, data and update contracts | Task-to-algorithm capability matrix with independent adapters | A supported algorithm name does not certify this application's task |
| [Stable-Baselines3 evaluation advice](https://stable-baselines3.readthedocs.io/en/v2.9.2/guide/rl_tips.html) (`DIRECT_DOC`) | Separate evaluation environment, multiple tests, budget/tuning awareness | Blind evaluation split, baseline comparison, explicit stop rules | Training reward cannot substitute for holdout performance |
| [CleanRL](https://docs.cleanrl.dev/) (`OPEN_SOURCE`, `DIRECT_DOC`) | Inspectable single-file variants and reproducible experiment conventions | Reference algorithm audit cases and minimal traceable implementation slices | One-file architecture is not mandated or proof of reproducibility across GPUs |
| [Ray RLlib algorithms](https://docs.ray.io/en/latest/rllib/algorithms.html) (`DIRECT_DOC`) | Workers, learners, policy sync and offline/online modes have different data flow | Optional high-scale backend semantics and versioned policy snapshots | Distributed training does not imply exact ordering or replay |
| [d3rlpy](https://d3rlpy.readthedocs.io/en/latest/) (`DIRECT_DOC`, `OPEN_SOURCE`) | Offline datasets, policy selection and off-policy evaluation are first-class | Fixed-data training profile, support-coverage and OPE uncertainty contracts | Offline evaluation alone does not establish online returns |
| [PettingZoo AEC](https://pettingzoo.farama.org/api/aec/) and [Parallel](https://pettingzoo.farama.org/api/parallel/) (`DIRECT_DOC`) | Sequential and simultaneous agent semantics differ | Negative conversion fixtures and future multi-agent boundary | D01 does not implement MARL or silently convert AEC to parallel |
| [Gerbil `:std/test`](https://gerbil.scheme.org/reference/std/test.html), [test runner](https://gerbil.scheme.org/reference/dev/test.html), [FFI](https://gerbil.scheme.org/guide/ffi.html) (`DIRECT_DOC`) | Mock tests and Gambit-backed foreign bindings are feasible | Small pure reference core first; real FFI qualified per toolchain | FFI existence does not prove tensor throughput or safe resource ownership |
| [Gerbil interfaces](https://gerbil.scheme.org/reference/std/interface.html) (`DIRECT_DOC`) | Checked and unchecked dispatch exist | Explicit checking at host boundary and tests for bypass risks | An interface declaration is not a security sandbox |
| LFR [K0-D01](LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md) and [K1-D01](LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md) (`LFR_DESIGN`) | Typed identities, evidence, observation, objectives and lifecycle | Use only selected contracts/mechanisms, never all K1 components | Their proposed qualification is not a runtime PASS |
| LFR [P08-D01](LFR-P08-D01-spacetime-decision-causal-consequence-design-2026-11-13.md), [D02](LFR-P08-D02-game-world-scene-architecture-2026-11-13.md), [D03](LFR-P08-D03-game-world-procedural-generation-architecture-2026-11-13.md) (`LFR_DESIGN`) | World intervention, state and generation are separated | P08 may be an environment source through P09-owned conversion | RL has no write authority over world causality or generator algorithms |
| LFR [P03-D01](LFR-P03-D01-semantic-state-dsl-detailed-design-2026-11-12.md), [P06-D01](LFR-P06-D01-branching-causal-reversibility-design-2026-11-13.md) (`LFR_DESIGN`) | Event order and historical records are separate models | Explicit clocks, episode and history mapping when selected | Reset is neither time travel nor undo of external side effects |

**Research anchors for conceptual semantics:** Sutton & Barto, *Reinforcement Learning: An Introduction*, second edition (http://incompleteideas.net/book/the-book-2nd.html); DQN/PPO/SAC primary publications may explain an algorithm, but **implementation source and executed benchmarks** are the main qualification evidence. No reimplementation is considered correct solely because its equations resemble a paper.

## 2. Edition architecture: four truths, five runtime roles

```text
                      APPLICATION / TASK OWNER
                                 |
                  EnvironmentSpec + ObjectiveSpec
                                 v
  +--------------------- P09 SYSTEM ----------------------+
  |  Environment Host  | Policy Inference | Data Collector |
  |  Learner Backend   | Independent Evaluator             |
  |                                                      |
  |  Separate registries:                                |
  |  env revision | policy revision | learner revision    |
  |  dataset revision | evaluation protocol revision      |
  +-------------------------+----------------------------+
                            |
        optional qualified adapters, not mandatory imports
          P08 world | pure toy model | control simulator
          native ML backend | remote runner | offline data
```

**Four truths must not collapse:**

- `World/Environment Truth`: transition law and hidden environment state. It may be partly unobservable.
- `Actor Observation`: scoped, possibly noisy, delayed or masked evidence accessible to a given policy.
- `Learning Record`: collected samples, labels, policy versions, rewards and inferred values; may contain sampling bias or stale trajectories.
- `Evaluation Verdict`: task-relative measurement against independent baselines/oracles, with uncertainty and declared data scope; not the world's underlying causal truth.

**Five roles:** environment executor; policy inference; data collector; learner/update engine; independent evaluator. One implementation may host multiple roles, but the contracts, versions and permissible operations stay separate. A policy is a versioned decision function, **not** an agent identity, world state or learner process.

### 2.1 Domain independence

A finite grid, stochastic control toy, logged offline dataset, game world or simulation can provide environment transitions if it meets the P09 contract. P08 compatibility is a **source-design relationship**, not a hard-coded P08 RL port. A consumer must create and qualify the adapter it needs; P09 may own a general Gymnasium-like envelope but does not modify P08 code or documents.

### 2.2 Optional K1 reuse

Candidate K1 profiles: M03 pure state transitions, M04 observation, M05 time, M06 bounded exploration, M08 objectives, M12 snapshots, M14 experiments, M15 constraints, M16 evidence and M17 numeric validity. Adopt only individually qualified profiles and do not require an application to install all of them. No K1 profile supplies an RL algorithm automatically.

## 3. Environment contract and episode semantics

```text
EnvironmentSpec {
  environment_ref, environment_revision, environment_kind,
  observation_space_ref, action_space_ref, state_visibility,
  reset_profile, step_profile, time_profile, stochasticity_profile,
  reward_profile_ref, cost_profile_refs[], constraint_profile_refs[],
  termination_rules_ref, truncation_rules_ref,
  action_legality_profile_ref, default_budget_ref,
  evaluator_contract_ref, toolchain_or_backend_lock
}
EpisodeAnchor {
  environment_ref, environment_revision, episode_id,
  episode_generation, step_index, model_state_ref_optional,
  rng_stream_refs[], initial_condition_ref, input_locks[]
}
StepInput {
  episode_anchor, policy_ref, policy_revision,
  action_value, legal_action_witness_optional,
  environment_inputs[], time_advance_ref, expected_step_index
}
StepReceipt {
  previous_anchor, next_anchor, action_ref,
  next_observation_ref, reward_vector, cost_vector,
  terminated, truncated, terminal_reason_optional,
  truncation_reason_optional, constraint_observations[],
  effect_intents[], transition_evidence_ref,
  rng_locks[], state_access_scope, status, budget_usage
}
```

`reset(seed, initial_profile) -> EpisodeAnchor + Observation + ResetReceipt` creates a new episode generation, with declared initial-state stochasticity and information. `reset` is **not P06 reconstruction, P08 historical intervention, rollback, or proof of reversibility**.

`step(episode, action, expected_step) -> StepReceipt | Rejection | Unknown` must validate action schema, current generation, allowed action mask, step index and environment revision. A post-terminal step without a declared auto-reset wrapper rejects rather than silently creating a new episode. Auto-reset, when selected, has its own wrapper revision and final-observation semantics.

**Termination vs truncation:** `terminated` means the task-defined episode terminal condition is satisfied; `truncated` means the run ended via a separately declared horizon, time/resource rule or external condition. Both may be true when a profile explicitly permits it. An algorithm may use bootstrap on truncation but not on a genuine terminal transition according to its named target semantics. It cannot infer either from a generic `done` bit.

**Never silently lose terminal observations.** A vector environment may reset a finished worker immediately; the wrapper must retain a separate `final_observation`, `final_info`, reset event and episode identity or explicitly refuse that normalization.

### 3.1 Observation is not state

POMDP observation and delayed/noisy sensors must be representable without fabricating full Markov state. Policy input may include a recurrent hidden state or belief estimate; those are policy-side artifacts, not privileged environment truth. Observation masking and action masking are different contracts. An action-mask error may allow an illegal move and taint a training run.

### 3.2 Stochastic inputs and time

Seeding is a **reproducibility input**, not proof of end-to-end deterministic execution. Pin environment, generator and RNG revisions; worker order, GPU kernels, external data and numerical modes remain separate provenance. Simulation steps, logical event order, rendered frames and wall time are not equivalent clocks. Record when samples were collected and which policy generated each sample.

### 3.3 Reward, cost, constraint and goal

```text
FeedbackSpec {
  reward_terms[], reward_aggregation_rule,
  cost_terms[], constraint_thresholds[],
  goal_or_terminal_predicate_ref,
  feedback_visibility_scope, revision
}
```

A large scalar reward does not imply that a hard constraint was respected. `reward`, `cost`, `constraint violation`, `termination` and `admission/permission` are different fields. Reward shaping must have a revision; any policy-invariance claim requires a separately validated transformation and task assumptions. Rewards observed by an agent may differ from privileged evaluator metrics; private state must not leak through the learner interface.

## 4. Algorithm capabilities and selection by task

**Do not mistake an algorithm inventory for an implementation qualification.** Selection begins with the application objective, action space, observation structure, interaction mode, data access, compute budget, and policy-latency requirements. An algorithm is then a **candidate for a named job** with necessary capabilities and empirical checks, not an endorsed universal solution.

| Profile | Proposed candidate | Task fit / baseline | Required qualification distinction |
|---|---|---|---|
| `A0` finite control | Dynamic programming/value iteration on an explicitly enumerable MDP | Exact oracle for tabular learners | State enumeration, Bellman residual and terminal semantics |
| `A1` tabular online | Q-learning, SARSA | Small discrete tasks; environment and learner smoke qualification | TD target, exploration schedule, visits, finite convergence diagnostics |
| `A2` discrete deep RL | DQN or qualified value-based variant | Large discrete state features, finite actions | Replay sampling, target sync, gradient/numeric checks, baseline returns |
| `A3` on-policy deep RL | PPO | Stochastic/discrete or supported continuous policies | Policy-age, rollout integrity, advantage/GAE, clipped objective and update evidence |
| `A4` continuous off-policy RL | SAC or TD3 | Bounded continuous action tasks | Action scaling, entropy/target-network semantics, replay data freshness |
| `A5` fixed-dataset learning | Behavior Cloning first, then IQL/CQL as justified | Offline behavior data, constrained interaction | Coverage/support diagnostics, no invented policy value outside support |
| `A6` model-based / planning | Finite MDP planning first; later model-predictive or learned-world-model approach | Explicit model and bounded planning horizon | Model error, horizon validity, planning budget, simulation-only claims |

Algorithms above are **candidate families**, not a commitment to implement all of them. Variants (e.g., DQN with Double/Dueling/PER, recurrent PPO, TQC) have separate identities and are not silently claimed by their parent labels. Multiple candidates may be rejected for a task without implying that all RL is impossible.

### 4.1 Minimal exact baseline: finite MDP oracle

For a finite fully known reference environment, enumerate states/actions, transition probabilities and terminal labels; compute the optimal value/policy with an independent value-iteration or policy-iteration oracle under declared tolerance. Define a deterministic tie-breaker for equally optimal actions. Stochastic transition probabilities must be normalized and sampled by a qualified reference RNG. A learner's training success should be assessed relative to the reachable oracle score **and** random/fixed-policy baselines, not just compared to its own first episode.

### 4.2 Tabular Q-learning reference semantics

For a sampled transition and a named valid target profile:

```text
Q_new(s,a) = Q_old(s,a) + alpha * [ r + gamma * B(s') - Q_old(s,a) ]
B(s') = 0                                      if genuinely terminated
      = max_{a' in legal(s')} Q_old(s',a')     otherwise
```

A truncation does not automatically set `B = 0`. If a finite-horizon task defines time as part of its Markov state, its terminal/horizon semantics must be explicit. `gamma`, learning rate schedule, exploration policy and invalid action handling are versioned. The sampled transition must not be used with a different environment revision or reward scale without a valid transformation.

**SARSA distinction:** on-policy bootstrap follows the actually sampled next legal action under the pinned behavior policy; this is not interchangeable with Q-learning's greedy next-state target. A simple reference learner should expose exact TD-error traces for independent testing.

### 4.3 Deep learner contract: no fake Gerbil-native gradients

Deep models require numerical kernels, gradients, optimizer state and device/error semantics absent from this document. Gerbil owns typed semantic orchestration and model/experiment version locks. A native ML engine, vetted open-source runtime or external worker may own tensor training under a **qualified backend contract**. The proposed ML backend is not presumed to be Rust, C++, Python, CUDA or a particular framework.

For each candidate implementation, inspect *actual code*, lock dependency and variant revisions, execute independent gradient/loss checks where possible, compare with a trusted implementation on small fixtures and evaluate task-level improvement under equal budgets. Reproducibility scopes are platform-specific, particularly for asynchronous workers and GPU kernels.

## 5. Policy, learner and training lifecycle

```text
PolicyArtifact {
  policy_ref, revision, model_architecture_ref,
  weight_or_table_artifact_ref, action_distribution_ref,
  preprocessing_ref, observation_space_ref, action_space_ref,
  inference_runtime_ref, rng_or_sampling_profile_ref,
  authoring_or_training_receipt_ref, claim_scope
}
LearnerProfile {
  learner_ref, algorithm_variant_ref, update_rule_ref,
  optimizer_ref_optional, hyperparameter_revision,
  required_data_profile, required_numeric_backend,
  input_space_contract, objective_ref, training_budget,
  checkpoint_contract, qualification_ref
}
TrainingRun {
  run_ref, environment_refs[], learner_ref, initial_policy_ref,
  data_sources[], split_policy_ref, experiment_protocol_ref,
  compute_resource_limits, seed_family_ref, started_at_ref,
  steps_collected, learner_updates, wall_resource_usage,
  checkpoints[], anomalies[], outcome_status
}
```

`PolicyInference` is a read-scoped operation `infer(policy_revision, observation, hidden_state_optional) -> ActionProposal + InferenceReceipt`, not an authorization to execute the action. The environment (or task owner) validates the proposed action separately. The frozen policy used by a rollout is distinct from the mutable learner updating future revisions.

**Update pipeline:** ingest verified trajectories → validate environment and behavior-policy versions → build algorithm-specific targets → update learner parameters under bounded budget → numeric checks → create a new immutable candidate policy revision → evaluate independently → optionally promote the policy after explicit owner/task criteria. A training run cannot mutate a previously published policy artifact in place.

**On-policy freshness:** PPO or another on-policy algorithm requires a declared data-age/behavior-policy relationship. A stale rollout is not made on-policy by renaming its producer revision. **Off-policy data:** replay is permitted only when the algorithm supports it, with dataset coverage, importance/reuse semantics and sampling provenance. `ExperienceReplay` is neither causal history nor a guaranteed exact replay of the outside world.

## 6. Trajectories, datasets and offline learning

```text
TrajectoryStep {
  dataset_ref, episode_ref, step_index,
  environment_revision, observation_ref, action_ref,
  behavior_policy_ref_optional, action_probability_optional,
  reward_terms[], costs[], next_observation_ref,
  terminated, truncated, final_observation_ref_optional,
  legal_action_mask_ref_optional, external_input_refs[],
  time_and_rng_refs[], provenance_ref
}
TrajectoryDataset {
  dataset_ref, format_revision, schema_revision,
  source_environment_locks[], collection_policy_refs[],
  episode_index_ref, split_manifest_ref, support_diagnostics_ref,
  duplicate_policy, missingness_policy, privacy_scope,
  canonicalizer_ref, integrity_receipt
}
```

- Prevent transition/episode confusion, off-by-one next observations and contamination of train/test/holdout splits.
- Dataset must not disguise a trajectory from another environment/reward/behavior policy as compatible without a migration witness.
- Unknown behavior propensities prohibit unsupported importance-sampling OPE claims; absence of coverage is reported, not replaced with optimistic extrapolation.
- Offline RL may fit a policy to fixed data without online interaction, but its policy quality outside dataset support remains **unestablished** unless independently evaluated under justified assumptions or real interaction.
- No online step, simulator interaction, host command or physical action is an implicit part of `offline_train`.
- A replay buffer may be ring-buffer/columnar/on-disk; it must declare indexing, eviction, episode-boundary handling, numerical representation, concurrent ownership and privacy/retention policy.
- Episode data, recorded dataset and policy checkpoint have different persistence semantics. A successful `save` request is not equivalent to verified durable storage.

## 7. Evaluation: algorithm works for the task, not a paper title

```text
EvaluationPlan {
  plan_ref, target_task_ref, frozen_environment_revision,
  frozen_reward_and_cost_revision, selected_policy_revision,
  baseline_policy_refs[], oracle_ref_optional,
  train_and_eval_split_refs[], evaluation_seed_manifest_ref,
  max_environment_steps, max_wall_duration, max_memory,
  episode_count_rule, confidence_procedure_ref,
  primary_metrics[], hard_constraints[],
  stopping_and_failure_rules[], comparison_budget_policy,
  predeclared_acceptance_thresholds[], protocol_revision
}
EvaluationResult {
  plan_ref, policy_ref, baseline_refs[], evidence_scope,
  per_episode_returns[], costs_and_violations[],
  normalized_scores_optional, resource_usage,
  numeric_anomalies[], uncertainty_intervals[],
  declared_hypotheses_tested[], failure_cases[],
  disposition, qualification_receipt
}
```

**Primary rule:** Before running a candidate, name the task-relevant success criterion: for instance, finite-MDP oracle gap, time to reach target return, sample efficiency, intervention-constraint violations, or performance on a frozen test suite. Do not fail an algorithm because it cannot prove a property the application does not require; do not relax the threshold after seeing results and claim a pre-registered PASS.

Evaluation minimum:

1. Lock environment/reward/cost/observation and algorithm variants; define what exactly counts as an environment step, policy update and GPU/CPU time.
2. Establish **random, fixed and/or known-good algorithm baseline** appropriate to the task. If an exact optimal policy exists, use it as independent oracle.
3. Separate training, tuning/validation and final evaluation data or seeds; report split overlaps and leakage.
4. Compare multiple independent seed runs when stochastic; predeclare how many and how aggregation/uncertainty is calculated. One lucky seed is not a robust improvement claim.
5. Report both reward and costs/constraint violations, terminal/truncation rates, tail or worst-case metrics where material, sample count, wall time and memory.
6. Preserve failure episodes and counterexamples; training divergence, NaN/Inf and exceeded budgets cannot be filtered from final evidence as though no attempt occurred.
7. Repeat under exact platform/backends when making implementation equivalence claims; a numerical tolerance requires separate bound and test.
8. Keep **evaluation PASS**, **algorithm implementation QUALIFIED**, and **deployment authorization** separate. Policy selection is an explicit operation, never an evaluator side effect.

### 7.1 Task-specific qualification examples

| Task | Meaningful success | Meaningless replacement |
|---|---|---|
| Finite deterministic grid | Small oracle return gap, valid legal moves, held-out starts | “Loss went down” |
| Noisy partially observed toy | Improvement over memoryless and fixed policies; scoped observation | Assuming privileged hidden state visible to policy |
| Continuous pendulum/control simulation | Holdout return/cost, action clipping, sample/wall budgets | Continuous-space compatibility inferred from DQN label |
| Offline historical dataset | Support diagnostics, behavior cloning baseline, qualified OPE or holdout simulation | Fabricated online return or causal/trading alpha |
| P08 game-world fixture | Version-locked observations/actions and admitted state transitions | Calling a preview future a committed world outcome |

### 7.2 Honest dispositions

`SPEC_BLOCKED` (unresolved task/contract); `ENV_INVALID`; `ALGORITHM_INCOMPATIBLE`; `IMPLEMENTATION_INVALID`; `NUMERIC_FAILURE`; `BUDGET_EXHAUSTED`; `EVIDENCE_INSUFFICIENT`; `TASK_UNDERPERFORMING`; `TASK_QUALIFIED_ON_PROFILE`; `ADAPTER_QUALIFIED`; `DEPLOYMENT_NOT_ESTABLISHED`. These are **not** interchangeable. Rejected candidates remain in the evidence ledger to avoid repeating known mistakes.

## 8. Gerbil and backend strategies

The first version is a **Gerbil-native reference package**, not a native neural-network framework. It should include immutable environment/policy/trajectory records, deterministic tabular stepping, tabular value iteration, Q-learning/SARSA reference updates, test oracles and receipts. This provides something demonstrably useful without requiring an FFI layer, GPU or external Python environment.

### 8.1 A/B/C backend choices, selected per task

| Backend | Suitable work | Admission evidence |
|---|---|---|
| **A — Gerbil-native** | Finite/typed environments; reference or lightweight learner; simple symbolic policy; validated coordination | Independent small-model oracle, bounded runtime, memory and reproducibility checks |
| **B — Gerbil + native/ML runtime** | Neural policy inference, gradients, tensor operations, accelerated sampling when measured necessary | ABI, buffer ownership, numeric fidelity, lifecycle, device and end-to-end benchmark |
| **C — Gerbil + external training service** | Large batches, distributed training, separate research stacks when cost/reliability justify it | Authenticated requests, job/version identity, data scope, bounded retries, checkpoints, reconciliation, cost and availability |

The three options can coexist per *role*: an application may run a pure Gerbil environment, a native learner and an external evaluator. No option is privileged in advance; **the selected method must prove task fit**. The P08 generator's A/B/C profile and the P09 learner's A/B/C profile are separate decisions with different correctness/performance oracles.

**Interop contract:** `BackendRequest {task_ref, algorithm_revision, env_or_dataset_refs, policy_checkpoint_ref_optional, schema_and_numeric_locks, resource_budget, authority_scope, idempotency_or_job_id}`. `BackendReceipt` returns execution state, trained artifact ref, input locks, resource usage, uncertainty, evaluation evidence refs and unresolved effects. A backend returning HTTP success or exit 0 does not certify that a policy is good.

### 8.2 Gerbil code/data separation

Trusted macros may construct inert typed `EnvironmentIR`, `AlgorithmIR`, `EvaluationIR`. Serialized task specs and user content remain data; they cannot execute arbitrary Gerbil code, import an unreviewed FFI target or grant themselves effect authority. A proposed macro syntax is **illustrative** until compiled against an exact pinned Gerbil/Gambit version. Interface checks must be performed at the caller boundary even if the implementation language supplies unchecked dispatch variants.

```scheme
(define-rl-experiment finite-grid-reference
  (environment (model finite-grid-v1)
               (observations scoped)
               (actions discrete)
               (termination task-terminal)
               (truncation explicit-budget))
  (candidate (algorithm tabular-q-learning)
             (update-profile ql-v1))
  (evaluation (oracle value-iteration)
              (baselines random-policy fixed-policy)
              (held-out-starts true)
              (resource-budget bounded)))
```

The example is a proposed authoring *description*, not existing Gerbil API. Even a valid DSL document cannot authorize real external effects or pass test gates on its own.

### 8.3 Proposed package layout

```text
editions/rl-adaptive/
  src/
    contract/{environment,episode,action,observation,feedback}.ss
    policy/{identity,inference,action-distributions}.ss
    learner/{algorithm-profile,tabular,update-receipts}.ss
    data/{trajectory,dataset,replay,split-checks}.ss
    evaluation/{oracle,baselines,metrics,uncertainty}.ss
    experiment/{run-lifecycle,budgets,checkpoints}.ss
    ports/{environment,training-backend,policy-deployment}.ss
    authoring/{normalized-ir,diagnostics}.ss
  tests/
    fixture/{bandit,finite-mdp,stochastic-grid,masked-actions,
             truncated-episode,hidden-observation,offline-log}/
    contract/ algorithm/ data/ evaluation/ adapter/ adversarial/
  qualification/
    matrix.md
    evidence/      # empty until tests are run
```

## 9. Interfaces to other specialist editions

**P08:** P09 may read the P08-D01/D02/D03 **published design contracts** as evidence for environment observation, admissible action and generation/version identity. P09 does not require a P08 import, inject a new RL/control port into P08, change world rules, interpret P06 histories or own procedural generation. A concrete P08→P09 adapter is a separately designed consumer component only when a real game task requires it.

**P03:** bounded temporal event/director contracts may inform an environment host, but P09 does not inherit P03 scheduling or invent one universal environment `step` implementation. **P04:** external tool/hardware effect authority is held by the host and separately qualified effect drivers. **P05:** durable dataset/checkpoint backends need independent storage guarantees. **P06:** a historical branch is not an episode; reset does not erase durable records. **K0/K1:** use selected exact versioned contracts, not mandatory all-edition imports.

**Future multi-agent specialization:** P09-D01 is single-agent first. Parallel and AEC environments are not silently converted; multi-agent role identity, turn order, shared policies and conflicting simultaneous actions require separate design and qualification. P09 may later supply algorithmic learning components to a multi-agent consumer without claiming game-theoretic equilibria or MARL correctness here.

## 10. Adversarial and cross-domain fixtures

**F1 — Finite grid:** deterministic 5×5 grid, terminal goal/pit, a wall and a small legal action set. Enumerate every transition and compute oracle action values. Verify wrong action, terminal, truncation and repeatable seeds. This is the first qualified target, no game renderer or P08 dependency.

**F2 — Two observations, one hidden state difference:** distinct latent states generate the same visible symbol, leading to different optimal responses with memory. Verify observation cannot leak state; any memoryful policy is separately qualified.

**F3 — Cut-off before terminal:** episode reaches time limit at a nonterminal state. The Q-learning target still bootstraps under the declared truncation semantics. A `done=terminated OR truncated` implementation must fail its numerical oracle.

**F4 — Same policy, different environment revision:** update rewards or legal actions between collection and training; old samples are quarantined or used only under a qualified migration, never implicitly combined.

**F5 — Stale PPO rollout:** learner policy updates while collector returns an old rollout. An on-policy learner must reject/handle it under declared staleness bounds, not silently label it current.

**F6 — Offline dataset without support:** historical logged data omits a class of actions. A proposed unseen-action value improvement may not be called verified; report support gap and OPE limitation.

**F7 — P08 world source:** a region rule admits a world intervention, but the player-visible policy observation omits hidden causal facts. The environment adapter cannot expose privileged world state nor claim `preview_consequences` is a committed step; P08 retains world authority.

**F8 — Control/financial non-game mock:** same P09 episode/learner/evaluation records are exercised on a numeric simulator with explicit costs and terminal/truncation. Neither physical safety nor live trading success follows from the mock.

**F9 — Asynchronous worker:** duplicated, delayed or reordered inference and update receipts cannot silently advance policy revisions or double-count experience.

**F10 — Reward hacking fixture:** agent maximizes a metric while violating a declared cost/constraint. Reward may increase; evaluator must still return constraint failure.

## 11. Invariants and failure taxonomy

### 11.1 Core invariants I01–I24

1. **I01 — Disjoint identities:** environment, episode, agent, policy, learner, dataset, evaluation and backend job refs are not aliases.
2. **I02 — Version closure:** every sample and model result pins environmental, reward, observation, action and algorithm variants.
3. **I03 — Termination discipline:** `terminated` and `truncated` remain separate from collection to target computation and evaluation.
4. **I04 — No reset-as-rollback:** reset neither restores external effects nor rewrites historical causal records.
5. **I05 — Observation isolation:** hidden environment state cannot be read through ordinary policy observations without authorized scope.
6. **I06 — Action admissibility:** invalid/masked actions fail or follow an explicit documented environment profile.
7. **I07 — Feedback separation:** reward, costs, constraints and authorization are distinct contract fields.
8. **I08 — Policy revision immutability:** learner updates create new candidate revisions; existing deployed/frozen policy artifacts remain unchanged.
9. **I09 — Behavior policy provenance:** rollout samples record the actual behavior-policy version, not retrospectively assigned learner version.
10. **I10 — On/off-policy distinction:** algorithm admits data only under its declared compatibility and freshness rules.
11. **I11 — Offline support honesty:** unknown behavior probabilities or absent support cannot be converted into qualified policy values.
12. **I12 — No arbitrary backend trust:** external success statuses do not establish correct numeric result or policy performance.
13. **I13 — Splits are independent:** evaluation data/seed leakage or reuse must be detected and labeled before qualification.
14. **I14 — Baseline anchoring:** gains must be reported against appropriate frozen baselines/oracles, not only training curves.
15. **I15 — Finite resource limits:** steps, updates, wall time, memory, workers, model/checkpoint bytes and evaluations are bounded.
16. **I16 — Outcome clarity:** underperformance, invalid implementation, insufficient evidence and budget exhaustion are distinct.
17. **I17 — Determinism scoped:** a seed is an input lock, not a universal replay guarantee.
18. **I18 — Numeric validity:** NaN/Inf, overflow, invalid action distributions and gradient anomalies are explicit failures.
19. **I19 — Evidence retains failure:** aborted runs and failed candidate trials remain counted in evaluation evidence.
20. **I20 — No false policy promotion:** selection/deployment requires an independent owner/host admission step.
21. **I21 — External effect separation:** a learned action proposal is not execution authority and cannot bypass host safety checks.
22. **I22 — Semantic portability:** non-game consumers can use P09 without P08/game engine imports.
23. **I23 — No global algorithm winner:** performance/qualification applies only to named tasks and evidence scopes.
24. **I24 — No false maturity:** a design test register is not an executed qualification result.

### 11.2 Typed outcomes

| Condition | Required result |
|---|---|
| Unknown environment revision / action schema | `ENV_INVALID` or `SCHEMA_INCOMPATIBLE`, no step |
| Stale episode generation or repeated step | `STALE_EPISODE` / `STEP_ORDER_CONFLICT` |
| Illegal or masked action | `ACTION_REFUSED`, with model-defined reason |
| Hidden-state exposure / unauthorized observation | `OBSERVATION_SCOPE_VIOLATION`, redacted receipt |
| Terminal and truncation conflated | `ALGORITHM_TARGET_INVALID`, no learner qualification |
| Policy/data stale or unsupported on-policy mixing | `BEHAVIOR_POLICY_MISMATCH` |
| Missing offline coverage or behavior propensity | `SUPPORT_UNESTABLISHED` / `OPE_UNSUPPORTED` |
| Train/eval overlap | `EVALUATION_LEAKAGE`, no claimed holdout |
| NaN/Inf/invalid gradient or action probability | `NUMERIC_FAILURE`, preserve run evidence |
| Lost external job acknowledgment | `BACKEND_OUTCOME_UNKNOWN`, reconcile before unsafe retry |
| Budget exceeded | `BUDGET_EXHAUSTED`, typed partial/failed status |
| Strong reward, violated hard constraint | `CONSTRAINT_FAILED`, no fabricated task success |
| Policy qualified in mock but deployment untested | `TASK_QUALIFIED_ON_PROFILE / DEPLOYMENT_NOT_ESTABLISHED` |

## 12. Qualification plan: 72 proposed tests and 12 gates

Tests are specifications, not executed evidence. Toolchain and exact package versions must be pinned at execution; an independent environment oracle and algorithm reference are required for the first finite task. A failed test records the variant and counterexample instead of broadening definitions until it passes.

### 12.1 Gates Q0–Q11

| Gate | Requirement | Evidence ceiling |
|---|---|---|
| **Q0 — Toolchain** | Pin Gerbil/Gambit, fixture, numeric mode, algorithm code and dependency closures | Exact tested build/platform |
| **Q1 — Contract** | Action, observation, episode, identity and schema checks, including negative cases | Named environment profile |
| **Q2 — Episode semantics** | Terminal/truncation/reset and final-observation truth table | Exact step/wrapper contract |
| **Q3 — Reference oracle** | Independent finite MDP transition and Bellman oracles | Finite enumerated fixtures only |
| **Q4 — Learner arithmetic** | Q-learning/SARSA targets, legal actions, exploration and bounded updates | Exact reference algorithm variant |
| **Q5 — Data integrity** | Trajectory versioning, support, splits, behavior policy and replay properties | Selected dataset schema |
| **Q6 — Experiment governance** | Training/evaluation isolation, budgets, checkpoints and counted failures | Specific experiment protocol |
| **Q7 — Task performance** | Predeclared baselines, held-out metrics, uncertainty and constraint results | Named task and thresholds |
| **Q8 — Numeric/backend** | If selected, tensor/FFI/service ownership, numeric and failure parity | Exact runtime/backend/profile |
| **Q9 — P08/other adapters** | Cross-edition independence, authorized action and observation isolation | Selected adapter only |
| **Q10 — Robustness** | Stale data, delayed workers, adversarial, reward hacking and leakage negatives | Tested disturbances, not universal safety |
| **Q11 — Independent audit** | Verify claims, evidence links, non-PASS dispositions and implementation scope | Only demonstrated modules and workloads |

### 12.2 Test register

The register below is **future qualification work**, not results. Each test must have a fixture revision, run command, pass oracle, observed values and failure trace when implemented.

| ID | Challenge | Pass oracle |
|---|---|---|
| **P09-T01** | Environment ID collision across namespaces | Distinct typed identity / no cross-env cache reuse |
| **P09-T02** | Episode generation reused after reset | Old step rejected STALE_EPISODE |
| **P09-T03** | Unknown action schema field | Strict validation refusal |
| **P09-T04** | Incompatible observation schema | No silently coerced observation |
| **P09-T05** | Policy revision rebound to new model | Explicit incompatibility/migration |
| **P09-T06** | Missing RNG/source revision lock | Replay claim blocked |
| **P09-T07** | Malformed feedback vector | Feedback validation fails |
| **P09-T08** | Unknown worker authority class | No live-effect admission |
| **P09-T09** | Reset emits initial observation | Correct generation and schema |
| **P09-T10** | Step after true terminal | Reject until reset unless qualified wrapper |
| **P09-T11** | Nonterminal truncation | Separate truncated status and valid bootstrap semantics |
| **P09-T12** | Simultaneous terminal/truncation declared | Both flags preserved under named profile |
| **P09-T13** | Lost final observation during auto-reset | Final observation required or wrapper refused |
| **P09-T14** | Off-by-one step index | STEP_ORDER_CONFLICT |
| **P09-T15** | Clock-domain mismatch | No implicit time conversion |
| **P09-T16** | Noisy stochastic transition with locked RNG | Exact declared replay or honest non-equivalence |
| **P09-T17** | Two latent states share observation | No privileged state leak |
| **P09-T18** | Unobserved variable used by policy | Scope violation blocked |
| **P09-T19** | Illegal action under fixed mask | Action refused |
| **P09-T20** | Mask changes under environment revision | Old mask invalidated |
| **P09-T21** | Observation and action shape mismatch | Typed schema rejection |
| **P09-T22** | Policy hidden state without version | Recurrent-state compatibility refusal |
| **P09-T23** | Reward and cost contradictory incentives | Cost violation remains visible |
| **P09-T24** | Reward shaping revision changed | Old feedback records quarantined |
| **P09-T25** | Finite MDP transition sum != 1 | Model invalid |
| **P09-T26** | Finite value iteration exact check | Residual/oracle tolerance passed |
| **P09-T27** | Deterministic tie optimal actions | Documented tie policy consistent |
| **P09-T28** | Q-learning nonterminal target | Numeric target matches independent oracle |
| **P09-T29** | Q-learning truncated target | Does not incorrectly zero bootstrap |
| **P09-T30** | Q-learning terminal target | Correct zero bootstrap |
| **P09-T31** | SARSA vs Q-learning next action | Different justified target |
| **P09-T32** | Invalid gamma/alpha/exploration schedule | Reject invalid hyperparameter |
| **P09-T33** | Trajectory transition ordering | No skipped/duplicated step |
| **P09-T34** | Behavior policy version absent from on-policy sample | On-policy admission fails |
| **P09-T35** | Offline dataset source mismatch | No forced merge without migration |
| **P09-T36** | Replay buffer crosses episode terminal | Terminal boundary maintained |
| **P09-T37** | Dataset duplicated samples | Dedup policy enforced and recorded |
| **P09-T38** | Train/eval split overlap | EVALUATION_LEAKAGE detected |
| **P09-T39** | Offline behavior propensity unknown | Importance-sampling OPE refused |
| **P09-T40** | Offline unseen action support | Support gap reported, no qualified value |
| **P09-T41** | Policy inference from immutable revision | Frozen output and version receipt |
| **P09-T42** | Learner update policy revision race | No mutation of frozen policy |
| **P09-T43** | Stale PPO rollout arrives | On-policy freshness contract enforced |
| **P09-T44** | Off-policy replay accepted conditionally | Only configured learner permits reuse |
| **P09-T45** | Optimizer checkpoint lacks normalizer | Cannot claim restored equivalence |
| **P09-T46** | Episode steps vs updates miscounted | Separate counters reconcile |
| **P09-T47** | Run resource limit exhaustion | Typed budget failure |
| **P09-T48** | NaN/Inf learner update | Numeric failure and evidence retained |
| **P09-T49** | Random policy baseline on grid | Independent baseline reproduced |
| **P09-T50** | Exact optimal-policy baseline | Learned gap computed correctly |
| **P09-T51** | Holdout seed contamination | No holdout PASS |
| **P09-T52** | Reward improves, hard constraint violated | Constraint failure wins |
| **P09-T53** | Many seed results vs cherry-picked best | Predeclared aggregation and interval |
| **P09-T54** | Evaluation wrapper changes termination | Mismatch detected or qualified mapping |
| **P09-T55** | Per-episode tail metric required | Tail/worst-case results recorded |
| **P09-T56** | Candidate underperforms task target | TASK_UNDERPERFORMING |
| **P09-T57** | P08 game observation adapter | No hidden causal leakage |
| **P09-T58** | P08 world preview vs actual step | Preview not treated as committed state |
| **P09-T59** | P08 procedural generation backend change | Version and environment source locks retained |
| **P09-T60** | Non-game numeric environment | No dependency on game engine |
| **P09-T61** | ML backend invalid tensor layout | Adapter refuses/returns numeric failure |
| **P09-T62** | Native inference resource release unknown | No release-success claim |
| **P09-T63** | External learner lost acknowledgment | No invented success or blind unsafe retry |
| **P09-T64** | Remote policy artifact reorders results | Request/model identity checked |
| **P09-T65** | Duplicate worker completion | No duplicate policy update or data count |
| **P09-T66** | Truncation flag stripped in import | Import rejected or incompatible |
| **P09-T67** | Data-only DSL injects arbitrary eval | No execution/authority grant |
| **P09-T68** | Reward-hacking candidate high score | Constraint/oracle catches failure |
| **P09-T69** | Stochastic GPU run same seed differs | No unconditional byte-replay claim |
| **P09-T70** | Environment offline OPE unsupported | Qualification ceiling retained |
| **P09-T71** | Simulation-qualified policy requests hardware action | DEPLOYMENT_NOT_ESTABLISHED |
| **P09-T72** | Independent audit with known failed runs | Failures retained, no all-PASS invention |


### 12.3 Qualification claim matrix

| Claim | Minimum evidence | Ceiling |
|---|---|---|
| `P09_CONTRACT_DESIGN_PROPOSED` | D01 published/peer-reviewed | No executed code |
| `P09_ENV_REFERENCE_QUALIFIED` | Q0–Q3 + relevant Q9–Q11 with executed Gerbil fixtures | Named environment and toolchain |
| `P09_TABULAR_ALGORITHM_QUALIFIED` | Q0–Q7, Q10–Q11 on finite tasks with actual improvement and numeric checks | Named algorithm/task/seed protocol |
| `P09_DEEP_ALGORITHM_ADAPTER_QUALIFIED` | Relevant Q0–Q11 including Q8, independent code/gradient and performance checks | Exact backend/variant/workloads |
| `P09_OFFLINE_POLICY_EVALUATION_QUALIFIED` | Q5/Q6/Q7/Q10/Q11 with support assumptions, coverage and uncertainty | Selected data/support/policy class only |
| `P09_DEPLOYMENT_ADMISSIBLE` | Separate real-host safety, permission, observational and effect qualification | Named host; not inferable from P09 alone |

**Release is modular:** qualifying a tabular learner does not certify PPO, SAC, offline OPE, a native tensor engine, a P08 game world or deployed control. All 72 registered cases need not be executed in one first-release package; the exact applicable subset must be declared before a profile is qualified, while every mandatory gate for the claimed profile must actually pass.

## 13. Implementation sequencing post separate authorization

1. **P09-00 — Evidence and locks:** pin K0/K1/P08 interface revisions and external implementation/docs; define exact algorithm/task success and negative controls.
2. **P09-01 — Environment/episode contracts:** pure Gerbil finite grid, typed terminal/truncation semantics and independent transition oracle.
3. **P09-02 — First working algorithm:** value iteration baseline, tabular Q-learning and SARSA, with reproducible per-step target traces, observation/action tests and task performance qualification.
4. **P09-03 — Data/evaluator:** trajectory versions, replay integrity, frozen splits, random/fixed/oracle baselines, multi-seed evaluation and failure ledger.
5. **P09-04 — Algorithm candidate tour:** choose one additional algorithm *because a named task requires it*; inspect maintained implementation source, adapters and numeric/parity evidence before attempting broad training.
6. **P09-05 — Optional advanced backends:** select A/B/C by measured fit; add DQN/PPO/SAC/offline paths one by one under separate task qualification, no compulsory neural trainer.
7. **P09-06 — Independent integration audit:** test a self-contained environment and a non-game numeric environment; test P08 only as a deliberately selected adapter when available.
8. **P09-07 — Separate future design authority:** advanced training backend algorithms, multi-agent/games and actual application deployment each require their own scoped designs. D01 does not pre-commit the entire program.

**Task prioritization:** if time/compute is constrained, build a *correct and measurable small learner* before an impressive but unqualified deep-learning dashboard. A reproducible finite baseline reveals target bugs before they are hidden in hundreds of millions of gradient updates.

## 14. Open issues and design verdict

| Issue | Proposed baseline | Required evidence before promotion |
|---|---|---|
| First algorithm | Tabular Q-learning + SARSA, value-iteration oracle | Correct numeric targets and task improvements |
| First environment | Finite grid and stochastic variant | Complete reference truth table |
| Observation | Partial observations are representable; first tabular task fully observable | Scope and leakage negative tests |
| Deep RL | Separate algorithm/backend profile, no assumed neural compiler in Gerbil | Code audit, gradient parity, benchmark and task evidence |
| Offline RL | Support-aware data contracts and BC baseline before IQL/CQL | Data coverage, OPE uncertainty and offline task results |
| On-policy concurrency | Version-pinned collector and learner; reject unsupported staleness | Race/failure injection and algorithm-specific data acceptance |
| Rewards vs safety | Distinct reward, cost, hard constraint and host authority | Negative reward-hacking cases |
| Multi-agent | Separate future specialist | AEC/Parallel semantics and policy-mapping contract |
| P08 game world | Relevant source, never required | Scoped observation/action adapter and D01/D02 revision locks |
| Algorithm qualification | Named task + implementation + budget + held-out baselines | Independent executed oracle and seed/range evidence |
| Production claims | No deployability implied | Separate user/host authority and real-world validation |

**Final design disposition:**

```text
P09_D01_RL_ADAPTIVE_INTELLIGENCE_DESIGN_PROPOSED /
ENVIRONMENT_POLICY_LEARNER_EVALUATOR_SEPARATE /
GERBIL_TABULAR_REFERENCE_FIRST / GAME_INDEPENDENT /
TERMINATION_NE_TRUNCATION / OBSERVATION_NE_HIDDEN_STATE /
REWARD_NE_COST_NE_PERMISSION / TRAIN_NE_EVALUATE /
TASK_RELEVANT_ALGORITHM_QUALIFICATION / OFFLINE_SUPPORT_AWARE /
P08_SOURCE_NOT_DEPENDENCY / NO_UNIVERSAL_RL_ALGORITHM /
A_B_C_OPTIONAL_BACKENDS / 72_FUTURE_TESTS / Q0_TO_Q11 /
UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED
```

> **Closing principle:** A learner may propose new decisions, but it does not earn authority over the environment, define its own evaluation standard, or turn training reward into proof of useful behavior. Select an algorithm because it accomplishes a declared task; verify that claim with independent observations and resource accounting. The world does not owe an agent the result it hoped to learn.