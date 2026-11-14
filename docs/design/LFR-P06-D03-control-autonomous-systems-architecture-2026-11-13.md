# LFR-P06-D03 | Adaptive Intelligence & Autonomous Systems / Control & Autonomous Systems

**Document class:** Source-grounded specialist detailed-design proposal  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `ADAPTIVE SYSTEMS` — Adaptive Intelligence & Autonomous Systems  
**Edition:** P06 — Adaptive Intelligence & Autonomous Systems  
**Design slice:** D03 — Control & Autonomous Systems  
**Primary design language:** Gerbil Scheme for versioned contracts, finite reference plants, deterministic control and supervision; qualified native/solver/service adapters when measured requirements justify them  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P06-D03-control-autonomous-systems-architecture-2026-11-13.md`  
**Status:** `SOURCE_GROUNDED_DESIGN_PROPOSED / 80_FUTURE_TESTS / 12_FUTURE_GATES / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. Implementation, installation, device access and deployment follow separate authorization.

> **Design decision:** P06-D03 composes a **plant model, measurement/estimation path, control candidate, supervisory admission and effect host** through exact time-, unit-, model- and policy-version contracts. The baseline is a finite simulation with analytic transition and robust-admissibility oracles. Classical controllers, optimization-based controllers and learned candidates are task-selected alternatives. Safety evidence is qualified against a declared plant, disturbance, observation and execution envelope.

## 0. Objectives, capability, and boundaries

**Research question:** Can a Gerbil-led control specialization supply operationally useful closed-loop control, estimation, hybrid supervision and autonomy integration, with explicit claims about feasibility, stability, timing and authority, while allowing numerical solvers or native control runtimes to perform measured heavy computation?

**First useful product:** `BoundedAxis`, a discrete scalar plant laboratory with an exact transition oracle, bounded actuator, optional observation interval and bounded disturbance. It supports P/PI/PID candidates, scalar feedback, robust one-step admissibility, supervisory selection, trace replay and a task-relative evaluation report. A later `DoubleIntegrator` fixture introduces coupled position/velocity dynamics and stopping-distance constraints.

**In scope:** discrete and continuous plant descriptions; linear/nonlinear/hybrid modes; model/units/time validity; state estimation and uncertainty; reference tracking; PID/LQR/MPC candidates; supervisory arbitration; bounded fallback; observer/controller scheduling; operational authority; effect receipts; synthetic experiments; calibration; model mismatch; algorithm qualification; optional learning and multi-agent proposal ports.

**Adjacent owners:** P06-D01 owns learner and policy-update lifecycle; P06-D02 owns multi-agent behavior and joint proposals; P06-D04 will own distributed delivery/consistency; P07-D01 owns general scientific/physical simulation; P05-D00 owns formal/symbolic proof machinery if later designed. K0 owns shared boundary vocabulary, and K1 mechanisms are individually optional. P05 world causality and scene truth remain with P05.

**Execution scope:** the first qualification class is `SIMULATION_CONTROL`. Real plant control is a separately qualified `DEVICE_CONTROL` class with hardware-specific safety case, operator authority, lifecycle, watchdogs, I/O interface and timing evidence. Finance/robotics examples are interface fixtures, not executable trading or robot-driving profiles.

### 0.1 Four distinct success claims

| Claim | Necessary evidence | Claim ceiling |
|---|---|---|
| `MODEL_VALIDATED` | Units, transition semantics, parameter provenance, oracle and uncertainty assumptions | Declared model and initial-condition envelope |
| `CONTROL_QUALIFIED` | Closed-loop task metrics, stability/constraint evidence, baselines, independent rollouts | Named plant, task, controller and time profile |
| `SUPERVISION_QUALIFIED` | Monitor correctness, freshness, switch timing, fallback feasibility and negative tests | Named operational envelope and supervisory configuration |
| `DEVICE_ADMISSIBLE` | Host permission, hardware/OS deadlines, device state, operator and hazard-specific approval | Explicit real-world device and operating context |

A successful simulation test substantiates only its declared simulation profile. A verified optimization solver status is an input to the control verdict, not its substitute.

## 1. Source and implementation evidence ledger

Evidence labels: `DOC` official project documentation; `SOURCE` inspectable code/reference; `RESEARCH` conceptual technique; `LFR` existing design proposal; `PROPOSED` P06-D03 design awaiting implementation. Documentation and source inspection are separate from runtime qualification.

