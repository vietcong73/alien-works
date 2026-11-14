# LFR-P10-D02 | Experimental & Research Infrastructure Architecture

**Document class:** Source-grounded specialist detailed-design proposal  
**Date:** 14 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `SCIENTIFIC RESEARCH` — Scientific Simulation & Experimental Systems  
**Edition:** P10 — Scientific Simulation & Experimental Systems  
**Design slice:** D02 — Experimental & Research Infrastructure  
**Primary design language:** Gerbil Scheme for study/run contracts, local execution, manifests, evaluators and evidence; qualified storage, statistics, telemetry and compute adapters by workload  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P10-D02-experimental-research-infrastructure-architecture-2026-11-14.md`  
**Status:** `SOURCE_GROUNDED_DESIGN_PROPOSED / 80_FUTURE_TESTS / 12_FUTURE_GATES / 30_INVARIANTS / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. Experimental execution, external data acquisition and deployment require separate authorization.

> **Design decision:** P10-D02 supplies a standalone, versioned experimental research infrastructure for planning studies, executing bounded trials, preserving run/artifact lineage, comparing candidates and issuing evidence-scoped qualification decisions. Its first release is a finite Gerbil experiment ledger with exact factorial and paired-comparison oracles. Domain solvers, learners, models, remote schedulers, databases and research-package exporters attach through qualified interfaces.

## 0. Objectives, first capability and boundaries

**Research question:** Can a Gerbil-centered system make experiments repeatable, comparable, independently auditable and efficiently orchestrated across scientific, software and quantitative workloads while preserving the distinct responsibilities of experimental design, execution, artifact storage and inference?

**First useful product:** `ExperimentLedgerLab` registers a sealed four-cell factorial study, executes each cell in an isolated bounded local worker, records trial and attempt identities, checks the exact effect coefficients, persists immutable manifests, replays a selected trial and emits an evidence-scoped qualification report. Additional fixtures test paired benchmarking and interrupted artifact commits.

**Coverage:** protocol registration; study design; deterministic and randomized trial schedules; parameters and data locks; run lifecycle; reproducibility grades; content-addressed artifacts; provenance; statistics; benchmark comparison; budget control; trace evidence; report composition; archival/export; optional external workers and research packages.

**Primary deliverables:** a stable CLI/record API, a queryable local experiment index, finite test fixtures, a study summary with per-claim evidence, and a portable experiment bundle. Dashboard and interactive UI are optional clients of this protocol.

**Evidence language:** `DESIGN_PROPOSED`, `RECORD_CONFORMANT`, `RECONSTRUCTABLE`, `REPLAY_MATCH`, `NUMERIC_EQUIVALENT`, `STATISTICALLY_EVALUATED`, `TASK_QUALIFIED`, `EXTERNALLY_REPLICATED` and `UNRESOLVED` refer to different claim scopes. They are issued only after their corresponding execution gates pass.

**System boundary:** The experiment owner defines task and acceptance; the domain adapter executes one job and returns domain evidence; D02 governs the study, trial, measurement, comparison, artifact and publication workflow; a separately authorized host performs external effects when requested.

**Dependency rule:** The finite reference uses K0 boundary vocabulary and D02-owned Gerbil code/ports. K1 mechanisms, P10-D01 simulation, P09-D01 learning, P09-D03 control, P09-D04 distributed execution, P05 persistent evidence and other domains integrate through explicitly selected adapters. D02 can run its exact fixtures with none of those specialist runtimes.

### 0.1 Independent scientific claims

| Claim | What evidence establishes it | Scope |
| --- | --- | --- |
| `RECORD_CONFORMANT` | Validated manifest, event transitions, artifact references and immutable IDs | Recorded experiment |
| `ENVIRONMENT_RECONSTRUCTABLE` | Pinned build/runtime/data closure and successful environment rebuild | Named platform and closure |
| `REPLAY_MATCH` | Independent rerun compared by declared bitwise/semantic/numeric comparator | Named job and equivalence profile |
| `STATISTICALLY_EVALUATED` | Predeclared design, validated sample units, uncertainty and comparator | Population/sample-design assumptions |
| `TASK_QUALIFIED` | All task-specific correctness, performance and evidence gates passed | Declared task, backend and budget |
| `EXTERNALLY_REPLICATED` | Independent group/toolchain/site reproduces selected scientific conclusion | Named external replication protocol |

## 1. Research and implementation evidence ledger

Evidence classes: `STANDARD` normative published specification; `DOC` official maintainer material; `LFR` previously specified design; `PROPOSED` D02 target. Documentation and inspected API affordances inform design decisions. They are not evidence that a Gerbil integration has been executed.

