# LFR-P08-D03 | Game World Procedural Generation Architecture

**Document class:** Source-grounded detailed design proposal for one gaming specialist slice  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `GAMING` — Gaming & Interactive Worlds  
**Edition:** P08 — World, Scene & Spacetime Decision Systems  
**Design slice:** D03 — Game World Procedural Generation Architecture  
**Primary design language:** Gerbil Scheme; qualified native or remote compute adapters selected by measured need  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`, path `docs/design/LFR-P08-D03-game-world-procedural-generation-architecture-2026-11-13.md`  
**Status:** `DESIGN_PROPOSED / SOURCE_GROUNDED / GAMING / NO_IMPLEMENTATION / NO_RUNTIME_QUALIFICATION / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. Source implementation, backend installation, and deployment remain separately authorized steps.

> **Design decision:** P08-D03 supplies a **versioned procedural-content pipeline** capable of representing and, once separately implemented, generating world regions without owning world causality or authoritative scene/entity state. A **pure Gerbil implementation (A)**, **Gerbil-governed native execution (B)**, and **Gerbil-coordinated external generation service (C)** are equally legitimate profiles. Select per workload using verified semantic parity and measured end-to-end performance. Never force a single language across all generation stages.

## 0. Objectives and boundaries

**Research question:** Can independently specified world-generation algorithms and a reproducible content pipeline be implemented in Gerbil Scheme where practical, while retaining a strict, low-overhead route to native code or external generation services when demonstrated workloads demand it? In particular, can the architecture support large, progressively materialized game worlds inspired by the **documented techniques** used in *No Man's Sky*, without copying undocumented proprietary engine implementations?

**Useful first deliverable:** a deterministic 2D region generator and reference materialization adapter, able to generate repeatable terrain/biome/placement descriptors; later opt-in profiles add 3D density fields, mesh extraction, planetary coordinate charts, ecological growth and more demanding backends. Each profile has its own qualification and does not make every world dependent on a renderer or physics engine.

**In scope:** algorithm manifests; typed generation recipes and intermediate representations; coordinate-addressable randomness; stage graphs; field sampling; region boundaries; 2D/3D/planet profiles; geometry/biome/population descriptors; LOD, caching and generation revisions; controlled backend dispatch; correctness and performance benchmarks; explicit optional P08-D01/D02 integration.

**Out of scope:** ownership of world consequences (D01); canonical scene/entity storage, streaming and residency (D02); P06 branching/retention/synthesis; physics integration and evolving agents; graphics rendering; nonlocal world-causal inference; autonomous RL training; reproducing or redistributing *No Man's Sky* source; guaranteeing universal procedural aesthetics; arbitrary Scheme evaluation from untrusted recipes; claiming a production game or a qualified generator exists.

### 0.1 Separation of responsibilities

| Owner | Receives | Must not take over |
|---|---|---|
| **P08-D01** | Generation suggestions relevant to a candidate world transition | Geometry calculations, mesh extraction, native accelerator ownership |
| **P08-D02** | Validated region artifacts under exact world/materialization locks | Procedural algorithms, causal admission or semantic interpretation of an unknown output |
| **P08-D03** | A bounded region/recipe request with explicit algorithm and backend profiles | Changing authoritative world state, deciding historical ancestry or granting visibility |
| **P06** (optional) | Versioned history references from the application | Generation algorithm identity or materialized asset caches |
| **K0/K1** | Typed identity, validation, outcomes, optional pure mechanisms | A mandatory game engine, RNG policy, GPU runner or universal simulator |

Two worlds can use the same generator and seed but possess different causal overlays, histories, scene identities or world-model revisions. Their **materialization cache keys are different unless a separately qualified equivalence witness permits reuse**.

## 1. Research and source-evidence ledger

**Evidence levels:** `DIRECT_DOC` = official documentation; `PUBLIC_TALK` = publicly described industrial pipeline, no shipped code inspected; `OPEN_SOURCE` = publicly inspectable implementation/reference, not tested in this project; `LFR_DESIGN` = committed proposal, not executed evidence; `HYPOTHESIS` = to be implemented, measured and falsified.

| Source and evidence level | What it supports | Limit on inference |
|---|---|---|
| **Hello Games / Innes McKendrick**, GDC 2017 *Continuous World Generation in No Man's Sky* (`PUBLIC_TALK`) | Continuous, on-demand planetary world generation from **voxel-based world generation → polygonization → texturing → population → simulation** | Pipeline outline is public; exact shipped kernels, data layout, compiler, source, algorithms and performance figures are not available for direct porting |
| **Hello Games**, *Worlds Part II* 5.50 (`DIRECT_DOC`) | New terrain patterns and new purple star systems without regenerating established systems or resetting bases | The exact implementation of selective versioning/migration is not disclosed |
| **Hello Games**, *Origins* 3.0 (`DIRECT_DOC`) | New planetary diversity while retaining old world's terrain and bases | Demonstrates a product compatibility goal, not a specific state-store algorithm |
| **FastNoiseLite** by Auburn (`OPEN_SOURCE`) | Named 2D/3D value, Perlin, OpenSimplex2, cellular, fractal and warp noise families in multiple languages | Its published throughput benchmarks are hardware/workload-specific and do not establish Gerbil performance or exact parity across implementations |
| **Eric Lengyel**, Transvoxel (`DIRECT_DOC` + published algorithm) | Transition-cell stitching for meshes extracted from volumetric terrain at different resolutions | This is one candidate for a voxel-LOD seam problem, not a requirement for all terrains; licenses/attributions must be reviewed before code reuse |
| **Epic Games**, Unreal World Partition HLOD (`DIRECT_DOC`) | Represent distant/unloaded cells with lower-cost proxies; view residency differs from authority | A renderer-specific HLOD system cannot be equated to causal world state |
| **Godot**, procedural geometry docs (`DIRECT_DOC`) | Different tradeoffs for ArrayMesh, SurfaceTool, ImmediateMesh, MeshDataTool | CPU mesh API choices are engine-specific; no Gerbil counterpart is assumed |
| **Gerbil/Gambit** official FFI/build docs (`DIRECT_DOC`) | Source-level Gerbil/Gambit-to-C interop, build and generated native interface possibilities | FFI existence proves neither native thread safety nor zero-copy transfer/performance |
| **P08-D01**, **P08-D02**, **P06**, **K0**, **K1** (`LFR_DESIGN`) | Exact contracts for decision/world identity, region requests, version locks, historical roles and generalized mechanisms | No implemented runtime, no generator benchmark PASS, no native ABI qualification |

**Primary references** (source claims above are bounded to these public artifacts):

