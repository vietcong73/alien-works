# LFR-P10-D01 | Scientific & Physical Simulation Architecture

**Document class:** Source-grounded specialist detailed-design proposal  
**Date:** 14 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `SCIENTIFIC SIMULATION` — Scientific Simulation & Experimental Systems  
**Edition:** P10 — Scientific Simulation & Experimental Systems  
**Design slice:** D01 — Scientific & Physical Simulation Architecture  
**Primary design language:** Gerbil Scheme for model contracts, reference solvers, time and event semantics, numeric oracles, and orchestration; qualified native/FMI/service adapters by workload  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P10-D01-scientific-physical-simulation-architecture-2026-11-14.md`  
**Status:** `SOURCE_GROUNDED_DESIGN_PROPOSED / 80_FUTURE_TESTS / 12_FUTURE_GATES / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. Implementation, native library installation, external execution and deployment follow separate authorization.

> **Design decision:** P10-D01 provides versioned scientific models, state and field representations, numerical integration, event handling, physical invariants, coupling and error evidence. A compact Gerbil reference is the executable semantic baseline proposed for future implementation. Scalable solvers and specialized simulators connect through independently qualified ports. Simulation results are claims about a declared model, discretization, numerical method and parameter envelope.

## 0. Objectives, first useful capability and boundaries

**Research question:** Can Gerbil coordinate physically meaningful ODE, discrete, hybrid and spatial simulation jobs while retaining analytic or independent numerical evidence for correctness, and select specialized solvers by the actual problem rather than by framework preference?

**First useful product:** `OscillatorLab`, a bounded damped-oscillator reference with analytic position and velocity, kinetic/potential energy accounting, explicit Euler, symplectic Euler, velocity-Verlet for the undamped profile, and RK4. Each run returns a reproducible trajectory, model/solver lock, time-grid description, error vector, invariant measurements and budget receipt.

**Next reference fixtures:** `PeriodicHeat1D` demonstrates conservative spatial discretization and timestep restrictions; `BouncingBall` demonstrates event location, instantaneous state reset and accumulation of discrete transitions. A separately scoped `CoupledThermalRC` fixture introduces two coupled subsystems and communication-step errors.

**Coverage:** finite-dimensional differential equations, initial/boundary-value problems, one-dimensional diffusion, discrete-event and hybrid transitions, low-order mechanical dynamics, uncertainty propagation, model coupling, solver interoperability, checkpoint semantics, and numeric/scientific evaluation.

**Integration boundary:** P09-D03 owns control decisions, supervisor admission and device-facing authority. P09-D04 owns distributed work delivery and failure reconciliation. P09-D01 may consume a simulation through a separately qualified environment adapter. P08 owns game-world causality and scene truth. P10-D02 will own cross-experiment portfolio management and research orchestration; D01 owns one simulation's state, numerical execution and scientific evidence.

**Runtime classes:** `FINITE_REFERENCE`, `LOCAL_NUMERICAL`, `PDE_DISCRETIZATION`, `HYBRID_EVENT`, `CO_SIMULATION`, `NATIVE_SOLVER`, `DISTRIBUTED_SIMULATION`. Each qualification claim identifies the class, model family, parameter domain and backend.

### 0.1 Separation of claim types

| Claim | Minimum evidence | Ceiling |
|---|---|---|
| `MODEL_WELL_FORMED` | Units, topology, initial/boundary data, closure, valid parameter domain | A coherent mathematical specification |
| `REFERENCE_CORRECT` | Analytic/manufactured solution, exact finite oracle or independent derivation | Named finite model and arithmetic profile |
| `NUMERICALLY_QUALIFIED` | Convergence, stability/constraint evidence, event/mesh errors and reproducible execution | Solver + model + precision + tested region |
| `COUPLING_CONFORMANT` | Interface/time/rollback semantics, conservation/interface residuals and negative cases | Named coupled model and interface revisions |
| `BACKEND_QUALIFIED` | Repeated cross-backend numeric tests, performance and fault evidence | Pinned solver/ABI/host/platform and workload |
| `PHYSICAL_VALIDATED` | Independent empirical measurement, uncertainty/calibration and domain expert acceptance | Specified physical operating envelope |

These claims may be awarded independently; qualification of a numerical scheme and validation of a physical model answer different questions.

## 1. Source and implementation evidence ledger

Evidence labels: `STANDARD` is normative interface documentation; `DOC` is official maintainer documentation; `SOURCE` is publicly inspectable implementation; `LFR` is a committed design; `PROPOSED` is a new unimplemented contract. Proposed transfer is a design hypothesis until runtime tests establish it.

