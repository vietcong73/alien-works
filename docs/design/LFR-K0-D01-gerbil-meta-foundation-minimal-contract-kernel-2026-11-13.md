# LFR-K0-D01 | Gerbil Meta-Foundation: Minimal Contract Kernel

**Document class:** Source-grounded, cross-domain detailed design proposal  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve, LFR-C02  
**Target:** Independent Gerbil Scheme K0 contract package  
**Status:** `DESIGN_PROPOSED / CROSS_SOURCE_LINEAGE_MAPPED / GERBIL_DOCS_CROSS_CHECKED / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved design document publication to `docs/design/` on `audit/lisp-foundation-reserve`. This does not authorize source porting, implementation, installation, native execution, or deployment.  
**Scope:** K0 only. K1 optional mechanisms, all P01–P07 implementations, other repositories and preserved originals remain unchanged.

> **Thesis:** K0 standardizes the *meaning of boundaries* across distinct systems without pretending to be the mechanism that crosses them. Its six small families describe identity/version, schemas/validation, effects, authority decisions, resource ownership/lifetime, and results/failures/receipts. An application may use K0 alone; each specialist may implement its own substantive mechanics.

## 0. Research correction: all source lineages count, not just seven project documents

This design is assessed against **two original game-oriented architecture subjects** and their associated research corpus, **all seven specialist reserve proposals**, and the **official Gerbil/Gambit language documentation**. Terminological precision matters: *Alien-Works is a Common Lisp game-development framework, not a named released game*; *Uncharted 2: Among Thieves is a game, but its proprietary production code is not available for this project*. Neither becomes a Gerbil runtime dependency.

### 0.1 The two initial sources

| Initial source | What is inspectable | Distinct architectural contribution | Boundary K0 must preserve |
|---|---|---|---|
| **Alien-Works**, `borodust/alien-works@86381f8bce44a396347cbe8de70349be5b43ece1` | Pinned Common Lisp files and bounded source audits | Lisp host sovereignty, CLOS/macros, native integrations, explicit lifetime scopes, callback identity, material/shader descriptions | Host/native authority and ownership do not follow automatically from a macro or lexical wrapper; no mandatory GPU/FFI/library graph |
| **Naughty Dog / Uncharted 2**, public GDC 2008–2009 papers/slides and previously published architecture review | Primary presentation material, not internal game-engine implementation | PLT Scheme data/code authoring, state scripts, directors, event/time semantics, restricted bridge to native functionality | A data script does not acquire native authority; a state director is not a renderer queue; no proprietary code copied, no native VM inferred |

**Comparison result:** the shared contribution suitable for K0 is *boundary vocabulary*, not a fusion of a Common Lisp native host with a proprietary Scheme VM. This preserves the spirit of Lisp-centered authoring without automatically approving the previously isolated Alien-Works × Uncharted architecture concept 04.

### 0.2 Included supporting projects, references and evidence tiers

| Corpus | Evidence type | Study focus | Eligible K0 transfer; exclusions |
|---|---|---|---|
| **Trial** (`Shirakumo/trial@5629b23d...`) | Inspectable Common Lisp comparator | Allocate/deallocate, resource dependency/context | Owner/lease vocabulary; no CL/CLOS or GL dependency |
| **CEPL** (`cbaggers/cepl@88e9d005...`) | Source/architectural comparator | REPL-facing GPU lifecycle, foreign state | Effect and resource boundary only; no OpenGL runtime |
| **Varjo** (`cbaggers/varjo@68132f72...`) | Architectural/code lineage comparator | Lisp domain syntax and shader lowering | Descriptor-versus-execution distinction; no shader compiler |
| **Gerbil SDL2**, `nymacro/gerbil-sdl2@ed03678d...` | Experimental source specimen | Gambit/C native handle management | Foreign lifetime negative fixtures; not qualified ownership authority |
| **Glow** (`Glow-Lang/glow@47753e67...`) | Pinned Gerbil code review | Multi-pass semantics, normalized IR, participant projection, fixed-point analysis | Schema/version/validation vocabulary; no compiler or consensus |
| **GloUI** (`Glow-Lang/gloui@bac31fab...`) | Pinned Gerbil source, reduced JS reproduction | Typed HTTP presentation, async status, generated JS entities | Per-instance identity, typed error/result; no HTTP/JS UI stack |
| **Coding Harness** (`mark-watson/gerbil_scheme_book@1335cb...`) | Pinned Gerbil source review | Tool registry, approvals, effect boundary, command results | Declarative effect + authority decision; no agent or shell executor |
| **AutoContext** (same Gerbil book pin) | Pinned Gerbil source review | Scored retrieval, corpus references, evidence | Source identity, failure/uncertainty; no retriever or embedding |
| **Personal AI Assistant CLI** (same pin) | Pinned Gerbil source review | Cached memory, external API and error ambiguity | Typed failure and provenance; no model API or database |
| **Chess AI** (same pin) | Pinned Gerbil source review | State mutation/undo, history-sensitive identity, search receipts | Qualified state/version references; no search tree |
| **GNU Guix / Guile**, `guix-mirror/guix@18f545c...` | Inspectable Scheme source & docs | Declarative package graph, staging, generations, time-machine | Version/provenance and declared-versus-actual effect; no Guix/Guile bridge |
| **GNU Shepherd** | Source review and docs | Service lifecycle, restarts, supervision | Lifecycle outcome vocabulary; no daemon or scheduler |
| **Spritely Goblins** | Capability-security comparator, not target runtime | Authority-bearing references and message boundaries | Negative lesson: claims of capabilities require enforcement; no actor runtime |
| **Gambit Scheme C interface** | Language/runtime reference | Foreign calls, memory/threads constraints | Prevent confusing Gerbil contracts with C ABI enforcement |
| **Gerbil Scheme official docs** | Language normative/reference guidance, some pages incomplete | Modules, structs/classes, macros, interfaces, errors, tests, FFI, build | Check *feasibility and available primitives*, not claim tests passed |
| **Other contextual systems**: Chickadee, Raylib-Gambit | Prior bounded comparison only | Alternate Scheme/native game tooling | Background questions only; no direct K0 implementation dependence |

**Explicitly not admitted:** inaccessible Gerbil MCP or Gerbil Shell source as audited proof; previously removed microGPT; proprietary Naughty Dog code; unsourced claims about exact implementation of an unavailable game VM. Reference materials and source reviews are observations, **not successful K0 qualification**.

### 0.3 Primary anchors for independently repeatable inspection

- Alien-Works root: https://github.com/borodust/alien-works/tree/86381f8bce44a396347cbe8de70349be5b43ece1
- Alien-Works host/renderer/DSL: https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/engine.lisp and https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp
- Naughty Dog original GDC 2008 data compilation: https://media.gdcvault.com/gdc08/slides/S6636i1.pdf
- Gregory GDC 2009 Uncharted state-based scripting: https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf
- Naughty Dog evidence review: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/naughty-dog-uncharted-lisp-family-review.md
- LFR source inventory: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/systematic/alien-works-centered-systematic-evidence-review-2026-11-12.md
- C02 governing concept: https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/concept/LFR-C02-source-lineage-gerbil-reserve-concept-2026-11-12.md
- Specialist references: `docs/design/LFR-P01-D01-native-metaprogramming-detailed-design-2026-11-12.md`, `LFR-P02-D01-declarative-systems-process-supervision-detailed-design-2026-11-12.md`, `LFR-P03-D01-semantic-state-dsl-detailed-design-2026-11-12.md`, `LFR-P04-D01-agent-authority-controlled-execution-detailed-design-2026-11-12.md`, `LFR-P05-D01-retrieval-persistence-evidence-detailed-design-2026-11-12.md`, `LFR-P06-D01-branching-causal-reversibility-design-2026-11-13.md`, and `LFR-P07-D01-ui-interoperability-detailed-design-2026-11-13.md` in the same branch's `docs/design/` directory.

### 0.4 Official Gerbil documentation: a mandatory feasibility ledger

| Gerbil reference | Specific decision constrained | Caution |
|---|---|---|
| [Gerbil Introduction](https://gerbil.scheme.org/guide/intro.html) | Core objects, structs, modules, macros and interfaces | Illustrations are not proofs of K0 behavior |
| [Core expander syntax](https://gerbil.scheme.org/reference/gerbil/prelude/core-expander-syntax.html) | Compile-time/runtime phase separation | Expansion and runtime authority must not be conflated |
| [Prelude macros](https://gerbil.scheme.org/reference/gerbil/prelude/macros.html) | Restricted export/import, `defsyntax` family | K0 need not expose a macro DSL |
| [Interfaces `:std/interface`](https://gerbil.scheme.org/reference/std/interface.html) | Public checked method dispatch vs unchecked `&` calls | Unchecked methods can bypass contracts; do not export as public trust boundary |
| [Errors `:std/error`](https://gerbil.scheme.org/reference/std/errors.html) | Exception taxonomy vs K0 outcome values | Raised exception is not the same as an effect receipt |
| [Standard test library `:std/test`](https://gerbil.scheme.org/reference/std/test.html) | Nonzero assertion-based qualification harness | Tests require actual execution; no paper PASS |
| [FFI guide](https://gerbil.scheme.org/guide/ffi.html) | Foreign backend remains optional; Gambit bridge | FFI is not a sandbox or automatic resource safety |
| [Build tool](https://gerbil.scheme.org/reference/dev/build) | `gerbil.pkg`, isolated Gerbil package build, dependency audit | Tooling varies by version; pin actual toolchain for runtime claims |
| [Expander runtime](https://gerbil.scheme.org/reference/gerbil/expander/) | Cross-check syntax inspection/staging | Some official expander entries are incomplete; unresolved APIs get explicit `NEEDS_PROBE` |
| [Gambit C interface](https://gambitscheme.org/latest/gambit.pdf) | Native execution boundary, not K0 implementation | Toolchain ABI and platform effects need independent qualification |

**Documentation policy:** use an exact Gerbil + Gambit version lock for implementation, with checked compilation probes of selected API surfaces. A web documentation page can change or be incomplete; its availability alone does not authorize an API or qualify behavior. Prefer standard documented constructs for the first K0 slice; keep possible implementation conveniences out of the core contract.

## 1. Cross-domain abstraction extraction

Each proposed K0 construct passes four paper-level screening criteria before entering the design:

1. **Two independent domain witnesses:** the concept has the same semantic problem in at least two unlike sources or specialist mock consumers.
2. **Stable invariant:** the same invariant can be stated without a renderer, tool runner, game engine, model, HTTP server or database.
3. **Small dependency closure:** no domain-specific concrete backend is pulled transitively into K0.
4. **Negative transfer guard:** what the source mechanism *does not* establish is made explicit and falsifiable.

| K0 family | Distinct source / specialist witnesses | Same invariant across domains | Deliberately left out |
|---|---|---|---|
| Identity/version | Guix generation + P06 historical node; GloUI entity + P05 source chunk | Identity, revision and semantic equivalence are distinct relations | Git graph, object store, entity registry, causal history |
| Schema/validation | Glow P03 IR; P07 messages; Guix declaration; P04 tool request | Failed validation must be visible and cannot silently admit a malformed boundary object | Universal DSL, solver, compiler or JSON stack |
| Effect descriptor | Uncharted native hook vs Alien-Works native wrapper; P02 activation vs P04 command | Describing an effect is not performing it | Effect runner, OS process, scheduling, transaction manager |
| Authority decision | P04 approval vs P07 effect endpoint; Uncharted restricted hook semantics | An action requires a separately bound authoritative decision before effect execution | Universal policy evaluator, sandbox, consent UI |
| Resource ownership | Alien-Works scoped foreign handle vs Trial resource; P02 child handle vs P07 task observation | Owner, generation, lease and terminal release must not be confused | Real destructor, PID manager, GPU handles, GC finalizer policy |
| Result/failure/receipt | GloUI transfer worker vs P04 run; Personal AI error vs P05 corruption; P06 undo vs Guix rollback | `FAILED` and `UNKNOWN` do not imply rollback or success; receipts carry evidence scope | Logging daemon, persistence, causal proof engine |

**Non-admitted apparent similarities:** SDL event polling vs Uncharted director; Git branch vs physical alternate timeline; token hash vs exact equality; asynchronous acceptance vs completed external effect; interface method vs enforced permission; macro expansion vs safe untrusted-code execution.

## 2. Architectural position, imports and trust

```text
APPLICATION
  |-- imports K0 contracts directly (allowed)
  |-- imports selected P01..P07 specialist contracts/implementations (allowed)
  `-- imports K1 mechanism only if individually qualified (future)

P01 .. P07 [independent implementations]
  -> may adapt to a stable K0 public contract
  -> do NOT import each other's internal engines through K0

K0 / minimal Gerbil-only definitions and *pure checks*
  identity.ss  version.ss  schema.ss  effects.ss
  authority.ss owner.ss    outcome.ss receipt.ss

No K0 import of K1, P01..P07, original repositories, Guix/Guile,
Shepherd, Common Lisp runtime, graphics, backend, actor, model, database,
network server or UI toolkit.
```

