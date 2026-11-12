# LFR-P01-D01 | Native Integration & Metaprogramming Reserve

**Document class:** Detailed Design Proposal, one specialized edition only  
**Date:** 12 November 2026  
**Status:** `DESIGN_PROPOSED / SOURCE_LINEAGE_MAPPED / IMPLEMENTATION_NOT_STARTED / RUNTIME_UNQUALIFIED`  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Edition:** P01, Native Integration & Metaprogramming  
**Target language:** Gerbil Scheme, using its Gambit-backed C interface where verified  
**Research nucleus:** Alien-Works, `borodust/alien-works@86381f8bce44a396347cbe8de70349be5b43ece1`  
**Publication and authority:** Approved for documentation publication on 12 November 2026 under `docs/design/` in `vietcong73/alien-works` (`audit/lisp-foundation-reserve`). This design document does not authorize source edits, source imports, deployment, a new repository, or implementation.  
**Supersedes:** The P01 design slice of conceptual LFR-C01 and LFR-C02. It does not supersede the independent source audits or the historical isolated concept 04.

## 0. Decision in one paragraph

P01 preserves Alien-Works as an immutable, **historical source specimen** and creates, separately, a **proposed Gerbil-native specialized edition** with two complementary capabilities: **(A)** macros and typed intermediate descriptions to author native/domain interfaces, and **(B)** explicit foreign-resource execution with ownership, ABI and thread-affinity rules. Its first executable target is a dependency-light mock backend, followed by one small opt-in native vertical slice. It is **not** a game engine, GPU abstraction framework, general-purpose language runtime, full graphics compiler, or implementation of the overall Meta-Foundation.

The core distinction is between three forms of evidence: *the original source demonstrably implements a mechanism*; *the proposed Gerbil design intends to improve its contracts*; and *only executed tests could qualify the new implementation*. None implies the others.

## 1. The problem P01 solves

A Lisp programmer should be able to define a meaningful domain operation (for example, "create a graphics context" or "define a material parameter") in a compact, inspectable form, and have a backend carry it out **without losing control of native handles, initialization failures, callback lifetimes and thread ownership**.

P01 therefore targets three recurring failure modes:

1. **Expressiveness vs. coupling:** macros make a foreign API pleasant to use but can inadvertently capture one concrete engine's global state, internal compiler API and resource model.
2. **Native correctness vs. convenience:** a `with-*` macro can ensure cleanup after successful creation but cannot magically clean up resources acquired before an exception-safe scope begins, prevent use-after-release, or guarantee that native destruction occurs on the correct thread.
3. **Reuse vs. dependency explosion:** a useful resource or DSL pattern should be separately usable without loading a complete 2D/3D/audio/physics platform.

**Success criterion at design level:** the same small P01 contracts can express a mock foreign handle and a real SDL-class handle while preserving consistent lifetime/error semantics. A sophisticated rendering API is neither necessary nor sufficient for this criterion.

## 2. Source lineage: what each source solved, what it did well, and what P01 must improve

All "observed" statements below refer to source/static audits; they are not claims of independently executed runtime behavior.

