# LFR-C02 | Source-Lineage Architecture for a Gerbil Foundation Reserve

**Date:** 12 November 2026  
**Status:** `REVISED_CONCEPT_PROPOSED / SOURCE_LINEAGE_EXPLICIT / IMPLEMENTATION_NOT_AUTHORIZED`  
**Supersedes:** LFR-C01 **as a concept proposal only**; C01 remains a historical record.  
**Research nucleus:** Alien-Works, upstream `borodust/alien-works@86381f8bce44a396347cbe8de70349be5b43ece1`.  
**Evidence baseline:** LFR-SR-01; existing Alien-Works audits 02/03/05; Uncharted public-evidence review; Guix/Guile source review; six separately published Gerbil source reviews.  
**Implementation language contemplated:** **Gerbil Scheme**. All proposed foundation semantics and specialized *new* editions are Gerbil-native.  
**Publication/implementation:** Published with owner approval on 12 November 2026 under `docs/concept/` in `vietcong73/alien-works` (`audit/lisp-foundation-reserve`). Documentation publication does **not** authorize code creation, source changes, deployment, or copy/port of source.

## 0. The correction to C01

C01 correctly separated preserved originals, specialist editions and a general foundation, but left *why each source matters* under-specified. It also proposed a mandatory Gerbil-to-Guile/Guix bridge. This latter proposal is explicitly withdrawn.

**Guix, Guile and Shepherd are source-evidence subjects, not runtime requirements.** Study the Guile implementations of staged computation, service extensions and process supervision; derive independently tested Gerbil mechanisms where they are generalizable. If a future Gerbil application happens to run under Guix, that is an optional deployment choice, not a foundation dependency. Do **not** port Guix itself, embed Guile, create a Guile bridge by default, or represent all runtime work as a Guix derivation.

This document does **not** ratify the separately isolated Alien-Works × Uncharted concept 04. That historical hypothesis is a source of questions and comparison, not a prior design decision. It also does **not** declare the original source code executable or safe for reuse.

## 1. Design thesis

Create a **source-preservation research collection** organized around the demonstrated strong area of each Lisp/Scheme system; from it derive independently versioned, narrowly scoped **Gerbil specialist reserves**; below them offer one **minimal, highest-abstraction Gerbil Meta-Foundation**. A future application can import selected specialist reserves and the foundation, or adapt/fork a specific version of the foundation.

“Highest abstraction” refers to **cross-domain semantic contracts** (identity, schema, effects, authorization, state, lifetime and evidence), *not* universal ownership of every scheduler, renderer, model, build graph or process. The default foundation must have no SDL, GPU, Guix, Guile, Shepherd, proprietary VM, LLM provider, cloud or database dependency.

**Source lineage is mandatory:** for each preserved system, specify (a) what problem it solved, (b) what implementation proves it, (c) why it is useful, (d) which source defects/limits matter, (e) what the Gerbil edition would solve differently, and (f) what tests could falsify the improvement claim. An example application alone is insufficient evidence of a general abstraction.

### Architectural dependency directions

```text
RESEARCH / PRESERVATION PLANE (not executable dependencies)
    Alien-Works (primary)   Guix/Guile/Shepherd   Glow   Uncharted public evidence
    Coding Harness         AutoContext           Personal AI CLI
    Chess AI               GloUI                 Trial/CEPL/Varjo comparators
                  | source pins + audits + mechanisms + negative results
                  v
            LINEAGE / QUALIFICATION REGISTER
                  | independently chosen design mechanisms
                  v
         NEW GERBIL SPECIALIST RESERVES (optional, separately tested)
    Native/Graphics   Semantic/DSL     Agent/Governance   Retrieval/Memory
    Search/Simulation Systems/Processes  Interface/Interop
                  \       |                /
                   \ contracts, never implementation imports
                    v
                GERBIL META-FOUNDATION
          Small K0 contract kernel + opt-in K1 mechanisms
                    ^
                    | used by selected editions and applications
                    |
             FUTURE GERBIL APPLICATIONS
```