The design does **not** retroactively mandate changing already drafted specialist code. K0 is qualified with *mock consumers first*, and adapters can be added later under separate approval. K0 does not centralize all existing policy into one global registry.

**Operational trust model:** K0 can check that a `Decision` describes a particular request/version/scope; it cannot certify who signed it, whether the holder possesses a real operating-system capability, whether an untrusted process is confined, or whether a handler was bypassed. Those are consumer/driver security obligations.

### 2.1 Source/data/command trust domains

- **Source/audit evidence:** may justify design requirements, never grants runtime permissions.
- **Trusted Gerbil developer macros:** can run at compile time; K0 does not interpret arbitrary Scheme submitted by a user.
- **Untrusted data:** parsed against an allowlisted inert schema; never evaluated as code by K0.
- **Declared effects:** data only; no hidden automatic execution on constructor, validation, serialization or error handling.
- **Backend execution:** beyond K0 and subject to separate policy and isolation checks.
- **Receipts:** scoped observations, not automatically verified claims about the outside world.

## 3. K0 value model and identities

### 3.1 Typed, opaque value envelopes (normative semantics, not exact compiled API)

```text
IdentityRef {
  namespace, kind, local_id, authority_domain, generation_optional
}
VersionRef {
  subject: IdentityRef, schema_revision, semantic_revision,
  artifact_digest_optional, source_lock_optional
}
RefBinding {
  identity: IdentityRef, version: VersionRef,
  equivalence_profile_optional
}
```

