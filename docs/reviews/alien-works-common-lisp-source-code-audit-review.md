# Alien-Works: Comprehensive Common Lisp Source Code Audit & Review

**Audit type:** Direct static source-code review, implementation correctness and subsystem evaluation  
**Reviewed repository:** `vietcong73/alien-works`  
**Report branch:** `audit/lisp-foundation-reserve`  
**Immutable code baseline:** `86381f8bce44a396347cbe8de70349be5b43ece1` (May 1, 2023)  
**Report date:** November 12, 2026  
**Evidence scope:** GH-DC01 (historical GitHub dependency reconnaissance), 02 (core source), 03 (subsystems), 05 (closure), plus direct source re-inspection for this review.  
**Explicit exclusion:** Audit 04 and all isolated hybrid-architecture concepts.  
**Qualification boundary:** No full ASDF build, SBCL runtime integration, native-library qualification, sanitizers or benchmarks were executed.

## 1. Executive technical verdict

**Alien-Works is a technically ambitious Common Lisp framework with substantial strengths in metaprogramming, native-library integration, domain-specific interfaces and graphics-oriented resource management. Its implementation quality, however, is uneven.** Several defects affect the very abstractions intended to make higher-level subsystems reliable: stack allocation, typed memory vectors, stream contracts, codec framing and failure-path ownership.

The significant distinction is between **sound architectural ideas** and **verified implementation correctness**. A well-designed abstraction cannot be called reliable merely because it reduces boilerplate; its allocation, error, cleanup, data transfer and concurrency contracts must also hold.

**Disposition:** `LISP_SOURCE_REVIEW_COMPLETE / MATERIAL_CORRECTNESS_DEFECTS_FOUND / EXECUTABLE_QUALIFICATION_OPEN`.

- **77 Lisp source files** were inventoried in `src/` and `tools/`, **not** individually runtime-tested.
- This review tracks **12 principal correctness findings (C01–C12)**. Some are demonstrated from source and contracts; others require a specified invocation to expose them.
- It separately tracks **13 conditional or integration risk findings (R01–R13)**. These must not be misrepresented as observed crashes.
- A reproducible build and complete native dependency closure have **not** been established. This does not establish that Alien-Works can never build.

## 2. What the Common Lisp implementation does well

### 2.1 Lexical native-resource scopes