| Lineage ID | Source, strength and original problem | Evidence of limitation | P01 mechanism / improvement hypothesis | Required falsification test |
|---|---|---|---|---|
| **L01** | **Alien-Works material/shader macros** turn specialized GPU descriptions into Lisp-authored classes, registries and Varjo shader forms. They reduce manual boilerplate and enable domain syntax. | The generated representation is tightly attached to Filament/Varjo and private/internal APIs; a material DSL is not a safe general execution policy. | Make macro expansion yield a checkable, versioned, backend-neutral *descriptor*; backend lowering is an explicit later step. | Expand and validate the same descriptor without installing graphics libraries; reject unsupported target features before native calls. |
| **L02** | **Alien-Works `with-*` and `unwind-protect` wrappers** capture a lexical resource scope and cleanup after normal or exceptional body exit. | Audit 02 documents partial-init leakage paths in host/window startup, plus unsafe allocator behavior. | Acquisition journal, explicit `Owned/Borrowed/Transferred/Released` relationships and ordered reverse cleanup, including failures halfway through construction. | Fault inject failure after every acquisition; assert no live owned resources remain under the mock model. |
| **L03** | **Alien-Works memory/layout wrappers** express typed buffers and fast foreign allocation, addressing performance and FFI boilerplate. | A02-M01 (stack-chunk free), M02 (type descriptor multiplied as a size), M03 (malformed typed macro call). | Independent checked buffer, alignment and bounds contracts; *do not port the custom stack allocator in the baseline*. | Property tests over sizes, offsets, type widths, overflow, zero-length, double release and emitted macro arguments. |
| **L04** | **Alien-Works SDL host** normalizes native windows, controllers and events for Lisp consumers. | It is coupled to the renderer lifecycle; prior audit found startup-failure and X11-specific surface assumptions. | Optional host adapter that negotiates backend/window-system capabilities rather than assuming X11 or any renderer. | Capability rejection on unsupported window backend; partial-startup teardown; X11/Wayland tests only when actual native CI exists. |
| **L05** | **Alien-Works canvas/renderer queues** route work to native contexts with latches and guarded task queues. | Ordering, reentrancy, shutdown and exception interactions remain unqualified; a queue is not a semantic director. | Explicit executor affinity, bounded submission, request IDs, cancellation states and completion receipts. | Wrong-thread rejection, capacity exhaustion, shutdown-race, cancellation-before-start and stuck-task fixtures. |
| **L06** | **Alien-Works callback registry** maps native callback identifiers back to Lisp closures. | Prior audit found no visible synchronization in the shared counter/table; ID reuse and callback lifecycle require qualification. | Generational registration tokens, in-flight accounting, deregistration barrier and a verified callback-thread strategy. | Re-registration/ABA tests, late callback after close, concurrent registration, listener abort and library-unload guards. |
| **L07** | **Trial** models allocation, deallocation, dependencies and graphics-context ownership as explicit resources. | Trial uses Common Lisp/CLOS-specific mechanisms, and context correctness depends on backend/driver contracts; it is not a Gerbil-native drop-in. | Reuse *resource state, dependency and context ownership ideas*, not the CLOS implementation. | Use same state protocol in mock and first native adapter without importing Trial or a CL runtime. |
| **L08** | **CEPL/Varjo** distinguish a Lisp-facing shader authoring language from generated GPU code and API execution. | Their compiler and graphics semantics are Common Lisp/OpenGL-specific; no Gerbil equivalence is demonstrated. | Separate syntax, typed domain IR, validation and backend-specific emission. | Golden IR tests, unsupported-feature errors, deterministic canonical output and no execution during expansion. |
| **L09** | **Gerbil/Gambit FFI** supports `begin-foreign`, `begin-ffi`, `define-c-lambda`, `define-c-struct` and foreign release mechanisms. | FFI presence does not guarantee ABI compatibility, promptly scheduled native destruction or safety when a C callback arrives on an arbitrary native thread. | Narrow FFI layer, ABI manifest, explicit owner-thread destructor paths and safe callback boundaries. | `sizeof`/alignment probes, release-on-owner fixtures and C-to-Gerbil callback-thread validation. |
| **L10** | **Gerbil SDL2 binding specimen** uses Gambit FFI macros and weak-reference/finalizer experiments. | Its README explicitly calls the implementation experimental (2019) with GC integration still ongoing. | Retain it as a syntax/interface reference, not an authoritative ownership implementation. | No source port admitted until an independent backend passes P01's owner/release/callback gates. |

### Pinned representative anchors