- `kind` distinguishes source, entity, runtime instance, artifact, resource, policy or node; new kinds may be namespaced without K0 interpreting their contents.
- `namespace` and `authority_domain` distinguish equal human-readable names in unrelated systems.
- `generation` protects against ABA handle reuse; a generation increment does not imply a wall-clock timestamp.
- `artifact_digest` may help check byte-level identity if its algorithm and canonical encoding are specified; it is not semantic equivalence, provenance authenticity or a safety certificate.
- Version compatibility is a *named, evidence-bearing relation* supplied by a consumer. No implicit rule that newer is compatible, nor one universal semantic-version comparator.
- Imported records with duplicate keys, conflicting versions, invalid Unicode representations or unsupported canonical encodings must be rejected before use.

### 3.2 Exact identity vs observed equivalence

```text
same_ref(A, B)           = checked exact identity/version equality
compatible(A, B, rule)   = result of a named consumer compatibility check
same_observation(A,B,O)  = equivalence under explicitly declared observer O
```

These are different relations. `same_observation` may hold while `same_ref` is false, as in P06 histories with identical state images but distinct lineage, or two distinct UI entities with identical displayed fields.

## 4. Schema / validation contract

```text
SchemaRef { id: IdentityRef, revision: VersionRef, profile, constraints_ref }
ValidationResult =
    VALID(canonical_value, schema_ref, checks_performed)
  | INVALID(violations: nonempty bounded list)
  | UNVERIFIED(reason, missing_evidence)
```

**Pure validation contract:** no IO or backend calls, no arbitrary user-supplied predicate execution, no mutation of caller-owned source records. A constructed `VALID` is meaningful only for the named schema/validator revision, not automatically for a later schema or different semantic domain.

