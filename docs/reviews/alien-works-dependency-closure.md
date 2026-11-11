# Alien-Works Dependency Closure Inventory (DC-01)

**Status:** STATIC_DEPENDENCY_INVENTORY_COMPLETE / EXECUTABLE_CLOSURE_BLOCKED  
**Audited repository:** `vietcong73/alien-works`  
**Audit branch:** `audit/lisp-foundation-reserve`  
**Audited baseline:** `borodust/alien-works@86381f8bce44a396347cbe8de70349be5b43ece1` (2023-05-01)  
**Assessment date:** 2026-11-12  
**Scope:** Dependency and provenance closure **only**, not a build, host lifecycle, memory-safety, subsystem, or performance audit.

## 1. Decision

The source-level dependency inventory can be traced for the principal public wrapper repositories, but an **executable, reproducible dependency closure is not established**.

Do **not** claim that the audit fork can be built from the currently accessible sources without further work. Do **not** use the 2021 `aw-projects` pins as though they were a lockfile for the audited 2023 Alien-Works baseline.

## 2. Root ASDF systems

Source: [alien-works.asd](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/alien-works.asd)

### `:alien-works` core

The `:alien-works` system declares **23 effective direct dependency entries per platform**, counting only one of the `cl-opengl` / `cl-opengl/es2` feature alternatives:

- Foundation: `alien-works-foundation`.
- Native-binding systems: `aw-sdl`, `aw-glm`, `aw-filament/runtime`, `aw-physx`, `aw-chipmunk`, `aw-skia`, `aw-openal`, `aw-opus`.
- Platform graphics: `cl-opengl` for non-Android or `cl-opengl/es2` for Android.
- FFI, buffers and foreign utilities: `static-vectors`, `claw-utils`, `cffi`, `cffi-c-ref`.
- General infrastructure: `uiop`, `alexandria`, `trivial-main-thread`, `trivial-gray-streams`, `flexi-streams`, `bordeaux-threads`, `atomics`, `cl-muth`, `varjo`.

### `:alien-works/tools` tools

Additional direct systems: `alien-works`, `alien-works-foundation/tools`, `aw-filament/tools`, `aw-assimp`. This is a distinct graph; do not treat it as required for core runtime unless demonstrated.

## 3. Dependency layers and discovery

| Layer | Source/discovery | Status |
|---|---|---|
| Root Common Lisp application and local source tree | `vietcong73/alien-works` at audit branch | **AVAILABLE** |
| Alien-Works foundation and its tools system | `borodust/alien-works-foundation`, referenced in `aw-projects/.gitmodules` | **UNRESOLVED: HTTP 404** |
| Main AW binding wrappers | `borodust/aw-sdl`, `aw-glm`, `aw-filament`, `aw-physx`, `aw-chipmunk`, `aw-skia`, `aw-openal`, `aw-opus` | **SOURCE/ASDF DISCOVERED; compatibility unverified** |
| Extra tooling wrapper | `borodust/aw-assimp` | **SOURCE/ASDF DISCOVERED; compatibility unverified** |
| IFFI runtime | `iffi.asd` inside `borodust/claw` | **LOCATION RESOLVED; load unverified** |
| Other author-specific Lisp support | `borodust/claw-utils`, `borodust/cffi-c-ref`, `borodust/cl-muth` | **SOURCE/ASDF DISCOVERED** |
| `cl-muth` transitives | `bodge-queue` and `bodge-heap`, each with a public `borodust` repository | **SOURCE DISCOVERED; not pinned** |
| Generated AW bindings | Platform-selected `*-bindings.asd` in wrapper repos, with generated `bindings/` code | **MANIFESTS DISCOVERED; not executed** |
| Foreign/native libraries and adapters | Git submodules under wrapper `src/lib/`, plus runtime native loading | **REPOSITORY LINKS DISCOVERED; native closure unverified** |
| Third-party Common Lisp libraries | Root ASDF and generated-binding `depends-on` clauses | **NAMED; exact versions and transitive resolution unverified** |
| Compiler, OS, ABI, driver and native artifacts | Platform features and foreign-library requirements | **UNVERIFIED** |

A repository being publicly accessible is evidence of source availability, **not** evidence of successful dependency resolution, ABI compatibility, or build reproducibility.

## 4. Key source evidence

1. The upstream 2023 Alien-Works baseline depends on `alien-works-foundation` for core runtime and `alien-works-foundation/tools` for tools; the expected public repository URL currently returns **HTTP 404** under the connected GitHub account. A 404 alone does not prove deletion: it may also indicate access restrictions, renaming or other unavailability.

2. `borodust/aw-projects/.gitmodules` names `alien-works-foundation`, `alien-works`, AW wrappers, `claw`, `claw-utils`, `cffi-c-ref`, etc. The `aw-projects/master` Git tree pins `alien-works` at `b0fa75c6a17c37763b2bf23cb391460dc57ae13a` and foundation at `56e20e2d6bfda834694774980c9f7c5cb58d9e85`. The `aw-projects` HEAD is `8cd6a685ad28e90daeb12617c5d3f95dcc5e989d` (2021-10-25); these pins are not a validated closure for the 2023 audited baseline.

3. `aw-projects/.gitmodules` also names `cl-muth`, but its current tree does **not** contain a corresponding `cl-muth` gitlink; this is a concrete manifest/tree discrepancy, not necessarily a code defect.