The arrow through the lineage register is **research influence**, not runtime linking. The actual code-dependency direction is **application -> selected editions -> foundation**, with the option for an application to depend on `foundation` alone. The foundation must not depend on any edition.

## 2. Three separate asset types and their rules

1. **Source Vault / Preserved Specimens.** Immutable upstream commits, release archives or pinned links, checksums, authors, dates, licenses, dependency graphs, file lists, code-review findings. Mirror full source only where lawful, useful and feasible; otherwise preserve verifiable Git references/bundles and evidence. This layer is historical, not an API dependency.
2. **Gerbil Specialist Reserves.** New Gerbil source modules specializing in a given area, with explicit contract, provenance lineage, independent build/tests and known limits. These are *not* represented as byte-for-byte ports unless that is proved and licensed. Their APIs are not required by every application.
3. **Gerbil Meta-Foundation.** General contracts and minimal reference machinery. An unqualified source specimen cannot register a privileged executable capability merely by appearing in the vault.

**Uncharted exception:** preserve accessible public presentations, citations and verifiable architectural descriptions only. Do not copy proprietary engine scripts, compiled bytecode or unreleased Naughty Dog tools.

## 3. Cross-source lineage matrix: which problem, from whom, and what changes

| ID / source | Problem the original tackles | Demonstrated useful mechanism | Original weakness / unsolved boundary | What a **new Gerbil edition** should resolve | Evidence required |
|---|---|---|---|---|---|
| **P01 / Alien-Works** | Express complex native graphics/game subsystems from Common Lisp instead of hand-coding C/C++ glue everywhere | Macros, CLOS builders, `defmaterial`, Varjo shader DSL, CFFI/IFFI facades, `unwind-protect` scopes, renderer queues | Monolithic dependency graph; unresolved `alien-works-foundation` closure; source defects in allocator, typed memory, streams, codec, cleanup and failure handling | Retain DSL/metaprogramming *patterns* while independently expressing checked resource lifetimes, generated bindings contracts and isolated optional native backend profiles in Gerbil | ABI/layout tests; failure injection; scope/ownership invariants; independent profile builds |
| **P02 / GNU Guix + Guile** | Describe and reproducibly construct packages, environments and services rather than manage mutable ad hoc installations | Scheme records, G-expressions, host/build staging, explicit dependency lowering, derivations, service-extension graph, generations | Staging is environment-specific; build/store graph is not a general runtime scheduler; activation and external mutable state are not an atomic transaction; complexity of debugging staging | Study/implement a **generic Gerbil description/validation/dependency graph** with explicit separation of description vs effect. No Guile interpreter or Guix protocol required | DAG property tests, stage purity/closure checks, rejected cycles, replayed effect receipts, declaration/observation divergence tests |
| **P02b / GNU Shepherd** | Reliably manage processes and service dependencies, including failure and restart | Service provision/requirement, start/stop, process monitoring, respawn policy/limit, lifecycle states | A system/user daemon manager is not an in-process actor scheduler; restart does not erase prior external effects; service semantics are version-specific | Study an **optional Gerbil local process-supervision edition**, distinguishing actor scheduling from OS processes and capping restart storms | Controlled crash tests, backoff/limit tests, PID/reaping tests, cancellation/child handling, service dependency ordering |
| **P03 / Glow** | Compile a declarative multiparty interaction into participant-specific behavior and runtime representations | Multi-pass compiler, IR transformations, checkpoint/projection, test vectors | Checkpoint-liveness backward analysis has a source-grounded missing fixed-point concern; some runtime/consensus claims unqualified | Independently implement inspectable Gerbil pass/IR contracts and correct graph/fixed-point analyses for relevant DSLs | Property tests and graph counterexamples, pass invariants, mutation tests, interpreter/compiled differential results |
| **P03b / Uncharted public materials** | Let domain authors define game behavior, state, timing and scene/encounter orchestration without hand-editing low-level C++ | PLT Scheme data-definition/state scripts, state transitions, stand-alone directors, waits/signals, bounded native hooks | Domain-specific and proprietary; native lifecycle and timing hazards; no production source to port | Borrow *semantics as research questions*: state/event/director models, bounded domain authority and time ordering, without copying implementation or mandating a VM | Written formal semantics, deterministic state-machine conformance tests and independent legal implementation |
| **P04 / Coding Harness** | Give LLMs controlled code-editing/tool capabilities with human review and post-edit checks | Gerbil tool registry, per-language profiles, diff display, stale-base matching, syntax diagnostics | Root path traversal, auto-approval ≠ sandbox, edits written before checks, historical-string exit status, timeout limits | Canonical project-root boundaries, typed authority decisions, staged edit/check/commit with rollback rules, structured outcomes, subprocess deadlines | Traversal/symlink tests; refusal tests; crash/partial-edit recovery; controlled command/process trees |
| **P05 / AutoContext** | Retrieve selected corpus passages for an LLM without external embedding infrastructure | BM25, TF-IDF sparse vectors, cosine, chunk IDs, union/dedup | Negative BM25 IDF filtered as irrelevant, zero-average-length, no total prompt budget, naïve result fusion | Numerical stable retrieval, empty corpus handling, bounded context and source-provenance metadata | Reference scoring vectors, adversarial corpora, top-k correctness, corpus edge cases, token budgets |
| **P05b / Personal AI CLI** | Preserve and reuse past useful model answers in a simple terminal assistant | Small Gerbil HTTP/JSON client and s-expression cache | Read errors silently become empty cache and can later overwrite data; API errors are treated as answers; weak substring retrieval | Typed `Result` vs `Error`; atomic persistence, integrity/recovery; store origin and trust class, not just answer text | Corrupt-cache/restart tests; API failure tests; filesystem crash tests; provenance verification |
| **P06 / Chess AI** | Demonstrate deterministic game-state representation and bounded adversarial search | Make/unmake, Zobrist hashes, negamax/alpha-beta, quiescence, perft fixtures | In-check quiescence leaf handling, invalid FEN castling rights, weak perft and undo assertions | Explicit legal-state invariants and general reversible search/rollback protocols, independent of chess heuristics | Perft edge positions, differential search results, hash recomputation, make/unmake identity tests |
| **P07 / GloUI** | Couple Gerbil backend operations with a UI/JS presentation layer | Capability routing, HTTP/JS interop, asynchronous task status | ERC20 decimals assumptions, error state consistency, shared JS prototype state, transaction-route authorization boundary | Typed UI/backend messages, strict per-entity state isolation, error-result typing, permissions on effectful endpoints | Serialization/isolation tests, invalid input tests, permission checks, end-to-end UI contract tests |