- Strictly define unknown field policy: refuse, preserve opaque extension, or explicitly ignore only under the named schema profile.
- Normalize a *copy*, with source field paths and stable diagnostics; do not silently coerce precise units, resource identities, approval scopes or effect classes.
- Validation must not be treated as an authorization check; a well-formed malicious request remains a well-formed malicious request.
- Round-trip type preservation is a consumer adapter claim, not a K0 assumption about JSON/JavaScript/Protobuf.

## 5. Effect description and authority decisions

### 5.1 Effects are *descriptions*, not generic execution

```text
EffectDescriptor {
  effect_id, requested_operation_ref, effect_class, target_ref,
  scope_ref, argument_commitment, expected_preconditions,
  compensation_class, declared_observation_profile
}
```

`effect_class` uses a minimal *classification lattice* as bounded data: `PURE`, `READ`, `MUTATION`, `PROCESS`, `NATIVE`, `NETWORK`, `UNKNOWN`. It is **not** necessarily a total order of privileges: a read of private data can be more sensitive than a write of noncritical temporary state. A compound effect is explicitly a set of classes and scopes; no implicit promotion by choosing the least risky label.

- `PURE` means the **declared operation** purports no external effects, not that arbitrary code has been proven pure.
- Unknown/unclassified effect defaults to `UNKNOWN` and cannot be silently admitted as harmless.
- Compensation is `NOT_DECLARED`, `POSSIBLE_UNVERIFIED`, `QUALIFIED_CONDITIONAL` or `IRREVERSIBLE`; no implicit undo semantics.

### 5.2 Authority decisions must bind exactly, and never self-authorize

```text
AuthorityDecision {
  decision_id, request_commitment, subject_ref, scope_ref,
  policy_ref, authority_source_ref, expiry_optional,
  action: DENY | ALLOW | REQUIRE_REVIEW | UNKNOWN,
  conditions, evidence_refs
}
```

- `ALLOW` is merely a *decision record*. Actual effectful execution needs a trusted consumer policy gateway, which must check the decision is authentic/admissible for its authority model.
- Only a decision from the correct configured authority source for the exact request, scope, policy/version, and validity window may be considered by a consumer. K0 does not issue security credentials.
- A request commitment is based on a canonical, length-bounded form. Compare original material or trusted authenticated bindings before effects; digest equality alone is not a proof of equivalence or authority.
- Default outcome when authority is absent, expired, changed or unverifiable: `DENY` or `UNKNOWN`, **never ALLOW**.
- A policy declaration or a metaprogrammed rule cannot grant new K0-level trust powers to itself. Upgrade of the trust roots is a separate owner-governed consumer decision.
- The producer of an `AuthorityDecision` must be distinguished from a system that can verify and enforce it. Do not conflate K0 records with P04's policy engine or P06's internal trusted kernel.

## 6. Resource ownership and lease state

```text
ResourceOwner { owner_ref, resource_ref, generation, ownership_kind }
LeaseState = UNACQUIRED | OWNED | BORROWED | TRANSFER_PENDING |
             TRANSFERRED | RELEASE_PENDING | RELEASED |
             RELEASE_FAILED | UNKNOWN
LeaseObservation { owner_ref, lease_state, generation, observed_at_ref,
                   evidence_refs, residual_effects_possible }
```

K0 owns *the representation and checked transitions* for descriptive resource state, **not native resource allocation or release execution**. It does not require GC-based finalization and does not promise deterministic destruction for hostile foreign code.

Proposed checked declarative rules:

- `UNACQUIRED -> OWNED` only after consumer supplies acquisition outcome evidence.
- `OWNED -> TRANSFER_PENDING -> TRANSFERRED` only when a verified target owner accepts the transfer; incomplete transfer returns uncertainty, not two assured owners.
- `OWNED -> RELEASE_PENDING -> RELEASED` only after independently observed release success; an exception or missing native acknowledgment yields `RELEASE_FAILED`/`UNKNOWN`.
- A `BORROWED` holder does not gain destroy authority; stale generations are rejected; duplicate release is typed failure, not assumed idempotence.
- Lease lifetime does not imply that a native callback is quiescent. Callback draining, thread affinity and OS handles remain P01/P02 concerns.

## 7. Results, failures, uncertainty and receipts

```text
Result[T] = OK(value: T, evidence_optional)
          | FAIL(failure: Failure)
          | UNKNOWN(uncertainty: Uncertain)

Failure { code, stage, source_ref, scope_ref, message_safe,
          attempted_effects, partial_effects_possible }
Uncertain { code, last_verified_stage, evidence_gaps,
            residual_effects_possible, permitted_next_checks }
Receipt { receipt_ref, subject_ref, operation_ref, input_version,
          outcome_class, observer_ref, evidence_refs, scope,
          limitations, claim_level }
```

- Distinguish `ACCEPTED`, `STARTED`, `COMPLETED`, `FAILED`, `CANCEL_REQUESTED`, `CANCELLED`, `UNKNOWN` as **consumer observations**, not one shared scheduler state machine in K0.
- A transport-level `200` or `202`, return value, successful validation, process exit zero or UI label is not in itself proof that intended external state is achieved.
- A failed operation may have partial external effects; a compensating operation is a new effect with its own authorization and receipt.
- An error message containing `SUCCESS` cannot become `OK`; terminal status is typed data, never inferred from history-string search.
- Receipt has explicit evidence **level**: `DECLARED`, `OBSERVED`, `VERIFIED_AGAINST_ORACLE`, `INDEPENDENTLY_AUDITED`. This is a *scope-aware classification*, not universal truth and not a replacement for domain evidence.
- K0 creates **no durable store** and no universal log bus. P05 or a consumer backend may persist admissible data under separately qualified guarantees.
- Redact secret and high-sensitivity payloads at the point of receipt creation. Retaining a digest does not automatically make a record anonymous.

