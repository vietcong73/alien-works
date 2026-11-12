# Alien-Works-Centered Systematic Evidence Review

**Evidence review ID:** LFR-SR-01  
**Research date:** 12 November 2026  
**Status:** `EVIDENCE_MAPPED / STATIC_COMPARISON / DESIGN_NOT_STARTED`  
**Primary case:** `borodust/alien-works@86381f8bce44a396347cbe8de70349be5b43ece1` (1 May 2023); preservation fork `vietcong73/alien-works`  
**Previously isolated reference:** `lisp_reserve_hybrid_concept_04_isolated.md` — historical hypothesis only, **not** a selected architecture  
**Future language being investigated:** Gerbil Scheme. **No implementation selection, architecture concept, build, patch, benchmark, or deployment is authorized by this report.**

## 0. Review question, epistemic ceiling, and exclusions

**Question.** What can be established from inspectable implementations, original technical presentations, language/runtime references and prior bounded source audits about (a) the mechanisms Alien-Works already implements, (b) their actual limitations, (c) mechanisms implemented elsewhere in related Lisp/Scheme/native systems, and (d) the **currently demonstrated** technical basis for further Gerbil-focused inquiry?

**This is an evidence-mapping / structured scoping systematic review**, not an exhaustive PRISMA literature review, meta-analysis, design specification, code selection, cost estimate, implementation plan, or proof of runtime equivalence. The current source selection is purposive, traceable and limited to sources accessed during this pass. Search-engine coverage is not assumed complete. The absence of source evidence does not prove that a capability cannot exist.

**Hard exclusions:**

1. Do not promote the isolated architecture 04 into an accepted concept. Its earlier Common Lisp-host sketch is a **historical hypothesis**. Gerbil is now a planned investigation target, not a validated drop-in equivalent.
2. Do not select between streamline, extract, rewrite, rebuild, custom VM, native engines, graphics stacks or project architecture.
3. Do not copy/propose copying Naughty Dog proprietary code or infer its undisclosed source layout.
4. Do not treat a project README or a diagram as a source-level audit.
5. Do not admit Gerbil MCP or Gerbil Shell (code inaccessible in earlier reviews) as audited implementations. Exclude microGPT explicitly per owner direction.
6. Do not mutate the preservation fork or another remote repository, or modify existing audit conclusions.

## 1. Search and evidence protocol

**Collection methods:** authenticated GitHub source-tree/blob inspection, archived immutable source links, official Gerbil/Gambit API documentation, original Naughty Dog GDC slide PDFs (with visual inspection of representative pages), primary project sites and source-repository READMEs. Earlier local audits 02/03/04/05 and the six separately committed Gerbil reviews form **background evidence**, not fresh runtime tests.

**Search families used:** `Alien-Works CFFI Filament SDL2 PhysX Skia`, `Uncharted PLT Scheme data compiler state scripting`, `Gerbil FFI Gambit C-interface`, `Gerbil actor API`, `Gerbil SDL2 binding`, `Common Lisp Trial event loop resources`, `CEPL/Varjo graphics language`, `Guile Chickadee`, `Spritely Goblins capability`, and `Gambit Raylib`. Repository names and pinned files were then opened directly. Sampling is oriented toward **mechanism coverage**, not popularity ranking.

**Evidence grades:**

- **A — Direct source:** specific implementation statements read from immutable source paths; confirms code shape and local control flow, not that a program executes correctly.
- **B — Original primary presentation / official reference:** first-party architectural and API behavior explicitly described; cannot establish unprovided internal source.
- **C — Previous bounded review:** carries forward clearly bounded findings with source anchors; does not imply new independent retesting.
- **D — Project README / secondary context:** discovery and claims for further investigation; no performance, compatibility or correctness qualification.
- **U — Unresolved:** necessary evidence absent or contradictory, with no inference of success or failure.

**No runtime evidence in this pass.** No Gerbil compiler, ASDF, graphics engine, C ABI harness, memory sanitizer, native dependency loader, scene renderer, or concurrency stress rig was executed. All numerical performance claims are unqualified.

## 2. Reproducible source register