**Source distinction:** Several findings are *static audit conclusions*, not reproduced runtime failures. The source review must never silently promote `potential defect` to `proven exploit`, `architecture pattern` to `correct implementation`, or `reference mechanism` to `license to copy`.

## 4. Priority source-preservation editions

The original-source snapshot and the prospective Gerbil edition must have distinct IDs and release histories. The priorities below indicate **research value**, not permission to implement or copy existing licensed code.

### P01 / Metaprogramming + Native Capabilities — Alien-Works is the research center

- **What to learn:** the progression from macros/CLOS builders to composable domain languages (`defmaterial`, fragment shader declarations); FFI-wrapper ergonomics; explicit lexical resource scopes; render queues, native handles and callback lifetimes.
- **Problem it solves:** an application author can express high-level graphics/native domain intent rather than build raw ABI plumbing at every call site.
- **Problems to solve *beyond* Alien-Works:** excessive coupling, undocumented foreign ownership, known allocator/stream/codec defects, incomplete build/dependency evidence, native calls leaking through abstractions with insufficient checkable contracts.
- **Gerbil reserve target:** *not* a port of Common Lisp CLOS/macros. A Gerbil-idiomatic declarative capability/DSL authoring specimen plus optional native adapters, with explicit init/use/shutdown and ABI qualification. Start pure/mock before native; no mandatory renderer in the foundation.
- **Comparator lessons:** Trial makes resource allocation/deallocation and event loops explicit; CEPL/Varjo illustrate a different Lisp-to-GPU compilation boundary; Gerbil-SDL2 shows Gambit C-FFI possibilities but labels itself experimental.
- **Must not claim:** correct cross-language ABI, shader equivalence, or source portability from lexical resemblance.