## 8. Contract composability: an illustrative boundary trace

```text
Caller describes request (data)
  -> normalize and validate SchemaRef
  -> identify target / requested EffectDescriptor
  -> consumer policy engine evaluates exact scoped request
  -> AuthorityDecision record (K0 representation)
  -> consumer executor independently re-verifies authority
  -> executor runs effect (OUTSIDE K0)
  -> observer reports Result and scoped Receipt
  -> optional specialist handles compensation / recovery
```

Not every use involves effects. A pure P03 compiler analysis or P05 lexical ranking can use `Schema`, `VersionRef` and `Result` without any `AuthorityDecision` or lease. **K0 is a suite of thin contracts, not a mandatory workflow for all consumers.**

### 8.1 Mock profile A: native resource in a game host

- Alien-Works/Trial motivate a `ResourceOwner` with an acquired mock handle, generation, and lease.
- A deliberate partial-construction failure yields `FAIL` with acquisition receipt; any cleanup is performed by a mock P01 adapter.
- K0 can assert that a `RELEASED` report includes a release observation for the same generation; cannot infer C finalizers or GPU state.

### 8.2 Mock profile B: Uncharted-style bounded domain action

- A domain script declares `open-door` transition and a restricted native capability request. Its domain state machine lives in a mock P03-like adapter.
- K0 schema validates the request, and effect descriptor declares native mutation. A fake consumer authority denies an unpermitted invocation.
- No engine VM or actual Uncharted code is imported; the actor's state-machine logic remains outside K0.

### 8.3 Mock profile C: coding assistant file mutation and UI observation

- A mock P04 edit request binds canonical input and version; fake P07 UI renders accepted/pending/result states without becoming the authority.
- Unauthorized or stale-base effect is refused before a mock write; a later verification failure remains `FAIL` or `UNKNOWN`, not an invented rollback.

### 8.4 Mock profile D: retrieval and multiple causal histories

- P05 source revision and P06 history node each use `IdentityRef` / `VersionRef`; even identical bytes do not mean identical lineage.
- P06 owns fork/synthesis and does not import its history graph into K0. K0 can report candidate version/provenance and scoped qualification outcomes.

## 9. Minimal Gerbil design: APIs and module skeleton

A *small* public surface is preferred over a reflection-first system. Candidate modules below are placeholders, not files created or compiled:

```text
foundation/k0/
  gerbil.pkg
  build.ss
  README.md
  src/
    identity.ss      # IdentityRef, VersionRef and exact ref checks
    validation.ss    # SchemaRef, ValidationResult and diagnostics
    effects.ss       # inert EffectDescriptor class/category
    authority.ss     # inert AuthorityDecision and commitment metadata
    lease.ss         # ResourceOwner and LeaseObservation
    outcome.ss       # Result, Failure, Uncertain, Receipt
  tests/
    identity-test.ss
    validation-test.ss
    effects-test.ss
    authority-test.ss
    lease-test.ss
    outcome-test.ss
    cross-domain-test.ss
    dep-boundary-test.ss
  qualification/
    sources.lock     # external source refs and documentation revision notes
    fixtures/        # synthetic independent cases, no proprietary assets
```

**No generic registry service in K0.** Records are immutable after public construction; optional small checked constructors and predicates enforce shape. Use Gerbil `defstruct` or an equally narrow documented representation where feasible, rather than a global mutable object table. Exact syntax and export policy require compilation probes against the selected toolchain.

### 9.1 Candidate API names (descriptive, not build-verified Gerbil code)

```scheme
;; Normative surface sketches; NOT executable source claims.
(make-identity-ref namespace kind local-id authority-domain generation)
(make-version-ref identity schema-revision semantic-revision lock)
(same-ref? a b)
(validate-value schema-ref inert-value)
(make-effect-descriptor operation-ref class scopes commitment)
(make-authority-decision decision request-binding policy-ref status)
(verify-decision-binding decision normalized-request expected-scope)
(make-lease-observation owner resource generation state evidence)
(make-ok value evidence)
(make-failure code stage evidence residual-effects?)
(make-unknown code last-verified evidence-gaps)
(make-receipt operation-ref outcome observer evidence scope)
```

`verify-decision-binding` checks **shape and exact reference bindings only**; it may not pronounce a signature, issuer, protected path or operating-system permission valid. A consumer must enforce its own authority sources. No `execute!`, `start-thread`, `send-message`, `open-file`, `eval`, `compile-policy`, `fork`, `merge`, `register-global`, `database-connect` or `ffi-call` belongs here.

### 9.2 Schema and version compatibility policy

- A wire format is a **qualified adapter**, not a K0 default. Implement in-memory records first.
- Reference canonicalization must define numeric, Unicode, ordering, optional-field and integer-range behavior. The basic K0 contract can require deterministic structural comparison without promising universal stable hashing.
- Explicitly separate `schema_revision` (shape), `semantic_revision` (behavioral interpretation), and `source_lock` (producing code/environment). All three can matter to an application; no friendly version string substitutes for them.
- K0 default unknown-version disposition is `UNVERIFIED`/reject. Cross-version migration belongs to a qualified adapter, potentially K1 or a specialist.

### 9.3 Source/compile/runtime phases