4. `borodust/claw/iffi.asd` defines the `:iffi` system and depends on `alexandria`, `cffi`, `trivial-features`. The generated binding manifests of several AW wrappers depend on `:iffi`. Thus `iffi` is discoverable even though no separate `borodust/iffi` repository was required.

5. `borodust/cl-muth/cl-muth.asd` depends on `bodge-queue` and `bodge-heap`, which are separately discoverable.

6. Generated bindings are not a uniform frozen platform snapshot. Examples: `aw-filament-runtime-bindings.asd` carries generation timestamp **2023-04-30** and selects x86_64 Linux, aarch64 Android and x86_64 Windows; `aw-glm-bindings.asd` currently carries generation timestamp **2026-02-06** and lists only x86_64 Linux. This does not establish end-to-end portability or compatibility with the 2023 root.

7. `src/framework/framework.lisp` references `dissect` and `float-features`; these are not direct entries in root `:alien-works`'s `depends-on`. They may be supplied transitively, but direct dependency discipline and load ordering are not independently demonstrated.

8. The root source tree has no `.gitmodules` or verified lockfile providing exact revisions of the remote bindings and native submodules; the available `aw-projects` assembly is historical and mismatched.

## 5. Runtime versus regeneration graph

```text
RUNTIME / APPLICATION
:alien-works
  +-- :alien-works-foundation                  [MISSING/INACCESSIBLE]
  +-- AW wrappers (SDL, GLM, Filament, etc.)   [SOURCE AVAILABLE]
  |    +-- generated *-bindings
  |    +-- CFFI / IFFI / CLAW-utils / features
  |    +-- corresponding native library or shim [NOT VERIFIED]
  +-- general CL libraries                    [VERSION UNPINNED]
  +-- OS / ABI / toolchain requirements       [NOT VERIFIED]

OPTIONAL TOOLS
:alien-works/tools
  +-- :alien-works
  +-- :alien-works-foundation/tools             [MISSING/INACCESSIBLE]
  +-- :aw-filament/tools
  +-- :aw-assimp

BINDING REGENERATION (separate from ordinary use)
:aw-*/wrapper
  +-- :claw / :claw-utils
  +-- foreign headers, libclang/libresect, native build toolchain
```

The author's explanation distinguishes **CLAW-generated binding consumption** from running CLAW to regenerate bindings. See [CLAW README](https://github.com/borodust/claw/blob/master/README.md) and [author's 2021 architecture note](https://borodust.github.io/2021/05/30/alien-works/). Do not drag the regeneration toolchain into the minimal runtime by assumption.

## 6. Closure blockers and bounded next remedies

| ID | Issue | Severity for reproducible full build | Resolution criterion |
|---|---|---|---|
| DC-B1 | Foundation source unavailable via declared repository | **BLOCKING** | Obtain an authorized, provenance-checked source or decide to replace the dependency |
| DC-B2 | No coherent 2023-compatible pins for AW submodules | **BLOCKING** | Record a single validated revision manifest for required repos and native dependencies |
| DC-B3 | No verified native binaries / ABI / dynamic-linking setup | **BLOCKING** | Demonstrate platform-specific load and basic integration smoke test |
| DC-B4 | Full CL transitive closure and explicit direct dependencies not verified | **OPEN** | Resolve ASDF graph; make `dissect`, `float-features`, and other implicit references explicit if required |
| DC-B5 | Platform-specific generated bindings differ; mixed generation dates | **OPEN** | Qualify one target environment first; defer unsupported platforms |
| DC-B6 | Runtime/tooling/regeneration graphs not enforced as separate build profiles | **OPEN** | Define explicit minimal runtime and opt-in tools/regeneration profiles |

A 2021 submodule manifest, a 2026 generated-binding timestamp, and a 2023 application source tree are not a usable lockfile merely because every path is spelled correctly.

## 7. Reserve implications (not an architecture verdict)

- Treat Alien-Works as **source evidence and design reference**, not a verified ready-to-reuse foundation.
- Preserve runtime/bindings/tooling boundaries. Prefer a minimal, explicitly pinned Lisp host for the reserve.
- Avoid reimplementing missing foundation behavior without first inventorying the exact capabilities called by the root.
- Do not choose STREAMLINE vs REBUILD on this closure alone. That requires subsequent bounded source/subsystem evaluation.
- No build/test, no source code edit, and no claim of runtime qualification were performed in DC-01.

## 8. Evidence links

- [Audited fork](https://github.com/vietcong73/alien-works/tree/audit/lisp-foundation-reserve)
- [Audit provenance](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/lisp-foundation-reserve-audit-provenance.md)
- [Root ASDF](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/alien-works.asd)
- [aw-projects Git submodules](https://github.com/borodust/aw-projects/blob/master/.gitmodules)
- [aw-projects Git tree](https://github.com/borodust/aw-projects/tree/master)
- [CLAW / IFFI](https://github.com/borodust/claw/blob/master/iffi.asd)
- [cl-muth ASDF](https://github.com/borodust/cl-muth/blob/master/cl-muth.asd)
- [Filament ASDF](https://github.com/borodust/aw-filament/blob/master/aw-filament.asd)
- [Filament generated binding manifest](https://github.com/borodust/aw-filament/blob/master/aw-filament-runtime-bindings.asd)
- [GLM generated binding manifest](https://github.com/borodust/aw-glm/blob/master/aw-glm-bindings.asd)

**DC-01 disposition:** static inventory checkpoint complete, **full runtime dependency closure not proven**. Stop here before the next audit unit.