### P02 / Declarative Systems, Staging and Process Supervision — Guix, Guile, Shepherd

- **What to learn from Guix/Guile:** high-level description records; G-expressions as *staged code carrying dependency references*; lowering to derivations; dependency DAG construction; service-extension folding; versioned generations and provenance.
- **Problem Guix solves:** reliable specification and construction of a package/system/environment graph rather than manual assembly from unknown mutable inputs.
- **What to learn from Shepherd:** services have dependencies, lifecycle state, restart/respawn policies and a monitored OS process boundary.
- **Problem Shepherd solves:** OS-level services need managed start/stop/recovery rather than orphaned shell processes and unbounded restart loops.
- **Limitations not to inherit:** build graphs are not agent task graphs; actor mailboxes are not PID/service managers; generations do not reverse effects on mutable databases; activation is effectful; Guix is a real package manager, not just a planning library.
- **Gerbil reserve target:** an independent Gerbil **description graph and staged-effect model**, plus an optional **process-supervision profile**. The general concepts are learned, tested and re-expressed in Gerbil. **NO required Gerbil↔Guile/Guix integration and NO runtime dependency on GNU Guix, Guile or Shepherd.**
- **Deployment clarification:** packaging any future application for an actual Guix machine is a separate system-integration project. It does not alter foundation semantics.

### P03 / Semantic Compilation and Temporal State — Glow and Uncharted evidence

- **What to learn from Glow:** compiler passes, IR normalization, checkpoint analysis, test vectors and participant-specific projection.
- **Problem Glow solves:** move from one declarative interaction to multiple executable perspectives with explicit transformations.
- **What to learn from Uncharted:** Scheme-authored domain definitions and bounded state/director semantics for temporal coordination.
- **Problem Uncharted solves:** make domain sequences and multi-object behavior authorable without giving every script direct control over the native engine.
- **Limitations:** Glow's source-grounded liveness fixed-point concern; native/DSL semantic mismatches; Uncharted's unavailable proprietary source and game-specific VM.
- **Gerbil reserve target:** inspectable pass contracts and finite-state/event semantics, with a *testable* fixed-point layer when graph analysis demands it. No custom bytecode VM unless a benchmarked requirement arises.

### P04 / Agent Authority and Controlled Execution — Coding Harness

- **What to learn:** tool registration, explicit user review, stale-base checking, syntax diagnostics and separated model/tool loops.
- **Problem it solves:** LLM output becomes a reviewable, bounded tool workflow rather than arbitrary opaque edits.
- **Limitations to correct:** project-root escape, edit-then-check partial failure, approval conflation with sandboxing, unstructured exit states, soft time limits.
- **Gerbil reserve target:** an optional Gerbil execution-governance package where policy is a real gate, every outcome is typed, and file/process effects are scoped and logged. It must not be the authority source of every non-agent application.

### P05 / Retrieval, Persistence and Evidence — AutoContext + Personal AI CLI

- **What to learn:** explainable retrieval and compact persistence in Gerbil; score/identity separation; explicit opt-in model API use.
- **Problem it solves:** reintroduce relevant context without permanent unbounded conversation history.
- **Limitations to correct:** BM25 pathological IDF/zero-length cases, naïve dedup and budget, silent cache loss, saving API failures as memory, missing provenance.
- **Gerbil reserve target:** independent retrieval algorithms, bounded context assembler and integrity-aware memory. An LLM should remain an *optional* consumer, not a foundation dependency.