- Trusted source macros are compile-time Gerbil facilities; a generated `defstruct` descriptor is not an authority grant.
- Data-only schema loading must not call `eval`, read an executable module, trigger FFI, import network/GUI modules, or call user-supplied validators.
- The first implementation slice can use direct constructors and explicit checked predicates. **Do not build a general macro compiler for six records.**
- When using `:std/interface`, publish only checked facades from K0 public modules. The documented `&` unchecked interface methods bypass checks and must not form the untrusted boundary.
- `:std/error` exceptions remain possible during programming faults, but expected boundary refusals use tagged `ValidationResult` or `Result` so callers cannot confuse error text with success.
- Verify actual Gerbil and Gambit versions and dependency closure with `gerbil build` and `:std/test` when implementation is authorized; paper API examples do not pass compilation by decree.

## 10. Minimality budget and rejected generalizations

| Temptation | K0 verdict | Correct destination |
|---|---|---|
| A universal state machine for Uncharted, Chess and process services | **Reject** | P03, P06, P02 respectively |
| General causal hypergraph/merge protocol | **Reject** | P06; optional K1 adapter if independently warranted |
| Macro interpreter / DSL compiler | **Reject** | P03/P07/P01, possibly K1 after proofs |
| A global agent/tool permission authority | **Reject** | P04 and consumer policy |
| General actor/message bus or work scheduler | **Reject** | Relevant specialist or K1 |
| OS sandbox, crypto signing, ACL or capabilities broker | **Reject** | External security/consumer enforcement |
| Native handle allocator, GPU/SDL runtime, callbacks | **Reject** | P01 |
| Guix/Guile/Shepherd bridge or package graph | **Reject** | P02 research / external deployment only |
| Universal transaction/rollback/effect compensation | **Reject** | P04/P06/consumer-specific, later K1 evidence |
| DB, cloud evidence store or RAG layer | **Reject** | P05 qualified backend |
| UI, HTTP routing, task status transport | **Reject** | P07 |
| Full code-as-data meta-control plane | **Reject** | P06/P03/P07; K0 standardizes policy/version refs only |

The entire kernel should be independently importable with no source imports from specialists or research repositories. Six families can be implemented as small modules without forcing a single monolithic source file or dependency graph.

## 11. Independent qualification design

No test listed below has been executed. `PASS` is a future evidence-dependent result, not a label for a successful proposal.

### 11.1 Proposed K0 qualification gates

| Gate | Required criterion | Proof |
|---|---|---|
| Q0 — Source & legal provenance | Pinned sources, docs, review evidence classes, no proprietary code copy | Register and independent citation audit |
| Q1 — Pure Gerbil dependency closure | K0 imports only documented Gerbil core/selected pure stdlib | Clean environment dependency graph/build |
| Q2 — Identity, revision, equality | Exact identity distinct from observation equivalence; ABA controls | T01–T12 |
| Q3 — Schema validity & inert input | Invalid or hostile declarative input cannot cause side effects | T13–T24 |
| Q4 — Effect description | Classification is information, never execution | T25–T32 |
| Q5 — Authority binding | Denied/stale/mismatched/forged data does not masquerade as permitted action | T33–T42 |
| Q6 — Resource/lease representation | Transfer/release uncertainty and stale handles properly represented | T43–T52 |
| Q7 — Typed outcome & evidence | Failure/unknown/partial effects cannot become success | T53–T62 |
| Q8 — Cross-domain mock conformance | Native game host + bounded script + agent/UI + retrieval/history comparisons | T63–T68 |
| Q9 — Toolchain API conformance | Core constructors, checked interfaces and test runner compile on pinned Gerbil/Gambit | Compilation probes plus T69 |
| Q10 — Negative dependency/security review | No unwanted transitive imports, no eval/side-effect-on-parse | T70–T71 and dependency scans |
| Q11 — Independent review and qualification honesty | Third-party fixtures, documentation gaps marked, staged release semantics | T72 + independent audit receipt |

### 11.2 Exact 72-case future test register


#### Identity & version: T01–T12

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T01** | Two refs share local ID but differ in namespaces | Exact identity rejects alias |
| **K0-T02** | Two refs share namespace/ID but have different kinds | Kind mismatch prevents equality |
| **K0-T03** | Same resource ID reused after lease closure with later generation | Stale prior-generation ref rejected |
| **K0-T04** | Same state bytes under two P06 historical lineages | Byte similarity does not imply historical identity |
| **K0-T05** | Same P07 entity values under distinct entity IDs | Entity identities remain distinct |
| **K0-T06** | Same display version string but changed semantic revision | Version mismatch detected |
| **K0-T07** | Digest forced equal for two different canonical objects | Exact comparison does not trust digest only |
| **K0-T08** | Source lock absent in a claim requiring original compiler pin | Cannot claim pinned reconstruction |
| **K0-T09** | Unknown namespace and unsupported kind in imported record | Typed validation error, no fallback to default namespace |
| **K0-T10** | Invalid or mixed-form Unicode ID normalization inputs | Deterministic declared policy or refusal |
| **K0-T11** | Null/empty or oversized identifiers | Refused with bounded diagnostic |
| **K0-T12** | Reflected artifact version differs from source revision | No automatic substitution or inferred compatibility |