| ID | Source and version | Inspection level | Mechanism studied | Limitation |
|---|---|---|---|---|
| AW | [Alien-Works `86381f8b`](https://github.com/borodust/alien-works/tree/86381f8bce44a396347cbe8de70349be5b43ece1), 82 tracked blobs; 77 Lisp source positions in earlier audit | **A + C** | ASDF graph, scoped lifecycle, foreign handles, event poller, render queue, canvas worker, material/shader macros | 2023 snapshot; dependency closure not established |
| ND08 | [Liebgold, GDC 2008, *Adventures in Data Compilation*](https://media.gdcvault.com/gdc08/slides/S6636i1.pdf) | **B** | Scheme as code/data middle layer, typed data authoring | Slides, not proprietary source |
| ND09 | [Gregory, GDC 2009, *State-Based Scripting in Uncharted*](https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf) and [GDC Vault](https://www.gdcvault.com/play/1730/State_Based_Scripting_in_UNCHARTED__DRAKE_S_FORTUNE_and_UNCHARTED_2__AMONG_THIEVES) | **B + C** | States, tasks, directors, runtime tracks, native hooks | Primary PDF text accessed; render screenshot failed due large PDF; no production source |
| TR | [Trial `5629b23d`](https://github.com/Shirakumo/trial/tree/5629b23d099df7d682093e56abc270c46340cd4e) | **A** | `resource.lisp`, `event-loop.lisp`, `scene.lisp` | GitHub archived 2026; moved to Codeberg; mirror does not establish current tip |
| CE | [CEPL `88e9d005`](https://github.com/cbaggers/cepl/tree/88e9d005e86c54d681b640bbd14f0ef25fb449d4) | **A + D** | `core/lifecycle.lisp` and REPL-first GPU model | OpenGL / Common Lisp; not Gerbil |
| VA | [Varjo `68132f72`](https://github.com/cbaggers/varjo/tree/68132f727d17d953ae1f7ac5dbc184df83a65f20) | **D + AW source linkage** | Lisp-like Vari to statically typed GLSL, macro authoring | Not a verified full compiler audit in this pass |
| GER-FFI | [Gerbil FFI guide](https://gerbil.scheme.org/guide/ffi.html) and [Gambit C-interface](https://gambitscheme.org/4.2.3/gambit.pdf) | **B** | `begin-ffi`, `define-c-lambda`, owned/borrowed foreign pointers, release functions | Historical Gambit v4.2.3 manual plus current Gerbil guide; not actual AW native adapter port |
| GER-ACT | [Gerbil v0.18+ actor reference](https://gerbil.scheme.org/reference/std/actor.html) | **B** | Message envelopes, expiry, request/reply, actor handles and supervision APIs | API availability is not an application scheduler/authorization proof |
| GER-BLD | [Gerbil build tool](https://gerbil.scheme.org/reference/dev/build) and [development guide](https://gerbil.scheme.org/guide/getting-started.html) | **B** | Isolated dependency environments, optimized/release builds | Reproducible build of target code not tested |
| GSDL | [nymacro/gerbil-sdl2 `ed03678d`](https://github.com/nymacro/gerbil-sdl2/tree/ed03678de77807146718d4d23397ec12fd593cc0) | **A + D** | Gambit SDL2 C binding, finalizers, weak reference table | Last pinned commit 2019, explicitly experimental |
| RAY | [Gambit Raylib binding `df7ea6db`](https://github.com/georgjz/raylib-gambit-scheme/tree/df7ea6db54a3196d13c66dac4909c64cea6e5a88) | **D** | Alternate Scheme/C graphics bridge | 2021 experimental README; code not deeply audited |
| GOB | [Spritely Goblins official overview](https://spritely.institute/goblins/) and [architecture paper](https://files.spritely.institute/papers/spritely-core.html) | **B** | Capability-security, encapsulated actors, local transactions, distributed references | Guile/Racket, not Gerbil; no source-port equivalence established |
| CHI | [Chickadee project](https://dthompson.us/projects/chickadee.html) and [Guix package description](https://github.com/lfam/guix/blob/master/gnu/packages/game-development.scm) | **D** | Guile Scheme with SDL2/OpenGL game tooling | Upstream source access failed; retained only as contextual comparator |
| G6 | [Six source-grounded Gerbil application reviews](https://github.com/vietcong73/alien-works/tree/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects) | **C** | Glow, GloUI, Chess AI, Daily-use, Coding Harness, AutoContext | Separate static audits; no new runtime assertions; microGPT absent |

**Selection note:** A project can appear in this table without being admitted as a code-reuse candidate. Source access and evidence specificity determine claim strength.

## 3. The Alien-Works nucleus: observed mechanisms, not desired design

### AW-01. Intended foundation and dependency boundary

The upstream project describes itself as a **high-performance, tightly coupled Common Lisp game foundation**, not a complete game engine. The upstream principles expressly favor low allocation in hot loops and native performance over pure-Lisp implementation, but those are **design priorities**, not independently measured outcomes. [`README.md`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/README.md).

[`alien-works.asd`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/alien-works.asd) directly depends on `alien-works-foundation`, SDL, GLM, Filament, PhysX, Chipmunk, Skia, OpenAL, Opus, OpenGL plus Common Lisp tooling, CFFI/IFFI, thread and shader libraries. The ASDF root is a large native-runtime dependency umbrella. The former isolated DC-01 investigation failed to establish a coherent, pinned, reproducible dependency closure, especially for `alien-works-foundation` and wrapper revisions. That historical negative finding remains **unresolved**, not retested or silently upgraded here.

**Established:** actual root dependency declarations and intentionally tight coupling. **Not established:** successful loading on any current OS, native compatibility, performance superiority, general-purpose modularity, Gerbil linking or all transitive licensing.

### AW-02. Scoped foreign-resource composition

[`src/framework/framework.lisp`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/framework/framework.lisp) composes nested `with-window`, `with-audio` and `with-renderer` contexts and adds error reporting. This is a real implementation of host lifecycle composition, although the earlier core audit found defects in memory, stream, startup/failure handling. A successful happy-path macroexpansion does not establish exception-safe or thread-safe native teardown.

### AW-03. Events and native execution

[`src/host/host.lisp#L542-L606`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L542-L606) receives SDL events through `poll-event` and dispatches a handler; `event-kind` categorizes raw event types. **This is a poller and normalization layer, not a declarative state machine or director runtime.**

[`src/graphics/engine.lisp#L55-L125`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/engine.lisp#L55-L125) allocates renderer resources, maintains a guarded task list, drains tasks in frame context, and uses `unwind-protect` around rendering operations. It provides a concrete task-to-render-thread pattern. It does **not** establish a general workflow scheduler, exactly-once event semantics, bounded backlog or cancellation policy.

[`src/graphics/canvas.lisp#L350-L433`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/canvas.lisp#L350-L433) adds worker tasks, latches, a nominal 60 Hz loop and shutdown coordination. Historical audits list unresolved failure-path and synchronization qualifications. No missed deadline, deadlock or throughput rate is claimed without execution.

### AW-04. Foreign callbacks, resources and DSL compilation

[`src/utils/utils.lisp#L310-L321`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/utils/utils.lisp#L310-L321) registers callbacks with numeric IDs and removes them after execution via `unwind-protect`. This establishes a callback identity/lifecycle protocol, but no complete concurrency/GC/ABI protocol.

[`src/graphics/material.lisp#L418-L458`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp#L418-L458) defines a domain material macro that emits classes/defaults; [`#L625-L655`](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp#L625-L655) defines shader macros integrated with Varjo. These are concrete **domain authoring and translation** facilities. They are not restricted execution of untrusted code, nor evidence of a VM/agent policy enforcement boundary.

### AW-05. Prior audit assurance ceiling

The local 02/03/05 reviews previously found source-level memory/stream failures, lifecycle concerns and tooling mismatches; 77 Lisp source positions received subsystem classifications. Those classifications cover **file positions**, not 77 formal end-to-end correctness proofs. The conclusion already established was `WHOLE_FRAMEWORK_CORE_IMPORT_REJECTED_FOR_CURRENT_EVIDENCE / SELECTIVE_EXTRACTION_CONDITIONAL`, with **streamline versus rebuild undecided**. This review neither reverses nor extends that decision.

## 4. External comparative observations

### C-01. Naughty Dog / Uncharted — distinct semantic/temporal reference

The **2008 original slide deck** explicitly positions Lisp/Scheme in the space between code and passive data, with particle definitions, animation and gameplay scripting as examples; it shows Scheme-style typed declarations. The **2009 original presentation** describes PLT Scheme for data definitions and runtime scripting, state-based control, engine object extension, and a bounded runtime layer. The prior evidence review covers director/task roles, tracks, waits and native hooks. This is **evidence of a production architectural pattern, not access to production source or an implementation to copy**.

**Important non-equivalence:** domain state scripts are not SDL event polling; compiled tracks are not a renderer task queue; custom bytecode is not implied by `defmacro`; scripts' restricted native hooks are not equivalent to unrestricted FFI in a trusted host.

### C-02. Trial — resource and event model contrast

At the pinned GitHub archive, `resource.lisp` defines generic `allocate`, `deallocate`, `allocated-p`, `finalize`, dependency-sensitive `ensure-allocated`, and context checks for GL resources. `event-loop.lisp` defines events/listeners, pools, handlers and restarts; `scene.lisp` composes event loop with a scene graph/registrations. These are concrete examples of **separable application-resource and scene responsibilities**. It is **not** proven that all of Trial's lifecycle/error behavior is correct, nor is the archived GitHub revision the new Codeberg tip. Its role here is comparator, not a recommendation to use Trial.

### C-03. CEPL and Varjo — alternative authoring-to-GPU boundary

CEPL's design documentation characterizes the graphics surface as a live REPL output; `core/lifecycle.lisp` expresses `uninitialized`, `suspended`, `active`, `shutting-down` states and listeners. Varjo documents a Lisp-derived, statically typed shader dialect compiled to GLSL. Alien-Works directly uses Varjo in its material macros. These materials help distinguish **host language code, authored shader representation and GPU execution**. They do not show any automatic path from Common Lisp shader/compiler internals into Gerbil.

### C-04. Gerbil/Gambit native capabilities — available mechanisms

Official Gerbil `:std/foreign` documentation shows `begin-ffi`, `define-c-lambda`, `define-c-struct`, and explicit borrowed-pointer, release-function and foreign-type concepts. The cited historical Gambit v4.2.3 manual (not the current integrated runtime revision) discusses when release functions run and `foreign-release!`; contemporary API compatibility is corroborated only where the current Gerbil FFI guide documents it. These features establish that a **C foreign interface exists and has ownership vocabulary**. They do **not** establish equivalence with Alien-Works's CFFI/IFFI macroexpansions, C++ wrappers, GL/SDL thread affinity, native callback re-entry, or packaged binary ABI.

The `:std/actor` reference for Gerbil v0.18+ documents message envelopes, sender/destination metadata, expirations, request/reply, remote handles and actor/ensemble management. Crucial documented caveat: local threads are implicitly authorized for actor administration; a remote ensemble without an administrative public key may regard remote actors as authorized. Actor messaging therefore **cannot be treated by itself as an object-capability security proof**.

Official Gerbil build docs cover local dependency environment isolation and optimized/release builds. This is toolchain support, **not a reproducible native build of Alien-Works**.

### C-05. Gerbil SDL2 — a limited empirical binding specimen

`nymacro/gerbil-sdl2` has actual Gambit C-interface source, a finalizer macro using `make-will`, weak-key reference tracking, and a tiny application example. Its own README calls the bindings *experimental*, explicitly states garbage-collector tying is ongoing, and flags `SDL2_image` as needing work. Its pinned source is from 2019. It is evidence that **a Scheme/C SDL binding was attempted**, not proof of production-ready SDL resource semantics in Gerbil 0.18.2 or of Alien-Works SDL equivalence.

### C-06. Spritely Goblins — capability-security model, distinct runtime

The Spritely Institute describes Goblins as transactional distributed object programming with encapsulated behavior and asynchronous actors, currently available for Guile/Racket. Its security and local transaction claims are first-party architectural/reference evidence. No Gerbil port, compatible object identity, transactional engine, wire protocol or formal equivalence has been demonstrated here. This comparator highlights that **actor concurrency, capability authority and transactional state are different properties**.

### C-07. Six previously audited Gerbil codebases — application evidence and failure catalog

Previously committed independent reviews cover Glow (compiler/runtime), GloUI (HTTP, transaction lifecycle and Scheme/JS interop), Chess AI (state/search), Daily-use (LLM API/cache), Coding Harness (agent tools/approval/check), and AutoContext (BM25/TF-IDF). Collectively they document Gerbil expressiveness across several application classes **and** source-level pitfalls in state transitions, boundary validation, error classification, resource authority and numerical edge cases. These are *not* direct functional replacements for Alien-Works subsystems; no shared acceptance tests or comparable benchmarks exist.

### C-08. Low-assurance surrounding projects

Gambit/Raylib, Chickadee/Guile and other potential Scheme-native projects were found, but source access or qualification depth in this pass does not support claims about their readiness, feature equivalence or maintainability. They are retained as **discovery records only**, not silently counted as audited peers.

## 5. Mechanism comparison matrix (evidence only)

| Mechanism / question | Alien-Works observed | External evidence | Gerbil evidence | Residual uncertainty |
|---|---|---|---|---|
| Domain authoring | `defmaterial`, shader macros | ND data compiler; Varjo/Vari | Gerbil macros exist, but no equivalent material pipeline qualified | Translation semantics and validation |
| Native API integration | CFFI/IFFI with many wrapped native libraries | Trial, CEPL, SDL implementations | Gambit C interface; `begin-ffi` | ABI, C++ mediation, ownership and version alignment |
| Native lifecycle | Scoped renderer/audio/window, `unwind-protect` | Trial resources, CEPL lifecycle | Foreign release functions and scoped Scheme constructs | Deterministic teardown and exceptions |
| Input/event handling | SDL polling and event categorization | Trial event listeners; ND state scripts | Gerbil actors/messaging and experimental SDL binding | Ordering, affinity, backpressure, true state transitions |
| Temporal scheduling | Render-frame work and canvas worker | ND tracks/waits/directors | Actors, messages, expiry, timeout | Deadlines, cancel/resume and durable events not qualified |
| Authoring-to-GPU | Varjo material/shader macro | CEPL + Varjo pipeline | Generic Gerbil compilation/FFI | No verified Gerbil-to-GLSL/GPU path |
| Capability authority | Raw native operations called by trusted CL runtime | ND bounded script hooks; Goblins object capabilities | Actor API and documented admin defaults | Per-object effect permissions not shown by FFI alone |
| Observability | Conditions, bounded streams, callback IDs | ND production debugging burden; Trial restarts | Gerbil logging/tests/build APIs | Cross-boundary causal traces, replay evidence absent |
| Packaging | ASDF root with broad native dependencies | Trial systems; CEPL host separation | `gerbil build`, isolated `GERBIL_PATH` | Exact Guix/Linux binary dependency closure and licensing |
| Runtime correctness | Static contradictions in prior audits | Independent codebases have known/fixed/unknown defects | Sample application source audits | No integrated common test corpus |

**Rule for reading this matrix:** The presence of a method in two systems does not mean the implementations are equivalent or interchangeable. `FOUND` is about evidence of a mechanism; `UNKNOWN` must remain unknown until a relevant test or source inspection closes it.

## 6. Portability and equivalence: verified versus unverified

### Verified from public Gerbil reference

- Gerbil has a native module/build toolchain and Gambit-derived C FFI.
- Gerbil foreign structs have explicit distinctions between owned, borrowed and released objects in the documented interface.
- Gerbil offers actor messaging, expiry, timeouts, remote handles and documented administrative behavior.
- Compilation and build isolation are supported by official tooling.

### Not verified by this investigation

- Any automatic or semantics-preserving **Common Lisp CLOS -> Gerbil object** translation.
- `defmacro`/Varjo/ASDF/CFFI/IFFI source-level compatibility with Gerbil.
- Alien-Works' exact wrapper ABI signatures, pointer ownership, C++ classes or callback calling conventions under Gambit.
- Filament/Skia/PhysX/SDL/OpenAL/Opus integration in a fresh Gerbil process under GNU Guix.
- Context-thread ownership, native object destruction order, GPU synchronization, callback pinning and GC/finalizer timing under target workloads.
- Performance parity, memory-allocation limits, system cold-start time, binary footprint, and long-running durability.
- A bounded domain-language runtime equivalent to Naughty Dog's proprietary engine.
- Authorization guarantees from actor message passing alone.

**Inference ceiling:** Gerbil is technically credible as a future implementation language in light of its documented language/runtime mechanisms and multiple actual applications. This is **not** a port-feasibility verdict for any particular Alien-Works subsystem.

## 7. Cross-system failure and negative-evidence ledger

| Risk class | Evidence source | Status | What the evidence actually establishes |
|---|---|---|---|
| AW root dependency mismatch | Historical Alien-Works DC-01 / 05 | **U** | No coherent end-to-end pinned dependency graph established |
| AW memory/stream correctness | AW audits 02/03 | **A/C** | Static source defects and conditional failure paths recorded; no fix or test claimed |
| Native startup/teardown | AW `framework.lisp`, engine, canvas; prior audit | **A/C + U** | Lifecycle composition exists; failure injection not performed |
| GC and foreign ownership | Gerbil/Gambit docs; SDL binding | **B/A** | Binding tools exist, finalizer experiments exist; native lifetime equivalence absent |
| False 'state DSL' equivalence | AW poller vs ND slides | **A/B** | Polling does not implement ND state-script semantics |
| False 'capability security' equivalence | Gerbil actor admin reference vs Goblins | **B** | Actor protocols and object-capability models are materially different concepts |
| Shader DSL transplantation | AW Varjo integration, Varjo reference | **A/D + U** | DSL translation exists in CL; Gerbil port unverified |
| Early/late events and dead objects | ND public presentation + AW callbacks | **B/A** | Lifecycle and timing hazards matter in both; no comparative incident rate |
| Legacy wrapper maturity | `gerbil-sdl2` 2019; Raylib/Gambit 2021 | **A/D** | Examples of FFI exist; maintenance and compatibility cannot be assumed |
| Overclaiming production readiness | All static case reviews | **U** | No runtime, integration, security or performance qualification during this pass |

## 8. Open evidence questions (not implementation tasks)

These are **questions to answer in a later evidence phase**, not design decisions or coding instructions:

1. Which exact versions and source repositories of `alien-works-foundation`, generated CL bindings and native libraries match Alien-Works commit `86381f8b`?
2. Which native wrapper APIs expose a stable C ABI versus C++-specific ownership/RAII requirements?
3. What are the concrete resource lifetime contracts for SDL/Skia/Filament/PhysX/OpenAL/Opus handles in the audited source and external upstream libraries?
4. Under what circumstances are Alien-Works callbacks registered, invoked, cancelled, destroyed and invoked across threads?
5. Can callback, thread-affinity and shutdown correctness be independently determined from primary native APIs without running the original code?
6. How much behavior is authored in the Alien-Works macro/Varjo layer and how much resides in backend-specific native engines?
7. Which exact semantics are public in Naughty Dog's data compiler/state-script lectures, and which remain unavailable as proprietary implementation details?
8. What is the version-/platform-specific ABI and native packaging situation for Gerbil 0.18.2 on GNU Guix/Linux?
9. Where do Gerbil's actor admin/control APIs allow local or unauthenticated operations by default, and how is that different from Goblins' least-authority model?
10. What validation corpus and measurement evidence would distinguish correctness, compatibility, maintainability and performance claims without assuming the architecture in advance?

Every open question is intentionally unresolved. The answer must be supplied by **source, docs, reproducible builds or tests**, not by architectural preference.

## 9. Boundaries retained from isolated 04

The isolated concept hypothesized a Lisp-centered semantic authoring/control side with native capabilities, originally framed around a **trusted Common Lisp host**. The current user direction instead says **Gerbil is the intended language for a later reserve codebase**. These statements are not contradictory if their stages remain separate: the older document is a reference-only historical hypothesis and the present report maps evidence about possible language/runtime mechanisms. **Neither document is an accepted Gerbil concept specification.**

The earlier pairing of AW and Uncharted is not assumed superior to simpler solutions. This report supplies **no layer diagram**, module list, interface schema, product requirements, package decisions, dependency selection, API design, project roadmap, or implementation approval.

## 10. Summary of established information

1. **Alien-Works is the uniquely detailed nucleus:** source-confirmed native integration, scoped resources, macros/DSLs and renderer/worker task paths, with an unresolved full native dependency closure and important static correctness findings.
2. **Uncharted independently confirms a substantial Scheme semantic/data/state layer** through original developer presentations, but neither proprietary source nor direct reuse rights are provided.
3. **Trial/CEPL/Varjo supply distinct Common Lisp comparators** for resource separation, event handling, lifecycle and GPU-authoring semantics; none is directly a Gerbil implementation.
4. **Gerbil/Gambit has documented FFI and actor/build primitives**, but the relevant native lifecycle, correctness, semantics and packaging equivalence for Alien-Works remains untested.
5. **Experimental SDL2/Gambit bindings prove only a limited bridge exists**, not a safe, supported complete graphics/game runtime.
6. **Goblins contributes evidence on capability-oriented actor semantics** without demonstrating portability to Gerbil.
7. **Six earlier Gerbil audits provide independent application-level specimens and failure classes**, not solutions to the Alien-Works dependency/ABI/lifecycle questions.

**Final systematic-evidence disposition:**

`SOURCE_AND_PRIMARY_DOCUMENT_EVIDENCE_REGISTERED / CROSS-SYSTEM_MECHANISMS_MAPPED / GERBIL_TRANSFER_NOT_QUALIFIED / ISOLATED_04_UNSELECTED / CONCEPT_NOT_STARTED / NO_REPO_CHANGES`

---

## References and source links

- Primary center: [Alien-Works upstream](https://github.com/borodust/alien-works/tree/86381f8bce44a396347cbe8de70349be5b43ece1); [ASDF](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/alien-works.asd); [framework](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/framework/framework.lisp); [SDL host](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp); [renderer](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/engine.lisp); [canvas worker](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/canvas.lisp); [shader DSL](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp).
- Prior internal evidence: [Alien-Works source review](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/alien-works-common-lisp-source-code-audit-review.md); [Uncharted review](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/naughty-dog-uncharted-lisp-family-review.md); [six Gerbil reviews](https://github.com/vietcong73/alien-works/tree/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects).
- Uncharted: [Liebgold GDC 2008 slides](https://media.gdcvault.com/gdc08/slides/S6636i1.pdf); [Gregory GDC 2009 slides](https://www.gameenginebook.com/resources/gdc09-statescripting-uncharted2.pdf); [GDC session](https://www.gdcvault.com/play/1730/State_Based_Scripting_in_UNCHARTED__DRAKE_S_FORTUNE_and_UNCHARTED_2__AMONG_THIEVES); [Naughty Dog public talk announcement](https://www.naughtydog.com/blog/naughty_dog_at_gdc_2009).
- Common Lisp comparators: [Trial source archive](https://github.com/Shirakumo/trial/tree/5629b23d099df7d682093e56abc270c46340cd4e); [CEPL](https://github.com/cbaggers/cepl/tree/88e9d005e86c54d681b640bbd14f0ef25fb449d4); [Varjo](https://github.com/cbaggers/varjo/tree/68132f727d17d953ae1f7ac5dbc184df83a65f20).
- Gerbil/Gambit: [Gerbil FFI guide](https://gerbil.scheme.org/guide/ffi.html); [Gerbil actors](https://gerbil.scheme.org/reference/std/actor.html); [Gerbil build guide](https://gerbil.scheme.org/reference/dev/build); [Gambit manual](https://gambitscheme.org/4.2.3/gambit.pdf); [Gerbil SDL2](https://github.com/nymacro/gerbil-sdl2/tree/ed03678de77807146718d4d23397ec12fd593cc0); [Gambit Raylib](https://github.com/georgjz/raylib-gambit-scheme/tree/df7ea6db54a3196d13c66dac4909c64cea6e5a88).
- Capability comparator: [Spritely Goblins](https://spritely.institute/goblins/); [Spritely whitepaper HTML](https://files.spritely.institute/papers/spritely-core.html).

**Publication status:** This report is a generated local research artifact only. No GitHub push, no branch modification, no source modifications, and no concept implementation.