### P06 / Reversible State and Search — Chess AI

- **What to learn:** explicit mutable state, make/unmake, hash identity, bounded search, test fixtures.
- **Problem it solves:** efficient exploration of alternative states without copying entire worlds at every node.
- **Limitations to correct:** legality/invariant gaps and false-positive test contracts.
- **Gerbil reserve target:** domain-neutral reversible-state interfaces and bounded exploration. Chess remains a validation corpus, not an assumption that every controller is a game tree.

### P07 / UI and Interoperability — GloUI (optional)

- **What to learn:** Gerbil backend interfacing with embedded JavaScript, HTTP routes, request/process/status flows.
- **Problem it solves:** present asynchronous backend capabilities through a usable interface.
- **Limitations to correct:** shared frontend state and missing/weak effect boundaries; token-decimal and status consistency findings.
- **Gerbil reserve target:** schema-versioned UI/backend envelopes and per-entity isolation, with effectful operations denied by default. Not required in Meta-Foundation.

## 5. Meta-Foundation: highest abstraction, smallest runtime obligation

**K0 — Mandatory minimal Gerbil contracts** (names provisional):

| Contract | Source lineage | Cross-domain problem solved | What it must never claim automatically |
|---|---|---|---|
| `IdentityRef` / `VersionRef` | Guix generations; pinned source register; Glow agreements | Distinguish exact source, configuration, runtime instance and artifact identity | Hash identity alone guarantees deterministic behavior |
| `Schema` / `ValidationResult` | Guix records; Glow IR; Coding Harness tools | Reject invalid data at a named boundary | Validation automatically limits side effects |
| `EffectDescriptor` | Guix build/activation boundary; Coding Harness tools; Alien-Works native scopes | Make pure/readonly/mutation/process/native/network actions explicitly classifiable | A declaration itself is OS sandboxing |
| `AuthorityDecision` | Coding Harness approvals; Uncharted bounded domain capabilities | Separate desire/request from permission and execution | LLM prompt is permission enforcement |
| `ResourceOwner` / `LeaseState` | Alien-Works `unwind-protect`; Trial resources | Model creation/use/release, owner, generation and invalidated handles | GC/finalizer alone establishes deterministic native lifetime |
| `Result` / `Failure` / `Receipt` | Coding Harness, Personal AI CLI, Guix provenance, Guix rollback limits | Prevent errors from becoming valid answers, preserve action outcomes | `FAILED` means side effects were undone |

The K0 contract kernel can represent a request, a validation decision, an effect class, an owner, and a result. It **does not have to start threads, launch processes, allocate GPU resources, run models or manage an OS**. Avoid a general `execute-anything` singleton.

**K1 — Opt-in reusable mechanisms**, each independently warranted by two different domains: `dependency-graph`, `state-transition`, `event-scheduling`, `bounded-executor`, `observability`, `transaction/compensation`, `persistence` and `structured-protocol`. Promotion into K1 requires evidence of cross-domain usefulness, not just resemblance of names.

**Pxx — Specialist implementations:** domain packages adopt K0/K1 contracts where appropriate, but carry their own algorithms, native bindings, state and dependencies. P01 must not import P05; P05 must not import P01; P02 need not import P04. An application chooses only the packages it needs.

### Example common effect lifecycle (conceptual, not API spec)

```text
DECLARE -> VALIDATE -> RESOLVE -> AUTHORIZE -> PREPARE -> EXECUTE
                                                          |
                                       +------------------+--------------+
                                       |                  |              |
                                    COMPLETED           FAILED        CANCELLED
                                       |                  |              |
                                       +----------- OBSERVE / RECEIPT ---+
                                                      |
                                                 RECONCILE / RELEASE
```