#### Schema & inert validation: T13–T24

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T13** | Well-formed valid fixture checked repeatedly | Canonical result stable and input object unchanged |
| **K0-T14** | Required field missing | INVALID with path and no backend effects |
| **K0-T15** | Duplicate conflicting record key | Rejected rather than last-write-wins |
| **K0-T16** | Unknown field under strict schema profile | Explicit rejection |
| **K0-T17** | Unknown field under extension-preserving profile | Opaque bounded extension preserved without execution |
| **K0-T18** | Cross-version schema validated against wrong revision | Not VALID for original schema |
| **K0-T19** | Untrusted list contains executable Scheme-looking payload | Parsed as inert data or refused, never evaluated |
| **K0-T20** | Invalid nested type deep in record | Typed bounded path to failing field |
| **K0-T21** | NaN/infinity/unsupported numerical value at exact-data boundary | Typed refusal |
| **K0-T22** | Numerical unit mismatch or silent decimal scaling attempt | Refused absent explicit converter |
| **K0-T23** | Huge nested object, cycle or oversized text input | Quota enforcement and no unbounded recursion |
| **K0-T24** | Validation succeeds but request effect is unauthorized | Schema VALID never converted into authority ALLOW |

#### Effect descriptors: T25–T32

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T25** | Construct a read-only effect descriptor | No filesystem read occurs during construction |
| **K0-T26** | Construct native/process/network descriptor | No native call/thread/socket starts |
| **K0-T27** | Absent or unrecognized effect class | UNKNOWN, never PURE |
| **K0-T28** | Compound read and write classified by union | All effect classes retained, not minimized |
| **K0-T29** | Read private record versus write public temp value | No universal privilege ranking inferred |
| **K0-T30** | Compensation declared possible but never tested | Cannot report rollback verified |
| **K0-T31** | Effect descriptor with stale target reference | Unverified/refused by consumer binding check |
| **K0-T32** | Effect declared PURE but instrumented mock observes side effect | Declaration flagged inconsistent; purity not certified |

#### Authority decisions: T33–T42

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T33** | No decision supplied for requested mutation | Refused/UNKNOWN, never ALLOW |
| **K0-T34** | Decision request digest matches but exact request differs | Exact binding fails |
| **K0-T35** | Decision bound to prior target version | Stale decision not reusable |
| **K0-T36** | Decision for wrong namespace, object, resource or scope | Binding fails before fake effect |
| **K0-T37** | Decision expired according to trusted consumer clock | Admission refuses, no automatic refresh |
| **K0-T38** | Untrusted policy data self-assigns ALLOW | Cannot become trusted authority |
| **K0-T39** | Unknown/contradictory decision enum | Typed invalid decision |
| **K0-T40** | Forged producer claim embedded in plain record | K0 never labels it authenticated |
| **K0-T41** | Allow decision for operation that needs actual OS confinement | Receipt does not claim OS sandbox |
| **K0-T42** | Concurrent mock callers race on mutable request versions | No grant from mismatched snapshot |

#### Resource & lease: T43–T52

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T43** | Partial acquisition fails before owner recorded | No fictitious OWNED state; failure evidence retained |
| **K0-T44** | Acquire completes and owner/generation observed | OWNED only with matching evidence |
| **K0-T45** | Borrowed handle requests destruction | Not represented as authorized release |
| **K0-T46** | Transfer pending without accept confirmation | State remains uncertain, not double-owned |
| **K0-T47** | Two owner claims for same exclusive generation | Conflict detected in consumer test |
| **K0-T48** | Release requested but callback pending | Not prematurely marked RELEASED |
| **K0-T49** | Destructor reports exception | RELEASE_FAILED/UNKNOWN with residual risk |
| **K0-T50** | Double release with old observation | Typed duplicate/stale result |
| **K0-T51** | Old callback uses reused resource ID with new generation | ABA guard rejects old generation |
| **K0-T52** | Unverifiable thread affinity of native destructor | K0 never claims correct native thread |

#### Result, failure & receipt: T53–T62

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T53** | HTTP 202 accepted but worker not yet finished | Typed acceptance is not success |
| **K0-T54** | Worker started then UI disconnected | Task outcome UNKNOWN, not FAILED or COMPLETED |
| **K0-T55** | Failed command output includes literal SUCCESS | Typed failure preserved |
| **K0-T56** | Successful check after earlier failure text | Current outcome success only with current evidence |
| **K0-T57** | Failed filesystem write may have partially mutated disk | Residual effects possible present |
| **K0-T58** | Cancellation requested before external acknowledgment | Cancellation request not completion |
| **K0-T59** | Compensation attempted and failed | Original failure and failed compensation separate |
| **K0-T60** | Receipt produced without independent verification | Claim level DECLARED/OBSERVED, not independently audited |
| **K0-T61** | Sensitive credential placed in receipt input | Redaction/refusal before retention |
| **K0-T62** | Truncated or missing evidence attachment | Explicit gap; never exhaustive verification |

#### Cross-domain mocks: T63–T68

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T63** | P01-like native handle plus Alien-Works/Trial teardown failure | Same K0 lease/result vocabulary without FFI imports |
| **K0-T64** | Uncharted-like domain state hook denied by mock policy | State/script semantics independent from authority record |
| **K0-T65** | P04 edit + P07 display observe accepted versus failed | No false UI success; server authority kept distinct |
| **K0-T66** | P05 corpus source vs P06 causal node equal content | Both use Ref/Version yet have different identity semantics |
| **K0-T67** | P02 service readiness vs P03 director signal | Shared result semantics without shared scheduler |
| **K0-T68** | Run all four mock profiles with one unchanged K0 codebase | No specialization-specific source edits or dependencies |

#### Toolchain & dependency: T69–T72