- [Alien-Works root system](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/alien-works.asd), [materials and shaders](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp#L418-L655), [engine lifecycle and queues](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/engine.lisp#L55-L120), [memory allocation](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/memory/allocator.lisp), [typed buffers](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/memory/memory.lisp), [callback registry](https://github.com/borodust/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/utils/utils.lisp#L310-L321).
- [Alien-Works A02 static defect register](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/alien-works-common-lisp-source-code-audit-review.md).
- [Trial resource](https://github.com/Shirakumo/trial/blob/5629b23d099df7d682093e56abc270c46340cd4e/resource.lisp), [CEPL lifecycle](https://github.com/cbaggers/cepl/blob/88e9d005e86c54d681b640bbd14f0ef25fb449d4/core/lifecycle.lisp), [Varjo compiler](https://github.com/cbaggers/varjo/tree/68132f727d17d953ae1f7ac5dbc184df83a65f20), [Gerbil SDL2](https://github.com/nymacro/gerbil-sdl2/tree/ed03678de77807146718d4d23397ec12fd593cc0).
- [Gerbil FFI reference](https://gerbil.scheme.org/guide/ffi.html), [Gerbil syntax macros](https://gerbil.scheme.org/reference/gerbil/prelude/macros.html), [Gerbil build tool](https://gerbil.scheme.org/reference/dev/build), [Gambit C interface](https://gambitscheme.org/4.5.1/gambit.pdf).

## 3. Scope, exclusions and trust boundaries

**P01 owns:** foreign type descriptions, ABI checks, resource and borrow semantics, scoped acquisition, checked buffers, native callback lifetime, execution-context affinity, bounded context commands, domain-definition syntax and backend-specific lowering adapters.

**P01 does not own:** general workflow planning, global authority adjudication, the core source vault, arbitrary agent tool policies, DSL bytecode VM, system process supervision, OS deployment, package management, game entities, full shader compilation, remote actors, generic GUI framework, full GPU renderer or physics world.

**Specific language constraint:** the *new* code would be Gerbil. Guix/Guile/Shepherd are **research specimens for another edition**, not dependencies, bridges, deployment adapters or subprocess requirements of P01. A future application may be packaged by Guix without making Guix part of this library's architecture.

Trust boundary distinctions:

1. **Trusted developer macros** may execute at compilation time. A Gerbil macro is **not** a sandbox and shall not consume untrusted executable Scheme source as if it were validated declarative data.
2. **Declarative descriptors** are data. They must be parseable and validated without `eval` or dynamically loading an arbitrary backend.
3. **Native adapters** are effectful trusted code. They run only when invoked through a validated public runtime contract in a process that has deliberately loaded that adapter.
4. **FFI and C** execute with operating-system process permissions. P01's checks prevent misuse of its API but do not provide a process sandbox or privilege separation.
5. **Borrowed foreign handles** must never be confused with owned handles; a finalizer or GC path is not permitted to silently destroy an engine-owned object.

## 4. Architecture and dependency direction

```text
  IMMUTABLE HISTORICAL SOURCE VAULT          (reference only)
  Alien-Works / Trial / CEPL / Varjo / Gerbil SDL2
                         |
                         | lineage + audit + tests
                         v
           P01 GERBIL DOMAIN DEFINITION
    syntax -> descriptor -> validator -> typed IR
                         |
                         | capability-checked lowering
                         v
           P01 GERBIL RUNTIME FACADE
     leases / buffers / scopes / callbacks / executor
                         |
                 backend protocol
                    /         \
               Mock backend    Opt-in SDL-like backend
              (first gate)     (later proof slice)
                    \         /
                  structured result

    Future LFR-Meta <- optional *contract mapping only* -> P01
```

Rules:
- Historical originals and new Gerbil modules never share an implicit runtime.
- P01 can build/test against mock alone, without SDL, GPU, Filament, Guix, Guile, Shepherd, AI/LLM or Common Lisp.
- Native backend modules import P01 contracts; P01 core must **not import a concrete backend**.
- An unqualified backend cannot be selected silently by an installed-but-unused dependency.
- Domain IR must be available independently of the runtime context.

## 5. Proposed module map (names are design placeholders)

```text
editions/p01-native-metaprogramming/
  README.md
  gerbil.pkg
  build.ss
  source-lineage.lock             # source refs, audits and licensing, not vendored by default
  design/
    abi-contract.md
    ownership-contract.md
    domain-ir.md
    adapter-protocol.md
  src/
    core/ids.ss                   # module-local opaque IDs/generation tags
    core/result.ss                # Result/Failure categories
    core/descriptor.ss            # validated domain/native descriptor values
    meta/declare.ss               # trusted Gerbil developer macros
    meta/parse.ss                 # syntax -> declared forms
    meta/ir.ss                    # typed/canonical domain IR
    meta/validate.ss              # validation and feature checks
    meta/lower.ss                 # pluggable lowering protocol, no backend selected
    ffi/abi.ss                    # expected size/alignment/version/feature probes
    ffi/handle.ss                 # opaque owned/borrowed/transferred lease registry
    ffi/scope.ss                  # acquisition journal and reverse cleanup
    ffi/buffer.ss                 # bounds-checked native byte regions/views
    ffi/callback.ss               # registered C callback lifetime & routing
    exec/context.ss              # native context lifecycle & affinity
    exec/queue.ss                # bounded command requests, futures and receipts
    backend/protocol.ss          # backend contract and registration
  backends/
    mock/                         # fake allocation, failure hooks, simulated callbacks
    sdl2/                         # OPTIONAL; only after mock gates and exact ABI review
  examples/
    descriptor-only/              # no native shared library
    mock-window/                  # no window manager dependency
    native-window/                # optional second-stage vertical slice
  tests/
    unit/                         # pure Gerbil properties and macroexpansion snapshots
    mock/                         # lifecycle, failure injection, state invariants
    native/                       # C shim, sanitizers, actual SDL+window tests
  qualification/
    fixtures/
    results/                      # generated, not committed as fabricated proof
```

Directory names, individual modules and proposed function names below are **not an assertion that a working package exists**. Final naming requires inspection against the installed Gerbil release and import/package conventions.

## 6. Public contract sketch

These are **interface signatures and semantic contracts**, not executable Gerbil code.

| Interface | Input | Output | Invariant |
|---|---|---|---|
| `describe-native-type` | type ID, ABI metadata, ownership mode | `TypeDescriptor` or validation failure | No implicit pointer escape, no silent ABI inference |
| `describe-native-operation` | name, arguments, outputs, effect, ABI and backend feature | `OperationDescriptor` | Descriptors contain no code to eval |
| `compile-descriptor` | trusted macro-defined data | canonical typed IR / syntax diagnostic | No runtime native calls during expansion |
| `validate-descriptor` | candidate IR, feature manifest | structured validation | Reject unrepresentable types and missing backend features |
| `open-context` | selected backend, options, owner-thread choice | `ContextLease` or failure | Partially acquired components are rolled back |
| `acquire-resource` | validated operation, parent context | opaque owned `Lease` or failure | One owner and an explicit destructor/affinity contract |
| `borrow-resource` | live lease, scope/parent | `BorrowRef` or failure | Borrow cannot outlive its owner, borrow never destroys |
| `transfer-resource` | owned lease, new owner | new valid owner token / failure | Old owner loses release/use rights once transfer commits |
| `release-resource!` | live owned lease | release receipt / error | At most one API-level destructor submission; native failure is not claimed success |
| `with-native-scope` | acquisitions and body | result plus cleanup receipts | Reverse-order cleanup on success and exceptions, including partial construction |
| `allocate-native-buffer` | count, element layout, alignment | bounded `BufferLease` or failure | Checked multiplication, padding and alignment |
| `slice-native-buffer` | buffer, offset, length | checked view or bounds failure | View generation tied to parent lease and permitted range |
| `register-native-callback` | callback signature and owner | generational callback token | Registration and deregistration are serialized/observable |
| `submit-native-command` | context and validated command | `CommandReceipt/Future` or rejection | Explicit capacity and affinity, no hidden fire-and-forget |
| `close-context!` | context lease | close receipt | Rejects new submissions, drains/cancels appropriately, releases children |

### 6.1 Typed records and stable public fields

`TypeDescriptor`:
- `type_id`, `native_name`, `primitive_or_opaque`, `size_bytes`, `alignment_bytes`, `signedness`, `endianness`, `pointer_tags`, `ABI_version`, `compiler_target`, `ownership_default`, `backend_id`, `source_provenance`.
- A descriptor is invalid if any required measured ABI field is unknown. **Opaque pointer types** may intentionally omit layout but must remain non-dereferenceable in the public API.

`OperationDescriptor`:
- `operation_id`, `input_types`, `output_type`, `effect_tag`, `required_backend_features`, `context_affinity`, `error_contract`, `create/destroy_pair`, `version`, `source_provenance`.
- Effects remain informational within P01 until the host/application supplies an authority decision. P01 must not invent universal authorization on behalf of the future Meta-Foundation.

`Lease` (opaque to clients):
- Internally: `resource_id`, `generation`, `context_id`, `owner_id`, `native_handle`, `destructor_id`, `affinity`, `state`, `parent_id`, `child_count`, `borrow_count`, `provenance`, `diagnostic_info`.
- Externally: only a nonforgeable-by-ordinary-API token, type tag, lifecycle status and safe operations. **Not a security token against malicious code in the same process.**

`Result`/`Failure`:
- `status` (`OK`, `REJECTED`, `FAILED`, `TIMED_OUT`, `CANCELLED`, `UNKNOWN`), `phase`, `operation_id`, `context_id`, `resource_id`, `error_code`, `native_error`, `cleanup_outcome`, `observed_effect`, `evidence_ref`.
- `FAILED` after an effect started does not imply nothing happened. `UNKNOWN` is necessary if native result or partial effect cannot be established.

### 6.2 Ownership state transitions

```text
NEW --acquire()--> ACQUIRING --native success--> OWNED_LIVE
  \                  |
   \--invalid------> FAILED --release all partial children--> CLOSED
                     
OWNED_LIVE --borrow()--> BORROWED_VIEW (dependent, not an owner)
OWNED_LIVE --transfer()--> TRANSFERRED_OLD / OWNED_LIVE_NEW
OWNED_LIVE --release()--> CLOSING --destructor success--> RELEASED
                                   \--destructor fails--> RELEASE_FAILED

CONTEXT: CREATED -> STARTING -> READY -> QUIESCING -> CLOSED
                                           \-> CLOSE_INCOMPLETE
```

`RELEASE_FAILED` must remain in the registry, with error evidence; it must not be marked `RELEASED` and silently retried without an explicit idempotency/double-destruction policy. A process crash can prevent cleanup even when source-level `unwind-protect` is correct.

### 6.3 Critical invariants

**OWN-01:** Each native allocation is assigned exactly one owning lease when creation commits.  
**OWN-02:** A borrowing view has no destruction authority and becomes invalid when its parent generation changes.  
**OWN-03:** A transferred lease cannot be used or released through its former owner token.  
**OWN-04:** A destructor must execute on a backend-compatible thread/context, or return `AFFINITY_VIOLATION` before any effect.  
**OWN-05:** A failure during a multi-step initializer must trigger reverse-order cleanup for every successfully acquired child.  
**OWN-06:** Cleanup errors remain visible and may leave state `CLOSE_INCOMPLETE`; the public API never manufactures a successful receipt.  
**OWN-07:** No native pointer is exposed by the default public API. Any raw-pointer escape hatch is a separate unsafe opt-in and excluded from foundation qualification.  
**OWN-08:** Native libraries are not unloaded while live handles, callbacks or in-flight commands refer to them.  
**OWN-09:** All views and buffers carry bound/context generations, preventing stale reuse following destruction and new allocation at a recycled address.  
**OWN-10:** Cancellation before execution prevents dispatch; cancellation during native execution does not assert the native effect was undone.

## 7. Metaprogramming and the domain IR

### 7.1 Why Alien-Works is the anchor

Alien-Works demonstrates *actual language construction*: `defmaterial` creates domain classes/configurations while `define-fragment-shader` expands Lisp forms into Varjo-directed shader definitions. This is stronger than writing a thin C wrapper: it lets domain authors formulate operations in terms of domain intent. P01 retains **that role of metaprogramming** but rejects mandatory dependence on the original material registry or Filament representation.

### 7.2 Two input tracks, never conflated

**Track A: trusted Gerbil macros** for developers declaring operation families. Gerbil `defsyntax`/`syntax-case` support is documented; exact expansion and hygiene must be tested under the selected compiler version. Developer macros can run compile-time code and therefore require trusted source review.

**Track B: external declarative descriptions** are *restricted data*, accepted by a parser that recognizes an explicit schema and never invokes `eval`, `load`, arbitrary macro expansion or FFI calls.

### 7.3 Proposed compilation path

```text
trusted developer macro / restricted descriptor data
      -> source location + raw description
      -> structural parse
      -> semantic validation (types, lifetimes, effects, feature set)
      -> canonical DomainIR (stable schema version and digest)
      -> optional backend lowering (C shim wrapper, SDL type, later graphics)
      -> generated invocation adapter
      -> separately authorized runtime call
```

A domain descriptor and its generated adapter are **different artifacts**. The former can be inspected and compared without a native backend; the latter must be built for a measured ABI.

### 7.4 One illustrative specification (NON-EXECUTABLE pseudocode)

```scheme
;; Intentional pseudo-Scheme; not Gerbil API that currently exists.
(native-operation 'create-window
  (backend 'sdl2)
  (inputs '(title:utf8 width:u32 height:u32))
  (result 'WindowHandle)
  (ownership 'owned)
  (destroyer 'destroy-window)
  (affinity 'ui-context)
  (effects '(native-allocate native-window))
  (requires '(video-initialized))
  (failure 'SDL-Error))
```

Expected compiler behavior: emit a typed `OperationDescriptor`, reject an absent destroyer for an owned handle, reject invalid dimensions at the boundary, fail cleanly when `sdl2` is not available, and create no window at compile time. No claim of syntax compatibility is made.

### 7.5 IR constraints

- Canonical, versioned and deterministic serialization for defined inputs; preserve source-span provenance.
- No arbitrary executable closures embedded in portable descriptors.
- Explicit backend feature requirements, ownership rules and effect metadata.
- Compile-time errors preserve original domain-source location and violated contract.
- A transformation pass records input/output IR hashes; a changed backend lowering does not silently rewrite upstream semantics.
- Version compatibility is explicit (`major.minor` or declared migration), never inferred from the existence of a symbol.
- Shader or material-specific DSL lowering is an **optional later demonstration**; a full GLSL compiler is not required for P01-D01 acceptance.

## 8. Native ABI and memory boundary

### 8.1 The actual Gerbil mechanism

The official Gerbil reference delegates its foreign C interface to Gambit, notably through `begin-foreign`, `:std/foreign`/`begin-ffi`, `define-c-lambda` and `define-c-struct`. Gambit foreign objects support tagged pointers and release functions; explicit foreign release exists, but such features do **not** establish that a GPU object may safely be destroyed by a GC finalizer on an arbitrary thread.

### 8.2 ABI manifest and checks

Per backend/target, record C compiler, target triple, native library version/build ID, pointer width, relevant `sizeof`/`alignof` values, calling conventions, enum widths, struct-layout/field-offset probes and required native features. Layout probes should be **generated from authoritative installed C headers and compiled by the backend's own build**, not guessed from Scheme bytevector sizes.

Boot-time or build-time validation fails closed on a mismatch for dereferenced structures. Opaque pointers may bypass layout measurement only when their operation contracts never expose or interpret the underlying representation.

### 8.3 Buffers

- Validate `count`, `element_size`, `alignment`, total byte size and offset arithmetic against overflow.
- Distinguish element count, byte offset, byte length and capacity; never overload a numeric field.
- Explicitly specify whether copied/pinned data remain valid for an asynchronous call; do not promise zero-copy until proven.
- Buffer views require a live parent generation and cannot alias bytes outside a checked range.
- Native writes must return actual bytes/elements written and propagate short writes, not requested counts.
- Do **not** port Alien-Works stack allocator or typed memory-vector macro as the first implementation.

## 9. Callbacks and threading

### 9.1 Registrations

A native callback registration is represented by `(callback_id, generation, backend_id, parent_context_id, affinity, active, in_flight_count)`. A returned token is invalid after deregistration, context generation change or library unload. An ID alone is insufficient because it can be reused.

### 9.2 Unregistration barrier

`unregister-callback!` must stop new callback deliveries, acknowledge the backend's unregister operation, and wait/observe completion of any in-flight handler **when the backend offers that guarantee**. Otherwise the API returns an explicitly weaker `QUIESCENCE_UNVERIFIED` result and delays final library teardown. It must never claim native quiescence merely because an entry was deleted from a Scheme hash table.

### 9.3 Native thread ingress

**Default assumption:** a callback can arrive on a native thread not owned by the Gerbil scheduler. Unless the specific Gambit/foreign callback API and native library prove this supported, the C shim should capture minimal POD events into a bounded, synchronized queue, and a Gerbil-owned context thread should poll/drain them. The shim must define overflow behavior, memory ordering, lifetime and event-copy semantics.

Synchronous native APIs that require an immediate callback return value **cannot automatically use this asynchronous approach**; classify them as `UNSUPPORTED_UNTIL_THREAD_BRIDGE_QUALIFIED` rather than improvising a call into Scheme from unknown native threads.

### 9.4 Executor model

Each effectful context has a designated owner/executor. Commands are submitted with request ID, operation ID, typed args, deadline, cancellation token, optional idempotency key and expected context generation. The executor serializes native context operations where required, provides a bounded queue and returns structured completion receipts. Gerbil actors are a possible implementation tool, **not an authorization or thread-affinity guarantee by themselves**.

States: `QUEUED → DISPATCHED → COMPLETED|FAILED|UNKNOWN`, and `QUEUED → CANCELLED`; after `DISPATCHED`, cancellation becomes a request, not a guarantee of rollback. Shutdown has a quiescing stage that rejects new commands and resolves or reports remaining work.

**No universal real-time guarantee.** Deadline semantics and interruptibility depend on the actual native operation; a noninterruptible GPU or C call may outlast the requested deadline.

## 10. Backend profile strategy

**Profile M: Mock (required).** Pure Gerbil, deterministic fake IDs/handles, configurable failure at each acquisition stage, simulated callbacks, stale-generation reuse, queue capacity, explicit resource census. Must be the default profile for tests and build examples.

**Profile N1: One small native backend (conditional).** An SDL2-class window/input context is a reasonable *candidate*, because it tests exactly the host/callback/lifecycle concerns seen in Alien-Works. But first validate current SDL2 availability, binding license, Gambit version, ABI and GUI driver choice. This could be replaced with a smaller standalone C test library if SDL imposes too much environmental coupling.

**Profile N2+: Later optional candidates.** Filament, Skia, OpenAL, PhysX, GLSL material lowering, image/audio/asset tools. **No import, schedule or compatibility promise** in this design. Each would require its own ABI profile and runtime qualification. Their presence in Alien-Works does not justify admission.

**Profile X: Unsafe raw pointer API.** Not present in the default edition; if research later requires one, isolate it under a separately named experimental module with no guarantee of the safe contract.

## 11. Error, effect and recovery semantics

Every public effectful operation returns a typed outcome whose `phase` is one of `DESCRIBE`, `VALIDATE`, `PREPARE`, `ACQUIRE`, `EXECUTE`, `RELEASE`, `RECONCILE`. A native error contains library/API identifier, portable category, raw native code/text (when available), context ID, monotonic observation time and source operation reference. Secrets and pointer addresses are not logged by default.

Failure classifications:

| Class | Meaning | Allowed claim |
|---|---|---|
| `CONTRACT_FAILURE` | Invalid descriptor, ownership or buffer bound | Rejected before native side effect |
| `ABI_MISMATCH` | Measured native type/version contract differs | Native operation not dispatched |
| `ACQUIRE_FAILURE` | Native constructor failed | Partial acquisitions were subjected to documented cleanup; cleanup outcomes recorded |
| `AFFINITY_VIOLATION` | Call attempted on wrong owner context | No dispatch by the guarded API |
| `CALLBACK_UNSAFE` | Unknown callback/thread/lifetime state | No false claim of safe callback release |
| `QUEUE_FULL` | Executor capacity reached | Rejected or bounded wait according to explicit policy |
| `NATIVE_FAILURE` | Native operation returned failure | Effect may have partially occurred |
| `DEADLINE_EXCEEDED` | Deadline elapsed | Actual native completion/cancellation might remain unknown |
| `RELEASE_FAILED` | Destructor failed or could not be verified | Resource remains unresolved; no success receipt |

A transaction-like scoped API is **not** an OS transaction. Rollback means compensating destruction of known acquired resources under backend rules; it cannot reverse irreversible external effects, device state or a crashed process.

## 12. Specific tests and acceptance gates

All tests below are **future acceptance criteria**, not performed results. Every test must return nonzero on failure and include a source/build revision in its receipt.

| ID | Test / stimulus | Required evidence |
|---|---|---|
| **P01-T01** | Build descriptor-only + mock profile with only Gerbil/Gambit toolchain | No SDL, Filament, Guix, Guile, CL runtime, LLM or GPU dependency |
| **P01-T02** | Macroexpansion snapshots across supported compiler version | Hygienic bindings, correct field mapping, useful source-position errors |
| **P01-T03** | Unsupported domain/backend feature | Compile/validation rejection before any native activity |
| **P01-T04** | Canonical IR for identical descriptor inputs | Bitwise-stable serialized IR and identical digest within pinned toolchain |
| **P01-T05** | Fail during acquisition at each step of a three-resource initializer | Reverse cleanup of every successfully owned child; unresolved releases reported |
| **P01-T06** | Double release, borrow after release, old-owner use after transfer | Safe rejection; no second native destructor dispatched |
| **P01-T07** | Reuse a freed native address with a new generation | Stale token rejected (ABA prevention) |
| **P01-T08** | Wrong-thread/context destructor | Refusal or guaranteed owner-dispatch; never silently execute on wrong thread |
| **P01-T09** | Buffer overflow, negative count, invalid alignment, out-of-bounds slice | Rejected before memory read/write |
| **P01-T10** | Simulated short write/read and truncated input | Exact transferred-count semantics, no falsely reported completion |
| **P01-T11** | Two concurrent callback registrations; reuse callback ID after unregister | Correct tokens, no cross-target delivery |
| **P01-T12** | Callback races with shutdown and later C delivery | No invocation of released closure; quiescence unresolved until proven |
| **P01-T13** | Full queue, cancelled queued command, repeated request ID | Bounded rejection or explicit wait; correct cancellation/idempotency semantics |
| **P01-T14** | Context closes with outstanding child leases and queued commands | Quiescing and explicit leak/unresolved-resource report |
| **P01-T15** | Native C shim compiled with altered struct layout or enum width | ABI mismatch detected before dereference |
| **P01-T16** | Native library missing/incompatible | `BACKEND_UNAVAILABLE`; mock and descriptor-only imports still load |
| **P01-T17** | Native allocation/release under fault injection and sanitizers | No leaks/double-frees in the specific tested path, with evidence artifact |
| **P01-T18** | Linux backend selection under X11 vs Wayland vs headless | Explicit feature negotiation/failure; no blind X11 pointer interpretation |
| **P01-T19** | Macro-controlled DSL emits descriptors without native backend | Source-level authoring demonstrably independent of runtime initialization |
| **P01-T20** | Two unrelated mock native APIs use same owner/buffer/result machinery | No edits to P01 core contracts required by the second consumer |
| **P01-T21** | GC finalization, context owner shutdown, native thread callback | Destructors follow verified affinity policy or operation stays unqualified |
| **P01-T22** | License and provenance check before redistributing any upstream code | Attribution and explicit reuse rights, or pointer-only preservation |

### Gates (strictly within P01)

- **G0 / Preserve:** Pinned upstream specimen, license and audit lineage registered. No copied code until per-file rights are reviewed.
- **G1 / Descriptor:** Pure macro/IR and validation tests T01–T04, T19 pass without native packages.
- **G2 / Mock lifetime:** T05–T14, T20 pass with deterministic mock fixtures.
- **G3 / ABI:** Target-specific probes and incompatible-layout tests T15–T16 pass.
- **G4 / Native lifecycle:** T17, T18, T21 run with a real backend and report exact toolchain/OS/lib versions. No global portability claim.
- **G5 / Domain reuse:** A second independent native API can consume the same descriptor, resource and effect contracts without modification to their public semantics.

Do not mark G4/G5 PASS merely on a successful `gxc` build or a screenshot of a window.

## 13. Evidence ledger for P01 qualification

Each design or later implementation decision receives an entry:

```text
P01-<ID>
  source: original-repo@SHA/path#lines
  original_problem: ...
  original_mechanism: ...
  observed_strength: ...
  source_defect_or_limit: ...
  proposed_gerbil_semantics: ...
  portability_difference: ...
  license_status: ...
  test_id: ...
  result: NOT_RUN | PASS | FAIL | INCONCLUSIVE
  exact_build_abi_environment: ...
  disposition: DESIGN_ONLY | IMPLEMENTED_UNQUALIFIED | QUALIFIED_FOR_TARGET
```

**Evidence-grade discipline:** Source demonstrates presence of mechanism; a mock test establishes only the mock contract; ABI tests establish only one ABI/target; native integration tests establish only their measured runtime case. Performance must be benchmarked, and a performance-first upstream README is not performance evidence.

## 14. Explicitly rejected approaches

- **Do not** fork the entire Alien-Works ASDF graph as the foundation runtime.
- **Do not** transliterate `defmaterial` and CFFI code into Gerbil while assuming semantics or dependency compatibility.
- **Do not** use automatic GC finalizers as the sole destruction mechanism for thread-bound native engine objects.
- **Do not** port the known-broken stack allocator into the first safe buffer abstraction.
- **Do not** presume a render task queue implements state/event/director semantics.
- **Do not** load Filament, Skia, PhysX, audio or GUI stacks into descriptor-only/core imports.
- **Do not** treat Gerbil actor administration as a security boundary against same-process arbitrary code.
- **Do not** make Guile, Guix or Shepherd mandatory dependencies; their architecture belongs to the P02 lineage study.
- **Do not** introduce a general-purpose custom VM, runtime Lisp `eval` of untrusted DSL source or a universal device scheduler without a proven need.
- **Do not** declare safety/portability from source inspection alone.

## 15. Sequencing within P01 only

1. **P01-D01** (this artifact): approve or amend the explicit problem, source lineage, contracts and exclusion boundaries.
2. **P01-D02, if authorized later:** resolve the unsettled ABI/ownership/callback choices into an exact API and test fixture specification for the pinned Gerbil/Gambit toolchain. This is still **design**, not coding.
3. **P01-I01, only with separate authorization:** implement descriptor-only Gerbil modules plus pure validation tests.
4. **P01-I02:** implement mock leases, scoped cleanup, buffers, command receipts and deterministic fault injection.
5. **P01-I03:** select and qualify exactly one real native backend/target, ideally an SDL-class or simple dedicated C shim.
6. **P01-Q01:** independently run negative tests, dependency-isolation checks and lifecycle diagnostics; publish qualified/failed verdicts with evidence.

No other specialized edition, Meta-Foundation implementation or new repository mutation is part of this sequence. Once P01 has real contracts, the programme may later *compare* them with unrelated editions before promoting a shared abstraction into K0.

## 16. Open design decisions (not hidden assumptions)

| Decision | Current default hypothesis | Evidence necessary before locking |
|---|---|---|
| Which native backend first? | Minimal C mock/fixture, then optional SDL-class slice | Target platform and installed header/ABI availability, test cost |
| Foreign ownership mode? | Explicit owned leases, scoped borrowed views | Gambit release behavior and native-destructor thread requirements |
| Callback ingress? | C shim copies POD into bounded queue; Gerbil owner drains | Native API synchronous-callback requirements and memory-order tests |
| Compiler staging? | Canonical declarative IR before any native codegen | Gerbil macroexpansion fixtures and source-location propagation |
| Error representation? | Typed result/phase, no parsing stderr strings | At least two backend examples validating distinctions |
| License and copying? | Source links/locks by default; reimplement independently | Per-component license review, especially third-party bindings/Varjo |
| First Linux execution platform? | Guix-managed x86_64 GNU/Linux as *test environment*, not dependency | Gerbil/Gambit version, C toolchain, display backend and CI reproducibility |
| Cross-edition K0 contract? | Only a projection of ID/effect/result/resource semantics | At least one other domain edition independently demonstrating commonality |

## 17. Final disposition

`P01_D01_DETAILED_DESIGN_PROPOSED / ALIEN_WORKS_LINEAGE_EXPLICIT / GERBIL_NATIVE / MOCK_FIRST / FFI_OWNERSHIP_UNQUALIFIED / OTHER_EDITIONS_UNTOUCHED / DOCUMENT_PUBLISHED`

**P01 should inherit Alien-Works' capacity to *express native domain capabilities using Lisp metaprogramming*, while addressing exactly the failure surfaces that constrain its reusable architecture: coupling, resource ownership, partial initialization, typed memory, callback lifetime, ABI and execution-context affinity.** The proposed deliverable is one independently qualified specialized edition, not a new universal engine and not a claim that existing source can be copied into Gerbil unchanged.