- Description and preparation must be distinguishable from effectful activation.
- Explicit side effects have actor/authority, scope, version, expiry and outcomes.
- A failed action may have partially affected external state. Recovery/compensation requires an explicit contract.
- A renderer task, background OS service and agent tool call may share *result/effect descriptors* but **not** an identical execution or scheduling model.

## 6. Mechanism-to-problem crosswalk: examples of real reuse, not analogy

| A future system's challenge | Source lessons selected | What the foundation standardizes | What remains specialist |
|---|---|---|---|
| Native device or rendering context must be released after failure | Alien-Works scopes; Trial resource state | `ResourceOwner`, release outcomes, invalidation receipts | FFI ABI, thread affinity, actual native destructor |
| Multi-stage workflow depends on prior steps and can fail midway | Guix service DAG; Glow passes; Coding Harness checks | Typed dependency/precondition and receipts | Compiler lowering, agent check, environment action |
| One model requests access to files and command execution | Coding Harness and Goblins as capability-security comparator | Effect class, authority decision, request scope | OS sandbox, specific tool executor, model vendor |
| Need deterministic replay of a state decision | Chess make/unmake; Glow checkpoints; Guix provenance | State/version/evidence identity | Domain transitions and equivalence/oracle definition |
| A long-running process crashes | Shepherd supervision | Failure and lifecycle schema | PID management, restart implementation, OS integrations |
| A retrieved answer is stale/corrupt/untrusted | Personal AI cache; AutoContext | Typed result, data provenance, bounded references | Ranking/scoring, storage implementation, prompt format |
| Two UI components share mutable state accidentally | GloUI observed JS prototype defect | Per-instance identity, schema and error envelopes | JS frontend ownership and reactive rendering |

The correct aspiration is not one implementation that runs every task. It is **consistent boundaries between different kinds of task**.

## 7. Conformance and falsification tests

The concept is useful only if it can fail tests. These are *future acceptance questions*, not implementation authorization.

- **T00 Provenance:** every preserved specimen has exact origin/ref, source hash, license, mirror policy, last verified date and audit status. No proprietary code mirroring.
- **T01 Pure-foundation build:** K0 builds on Gerbil without Guix, Guile, Shepherd, SDL, LLM providers, GPU, database or network access.
- **T02 Cross-domain reuse:** the *same* K0 identity/effect/result contracts support at least two unrelated mock profiles (for example resource lifetime and agent file operation), with no source edits to K0.
- **T03 Authority:** unauthorized action is rejected before any effect; replaying a permission decision against a different input/config is rejected; denied paths are testable.
- **T04 Resource lifetime:** acquisition failure, double release, stale handle, cancellation, thread-affinity mismatch and destructors are tested with mock handles before native backends.
- **T05 Declarative systems:** dependency cycles are rejected; staging/activation semantics are separated; a failed activation yields a non-success receipt rather than an invented rollback.
- **T06 Process behavior:** controlled process exits, restart caps, descendant handling and deadlines have observable outcomes; no assumption that a user daemon is PID 1.
- **T07 Compiler/state:** graph fixed-point tests cover joins/loops; state transitions have before/after identities and event ordering.
- **T08 Retrieval:** empty corpus, all-stopword corpus, negative/zero IDF, rank/score fixtures, bounded context and corrupted persistent stores are handled explicitly.
- **T09 Search:** legal-state guards, make/unmake identity, hash recomputation and regression fixtures return failing exit statuses on mismatches.
- **T10 UI:** per-instance state never crosses another instance; high-impact operations require scoped authorization.
- **T11 Negative dependency test:** a program importing K0 must not transitively load graphics, Guile, Guix, Shepherd, crypto API, LLM provider or package manager dependencies.
- **T12 License/reuse:** origin and derivative use are recorded; inclusion is blocked pending incompatible license terms or missing provenance.