| ID | Falsification fixture | Required observation |
|---|---|---|
| **K0-T69** | Pinned Gerbil/Gambit build + :std/test checked facade probes | Compilation and real assertion exit status required |
| **K0-T70** | Import K0 in clean package without P01–P07/Guix/SDL/LLM/network | Dependency closure remains pure and minimal |
| **K0-T71** | Attempt unchecked interface export or untrusted eval/FFI in K0 | Negative static scan plus runtime instrumentation reject |
| **K0-T72** | Independent auditor reproduces negative boundary fixtures | Gate statuses evidence-based; untested claims UNQUALIFIED |

## 12. Phased execution plan, if separately authorized

1. **K0-00:** freeze one Gerbil/Gambit version and exact official-doc references; inspect and record APIs rather than assume earlier source syntax compiles on the target toolchain.
2. **K0-01:** implement immutable `IdentityRef`, `VersionRef`, `SchemaRef` records and no-effects validation; qualify Q1–Q3 before adding effect records.
3. **K0-02:** implement inert `EffectDescriptor` and `AuthorityDecision` data model, plus exact-binding helper without any authenticating or executing power; qualify Q4–Q5 with adversarial fixtures.
4. **K0-03:** implement `ResourceOwner` and `LeaseObservation`; mock acquisition, failure, transfer and destruction; qualify Q6.
5. **K0-04:** implement `Result`, `Failure`, `Uncertain`, `Receipt`; define claim-level and secret-safe output; qualify Q7.
6. **K0-05:** run independent mock profiles for Alien-Works/Trial resources, Uncharted state/native hook, P04+P07 controlled requests, P05+P06 identity/version, and P02+P03 status boundaries. Qualify Q8 without adding any specialist dependencies.
7. **K0-06:** pure-import audit, pinned-toolchain build, negative unsafe-interface/export tests and independent qualification, Q0/Q9–Q11. Only then consider a first `K0_CONTRACTS_QUALIFIED` release.
8. **After K0:** separately research K1 candidates. A pattern is not promoted just because it appeared in two documents. It needs independently working consumer evidence and no hidden specialization.

**No automatic implementation authorization:** this sequence is a plan, not a request to run scripts, alter source, compile untrusted code or push to GitHub. No runtime test results are asserted by producing this document.

## 13. Open issues and decision register

1. **Canonicalization:** exact byte encoding and cross-process representation, including Unicode and numeric normalization; K0 may use in-memory exact comparisons until a wire profile qualifies.
2. **Semantic revision compatibility:** whether to provide any K0 comparator beyond exact equality; the conservative baseline does not.
3. **Authority authentication:** K0 stores issuer/version data but cannot verify real credential provenance by itself; external policy/crypto owner required.
4. **Effect classes:** the default is descriptive categories, not a total privilege lattice; domain-specific risks must not be suppressed by an enum ordering.
5. **Resource transfer:** represent partial transfers without pretending distributed atomicity.
6. **Receipt cardinality:** one operation can emit multiple observations; K0 must not require a single consolidated terminal account without a consumer reconciliation policy.
7. **Error taxonomy:** separate programmer errors thrown by Gerbil from expected operation failures returned as data. A code-generating macro does not solve this choice.
8. **Gerbil macro/record API:** confirm against pinned version and official docs in an executable compiler probe; some reference pages remain incomplete.
9. **Dependency purity:** `:std/test` and build machinery can be development-only; the installed runtime dependency closure should remain minimal.
10. **Evidence semantics:** a source review is evidence that a particular code path exists, not that it passed; a receipt is evidence that an observation occurred at a scoped boundary, not universal truth.
11. **Game architecture transfer:** retain Alien-Works native-host insights and Uncharted bounded script semantics as *separate* witnesses; do not silently resurrect isolated architecture concept 04.
12. **Actual application selection:** K0's first consumers must be synthetic unrelated profiles; selecting a particular production application belongs to future implementation governance.
13. **License strategy:** independent Gerbil implementation is the current design. Before copying any upstream code, verify exact file license, dependencies and derivative obligations. Naughty Dog proprietary source is excluded.
14. **Co-evolution:** specialists P01–P07 are not rewritten by K0 design decree; a separate compatibility matrix and adapter migration plan is required once K0 itself is qualified.
15. **Qualification honesty:** zero future tests are marked passed before an executed clean build, negative fixture evidence and independent reproduction.

## 14. Final design verdict

`LFR_K0_D01_MINIMAL_CONTRACT_KERNEL_PROPOSED / BOTH_ORIGINAL_GAME_ORIENTED_SOURCES_INCLUDED / ALL_P01_TO_P07_LINEAGES_CROSSED / ASSOCIATED_COMPARATORS_INCLUDED / OFFICIAL_GERBIL_DOCS_CROSS_CHECKED / SIX_MINIMAL_CONTRACT_FAMILIES / NO_UNIVERSAL_EXECUTOR / NO_GERBIL_GUIX_BRIDGE / NO_PROPRIETARY_CODE / CROSS_DOMAIN_MOCK_PROFILES / 72_FUTURE_TESTS / 12_FUTURE_GATES / NO_RUNTIME_QUALIFICATION / DOCUMENT_PUBLISHED`

**Core invariant:** a source may inspire a descriptor; a descriptor may be validated; a validated descriptor may be considered by a consumer authority; an authoritative consumer may execute through its separately qualified backend; and a scoped receipt may record what was actually observed. **No one step grants the assurances of the next.**

K0 is successful only when two or more unlike consumers genuinely share stable semantics without importing one another's engines. That is a harder and more useful achievement than maximizing the number of things called `core`.