| Source | Observed facility / evidence | D02 transfer | Required falsification test |
| --- | --- | --- | --- |
| [MLflow Tracking](https://mlflow.org/docs/latest/tracking/) | Runs group metadata, params, metrics and artifacts into experiments | Study/run hierarchy; run-level metrics and artifact manifests | Record completeness and cross-run comparators |
| [DVC experiment management](https://doc.dvc.org/user-guide/experiment-management) | Experiment runs, parameters, metrics, plots, data and pipeline revisions | Versioned parameter/data closure; queued trial records | Changed parameter invalidates cache; restore only tracked inputs |
| [W3C PROV-O](https://www.w3.org/TR/prov-o/) | Entity/Activity/Agent provenance edges and relationships | Typed provenance graph with qualified responsibility links | No fabricated lineage or derivation by timestamp |
| [RO-Crate 1.2](https://www.researchobject.org/ro-crate/specification/1.2/) | Portable research objects and JSON-LD metadata | Optional portable archive/export profile | References, rights and closure validation |
| [OpenTelemetry](https://opentelemetry.io/docs/concepts/semantic-conventions/) | Common semantic attributes for traces, metrics and logs | Optional operational telemetry adapter | Telemetry may be sampled; do not infer lossless evidence |
| [Optuna](https://optuna.readthedocs.io/en/stable/faq.html) | Fixed sampler seeds; parallel search has nondeterministic outcomes | Sampler/selection provenance and search-order locks | Detect search-order drift and adaptive-selection bias |
| [SciPy bootstrap](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.bootstrap.html) | Paired resampling and confidence intervals | Independent comparator for uncertainty implementation | Paired indices preserved in resampling |
| [SciPy permutation test](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.permutation_test.html) | Exact or approximate paired permutation distributions | Exact small-N hypothesis-test reference | Correct null, pair mapping and exhaustiveness |
| [GNU Guix time-machine](https://guix-home.trop.in/Invoking-guix-time_002dmachine.html) | Pinned Guix revision/channel descriptions reconstruct package set | Environment closure manifest and optional Guix runner | Changing channel commit creates distinct environment identity |
| [Reproducible Builds](https://reproducible-builds.org/docs/definition/) | Specified outputs can be reproduced bit for bit under pinned inputs | Separate build reproducibility from result equivalence | Hash equal specified outputs, not logs by implication |
| [GitHub artifact attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations) | Signed build provenance binds workflow/commit/environment | Optional attestation import and verification | Do not treat self-declared digest as an attestation |
| [LFR P10-D01](LFR-P10-D01-scientific-physical-simulation-architecture-2026-11-14.md) | Domain-owned simulation model/solver/time/oracles; current review design | Simulation-result adapter and scientific comparator | No transfer of simulation step authority |
| [LFR K0/K1](LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md) | K0 typed identity/effect/result vocabulary; K1 optional mechanisms | Minimal common contracts with selected K1 integration | Design-level interfaces await compiled qualification |
| [LFR P05/P09-D04](LFR-P09-D04-distributed-systems-coordination-architecture-2026-11-14.md) | P05 durable research evidence and P09-D04 distributed coordination have distinct owners | Optional storage and distributed scheduler providers | Core ledger runs without their runtimes |

**Research conclusion:** existing tools provide useful components but solve different layers. MLflow is a tracking comparator, DVC a versioned pipeline/experiment comparator, PROV a provenance vocabulary, Guix a software closure provider, SciPy a statistical oracle, OpenTelemetry an observability protocol and RO-Crate a preservation/export protocol. D02 composes their *contracts* as selected capabilities, not their runtime dependency trees.

## 2. Architecture, ownership and dependency closure

```text
  Study owner / scientific task
        |
    Plan / Protocol / Acceptance criteria
        v
  +---------------- D02 PUBLIC PORT -------------------+
  | seal -> schedule -> observe -> compare -> publish |
  +-----------------------+----------------------------+
                          v
  +---------------- D02 GERBIL CORE ------------------+
  | study/trial/attempt identity + immutable plans     |
  | design matrix / candidate registry / budgets       |
  | local run state machine + append-only run ledger    |
  | artifact manifests + provenance + metric schemas   |
  | paired comparison / uncertainty / claim evaluator  |
  | exact reference fixtures + audit/report generation |
  +-----------------------+----------------------------+
                          | qualified ports
     +-------------+------+-------------+-------------+
     |             |                    |             |
 RunPort        RecordStorePort     ArtifactPort    Clock/RNGPort
     |             |                    |             |
 local fake /   atomic local /      hashed files /  virtual / pinned
 domain solver  transactional db    remote store    clocks + streams
     |             |                    |             |
      +------- optional telemetry/export/scheduler ----+

  Shared K0: identity / validation / outcomes / receipts
  Selected K1: mechanisms composed per job
  Domain integrations: P10-D01, P09, P08, P05, P09-D04, others
```

**Ownership:** D02 owns the identity and governance of experiment runs, statistical comparators, result catalogs and publication decisions. Each domain supplies its own model truth, execution outcome and task-specific validity evidence. Storage providers implement declared durability; schedulers transport bounded tasks; an experiment report is a derived artifact from immutable records.

| Class | Implementation / interface | Required in first reference | Contract |
| --- | --- | --- | --- |
| Shared foundation | K0 IDs, schema, status/receipt vocabulary | Yes | Versioned representation and validation |
| D02 core | Plan, run ledger, local scheduler, statistics, artifacts and oracles | Yes | Locally compiled Gerbil reference |
| Local capability ports | RunPort, RecordStorePort, ArtifactPort, ClockPort, RNGPort | Yes as interfaces | Deterministic fake/atomic file implementation |
| Qualified extensions | K1 M12/M13/M14/M16/M17; optional telemetry | No | Selected mechanism contract and evidence |
| Domain integrations | D01 simulation, P09 learning/control, P08 or other software | No | Inputs/results remain domain-owned |
| External infrastructure | P09-D04, MLflow, DVC, SQL/object storage, Guix, RO-Crate | No | Individually qualified adapters |

### 2.1 Separation of experiment state and domain state

D02 may request a run, observe a result, freeze a dataset reference and compare metrics. A scientific model's state trajectory, an RL policy update, a physical control command and a game's causal publication are committed by their respective domain host. D02 records their receipts and associated experimental conditions. This prevents a study retry from silently executing a second real-world effect.

D02 creates and owns its **experiment metadata ledger**. It can store immutable local run records independently of P05. P05 may provide a durable evidence collection, and D04 may provide remote scheduling, if those adapters are selected. A read-only data provenance edge does not create a dependency on either runtime.

**Authority boundary:** an experiment plan may include effect requests as metadata, but a `RunPort` capability must explicitly define effect scope, authorization, retry/idempotency and host receipts. The finite reference runs only synthetic pure tasks. Real instruments, financial systems and robots require their own host profiles.

## 3. Versioned study, trial, run and artifact contracts

**Identity domains:** `StudyId` groups a hypothesis/design; `ProtocolRevision` freezes hypotheses, sampling plan and acceptance criteria; `TrialId` denotes one exact parameter/data/design cell; `AttemptId` denotes one execution attempt of that trial; `ArtifactId` refers to immutable bytes and semantic kind. A rerun uses a new attempt ID and retains its trial ID. A revised protocol has a new revision and distinct eligibility for comparisons.

```text
StudySpec {
  study_id, study_revision, title, research_objective_ref,
  hypothesis_registry[], design_protocol_ref, candidate_registry[],
  sampling_unit_schema, data_split_policy_ref, preregistered_analyses[],
  evaluation_criteria_refs[], budget_profile_ref, access_scope,
  publication_policy_ref, owner_ref, K0_schema_revision
}
ProtocolSeal {
  study_id, study_revision, protocol_digest, sealed_at,
  accepted_candidates[], comparison_plan_ref, correction_policy_ref,
  trial_design_ref, evaluator_revision, permitted_deviation_policy
}
TrialSpec {
  trial_id, study_ref, protocol_digest, design_cell_ref, parameter_vector,
  parameter_schema_revision, dataset_manifest_digest, environment_lock,
  code_revision, run_adapter_revision, RNG_stream_spec, repetition_index,
  expected_metrics[], time_and_resource_budget, planned_comparison_group
}
RunAttempt {
  trial_id, attempt_id, attempt_generation, claim_token, worker_ref,
  start_evidence, environment_actual_ref, measured_inputs_digest,
  state, terminal_receipt_ref_optional, failure_ref_optional
}
ArtifactManifest {
  artifact_id, attempt_id, media_type, semantic_role,
  digest_algorithm, byte_digest, byte_length, canonicalization_ref,
  storage_uri, publication_state, access_class, provenance_parent_refs[]
}
MetricObservation {
  metric_ref, schema_revision, attempt_id, sample_unit_ref,
  value, unit, direction, aggregation_rule, uncertainty_ref_optional,
  source_artifact_ref, event_time, valid_status
}
```

**Canonical identity:** byte hashes use pinned algorithm and canonical serialization. Numeric equivalence is established separately through a typed comparator; two bytestrings of equal hash do not by themselves prove scientific validity. Experiment identity includes the actual execution closure, not merely a display-name tag.

**Plan changes:** any post-seal modification becomes a new protocol revision or explicit deviation event. Reporting labels preregistered and exploratory results separately, while preserving the exact history of candidate additions and exclusions.

## 4. Study lifecycle, scheduling and outcome accounting

```text
DRAFT -> VALIDATED -> SEALED -> SCHEDULED
                                |
          +---------- CLAIMED / RUNNING -----------+
          |                    |                   |
       FAILED               FINALIZING          CANCELLED
          |                    |
      RETRY_ELIGIBLE    ARTIFACTS_STAGED
          |                    |
          +------> NEW ATTEMPT  COMMITTED
                                     |
                             EVALUATED / REPORTED
```

A trial's logical status and each attempt's execution status have different namespaces. `FAILED`, `TIMED_OUT`, `CANCELLED`, `UNKNOWN` and `SUCCEEDED` all remain counted in design coverage and budget accounting. Missing runs are reported by cell and cause, instead of being dropped from denominators.

**State transitions:** `seal_protocol` validates plan closure; `schedule_trials` admits a bounded design; `claim_trial` issues generation-bound attempt identity; `record_observation` appends typed metrics; `stage_artifact` hashes and checks bytes; `commit_attempt` atomically publishes manifest and outcome; `compare_study` reads committed attempts under a frozen analysis protocol; `publish_verdict` seals the evidence packet.

**Retry rules:** retry creates an independent attempt with the same trial/condition fingerprint. The host adapter declares whether a failed attempt left effects. A retry reusing an old attempt ID is rejected; an external effect with unknown status remains pending reconciliation until a host query or approved effect policy resolves it.

**Budget rules:** cap studies, design cells, search proposals, concurrent attempts, wall time, CPU/memory, storage bytes, telemetry cardinality and report size. Admission is atomic against the planned budget; observed resource usage is retained even for rejected, partial and failed runs.

**Resumption:** sealed study manifest + committed attempts + uncommitted intent records determine a recovery queue. Resume may run missing trials, but does not mutate any prior attempt. Artifact replay and attempt identity are checked against a generation token.

## 5. Reproducibility closure, provenance and data lineage

`EnvironmentClosure` identifies: source commit/tree digest; dependency definitions and exact runtime/build revisions; compiler/flags; OS/kernel/library/toolchain profiles; architecture and CPU/GPU numeric capabilities where relevant; Guix channels commit locks when used; locale/timezone and relevant environment variables; container/image digest when used; input dataset content/partition digest; RNG algorithm and stream mapping; backend configuration; external service versions and availability ceiling.

**Reproducibility grades:**

- `PLAN_RECONSTRUCTABLE`: protocol, parameters, data and environment references resolve.
- `ENVIRONMENT_REBUILT`: a pinned execution closure has been successfully restored on a named platform.
- `ARTIFACT_BITWISE_MATCH`: the specified bytes match an independent rerun under the declared test.
- `NUMERICAL_EQUIVALENCE_MATCH`: output values meet a predeclared unit-aware norm/tolerance and comparison scope.
- `STATISTICAL_REPLICATION`: independent samples/procedures support the same declared task-level conclusion within defined uncertainty.
- `UNRESOLVED`: missing evidence or an unmatched comparator leaves the claim open.

A seed and a pinned Guix channel are useful controls. They do not alone seal GPU kernels, thread scheduling, measurement noise or external instruments. D02 keeps actual execution closure separate from requested closure so a silent environment fallback becomes an audit failure.

**Provenance grammar:** `Entity` (code/data/artifact), `Activity` (attempt/evaluation/export), `Agent` (actor or system responsible), `used`, `wasGeneratedBy`, `wasDerivedFrom`, `wasAssociatedWith`. Edges are typed and refer to exact immutable revisions. Time order does not automatically create a causal-derivation edge. Provenance import/export may map to PROV-O without forcing an RDF runtime into the local core.

**Artifact identity and access:** hashes are integrity witnesses with algorithm declarations; access control and origin authentication are separate. Signed build attestations can be consumed through an optional verifier. Secrets are redacted prior to artifact admission; redaction yields a derived artifact with explicit scope, never silently overwrites the original.

**Dataset lineage:** datasets carry content hash or immutable external version, schema, license/access class, split identity, sample-unit definition, missingness and preprocessing chain. Training/test leakage checks operate on actual sample IDs and lineage, including derived or duplicated rows when the task warrants it.

## 6. Durable experiment ledger and artifact publication

The baseline record store is a small transactional local adapter. Its authority scope includes study/trial/attempt metadata and committed artifact manifests. An artifact store holds bytes; a provenance registry holds relationships. They share atomicity only where the selected `RecordStorePort`/`ArtifactPort` contract proves it.

**Local commit protocol:** (1) create attempt under exclusive generation token; (2) write candidate artifact to a staging path; (3) validate hash/length/schema; (4) flush and atomically publish the content-addressed file when supported; (5) commit the manifest + attempt terminal receipt in a qualified transaction; (6) surface `COMMITTED` only after the declared durability boundary is met; (7) reclaim orphan staging data after audit/retention checks.

**Crash cases:** failure before staging; after bytes written before fsync; after staged bytes before manifest transaction; after manifest transaction but before client ACK; duplicate commit retry; corrupted bytes after storage; stale worker completing a replaced attempt. The core keeps `UNKNOWN` where the authoritative store cannot establish a terminal result.

**CAS and immutability:** a submitted `AttemptId` is unique; committed artifact digest and metric values cannot be rewritten under that ID. Corrections use a new derived artifact/revision with a linked supersession record. A store's at-least-once notification is distinct from exactly-once logical publication through idempotent transactions.

**Storage profiles:** A = local atomic filesystem + small index/ledger; B = transactional database plus qualified artifact files/object store; C = external experiment tracking/archive service. Evaluation and claim semantics remain D02-owned. P05 and D04 are optional integrations, useful for storage/remote delivery but unnecessary to run the finite fixtures.

## 7. Experimental design, sampling and candidate selection

The design protocol declares the **unit of independence**: trial/seed, physical specimen, patient, machine, time block, dataset split, node or environment. Repeated observations from one unit do not become independent replicates through aggregation.

**Design families:** exact full-factorial and fractional-factorial; randomized complete block; paired A/B; stratified/clustered sampling; repeated measurements; sequential and adaptive optimization; ablation; held-out replication. Each family uses its own randomization, assignment and inference contract. Descriptive benchmarking can proceed without a hypothesis test when the task requires only a bounded deterministic oracle.

**Preregistration:** fixed primary metric and direction; practical improvement threshold; sample size/stop rule; control and baseline identity; data splits; exclusion policy; multiplicity policy; failure handling; selected statistical estimator; uncertainty method; target population; budget. Exploratory analyses are reported with their search/exploration history.

**Candidate tour:** classify the job, pick compatible algorithms or models, pin implementation and hyperparameters, run small structural/semantic probes, screen for task validity, then compare on the declared objective. A missing oracle yields `UNRESOLVED` with a specific evidence gap. An unsupported candidate is `NOT_APPLICABLE_FOR_TASK`, while a failing candidate receives a task-specific falsification receipt.

**Adaptive search:** optimization trials record suggestion algorithm, sampler state, objective revision, full search history, pruning/early-stop decisions and trial completion order. A fixed seed constrains one sampler, but asynchronous completions can alter future suggestions. Final claims use an independent frozen evaluation set or a declared nested/selective-inference procedure, including comparison-family accounting.

**Missingness:** `NOT_RUN`, `INVALID_INPUT`, `FAILED_EXECUTION`, `TIMED_OUT`, `CENSORED`, `EXCLUDED_PREDECLARED`, `EXCLUDED_POST_HOC` and `OBSERVED` are distinct. Reports present denominator tables and sensitivity analysis where missingness may be informative.

## 8. Benchmarks, statistical analysis and claim gates

**Benchmark unit:** a comparable observation couples workload identity, correctness envelope, hardware/environment revision, resource meter, repetition/block and measured timing. Warm-up, cache state, background load, garbage collection, CPU frequency/power and threading are recorded according to the profile. Latency distributions and tail costs accompany throughput and memory.

**Fair candidate comparison:** accuracy/quality gate first; match accuracy where possible or publish error-versus-cost frontier. Use shared workload IDs and paired blocks for matched designs, randomized order when order effects matter, and independent counters for failed runs. End-to-end includes serialization, loading, IPC, retries and staging.

**Statistical plan:** estimator `delta_hat`, comparison direction, confidence level, bootstrap/permutation method, cluster/paired structure, multiplicity/selection policy and meaningful effect size are specified before opening evaluation data. An exact finite comparator is preferred where possible. Bootstrap intervals are not calibrated automatically under dependent samples or tiny degenerate designs.

**Report content:** predeclared objectives; observed eligible/missing attempts; per-candidate summary; uncertainty; practical threshold; environment variance; resource-cost distribution; task-specific validity; failed hypotheses; selection history; applicability ceiling. Job-dependent efficiency is evaluated against the named workload, correctness/quality envelope, objective metric, resource budget and experimental protocol.

**Performance qualification status:** `EQUIVALENCE_PASSED`, `TASK_OBJECTIVE_PASSED`, `CONDITIONALLY_QUALIFIED`, `REJECTED_FOR_TASK`, `UNRESOLVED`, `OUT_OF_SCOPE`. Scientific validity and physical validation remain the responsibility of the associated domain owner; D02 verifies the comparison and evidence protocol.

## 9. Exact reference fixture A: FactorialLedger

**Purpose:** verify experiment design registration, factorial parameter identities, exact arithmetic, balanced assignment, provenance and deterministic report generation.

For `a, b in {-1,+1}`, define the synthetic oracle:

```text
y(a,b) = 3 + 2*a + b + a*b
X row = [1, a, b, a*b]
Cell order = [(-1,-1),(-1,+1),(+1,-1),(+1,+1)]
Expected y = [1, 1, 3, 7]
Least-squares coefficients = [3, 2, 1, 1]
Factor A mean difference = 4
Factor B mean difference = 2
Interaction difference-in-differences = 4
```

The four-cell matrix is full-rank and orthogonal under the specified coding. A complete noiseless run recovers the exact coefficients. A duplicated cell with a missing cell creates a rank deficiency. A swapped cell label changes provenance and should be rejected before estimation. All cells use a sealed protocol and distinct trial IDs; retry attempts remain traceable.

**Required comparators:** exact design-matrix hash; 4-of-4 cell coverage; rank four; coefficient/error oracle; factorial main-effect and interaction contrasts; artifact checksum; independent replay of the report from a shuffled event order.

**Negative cases:** duplicate trial ID, wrong sign coding, changed parameter after sealing, missing arm, sampled result with NaN/Inf, post-hoc selected metric, corrupted artifact, stale attempt generation and untracked environment substitution.

**Claim ceiling:** finite exact experimental-registration and arithmetic compliance. These synthetic results do not establish statistical power, scientific effect validity or external reproducibility.

## 10. Exact reference fixture B: PairedBenchmarkLab

**Purpose:** verify paired blocks, exact finite permutation inference, fairness checks, quality gating and explicit small-sample uncertainty.

```text
Block ID       1   2   3   4
Reference A   10  12   9  11
Candidate B    8  10   7   9
Paired A-B     2   2   2   2
Mean paired improvement = 2 time units
Exact two-sided sign-flip p-value = 2/16 = 0.125
```

Treat the table as synthetic, measured at four independent matched blocks under a frozen deterministic generator. The exact sign-flip test enumerates 16 sign configurations under the sharp paired exchangeability null; the two extreme configurations yield the specified p-value. All differences being equal makes ordinary bootstrap estimates degenerate, so the report explicitly identifies limited inferential generality.

**Required checks:** block ID join; permutation enumerator count; two-sided tail handling; unit consistency; order randomization metadata; quality/accuracy eligibility; per-run failures retained; resource-censoring flags; no unconditional statistical-significance claim at 5%.

**Adversarial edits:** shuffle candidate records without preserving block IDs; drop the slowest A result; selectively exclude a timeout; report 4 repeats on one machine as four independent machines; change metric direction; compare unequal accuracy and claim efficiency from raw speed alone.

**Claim ceiling:** correctness of D02's paired-statistic machinery on a finite fixture, not any actual solver speed advantage.

## 11. Reference fixture C: CrashReplayLedger and integration fixture D

### 11.1 CrashReplayLedger

A four-event protocol tests commit truth: `PLAN_SEALED`, `ARTIFACT_STAGED`, `MANIFEST_COMMITTED`, `ACK_EMITTED`. Inject crashes after each event and rerun recovery with fresh worker generations. A lost ACK after the manifest commit must return the original committed receipt on retry, while a crash before manifest commit must not surface the staged bytes as a published artifact.

**Invariants:** one terminal manifest per attempt ID; exact byte digest; no duplicate committed outcome; orphan staging detectability; replay from committed ledger yields equivalent study catalog; stale attempt cannot commit; `UNKNOWN` persists when storage cannot determine the outcome.

### 11.2 P10-D01 `OscillatorLab` integration

A D02 trial sends `(model_ref, solver_ref, tolerance, seed_stream, time_grid)` through a qualified `RunPort`. P10-D01 returns a domain-owned `SimulationReceipt` containing trajectory digest, analytic-error comparator and budget usage. D02 records these as typed metrics and compares repeated solver configurations only after the D01 scientific validity gates and the D02 fairness protocol have both passed.

The domain retains numerical state/time, event localization and physics claims; D02 owns plan revision, trial assignment, cross-run analysis and portable evidence packet. Reference fixtures A–C run without D01 installed. Similar adapters may later bind P09-D01 learners, P09-D03 controllers or independent software benchmarks.

### 11.3 Evidence portability fixture

Export a minimal bundle containing sealed protocol, dataset/closure manifest, four run receipts, exact scores, provenance edges, comparison protocol, report and artifact digests. Import into a clean local instance and compare canonical records. A selected RO-Crate adapter may serialize additional linked metadata and third-party files while preserving privacy/license/access restrictions.

## 12. Gerbil implementation and backend selection

| Profile | D02-owned functionality | Appropriate job | Qualification |
|---|---|---|---|
| **A — Gerbil local reference** | Typed plans, deterministic design generator, local attempts, small exact statistics, atomic file ledger | Small-to-medium experiments and correctness fixtures | Compiler/API probes, artifact crash tests, exact reference oracles |
| **B — Gerbil + qualified native/data/statistical adapter** | Contracts, study governance and reports; specialized SQL/numeric/stats backend executes selected heavy job | Large comparisons, high-volume metrics, native statistical kernels | ABI/numeric parity, storage consistency, end-to-end throughput |
| **C — Gerbil + external research/experiment service** | Protocol/identity, qualification and evidence admission; remote service stores or executes declared work | Federated studies, distributed sweeps, shared experiment service | Auth, privacy, API version, idempotency, receipt parity, outage recovery |

**Backend-to-job matrix:** (i) local experiment ledger → Gerbil + atomic local adapter; (ii) parameter/data pipelines → versioned DVC-like adapter when chosen; (iii) telemetry → OpenTelemetry only for operations diagnostics; (iv) large numerical comparisons → qualified statistics backend; (v) distributed trial scheduling → D04 integration; (vi) portable paper/data archive → RO-Crate adapter; (vii) reproducible Guix closure → pinned channel lock and `time-machine` runner.

### 12.1 Proposed namespace

```text
lfr/p10/experiments/
  contracts/{study,protocol,trial,attempt,metric,artifact,claim}.ss
  design/{factorial,paired,block,randomization,search-plan}.ss
  execution/{state-machine,local-runner,budgets,checkpoint}.ss
  provenance/{lineage,closure,validation}.ss
  store/{ledger,manifest,atomic-file,cas,retention}.ss
  analysis/{exact-oracle,contrasts,permutation,bootstrap,missingness}.ss
  benchmark/{eligibility,measurement,paired-comparison,pareto}.ss
  reporting/{claim,qualification,summary,archive}.ss
  ports/{run,record-store,artifact,clock,rng,evaluator}.ss
  integrations/{p10-d01,p09-d01,p09-d03,p09-d04,p05,guix,mlflow,dvc,ro-crate,otel}.ss
  tests/{fixtures,plan,execution,storage,inference,interop}.ss
```

### 12.2 Public reference API

```text
create_study(study_spec) -> StudyRef | ValidationError
seal_protocol(study_ref, protocol_spec) -> ProtocolSeal | Rejection
expand_design(protocol_ref, design_profile) -> TrialManifest | Error
schedule_trials(manifest_ref, budget) -> ScheduleReceipt | Rejection
claim_trial(trial_ref, attempt_generation) -> AttemptLease | Conflict
record_metric(attempt_ref, metric_observation) -> MetricReceipt | Error
stage_artifact(attempt_ref, bytes_ref, semantic_kind) -> StageReceipt | Error
commit_attempt(attempt_ref, outcome, staged_manifest) -> CommitReceipt | Unknown
reconcile_attempt(attempt_ref) -> CommitReceipt | Unknown
compare_study(study_ref, analysis_protocol) -> ComparisonReport | Unresolved
export_evidence(study_ref, profile) -> ArchiveRef | MissingClosure
check_qualification(study_ref, claim_profile) -> Verdict + EvidenceRefs
```

Names are semantic descriptions to be compiled and reconciled with pinned Gerbil/Gambit toolchains during implementation. Macros can help trusted authoring; serialized study specifications enter through validated data parsers. A long-running experiment worker cannot mutate the protocol after its seal.

## 13. Invariants and typed failure semantics

### 13.1 Core invariants I01–I30

1. **I01 — Protocol identity:** A sealed protocol digest binds its exact revision and acceptance rules.
2. **I02 — Trial identity:** Each trial binds one immutable design cell, data and environment request.
3. **I03 — Attempt identity:** Each execution attempt has a unique generation-bound reference.
4. **I04 — Trial/attempt separation:** Retries are new attempts attached to the same logical trial.
5. **I05 — Frozen design:** Changed factor/candidate/split definitions create an explicit revision.
6. **I06 — Metric semantics:** Metric value always binds units, direction, source and comparator version.
7. **I07 — Observation status:** Missing, invalid, partial and censored values remain distinguished.
8. **I08 — Data lineage:** Every input dataset binds a version and declared sampling unit.
9. **I09 — Input closure:** Code, inputs, RNG and relevant environment versions are recorded.
10. **I10 — Actual closure:** Executed environment deviations are visible from requested locks.
11. **I11 — RNG streams:** Trial replication and search sampler streams have distinct identities.
12. **I12 — Artifact digest:** A committed artifact binds exact bytes and digest algorithm.
13. **I13 — Atomic manifest:** Published artifact manifest references only admitted content.
14. **I14 — Commit idempotency:** A repeated attempt commit with same identity returns the prior receipt.
15. **I15 — Conflict detection:** Same identity with divergent payload or outcome is rejected.
16. **I16 — Recovery truth:** Unknown or interrupted attempts never auto-promote to success.
17. **I17 — Ledger immutability:** Corrections create new linked events; prior receipts remain addressable.
18. **I18 — Budget accounting:** Every attempt consumes observed resources within declared accounting.
19. **I19 — Selection provenance:** Each adaptive choice records sampler state, order and prior observations.
20. **I20 — Sampling units:** Independent sample count reflects design units, not raw trace count.
21. **I21 — Pair matching:** Paired comparisons join observations by exact block/subject identity.
22. **I22 — Split isolation:** Held-out evaluation references are distinct from training/selection inputs.
23. **I23 — Comparator lock:** Statistical method, tails, confidence and multiplicity remain pinned.
24. **I24 — Benchmark validity:** Correctness/accuracy eligibility precedes cost-performance promotion.
25. **I25 — Censored results:** Timeouts, failures and post-hoc exclusions are counted and explained.
26. **I26 — Provenance relation:** Derivation and responsibility edges cite exact entities and activities.
27. **I27 — Access scope:** Artifacts and datasets obey access/retention scope of their owners.
28. **I28 — Domain authority:** D02 does not rewrite a domain model, policy or external effect outcome.
29. **I29 — Standalone core:** Finite fixtures compile and run without other specialist runtimes.
30. **I30 — Claim ceiling:** Release claims name completed gates and their model/backend/data scope.

### 13.2 Typed outcomes

`PROTOCOL_INVALID`, `REVISION_CONFLICT`, `TRIAL_DUPLICATE`, `ATTEMPT_STALE`, `PARAMETER_DRIFT`, `DATASET_MISSING`, `ENVIRONMENT_DRIFT`, `RUN_FAILED`, `RUN_TIMED_OUT`, `RUN_CANCELLED`, `RUN_UNKNOWN`, `ARTIFACT_STAGED`, `ARTIFACT_CORRUPT`, `COMMIT_CONFLICT`, `STORAGE_UNAVAILABLE`, `REPLAY_MISMATCH`, `METRIC_INVALID`, `INSUFFICIENT_SAMPLE`, `PAIRING_INVALID`, `SELECTION_LEAKAGE`, `INFERENCE_UNRESOLVED`, `BUDGET_EXHAUSTED`, `ACCESS_DENIED`, `TASK_REJECTED`, `TASK_QUALIFIED`.

Each failure includes relevant study, protocol, trial, attempt and artifact references; last authoritative state; known retry rights; missing evidence and a bounded explanation. `UNRESOLVED` is a valid verdict when independent data or the required oracle is unavailable.

## 14. Qualification plan: 80 proposed tests and 12 gates

The test register is a set of proposed procedures, not executed results. An implementation qualifies only the claim-specific subset plus all mandatory gates for that claim. Profile-specific `NOT_APPLICABLE` is a declared outcome with rationale, not a way to bypass a failing applicable test.

### 14.1 Qualification gates Q0–Q11

| Gate | Decision condition | Required evidence |
| --- | --- | --- |
| **Q0 — Source and environment closure** | Pinned Gerbil/toolchain, exact design, dependency and environment lock | Source digests, K0 version, local-core closure and build transcript |
| **Q1 — Contract and schema validity** | IDs, manifest schemas, units and reject paths validated | Golden vectors, malformed inputs, protocol revision negative cases |
| **Q2 — Design and protocol sealing** | Factorial/paired assignment, sampling plan and freeze semantics exact | Design-matrix rank, randomization lock and immutable protocol witness |
| **Q3 — Run lifecycle and budgets** | Attempts, retry identities and bounded execution audited | Lifecycle state trace, quotas, timeout/reconciliation cases |
| **Q4 — Artifact persistence and recovery** | Selected storage claim survives specified crash/restart | Atomic commit, checksum, staging and recovery matrix |
| **Q5 — Provenance and data lineage** | Dataset, entity/activity, code and artifact closure complete | Provenance graph edges, leakage fixtures, missing source failures |
| **Q6 — Reproducibility and equivalence** | Specified replay/rebuild class demonstrated | Independent rerun, exact/numeric comparator, env drift negatives |
| **Q7 — Statistical and selection validity** | Estimators/uncertainty match preregistered design | Exact factorial and permutation oracle, multiplicity/holdout checks |
| **Q8 — Benchmark and performance fairness** | Quality eligibility, matched workload and resource accounting | Paired measurement, warm-up, failure/tail reporting, cost evidence |
| **Q9 — Access and research governance** | Permission, redaction, retention and policy controls checked | Access-denial, privacy labels, export scope and correction audit |
| **Q10 — Selected external adapter** | One chosen domain/backend/export/scheduler adapter meets semantics | Per-adapter conformance, failure and version/parity evidence |
| **Q11 — Independent audit and claim** | Verifiable report binds all passed and unresolved gates | Readback, artifact hashes, independent oracle and claim ceilings |

### 14.2 Test register (T01–T80)

Each test has an independently specified expected result. The matrix is divided into eight evidence domains so progress can be reported without conflating implementation, data preparation, comparison and scientific validity.

#### Protocol and design

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T01** | Study creation | Valid study receives unique revisioned ID |
| **P10-D02-T02** | Malformed study | Invalid schema rejected with field-specific error |
| **P10-D02-T03** | Protocol seal | Digest preserved after seal |
| **P10-D02-T04** | Post-seal mutation | Protocol modification creates new revision or denial |
| **P10-D02-T05** | Trial matrix complete | Factorial four cells exactly once |
| **P10-D02-T06** | Factor label inversion | Wrong sign-coded factor detected |
| **P10-D02-T07** | Duplicate trial ID | Collision rejected |
| **P10-D02-T08** | Sampling unit schema | Unit type and strata validated |
| **P10-D02-T09** | Candidate registry drift | Late candidate addition recorded as revision |
| **P10-D02-T10** | Exact factorial rank | Four-cell design rank equals four |

#### Run lifecycle

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T11** | Trial scheduling | Bounded schedule contains all eligible trials |
| **P10-D02-T12** | Claim attempt | Unique generation-bound attempt issued |
| **P10-D02-T13** | Duplicate claim | Conflicting worker receives stale/conflict |
| **P10-D02-T14** | Run start state | State transition and actual environment recorded |
| **P10-D02-T15** | Run terminal state | Success requires terminal receipt |
| **P10-D02-T16** | Run timeout | Timeout included in denominator and budget |
| **P10-D02-T17** | Run cancellation | Cancellation cannot masquerade as zero score |
| **P10-D02-T18** | Retry linkage | New attempt attached to original trial |
| **P10-D02-T19** | Stale worker finish | Old generation rejected |
| **P10-D02-T20** | Budget exhaustion | Typed exhausted receipt and partial evidence |

#### Data and provenance

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T21** | Dataset digest | Input content lock checked |
| **P10-D02-T22** | Dataset version drift | Changed source produces new identity |
| **P10-D02-T23** | Untracked data input | Input closure fails |
| **P10-D02-T24** | Split leakage | Same unit across train/test detected |
| **P10-D02-T25** | Derived-row leakage | Duplicate lineage carries split conflict |
| **P10-D02-T26** | Actual environment drift | Unexpected backend/toolchain revision flagged |
| **P10-D02-T27** | RNG stream identity | Trial stream distinct from sampler stream |
| **P10-D02-T28** | Provenance derivation | Entity/Activity relation preserved |
| **P10-D02-T29** | Fabricated provenance | Unknown artifact parent rejected |
| **P10-D02-T30** | Redacted export lineage | Derived/redacted artifact trace retained |

#### Artifacts and recovery

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T31** | Stage artifact | Byte digest and length verified |
| **P10-D02-T32** | Commit artifact | Manifest published under declared atomicity |
| **P10-D02-T33** | Staged orphan | Uncommitted artifact excluded from catalog |
| **P10-D02-T34** | Crash before bytes flush | No durable output claimed |
| **P10-D02-T35** | Crash before manifest | Recovery retains staging status |
| **P10-D02-T36** | Crash after manifest before ACK | Retry returns original committed receipt |
| **P10-D02-T37** | Divergent duplicate commit | Same attempt conflicting digest rejected |
| **P10-D02-T38** | Corrupt artifact | Readback digest failure surfaced |
| **P10-D02-T39** | Atomic journal replay | Committed catalog reconstructed |
| **P10-D02-T40** | Storage unavailable | Unknown durability recorded without success |

#### Reproducibility and replay

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T41** | Environment rebuild | Pinned environment restored or typed unavailable |
| **P10-D02-T42** | Bitwise exact repeat | Specified bytes match under defined closure |
| **P10-D02-T43** | Numeric tolerance replay | Unit-aware comparator passes within tolerance |
| **P10-D02-T44** | Numeric mismatch | Beyond-tolerance result rejected |
| **P10-D02-T45** | Seed-only mismatch | Unpinned scheduling prevents exact claim |
| **P10-D02-T46** | Guix channel drift | Changed channel commit changes lock |
| **P10-D02-T47** | Thread-order change | Reduction/ordering difference reflected |
| **P10-D02-T48** | Replay dataset missing | Missing input creates unresolved |
| **P10-D02-T49** | Independent reconstruction | New host reads portable closed manifest |
| **P10-D02-T50** | External replication claim | Requires separate independent evidence |

#### Statistics and selection

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T51** | Factorial effects | Coefficients [3,2,1,1] exact |
| **P10-D02-T52** | Factorial main contrasts | Effects A=4 and B=2 |
| **P10-D02-T53** | Factorial interaction | Difference-in-differences=4 |
| **P10-D02-T54** | Missing factorial cell | Rank/coverage failure |
| **P10-D02-T55** | Paired mean difference | Mean A-B equals 2 |
| **P10-D02-T56** | Exact paired permutation | Two-sided p-value equals 0.125 |
| **P10-D02-T57** | Broken pair mapping | Block mismatch rejected |
| **P10-D02-T58** | Multiplicity policy | Unregistered multiple comparison detected |
| **P10-D02-T59** | Holdout contamination | Selection/evaluation overlap flagged |
| **P10-D02-T60** | Post-hoc exclusion | Changed denominator disclosed and scoped |

#### Benchmark and budget

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T61** | Equal workload | Shared workload IDs required |
| **P10-D02-T62** | Accuracy gate | Fast inaccurate candidate excluded from speed claim |
| **P10-D02-T63** | Warm-up protocol | Measured and warm-up samples distinct |
| **P10-D02-T64** | Clock mismatch | Timing source labels and calibration checked |
| **P10-D02-T65** | Tail latency | p50/p95/p99 reflect defined valid sample set |
| **P10-D02-T66** | Censored timeout | Timeout retained with censor status |
| **P10-D02-T67** | Resource limit | Memory/CPU/storage receipt checked |
| **P10-D02-T68** | Sequential/randomized order | Order assignment recorded |
| **P10-D02-T69** | Pareto comparison | Error-versus-cost frontier correct |
| **P10-D02-T70** | Performance platform scope | Claim binds named host and version |

#### Integration and audit

| Test ID | Fixture/action | Expected comparator |
| --- | --- | --- |
| **P10-D02-T71** | D01 adapter | Oscillator receipt remains domain-authoritative |
| **P10-D02-T72** | P09 learning adapter | Study ledger records policy/dataset refs without mutation |
| **P10-D02-T73** | D04 scheduler adapter | Duplicate delivery returns one logical attempt commit |
| **P10-D02-T74** | P05 storage adapter | Selected provider durability matches declaration |
| **P10-D02-T75** | Local standalone | No specialist imports required for reference fixtures |
| **P10-D02-T76** | Research bundle export | Manifest/digests/reference closure validated |
| **P10-D02-T77** | Research bundle import | Readback reproduces canonical study index |
| **P10-D02-T78** | Unknown host effect | No auto-retry without reconciliation policy |
| **P10-D02-T79** | Claim gate omissions | Omitted mandatory gate blocks promotion |
| **P10-D02-T80** | Independent report regeneration | Frozen ledger reproduces same metrics and exclusions |

### 14.3 Claim and release matrix

| Profile | Required gates | Permitted claim |
| --- | --- | --- |
| Finite reference lab | Q0/Q1/Q2/Q3/Q4/Q5/Q7/Q9/Q11; applicable Q6/Q8 | `REFERENCE_CONTRACT_QUALIFIED` within exact fixtures |
| Local reproducible experiment | Above + Q6/Q8 | `LOCAL_REPLAY_QUALIFIED` for declared environment/equivalence |
| Domain experimental campaign | Local gates + selected domain validity + Q10 | `DOMAIN_EXPERIMENT_CONFORMANT` for named adapter and task |
| Research statistical comparison | Q0/Q1/Q2/Q3/Q5/Q7/Q8/Q11 | `STATISTICALLY_EVALUATED` under declared sample assumptions |
| Durable external tracking | Q0/Q1/Q3/Q4/Q5/Q9/Q10/Q11 | `TRACKING_ADAPTER_QUALIFIED` for named store/failure model |
| Independent replication | Claim-specific gates and independently audited rerun protocol | `EXTERNALLY_REPLICATED` for named conclusion and conditions |

**Qualification policy:** T01–T80 are the total design register. Every core release includes T05, T10, T36, T51–T56, T75 and T80, plus applicable negative cases for its declared profile. Remaining cases are selectively mandatory for the selected claim. A statistical-inference claim requires the statistical oracle tests, while a data-storage claim requires crash recovery. Each report publishes coverage, omitted tests, applicability and unresolved outcomes.

## 15. Implementation sequencing post separate authorization

1. **D02-00 — Evidence and toolchain closure:** pin K0/Gerbil/Gambit, choose canonical hashes and local storage primitives, record exact source documentation revisions.
2. **D02-01 — Immutable contracts and planning:** study/protocol/trial/attempt manifests, schema validators, full-factorial design generator and static negative tests.
3. **D02-02 — First working `FactorialLedger`:** pure Gerbil finite oracle, deterministic local runner, run ledger, 2x2 exact effects and artifact index.
4. **D02-03 — `PairedBenchmarkLab`:** matched-block comparator, exact sign-flip oracle, failure/eligibility accounting and report tables.
5. **D02-04 — `CrashReplayLedger`:** atomic local commit, fault schedule, checksum readback, correction events and recovery trace.
6. **D02-05 — Guix/reproducibility:** exact channels/toolchain/data closure, bitwise/numeric equivalence profiles, external rerun fixtures.
7. **D02-06 — Optional domain adapters:** P10-D01 oscillator campaign first; additional P09/P05/D04/MLflow/DVC/RO-Crate integrations by named user workload.
8. **D02-07 — Independent audit:** execute applicable Q0–Q11, publish qualification matrix, determine task-specific evidence ceilings and lock a migration-ready portable archive.

**Task allocation:** the initial Gerbil standalone reference is prioritized over constructing a web dashboard or distributed service. A dashboard is justified after the run ledger and query/report contracts have passed their tests; scheduling services are justified by measured throughput/fault requirements.

## 16. Open engineering decisions

| Decision | Proposed baseline | Evidence required before expansion |
| --- | --- | --- |
| Local persistent store | Atomic files + small index and journal | Crash consistency, restart and checksum evidence |
| Canonical serialization | Explicit versioned sorted-key form | Round-trip, Unicode/float/NaN policy and golden digests |
| Source environment | Pinned Guix channel manifest where installed | Build/execute evidence on target Guix system |
| Statistical kernel | Gerbil exact arithmetic for finite fixtures | Finite oracles before scalable native statistics |
| Adaptive optimization | Separate sampler-state interface | Search replay, holdout and selection bias tests |
| Distributed runs | D04 adapter only under chosen workload | Idempotency and remote recovery evidence |
| Durable evidence exchange | P05 or external store if selected | Storage ownership and transaction semantics |
| Research packaging | Portable D02 manifest, optional RO-Crate 1.2/1.3 | Archive validation and rights/access audit |
| Physical/scientific claims | Domain owner supplies validation evidence | Model and measurement comparison by domain |
| Statistical multiplicity | Preregistered comparison family and correction | Exact finite selection and coverage tests |
| UI/dashboard | Read-only queries over D02 records | Data freshness, access and consistency checks |
| Scope of reproducibility | Separate build/run/numeric/scientific profiles | Independent reconstruction or replication receipts |

## 17. Final design verdict

**Design maturity:** source-grounded architecture with a standalone Gerbil reference core, distinct study/trial/attempt identities, frozen experimental protocols, reference design and statistical oracles, local durable ledgers, reproducibility and provenance contracts, task-specific benchmark qualification, 30 invariants, 80 proposed tests and 12 qualification gates. No implementation or execution qualification is claimed.

```text
P10_D02_EXPERIMENTAL_RESEARCH_INFRASTRUCTURE_DESIGN_PROPOSED /
STUDY_PROTOCOL_TRIAL_ATTEMPT_ARTIFACT_IDENTITIES_SEPARATE /
GERBIL_FINITE_REFERENCE_FIRST / EXACT_FACTORIAL_AND_PAIRED_ORACLES /
LOCAL_LEDGER_AND_ATOMIC_MANIFEST / REPRODUCIBILITY_GRADES /
PROVENANCE_DATA_SOURCE_AND_ENVIRONMENT_CLOSURE /
PREREGISTERED_COMPARATORS / MISSINGNESS_AND_SELECTION_ACCOUNTING /
DOMAIN_SCIENTIFIC_VALIDITY_OWNER_SEPARATE /
P10_D01_P09_D04_P05_K1_OPTIONAL_INTEGRATION_PORTS /
A_B_C_BACKENDS_SELECTED_BY_JOB / THIRTY_INVARIANTS /
80_FUTURE_TESTS / Q0_TO_Q11 / NO_RUNTIME_EVIDENCE /
NO_BENCHMARK_PASS / UNIMPLEMENTED / DOCUMENT_PUBLISHED
```

> **Closing principle:** A good experiment infrastructure makes an observation inspectable, a comparison fair, an omission visible, and a conclusion bounded by evidence. Research quality is established through the sequence from protocol to measurement to independent challenge, not by the attractiveness of a dashboard.