Any use of “safe,” “reproducible,” “portable,” “faster,” “sandboxed” or “qualified” must be scoped to executed evidence and target version/environment. No performance claims without benchmarks.

## 8. Preservation topology and possible workspace layout (not generated code)

```text
lisp-foundation-works-reserved/              # prospective separate repo; not created
  README.md
  research/
    lineage-register.md                    # source -> problem -> pattern -> defects -> validation
    evidence-index.md                      # links back to original source audits
    negative-evidence.md
  preservation/
    specimens.lock                         # source SHA, license, origin, mirror/URL
    native-metaprogramming/                # Alien-Works primary
    declarative-systems/                  # Guix/Guile and Shepherd references
    semantic-dsl/                         # Glow; Uncharted docs only
    agent-governance/
    retrieval-memory/
    search-simulation/
    interface-interop/
  foundation/                               # Gerbil K0, later K1
    core/
    optional/
  editions/                                 # standalone Gerbil source, independent versioning
    native-graphics/
    declarative-systems-processes/
    semantic-dsl/
    agent-governance/
    retrieval-memory/
    reversible-search/
    interface-interop/
  qualification/
    contracts/
    counterexamples/
    fixtures/
    executed-evidence/
```

The existing `vietcong73/alien-works` remains the **research hub** and preservation fork; do not overwrite its historical Common Lisp source with Gerbil experiments. The future reserve repository, if approved, owns *new* Gerbil code. The vault may preserve references rather than copying code wholesale. Creation or naming of a new reserve repository, or any implementation push, requires separate explicit direction; publication of this concept document has been expressly approved.

## 9. Defect lineage: what gets corrected, and what is not inherited

A defect-remediation claim must have: `{source_ref, claim, minimal counterexample, Gerbil requirement, regression_test, validation_status}`. For example:

```text
AW native scope failure -> acquire partial allocation risk
   -> Gerbil lease must record ownership from acquisition
   -> fault-inject constructor and assert exactly-once release

Coding Harness historical status parsing -> previous failure contaminates final exit
   -> typed final result independent from prior message text
   -> fail -> fix -> pass test must report final PASS with trace of earlier failure

Personal AI cache load exception -> empty cache -> overwrite on close
   -> integrity-aware load and atomic write with quarantine on corruption
   -> corrupt-file/restart test must not erase original bytes

Glow liveness join path -> visited-node without repropagation
   -> worklist/fixed-point algorithm contract
   -> uneven-join graph test must converge to expected live set

Guix generation switch != rollback of external mutation
   -> effect receipt and explicit compensation eligibility
   -> simulated partial external write must stay recorded after failed activation
```

These are **design requirements**, not claims of completed repairs. Historical defects remain in source audit ledgers, unchanged.

## 10. Licensing, qualification and negative-transfer rules

- Alien-Works root source is MIT at the audited revision, but its dependencies/embedded generated bindings can carry other obligations. Guix code is generally GPL-3-or-later; reviewed Coding Harness states AGPL-3.0. Verify per-file/submodule terms before any copied/derived code is distributed. Cite other source licenses as individually verified, not guessed.
- Source lineage can be recorded without copying implementation. Conceptual reuse is not the same as source-code derivation; clean-room or independent implementations may still need legal evaluation in edge cases.
- Do not import Naughty Dog's proprietary source, even if public talks show illustrative snippets. No proprietary VM reconstruction as if it were an open module.
- Gerbil's Gambit C-interface is not CFFI. CL CLOS and compiler macros are not assumed interchangeable with Gerbil syntax objects/macros. Guile G-expressions are not automatically Gerbil syntax or a portable build runtime.
- An actor/message abstraction is **not** equivalent to a security permission mechanism. OS process management is **not** actor scheduling. A Guix build derivation is **not** an application plan. SDL polling is **not** a director DSL.
- Preserve existing six Gerbil source reviews; leave Gerbil MCP, Gerbil Shell unreviewed due inaccessible code; exclude microGPT by explicit owner decision.