[`src/math/vec3.lisp:21–39`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/math/vec3.lisp#L21-L39)

`with-vec3` wraps creation and destruction using `unwind-protect`. This expresses an important foreign-resource protocol in an idiomatic Lisp macro and reduces repeated cleanup code. The mechanism protects the body **after successful construction**, not partial allocations made *inside* a failing constructor. The latter requires internal cleanup discipline.

### 2.2 CLOS builders and declarative DSLs

[`src/graphics/engine.lisp:165–291`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/engine.lisp#L165-L291) · [`src/graphics/material.lisp:413–510`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp#L413-L510) · [`src/graphics/material.lisp:625–655`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp#L625-L655)

Builder option generics and methods make Filament's builder model easier to use from Common Lisp. `defmaterial` and `define-fragment-shader` do considerably more than provide foreign calls: they define a symbolic material/shader authoring language through Lisp macros, CLOS classes and Varjo integration.

This is a concrete strength, although these DSLs are coupled to the Filament/Varjo representation and cannot be classified as backend-neutral without further refactoring and tests.

### 2.3 Structured foreign-library facades

GLM, Filament, PhysX, OpenAL, Opus, Skia and SDL are exposed through distinct Lisp namespaces and wrappers. That organization limits raw foreign API details in higher-level source. However, Lisp type declarations and FFI signatures alone do **not** prove buffer lifetime, pointer ownership, concurrency safety or ABI compatibility.

### 2.4 Buffering and execution scheduling

[`src/graphics/surface.lisp:4–47`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/surface.lisp#L4-L47) · [`src/graphics/canvas.lisp:178–204`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/canvas.lisp#L178-L204)

Triple-buffered values, reusable draw-command arrays, fill-pointer management, guarded work queues and asynchronous canvas processing demonstrate deliberate attempts to reduce allocations and separate production from consumption.

**Audit correction:** `atomics:cas` has library-specific success semantics. The `loop until (atomics:cas ... nil t)` spin lock must not be declared inverted merely by assuming a different compare-and-swap return convention. Busy spinning and fairness deserve separate analysis.

## 3. Principal source-code correctness findings

The following severity ratings represent potential impact, **not measured frequency**. "Source-demonstrated" means a direct code contradiction or clear counterexample is present. "Invocation-dependent" identifies an operation that fails under a specified condition. No finding below implies native execution was observed.

### 3.1 Memory subsystem

**C01 — Stack free reads head as tail (HIGH; source-demonstrated).**  
[`src/memory/allocator.lisp:81–97`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/memory/allocator.lisp#L81-L97)

`stack-chunk-free` binds both `head-address` and `tail-address` to `(stack-chunk-head chunk)`. Suppose a block starts at address 1000 and a 16-byte allocation advances the head to 1016. Freeing the returned pointer 1000 triggers the check `1000 < 1016` and rejects a valid allocation. This breaks normal LIFO deallocation semantics. The corresponding code should use the actual tail field, followed by tests of alignment, underflow, chunk transitions and nested allocation/free sequences.

**C02 — Foreign-type designator used as a numeric byte size (HIGH; source-demonstrated in ordinary function body).**  
[`src/memory/memory.lisp:79–92`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/memory/memory.lisp#L79-L92)

The ordinary `make-memory-vector` body computes `(* length (find-foreign-type type))`. `find-foreign-type` returns a type designator such as `:uint8`, not a byte count. A compiler macro uses `cffi:foreign-type-size` instead, potentially masking the defect in compiled paths. Ordinary and optimized paths must not disagree about fundamental correctness.

**C03 — Typed allocation macro constructs an invalid call (HIGH; macroexpansion-demonstrated).**  
[`src/memory/memory.lisp:113–117`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/memory/memory.lisp#L113-L117)

`with-memory-vector` adds a quoted type as a positional argument to `make-memory-vector`, whose signature expects `&key type`. A typed invocation such as `(with-memory-vector (buf 4 :uint32) ...)` does not expand to the required `:type :uint32` keyword pair.

### 3.2 Host streams and Lisp-facing API contracts

**C04 — Input stream `read-sequence` returns byte count rather than final destination index (HIGH; source-demonstrated).**  
[`src/host/host.lisp:110–130`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L110-L130)

When `:start 5` and three elements are successfully read, Common Lisp `read-sequence` should return 8. The method returns 3. This breaks callers that use the result as a sequence boundary.

**C05 — Absolute file-position setter performs a relative seek (HIGH; source-demonstrated).**  
[`src/host/host.lisp:133–139`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L133-L139)

The method for `(setf gray:stream-file-position)` passes SDL `RW_SEEK_CUR`. Setting file position to zero from a nonzero offset thus does not rewind; this contradicts the intended absolute-position operation.

**C06 — Invalid SDL stream object may survive constructor failure (HIGH; failure-path conditional).**  
[`src/host/host.lisp:86–100`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L86-L100)

On a null SDL handle, initialization calls `signal` with a `file-error` rather than unconditionally exiting via `error`. In the absence of a nonlocal exit by a handler, construction can continue with a null handle. The `close` branch also couples freeing its buffer to the existence of `handle`. This is a direct failure-path design defect; the precise outcome depends on the condition handler and construction state.

**C07 — Bounded output string method performs arithmetic on omitted optional arguments (HIGH; invocation-dependent).**  
[`src/utils/utils.lisp:435–441`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/utils/utils.lisp#L435-L441) · [`src/framework/framework.lisp:4–17`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/framework/framework.lisp#L4-L17)

`gray:stream-write-string` computes `(- end start)`, although both `start` and `end` are optional and may be NIL. Ordinary calls with omitted bounds can fail. The framework's exception-reporting path uses a bounded wrapped output stream, so an error-reporting path can itself encounter a stream error.

### 3.3 API and asset pipeline

**C08 — Controller button count calls axis-count API (MEDIUM; source-demonstrated).**  
[`src/host/host.lisp:831–832`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L831-L832)

`game-controller-button-count` dispatches to a joystick axis count rather than a button count. Consumers can receive a plausible but semantically wrong integer.

**C09 — Allocator macro export does not match its definition (MEDIUM; source-demonstrated).**  
[`src/memory/packages.lisp`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/memory/packages.lisp) · [`src/memory/allocator.lisp`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/memory/allocator.lisp)

The package exports `with-system-memory-allocator` while the implementation defines `with-system-allocator`. Verify the intended public name and migrate or alias deliberately.

**C10 — Scene importer calls image loader with wrong arity (HIGH; source-demonstrated for image-bearing scenes).**  
[`tools/resources/scene/scene.lisp:68–81`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/tools/resources/scene/scene.lisp#L68-L81) · [`tools/resources/image.lisp:4–5`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/tools/resources/image.lisp#L4-L5)

`parse-scene` evaluates `(load-image image full-path)`, but `load-image` accepts exactly one argument, `path`. A scene with image references reaches an incompatible call.

### 3.4 Opus codec

**C11 — Encoder produces an extra silent frame after a complete final frame (HIGH/MEDIUM; control-flow demonstrated).**  
[`src/audio/opus/opus.lisp:152–189`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/audio/opus/opus.lisp#L152-L189)

The `encode-audio` loop encodes a frame before testing `while (= samples-read frame-size)`. After an exact multiple of frame-size samples, the next read returns zero at EOF, the remaining frame buffer is zeroed, and a silent frame is still encoded. Empty input also reaches a zero-filled encode attempt. This affects the output framing/duration semantics, subject to encoder behavior.

**C12 — Packet decode compares final sequence index against byte count (MEDIUM; invocation-dependent).**  
[`src/audio/opus/opus.lisp:81–106`](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/audio/opus/opus.lisp#L81-L106)

`decode-packet` compares `read-sequence`'s result with `packet-len`. For a nonzero destination `start`, the result is a final index, not the number of bytes read. This can incorrectly flag a fully read packet as malformed.

## 4. Conditional correctness, lifecycle and integration risk register

These are **separate from C01–C12**. Some expose definite missing checks; consequences such as deadlock, leaks or corruption require further environmental conditions or runtime evidence.

| ID | Area | Source-based concern | Status |
|---|---|---|---|
| **R01** | [Host output](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L151-L174) | SDL `r-wwrite` return value is ignored for both single and sequence writes; short writes may be reported as success. | Conditional I/O failure |
| **R02** | [Bounded input](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/utils/utils.lisp#L361-L364) | Requested bytes are charged to the read quota before the wrapped stream's actual final index is known. Early EOF can consume too much quota. | Source-visible conditional |
| **R03** | [Bounded input](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/utils/utils.lisp#L367-L384) | Reaching the artificial bound returns NIL from byte/char reader rather than the `:EOF` marker used for underlying EOF. | Source-visible API inconsistency |
| **R04** | [Host file offsets](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L230-L249) | Requested file offset is reduced modulo positive file length. Out-of-range offsets can silently wrap rather than fail; policy is undocumented. | Behavioral contract unresolved |
| **R05** | [Opus header](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/audio/opus/opus.lisp#L13-L23) | `decode-packet-length` does not validate the result of a short `read-sequence` before interpreting subsequent length bytes. | Truncated-input risk |
| **R06** | [OpenAL](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/audio/openal/openal.lisp#L4-L21) | Audio device may remain open if context construction fails before cleanup protection begins. | Native failure-path leak risk |
| **R07** | [Canvas worker](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/canvas.lisp#L408-L433) | If worker initialization fails before opening `ready-latch`, creating thread can wait indefinitely. | Conditional hang risk |
| **R08** | [Filament release callbacks](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/filament/texture.lisp#L175-L208) | Global pointer-keyed callback table is mutated without visible synchronization; correctness depends on native callback threading and lifetime guarantees. | Thread/ownership unqualified |
| **R09** | [Material listeners](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/graphics/material.lisp#L13-L34) | Listener callbacks run while a guarded registry reference is held. Reentrant registration/removal can deadlock under non-reentrant locking. | Conditional reentrancy |
| **R10** | [Window lifecycle](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/host.lisp#L386-L462) | Some initialization occurs before full cleanup protection; root-window/context teardown and null checks require verification on partial failures. | Native lifecycle unqualified |
| **R11** | [Linux window surface](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/host/system/linux.lisp#L7-L13) | Native window extraction explicitly uses SDL's X11 union member; native Wayland path is not implemented here. XWayland may work. | Portability limitation |
| **R12** | [Material serialization](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/tools/graphics/filament/material.lisp#L11-L25) | Native material input receives `(length source)` as size, which counts Lisp characters, not necessarily UTF-8 bytes. | Non-ASCII input risk |
| **R13** | [Generic foreign callbacks](https://github.com/vietcong73/alien-works/blob/86381f8bce44a396347cbe8de70349be5b43ece1/src/utils/utils.lisp#L310-L321) | Global callback IDs and hash table have no visible synchronization; uninvoked callbacks may accumulate. | Concurrency/lifecycle unqualified |

**Important negative findings:** No assertion is made that R08/R13 were observed racing, that R10 leaked resources in a measured run, or that R11 prevents operation through XWayland. The source establishes the design assumptions and plausible failure modes, not their runtime incidence.

## 5. Subsystem-by-subsystem code review

| Subsystem | Confirmed implementation strength | Main problem | Current disposition |
|---|---|---|---|
| `utils` | Macros, enums, stream and callback adapters | Gray-stream contract defects; global callbacks | **ADAPT AFTER CORRECTION** |
| `memory` | Typed layouts, lexical allocation protocols | Critical allocator and typed-constructor defects | **BLOCK DIRECT REUSE** |
| `math` | Vector/matrix lexical interfaces, GLM operations | Native GLM/FFI coupling; numeric semantics not qualified | **OPTIONAL NATIVE ADAPTER** |
| `host` | SDL window/input/I/O facade | Read/seek/write contracts; failure-path handling | **CORRECT FIRST** |
| `framework` | Scoped composition of window/audio/graphics | Monolithic runtime startup; inherited lifecycle issues | **APPLICATION-SPECIFIC** |
| `graphics/filament` | CLOS builders, typed façade for GPU objects | Callback lifetime and ownership unverified | **OPTIONAL BACKEND** |
| `graphics/skia`, `canvas`, `surface` | Reusable commands, triple buffering, async drawing | Worker lifecycle and synchronization | **AUDIT FURTHER** |
| `graphics/varjo`, materials | Genuine Lisp-based shader/material DSL | Strong coupling to shader/backend internals | **DOMAIN-SPECIFIC** |
| `audio/openal`, `opus` | Audio context scopes and codec integration | Device cleanup, EOF and packet framing | **CORRECT FIRST** |
| `physics/physx` | Native physics façade | Native dependence; main physics entry is demo-oriented | **OPTIONAL BACKEND** |
| `tools/resources` | Assimp, glTF, image/mesh handling | Image loader arity and data boundaries | **CORRECT FIRST** |
| `tools/ui` | ImGui inputs, touch emulation and helper layer | Depends on host + Filament internals | **OPTIONAL TOOLING** |

"Optional backend" is a source classification, **not an assertion of runtime readiness**. The reserve decision is not part of this report.

## 6. GH-DC01 — Historical GitHub Dependency Reconnaissance

This title deliberately distinguishes the **GitHub-published then deleted** dependency reconnaissance from the later locally produced source and subsystem audits.

**Historical audit file commit:** `4a1182d29cb2deb2340dc1f455b3f59b6cebaa53`  
**Subsequent deletion commit:** `b6cbed58d7a285cc59871a5fdce009241f16ecd2`  
**History:** [GH-DC01 commit](https://github.com/vietcong73/alien-works/commit/4a1182d29cb2deb2340dc1f455b3f59b6cebaa53). The original inventory is intentionally **not restored as an active branch file**.

The core `alien-works.asd` declares approximately **23 effective direct dependency entries per platform** when selecting one platform OpenGL alternative. These include `alien-works-foundation`; `aw-sdl`, `aw-glm`, Filament, PhysX, Chipmunk, Skia, OpenAL and Opus wrappers; FFI and generic Common Lisp libraries. The tools system adds further dependencies including foundation/tools, Filament tools and Assimp.

The expected `alien-works-foundation` repository was not accessible through the checked GitHub path. The historical `aw-projects` submodule snapshot is dated 2021 and does not itself lock a 2023-compatible dependency graph. Generated binding sets differ by platform and generation date. Native artifacts, ABI compatibility, complete transitives and execution in a pinned environment remain unverified.

**GH-DC01 verdict:** `STATIC_DEPENDENCY_RECONNAISSANCE_COMPLETE / EXECUTABLE_CLOSURE_NOT_ESTABLISHED`. No claim is made that a successful build is impossible.

The root ASDF's source modules are separated by directory, but it still declares graphics, audio, physics and other native wrappers as part of the primary system. **Source modularity does not equal independently loadable minimal profiles.**

## 7. What the evidence does *not* establish

- The audit does **not** demonstrate a successful `(asdf:load-system :alien-works)` or application startup.
- It does **not** prove all 77 files independently compile or execute.
- It does **not** prove that every conditional risk causes failure in practice.
- It does **not** measure performance, numerical accuracy, driver behavior or native memory safety.
- It does **not** establish that FFI, dynamic binding or macro complexity is inherently defective.
- It does **not** establish a release-ready patch set or authorize any code changes.
- It does **not** settle whether to streamline, selectively reuse or rebuild.

The audit is strongest where an actual source expression contradicts a specified API or control-flow contract, particularly C01–C05, C07, C08–C12. For native callbacks and resource failures, assertions are deliberately narrower.

## 8. Priority of engineering verification

1. **Memory correctness:** Fix and test stack allocation/free ordering, reset conditions, alignment, typed sizes and macroexpansion behavior.
2. **Stream contract conformance:** Verify nonzero `:start`, absolute seek, EOF handling, short reads/writes and error paths with concrete Gray-stream tests.
3. **Codec correctness:** Test Opus empty input, exact frame multiples, truncated packet headers, nonzero destination offsets and round-trip data length.
4. **Resource and concurrency safety:** Fault-inject SDL/OpenAL/Skia/Filament initialization failures, callback execution, reentrant listener changes and shutdown ordering.
5. **Reproducible minimal integration:** Resolve dependency versions and qualified ABI/native libraries on a specified target platform before claiming runtime readiness.

This is a **qualification priority list**, not a claim that any of these repairs have been implemented or authorized.

## 9. Final assessment

Alien-Works uses Common Lisp in technically substantive ways: macros, CLOS, compiler macros, condition/restart facilities, Gray streams, dynamic bindings and native libraries are all given real work. The strongest design decisions deserve recognition.

The most consequential problem is **not a lack of abstractions**. It is that several attractive abstractions are implemented less reliably than their contracts require, especially around memory, I/O, framing, cleanup and thread boundaries.

**Final audit disposition:** `SOURCE_SPECIMEN_ACCEPTED / MATERIAL_LISP_DEFECTS_CONFIRMED / DIRECT_TRUSTED_CORE_REUSE_NOT_QUALIFIED`.

This is a bounded static review of the original Lisp source and its declared dependencies. It excludes Audit 04 completely and makes **no architecture-selection decision**.
