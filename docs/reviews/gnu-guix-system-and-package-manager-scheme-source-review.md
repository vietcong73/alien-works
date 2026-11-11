# GNU Guix: Scheme Source Code and Architectural Review

**Subjects:** GNU Guix package manager and GNU Guix System  
**Language:** GNU Guile Scheme (with a native execution/store substrate)  
**Review type:** Direct static source review: operating principles, philosophy, implementation architecture, strengths, limitations and correctness boundaries  
**Upstream source revision:** `18f545c14eb36c7c588d2cf356e387e17e53cd51`, November 9, 2026  
**Report date:** November 12, 2026  
**Upstream source:** [GNU Guix official Git repository](https://git.savannah.gnu.org/cgit/guix.git/) · [GitHub mirror, pinned revision](https://github.com/guix-mirror/guix/tree/18f545c14eb36c7c588d2cf356e387e17e53cd51)  
**Local reference only:** `vietcong73/guix-config` pins this Guix channel revision; machine configuration and user experience inform the questions but are not the target of review  
**Out of scope:** Detailed GNU Guix Home audit; Nonguix package definitions and proprietary firmware; hardware/GPU/desktop diagnosis; native daemon security audit; comprehensive runtime tests; code changes

---

## 1. Executive verdict

**GNU Guix is a technically substantive use of a Lisp-family language for systems software.** Scheme is used to construct an ontology for packages and operating systems, express staged computations, resolve dependency graphs, compose services, and generate versioned build artifacts. The language's metaprogramming capability serves a genuine architectural purpose, rather than merely replacing a conventional configuration-file syntax.

The package manager and Guix System are not competing engines. **They share one Guix source codebase and an underlying store/build infrastructure, with two different declared outputs: software environments and complete operating systems.** Guix System reuses package definitions, staged expressions, derivations, the store and profiles, adding an operating-system record and a service graph.

**What is particularly strong:** package and origin records, G-expressions and explicit execution staging, derivation construction, authenticated channels, service-extension composition, declarative configurations and generation switching.

**Where the design pays a price:** staged execution makes debugging demanding; lazy fields and composition can hide effective dependencies from readers; reproducibility requires actual deterministic builds; and activation, service restarts and bootloader updates remain effectful and are not one atomic live-system transaction.

**Review disposition:** `ARCHITECTURAL_SOURCE_REVIEW_COMPLETE / SOURCE_PATHS_INSPECTED / END_TO_END_CORRECTNESS_NOT_QUALIFIED`.

This is a source-grounded architectural review, not a declaration that all upstream functions have passed an independent audit.

## 2. Architecture: one codebase, two roles

The relevant conceptual pipeline is:

```text
Source package definitions / operating-system declarations
                    |
                    v
Scheme records + macros + transformations + service composition
                    |
                    v
G-expressions, packages/bags, derivations, build graph
                    |
                    v
Guix store daemon + builds/substitutes + /gnu/store objects
                    |
                    v
Package profiles / OS system generations / activation
```

Two principles organize this design:

1. **Describe desired artifacts and their dependencies before performing deployment effects.** Guix models an intended environment as values, then builds the corresponding outputs.
2. **Use immutable, version-addressable outputs and small mutable references.** Profile and system generation links select among constructed outputs.

In our own discussions, the labels *CFO* and *COO* respectively describe supply-chain/dependency governance and deployment/operational-state responsibilities. They are **not upstream Guix programs**, separate repositories, or official architectural names.

The most relevant distinctions are:

| Property | Guix package manager | Guix System |
| --- | --- | --- |
| Main declared object | `<package>` and manifest | `<operating-system>` and services |
| Principal transformation | Package → bag → derivation → output | OS → service graph → system derivation |
| Versioned deployment | Profile generations | System generations and boot entries |
| Key interface | G-expressions, build systems, channels | Service types, extensions, composition |
| Main non-pure effects | Build/substitution, store and profile operations | Activation, filesystem, bootloader and service changes |

## 3. Package manager: implementation mechanics

### 3.1 Package, origin and dependency ontology

See [`guix/packages.scm:614–667`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm#L614-L667) and [`guix/packages.scm:318–362`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm#L318-L362).

The `<package>` record encodes name, version, source, build-system, arguments, ordinary inputs, propagated inputs, native inputs, outputs, search paths, supported systems and metadata. The source `<origin>` record specifies retrieval method, URI, content hash and patches.

**Strength:** software packages become composable, inspectable Scheme values, not loosely structured installation scripts. The distinction between target/runtime, native/build and propagated inputs is explicit, supporting cross-builds and dependency analysis.

**Cost:** several fields are thunked or inherited; transformations and target parameters can modify their effective values. Merely reading a package expression may not reveal its final dependencies or build plan.

A source hash verifies the bytes retrieved against the declared content hash. It does **not** demonstrate that the source is safe or that its build result is deterministic.

### 3.2 Lowering: packages, bags, derivations

See [`guix/packages.scm:1812–1855`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm#L1812-L1855), [`guix/packages.scm:1537–1557`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm#L1537-L1557) and [`guix/derivations.scm:769–789`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/derivations.scm#L769-L789).

The `package->bag` step packages inputs, outputs, source, build method, target and arguments into a more concrete intermediate representation. `package-closure` traverses dependencies with a visited set. Derivations make the build action and its inputs explicit enough for store-level scheduling and hashing.

The derivation hash mechanism considers derivation contents and special cases such as fixed-output derivations. A **derivation identity** is not a proof that independent runs produce byte-identical output. Build scripts can still contain nondeterministic operations unless constrained or tested.

**Design virtue:** the input graph becomes a first-class object rather than an incidental result of a procedural installer.

**Design liability:** the path from high-level package to executable build has many intermediate representations, caches and transformations. Diagnosing an incorrect output can require understanding every layer.

### 3.3 G-expressions: two stages of Scheme

See [`guix/gexp.scm:1118–1218`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/gexp.scm#L1118-L1218) and [`guix/gexp.scm:1220–1268`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/gexp.scm#L1220-L1268).

G-expressions (gexps) express Scheme programs intended to run in a build context distinct from the authoring/evaluation context. `lower-gexp` resolves referenced objects, imported modules, compiler outputs and Guile for the target build. `gexp->derivation` translates a staged expression into a derivation.

This is the strongest language-design decision in the package manager. Lisp code-as-data is turned into a disciplined boundary for **constructing executable programs together with their build-time dependencies**.

The cost is semantic complexity. A mistake may appear in macro expansion, host-side evaluation, staging/escaping, lowering, module compilation or eventual execution. Scheme makes these transformations possible; it does not make the boundary errors impossible.

**Concrete upstream evidence of this cost:** the pinned commit itself fixes `guile-osc-lib` so that `target-guile-effective-version` runs on the **build side** rather than on the host side. The old implementation could resolve Guile from the host's `$PATH` while lowering the derivation, making the derivation depend on ambient host state. This is a *fixed historical defect*, **not an open defect of the pinned commit**. [Inspect the exact upstream fixing commit](https://github.com/guix-mirror/guix/commit/18f545c14eb36c7c588d2cf356e387e17e53cd51).

### 3.4 Store client and native execution boundary

See [`guix/store.scm:1256–1282`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/store.scm#L1256-L1282) and [`guix/store.scm:1392–1401`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/store.scm#L1392-L1401).

Scheme orchestrates requests to the store and exposes build, substitution, verification and garbage-collection APIs. It is **not** the entirety of the privileged execution substrate. Native daemon/store implementation, builders, operating system and filesystem behavior lie beyond this Scheme-only review.

The source explicitly notes that store repair is not atomic. Thus it is incorrect to read the functional model as a promise that every store maintenance action is transactional.

### 3.5 Manifests, package transactions and generations

See [`guix/profiles.scm:870–965`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/profiles.scm#L870-L965), [`guix/scripts/package.scm:140–179`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/package.scm#L140-L179) and [`guix/profiles.scm:2328–2367`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/profiles.scm#L2328-L2367).

`manifest-transaction` represents install/remove intentions as values. Other functions calculate proposed installs, upgrades, downgrades and removals, then produce a new manifest. `build-and-use-profile` builds the proposed profile output before switching generation links.

**Major strength:** an unsuccessful build need not destroy the active profile. The published generation reference can change after a completed build, and older generations can remain accessible.

**Important boundary:** atomic switching of a symlink does **not** atomically update every running process, external database, user file or service. Reverting a profile does not rewind arbitrary effects caused by a program that ran while the profile was active.

The operational claim should be **versioned, switchable software environments**, not full-state time travel.

### 3.6 Channels, provenance and the trust model

See [`guix/channels.scm:441–554`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/channels.scm#L441-L554) and [`guix/scripts/pull.scm:751–822`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/pull.scm#L751-L822).

Guix channel definitions support repository revisions, introductions and an authentication chain for signed repository history. The CLI has checks for trusted channels and special handling of channel files.

**Strong design choice:** package definitions are executable Scheme code; authenticating their provenance and pinning their revision is materially better than treating arbitrary downloaded recipes as passive data.

**Limit:** an authenticated revision is not automatically correct, benign or reproducibly buildable. Authentication answers a provenance question, not every safety question. The existence of options to bypass authentication also means actual guarantees depend on invocation and configuration.

## 4. Guix System: implementation mechanics

### 4.1 Operating system as a Scheme record

See [`gnu/system.scm:243–335`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/system.scm#L243-L335).

The `<operating-system>` record holds kernel, initrd, firmware, bootloader, mapped devices, filesystems, users, groups, packages, locale and services. Several fields are thunked, delayed, defaulted or sanitized.

**Strength:** a complete operating-system configuration can be manipulated as structured Scheme data, including inheritance and transformations. Important defaults are expressed in the same language as customizations.

**Cost:** "declarative" is not synonymous with "fully static." Effective values may be computed only when accessed or lowered; the complete configuration cannot always be understood from one record literal without following dependencies and defaults.

The large top-level record also has considerable semantic breadth. It is appropriate to an OS configuration API, but makes compatibility, validation and maintenance demanding.

### 4.2 The service extension graph

See [`gnu/services.scm:187–211`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/services.scm#L187-L211), [`gnu/services.scm:1319–1459`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/services.scm#L1319-L1459) and [`gnu/system.scm:913–917`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/system.scm#L913-L917).

A `<service-type>` declares **extensions**, **compose** and **extend** operations. A service can contribute configuration or outputs to other services, rather than merely identifying a daemon to start. `instantiate-missing-services` resolves defaults for referenced services, and `fold-services` propagates contributions into the target service.

The implementation includes dedicated conditions for missing and ambiguous target service types. This is valuable source-level evidence that the framework treats graph structure as a correctness concern, not an implementation afterthought.

**Architectural achievement:** services form a compositional dependency system for OS construction. A service can cause files, accounts, packages, shepherd definitions and other resources to appear by extending the appropriate service types.

**Cost:** effective system state becomes distributed across service definitions and extension paths. For a large graph, tracing the origin of a final configuration value can be more difficult than reading a linear configuration script.

**Audit caveat:** the inspected recursive graph fold does not, by itself, establish behavior under every possible custom extension cycle. A conclusion of "cycle handling is broken" would require complete call-path and test inspection, not inference from one recursive function.

### 4.3 From service composition to system derivation

See [`gnu/system.scm:1370–1386`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/system.scm#L1370-L1386) and [`gnu/services.scm:463–510`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/services.scm#L463-L510).

The key function `operating-system-derivation` obtains the OS's services, folds them into a root system service, and returns its derived output. That root describes the system directory, referencing kernel, initrd, boot and related artifacts.

The consequence is profound: **Guix System does not simply run a list of imperative installation actions; it constructs an operating-system artifact through the same derivation substrate as packages**.

This is why Guix System and the package manager can meaningfully share one codebase without reducing the OS to "a list of packages."

### 4.4 Deployment, activation and runtime effects

See [`guix/scripts/system.scm:790–853`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/system.scm#L790-L853), [`guix/scripts/system.scm:855–923`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/system.scm#L855-L923) and [`guix/scripts/system.scm:369–380`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/system.scm#L369-L380).

The system action pipeline prepares/checks configurations and builds system/boot artifacts. For `reconfigure` it then proceeds through side-effecting operations such as switching the system, installing the bootloader and upgrading Shepherd services.

The code explicitly emits advice to restart services manually when automatic restarts have not occurred.

**Strength:** desired system artifacts are built before deployment, and generations make alternate versions accessible.

**Limit:** reconfiguration is **not a single atomic transaction covering runtime services, the bootloader, filesystems and application data**. If a later effect fails after an earlier one succeeded, the code cannot be presumed to reverse all earlier external changes. A generation rollback similarly does not reverse arbitrary persistent user data or daemon side effects.

The right description is **versioned system construction with bounded deployment/recovery mechanisms**, not mathematically total reversibility.

### 4.5 Provenance and validation at OS scope

See [`gnu/system.scm:930–950`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/system.scm#L930-L950) and [`guix/scripts/system.scm:838–853`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/system.scm#L838-L853).

`operating-system-with-provenance` attaches a service recording configuration provenance, where available. Reconfigure paths check forward-update conditions, mapped devices, filesystem availability and initrd modules, subject to privilege/target restrictions and bypass flags.

This is an important strength: the OS path includes explicit validation and source-traceability mechanisms.

But a static preflight can only validate properties it knows how to check. It cannot prove that services will run correctly across all possible kernel, hardware and external-state combinations.

## 5. Philosophy, strengths and trade-offs

### 5.1 Functional paradigm in the correct sense

The strongest philosophical move is to **model desired outcomes, and defer external effects until the intended artifact has been calculated and built**. Immutable store items and versioned references make deployment more tractable.

The phrase "purely functional operating system" should be interpreted as a **functional approach to system and package construction**, not a claim that a running operating system has no mutable state.

### 5.2 What the Scheme code makes unusually good

1. **One language for data and transformation.** Package and OS declarations are objects that can be transformed, extended and validated.
2. **Staged metaprogramming with explicit build inputs.** G-expressions are a serious engineering use of Lisp's symbolic-programming affordances.
3. **Service-graph composition.** The service-type/extension mechanism provides a controlled vocabulary for complex OS configuration.
4. **Provenance and version selection.** Channel introductions and pinned revisions give meaningful source traceability.
5. **Versioned recovery.** Generations and profile switching are significantly more disciplined than ad hoc mutation of installed files.
6. **Structured failure handling.** Errors such as service-target ambiguity and profile-generation absence are modeled directly in Scheme APIs.

### 5.3 What the implementation makes difficult

1. **Cross-stage debugging.** Host code, gexp staging, derivation lowering, build-side code and daemon execution each have different failure modes.
2. **Effective configuration visibility.** Laziness, inherited records, transforms and distributed extensions make reading source harder than reading a flat configuration.
3. **Boundary correctness.** Staged-code escapes and imported modules can accidentally expose host environment state; the baseline's guile-osc fix demonstrates this concretely.
4. **Non-atomic live deployment.** System generation selection is not equivalent to a transaction across all processes, services and bootloader effects.
5. **Reproducibility is empirical, not automatic.** Derivations describe intended inputs; nondeterministic builders still require detection and remediation.
6. **The Scheme/native divide remains real.** Correct high-level source cannot substitute for testing the privileged store daemon, toolchains, kernel integration and filesystem operations.
7. **Supply-chain trust is nuanced.** Authenticated channels establish a trust path, not semantic safety of all downloaded code and binaries.

## 6. Source-grounded technical observations

This section deliberately distinguishes an implementation property from a newly proven bug.

| ID | Observation | Evidence | Classification |
| --- | --- | --- | --- |
| **G-01** | The pinned commit fixes host-side environment leakage during build-stage version selection for `guile-osc-lib`. | [Exact fixing commit](https://github.com/guix-mirror/guix/commit/18f545c14eb36c7c588d2cf356e387e17e53cd51) | **Historical defect, already fixed in baseline** |
| **G-02** | `build-and-use-profile` switches generation symlinks only after building the profile derivation. | [`guix/scripts/package.scm:140–179`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/package.scm#L140-L179) | **Positive design evidence** |
| **G-03** | `guix system reconfigure` calls multiple effectful deployment operations. | [`guix/scripts/system.scm:883–907`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/scripts/system.scm#L883-L907) | **Source-confirmed limitation of global atomicity; not itself a bug** |
| **G-04** | Guix explicitly notes store repair is not atomic. | [`guix/store.scm:1392–1398`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/store.scm#L1392-L1398) | **Source-confirmed effect boundary** |
| **G-05** | Missing/ambiguous service targets have dedicated error types. | [`gnu/services.scm:1297–1345`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/gnu/services.scm#L1297-L1345) | **Positive robustness evidence** |
| **G-06** | Package declarations can contain thunked inputs and arguments. | [`guix/packages.scm:614–633`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/packages.scm#L614-L633) | **Source-confirmed complexity/observability trade-off** |
| **G-07** | Channel authentication is explicit and can be disabled by option. | [`guix/channels.scm:494–544`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/channels.scm#L494-L544) | **Trust-policy boundary** |
| **G-08** | Generation rollback changes profile/system selection, not arbitrary application data. | [`guix/profiles.scm:2328–2367`](https://github.com/guix-mirror/guix/blob/18f545c14eb36c7c588d2cf356e387e17e53cd51/guix/profiles.scm#L2328-L2367) | **Scope limitation, not an upstream defect** |

**No new unpatched critical upstream bug is claimed by this bounded two-component review.** The earlier preliminary finding in `gnu/home/services.scm` concerns the separate Guix Home subsystem and is **not** included in this Package Manager + Guix System review or counted as a Guix System finding.

## 7. Relation to the local machine: reference, not audit target

The private `guix-config/channels.scm` reference declares the Guix channel revision inspected here, alongside a separate Nonguix channel. That makes this review directly comparable with the source revision recorded for the local system snapshot, while avoiding claims about uninspected hardware or changed runtime state.

The existence of multiple Guix executable paths and generations in the local experience illustrates an expected version-selection responsibility of a system with independently evolvable channels, profiles and user environments. It is **not evidence of a Guix upstream bug**, and the local configuration has not been modified as part of this review.

## 8. Comparative final verdict

| Dimension | Package manager | Guix System |
| --- | --- | --- |
| Design philosophy | Functional package/build model | Declarative, composable OS construction |
| Strongest Scheme abstraction | G-expressions, package/bag/derivation lowering | Service types and extension folding |
| Best operational property | Versioned software environments | Versioned system construction |
| Main implementation burden | Multi-stage build and dependency debugging | Service-graph and activation complexity |
| Main correctness boundary | Deterministic description ≠ deterministic build output | Versioned system ≠ atomic live-state rollback |
| Source-review verdict | **Architecturally strong** | **Architecturally strong** |

**The core insight:** GNU Guix turns Scheme into a language for *describing and transforming computing environments*. The package manager implements the construction substrate; Guix System composes that substrate into an operating system.

The most instructive source modules remain **`guix/gexp.scm`** and **`gnu/services.scm`**. The first explains staged, dependency-aware computation; the second explains compositional configuration at OS scale.

**Final disposition:** `ARCHITECTURAL_SOURCE_REVIEW_COMPLETE / SOURCE-BASED LIMITATIONS DOCUMENTED / DYNAMIC_VERIFICATION_OUTSTANDING`.

### Audit limitations

- Inspection was limited to relevant Scheme modules and source call paths; it is not a full line-by-line review of all Guix code.
- No reproduction of the Guix Home on-change defect is claimed here because Home is outside the requested two-component scope.
- No Guix source, Linux system configuration, package manifest or channel was edited.
- No command was executed on the user's machine.