| Source and evidence | Concrete engineering lesson | P06-D03 design use | Qualification limit |
|---|---|---|---|
| [python-control overview](https://python-control.readthedocs.io/en/latest/) (`DOC/SOURCE`) | State-space, nonlinear I/O, LQR, H-infinity, estimation and stability analysis in one implementation ecosystem | Reference oracle/benchmark for linear controllers and estimators | Python implementation is an external comparison, not compiled Gerbil functionality |
| [python-control discrete LQR](https://python-control.readthedocs.io/en/stable/generated/control.dlqr.html) (`DOC`) | Riccati-based discrete optimal state feedback and closed-loop eigenvalues | Frozen LQR gain and independent Riccati/eigenvalue tests | Unconstrained quadratic design requires separate actuator/robustness checks |
| [python-control discrete LQE](https://python-control.readthedocs.io/en/latest/generated/control.dlqe.html) (`DOC`) | Gain-based estimator depends on dynamics, observation and noise covariance | Estimator model/covariance locks and residual metrics | Kalman optimality is tied to its assumptions and objective |
| [SciPy DARE](https://docs.scipy.org/doc/scipy/reference/generated/scipy.linalg.solve_discrete_are.html) (`DOC`) | Discrete Riccati solution requires spectral/controllability conditions | Structural preflight and independent matrix oracle | Numerical solver success is not a hardware safety claim |
| [OSQP MPC example](https://osqp.org/docs/examples/mpc.html) (`DOC/SOURCE`) | Receding horizon QP with state/input bounds and warm starts | Constrained linear MPC candidate and per-tick solver receipt | Warm start does not ensure feasibility or deadline success |
| [OSQP solver statuses](https://osqp.org/docs/interfaces/status_values.html) (`DOC`) | Distinct solved, inaccurate, infeasible, iteration/time-limit, nonconvex statuses | Typed solver status and conservative admission policy | `solved inaccurate` requires profile-specific residual acceptance |
| [acados features](https://docs.acados.org/features/) (`DOC/SOURCE`) | SQP-RTI, split preparation/feedback, timeouts and generated solvers | Optional nonlinear MPC backend and latency measurement | RTI iteration does not itself certify model invariance |
| [acados Python/C interfaces](https://docs.acados.org/python_interface/index.html), [C API](https://docs.acados.org/c_interface/) (`DOC`) | High-level problem specification can produce C solver artifacts; C ABI has explicit layout | Generated-code provenance and FFI interface qualification | Generated code and deployed target build need separate verification |
| [CasADi docs](https://web.casadi.org/docs/) (`DOC`) | Differentiation/optimization-building blocks, not an OCP solver by itself | Optional model derivatives; optimizer remains separately identified | Do not label CasADi alone as the solver |
| [do-mpc documentation](https://www.do-mpc.com/en/latest/) (`DOC/SOURCE`) | Nonlinear/robust MPC and MHE have separate estimation/optimization pipelines | Candidates for uncertain nonlinear plants and estimators | Nonlinear feasibility and compute budgets are per problem |
| [ros2_control controller manager](https://control.ros.org/master/doc/ros2_control/controller_manager/doc/userdoc.html) (`DOC/SOURCE`) | Real loop reads state, updates controller, writes commands; fallback controllers and overruns need lifecycle checks | Host boundary, rate/deadline, actuator interface and fallback tests | Framework fallback availability depends on actual hardware interfaces |
| [ros2_control clocks](https://github.com/ros-controls/ros2_control/blob/master/controller_manager/doc/clocks.rst) (`SOURCE`) | Hardware and controller clocks have distinct semantics | Monotonic deadlines versus simulation logical time | Timestamps alone do not prove real-time determinism |
| [NASA RTA verification framework](https://ntrs.nasa.gov/citations/20240007986) (`RESEARCH`) | Advanced/reversionary controller, safety monitor and switch assumptions are explicit | Hybrid supervisor with monitored reversion | Device-specific safety analysis and switch timing remain necessary |
| [Gerbil interfaces](https://gerbil.scheme.org/reference/std/interface.html), [test library](https://gerbil.scheme.org/reference/std/test.html) (`DOC`) | Checked method contracts and repeatable test primitives exist | Gerbil reference and typed boundary/test harness | `&` unchecked dispatch is a trusted internal detail |
| LFR [K0-D01](LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md), [K1-D01](LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md) (`LFR`) | Identity, effect, authority, evidence; optional time, numeric, constraint and simulation mechanisms | Reuse exact portable concepts by interface | Prior design documents have no automatic runtime qualification |
| LFR [P06-D01](LFR-P06-D01-reinforcement-learning-adaptive-intelligence-architecture-2026-11-13.md), [P06-D02](LFR-P06-D02-multi-agent-behavioral-systems-architecture-2026-11-13.md) (`LFR`) | Frozen policy ownership, train/eval separation, agent identity/role and joint action proposal | Candidate policy or coordinated setpoint adapter | Learning/coordination results are separate from plant admission |

**Inspection policy:** before implementation, pin exact library commits/tags, license, supported toolchain, numerical algorithm, ABI and benchmark fixture; record benchmark on the same plant/time/precision profile. External comparisons can be implemented in other languages without becoming dependencies of the Gerbil reference runtime.

## 2. Architectural decomposition

```text
        Application / control-task owner
             | Setpoint + task + approved bounds
             v
  PlantModel  ObservationPort ----> StateEstimator
       |                                 |
  Truth / exogenous input            Estimate / interval / freshness
       |                                 |
       +--------------+------------------+
                      v
             Controller candidates
         PID | state feedback | LQR | MPC | learned proposal
                      |
             CandidateControlEnvelope
                      v
             Supervisor / monitor
        constraint feasibility | timing | version
             |         |          |
           accept    revise     reject/fallback
                      |
             Host admission port
                      |
             SimulatedPlant executor
                OR qualified device host
                      |
              Effect/Step receipt
                      |
          Independent evaluator & evidence
```

The **plant model** specifies dynamics and admissible state; the **estimator** publishes an estimate with its uncertainty and freshness; a **controller** proposes a command; the **supervisor** checks the proposal under a named safety/operating profile; the **host** executes the admitted command in a simulation or a separately qualified device. An independent evaluator compares actual transitions with objectives and bounds.

**Ownership of the authoritative plant:** in pure simulation the fixture host owns the true state. With a device, the hardware/plant owner owns the physical state and effect interfaces; D03 processes observations and admitted commands. A state estimate, even with small covariance, never substitutes for authoritative feedback.

### 2.1 Time and authority domains

| Domain | Meaning | Representative error |
|---|---|---|
| `plant_logical_tick` | State-transition index for a model fixture | Missing/duplicated transition |
| `measurement_time` | Sampling timestamp in declared clock | Sensor delay, out-of-order packet |
| `controller_deadline` | Deadline by which a candidate is usable | Result arrives after tick admission |
| `host_write_receipt` | Accepted/issued/observed-effect status | Unknown command application |
| `experiment_time` | Reproducible simulated time base | Wrong step size or integrator |

**Effect meaning:** `PROPOSED`, `ADMITTED`, `ISSUED`, `ACKNOWLEDGED`, `OBSERVED_APPLIED` and `UNKNOWN` are distinct lifecycle states. An acknowledgment does not establish that a physical device reached the requested state.

### 2.2 K0/K1 and specialist ports

- K0: exact identity, revision, validation, effect and result envelopes.
- K1: optional M04 observation, M05 time, M08 objective/cost, M09 resource scope, M10 effects, M11 reconciliation, M12 snapshot, M13 interop, M14 experiments, M15 constraints/supervision, M16 provenance, M17 numeric, M18 model coupling.
- P06-D01: frozen policy proposal, model/policy identity, learner and independent training/evaluation lifecycle.
- P06-D02: agent/team proposals and role constraints, converted into a bounded setpoint or command intent.
- P06-D04: future transport and remote consistency; D03 retains tick/plant/command semantics.
- P07-D01: optional physical-model/simulation adapters; plant solver ownership remains explicit.
- P05: optional game/world host for simulated agency and consequences.

## 3. Plant, operating envelope and typed contracts

```text
PlantModelSpec {
  plant_id, model_revision, plant_family: DISCRETE | CONTINUOUS | HYBRID,
  state_schema_ref, input_schema_ref, output_schema_ref,
  f_or_A_B_ref, h_or_C_D_ref, parameter_set_ref,
  units_and_frame_ref, sample_or_step_profile_ref,
  uncertainty_set_ref, disturbance_profile_ref,
  operating_modes_ref, numeric_profile_ref, provenance_ref
}
OperatingEnvelope {
  plant_ref, state_domain_ref, input_domain_ref,
  rate_and_slew_limits_ref, disturbance_assumptions_ref,
  estimate_uncertainty_profile_ref, time_bounds_ref,
  invariant_set_ref_optional, host_interface_ref,
  emergency_policy_ref, revision
}
MeasurementEnvelope {
  plant_ref, sensor_ref, sample_seq, measurement_time,
  observation_value, units, covariance_or_interval_optional,
  calibration_revision, health_flags, clock_profile_ref
}
StateEstimate {
  estimate_ref, plant_ref, model_revision, value,
  covariance_or_reachable_interval, estimator_ref,
  estimator_revision, source_measurements[],
  time_base_ref, estimate_time, freshness_bound, status
}
ControlTask {
  task_ref, objective_ref, setpoint_or_trajectory_ref,
  performance_metrics[], constraint_profile_ref,
  permitted_controller_families[], sample_budget,
  compute_budget, admission_class: SIMULATION | DEVICE,
  authority_ref, evaluation_protocol_ref, revision
}
ControllerCandidate {
  task_ref, plant_ref, estimate_ref, candidate_ref,
  controller_family, controller_revision, command_value,
  command_units, intended_tick, valid_until, solver_receipt_optional,
  predicted_state_bounds_optional, provenance_ref, status
}
SupervisorDecision {
  candidate_ref, envelope_revision, monitor_revision,
  decision: ACCEPT | REVISE | FALLBACK | REJECT | INFEASIBLE,
  selected_command_optional, feasibility_witness_optional,
  fallback_controller_ref_optional, evidence_ref, deadline_status
}
CommandReceipt {
  command_ref, plant_ref, tick, selected_controller_ref,
  supervisor_decision_ref, host_authority_ref,
  admission_status, issue_status, effect_observation_status,
  applied_command_optional, timestamp_refs, error_or_unknown_ref
}
```

**Validation order:** schema and dimensions → units/coordinate frame → plant/controller revision → sample index and timestamp → numeric finiteness → constraint/model assumptions → controller capability → supervisor feasibility → effect-host authority. Each failed layer records a typed status. Solver output is interpreted after independent schema and residual checks.

**Model revisions:** plant parameters, sample time, uncertainty set, state transformation, estimator covariance and safety envelope all participate in an exact compatibility lock. Changing any of them triggers a new qualification scope. Controller gain reuse requires explicit revalidation.

### 3.1 Continuous, discrete and hybrid models

Discrete linear baseline:

```text
x[k+1] = A x[k] + B u[k] + E w[k]
y[k]   = C x[k] + D u[k] + v[k]
```

Continuous candidate:

```text
dx/dt = f_mode(x, u, w, p, t)
y     = h_mode(x, u, v, p, t)
```

A hybrid plant adds `mode`, invariant/guard predicates, reset maps and event priorities. Discretization records hold convention, integration algorithm, tolerance, event detection, mode change and numeric profile. The model owner defines state and signal units. Invariants over continuous intervals require continuous-time or validated reachability evidence when sampling could miss a crossing.

### 3.2 Numeric and uncertainty contract

Validate `NaN/Inf`, signed zero and overflow behavior where relevant, dimensions, symmetric covariance, positive semidefiniteness tolerance, sensor saturation, quantization, clipping, physical units and scalar/vector ordering. Choose interval, deterministic bounded disturbance, stochastic covariance or distributional ambiguity explicitly. A confidence interval and a deterministic reachability set license different claims.

## 4. State estimation and observation handling

**Estimator profiles:** direct measured state; linear observer; discrete Kalman estimator; extended/unscented candidate; bounded interval observer; moving-horizon estimation. Each is selected by observation model, uncertainty assumptions, compute budget and required output form. Observability/detectability checks precede linear-estimator claims. Causal timestamp alignment and delayed-data policy are part of the estimator, rather than hidden in a controller.

**Update phases:** predict state and uncertainty to the current tick; validate measurement timestamp/units/covariance; apply measurement update; publish state estimate and residual/inconsistency evidence; record whether the estimate was predicted without a fresh observation. Missing or conflicting observations may enlarge uncertainty or reject the control tick according to the profile.

**Separation:** `estimate.value`, `estimate.uncertainty`, `measurement`, `true_state_for_evaluator` are independent records. Training-only privileged state from P06-D01 and evaluation truth are outside deployed estimator inputs unless the specific device genuinely exposes them.

**Example:** for a sensor reading `y = x + v` with `|v| ≤ e`, an interval estimate `[y-e, y+e]` is conditionally sound if the sensor bound holds. Propagating it to the next tick must account for command and disturbance bounds; reusing the same interval after a tick without propagation is stale.

### 4.1 Diagnostics and failure treatment

- Residual and normalized innovation where covariance assumptions permit them.
- Detectability/observability verdict and model mismatch diagnostic.
- Timestamp monotonicity, stale horizon and clock skew budget.
- Sensor stuck-at, dropout, delayed/out-of-order, quantization and saturation fixtures.
- Divergence/uncertainty growth, measurement conflict and estimate reset lifecycle.

An estimate violating its declared validity conditions returns `ESTIMATE_UNUSABLE` or `UNCERTAINTY_EXCEEDED`. A controller may subsequently propose a command only under a supervisor profile that explicitly handles that status.

## 5. Controller and algorithm qualification by task

| Family | Task fit and admissible assumptions | Reference/implementation target | Essential tests |
|---|---|---|---|
| P / PI / PID (+ anti-windup) | Small SISO/multiloop tracking with measurable error and known sample timing | Pure Gerbil finite reference, independent numerical oracle | Step response, actuator saturation, integral windup, derivative filtering, disturbances |
| State feedback / pole placement | Observable or estimated state, realizable gain and model | Pure Gerbil small matrices; python-control comparison | Closed-loop poles, numeric conditioning, controllability, limits |
| LQR / LQI | Linear quadratic model, stabilizability and cost matrices; optional integral state | Computed gain via qualified external oracle; Gerbil frozen-gain runtime | Riccati residual, poles, disturbance metrics, saturation/mismatch |
| Linear MPC | Known discrete model, polyhedral constraints, QP deadline | OSQP candidate with exact status and residual checks | Constraint satisfaction, warm-start parity, infeasible/timeout decisions |
| Nonlinear / robust MPC | Nonlinear or uncertain plant, feasible solver and time budget | acados/do-mpc candidates, separately pinned solver | Model mismatch, robust constraints, solver status, deadline tails |
| Barrier / viability filter | Verified admissible set under declared dynamics and uncertainty | Pure analytic scalar filter first; specialized QP later | Boundary, disturbance worst case, empty feasible set, reachable-set witness |
| Hybrid supervisor | Multiple controllers/modes, monitor and switching conditions | Gerbil finite-state monitor with exact trace | Guard precedence, fallback feasibility, hysteresis, dwell, urgent reversion |
| Learned policy | Frozen P06-D01 artifact with observation/control schema parity | Optional policy inference via D01 | Out-of-support state, timing, invalid output, adversarial model shift |

**Selection rule:** choose the smallest algorithm family that meets the *named control task* under the required uncertainty, constraint and runtime profile. A method failing the actual task is rejected for that profile. A valid task-specific controller stays qualified even if it lacks properties irrelevant to that task.

### 5.1 Baselines and fair comparison

Every tour declares the plant/model lock, initial-condition distribution or grid, disturbance range, sensor pattern, horizon, sample period, command limits, objective and evaluation budget. Compare no-control, saturated proportional control, a tuned PI/PID or frozen-gain state feedback, then add MPC/learned alternatives if the task calls for them. Report tracking RMSE/IAE, overshoot, settling time, saturation time, infeasible ticks, safety-bound breaches, CPU/solver time distributions, memory and switches. Cost optimization is measured separately from constraint satisfaction.

**Algorithm source rule:** actual implementation source and solver configuration are essential evidence when proposing a port. An academic formulation can specify expected math; it does not verify a library's behavior on the target plant. The comparison must include serialization, FFI and scheduling overhead for B/C backends.

### 5.2 Stability and control claims

Stability claims specify equilibrium/reference, norm, perturbation/uncertainty domain, sampling, invariant neighborhood and proof or empirical horizon. For discrete linear feedback `u = -Kx`, the closed-loop spectral radius criterion `rho(A-BK) < 1` supports local/global linear asymptotic stability *for the stated linear time-invariant unconstrained model*. Saturation, delays, switching and nonlinear disturbance require separate analysis. A finite simulation with no observed failure is an empirical bound, not a universal stability proof.

## 6. Supervisory control, operating modes and switching

**Supervisor input:** current mode, latest estimate/reachable set, frozen operating envelope, controller candidate(s), solver status, clock/deadline evidence, host state and one-shot command admission token. **Supervisor output:** selected command or typed refusal with decision evidence and a replayable switch reason.

Operational states:

```text
UNCONFIGURED -> READY -> NOMINAL -> REVERSION -> RECOVERY -> READY
                    |         |         |
                    v         v         v
                 DEGRADED  REFUSED   MAINTENANCE
```

- `READY`: calibrated model/estimator, supervisor, command host and reference controller version locks match.
- `NOMINAL`: qualified candidate admissible at current tick and envelope.
- `DEGRADED`: observation uncertainty, timing or model health reduced but a named degraded controller profile remains feasible.
- `REVERSION`: supervisor selects a qualified fallback whose feasibility witness applies to the current reachable set.
- `REFUSED`: missing authority, infeasible command set, incompatible model or unusable measurement.
- `RECOVERY`: explicitly evaluated re-entry conditions and hysteresis/dwell policy.

**Switch semantics:** check deadline and safety preconditions for each candidate; choose according to deterministic priority; issue a single command token per plant tick; record the selected controller and rejected alternatives. Hysteresis and dwell reduce chattering during ordinary switching, but urgent protective reversion cannot be delayed by a dwell rule. Reset integral and observer/controller internal states via versioned transfer functions when selected.

**Fallback feasibility:** a fallback controller is a separately specified control policy with invariant/reachability evidence under current state, disturbance and actuator limits. A default zero command is safe only if the relevant admissibility witness covers it. When no feasible input exists under the assumptions, report `INFEASIBLE` and escalate to the declared emergency host procedure.

### 6.1 Formal and proof-carrying extensions

A supervisor may receive a certificate containing the model/envelope hash, controller version, eligible operating region, assumed observation uncertainty, timing bound, validity interval and verification procedure. The host checks binding and freshness at admission. A certificate from a model checker, barrier verifier or offline reachability analysis has an explicit scope; the design does not infer a proof for all states from a proof about one region.

## 7. Exact first fixture: BoundedAxis

Consider a scalar sampled integrator, with position `x`, control `u`, bounded disturbance `w`, sample interval `dt > 0`, state limit `X > 0`, control limit `U > 0` and disturbance bound `W >= 0`:

```text
x[k+1] = x[k] + dt * (u[k] + w[k])
|x[k]| <= X       |u[k]| <= U       |w[k]| <= W
observed state interval: x[k] in [x_lo, x_hi]
```

For **one next transition** to remain in `[-X, X]` for *every* state in `[x_lo,x_hi]` and *every* disturbance in `[-W,W]`, valid controls satisfy:

```text
u >= (-X - x_lo)/dt + W
u <= ( X - x_hi)/dt - W
u in [-U, U]

lower = max(-U, (-X - x_lo)/dt + W)
upper = min( U, ( X - x_hi)/dt - W)
AdmissibleSet = [lower, upper] if lower <= upper
              = EMPTY otherwise
```

This inequality is a **necessary and sufficient one-step feasibility oracle** for the stated model, interval and symmetric bounds, assuming exact arithmetic. It is an excellent first Gerbil regression oracle; finite precision requires outward rounding or bounded numerical tolerance in the implementation. Repeating one-step admissibility can enforce interval containment **only while each tick begins inside the invariant set and the stated bounds hold**. Device timing, inter-sample behavior and plant mismatch need separate qualification.

**Executable reference contract:** given a model lock and observation interval, return the admissible set; clamp/replace a proposed controller output only when a selected supervisor rule authorizes revision; return an exact witness with the two inequalities; retain an `EMPTY_SET` outcome on infeasibility.

**Example with nonempty set:** `X=1, U=0.4, W=0.1, dt=0.2, [x_lo,x_hi]=[0.80,0.85]`. Lower bound from left safety is `-8.9`, upper is `0.65`; intersection is `[-0.4,0.4]`. A candidate `u=0.3` is admissible for one step. It is not guaranteed to achieve a tracking objective.

**Example with robust infeasibility:** `X=1, U=0.2, W=0.25, dt=1, [x_lo,x_hi]=[-1,1]` gives `lower=+0.25` and `upper=-0.25`, so `EMPTY_SET`. Setting `u=0` would violate the robust one-step guarantee near a boundary. **No safe command can be certified under these assumptions.**

### 7.1 Secondary fixture: DoubleIntegrator

```text
position_next = position + dt*velocity + 0.5*dt^2*(u+w)
velocity_next = velocity + dt*(u+w)
```

Add velocity bounds, position corridors, acceleration/slew limits, switching delays, stopping distance and sampled versus inter-sample checks. Analytic zero-order-hold transitions allow an independent oracle for constant acceleration; a braking-distance envelope depends on speed, available deceleration and disturbance. A scalar position clip from BoundedAxis cannot substitute for these constraints.

### 7.2 Hybrid fixture: ModeSwitch

Modes `NORMAL`, `DEGRADED`, `REVERSION`; event-driven sensor dropout and actuator limit changes; named guard priority, clock and reset maps. Independent oracle enumerates finite mode transitions. Inject switch latency and check whether the reversionary policy remains feasible when actual application is delayed.

## 8. Timing, scheduling and device-host integration

**Sample contract:** each read, estimate, compute, supervise and issue phase has a timestamp, budget and clock reference. A command computed for tick `k` cannot be reused at tick `k+1` without a declared hold policy. Include sensor sampling jitter, communication delay, compute tail latency, write acknowledgment and actuator response when evaluating `DEVICE_CONTROL`.

**Real-time evidence:** measure deadline miss rate and worst observed latency under specified host CPU load, scheduler policy, memory allocation, I/O faults and thermal conditions. Benchmarks on a general-purpose laptop qualify only that benchmark environment. A hardware-target release requires platform-specific tests and safety analysis.

**Controller ownership:** a single authoritative host arbiter grants a command interface to an active controller for a plant revision and tick. Chained commands (e.g., velocity setpoint -> torque) are typed stages with distinct ownership. Hardware read/write errors, stale claims and switch interruptions produce visible faults. The device may continue its previous command during a fault; the host's actual fail-safe behavior must be recorded and analyzed.

### 8.1 Backend profiles

| Profile | Ownership and computation | Fit | Required evidence |
|---|---|---|---|
| **A — Gerbil reference** | Gerbil owns plant/estimator/controller/supervisor semantics and finite numerical reference kernels | Small sampled plants, PID, scalar observers, finite hybrid modes, oracle fixtures | Pinned toolchain, numeric parity, reproducible fixtures, measured timing |
| **B — Gerbil + qualified native/solver** | Gerbil validates and dispatches; chosen native engine solves heavy optimization/estimation or hardware-timed kernels | LQR/QP/MPC/NMPC, intensive MHE, embedded kernels | ABI, provenance/license, solver status, worst-case timing, boundary copies, fallback behavior |
| **C — Gerbil + external control service** | Gerbil manages typed request/admission/evidence; remote worker handles separately qualified compute | Offline planning, optimization batches, slow supervisory planning | Request/response revision, deadline, authentication, retry ambiguity, service failure reconciliation |

The backend is selected by plant complexity and latency budget. **A remote computation service entering a hard real-time device control loop requires its own end-to-end timing and failover qualification.**

### 8.2 Proposed Gerbil namespace

```text
lfr/p06/control/
  contract/{plant,signal,units,time,observation,estimate,task,command,receipt}.ss
  model/{discrete,linear,hybrid,uncertainty}.ss
  estimation/{direct,interval,observer,kalman-adapter,diagnostics}.ss
  controller/{p,pid,state-feedback,lqr-gain,mpc-adapter,policy-adapter}.ss
  supervision/{envelope,feasibility,modes,switch,watchdog,admission}.ss
  experiment/{fixtures,oracles,metrics,trace}.ss
  adapters/{d01-policy,d02-proposals,p10-simulation,device,osqp,acados}.ss
  qualification/{register,evidence}.ss
```

These module partitions provide an implementation map. Public host-facing APIs validate inputs; optimized unchecked interfaces remain inside individually qualified internal modules.

### 8.3 Reference API sketch

```text
register_plant(spec) -> PlantHandle | ValidationError
open_fixture(plant, seed, conditions) -> FixtureHandle | Error
observe(handle, tick) -> MeasurementEnvelope | Error
estimate(estimator, measurements, prior) -> StateEstimate | Error
propose(controller, estimate, task) -> ControllerCandidate | Error
supervise(supervisor, candidate_set, envelope) -> SupervisorDecision
execute_fixture(handle, decision, expected_tick) -> CommandReceipt | Error
reconcile_effect(host, command_ref) -> Applied | Rejected | Unknown
score_run(trace, oracle, metric_profile) -> EvaluationReport
```

The API sketch describes data/control flow; exact Gerbil syntax, exported methods and numerical implementation are verified against a pinned toolchain during implementation.

## 9. Interoperability and autonomy composition

**P06-D01 policy:** adapter converts observation/estimate to the frozen policy input and returns a candidate action with policy version, action units, confidence/support metadata where available and inference deadline. D03 owns control admission and plant interpretation. Training data remain within D01's lifecycle.

**P06-D02 agents:** adapter maps role-approved tasks and joint proposals to a plant target/setpoint or bounded command intent. Multi-agent agreement is advisory to the control host; competing setpoints are arbitrated by the application and supervisor profile. An agent's task success is not equivalent to actuator completion.

**P06-D04 distribution:** design a typed gateway for remote controller computation, telemetry and reconciliation. D03 specifies command timing and feasibility; D04 later supplies message delivery and replicated/remote execution semantics. Network acknowledgment and physical effect observation remain distinguishable.

**P07-D01 simulation:** general physical/co-simulation engines supply model steps and events under a versioned adapter; D03 owns controller/evaluation contracts. Coupled models declare who advances time, handles algebraic loops and detects continuous events. FMI-like adapters are separate candidates and carry model/solver export provenance.

**P05 worlds:** a control output may be proposed as a P05 intervention or simulated actuator; P05 applies its own world rules and causal admission. A player/NPC decision does not directly alter P05 canonical state through a D03 control proposal.

**P05-D00 formal computing:** optional proof artifacts can carry invariant region/automaton/certificate evidence under exact binding to the plant and monitor profile. Runtime monitor validation and formal proof validity remain separately qualified.

## 10. Evaluation harness and experiment governance

| Evaluation axis | Primary measurements | Failure evidence |
|---|---|---|
| Task performance | IAE/ISE/RMSE, settling, overshoot, reference-tracking error | Divergence, objective miss, excessive control effort |
| Constraints | State/input/rate breaches, worst-case reachable bounds | Infeasible set, saturation, invalid shield |
| Estimation | Residuals, covariance calibration, interval coverage, freshness | Divergence, dropouts, out-of-order, covariance mismatch |
| Stability | Poles/Lyapunov/reachability witness or scoped finite-horizon metric | Unstable mode, chattering, incorrect switched stability claim |
| Timing | tick latency p50/p95/p99/max observed, deadline misses, jitter | Solver timeout, stale command, missed watchdog |
| Robustness | disturbance grid, parameter drifts, sensor/actuator faults | Model mismatch, sensitivity, loss of invariant set |
| Provenance | plant/solver/revision hashes, input histories, seeds, operator permit | Missing source lock, altered certificate, untraceable command |
| Cost | CPU/memory, kernel and end-to-end time, remote calls | Performance regression and budget overrun |

**Evidence partitions:** design/source review → model oracle → compiled reference → synthetic closed-loop task → solver parity → timing profile → hardware-in-loop/device evidence. A test in a later partition never retroactively makes an earlier assumption true. Report `PASS`, `FAIL`, `INFEASIBLE`, `UNRESOLVED`, `OUT_OF_SCOPE` or `SPEC_BLOCKED`, with details and deterministic rerun instructions where possible.

**Independent harness:** freeze plant truth/initial conditions and compare controller outputs on the same disturbances. Do not use an agent's internal reward, solver objective or controller's predicted trajectory as the sole performance oracle. Prefer exact formulas for simple plants and independent reference solvers/implementations for advanced control.

## 11. Invariants and failure taxonomy

### 11.1 Normative invariants I01–I30

1. **I01 — Plant identity:** every command binds to one plant ID and revision.
2. **I02 — Unit safety:** input/output and setpoint units match their declared conversions.
3. **I03 — Numeric validity:** invalid, nonfinite or malformed numeric input fails before actuation.
4. **I04 — Clock identity:** logical ticks and host deadlines carry explicit clocks.
5. **I05 — Freshness:** stale estimates cannot masquerade as current measurements.
6. **I06 — Truth isolation:** estimator outputs and evaluator-only plant truth remain distinct.
7. **I07 — Disturbance scope:** robust claims carry a bounded disturbance set.
8. **I08 — Estimation evidence:** covariance or intervals bind to their assumptions.
9. **I09 — Controller identity:** gains, algorithm, revision and numeric profile are fixed per candidate.
10. **I10 — Candidate nature:** proposed input is separately admitted by a supervisor and host.
11. **I11 — Input bounds:** admitted command satisfies current physical/input constraints.
12. **I12 — Feasibility:** empty admissible sets produce `INFEASIBLE` and a trace.
13. **I13 — One-step witness:** scalar robust admissibility satisfies both bound inequalities.
14. **I14 — Fallback preconditions:** fallback feasibility applies to current reachable state.
15. **I15 — Switching trace:** mode changes carry guard, priority and timing evidence.
16. **I16 — No delayed protection:** dwell/hysteresis cannot postpone urgent protective action.
17. **I17 — Single admission:** one authoritative command token applies per plant/tick profile.
18. **I18 — Effect honesty:** issue, acknowledgment and observed effect are distinct.
19. **I19 — Deadline admission:** a late candidate follows the declared reject/hold policy.
20. **I20 — Command continuity:** hold-last-command has explicit expiration and fault semantics.
21. **I21 — Algorithm-specific proof:** stability statements name dynamics and assumptions.
22. **I22 — Solver interpretation:** solver exit status, residual and feasibility are separately recorded.
23. **I23 — Controller state transfer:** integral/observer state reset at switching is versioned.
24. **I24 — Update separation:** learner updates do not alter a frozen deployed controller mid-tick.
25. **I25 — External authority:** device effects require an independently authorized host.
26. **I26 — Replay scope:** deterministic simulation replay and physical effect history remain distinct.
27. **I27 — Resource limits:** budget exhaustion yields explicit bounded outcome.
28. **I28 — Cross-domain adapters:** P05/P06/P07 ownership is preserved at interface boundaries.
29. **I29 — Evidence integrity:** evaluation records include exact model, code and data locks.
30. **I30 — Qualification scope:** passed claims name plant, controller, envelope and platform.

### 11.2 Typed outcomes

`VALIDATION_ERROR`, `UNIT_MISMATCH`, `MODEL_REVISION_STALE`, `ESTIMATE_UNUSABLE`, `UNCERTAINTY_EXCEEDED`, `CONTROL_CANDIDATE_INVALID`, `INPUT_LIMIT_BREACH`, `INFEASIBLE`, `SOLVER_INACCURATE`, `SOLVER_TIMEOUT`, `DEADLINE_MISSED`, `SUPERVISOR_REJECT`, `FALLBACK_UNAVAILABLE`, `SWITCH_GUARD_CONFLICT`, `UNKNOWN_COMMAND_EFFECT`, `HOST_AUTHORITY_DENIED`, `NUMERIC_DIVERGENCE`, `RESOURCE_EXHAUSTED`, `EVIDENCE_INCOMPLETE`, `PROFILE_UNQUALIFIED`.

All outcomes retain model/plant/tick/controller refs, evidence severity and the responsible boundary where available. Unknown device outcomes are reconciled with the effect host; automated replay of a physical command is subject to the host's explicit idempotency and safety semantics.

## 12. Qualification gates and test matrix

### 12.1 Qualification gates Q0–Q11

| Gate | Qualification question | Minimum evidence | Claim consequence |
|---|---|---|---|
| Q0 — Source/toolchain | Are model/source revisions and Gerbil/numeric dependencies pinned? | Source locks, license, exact toolchain, syntax smoke test | Enables implementation evidence |
| Q1 — Schema and units | Do plant/command/measurement contracts reject invalid inputs? | Dimensional, revision and numeric negative tests | Validated interface profile |
| Q2 — Exact plant semantics | Does reference dynamics match analytic oracles? | BoundedAxis and DoubleIntegrator parity, seed locks | Qualified finite plant model |
| Q3 — Estimation | Are observation uncertainty/freshness assumptions respected? | Interval/Kalman/observer negative cases and oracle | Qualified selected estimate profile |
| Q4 — Controller baseline | Does PID/state feedback meet declared task metrics? | Independent closed-loop task tours, baseline comparisons | Named baseline controller profile |
| Q5 — Optimization | Do MPC/solver statuses, residuals and deadlines bind correctly? | QP/NMPC adversarial cases for selected solver | Optional optimized controller profile |
| Q6 — Feasibility | Does supervisor enforce admissible sets and detect emptiness? | Analytic one-step robust oracle and failure fixtures | Qualified finite feasibility monitor |
| Q7 — Hybrid switching | Are guards, reversion and state transfers valid? | Finite mode oracle, dropout and switch-latency tests | Qualified hybrid supervisor profile |
| Q8 — Timing and faults | Is the host's measured tick/deadline behavior within profile? | Latency distributions, device/mock faults, watchdog | Named runtime host timing profile |
| Q9 — External boundaries | Do optional policy, agent, solver and device adapters preserve contracts? | Version/authority/cancellation/unknown-effect tests | Named adapter profile only |
| Q10 — Independent evaluation | Are results reproducible and claims scoped? | Held-out seeds, immutable metrics, provenance and negative control | Evidence-scoped task verdict |
| Q11 — Release and audit | Are every applicable gate and risk decision accepted by owners? | Frozen manifest, audit report, open-failure disposition | Profile-specific release decision |

A release declares exactly which gates apply to its profile. The first pure finite-plant release can be qualified under Q0–Q4, Q6–Q7 and Q10–Q11 when those gates' relevant checks pass; Q5/Q8/Q9 remain applicable only as the corresponding advanced/host integrations are included. Each named profile must pass all gates required by its claim.

### 12.2 Proposed 80-case test register


#### T01–T10: Contract and identity

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T01** | Plant ID revision lock | Reject candidate referencing another plant revision |
| **P06-D03-T02** | Command dimension | Reject scalar command into vector-only actuator profile |
| **P06-D03-T03** | Units mismatch | Reject degrees versus radians without declared conversion |
| **P06-D03-T04** | Missing numeric value | Reject null/NaN/Inf before compute/admission |
| **P06-D03-T05** | Discrete sample duration | Reject zero and negative dt; permit named valid dt |
| **P06-D03-T06** | Clock-domain mismatch | Reject wall time used as logical tick without adapter |
| **P06-D03-T07** | Operating envelope revision | Reject stale invariant-set hash |
| **P06-D03-T08** | Actuator slew schema | Enforce previous-command reference and slew constraint |
| **P06-D03-T09** | Covariance matrix shape | Reject nonsymmetric/invalid dimension matrices |
| **P06-D03-T10** | Double command claim | Only one host command token wins per same plant/tick |

#### T11–T20: Plant dynamics and oracles

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T11** | Scalar zero input | Match analytic scalar transition for exact w=0 |
| **P06-D03-T12** | Scalar disturbance endpoints | Match both +/-W oracle transitions |
| **P06-D03-T13** | Double-integrator zero-order hold | Match exact p/v transition with constant acceleration |
| **P06-D03-T14** | State bound at endpoint | Include equality endpoints under declared numeric tolerance |
| **P06-D03-T15** | Parameter revision changed | Invalidate controller claim after A/B matrix change |
| **P06-D03-T16** | Hybrid guard order | Use declared priority when two guards enabled |
| **P06-D03-T17** | Hybrid reset map | Verify state reset at a mode transition |
| **P06-D03-T18** | Numeric precision profile | Quantify float-versus-reference discrepancy |
| **P06-D03-T19** | Simulation time drift | Reject wrong dt sequence in replay |
| **P06-D03-T20** | Inter-sample crossing | Flag sampled endpoint passing despite continuous-bound breach |

#### T21–T30: Observation and estimation

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T21** | Direct observation match | Estimate equals measured state when direct profile selected |
| **P06-D03-T22** | Interval consistency | Retain truth inside declared bounded-noise interval |
| **P06-D03-T23** | Prediction-only uncertainty | Widen bounds under missing observation |
| **P06-D03-T24** | Out-of-order measurement | Reject/reconcile by explicit timestamp policy |
| **P06-D03-T25** | Delayed sensor window | Mark estimate stale at expired freshness bound |
| **P06-D03-T26** | Sensor stuck-at | Raise health diagnostic on configured stuck-at pattern |
| **P06-D03-T27** | Kalman covariance PSD | Check covariance symmetry/PSD within tolerance |
| **P06-D03-T28** | Unobservable linear mode | Block unsupported convergence claim |
| **P06-D03-T29** | Estimate/evaluator truth isolation | Deny truth-only fields to controller |
| **P06-D03-T30** | Reset estimator state | Version and trace reset after model or sensor change |

#### T31–T40: Controller performance

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T31** | Proportional baseline | Meet error target on small stable reference task |
| **P06-D03-T32** | Integral antiwindup | Bound integrator under saturation |
| **P06-D03-T33** | Derivative sampling | Bound noise amplification under selected filter |
| **P06-D03-T34** | Step tracking | Record overshoot/settling on frozen step test |
| **P06-D03-T35** | Disturbance rejection | Compare IAE against no-control and P baselines |
| **P06-D03-T36** | LQR gain parity | Compare gain and poles with independent DARE oracle |
| **P06-D03-T37** | Unstable gain | Reject stability label when closed-loop radius >=1 |
| **P06-D03-T38** | Saturation-induced shift | Reassess stability/performance with actuator clipping |
| **P06-D03-T39** | Gain revision replacement | Reject unversioned hot swap mid-tick |
| **P06-D03-T40** | Task-specific method comparison | Preserve a simple controller when it meets declared task |

#### T41–T50: Optimizer and backend

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T41** | OSQP solved | Check residual/constraint witness before admission |
| **P06-D03-T42** | OSQP solved inaccurate | Apply explicit residual threshold or reject |
| **P06-D03-T43** | Primal infeasible status | Emit infeasible control result and fallback trace |
| **P06-D03-T44** | Dual infeasible status | Handle separately from primal infeasibility |
| **P06-D03-T45** | Iteration budget | Reject over-budget candidate before plant tick |
| **P06-D03-T46** | MPC warm-start parity | Compare cold/warm feasible results within tolerance |
| **P06-D03-T47** | Model mismatch in MPC | Report predicted/actual deviation and breaches |
| **P06-D03-T48** | Generated solver provenance | Pin C solver source and ABI/precision profile |
| **P06-D03-T49** | Native buffer lifetime | Reject stale/aliased FFI output buffer |
| **P06-D03-T50** | Remote response lateness | Reject deadline-expired command with audit record |

#### T51–T60: Supervisor and feasibility

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T51** | Scalar robust lower bound | Match analytic robust interval lower endpoint |
| **P06-D03-T52** | Scalar robust upper bound | Match analytic robust interval upper endpoint |
| **P06-D03-T53** | Empty admissible set | Return INFEASIBLE for widened interval example |
| **P06-D03-T54** | At-boundary controller | Accept equality-control only under valid tolerance |
| **P06-D03-T55** | Fallback zero counterexample | Refuse zero command lacking robust feasibility witness |
| **P06-D03-T56** | Proposed command revision | Clamp/replace only when selected rule permits |
| **P06-D03-T57** | Dwell urgent reversion | Urgent protection overrides ordinary dwell |
| **P06-D03-T58** | Switch controller state | Transfer/reset integrator state under versioned map |
| **P06-D03-T59** | Conflicting guard priority | Match deterministic priority in hybrid oracle |
| **P06-D03-T60** | Fallback missing | Report FALLBACK_UNAVAILABLE and emergency host path |

#### T61–T70: Runtime and host integration

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T61** | Tick deadline | Reject candidate computed after admissible tick deadline |
| **P06-D03-T62** | Late sample | Use defined rejection/prediction policy for late observation |
| **P06-D03-T63** | Hold-last-command expiry | Stop hold on expiry and issue typed response |
| **P06-D03-T64** | Host admission denied | Record denial without issuing effect |
| **P06-D03-T65** | Issued not observed | Keep effect unconfirmed when only write acknowledged |
| **P06-D03-T66** | Unknown effect on timeout | Return UNKNOWN and reconcile before any retry |
| **P06-D03-T67** | Host command interface conflict | Detect two active controllers claiming one interface |
| **P06-D03-T68** | Hardware read error | Activate configured fault path; preserve actual-state uncertainty |
| **P06-D03-T69** | Jitter under load | Measure p99/max/deadline misses under frozen load |
| **P06-D03-T70** | Device simulation separation | Reject device claim based solely on simulated timing |

#### T71–T80: Cross-domain and evidence

| ID | Test | Required oracle or failure evidence |
|---|---|---|
| **P06-D03-T71** | Frozen RL policy adapter | D01 model/policy revision follows control candidate |
| **P06-D03-T72** | Learner mid-tick update | Reject policy revision change during command admission |
| **P06-D03-T73** | Multi-agent joint proposal | D02 consensus maps into one explicit control intent |
| **P06-D03-T74** | Agent authority mismatch | Reject action intent lacking host control scope |
| **P06-D03-T75** | P05 world intervention adapter | World host retains rule/transition admission |
| **P06-D03-T76** | P07 simulation clock owner | Prevent double plant-time advance on coupling |
| **P06-D03-T77** | Held-out disturbance seeds | Compare controller/task metrics on frozen held-out set |
| **P06-D03-T78** | Trace replay manifest | Replay exact finite-plant trace from hashes and inputs |
| **P06-D03-T79** | Incomplete evidence disposition | Report UNRESOLVED rather than PASS for missing oracle |
| **P06-D03-T80** | Profile-scoped report | Separate model, control, supervisor and device claim levels |

### 12.3 Claim-to-gate matrix

| Claim profile | Mandatory gate set | Optional gate expansion | Allowed disposition |
|---|---|---|---|
| `P09_D03_FINITE_PLANT_QUALIFIED` | Q0/Q1/Q2/Q10/Q11 | Q3–Q4/Q6–Q7 when included | Finite numerical/model semantics |
| `P09_D03_REFERENCE_CONTROL_QUALIFIED` | Q0/Q1/Q2/Q3/Q4/Q6/Q10/Q11 | Q7/Q8 if included | Named closed-loop simulation task |
| `P09_D03_SUPERVISOR_QUALIFIED` | Q0/Q1/Q2/Q3/Q4/Q6/Q7/Q10/Q11 | Q8/Q9 if included | Named simulation/host monitor profile |
| `P09_D03_OPTIMIZED_MPC_QUALIFIED` | Q0/Q1/Q2/Q3/Q4/Q5/Q6/Q10/Q11 | Q7/Q8/Q9 by interface | Named solver, plant, timing profile |
| `P09_D03_DEVICE_ADMISSIBLE` | Q0/Q1/Q2/Q3/Q4/Q6/Q7/Q8/Q9/Q10/Q11 + selected-algorithm gates | Q5 when optimizer is selected; external assurance is mandatory | Named device/operational envelope only |

Each claim is assembled from actual executed evidence; the presence of the proposed 80-case register does not imply a test has run. Every mismatch preserves raw traces, source versions and the responsible assumption.

## 13. Implementation sequencing post separate authorization

1. **D03-00 — Evidence/contract pinning:** inspect source tags and actual Gerbil/Gambit API; fix the numeric profile, plant schema and oracle fixtures; record initial permit boundaries.
2. **D03-01 — BoundedAxis model:** Gerbil sampled transition, measurement interval, actuator bounds and analytic robust feasible-set oracle; establish Q0/Q1/Q2.
3. **D03-02 — Minimal controller tour:** proportional and PI/PID with saturation/anti-windup, frozen task baselines and exact sampling; establish first Q4 evidence.
4. **D03-03 — Estimator/monitor:** interval estimator, sensor freshness, supervisor admission and infeasible fallback; exercise Q3/Q6.
5. **D03-04 — Hybrid reversion:** fixed small mode machine, priority, switch latency, hysteresis and bounded recovery; establish Q7.
6. **D03-05 — DoubleIntegrator:** zero-order-hold oracle, velocity/acceleration bounds and braking feasibility with inter-sample audit.
7. **D03-06 — Qualified optimization candidates:** choose LQR/QP-MPC/NMPC only where the frozen task requires them; inspect and pin implementation source; complete Q5 and fair latency budgets.
8. **D03-07 — Controlled integrations:** synthetic P06-D01 learner-policy and P06-D02 setpoint adapters, optional P07 simulator; check contract and truth boundaries with mock hosts.
9. **D03-08 — Real-time/device research branch:** analyze actual device host API and hazard profile; add dedicated hardware-in-loop tests and operator authorization if separately requested.
10. **D03-09 — Independent qualification:** run the applicable 80-case subset and gates; archive traces/negative results; release only evidence-scoped profiles.

**Optimization rule:** a compiled Gerbil baseline that meets task and latency constraints is a useful result. Native or external complexity is introduced by measured limitations or required capabilities, then retained only after correctness, reliability and end-to-end performance qualification.

## 14. Open issues, design decisions and final verdict

| Open decision | Initial baseline | Promotion evidence |
|---|---|---|
| First plant | BoundedAxis sampled integrator | Exact interval/worst-disturbance oracle |
| First controller | P, PI/PID with saturation | Closed-loop task and anti-windup tests |
| Estimation | Direct measurement and bounded interval | Coverage, freshness and divergence diagnostics |
| Initial supervisor | Analytic robust feasibility and finite mode reversion | Empty-set, urgent-switch and fallback test |
| Linear optimal control | Frozen-gain LQR through inspected comparator | Riccati/pole parity and task gain |
| Constrained optimization | OSQP-style linear MPC when required | Solver status/residual/latency and input bounds |
| Nonlinear control | acados/do-mpc candidates | Model-specific numeric and timing qualification |
| Continuous safety | Profile-specific reachability/validated integration | Inter-sample bound and solver error evidence |
| Learned controller | P06-D01 frozen policy adapter | Exact input/output, support and supervisor tests |
| Device support | Separate host-admissible design profile | Hardware target, hazard analysis, timing and permission |
| Distributed control | P06-D04 port when separately specified | Network, consistency and failure reconciliation |
| Scientific simulator | P07-D01 port when separately specified | Clock ownership and model/coupling evidence |

**Design maturity:** source-grounded, mathematically specified finite reference with proposed oracles, control/estimation/supervision contracts, 80 tests and 12 qualification gates. No Gerbil or native control implementation, benchmark pass, safety proof, hardware test or deployment authorization has been asserted.

**Final design disposition:**

```text
P09_D03_CONTROL_AUTONOMOUS_SYSTEMS_DESIGN_PROPOSED /
GERBIL_REFERENCE_FIRST / PLANT_ESTIMATE_CONTROLLER_SUPERVISOR_HOST_SEPARATE /
TIME_UNITS_VERSION_LOCKS / ROBUST_ONE_STEP_FEASIBILITY_ORACLE /
PID_LQR_MPC_HYBRID_CANDIDATE_PROFILES /
P09_D01_LEARNER_ADAPTER_OPTIONAL / P09_D02_MULTI_AGENT_ADAPTER_OPTIONAL /
P09_D04_DISTRIBUTED_PORT_OPTIONAL / P10_SIMULATION_PORT_OPTIONAL /
80_FUTURE_TESTS / Q0_TO_Q11 / NO_RUNTIME_OR_DEVICE_EVIDENCE /
NO_BENCHMARK_PASS / UNIMPLEMENTED / DOCUMENT_PUBLISHED
```

> **Closing principle:** A controller earns a performance claim from an observed closed loop, a supervisor earns an admissibility claim from a verified envelope, and a device command earns execution authority from its host. Their evidence meets at a precise, versioned boundary; none inherits the other's proof by association.