## 11. Decisions introduced by C02 versus C01

| Issue | C01 | C02 |
|---|---|---|
| Source contribution | Lists strengths and target edition | **Adds source-to-problem-to-defect-to-test lineage for every edition** |
| Guix/Guile | Mandatory Gerbil-to-Guile adapter suggested | **Removed.** Guix/Guile/Shepherd are research specimens; Gerbil-native independent abstractions |
| Foundation scope | Broad `PlanSpec`/`Execute` common interfaces | K0 small universal contracts; K1 orchestration only on demonstrated cross-domain demand |
| Preserved source versus Gerbil code | Three classes distinguished | Adds explicit legal/technical conversion gate and provenance requirement |
| Concept 04 | Declared isolated | Remains isolated and unselected; no automatic hybrid architecture import |
| Validation | General gates | Adds falsifiable per-source defect regression and cross-edition tests |
| Implementation | Not started | Still not started; C02 itself grants no remote/build authority |

## 12. Open design decisions, not hidden assumptions

1. **Scope of source mirroring:** full bundles for small permissively licensed codebases, SHA-locked references for large/copyleft sources, or a mixed preservation policy? Decide based on actual file and licensing review.
2. **Foundation compatibility:** do applications import versioned K0 directly or one selectable profile facade? Avoid a premature global plugin registry until real consumers exist.
3. **First cross-domain proof:** which two unrelated mock editions should test K0 without attracting large native dependencies? Use real failure cases, not demo-only success paths.
4. **Deployment scope:** does a future application need a native Guix package or Shepherd unit? Such packaging remains external to the general Gerbil kernel.
5. **Edition maturity:** which specialized reserves should be *executable* versus *historical/algorithm specimens*? No assumed parity with original native systems.
6. **Licensing:** select a license only after deciding whether any external source code is copied or independently reimplemented.

## 13. Concept verdict

`LFR_C02_PROPOSED / ALIEN_WORKS_PRIMARY_SOURCE / MECHANISM_LINEAGE_EXPLICIT / GERBIL_NATIVE_ONLY / GUIX_GUILE_SHEPHERD_RESEARCH_ONLY / FOUNDATION_MINIMAL / SPECIALIST_EDITIONS_ISOLATED / NO_IMPLEMENTATION_OR_PUSH`

**Thesis:** Preserve strong source systems without inheriting their unresolved defects, extract the *mechanisms* that made each valuable, requalify each Gerbil specialist reserve independently, and make the Meta-Foundation shared only where its contract meaning is genuinely invariant across domains.

## Source anchors

- Alien-Works immutable baseline: https://github.com/borodust/alien-works/tree/86381f8bce44a396347cbe8de70349be5b43ece1
- Published LFR-SR-01 evidence review: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/systematic/alien-works-centered-systematic-evidence-review-2026-11-12.md
- Alien-Works source audit: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/alien-works-common-lisp-source-code-audit-review.md
- Guix/Guile source audit: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gnu-guix-system-and-package-manager-scheme-source-review.md
- Gerbil reviews: https://github.com/vietcong73/alien-works/tree/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects
- Guix G-expressions manual: https://guix.gnu.org/manual/en/html_node/G_002dExpressions.html
- Guix service composition: https://guix.gnu.org/manual/en/html_node/Service-Composition.html
- GNU Shepherd manual: https://www.gnu.org/software/shepherd/manual/html_node/
- Gerbil FFI guide: https://gerbil.scheme.org/guide/ffi.html
- Gerbil actors introduction: https://gerbil.scheme.org/guide/intro.html
- Glow source: https://github.com/Glow-Lang/glow/tree/47753e6777ebe933be319f117c00401745eaeb2f
- Coding Harness, AutoContext, Chess AI and Personal AI CLI: https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code
- GloUI source: https://github.com/Glow-Lang/gloui/tree/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1
- Uncharted public review: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/naughty-dog-uncharted-lisp-family-review.md