| Source and inspected documentation | Engineering evidence | Design transfer | Qualification task |
|---|---|---|---|
| [FMI 3.0.x specification](https://fmi-standard.org/docs/main/) (`STANDARD`) | Model Exchange delegates integration to importer; Co-Simulation may contain a solver; Scheduled Execution delegates partition scheduling; Co-Simulation supports early return | Distinct ME/CS/SE adapters with negotiated time, state and rollback capabilities | Lifecycle, clocks, event mode, early return and FMU version tests |
| [FMI 3.0 Implementers' Guide](https://modelica.github.io/fmi-guides/main/fmi-guide/) (`DOC`) | Implementers encounter divergent FMU behavior and compatibility corner cases | Adapter acceptance matrix and malformed/partial FMU fixtures | Negative capability and error/compatibility tests |
| [SUNDIALS](https://sundials.readthedocs.io/en/latest/) (`DOC/SOURCE`) | CVODE: ODE Adams/BDF; ARKODE: explicit/implicit/multirate; IDA: DAE; associated linear/nonlinear solver modules | Specialized native solver-port candidates | Stiff ODE, residual, root, memory and ABI closure |
| [PETSc TS](https://petsc.org/main/manual/ts/) (`DOC/SOURCE`) | Scalable ODE/DAE timestepping, adaptive error control, implicit residual/Jacobian interfaces | Large-system and PDE semidiscrete integration candidate | Sparse scaling, adaptivity and distributed-memory evidence |
| [SciML OrdinaryDiffEq](https://docs.sciml.ai/DiffEqDocs/stable/solvers/ode_solve/) (`DOC/SOURCE`) | Distinct stiff/nonstiff, symplectic, SSP, BDF, implicit and interpolant methods | Algorithm-to-equation selection matrix; optional external solver bridge | Compare reference formulas and backend-specific solutions |
| [FEniCSx tutorial](https://jsdokken.com/dolfinx-tutorial/v0.10/chapter2/intro.html) (`DOC/SOURCE`) | Weak formulations, PDE discretizations, heat equation and finite elements | Spatial/domain discretization port and conservation diagnostics | Manufactured solution, mesh refinement, boundary treatment |
| [Gmsh documentation](https://gmsh.info/doc/texinfo/) (`DOC/SOURCE`) | Versioned mesh topology, meshing and geometry processing | Mesh import and validation candidates | Units/topology/connectivity and geometry audit |
| [Project Chrono](https://projectchrono.org/) (`DOC/SOURCE`) | Rigid/flexible dynamics, constraints, contacts and multiphysics integration | Qualified physical-engine backend for mechanical workloads | Contact, constraint, time integration and data parity |
| [Gerbil reference](https://cons.io/reference/) and [Gambit manual](https://gambitscheme.org/latest/gambit.pdf) (`DOC`) | Gerbil/Gambit facilities must be validated for numerical layout, type assumptions and foreign memory ownership | Compile-proven small reference and explicitly pinned FFI ports | Toolchain probe, checked numeric conversion and resource lifetime |
| LFR [K0-D01](LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md) / [K1-D01](LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md) (`LFR`) | K0 exact identity, validation and receipts; K1 M05 time, M13 interop, M17 numeric validity, M18 coupling candidates | Foundation plus optional reusable mechanisms | Standalone core and selected integration conformance |
| LFR [P09-D03](LFR-P09-D03-control-autonomous-systems-architecture-2026-11-13.md) / [P09-D04](LFR-P09-D04-distributed-systems-coordination-architecture-2026-11-14.md) (`LFR`) | Controller supervision and distributed coordination have independent domains and authority | Distinct plant-observation and remote-work adapters | Control/delivery adapter negative tests |

**Source boundary:** open-source and standard documents establish feasible interfaces and testable algorithms. This design contains original contracts and independently derived reference equations; it has no copied solver source. Dependency licenses and runtime versions require a fresh, pinned inspection before implementation.

## 2. Architecture and ownership

```text
               Model Author / Simulation Application
                            |
             ModelSpec + InitialCondition + RunSpec
                            v
               +-- P10-D01 PUBLIC CONTRACT --+
               | validate / initialize / run |
               | advance / observe / report  |
               +--------------+--------------+
                              v
               +-- GERBIL REFERENCE CORE -----+
               | model registry and units     |
               | state and parameter closure  |
               | time/event state machine     |
               | integrator/discretizer refs  |
               | trace, invariants, oracle    |
               +--------------+--------------+
                              |
                      Capability ports
             +-----------+-----------+-------------+
             |           |           |             |
          SolverPort   MeshPort    CouplingPort   CheckpointPort
             |           |           |             |
       Gerbil/native   grid/FE    FMI ME/CS/SE  memory/durable
       SUNDIALS/PETSc  adapters   local/service  optional adapters
                              |
                 Evaluated trajectory / errors

K0: shared contract vocabulary. K1: selected mechanisms.
P09-D03, P09-D01, P08, P09-D04, P10-D02: optional integration ports.
```

### 2.1 Dependency classes

| Class | Component | Finite reference needs | Contract |
|---|---|---|---|
| Foundation | K0 identity, schema, outcome, provenance | Contract vocabulary | Model/solver/state/evidence revision locks |
| D01-owned core | Time, state, validation, reference solver, invariant checks | Yes | Local deterministic oracle execution |
| Capabilities | `SolverPort`, `MeshPort`, `CouplingPort`, `CheckpointPort`, `TracePort` | Yes as interfaces; in-memory implementations initially | Declared numerical, temporal and state-lifecycle semantics |
| Optional mechanisms | K1 time/numeric/interop/coupled-system candidates | Selected independently | Qualified mechanism profile |
| Optional integrations | P08, P09-D01/D03/D04, P10-D02 and other applications | No | Adapter-specific request/result interpretation |
| Optional infrastructure | SUNDIALS, PETSc, FMI importers, Gmsh, Chrono, Julia service | No | Pinned backend version, ABI and empirical performance |

**Standalone acceptance:** the three reference fixtures are specified independently of other specialist runtimes and external solvers. Future compiled Gerbil fixtures must pass their applicable tests using K0 contracts and local capability implementations.

### 2.2 Responsibility partition

| Owner | Owns | Port into D01 |
|---|---|---|
| P10-D01 | Physical model, simulation state, integrator choice, trajectory, event localization, numerical errors | Simulation API |
| P09-D03 | Control objective, estimator/controller/supervisor, command admission | Frozen control input and measured observation adapter |
| P09-D04 | Remote work delivery, leases, quorum and reconciliation | Remote-simulation job envelope and result receipt |
| P09-D01 | Learner, policy updates and evaluation | Episode/observation adapter with explicit time mapping |
| P08 | World truth, scene state, game rules, materialization | Authored physical-model import and output adapter |
| P10-D02 (planned) | Experiment plans, datasets, benchmark campaigns and cross-run governance | D01 result/evidence artifacts |
| External host | Real device effects, access control and empirical measurement | Controlled measurement/input only |

The simulation's committed state is authoritative **within its model execution**. A consumer decides whether and how its domain state incorporates a simulated result.

### 2.3 Capability-port semantics

```text
ModelPort.validate(model_spec, parameter_lock) -> ModelVerdict | TypedError
SolverPort.initialize(model, initial, solver_profile) -> SolverHandle | Error
SolverPort.propose_step(handle, state_anchor, h, input_slice)
  -> CandidateState + ErrorEstimate + DenseOutputRef? + SolverReceipt | Rejected
EventPort.locate(state_anchor, candidate, event_profile)
  -> EventBracket + RootEvidence | None | Unresolved
CheckpointPort.capture(full_closure, scope) -> SnapshotReceipt | Unsupported
CheckpointPort.restore(snapshot, matching_profile) -> StateAnchor | Rejected
CouplingPort.exchange(link, t_comm, port_values, revision_locks)
  -> ExchangeReceipt | EarlyReturn | Unknown
TracePort.record(step_or_event_receipt, provenance) -> TraceAnchor | Error
```

**Step ownership:** the numerical solver proposes an increment; the simulation driver commits only a validated candidate with a precise `t_accepted`. A rejected attempt retains its rejection and proposed step size without mutating the published state. The port advertises interpolation, event-location, sensitivity, state serialization and rollback capabilities individually.

## 3. Model representations, units and physical domains

```text
PhysicalModelSpec {
  model_ref, model_revision, equation_family: ODE | DAE | PDE | DISCRETE | HYBRID,
  variable_manifest[], parameter_manifest[], physical_constants[],
  coordinate_frame_ref, units_profile_ref, time_domain_ref,
  domain_topology_ref_optional, discretization_ref_optional,
  rhs_or_residual_ref, initial_condition_schema,
  boundary_condition_refs[], forcing_input_schema,
  constraint_invariant_refs[], event_refs[],
  stochastic_profile_ref_optional, validity_domain_ref,
  source_provenance_refs[], numeric_profile_ref
}
StateAnchor {
  run_id, model_revision, parameter_revision, state_revision,
  accepted_time, continuous_state_ref, discrete_mode_ref_optional,
  mesh_revision_optional, active_boundary_revision,
  pending_events[], solver_state_ref_optional, rng_counter_ref_optional
}
RunSpec {
  model_lock, initial_state_lock, solver_profile_ref, error_budget_ref,
  t_start, t_end, output_schedule_ref, event_budget_ref,
  memory_and_wall_budget, backend_lock, reproducibility_profile_ref
}
```

**Equation families:** `ODE` uses an initial-value right-hand side `dy/dt=f(t,y,p,u)`; `DAE` uses a residual `F(t,y,ydot,p,u)=0` and consistency initialization; `PDE` specifies spatial domain, operators, fluxes/source and boundary data before spatial discretization; `DISCRETE` uses an exact declared transition; `HYBRID` adds modes, guards and reset maps.

**Units and dimensions:** values carry a dimensional signature and a declared conversion witness. For instance, `m`, `kg`, `s`, `K`, `J` and `N` are distinct; using centimeters for a meter-valued state requires an explicit scale conversion. Affine units such as Celsius require offset-aware conversion rather than naive multiplication. Dimensional checking validates *typed equations and input bindings*, while solver arithmetic may use dimensionless/scaled arrays after normalization.

**Frames and topology:** physical coordinates, mesh index coordinates and visualization frames are separate. Periodic, Dirichlet, Neumann, Robin and mixed boundary profiles have different flux meaning. Mesh adjacency, orientation, volume weights and ghost-cell semantics belong to the discretization revision.

**Parameter validity:** records specify physical constraints and uncertainty, e.g. `m>0`, `k>0`, `c>=0`, diffusion coefficient `kappa>0`, restitution `0<=e<=1`, and bounded initial data. Out-of-domain input returns `MODEL_PARAMETER_INVALID` with the offending field and dimensional witness.

### 3.1 Scientific state and evaluation artifacts

`SimulationState` includes the accepted model state and domain-specific conserved quantities. `Measurement` is an observation possibly filtered, delayed or noisy. `Estimate` belongs to the estimator or data-analysis profile. `TrajectorySample` records a timestamp, state scope, units and interpolation method. A run report binds all four rather than making an observation silently replace simulation state.

`PhysicalValidityProfile` captures where the model is meaningful. `NumericalValidityProfile` captures where a discretization/solver is qualified. Their intersection determines the permitted scope of scientific conclusions.

## 4. Numerical algorithms selected by equation and goal

| Workload | Reference / candidate algorithm | Main measurable requirements | Failure or counterexample |
|---|---|---|---|
| Small nonstiff ODE | Euler, RK4 reference; adaptive explicit RK as backend | Step-order convergence and solution error | Unstable/underresolved step or wrong tolerance norm |
| Long-time conservative mechanics | Symplectic Euler, velocity-Verlet, selected symplectic backend | Energy error structure, phase drift, momentum | Unbounded artificial energy drift under declared Hamiltonian profile |
| Stiff ODE / reaction system | Implicit Euler small reference; BDF/Rosenbrock/SDIRK native | Residual, stable large stiffness ratio, nonlinear convergence | Explicit stability violation or solver success with large residual |
| DAE / constrained dynamics | Index/profile check, consistent init; IDA-type backend | Constraint manifold residual, rank and Jacobian assumptions | Inconsistent initial algebraic state |
| Diffusion PDE | Conservative finite-volume flux; implicit/explicit time candidate | Mass balance, spatial convergence, positivity profile | Wrong boundary flux or unstable `r` |
| Wave/advection PDE | Finite-volume/finite-difference schemes, SSP where justified | Dispersion, numerical flux, CFL and shock diagnostics | Nonphysical oscillation, mass leak, or unstable timestep |
| Mechanics contact | Hybrid event reference; Chrono/physics backend | Time-of-contact bracket, momentum/energy law, nonpenetration | Missed contact, event chattering, invalid restitution |
| Thermal/structural field | FEniCSx/PETSc-class adapter when selected | Mesh topology, weak/strong residual, refinement | Mesh-inverted geometry or unconverged linear solve |
| Coupled subsystems | Explicit staggered or fixed-point iteration; FMI CS/ME/SE | Exchange conservation, communication error, event timing | Nonconverged interface or hidden algebraic loop |
| Ensembles/stochastic | Counter-based streams, stable aggregation, optional SDE solver | Ensemble statistics, scheme weak/strong errors by task | RNG-stream reuse or false exact replay |

**Selection policy:** classify equation family, stiffness, invariants, event characteristics, required error, space/time scales, Jacobian/mesh structure and expected workload. Only then shortlist algorithms that provide the necessary interface and numerical guarantees. A reference implementation is qualified by correctness first; native/external candidates are compared on end-to-end performance and evidence quality.

### 4.1 Error and stability vocabulary

- **Consistency:** discrete operators approximate the specified continuous law as resolution improves.
- **Convergence:** computed solution approaches the specified reference in a declared norm as spatial/time resolution improves, under method assumptions.
- **Stability:** perturbations remain controlled according to a named method and problem class; timestep restrictions are part of the claim.
- **Local truncation error:** error of a single accepted numerical step under the method's reference assumptions.
- **Global solution error:** accumulated difference from an independent reference at common physical times.
- **Invariant residual:** departure from an exact conservation, dissipation or constraint law of the stated model.
- **Model discrepancy:** mismatch between physical observations and the mathematical model, beyond numerical error.

Tolerance settings use absolute/relative scales attached to each state field. A norm such as `max_i |err_i| / (atol_i + rtol_i*scale_i)` is a declared algorithm criterion, with `scale_i` defined by the solver profile. A returned success status is included in the evidence record together with actual residuals and accepted/rejected steps.

## 5. Time, event, numerical state and checkpoint lifecycle

**Clock domains:** physical/simulation time `t`, event logical order, solver internal step index, output sample time, coupling communication time, wall-clock time, and any external host deadline carry separate identities.

```text
CREATED -> VALIDATED -> INITIALIZED -> RUNNING
RUNNING -> STEP_PROPOSED -> STEP_ACCEPTED -> RUNNING
                         -> STEP_REJECTED -> RUNNING
RUNNING -> EVENT_BRACKETED -> EVENT_LOCATED -> RESET_APPLIED -> RUNNING
RUNNING -> PAUSED / COMPLETED / FAILED / BUDGET_EXHAUSTED
```

**Step transaction:** capture `(state_rev, t_n, model_rev, input_rev, solver_rev)`; propose `h`; evaluate solver stages; validate finite numeric values, domain/boundary and per-step error; localize any event that must interrupt the proposed interval; commit exactly one accepted state/time/revision. If event localization detects an interior guard, only the pre-event portion is committed and its reset follows the guard's ordering profile.

**Event semantics:** a guard has a sign, direction, hysteresis or crossing condition, an event-priority rule, a root bracket, a tolerance and a reset map. Multiple simultaneous guards require a declared resolution policy. Bounding event counts per interval and detecting a vanishing inter-event time prevent silent unbounded event processing.

**Checkpoint closure:** state, time, discrete mode, model and parameter revisions, mesh and boundary revision, solver internals if resumable, RNG counters, pending events, input history needed by interpolation, and backend numeric profile. A state-only snapshot supports observation/restart from that state only when a compatible reconstruction procedure is proven; it carries no general exact-replay claim.

**Determinism grades:** `MODEL_DETERMINISTIC` (equations and inputs), `ORDER_DETERMINISTIC` (event and reductions order), `NUMERIC_TOLERANCE_REPRODUCIBLE` (bounded cross-platform discrepancies), `BITWISE_REPLAY` (fully pinned arithmetic, schedule and backend). A seed alone is an insufficient replay lock when solver scheduling or floating-point reductions differ.

**Failure receipt:** `STEP_REJECTED`, `ERROR_TOLERANCE_EXCEEDED`, `EVENT_BRACKET_UNRESOLVED`, `EVENT_BUDGET_EXHAUSTED`, `INCONSISTENT_DAE_INITIALIZATION`, `NONFINITE_STATE`, `BOUNDARY_VIOLATION`, `COUPLING_NONCONVERGED`, `ROLLBACK_UNSUPPORTED`, `BACKEND_DISCONNECTED`, `OUTPUT_SCOPE_INVALID` and `BUDGET_EXHAUSTED` retain model/time/revision identity and the last published anchor.

## 6. Spatial discretization, fluxes and conservation

The `DiscretizationSpec` binds mesh topology, cell/node meaning, quadrature, face orientation, boundary placement, interpolation/projection, discrete operator and refinement map. A discretized PDE is a new finite system whose residual and conservation laws are auditable in addition to the continuous model.

### 6.1 Finite-volume reference

For a scalar conserved field `u_i` in a cell of volume `V_i`, the semidiscrete balance is:

```text
V_i * d(u_i)/dt = sum_faces( signed_face_flux * area ) + V_i * source_i
```

Sign convention: `signed_face_flux` is positive into the cell. An interior face has opposite signs in its two neighbor cells. Summing all cells cancels interior fluxes exactly in mathematical arithmetic; external flux and sources account for changes of total mass/heat quantity.

A monotone positivity guarantee and mass conservation are distinct requirements. Floating-point summation error is reported in a declared norm; exact conservation is a symbolic property of the discretization under periodic or zero-flux boundaries.

### 6.2 Mesh and boundary contracts

Mesh import verifies positive/valid measures, element connectivity, boundary labels, orientation, coordinate frame and unit scale. Interpolation between meshes declares whether it preserves integral, positivity, continuity or a weaker named profile. Remeshing changes the state-space revision and invokes a projection witness. Output frames and render meshes are separate from authoritative numerical mesh state.

### 6.3 Multi-resolution and error control

Spatial refinement `dx -> dx/2` is paired with time refinement chosen to keep temporal error below the spatial comparison. Error reports distinguish continuum/manufactured-reference error from semidiscrete-exact evolution error. Richardson estimates require smooth-enough solutions, matching norms and a verified asymptotic refinement regime.

## 7. Coupled models and FMI integration

### 7.1 Coupling contracts

```text
CouplingGraph {
  component_refs[], typed_ports[], edge_map[], unit_and_frame_transforms[],
  algebraic_loop_groups[], causality_profiles[],
  communication_clock_ref, interpolation_extrapolation_rules[],
  solver_ownership_by_component[], rollback_capabilities[],
  event_resolution_profile, convergence_norm, iteration_budget,
  dependency_and_version_locks
}
CouplingExchange {
  graph_revision, time_anchor, component_state_anchors[],
  input_output_digests[], port_unit_witnesses[],
  accepted_time, residual_vector, early_return_ref_optional,
  rollback_receipts[], status
}
```

**Coupling approaches:** explicit one-way exchange; explicit staggered two-way exchange; fixed-point/iterative coupling for compatible rollback/restore capabilities; monolithic solve when supported by a selected backend. Each approach has its own consistency, algebraic-loop and convergence conditions.

**Common time:** a requested communication time is accepted only when each participating component reports compatible advancement to that time. Early return shortens the joint interval and triggers a synchronization decision. Mixed variable-step solvers require dense-output or explicit interpolation policies; extrapolation carries a validity horizon and quantified error where supported.

**Rollback:** an importer can retry a coupled communication interval only for components declaring a compatible restore capability or a separately valid recomputation closure. A component that has irreversible external side effects is isolated behind an effect-host contract; it cannot be treated as numerically reversible because an FMU interface exposes a state snapshot.

### 7.2 FMI interface profiles

| FMI 3 family | Time/solver ownership | Adapter responsibility | Negative case |
|---|---|---|---|
| Model Exchange (ME) | Importer controls integration and time progression | Provide derivative/event state/continuous-state mappings and consistent initialization | Importer uses Co-Simulation `doStep` semantics on ME |
| Co-Simulation (CS) | FMU advances its own model using declared internal numerical path | Handle `doStep`, early return, intermediate events, status and optional state save | Importer assumes requested time was reached after early return |
| Scheduled Execution (SE) | Importer schedules exposed model partitions | Resolve clocks, activation order and partition status | Scheduler ignores clock dependency or activation order |

A binding verifies the FMU's actual capability flags, platform binary, model-description XML, value references, declared units and state/lifecycle transitions against a pinned FMI version. FMI conformance and domain-specific scientific accuracy are evaluated separately.

### 7.3 Cross-specialist adapters

- **P09-D03:** simulation emits model state or measured-observation envelopes; controller inputs are sampled at declared times; the application host admits real controls separately.
- **P09-D04:** asynchronous simulation jobs carry exact model, solver, input and numerical profile locks; distributed delivery does not determine mathematical time or simulation commit semantics.
- **P09-D01:** simulation can supply an environment, but a simulation time horizon and an RL episode termination are different typed events.
- **P08:** physics values may inform world-state decisions through P08's domain adapter; it decides authoritative world outcomes.
- **P10-D02:** campaign-level parameter sweeps and reports consume D01 evidence without changing its solver result after publication.

## 8. Exact reference fixture A: OscillatorLab

### 8.1 Physical model and analytic oracle

For mass `m>0`, stiffness `k>0`, damping `c>=0` and no external forcing:

```text
m * x''(t) + c * x'(t) + k * x(t) = 0
omega0 = sqrt(k/m)
zeta   = c / (2*sqrt(k*m))
```

The underdamped analytic reference (`0 <= zeta < 1`) is:

```text
alpha = c/(2*m)
omegad = sqrt(k/m - alpha^2)
x(t) = exp(-alpha*t) * (
         x0*cos(omegad*t)
       + ((v0 + alpha*x0)/omegad)*sin(omegad*t))
v(t) = exp(-alpha*t) * (
         v0*cos(omegad*t)
       - ((alpha*v0 + (k/m)*x0)/omegad)*sin(omegad*t))
E(t) = 0.5*m*v(t)^2 + 0.5*k*x(t)^2
E'(t) = -c*v(t)^2
```

`v(t)` is the closed-form derivative of the position formula, independently checked by symbolic differentiation rather than differentiated from numerical output. Critically damped and overdamped regimes use their own exact formulas or an independent matrix exponential oracle, with continuity tests near critical damping.

**Oracle A1:** compare `x(t),v(t)` against the analytic solution at a set of fixed physical times. **Oracle A2:** measure energy balance `E(0)-E(T)-integral_0^T(c*v^2 dt)` using an independently controlled integration of the analytic velocity. **Oracle A3:** for `c=0`, evaluate energy and phase error across many periods with fixed steps.

### 8.2 Algorithm tour and acceptance

- `ExplicitEuler`: simple first-order comparator; for undamped oscillation it produces numerical energy growth, a useful negative fixture.
- `SymplecticEuler`: one of the two named update orderings is pinned; measure long-time bounded/oscillatory energy behavior for a stable step domain.
- `VelocityVerlet`: undamped separable Hamiltonian profile with exact update order; measure phase and energy structure. The damped profile requires a separately specified method.
- `RK4`: nonstiff high-accuracy baseline with observed fourth-order convergence in a sufficiently refined regime.
- `AdaptiveRK`: later backend with error-controller receipts, dense output and rejected-step traces.

**Acceptance fields:** method revision, `(m,c,k,x0,v0)`, unit profile, time step/output schedule, error norm, peak state, energy residual, accepted/rejected steps, CPU/memory cost and diagnostic disposition. A time-step warning is tied to the declared method and model parameters.

## 9. Exact reference fixture B: PeriodicHeat1D

### 9.1 Finite-volume semidiscrete model

On a periodic domain of length `L>0`, choose `N>=4` uniform cell values with `dx=L/N`, constant `kappa>0` and no source:

```text
d(u_i)/dt = kappa * (u_{i+1} - 2*u_i + u_{i-1}) / dx^2
u_{i+N} = u_i
M = dx * sum_i(u_i)
```

Initialize a Fourier mode with `j` an integer in `1..floor((N-1)/2)` and `|A|<=mean` when positivity is part of the selected fixture:

```text
u_i(0) = mean + A*cos(2*pi*j*i/N)
lambda_j = -(4*kappa/dx^2) * sin(pi*j/N)^2
u_i(t) = mean + A*exp(lambda_j*t)*cos(2*pi*j*i/N)
```

This is the **exact solution of the semidiscrete ODE system**, so it isolates temporal integration error from spatial truncation. The continuum Fourier decay rate `-(kappa*(2*pi*j/L)^2)` supplies a separate spatial convergence oracle as `N` increases with `L,j,kappa,t` fixed.

**Oracle B1:** discrete mass `M` remains constant under periodic fluxes. **Oracle B2:** each cell follows the semidiscrete exponential Fourier solution within the declared numerical norm. **Oracle B3:** on a smooth fixed mode, the discrete spatial decay-rate error reduces asymptotically with second order in `dx`.

**Forward Euler stability/positivity profile:** for `r=kappa*dt/dx^2`, nonnegative coefficients and `r<=1/2` imply each new cell value is a convex combination of the current cell and its two neighbors. The violating `r>1/2` case must be included as a detected invalid step **for a profile claiming this positivity guarantee**; other methods have their own restrictions and may remain stable for different bounds.

### 9.2 Conservation and boundary variations

A zero-flux variant should preserve mass, whereas Dirichlet boundary conditions introduce a physical boundary flux and generally change total mass. Add an explicit source field to validate `dM/dt = sum_sources*dx + net_boundary_flux`. A deliberately reversed interior face sign detects broken conservation even when plots remain smooth.

## 10. Reference fixture C: BouncingBall hybrid dynamics

Take height `h>=0`, upward velocity `v`, gravity `g>0` and restitution coefficient `e in [0,1]`. Between impacts:

```text
h(t) = h0 + v0*t - (g/2)*t^2
v(t) = v0 - g*t
impact_time = (v0 + sqrt(v0^2 + 2*g*h0))/g
impact_guard: h=0 with downward velocity (v<0)
impact_reset: (h,v) -> (0,-e*v_before)
```

The formula assumes a valid free-flight segment and identifies the next downward ground crossing. If a ball begins at contact with upward velocity, the event detector must disambiguate the initial zero from a future impact. If `e=0`, a resting/contact mode prevents repeated zero-duration impacts.

**Oracle C1:** the bracket contains the analytic impact time. **Oracle C2:** the post-impact upward velocity equals `-e*v_before`; kinetic energy at instantaneous impact scales by `e^2` when height is zero. **Oracle C3:** each accepted event carries an ordered guard/reset receipt and one mode transition. Accumulating impact intervals approaching zero triggers the declared Zeno/event-budget policy; no infinite event loop is an accepted result.

### 10.1 CoupledThermalRC bridge (conditional)

Two first-order thermal capacitances connected by a conductance define an independent low-dimensional coupling experiment with an analytic combined-system oracle. Use an explicit staggered exchange first and compare against the monolithic solution, then qualify iterative coupling with residual bounds. This fixture precedes importing arbitrary FMUs or PDE meshes.

## 11. Uncertainty, sensitivity and performance evidence

**Uncertainty sources:** initial-condition distributions, uncertain parameters, stochastic forcing, discretization error, solver tolerance, finite arithmetic, calibration discrepancy and observation noise are recorded separately.

**Deterministic parameter sweeps:** input sets contain exact parameter identity and per-run seed/method locks; report min/max/percentiles only for a declared sample design. Multi-run scheduling may be delegated later to P10-D02 or P09-D04 while each individual simulation retains D01 state and truth semantics.

**Sensitivity:** finite-difference perturbations declare scale, stencil, truncation sensitivity and roundoff check; automatic/adjoint differentiation becomes an optional backend profile requiring smoothness and event-discontinuity handling. Check sensitivities against analytic derivatives for the oscillator before using a solver's reported gradient as scientific evidence.

**Stochastic dynamics:** SDE profiles must declare Ito versus Stratonovich interpretation, drift/diffusion units, Wiener increment structure, RNG counters, strong/weak error metric and time refinement coupling. The baseline deterministic fixtures establish numeric correctness before a separately named SDE implementation is qualified.

**Performance experiments:** record elapsed time, evaluations (RHS/Jacobian/events), linear/nonlinear iterations, accepted/rejected steps, peak memory, allocations, data-transfer cost, energy if measured, scaling parameters and failure count. Compare algorithms using equal accuracy or a documented error-vs-cost Pareto front; equal timestep does not imply equal accuracy.

## 12. Gerbil implementation profiles and package structure

| Profile | Runtime ownership | Best initial workload | Qualification evidence |
|---|---|---|---|
| **A — Pure Gerbil reference** | Gerbil owns models, small steppers, time, event loop, oracle and trace | Oscillator, small heat grid, bouncing ball and coupling proof-of-concept | Compiled pinned Gerbil/Gambit, analytic oracle, refinement and memory limits |
| **B — Gerbil + qualified native/numerical backend** | Gerbil owns public contracts and admission; selected SUNDIALS/PETSc/Chrono/mesh library computes the heavy numerical operation | Stiff ODE/DAE, large sparse PDE, contact/mechanics, high-throughput ensembles | ABI/ownership, residual/accuracy, platform, memory and end-to-end speed comparison |
| **C — Gerbil + FMU/external simulation service** | Gerbil orchestrates version-locked jobs; selected FMI importer or service owns its declared solver, clock or schedule | Multimodel co-simulation, cross-tool FMU exchange, remote large jobs | FMI mode/lifecycle compatibility, coupling error, time and failure reconciliation |

A selected backend must return a result consistent with its specific solver profile. Different numerical schemes may legitimately produce different approximations within agreed tolerances and conservation requirements.

### 12.1 Proposed Gerbil namespaces

```text
lfr/p10/simulation/
  contract/{model,units,state,time,run,receipt}.ss
  model/{ode,dae,pde,hybrid,validation}.ss
  reference/{euler,rk4,symplectic,verlet,implicit-euler}.ss
  spatial/{grid1d,flux,diffusion,mesh-validate}.ss
  events/{guard,bracket,root,reset,zeno}.ss
  coupling/{graph,exchange,iteration,fmi-contract}.ss
  ports/{solver,mesh,coupling,checkpoint,trace}.ss
  oracles/{oscillator,heat-mode,bounce,thermal-rc}.ss
  adapters/{sundials,petsc,chrono,fmi,service}.ss
  evaluation/{error,conservation,convergence,benchmark}.ss
  tests/{contracts,reference,solver,spatial,event,coupling,parity}.ss
```

**Code-data boundary:** model specs and parameter manifests are parsed as bounded data; trusted authored Scheme modules implement reference right-hand sides and transformations. Loading a model document does not evaluate an arbitrary Scheme expression. Native callbacks are explicit function registrations with ownership, thread and exception contracts established by FFI tests.

### 12.2 Numeric and memory ownership

The reference uses explicit float64 or exact-rational test fixtures where justified. Array ordering, endianness, memory alignment, mutability, pin/copy strategy and lifetime scopes are pinned per backend. A native solver may retain callbacks and vectors across steps; the Gerbil foreign-owner wrapper must maintain validity until the backend confirms release. Callback exceptions map to typed solver failures and preserve the last committed state.

### 12.3 First API sketch

```text
register_model(model_spec) -> ModelRef | ValidationError
create_run(model_ref, initial, run_spec) -> RunHandle | Error
advance_run(handle, target_time, budget) -> RunReceipt | Error
observe_run(handle, time, interpolation_profile) -> Sample | Unsupported
checkpoint_run(handle, closure_profile) -> CheckpointReceipt | Unsupported
resume_run(checkpoint, backend_lock) -> RunHandle | Incompatible
validate_trajectory(run_ref, oracle_profile) -> EvaluationReport
compare_solvers(frozen_problem, candidates, accuracy_profile) -> TourReport
```

The names define a semantic interface and implementation plan. Gerbil-specific signatures, FFI and exact foreign libraries are to be compiled and verified against pinned toolchains during implementation.

## 13. Verification, evaluation and evidence ceilings

### 13.1 Independent oracle policy

Every mandatory reference claim must use an analytic solution, manufactured solution, algebraic law, independently implemented high-precision oracle or exhaustive finite reference whose failure mode differs from the candidate solver. A second invocation of the same solver with the same code path is a regression check, not independent validation.

**Order tests:** run a refinement ladder `h, h/2, h/4, h/8` at fixed physical times with adequate arithmetic precision and smooth inputs. Estimate `p=log2(err(h)/err(h/2))` only in a measured asymptotic regime. Report raw errors, norms and exclusions. Different methods have different expected orders and regularity requirements.

**Conservation tests:** attach domain-specific integral identities with explicit sign, boundary, source and sink conventions. A small residual is qualified relative to physics scale, time span and numerical roundoff; plot appearance has no acceptance role.

**Stiffness tests:** validate the fast/slow eigenmode system with an analytic solution and solver residual; compare explicit restricted steps with an implicit candidate and publish work/accuracy tradeoffs. The smallest successful step is a runtime metric, not automatic proof of mathematical stability outside the tested case.

**Coupling tests:** measure interface residual, drift at shared communication times and error caused by interpolation/early return. A coupled simulation claim includes the exact component and coupling graph revisions.

### 13.2 Negative and recovery cases

A wrong unit, stale mesh, inconsistent initial condition, oversized explicit step, nonfinite state, missed event, failed FMU advance, unsupported rollback, corrupted checkpoint and incomplete native callback must produce a typed failure receipt. After any failure, the last accepted state/time remains queryable. Partial output is marked partial and retains its numerical and evidence scope.

### 13.3 Performance and algorithm qualification

For each job, select a short candidate list after model classification. Pin source and build versions, solver tolerances, linear algebra choices, cores/threads and memory profile. Execute repeated timing with warm-up where applicable, compare on equal error envelopes and record invalid or timed-out cases. `QUALIFIED` applies only to tasks whose scientific and performance gates pass; `CONDITIONALLY_QUALIFIED` names a measured limit; `REJECTED_FOR_TASK` names falsifying evidence; `UNRESOLVED` records missing comparators or incomplete observations.

**Validation ceiling:** analytical error checks certify mathematical/numerical consistency for the named reference. Empirical predictive validity for a physical object requires calibration/measurement and a separate validation activity, using uncertainty and independent observations.

## 14. Core invariants and typed outcomes

### 14.1 Invariants I01–I30

1. **I01 — Model identity:** Every committed state resolves to an exact model revision.
2. **I02 — Parameter closure:** A result binds the complete set of parameter revisions and units.
3. **I03 — Time monotonicity:** Accepted simulation time increases according to the named time profile.
4. **I04 — Step rejection:** Rejected candidate steps leave the published state unchanged.
5. **I05 — Finite state:** Accepted numeric fields satisfy the declared finite-value profile.
6. **I06 — Units:** All external numeric fields have a checked dimensional conversion.
7. **I07 — Frame:** Mesh and world coordinate conversions require explicit frame witnesses.
8. **I08 — Boundary closure:** Every spatial face resolves to one declared interior or boundary relation.
9. **I09 — Conservation:** Flux cancellations match the signed face and source convention.
10. **I10 — Solver identity:** Solver method, parameters and tolerances are version locked.
11. **I11 — State generation:** Checkpoints cannot revive stale state generations.
12. **I12 — RNG partition:** Stochastic streams are bound to named run/subsystem counters.
13. **I13 — Error provenance:** Every error verdict declares norm, scale and comparator.
14. **I14 — Event bracket:** Every committed event has a guard/direction and root bracket or exact witness.
15. **I15 — Event order:** Simultaneous events use a declared, versioned resolution rule.
16. **I16 — Reset order:** An event reset is applied at the located event time before subsequent stepping.
17. **I17 — Zeno cap:** Event accumulation invokes the bounded-event policy.
18. **I18 — DAE initialization:** A DAE run validates algebraic consistency under its profile.
19. **I19 — Mesh revision:** State projections record old/new topology and conservation witness.
20. **I20 — Coupling closure:** Port units and times are checked for every coupling exchange.
21. **I21 — Early return:** A component early return revises the joint communication decision.
22. **I22 — Rollback capability:** Retries requiring restore are attempted only with qualified closure.
23. **I23 — Output sampling:** Interpolation claims name their dense-output or interpolation method.
24. **I24 — External state:** FMU/host status cannot promote an uncommitted numerical candidate.
25. **I25 — Native ownership:** Foreign arrays and callbacks remain valid for their claimed lifetime.
26. **I26 — Resource bounds:** Run steps, events, memory and wall time observe declared budgets.
27. **I27 — Truth separation:** Simulation state, measurement and domain-host state remain distinct.
28. **I28 — Accuracy ceiling:** A solver PASS is scoped to measured model, method and tolerance.
29. **I29 — Audit trace:** Each accepted state change has a correlatable receipt and anchor.
30. **I30 — Standalone:** Finite reference fixtures require only K0 and local D01 capability implementations.

### 14.2 Typed outcomes

`MODEL_INVALID`, `UNITS_MISMATCH`, `INITIAL_CONDITION_INCONSISTENT`, `SOLVER_UNAVAILABLE`, `STEP_REJECTED`, `ERROR_CONTROL_FAILED`, `EVENT_ROOT_UNRESOLVED`, `EVENT_BUDGET_EXHAUSTED`, `COUPLING_NONCONVERGED`, `ROLLBACK_UNSUPPORTED`, `MESH_INVALID`, `CHECKPOINT_INCOMPATIBLE`, `BACKEND_FAILED`, `NUMERIC_NONFINITE`, `BUDGET_EXHAUSTED`, `UNRESOLVED` and `QUALIFIED_FOR_PROFILE` are namespaced result families. A status binds the offending model/step, last valid anchor, retry eligibility and evidence.

## 15. Qualification plan: 80 proposed tests and 12 gates

### 15.1 Gates Q0–Q11

| Gate | Decision | Required execution evidence |
|---|---|---|
| **Q0 — Source and toolchain closure** | Source and toolchain closure | Pinned Gerbil/Gambit, model, algorithm sources, API versions, license and build profile |
| **Q1 — Model/units/schema validity** | Model/units/schema validity | Dimensional/domain checks, malformed fixtures, closure hashes, state revisions |
| **Q2 — Analytic oscillator reference** | Analytic oscillator reference | Closed-form state/energy checks, refinement ladder and negative Euler case |
| **Q3 — Numerical integrator integrity** | Numerical integrator integrity | Order, stability, error-controller and rejected-step traces |
| **Q4 — PDE and conservation** | PDE and conservation | Periodic/zero-flux mass, eigenmode decay, spatial/time refinement |
| **Q5 — Hybrid event correctness** | Hybrid event correctness | Analytic event time, guard direction, reset and Zeno bounds |
| **Q6 — DAE/implicit and stiffness** | DAE/implicit and stiffness | Residual, initialization, nonlinear solve evidence for selected profile |
| **Q7 — Coupling/FMI conformance** | Coupling/FMI conformance | Model/CS/SE time ownership, early return, rollback and interface residual |
| **Q8 — Native/backend parity** | Native/backend parity | ABI, memory, solver-state, error and end-to-end benchmark for selected adapter |
| **Q9 — Resource/security** | Resource/security | Budgets, model-data validation, callback, memory and isolation negative tests |
| **Q10 — Integration conformance** | Integration conformance | Selected P08/P09/D04/P10-D02 or external adapter and domain-source-of-truth checks |
| **Q11 — Independent audit and claim** | Independent audit and claim | Reproducible reports, independent oracle, hashes and uncertainty ceiling |

**Gate applicability:** every reference release requires Q0/Q1/Q2/Q3, applicable Q9 and Q11. Q4 applies to PDE, Q5 to hybrid events, Q6 to DAE/stiff solver claims, Q7 to coupling/FMI, Q8 to selected native backends, and Q10 to selected external integration profiles. A release claim publishes both passed tests and excluded scopes.

### 15.2 Test register

#### Model and identity

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T01** | Unit-compatible oscillator input | accept kg/N/m/s parameter dimensions under declared model |
| **P10-D01-T02** | Unit mismatch in spring law | reject invalid dimensional term and name field |
| **P10-D01-T03** | Affine temperature conversion | preserve correct absolute temperature difference semantics |
| **P10-D01-T04** | Stale model revision | reject step with mismatched model revision |
| **P10-D01-T05** | Parameter validity | reject negative mass and nonpositive diffusivity |
| **P10-D01-T06** | Finite numeric contract | reject NaN/Inf initial state |
| **P10-D01-T07** | Coordinate-frame conversion | transform known point with explicit frame witness |
| **P10-D01-T08** | Wrong mesh orientation | flag invalid topology/orientation fixture |
| **P10-D01-T09** | Missing boundary label | reject unresolved external face |
| **P10-D01-T10** | Run identity and restart | preserve model/solver/initialization locks in checkpoint |

#### Oscillator reference

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T11** | Undamped analytic solution | match cosine/sine state at fixed times |
| **P10-D01-T12** | Underdamped exact position | match exponential-trigonometric analytic oracle |
| **P10-D01-T13** | Underdamped exact velocity | match analytic derivative independent of solver |
| **P10-D01-T14** | Energy dissipation | verify E(0)-E(T)=integrated c*v^2 within declared integration error |
| **P10-D01-T15** | Undamped energy invariant | reference energy constancy for analytic trajectory |
| **P10-D01-T16** | Explicit Euler energy growth | detect known unstable growth for chosen h and period count |
| **P10-D01-T17** | Symplectic Euler phase/energy | measure stable-domain bounded energy oscillation and phase error |
| **P10-D01-T18** | Velocity-Verlet order | observe second-order global state convergence for undamped smooth case |
| **P10-D01-T19** | RK4 refinement order | observe near-fourth-order global error in asymptotic regime |
| **P10-D01-T20** | Critical damping continuity | compare limiting under/critical/overdamped oracle trajectories |

#### Stepper and stiff systems

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T21** | Rejected-step atomicity | last accepted state/time unchanged after error rejection |
| **P10-D01-T22** | Accepted-step monotonicity | time and revision progress once per accepted step |
| **P10-D01-T23** | Error norm normalization | use declared state-wise atol/rtol scale |
| **P10-D01-T24** | Dense output scope | reject observation outside valid interpolation interval |
| **P10-D01-T25** | Adaptive controller cap | stop after declared rejections and preserve trace |
| **P10-D01-T26** | Stiff diagonal analytic pair | compare two-mode exact decay at fixed times |
| **P10-D01-T27** | Implicit Euler scalar decay | match discrete implicit update and residual |
| **P10-D01-T28** | Nonlinear iteration failure | return typed residual/nonconvergence without false success |
| **P10-D01-T29** | Consistent DAE initialization | accept valid algebraic constraint state |
| **P10-D01-T30** | Inconsistent DAE initialization | reject incompatible y,ydot algebraic initial values |

#### PDE and mesh

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T31** | Periodic mass conservation | sum signed interior fluxes to zero |
| **P10-D01-T32** | Discrete Fourier eigenmode | compare heat trajectory to semidiscrete exact exponential |
| **P10-D01-T33** | Continuum spatial refinement | observe second-order eigenvalue convergence for smooth fixed Fourier mode |
| **P10-D01-T34** | Forward Euler r=1/2 | preserve nonnegative convex-combination profile |
| **P10-D01-T35** | Forward Euler r>1/2 | reject positivity guarantee under violating explicit step |
| **P10-D01-T36** | Boundary-flux source accounting | match change in mass to signed flux+source |
| **P10-D01-T37** | Zero-Neumann boundary | preserve mass with zero external flux |
| **P10-D01-T38** | Dirichlet boundary accounting | observe nonzero boundary contribution when present |
| **P10-D01-T39** | Mesh projection and conservation | declare transferred integral error on remesh |
| **P10-D01-T40** | Inverted/invalid cell | reject topology with nonpositive measure |

#### Hybrid dynamics

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T41** | Ball exact impact time | bracket and locate positive analytic root |
| **P10-D01-T42** | Impact guard direction | reject upward crossing as downward impact |
| **P10-D01-T43** | Restitution reset | match v_after=-e*v_before |
| **P10-D01-T44** | Impact energy ratio | match e^2 kinetic energy ratio for e>=0 |
| **P10-D01-T45** | Starting at ground rising | avoid spurious impact at t=0 |
| **P10-D01-T46** | Rest contact at e=0 | enter rest mode without infinite zero-time bounces |
| **P10-D01-T47** | Multiple simultaneous guards | apply declared event priority and trace |
| **P10-D01-T48** | Event inside long step | commit pre-event segment and reset before continuing |
| **P10-D01-T49** | Zeno accumulation | stop/report under event accumulation policy |
| **P10-D01-T50** | Rollback after invalid event | preserve last accepted anchor and event trace |

#### Coupling and FMI

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T51** | Thermal RC monolithic oracle | compare two-state coupled solution |
| **P10-D01-T52** | Staggered exchange error | measure communication-step convergence |
| **P10-D01-T53** | Iterative coupling residual | stop when interface norm passes declared threshold |
| **P10-D01-T54** | Divergent coupling iteration | return COUPLING_NONCONVERGED and preserve anchor |
| **P10-D01-T55** | FMI Model Exchange | map importer-owned solver/time contract correctly |
| **P10-D01-T56** | FMI Co-Simulation early return | honor last successful time before next exchange |
| **P10-D01-T57** | FMI Scheduled Execution | activate partitions by declared clock and order |
| **P10-D01-T58** | FMU incompatible capability | reject requested rollback/state save without support |
| **P10-D01-T59** | Unit mismatch across coupling | reject incompatible connected port dimensions |
| **P10-D01-T60** | FMU stale version reference | reject different model-description/backend revision |

#### Backend and reproducibility

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T61** | Gerbil standalone reference | compile/run with zero specialist runtime imports |
| **P10-D01-T62** | Pinned FFI lifetime | retain native state and callbacks until release |
| **P10-D01-T63** | Numeric endianness/layout | round-trip representative vectors with exact conversion evidence |
| **P10-D01-T64** | Native callback failure | return typed error and preserve accepted anchor |
| **P10-D01-T65** | Backend numeric parity | compare trajectories against independent oracle and tolerance envelope |
| **P10-D01-T66** | Backend over-tolerance | reject mismatched numerical output despite success status |
| **P10-D01-T67** | Checkpoint closure | resume with exact compatible model/solver/time/RNG revision |
| **P10-D01-T68** | Checkpoint revision mismatch | reject stale/incompatible restore |
| **P10-D01-T69** | Seed and scheduling separation | report deterministic-vs-nondeterministic replay grade |
| **P10-D01-T70** | Equal-error performance tour | measure end-to-end costs at matched numeric accuracy |

#### Integration, safety and audit

| Test | Fixture / challenge | Required observation |
|---|---|---|
| **P10-D01-T71** | D03 control sampling | deliver time-aligned simulation observation to selected adapter |
| **P10-D01-T72** | P09-D01 environment mapping | separate simulation time from episode termination |
| **P10-D01-T73** | D04 remote task mapping | preserve model/solver locks across delivery and reconnect |
| **P10-D01-T74** | P08 world-scene boundary | return simulation observation without domain-state admission |
| **P10-D01-T75** | P10-D02 experiment artifact | export self-contained run/evidence manifest |
| **P10-D01-T76** | Untrusted model input | reject executable payload and preserve validation trace |
| **P10-D01-T77** | Resource budget | stop bounded run on memory/wall/step limits |
| **P10-D01-T78** | Independent reviewer reproduction | replay one fixture from locked manifest and compare oracle |
| **P10-D01-T79** | Validation claim ceiling | decline physical-validation claim without empirical evidence |
| **P10-D01-T80** | Qualification partial/unknown | record unresolved tests without promoting untested backend |

### 15.3 Claim and release matrix

| Release profile | Mandatory additional gate(s) | Permitted claim |
|---|---|---|
| Oscillator reference | Q0/Q1/Q2/Q3/Q9/Q11 | `REFERENCE_CORRECT` and bounded `NUMERICALLY_QUALIFIED` for named methods |
| Periodic heat reference | Q0/Q1/Q3/Q4/Q9/Q11 | `PDE_DISCRETIZATION_QUALIFIED` under selected mesh/time profile |
| Hybrid event reference | Q0/Q1/Q3/Q5/Q9/Q11 | `HYBRID_EVENT_QUALIFIED` for selected event profile |
| Stiff ODE/DAE backend | Q0/Q1/Q3/Q6/Q8/Q9/Q11 | `NATIVE_SOLVER_QUALIFIED` under pinned model/solver ABI |
| FMI/coupled model | Q0/Q1/Q3/Q7/Q8/Q9/Q11 | `COUPLING_CONFORMANT` under named modes/time graph |
| External application integration | Base profile plus applicable Q10 and external-host gate | `INTEGRATION_CONFORMANT` for selected adapter only |
| Empirical physical model validation | Numerical profile and independent empirical validation protocol | `PHYSICAL_VALIDATED` only for the measured physical envelope |

**Execution evidence:** each test must record input/model hash, expected result/oracle, measured output, threshold/norm, solver/backend revision, failure trace and disposition. Tests are design specifications until executed; their presence does not constitute a PASS.

## 16. Implementation sequencing post separate authorization

1. **D01-00 — Evidence closure:** pin official solver/API sources, Gerbil/Gambit version, numeric profile, test oracles and dependency licenses.
2. **D01-01 — Model contract and analytic oscillator:** implement exact validated model/state contracts, oracle, Euler, symplectic Euler, Verlet and RK4; execute T01–T20.
3. **D01-02 — Step lifecycle and stiff reference:** rejected-step immutability, error and residual checks, implicit Euler, small stiff oracle; execute T21–T30.
4. **D01-03 — Spatial finite-volume core:** periodic heat, conservative flux, mesh/boundary checks and convergence; execute T31–T40.
5. **D01-04 — Hybrid event core:** guard, root bracket, reset, rest/Zeno policy; execute T41–T50.
6. **D01-05 — Coupling reference:** coupled thermal RC, scheduled exchanges and FMI interface probes; execute selected T51–T60.
7. **D01-06 — Native candidate tour:** SUNDIALS/PETSc or other selected backend only after task profile and oracle qualify; execute applicable T61–T70.
8. **D01-07 — Interoperability and independent audit:** optional D03/D04/P08/P09-D01/P10-D02 adapters, Q0–Q11, publication of evidence, unresolved cases and claim scope; execute T71–T80 where applicable.

## 17. Open engineering decisions

| Decision | Initial baseline | Evidence before extension |
|---|---|---|
| Primary numeric types | Pinned Float64 and exact-rational small oracles | Precision, roundoff and performance tests |
| First compiled integrators | Euler, symplectic Euler, velocity-Verlet, RK4 | Analytic state, energy and order verification |
| Adaptive integrator | Selected nonstiff method after baseline | Error-controller and dense-output parity |
| DAE handling | Consistency validation + native IDA candidate | Residual/Jacobian tests and index assumptions |
| Spatial scheme | Conservative finite volume, periodic heat | Flux, positivity, refinement and boundary tests |
| First multiphysics profile | Coupled thermal RC with common-time exchange | Monolithic oracle and interface residual |
| FMI import/export | Import-side binding after model/solver ownership audit | ME/CS/SE lifecycle and state capability tests |
| Mechanical contact | Analytic bounce first; Chrono later | Contact/nonpenetration/constraint and performance oracles |
| Long-running simulation | Local bounded run first | Checkpoint closure, durability and recovery proof |
| Distributed execution | Optional D04 job adapter | Independent numeric truth and remote receipt reconciliation |
| UI and visualization | Trace/sample export; optional P07 viewer | Dimension/frame mapping and non-authoritative presentation |
| Experimental campaign | D01 produces evidence; D02 later coordinates many runs | Manifest versioning, comparability and provenance |

## 18. Final design verdict

**Design maturity:** source-grounded, mathematically specified scientific simulation architecture with a Gerbil reference target, three independent analytic/discrete reference families, explicit numerical methods, conservation/event/coupling contracts, 30 core invariants, 80 proposed tests and 12 qualification gates. Runtime evidence and empirical physical validation are pending.

```text
P10_D01_SCIENTIFIC_PHYSICAL_SIMULATION_DESIGN_PROPOSED /
GERBIL_REFERENCE_AND_ANALYTIC_ORACLE_FIRST /
MODEL_STATE_UNITS_TIME_DISCRETIZATION_SOLVER_DISTINCT /
ODE_DAE_PDE_HYBRID_SEPARATE_PROFILES /
OSCILLATOR_HEAT_MODE_BOUNCE_REFERENCE_ORACLES /
CONSERVATION_CONVERGENCE_EVENT_AND_COUPLING_EVIDENCE /
FMI_ME_CS_SE_TIME_OWNERSHIP /
P09_D03_CONTROL_P09_D04_DELIVERY_SEPARATE /
A_B_C_BACKENDS_SELECTED_PER_WORKLOAD /
80_FUTURE_TESTS / Q0_TO_Q11 / 30_INVARIANTS /
UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED
```

> **Closing principle:** Physical equations define the model; numerical methods approximate its evolution; measurements test its correspondence with nature. Each claim carries its own source, time, uncertainty and error evidence. A credible simulator makes those boundaries measurable.