1. [GDC Vault — Continuous World Generation in *No Man's Sky* (2017)](https://www.gdcvault.com/play/1024265/Continuous_World_Generation_in__No_Man_s_Sky_) and [Game Developer synopsis](https://www.gamedeveloper.com/programming/video-how-continuous-world-generation-works-in-i-no-man-s-sky-i-).
2. [No Man's Sky — Worlds Part II update and patch notes](https://www.nomanssky.com/worlds-part-ii-update/); [Origins update](https://www.nomanssky.com/origins-update/).
3. [Auburn/FastNoiseLite source and README](https://github.com/Auburn/FastNoiseLite).
4. [Transvoxel — published algorithm, application and tables](https://transvoxel.org/).
5. [Unreal Engine — World Partition HLOD](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine).
6. [Godot — Procedural Geometry](https://docs.godotengine.org/en/stable/tutorials/3d/procedural_geometry/index.html).
7. [Gerbil Scheme — FFI guide](https://gerbil.scheme.org/guide/ffi.html), [introduction](https://gerbil.scheme.org/guide/intro.html), [build tool](https://gerbil.scheme.org/reference/dev/build), and [Gambit manual](https://gambitscheme.org/latest/gambit.pdf).
8. `docs/design/LFR-P08-D01-spacetime-decision-causal-consequence-design-2026-11-13.md`, `docs/design/LFR-P08-D02-game-world-scene-architecture-2026-11-13.md`, `docs/design/LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md`, and `docs/design/LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md` on branch `audit/lisp-foundation-reserve`.

### 1.1 What may be ported into Gerbil?

Three distinct cases must never be combined:

- **Public algorithmic ideas:** stable coordinate sampling, octave composition, cellular regions, density functions, polygonization, spatial LOD, stochastic placement and streamed generation can be independently designed and implemented from lawful public descriptions.
- **Open-source implementation:** importing/porting a concrete FastNoiseLite implementation or lookup table requires source provenance, exact version lock, license and notice compatibility, attribution where required, and independent correctness/performance tests.
- **Proprietary NMS implementation:** neither the GDC summary nor patch notes establish a lawful, publicly available full implementation to translate to Scheme. The programme must not fabricate kernel details or assert byte parity with NMS planets.

The proper question is **"Can our independently specified generator achieve the chosen workload efficiently in Gerbil?"**, not **"Can Gerbil run NMS's unavailable code?"**.

## 2. Layered world-generation architecture

```text
AUTHOR / GAME DESIGN / TOOLING (trusted rule authoring)
          |
    typed GenerationRecipe, revision-pinned world anchor
          v
GERBIL GENERATION COORDINATOR (P08-D03)
  validate recipe -> lower to GenerationIR -> select stage graph
  exact input closure -> budget & capability checks -> admit jobs
          |
          +--> A: Gerbil pure reference/compiled kernels
          +--> B: Gerbil -> batched native/GPU adapter
          +--> C: Gerbil -> versioned external service
          |
          v
GeneratedRegionArtifact + EvidenceReceipt
          |
  D02 validates identity/revision/working-set admission
          v
Resident representation / cache / view (not world truth)
          |
  D01 may separately admit any proposed semantic world changes
          v
Versioned authoritative WorldSnapshot (only via D01/D02 ports)
```

No generated *representation* can silently promote itself into a canonical world consequence. An authored initial-world fixture may install a baseline through a **separately qualified initialization gate**; it is not permission for a random mesh worker to spawn canonical entities during ordinary gameplay.

**Input closure:** an output is a function of the recipe and all declared inputs, not just a 32-bit seed. The full context includes algorithm revision, canonical coordinate frame, region boundaries, time/branch context when semantically relevant, causal overlay revision, sampled RNG streams, precision profile and any external asset/model dependencies.

## 3. Public contract parity with P08-D01 and P08-D02

P08-D03 adopts the materialization request/response contract defined by P08-D01 and consumed by P08-D02 as its public compatibility floor. The contract's field meanings, revision locks, admissibility and receipt semantics must remain consistent across all three designs.

### 3.1 Input and output records

```text
MaterializationRequest {
  world_ref, branch_ref_optional, world_model_ref,
  generator_profile_ref, generator_revision, seed_or_input_ref,
  space_scope, world_time_ref, causal_overlay_revision,
  desired_detail, budget_ref, expected_world_generation
}
MaterializationResult {
  region_ref, generated_artifact_ref, input_locks[],
  semantic_validation_result, world_generation,
  uncertainty_or_failure, provenance_receipt
}
```

D03 may wrap these with internal `GenerationJob` and `GenerationArtifact` records. It must not require extra public fields that make the existing D01/D02 port unusable. Additional metadata belongs in **versioned, optional extension manifests** referenced by `generator_profile_ref` or `provenance_receipt` and admitted by a compatibility profile.

**D02's request generation** remains distinct from world generation. If a region job is canceled then a new request reuses the display name or region address, the old completion must not bind to the new request.

### 3.2 Internal semantic records

```text
GeneratorManifest {
  generator_ref, generator_revision, recipe_schema_ref,
  canonicalizer_revision, stage_graph_ref, algorithm_refs[],
  coordinate_profile_ref, randomness_profile_ref,
  numeric_profile_ref, input_dependency_refs[],
  output_contract_ref, capability_requirements[],
  declared_nondeterminism, compatibility_policy_ref,
  qualification_ref, license_provenance_ref
}
GenerationRecipe {
  recipe_ref, generator_manifest_ref, world_anchor_ref,
  root_seed_ref, region_scope, output_kind, quality_profile,
  declared_world_inputs[], declared_overlay_refs[],
  budget_profile_ref, stage_parameters_ref, expected_result_schema
}
GenerationIR {
  normalized_stage_dag, typed_ports[], parameter_values,
  algorithm_revision_locks[], dependency_closure_receipt,
  numeric_and_rng_contract, output_schema_ref, signature_ref
}
StageResult {
  stage_ref, input_digest_ref, output_artifact_ref,
  exact_algorithm_revision, dependency_locks[],
  measurement_ref_optional, local_receipt,
  quality_and_correctness_status
}
GenerationArtifact {
  world_ref, region_ref, generation_request_ref,
  generator_manifest_ref, representation_kind,
  semantic_anchor_ref, overlay_ref, materialization_detail,
  payload_ref, payload_format_revision, output_validation_ref,
  cache_equality_profile_ref, provenance_receipt
}
```

### 3.3 Fundamental identities and relations

`WorldRef != RegionRef != SceneInstanceRef != GeneratorRef != GeneratedArtifactRef != CausalOverlayRef`. Two artifacts can be visually equal but not reuse-compatible; two regions can share source samples but have different world ownership; and a new mesh revision does not imply a new fictional event.

**Cache key minimum:** `(world, branch?, region, world model, recipe, generator+stage revisions, canonicalizer, root-seed/input refs, numeric+RNG, overlay, time scope, detail, view/privacy scope, backend equivalence profile)`. Some fields may be intentionally elided **only** under a separately verified semantic-independence/equivalence certificate. Hash equality by itself does not establish admissibility.

## 4. Algorithm families and honest portability

| Family | Candidate algorithms / representations | Minimal correctness oracle | When to use |
|---|---|---|---|
| **2D scalar fields** | seeded value/gradient noise, fractal octave composition, domain warp | exact sample table or bounded numeric oracle with explicit precision | initial terrain, climate, density masks |
| **Cellular / spatial partition** | Voronoi/Worley-style cell distance, bounded Poisson-like placement | deterministic ordering, spacing/coverage, edge ownership | settlements, forests, biomes, sparse content |
| **Growth / grammar** | bounded L-systems, cellular automata, branching grammars | termination/fuel, typed rewrite semantics, fixed seed witness | plant forms, organic structures, ruins |
| **3D density** | scalar field / signed distance profiles, multifield composition | sign convention, boundary continuity, finite samples | caves, overhangs, complex volumetric terrain |
| **Surface extraction** | marching cubes or other independently specified contourers | edge intersection, topology and mesh validity under declared assumptions | playable mesh from scalar fields |
| **Multiresolution** | tile grids, clipmaps, quadtree/octree adapters, voxel transition stitching | adjacent LOD seam oracle, no unmatched boundary holes | streaming at varying viewpoint distances |
| **Biome synthesis** | temperature/moisture/altitude fields, dependency-conditioned classification | deterministic classification, constraints and transition rules | planetary variety, ecosystems |
| **Population / prop placement** | rule-governed instance proposals, occupancy masks, weighted deterministic placement | no invalid overlap/placement, bounded resources | rocks, flora, structures, points of interest |
| **Planetary address space** | cube-face charts, local-tangent charts, hierarchical planet IDs | coordinate-face seam and transform error budgets | large worlds and space-to-surface continuity |
| **Cosmic hierarchy** | seeded star/system/body descriptors with structural constraints | no invalid hierarchy or physical-claim confusion | astronomical exploration, not astrophysical fidelity |

These are **candidate algorithm profiles**, not evidence that all are needed or currently implemented. Preserve the user-selected development order: simple 2D patterns/particles and organic dynamics first, then growth and terrain, eventually 3D and cosmic/planetary scales. This sequence is a *research progression*, not a condition that a game must import every stage.

### 4.1 Noise selection is not a religion

FastNoiseLite offers portable families, including OpenSimplex2, cellular and fractal/domain-warp variants. It is a useful public comparator, not a mandate. A noise family is chosen by **desired spectral/structural properties, reference correctness, continuity and measured performance**. Value noise can suffice for early 2D; a gradient/noise variant may improve landscape structure; cellular families can drive biome borders. Never call raw random samples a complete procedural world.

For a candidate octave composition:

```text
Field(x; theta) = sum_{i=0}^{N-1} a_i * Noise(f_i*x + offset_i, stream_i)
```

The equation is an **illustrative family**, not a universal NMS formula. Require declared amplitude/frequency schedules, bounded `N`, sampling convention, numeric tolerance and domain-warp behavior. Inputs must be bounded so zero, negative, NaN, infinite scales and degenerate coordinate frames do not crash a backend.

### 4.2 Volumetric terrain versus heightfields

A heightfield `h(x,y)` is a useful 2D-first profile but cannot directly represent undercuts and multiply valued surfaces. Volumetric density `d(x,y,z)` can represent caves/arches; surface extraction adds significant memory, topology and seam obligations. Do not expand a simple 2D design into a forced voxel engine merely because *No Man's Sky* used voxel-based generation. A world can switch representation only through a named migration/adapter with qualified geometry and coordinate equivalence.

### 4.3 Mesh correctness is separate from visual beauty

A contourer returns vertices/indices/material attributes under precise conventions: coordinate frame, winding, boundary samples, normals, degeneracy handling, index bounds and optional manifoldness guarantees. `mesh generated` does not imply `mesh usable for physics`, `mesh watertight`, `mesh artistically good` or `safe to import into a renderer`. Use independent geometric oracles: no NaN, bounded coordinates, valid indices, consistent orientation under selected profile, and declared boundary coverage. For complex non-manifold structures, a model may allow a weaker explicitly named topology class.

## 5. Deterministic randomness and numerical semantics

### 5.1 Addressable random streams

Avoid consuming one global mutable RNG sequence whose result changes when jobs finish in another order. A generation stage obtains a **domain-separated, version-locked stream** from `(world seed, generator revision, stage ID, region coordinates, local object ID, stream revision)`. The derivation can be implemented with an explicitly selected stable integer mixing/hash primitive and canonical encoding. No cryptographic claim follows unless a true cryptographic primitive and threat model are separately specified.

**Critical property:** generating region B before A must not change region A's valid output. Parallel job order, cancellation, cache hit/miss and retry must not silently change procedural identity. Local sequential PRNGs are allowed *inside* a stable region/stage partition if their draw order is contractually fixed and qualified.

### 5.2 Numeric precision profiles

`NumericProfile` states integer widths, overflow policy, floating precision, rounding, fused-operations policy where controllable, tolerances, unit scales, canonical serialization and NaN/Inf handling. Different GPUs or CPUs may not reproduce bit-identical transcendentals and floating reduction orders. One root seed is **insufficient** to claim exact cross-backend bytes.

Declare one of:

- `EXACT_REPRODUCIBLE`: exact canonical results under an exact pinned implementation/profile, checked with golden fixtures.
- `NUMERIC_TOLERANCE`: qualified error bound and how it propagates through stages; cannot imply identical mesh topology near threshold crossings.
- `TOPOLOGY_EQUIVALENT`: independently verified mesh/region structural property; not guaranteed by low sample RMSE.
- `SEMANTIC_EQUIVALENT`: named game-world constraints/observations agree; does not imply geometric identity.
- `UNQUALIFIED`: no parity claim; backend selection/reuse limited to isolated exploratory output.

A design may require exact reproducibility for save-critical content and permit numeric-tolerance equality for ephemeral purely visual decoration. It must **never** silently substitute the latter for the former.

### 5.3 Determinism and stochastic growth

Procedural generation can use random inputs while being reproducible under a sealed stream. This is distinct from **dynamic simulations** of growth or ecology with hidden disturbances. D03 can generate initial growth structures or parameterized snapshots; the temporal evolution of living agents and world decisions belongs to explicit world/simulation specialists. A generator may propose an ecological change, but D01 governs its admission if it alters world truth.

## 6. Region boundaries, seams and level of detail

### 6.1 Neighbor-independent sample coordinates

For each tile/chunk, derive field samples from **canonical world-space coordinates**, not from unrelated local RNG initialization without boundary agreement. Shared edge/halo samples must be specified by an identical ownership/rounding rule. Region partition and coordinate frame revisions from D02 are pinned.

- **Halo:** a bounded sampled margin for interpolation/meshing. Define ownership and reuse rules to avoid duplicate or inconsistent edge values.
- **LOD:** decimation/aggregation policies determine what is detail-dependent; visual LOD may change triangles without changing authoritative world state.
- **Seam:** neighboring artifacts must match in position/material/normal or a profile-defined transition region; mixed generator versions require explicit compatibility.
- **Delayed neighbor:** when B is absent, A must declare `SEAM_PENDING`, an admitted skirt/proxy, or `UNAVAILABLE`, rather than falsely claiming a seamless mesh.

Transvoxel is one publicly described technique for voxel meshes of differing resolution; **the choice to use it is a specialized algorithm decision**, requiring a license review and its own fixture oracles. Surface extraction is unnecessary for a graph-only world or 2D tile game.

### 6.2 Planet-scale coordinates

A planet profile may use hierarchical addresses `(body, face/chart, LOD, tile_x, tile_y, altitude band)` and local double/fixed-point coordinates. A global `float3` is not sufficient for precise spatial identity across astronomical scales. Face-to-face transitions, pole/edge singularities, axis conventions and tangent-frame transforms need independent testing. The first implementation should remain 2D tile-based until these tests and performance budgets are justified.

### 6.3 Generated geometry, canonical collision and physics

A low-detail render proxy cannot decide whether a cave is reachable, whether two actors collide, or whether a passage exists under D01 rules. Those are independent semantic/physics profiles. If gameplay depends on procedural topology, the **admitted world baseline** must include a versioned topology/occupancy contract with a qualified source of truth, not a renderer's opportunistic LOD decision.

## 7. Biomes, population and authored variety

### 7.1 Pipeline layers

1. **Planet/body parameters:** typed planetary class and environment constraints, without pretending random values are physical science.
2. **Geology or base geometry field:** terrain/density/erosion *proposal* from selected algorithms.
3. **Climate and biome fields:** altitude, moisture, temperature, regional variation with explicit dependencies.
4. **Population candidates:** flora, formations, resources and authored scene/asset placements.
5. **Validation:** placement overlap, navigability under specified topology, asset references and quota/scope constraints.
6. **Presentation proxies:** detail levels and meshes; no authoritative world-state mutation.

A single seed and many independent noise layers can still produce visually repetitive worlds. Evaluation should include within-world diversity, repeated-pattern detection and authored constraints alongside numerical correctness. **Aesthetic quality is an explicit product/design review**, not something a deterministic test alone can certify.

### 7.2 Generation does not spawn world entities

A `PopulationCandidate` names a proposed site, kind, asset family, geometric/structural preconditions and deterministic placement witness. If a game uses it to initialize the world, D02's authoring admission mode validates identity and ownership. If it becomes a gameplay mutation later, D01 must admit the consequence. A scene object rendered from a candidate is not automatically a canonical NPC or resource entity.

### 7.3 Preservation across generator upgrades

Borrow the **product-level compatibility lesson**, not a guessed proprietary implementation, from Hello Games: an updated generator can add new world ranges while preserving previously inhabited ones. The recommended baseline is *revision-stable world allocation*:

- Existing worlds/regions retain their `generator_revision` and reproducible input closure.
- New regions may opt into a newer revision under the world's explicit mixed-version policy and seam checks.
- A base, settlement, mined region or persistent authored scene pins its authoritative overlay and necessary baseline revision. New visuals cannot silently overwrite player construction.
- Migration is explicit, staged, versioned and reversible at the **data-version selection** level where supported; it does not promise reversing any external real-world effect.
- A missing old generator or required asset version returns `RECONSTRUCTION_UNAVAILABLE`, not "best effort" regeneration under the latest version while claiming unchanged history.

## 8. Job lifecycle, bounded resources and failure semantics

### 8.1 Stage graph

```text
RECEIVED -> VALIDATING -> ADMITTED -> QUEUED -> RUNNING
                 |                     |         |
               REFUSED              CANCEL   STAGE_FAILED
                                           |
RUNNING -> GENERATED -> VERIFYING -> READY_FOR_D02
               |            |
            PARTIAL       REJECTED/UNKNOWN
```

Stages include `recipe-normalize`, `field-generate`, `biome-classify`, `geometry-extract`, `population-candidates`, `artifact-validate` only if selected in the profile. A stage DAG is **acyclic for dataflow**, with bounded repetition/iteration explicit. A biological growth rule can have temporal feedback inside a bounded separate simulation profile; do not hide unbounded loops in DAG execution.

**Accepted != generated != validated != D02 resident != admitted world mutation.** Cancellation is a request, not proof that an in-flight FFI kernel or remote worker stopped. A timed-out external call may have computed an artifact whose status is `UNKNOWN` until reconciled by job identity.

### 8.2 Budgets and ownership

Profile limits: `max_stage_count`, `max_algorithm_recursion`, `max_region_samples`, `max_volumetric_cells`, `max_octaves`, `max_population_candidates`, `max_mesh_vertices`, `max_bytes_resident`, `max_bytes_staged`, `max_concurrent_jobs`, `max_output_bytes`, `max_wall_duration_if_enforced`, `max_gpu_allocation_if_supported` and `max_external_service_bytes/cost`. Do not claim a process/GPU budget the backend cannot enforce.

**Do not call FFI per voxel** for large hot loops by default. Batch per tile, field block, volume slab or complete stage with a typed descriptor; qualify copy cost, ownership, lifetime, pinning, alignment, thread affinity, cancellation and release. Native handles remain optional P01-qualified bindings; a generated buffer pointer is not permitted as a durable/world identity.

## 9. Three execution profiles and admission policy

### A — Pure Gerbil

**Use for:** stable reference oracle, small and medium 2D generators, symbolic/grammar-based generation, coarse region metadata, and any measured workload where Gerbil meets the application SLO. Gerbil can be compiled through Gambit, but this alone does not establish high-performance voxel output.

**Required evidence:** same input produces exact or selected-profile-equivalent output across qualified reruns; bounded heap behavior; correct RNG sequence and numeric contract; no accidental `eval`/host side effects; measured warm/cold runtime. Start with an independently implemented deterministic scalar field or simple cellular rule.

### B — Gerbil + native accelerator

**Use for:** profiled hotspots where a native field sampler, SIMD kernel, mesh extractor, GPU shader, memory-mapped buffer producer or terrain worker shows end-to-end advantage. Rust, C, C++, Zig or a qualified GPU stack are candidates.

**Required evidence:** approved FFI/IPC ABI, data layout, ownership, buffer validation, no stale native callback effects, error behavior, numeric equivalence grade, benchmark including copying/marshalling, toolchain pin and target-platform support. Keep the Gerbil coordinator in charge of recipe, world/revision checks and public receipts.

### C — Gerbil + external generation service

**Use for:** remote/distributed expensive tasks, heterogeneous solvers, cloud rendering/generation farms or isolated environments with independently demonstrated benefit. The service is not a privileged source of world truth.

**Required evidence:** authenticated request scope, request/idempotency identity, admission policy, quota, transport/schema version, retry/timeout semantics, unordered completion, result digest+revalidation, ambiguous acknowledgement reconciliation, service version, budget/cost accounting and explicit network/availability risk. No promise of global exactly-once behavior or distributed ACID.

### 9.1 Selection decision register

| Question | Rule |
|---|---|
| Is the output part of **authoritative saved-world semantics**? | Require the declared exact/compatible reproducibility grade and available old-generator closure; decline weaker backends |
| Does A meet measured end-to-end latency, memory and quality targets? | Prefer A for simplicity/portability if qualification is otherwise equal |
| Does B outperform A including transfer/copy cost, with equivalent or explicitly admitted numerical behavior? | Select B **for that exact hot-path profile only** |
| Does C meet error/availability/cost budget and provide needed scale? | Select C where its operational advantages outweigh serialization/network uncertainty |
| Does a chosen backend fail exact compatibility? | Refuse for save-critical jobs, or use a separately qualified **non-authoritative visual** profile |
| Is no backend qualified for a requested combination? | Return `UNSUPPORTED_PROFILE`/`NO_QUALIFIED_BACKEND`; no guessed automatic downgrade |

Backend choice can vary by stage or job: Gerbil may compute a rule graph, a native library may sample dense noise, and a service may build an offline planetary asset. The execution provenance records the selected composition. No automatic runtime self-replacement of code based only on unverified marketing benchmarks.

## 10. Gerbil authoring, package structure and version locks

**Staging:** trusted macros can lower into a strictly validated inert `GenerationIR` before any execution. User mods, downloaded recipes, external JSON or AI-generated text remain untrusted *data*, never directly executed Scheme. Macro hygiene and checked interfaces do not equal an OS sandbox. Every generated native/remote adapter is qualified separately.

```text
editions/game-world-procedural/
  src/
    contract/{manifest,recipe,materialization,artifact,receipt}.ss
    semantic/{ir,stage-graph,closure,compatibility}.ss
    fields/{rng,noise-reference,cellular-reference}.ss
    geometry/{heightfield,volume-profile,mesh-contract,lod-contract}.ss
    biome/{classification,placement}.ss
    coordination/{budget,job,cache,qualification}.ss
    ports/{d02-region-demand,d01-world-authoring,p06-history-optional}.ss
    backends/{gerbil-reference,native-adapter,service-adapter}.ss
    authoring/{normalize,diagnostics,macro-sugar}.ss
  tests/{contract,noise,space,seams,geometry,biome,job,backend,compatibility}/
  fixtures/{tile-2d,cellular-growth,cave-volume,planet-chart,
            lodded-neighbors,world-upgrade,nonlocal-overlay}/
  qualification/{gate-matrix,bench-profile,executed-evidence}/
```

This is a **candidate source topology**, not an installed Gerbil package tree or a claim any code has compiled.

**Illustrative declaration** (profile syntax to be specified and compiled only after actual Gerbil API probes):

```scheme
(define-generation-profile first-frontier
  (world-class grid-world-2d)
  (seed-domain versioned-region-streams)
  (stages
    (heightfield (algorithm value-noise-v1) (octaves 4))
    (biome-map (inputs height moisture) (rule-set biome-v1))
    (placements (algorithm bounded-cell-sampling-v1)))
  (output (region-artifact semantic-v1))
  (execution (candidate-backends gerbil native service))
  (parity (saved-world exact) (visual numeric-tolerance))
  (admission (no-direct-world-mutation)))
```

The declaration itself is not executable authority. A validated recipe is only admitted for its pinned implementation and explicitly installed backends.

## 11. Performance methodology and benchmark fairness

### 11.1 Measurement contract

Benchmark **end-to-end region generation** rather than only a noise kernel. Record `reference machine/CPU/GPU`, OS and architecture, Gerbil/Gambit/CC/runtime versions, backend commit and compiler flags, input fixture and algorithm revision, output class, cold/warm status, sample count, repetition count, p50/p95/p99 latency, throughput (regions/s, samples/s, triangles/s as appropriate), peak resident/staged memory, allocations, serialized bytes, cache hit rate, cancellation cost, setup/FFI/IPC/network overhead and correctness/quality grade.

Sample workloads: (1) small `64x64` 2D field; (2) medium `512x512` mixed biome+placement tile; (3) `32^3` bounded density field; (4) a cave/mesh boundary fixture; (5) mixed LOD neighbor set; (6) staged planetary-chart case; (7) cold revisits and generation-version migration. These are *benchmark sizes to attempt*, not achieved timings, throughput targets or mandatory hardware requirements. Scale up only with measured RAM/time safeguards.

### 11.2 Exact comparison protocol

- Same canonical inputs, region/window, seed streams, stage graph and output contract for A/B/C.
- Same output classification: equality grades must not be chosen after seeing a mismatch.
- Independent oracle checks geometry/schema, seams, placements and world/overlay version binding.
- Measure both pure compute time and **total user-observed time** including backend setup, materialization transfer and D02 validation.
- Assess cold start, warm cache, sequential generation, random-access revisits, concurrent loads and canceled/stale work.
- For C, measure network failure, lost ack and service drift, not only p50 on a good connection.
- Record sample size and uncertainty intervals where available; warmup/repetition are reproducible. Reject unjustified universal "X is faster" statements.
- Energy use and monetary cost are optional measured dimensions, not invented values.

**Decision:** a slower but exact/reconstructible backend may beat a faster incompatible one for a saved-world stage. A fast visual-only backend can coexist with an exact canonical semantic generator. Quality, portability and operational risk join speed in the selection, not merely peak FLOPS.

## 12. Worked scenarios and falsifying counterexamples

**F1 — Pure 2D settlement:** same world seed and region coordinate produce identical canonical height/biome/placement descriptors across requests regardless of generation order. D02 accepts a locked result; D01 has not been asked to decide any outcome. Failure if one global RNG causes order-sensitive output.

**F2 — Shared edge:** adjacent heightfield tiles generated by different worker schedules share exactly the declared edge samples and boundary convention. Failure if a seam appears from independent local-origin rounding.

**F3 — Changed future, same terrain:** D01 records a resource conflict causing a settlement's future economy to change but leaving its geometric terrain baseline unchanged. D03 may reuse a *qualified geometry-only* cache, while D02 keeps the causal overlay separate. Failure if reusing geometry mistakenly preserves stale resource counts.

**F4 — Historical revision preservation:** an older inhabited planet pins generator `g1`; new exploration space uses `g2`. D03 can still regenerate a visited g1 region and does not overwrite a base built on it. Failure if updated code silently regenerates the old surface with g2.

**F5 — A/B parity:** Gerbil noise reference and a Rust/C sampler yield compatible canonical outputs under a named parity profile. If float thresholds cause topology divergence, the native result cannot be used for save-critical geometry even when sample RMSE is tiny.

**F6 — C ambiguous completion:** a remote worker generates region R, but transport acknowledgment is lost. The coordinator returns `UNKNOWN` and reconciles by exact request/result identity; it does not create two canonical world artifacts or claim delivery based on HTTP acceptance.

**F7 — Cave and voxel seam:** a volumetric cave intersects two LOD chunks. A qualified mesher preserves the named seam topology; when its transition mode is unsupported, it returns a typed failure rather than claiming the surface is watertight.

**F8 — Sparse planetary world:** a planet has millions of conceptual addresses but only a tiny bounded working set. D03 creates only requested artifacts, while D02 owns residency and eviction. Unrequested regions are not assumed nonexistent or causally unaffected.

**F9 — Hidden biome:** an observer cannot access a region's secret mineral placement. The underlying generator can compute it under an authorized profile, but D02/host projection must not leak exact deposits through a public response or cache key. D03 cannot self-grant disclosure.

**F10 — Growth snapshot:** a bounded cellular growth recipe produces an initial vegetation layout. The live simulation then evolves under separately declared world decisions/time dynamics. D03 is not allowed to "regenerate" each tick and erase admitted environmental changes.

## 13. Invariants and typed failure taxonomy

### 13.1 Invariants I01–I24

1. **I01 — Ownership separation:** generation produces artifacts; D01 governs consequences; D02 owns world-state/residency; P06 owns history.
2. **I02 — Exact public port:** D03 accepts D01/D02's materialization request/result field semantics without a hidden incompatible replacement.
3. **I03 — Identity separation:** world/branch/region/artifact/generator/overlay/scene identity are never inferred from labels or one seed.
4. **I04 — Version closure:** algorithm, numeric, RNG, coordinate, schema, material/asset and dependency revisions are pinned when relevant.
5. **I05 — Input closure:** every actual result-affecting input is declared and bound to the receipt, or replay claims are refused.
6. **I06 — Order-independent addressability:** independent regions do not change canonical output when scheduled in different orders.
7. **I07 — No false parity:** byte, numeric, topology, semantics and appearance are separate equivalence classes.
8. **I08 — Geometry truth discipline:** LOD/render proxies cannot mutate or certify canonical world semantics.
9. **I09 — Seam honesty:** continuity/watertightness is claimed only for named qualified boundary/LOD profiles.
10. **I10 — No direct mutation:** generated content does not self-commit world-state effects or historical branches.
11. **I11 — Admitted initialization:** initial world baseline enters canonical state through a separately validated authoring/admission capability.
12. **I12 — Saved world preservation:** old inhabited regions retain reconstruction-relevant generator/input revisions or explicitly report loss.
13. **I13 — Stale request rejection:** late or canceled work cannot bind to newer region/world/request generations.
14. **I14 — Deterministic numeric scope:** cross-platform exactness is never inferred from equal seeds alone.
15. **I15 — Random stream isolation:** one backend's stage order cannot perturb unrelated region/stream output.
16. **I16 — Algorithmic boundedness:** samples, recursion, mesh vertices, jobs, staging bytes and outputs obey declared enforceable limits.
17. **I17 — Backend capability truth:** unsupported native or remote features fail explicitly, never silently promote into qualified execution.
18. **I18 — Non-privileged recipes:** untrusted source data cannot trigger arbitrary Scheme eval, FFI calls, network, file writes or authority elevation.
19. **I19 — Lease truth:** memory/native buffers remain pinned or uncertain until actual release acknowledgments, not mere cancellation requests.
20. **I20 — Observer isolation:** generated hidden-region artifacts are not automatically public observations.
21. **I21 — Independent qualification:** Gerbil, native and service variants each require exact profile-specific execution evidence.
22. **I22 — No invented NMS source:** public NMS talk/patch features cannot be marketed as recovered proprietary engine kernels.
23. **I23 — Public port compatibility:** D03's materialization adapter must preserve D01/D02 request and response semantics, including version locks, result validation and the boundary between generated representations and authoritative world state.
24. **I24 — No false maturity:** proposed tests and sources are not runtime or performance qualification.

### 13.2 Failures and uncertainty

| Failure/status | Operational response |
|---|---|
| `INVALID_RECIPE`, `UNSUPPORTED_FIELD`, `UNKNOWN_STAGE`, `INVALID_COORDINATE_FRAME` | Reject inert input before any dispatch |
| `GENERATOR_VERSION_MISSING`, `RECONSTRUCTION_UNAVAILABLE`, `INCOMPATIBLE_SCHEMA` | Do not substitute newest algorithm in a preserved world |
| `RNG_REVISION_MISMATCH`, `NUMERIC_PARITY_UNQUALIFIED`, `TOPOLOGY_MISMATCH` | Refuse stronger reproducibility or saved-world claim |
| `STAGE_DAG_CYCLE`, `UNBOUNDED_RULE`, `BUDGET_EXHAUSTED` | Terminate/return bounded partial result without false success |
| `SEAM_PENDING`, `LOD_TRANSITION_UNSUPPORTED`, `MESH_INVALID` | Report limited geometry/LOD capability |
| `STALE_WORLD_GENERATION`, `STALE_REGION_REQUEST`, `OVERLAY_REVISION_CONFLICT` | Quarantine late output; do not publish to D02 current view |
| `NATIVE_ABI_UNQUALIFIED`, `BUFFER_LEASE_UNKNOWN`, `GPU_PROFILE_UNSUPPORTED` | Reject/pin unsafe resources; no invented release success |
| `REMOTE_TIMEOUT`, `REMOTE_ACK_UNKNOWN`, `SERVICE_REVISION_DRIFT` | Unknown/reconcile; no blind reissue of non-idempotent work |
| `OBSERVER_SCOPE_VIOLATION`, `AUTHORITY_NOT_ESTABLISHED` | No hidden artifact leak or unapproved effect |
| `ARTIFACT_PARTIAL`, `QUALITY_NOT_ESTABLISHED`, `NO_QUALIFIED_BACKEND` | Report exact scope; never silently upgrade to complete/correct |

## 14. Qualification design and measurable evidence

**No implementation, benchmark or qualification result is asserted in this document.** The tests below are specifications for future work, not cases already executed. Every test records the generator/algorithm/numeric/RNG revisions, oracle implementation, toolchain, environment and exact backend capabilities. Failing tests must remain visible and may disqualify only the affected profile.

### 14.1 Qualification gates Q0–Q11

| Gate | Minimum evidence | Allowed claim ceiling |
|---|---|---|
| **Q0 — Toolchain and source lock** | Exact Gerbil/Gambit toolchain, algorithm versions, source/license records, build closure | Declared platform only |
| **Q1 — Schema and public-port parity** | D01/D02 materialization compatibility, strict version and identity fixtures | Typed contract only |
| **Q2 — Reference generator correctness** | Golden 2D field, RNG isolation, bounded growth/placement oracle | Named algorithm and numeric profile |
| **Q3 — Geometry/seam validity** | Neighbor/halo, LOD, mesh indices/topology and planetary chart tests | Named spatial/LOD representation |
| **Q4 — Generation lifecycle** | Stage DAG, budgets, cancellation, stale result, partial state | Local bounded job model |
| **Q5 — Version and world preservation** | Old generator replay, revision switching, inhabited overlay continuity | Exact declared reconstruction profile |
| **Q6 — Backend A** | Compiled/running Gerbil tests + end-to-end benchmark | Exact workload/toolchain only |
| **Q7 — Backend B** | ABI/buffer/thread, parity and measured end-to-end improvement or justified capability | Qualified native platform and stage |
| **Q8 — Backend C** | Transport versioning, authority, latency/cost/failure, ack reconciliation | Qualified service/availability profile |
| **Q9 — Observer / authority safety** | Recipe inertness, permission projection and side-effect tests | Tested threat/scope model only |
| **Q10 — Performance and quality tour** | Comparable p50/p95/p99, memory and output-quality reports across eligible backends | Exact hardware/input/profile only |
| **Q11 — Independent audit** | Adversarial fixtures, failure injection, artifact/provenance reinspection | Qualified subset of profiles, never entire portfolio |

### 14.2 Proposed 80-test register

#### Family 1 — Identity & public port

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T01** | Same seed, different world refs | Different identity and cache scope |
| **P08-D03-T02** | Same coordinates, different partition/frame revision | No stale region alias |
| **P08-D03-T03** | D01 public MaterializationRequest exact fixture | D03 parses without hidden mandatory fields |
| **P08-D03-T04** | D02 receipt response fixture | All required output fields preserved |
| **P08-D03-T05** | Missing generator revision | Typed refusal before dispatch |
| **P08-D03-T06** | Unknown required recipe tag | Strict inert-data rejection |
| **P08-D03-T07** | Duplicate canonical identifiers | No ambiguous construction |
| **P08-D03-T08** | Old result rebinds to reused request generation | Stale token rejected |

#### Family 2 — RNG & reference fields

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T09** | Same 2D region and full seed closure repeated | Exact reference field equality |
| **P08-D03-T10** | Regions generated A/B versus B/A | Each output invariant to scheduling order |
| **P08-D03-T11** | Random cancellation then regeneration | Unchanged region identity/results |
| **P08-D03-T12** | Stage-specific stream names overlap | Domain separation preserves independence |
| **P08-D03-T13** | Negative coordinates and region boundaries | Canonical sample agreement |
| **P08-D03-T14** | Invalid octave/scale or non-finite input | Reject before sampling |
| **P08-D03-T15** | Golden field values across compiled/interpreted qualified profile | Named numeric oracle passes |
| **P08-D03-T16** | Tiny-budget cellular/grammar growth | Terminates with typed bounded status |

#### Family 3 — Geometry & region seams

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T17** | Adjacent 2D tile halo samples | Identical edge values |
| **P08-D03-T18** | Heightfield output has NaN/Inf | Invalid geometry rejected |
| **P08-D03-T19** | 3D density tile with cave cross-boundary | Correct sign/boundary convention |
| **P08-D03-T20** | Mesh index exceeds vertex array | Reject invalid mesh |
| **P08-D03-T21** | Degenerate triangles under strict mesh profile | Detected and policy-handled |
| **P08-D03-T22** | Different LOD volume seam | Qualified stitching or explicit SEAM_PENDING |
| **P08-D03-T23** | Mixed generator revisions at neighbor boundary | Reject without migration/seam witness |
| **P08-D03-T24** | Planet chart face transition | Transform/continuity within declared error |

#### Family 4 — Biomes, growth & population

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T25** | Biome threshold on exact boundary | Stable tie-break and classification |
| **P08-D03-T26** | Conflicting biome rules | Typed conflict or named priority |
| **P08-D03-T27** | Placement within forbidden/occupied site | Candidate rejected; no silent relocation |
| **P08-D03-T28** | Placement at tile edges generated in reverse order | No duplicate/omitted canonical owner |
| **P08-D03-T29** | Over-cap population candidate request | Bounded failure or declared partial output |
| **P08-D03-T30** | Growth grammar recursive without progress | Fuel exhaustion, not infinite loop |
| **P08-D03-T31** | Shared asset definition in different scenes | Distinct placement/world identities |
| **P08-D03-T32** | Geometric prop mistakenly treated as canonical NPC | World mutation refused |

#### Family 5 — World overlays & preservation

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T33** | D01 decision changes economy but not geology | Qualified geometry reuse; overlay remains separate |
| **P08-D03-T34** | D01 decision changes terrain semantics | Old representation invalidated/revalidated |
| **P08-D03-T35** | Inhabited world retains g1 during g2 expansion | Old region reconstructed under g1 |
| **P08-D03-T36** | Old generator artifact missing after upgrade | RECONSTRUCTION_UNAVAILABLE |
| **P08-D03-T37** | Seed same but branch/overlay differs | No unsafe cache alias |
| **P08-D03-T38** | Late g1 completion after world g2/o6 | Quarantine stale result |
| **P08-D03-T39** | Visual LOD downgrade during interaction | Authoritative collision/topology unchanged |
| **P08-D03-T40** | Unknown nonlocal causal input changes recipe dependency | No false unaffected claim |

#### Family 6 — Stage graph & resource limits

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T41** | Cycle in stage dependency graph | Reject before execution |
| **P08-D03-T42** | Unknown stage implementation revision | No fallback without qualified profile |
| **P08-D03-T43** | Intermediate output exceeds byte quota | Stopped and typed partial/failed |
| **P08-D03-T44** | Max volumetric samples exceeded | No unchecked allocation |
| **P08-D03-T45** | Request cancellation during field sampling | Cancellation recorded, release separately observed |
| **P08-D03-T46** | Concurrent duplicate region requests | Coalesced only under exact lock/scope parity |
| **P08-D03-T47** | Low-priority jobs starved by high-priority stream | Fairness policy detected/qualified or limitation declared |
| **P08-D03-T48** | D02 evicts region with dirty canonical changes | No data loss or false success |

#### Family 7 — Backend A & numeric portability

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T49** | Gerbil compiled reference field fixture | Exact version-locked sample/output oracle |
| **P08-D03-T50** | Gerbil pure reference under controlled memory pressure | Memory cap or explicit unsupported enforcement |
| **P08-D03-T51** | Alternative numeric reduction order | No fabricated bitwise identity |
| **P08-D03-T52** | Cross-platform float values near mesh threshold | Topology rechecked independently |
| **P08-D03-T53** | Golden result under different Gerbil compiler lock | Requalification or compatible revision witness |
| **P08-D03-T54** | Nontrivial workload all stages enabled | End-to-end time and peak memory recorded |
| **P08-D03-T55** | No external libraries installed | Pure Gerbil reference still importable |
| **P08-D03-T56** | Pure generator attempts forbidden IO | Effect neutrality failure detected |

#### Family 8 — Backend B & FFI

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T57** | Native ABI version mismatch | Reject before buffer access |
| **P08-D03-T58** | Buffer size/alignment claim false | Reject unsafe input |
| **P08-D03-T59** | Native worker fails mid-batch | Partial result/lease uncertainty preserved |
| **P08-D03-T60** | Native cancellation races with completion | No double release or stale output activation |
| **P08-D03-T61** | Wrong-thread native release | Affinity violation/refused |
| **P08-D03-T62** | Gerbil/native exact-reference small fixture | Named parity or clear disqualification |
| **P08-D03-T63** | Faster kernel but slower total marshaled region | End-to-end selection does not misrank |
| **P08-D03-T64** | Native output valid geometry but altered semantic occupancy | Refuse stronger semantic parity |

#### Family 9 — Backend C & observer boundary

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T65** | Remote response after timeout | Unknown/reconciled by exact job ID |
| **P08-D03-T66** | Service changes algorithm revision mid-run | Version drift rejected |
| **P08-D03-T67** | Network duplicate delivery | No duplicate authoritative publication |
| **P08-D03-T68** | Remote job exceeds cost/byte quota | Typed refusal and bounded usage |
| **P08-D03-T69** | Untrusted recipe contains Scheme evaluation form | Treated as inert data, no eval |
| **P08-D03-T70** | Hidden-region generation returned to public observer | D02 scope boundary prevents disclosure |
| **P08-D03-T71** | Remote payload forged under untrusted channel | Authenticated validation failure |
| **P08-D03-T72** | Service says accepted but no final result | No completed or durable claim |

#### Family 10 — Qualification & migration audit

| Test | Challenge | Pass oracle |
|---|---|---|
| **P08-D03-T73** | A/B/C runs have differing output equivalence grades | No false single PASS across all stages |
| **P08-D03-T74** | Bench cold/warm mismatch or fixture drift | Reject comparison as non-comparable |
| **P08-D03-T75** | High p95 tail despite low average | Reported selection risk, not hidden |
| **P08-D03-T76** | Old scene/asset revision required on revisit | Original lock or explicit incompatibility |
| **P08-D03-T77** | License missing for imported lookup table | Admission blocked pending provenance |
| **P08-D03-T78** | NMS GDC talk cited as shipped code proof | Source-lineage claim rejected |
| **P08-D03-T79** | One rejected backend mixed into qualified set | Partial qualification only, no blanket release |
| **P08-D03-T80** | Independent game + non-game fixture consume same reference stage | Declared shared law holds; domain claims remain distinct |

### 14.3 Qualification reporting matrix

`DESIGN_ONLY` means no tests run. `MOCK_QUALIFIED` means only exact reference/mock fixtures passed; it does not establish native throughput or service security. `GERBIL_REFERENCE_QUALIFIED` is available only after executing the exact Gerbil toolchain Q0/Q1/Q2/Q4/Q6/Q9 subset with all required negative tests. `NATIVE_ADAPTER_QUALIFIED` additionally requires Q7 and corresponding B fixtures; `SERVICE_ADAPTER_QUALIFIED` requires Q8 and corresponding C fixtures. `PERFORMANCE_PROFILE_QUALIFIED` requires Q10 under declared targets. `RELEASE_QUALIFIED` requires Q11 and an explicitly declared limited module set, not blanket P08-D03 qualification.

Every verdict binds an exact `GeneratorManifest`, backend implementation, output/profile and observed evidence. Failure of C is not failure of A, and success of A is not automatic validation of B/C.

## 15. Design rollout, open questions and decisions

### 15.1 Sequential design-to-implementation slices (separate authorization)

1. **D03-00 — Source/license ledger:** exact NMS public evidence; inspect available open noise/mesh implementations; Gerbil/Gambit version lock, portability and owned provenance.
2. **D03-01 — Inert contracts + manifest:** compile/public-port parity with D01/D02, canonical identity, RNG/numeric/recipe validators and negative cases.
3. **D03-02 — Gerbil 2D reference:** one small noise/cellular stage, one biome classifier, one bounded placement stage; prove exact region order-independence and edge seams.
4. **D03-03 — D02 fake region integration:** request, generate, version-check, observe, evict and reconstruct without P06 or rendering engine.
5. **D03-04 — Fair algorithm tour:** identify actual hotspots and compare A versus available B with equal outputs, both kernel-level and end-to-end, using the same fixtures. No native port mandated.
6. **D03-05 — Optional 3D/planet profiles:** density, meshing, LOD/seams, chart transforms and planetary variety only after independent correctness/budget evidence.
7. **D03-06 — C only if justified:** remote generation and failure/ack reconciliation if real workload scale requires it. Do not create a service for a tiny single-host game.
8. **D03-07 — Independent audit + interface conformance:** execute relevant Q0–Q11; verify D01/D02 materialization port compatibility, version locks, negative cases and evidence scope.

### 15.2 Open issues

| Question | Preferred baseline | Evidence needed |
|---|---|---|
| Default generator shape | Pure 2D tile/field, Gerbil reference | Reproducible correctness and memory benchmark |
| World coordinates | D02 `GRID_2D` exact integer scope initially | Planetary chart/precision tests before 3D scale |
| Canonical vs visual geometry | Explicitly separate output classes | Independent gameplay topology/physics oracle |
| NMS-like planet scale | Treat as staged target, not a first-slice promise | 3D/planet/LOD benchmark and continuous streaming test |
| Generator revision upgrades | Existing inhabited worlds pin old revision | Source closure/reconstruction and seam migration tests |
| Native code languages | Open: Rust/C/C++/Zig or selected GPU backend | ABI, licensing, parity, platform and end-to-end data |
| Remote service | Unnecessary unless measured scale justifies it | Availability, budget, auth and retry qualification |
| Algorithm source choice | Independently specified first; lawful open source when useful | Code provenance/license and semantic parity |
| D01/D02 port conformance | Preserve the existing materialization contract | Cross-design request/response compatibility and negative adapter tests |
| Shared cross-domain value | Only separately qualified portable mechanisms | Non-game consumer fixture and invariant test |

## 16. Final design verdict

`P08_D03_GAME_WORLD_PROCEDURAL_GENERATION_DESIGN_PROPOSED / GAMING_TRACK / PUBLIC_NMS_PIPELINE_NOT_PROPRIETARY_PORT / P08_D01_DECISION_SEPARATE / P08_D02_CANONICAL_WORLD_SEPARATE / P06_HISTORY_SEPARATE / THREE_PER_JOB_BACKEND_PROFILES_A_B_C / PURE_GERBIL_FIRST_NOT_GERBIL_ONLY / REGION_SEED_AND_GENERATOR_VERSION_LOCKS / 2D_TO_3D_AND_COSMOS_INCREMENTAL / NO_FALSE_CROSS_BACKEND_PARITY / EIGHTY_PROPOSED_TESTS / Q0_TO_Q11 / NO_RUNTIME_PROOF / NO_BENCHMARK_PASS / NO_IMPLEMENTATION / DOCUMENT_PUBLISHED`

> **Closing principle:** A world generator may invent mountains, forests, caves and distant planets, but it does not get to rewrite the world's causal decisions or erase a player's past. Gerbil owns the generation semantics and public contracts; the chosen backend earns its right to compute through evidence, not language preference. Large worlds are made accessible by stable addressing, lazy work, versioned recipes and honesty about what has actually been generated.