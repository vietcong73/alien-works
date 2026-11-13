# LFR-P08-D02 | Game World & Scene Architecture

**Document class:** Source-grounded detailed design proposal, one specialist design slice  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `GAMING` — Gaming & Interactive Worlds  
**Edition:** P08 — World, Scene & Spacetime Decision Systems  
**Design slice:** D02 — Game World & Scene Architecture  
**Primary implementation target:** Independent Gerbil Scheme semantic/reference world substrate  
**Primary dependency:** P08-D01, *Game World Spacetime Decision & Causal Consequence Architecture* (committed, design-only)  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P08-D02-game-world-scene-architecture-2026-11-13.md`  
**Status:** `DETAILED_DESIGN_PROPOSED / GAMING / SOURCE_GROUNDED / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. This publication does not authorize code implementation, installation, source porting, or deployment.

> **Design decision:** P08-D02 owns **the representation, structural consistency, lifecycle, accessibility, and publication of a game world's spatial and scene state**. P08-D01 owns **what decisions mean and which world consequences are admissible**. P09 owns **how procedural content is generated**. P06 owns **historical lineage and synthesis**. A world can be implemented as sparse immutable regions with a small Gerbil reference store without adopting an ECS, renderer, physics engine, procedural generator, RL runtime, or scene toolkit.

## 0. Problem, deliverable, and correctness boundary

A decision system is not a usable world substrate unless an application can answer, consistently and with bounded resources: *Which world? Which region? Which entity generation? Which scene representation? Which revision? Which observer is permitted to see it? Which materialization is current?* P08-D02 supplies those answers **without redefining the rules that caused the world to change**.

### 0.1 In scope

1. World, scene, region, entity, component/view, instance and layer identities; their independent lifecycle and typed relations.
2. Spatial frame and region addressing, including graph-like, 2D, 3D and hierarchical spaces without one universal coordinate metric.
3. World snapshots, structural manifests, sparse region data, generation counters and conservative dependency boundaries.
4. Explicit scene composition, instancing, visibility and observer-relative projection; scene structure does not define causal truth.
5. Region load/materialize/activate/suspend/evict lifecycle, budgets, asynchronous request identity, stale-response rejection and bounded streaming policies.
6. Generation/overlay version locks and a P09 materialization port, **not** a procedural algorithm implementation.
7. Single-writer, in-memory world-state reference store with version-checked publication and optional persistence adapters.
8. Explicit integration contract with P08-D01 for state reads, delta validation, successor admission and observation receipts.
9. Negative tests for entity ABA reuse, spatial misidentification, forgotten effects after eviction, partial region state, future/branch confusion, stale generation, data leakage and unsupported consistency claims.

### 0.2 Out of scope

- P08-D01 causal-rule authoring, intervention logic, outcome selection or prediction; P06 branch DAG, time-travel laws, counterfactual semantics or N-ary historical synthesis.
- P09 noise, terrain, voxels, planetary generation, NMS proprietary algorithms, biome synthesis and backend optimization.
- Rendering, animation, audio, GPU, shader/material, physics/collision solving, pathfinding/navigation, ECS job scheduling, networking, distributed consensus, game/editor UI, RL agent or trainer.
- Global universal unit system; mandatory x/y/z coordinates; automatic real-world durability; semantic equivalence of unrelated game engine frameworks.

**Non-overlap principle:** P08-D02 may validate structural well-formedness and identity before committing a state, but **cannot declare a D01 intervention causally admissible** by merely accepting its shape. A successful streaming request is neither a world action nor a historical commit.

## 1. Evidence ledger and what actually transfers

Evidence classes: **SOURCE** = directly inspectable original source; **DOC** = public official engine/language documentation; **DESIGN** = existing LFR design; **PROPOSED** = P08-D02 requirement. A documented API does not prove a new Gerbil implementation is correct or fast.

| Source/evidence | Mechanism or lesson | Transfer to P08-D02 | Explicit non-transfer |
|---|---|---|---|
| P08-D01 (DESIGN, committed) | `WorldAnchor`, `SpaceScope`, world decisions, CAS publication and observer separation | D02 implements the world-state read/validate/publish port and preserves exact revision locks | D02 must not choose the consequence of an intervention |
| P06-D01 (DESIGN, committed) | Immutable historical identity, ancestry, reconstruction and branch references | Optional history reference/version in world anchors and separate history port | No duplicate causal DAG, history merge or time travel in D02 |
| K0-D01 (DESIGN, committed) | `IdentityRef`, `VersionRef`, validation, lease, result and receipt | Stable typed refs and explicit status/uncertainty | No security, backend durability or effectful authorization inferred from a K0 record |
| K1-D01 (DESIGN, committed) | Graph, observation, bounded operations, snapshot and numeric profiles | Optional qualified mechanism use where an invariant truly matches | No mandated all-in-one K1 runtime or global ECS ontology |
| Godot `SceneTree`, `Node`, `PackedScene` (DOC) | Runtime node hierarchy, scene instantiation, owner-dependent packing and lifecycle | Distinguish parent, scene owner, persistence owner and runtime instance; scene activation lifecycle | Godot's tree, main loop, node notifications and exact packing semantics are not adopted wholesale |
| Bevy ECS docs (DOC) | Entity/component/system organization and decoupled processing | Components can be typed partial views of canonical world entities | No ECS archetype layout, parallel scheduler, or mandatory component storage |
| Unreal World Partition/Data Layers/HLOD (DOC) | Streamable spatial cells; separate runtime data layers and proxy visual detail | Explicit region working sets, streaming sources, proxy/view data, independent visibility | No unreal `UWorld`, native streaming algorithm, HLOD renderer or speed guarantee |
| OpenUSD introduction (SOURCE/DOC) | Hierarchical scene description; non-destructive layer opinions; reference/payload composition | Keep asset definition, scene instance, layer override and deferred materialization separate | No USD format reader or implicit USD composition strength ordering in Gerbil |
| Godot large-world coordinates (DOC) | Floating-point precision loss away from origin | Coordinate-frame IDs, units, bounds, validity and optional local-origin transformations | No promised precision or physical stability for arbitrary planetary scales |
| Alien-Works review / native resource sources (SOURCE/DESIGN) | Native resource/callback lifetimes and affinity hazards | Treat renderer/native/asset handles as owned adapter resources with separate generations | No Alien-Works import, unqualified FFI or native renderer inside D02 |
| No Man's Sky public generation talks (PRESENTATION) | Large procedural worlds require a distinct generation/content pipeline | P09 requests must be revision-pinned, lazy and backend-neutral | No claimed access to NMS private generation source or direct port |
| Gerbil official documentation (DOC) | Modules, structs, macros, interface layers, test/build tooling | Plain Gerbil records + pure validation first; limited trusted authoring forms | No API signature is claimed compiled without an exact pinned toolchain probe |

### 1.1 Primary anchors for review

- P08-D01: `docs/design/LFR-P08-D01-spacetime-decision-causal-consequence-design-2026-11-13.md` in `vietcong73/alien-works@audit/lisp-foundation-reserve` (commit `03d729d1d7d84b10789c9e90135ca66f1d695ffc`).
- P06-D01: `docs/design/LFR-P06-D01-branching-causal-reversibility-design-2026-11-13.md` (same branch).
- K0-D01: `docs/design/LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md` (same branch).
- K1-D01: `docs/design/LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md` (same branch).
- [Godot SceneTree](https://docs.godotengine.org/en/stable/classes/class_scenetree.html), [Godot PackedScene](https://docs.godotengine.org/en/stable/classes/class_packedscene.html), [Godot Node](https://docs.godotengine.org/en/stable/classes/class_node.html), [large-world coordinates](https://docs.godotengine.org/en/stable/tutorials/physics/large_world_coordinates.html).
- [Bevy ECS](https://bevy.org/learn/quick-start/getting-started/ecs/).
- [Unreal World Partition](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition-in-unreal-engine), [Data Layers](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---data-layers-in-unreal-engine), [HLOD](https://dev.epicgames.com/documentation/en-us/unreal-engine/world-partition---hierarchical-level-of-detail-in-unreal-engine).
- [Pixar OpenUSD introduction](https://openusd.org/release/intro.html) and [OpenUSD source documentation](https://github.com/PixarAnimationStudios/OpenUSD/blob/dev/docs/intro.rst).
- [Gerbil introduction](https://gerbil.scheme.org/guide/intro.html), [interfaces](https://gerbil.scheme.org/reference/std/interface.html), [tests](https://gerbil.scheme.org/reference/std/test.html), [build](https://gerbil.scheme.org/reference/dev/build), [FFI](https://gerbil.scheme.org/guide/ffi.html).

**Research inference:** ECS, scene trees, asset layers and world streaming are **alternative/complementary organizational tools**. Their intersection supports common identity, view, ownership and lifecycle contracts. It does *not* justify a universal scene graph implementation, an inter-engine semantic equivalence claim or obligatory reuse of their internals.

## 2. Ownership and architecture

```text
                        GAME / EDITOR / WORLD TOOL
                                   |
          +------------------------+----------------------+
          |                                               |
  P08-D01 decision semantics                      Scene/view consumers
  (admissibility, consequences)                  (render, editor, UI)
          |                                               |
          v                                               v
   D02 WorldStatePort <---- P08-D02 WORLD SUBSTRATE ----> ProjectionPort
          |                  world manifest                |
          |                  entity/scene registry          |
          |                  region working set             |
          |                  versioned overlays             |
          |                  in-memory CAS store            |
          |                                               |
          +----------- optional interfaces ---------------+
                      |                |                  |
                  P09 materialize    P06 history      host native/render
                  A/B/C backend      separate DAG     and persistence

     D02 + pure mocks require K0 vocabulary, Gerbil and no other edition.
     Optional K1 mechanism implementations must each qualify separately.
```

### 2.1 Authority and responsibility matrix

| Responsibility | Owner | D02 must never claim |
|---|---|---|
| World-specific decision law and causal propagation | P08-D01 | That structural validation proves causal legality |
| Canonical world/scene/entity/region state and structural lifecycle | P08-D02 | That active scene = whole world |
| Procedural algorithm and backend A/B/C choice | P09 | That a correct seed alone ensures content parity |
| Historical branch topology / reconciliation / synthesis | P06 | That changing a scene path creates a historical branch |
| Event/director semantics | P03 | That region activation is a global game-loop scheduler |
| Native handles and FFI lifetime | P01 / selected host adapter | That Gerbil GC automatically releases GPU/physics handles |
| Durable authoritative store (when selected) | qualified consumer/P05 adapter | That a successful in-memory CAS survives a power failure |
| Renderer, physics, assets, networking | game host / qualified adapters | That spatial structure implies pixel or physics correctness |

**Default reference profile:** one in-memory world, one writer, immutable snapshot values, generational IDs, sparse regions, pure structural checks, mock materializer and explicit observer projection. No OS files or graphics.

## 3. Ontology: seven independent identities

**No object becomes another kind simply because two names or payloads happen to coincide.** D02 defines seven logically separate categories with typed references:

1. **World:** one authoritative model/configuration lineage and manifest, including world-generation/version changes; not a renderer process or a branch ref.
2. **Region:** an addressable spatial/topological scope and its generation; may overlap under a declared partition, with ownership rules explicitly defined.
3. **Scene:** an authored/compiled structure of placements, references, groups and presentation/interaction properties; may be instantiated zero, one or many times.
4. **Entity:** a logical world object whose identity persists across scene changes or region loads according to its named lifecycle policy.
5. **Instance:** a concrete placement/realization of an entity or scene definition in a context; an instance is not automatically a different entity.
6. **Asset:** a reusable definition or immutable content artifact, not a world-state mutation or active scene.
7. **Layer/view:** a bounded composition or visibility policy for a selected observer/target. Distinguish authored layers from authoritative causal overlays.

**Distinct associations, not aliases:** `scene-parent`, `scene-owner`, `spatial-membership`, `logical-affiliation`, `asset-reference`, `presentation-layer`, `causal-dependency`, `historical-ancestry` and `resource-lease` are distinct relation tags and must not be serialized as one generic `parent` edge.

### 3.1 Core value records (normative semantics; API names are provisional)

```text
WorldManifest {
  world_ref, world_model_ref, state_generation,
  region_partition_profile_ref, scene_registry_revision,
  entity_registry_revision, spatial_frame_registry_revision,
  causal_overlay_revision, materialization_policy_revision,
  world_time_ref, history_ref_optional,
  canonicalizer_ref, schema_ref, store_capabilities,
  retention_policy_ref, manifest_version
}
WorldSnapshot {
  anchor: P08-D01.WorldAnchor, manifest_ref,
  region_state_index_ref, entity_state_index_ref,
  structural_relation_index_ref, authoring_layer_stack_ref,
  causal_overlay_ref, materialization_index_ref,
  observation_policy_ref, snapshot_evidence
}
WorldEntityRef {
  identity: K0.IdentityRef, entity_generation,
  world_ref, semantic_kind_ref, lifecycle_profile_ref
}
EntityRecord {
  entity_ref, state_revision, attribute_schema_ref,
  canonical_attributes, relation_refs[],
  primary_authority_owner_ref, spatial_binding_refs[],
  scene_instance_refs[], observer_policy_ref,
  existence_status, validation_receipt
}
SceneDefinition {
  scene_ref, asset_revision, rooted_relation_graph_ref,
  node_specs[], asset_dependency_refs[],
  component_schema_refs[], validation_receipt
}
SceneInstance {
  instance_ref, definition_ref, world_ref, parent_instance_ref_optional,
  owning_scope_ref, transform_frame_ref, instance_revision,
  override_stack_ref, activation_status, generation
}
RegionRef {
  world_ref, partition_ref, frame_ref,
  region_id, region_generation, address_schema_ref
}
RegionRecord {
  region_ref, authoritative_state_ref_or_uninitialized,
  state_revision, materialization_ref_optional,
  overlay_revision, observer_policy_ref,
  structural_dependency_refs[], occupancy_evidence_ref,
  retention_class, lifecycle_status, state_completeness
}
```

**Identity rules:** public IDs and display labels are not equal; world-relative entity IDs are not global IDs; `RegionRef` includes partition/frame revisions; local `instance_id` cannot be interpreted outside its scope. A freed/reused slot must get a new generation. The first reference model may use immutable maps and simple lists; it need not optimize storage layout before qualification.

### 3.2 World-state identity and historical identity

- Two worlds may have byte-identical region payloads but distinct `world_ref`, model locks or causal overlays.
- Two scene instances may share one asset/definition but have different placement, ownership and overrides.
- A branch history ref is **optional** and, when present, is only a typed link to P06. D02 does not interpret historical DAG edges.
- World publication order, authored scene revision, fictional world time and materialization completion timestamp are **four different axes**.
- An unmaterialized region may still have authoritative semantic state: `NOT_MATERIALIZED` is not `DOES_NOT_EXIST`.

## 4. Spatial topology, frames, and addressing

### 4.1 Typed spatial profiles

D02 supports a versioned `SpatialProfile` instead of one coordinate class:

```text
SpatialProfile {
  topology: GRAPH | GRID_2D | GRID_3D | HIERARCHICAL |
            CHARTED_METRIC | USER_DEFINED,
  frame_ref, coordinate_schema_ref, unit_ref_optional,
  precision_profile_ref_optional, partition_ref,
  metric_or_adjacency_contract_ref,
  transform_contract_ref_optional, boundary_policy_ref,
  qualified_operations[], revision
}
```

`GRAPH` supports relational worlds (settlements, doors, trade networks) without a Euclidean geometry. `GRID_2D/3D` supports integer tile/chunk indexes and bounded signed coordinates. `HIERARCHICAL` supports planet/continent/region/scene scope paths but **does not** claim distance/adjacency from ancestry. `CHARTED_METRIC` supports named local charts and qualified transform bridges. Consumers ask `resolve_region(scope, profile)` rather than guessing coordinate meaning from a number of tuple entries.

### 4.2 Precision, frame and transform discipline

- `WorldPosition = {frame_ref, coordinate_tuple, units_ref, precision_profile_ref, source_revision}`. Numbers may be fixed-point, bounded integer or selected float profile. Non-finite values and unsupported transforms reject before mutation.
- Frame transforms are **typed partial operations** with declared domain, invertibility and error tolerance. Do not assume reversibility, associativity under finite precision or exactly identical floats.
- Large-world support may use local-origin views/segmented coordinates. The canonical identity remains region+frame+position, never a lone `float3` far from the origin.
- A rendering LOD/higher precision transform must not silently change causal decision variables, proximity constraints or physics truth. Semantic collision/distance models are separate qualified consumers.
- Periodic/wrapping spaces, portals, non-Euclidean adjacency, overlaid regions and graph edges are allowed only under named topology policies. No unconditional octree/quad tree implementation.

### 4.3 Partition and region ownership

```text
RegionPartitionPolicy {
  partition_ref, frame_ref, membership_evaluator_ref,
  overlap_policy, boundary_tie_breaker_ref,
  authority_ownership_rule, migration_contract_ref,
  query_completeness_contract, revision
}
```

The first profile uses **disjoint, deterministic grid cells** and one authoritative owner per entity-state field. Read-only scene projections or visualization membership may cross many regions. Overlapping canonical partitions require an independently qualified conflict/ownership policy. Regions with missing data are `UNRESOLVED`, not inferred empty. Region membership changes must be versioned and reconcile old/new references without losing the entity.

## 5. Scene composition and structural relations

### 5.1 Scene as description, world as truth

A `SceneDefinition` describes placements, groups, asset references, default attributes and presentation bindings. A `SceneInstance` says where and how that description is realized. A `WorldEntityRef` identifies a logical entity in the world. **One scene definition may have many instances; one entity may be presented in multiple scenes.** An unloaded scene is not proof its entities died. A renderer object is not automatically a world entity.

The implementation may later provide an ECS adapter, node hierarchy adapter or hybrid, but the reference contract uses typed relation graphs and versioned immutable records. No scene architecture must decide which code is called every frame.

### 5.2 Independent relation sets

| Relation | Invariant | Negative example |
|---|---|---|
| `scene-child` | A rooted DAG/tree according to declared scene profile; cycle rejected under strict-tree profile | Two nodes are each other's parent |
| `scene-owner` | Owning scene scope controls serialization/instantiation policy | Child node automatically serialized merely because attached |
| `entity-in-region` | Refers to existing region generation and declared partition scope | Moved entity still in two exclusive authoritative cells |
| `instance-of` | Definition revision and instance generation pinned | Instance silently retargets to new asset revision |
| `asset-depends` | Exact asset refs and deferred resolution policy | Missing asset becomes empty success |
| `component-of` | Typed schema and field owner; no hidden shared mutable component | Foreign scene instance writes canonical health without admission |
| `layer-applies` | Scoped, versioned precedence and visibility | Presentation override changes world-level resource quantity |
| `causal-depends` | Defined only by D01's rules, not inferred from scene parenting | Parent node mistakenly means causal parent |

### 5.3 Layer resolution

Maintain **separate stacks**, not one misleading universal override order:

1. **Authoring/asset composition:** base scene definition, explicit authored variants and instance overrides under a named deterministic precedence policy.
2. **World-semantic causal overlay:** validated canonical world-state changes admitted through D01; not overwritten by an asset load or visual LOD.
3. **Presentation/observer projection:** visibility, temporary renderer details and user-specific display settings, without authority to modify world truth.

`resolve_scene_view(scene_instance, world_anchor, observer, layer_policy)` returns a typed view with field-level provenance and `UNRESOLVED/REDACTED` flags where necessary. A layer conflict cannot be silently solved by “last dictionary entry wins” unless that exact precedence was specified and qualified. Stronger presentation layer values never override canonical causal state.

### 5.4 Definition upgrades and live instances

Changing a `SceneDefinition` yields a **new version**. An existing instance remains pinned until a declared migration is applied. Migration must check renamed/missing nodes, dependent assets, local overrides, identity and compatibility. A new definition cannot retroactively rewrite historical published world snapshots or valid old instance identities. Failure yields `MIGRATION_REQUIRED`/`INCOMPATIBLE_ASSET_REVISION`, not arbitrary auto-rebinding.

## 6. Entity lifecycle and authoritative state ownership

### 6.1 Existence is not residency

```text
EntityExistence = DECLARED | PRESENT | SUSPENDED_IN_WORLD |
                  REMOVED | TOMBSTONED | UNKNOWN
EntityResidency = UNRESOLVED | METADATA_ONLY | RESIDENT |
                  PRESENTED | EVICTED | FAILED
```

The two axes are independent. A merchant who has moved to an unloaded city can remain `PRESENT` in the canonical world while their visual instance is `EVICTED`. A dead character may have a retained tombstone, historical references or an asset definition without being a live entity. `PRESENTED` is scoped by an observer/host: one client showing an entity does not mean every other client is showing it.

**Generational validity:** once an entity identity is tombstoned and its allocator slot reused, any outstanding `EntityRef` with the old generation must return `STALE_ENTITY_GENERATION` and may not silently target the successor. `name`, path and coordinate are never identity keys.

### 6.2 Canonical state and component projections

Canonical entity values live in a **versioned world-entity state store** with explicit field ownership. An optional ECS view can project selected attributes into components. Its cache or render-only transforms cannot independently mutate canonical fields. Conversely, a qualified host may propose state changes, but must pass through the selected world mutation/admission boundary and exact world generation.

A structurally valid `spawn` is not necessarily a causally legal gameplay spawn; a structurally valid `move` is not automatically an admissible fictional teleport. Authoring/build mode may create initial world fixtures under separate authoring authority, while live gameplay uses P08-D01. The two modes have different receipts and cannot be silently confused.

### 6.3 Cross-region movement

A movement proposal that changes authoritative region membership follows one of these precise profiles:

- **Local atomic snapshot:** validate source and destination region refs, entity generation and version locks; publish one new immutable whole-world manifest through one expected-version CAS. No half-moved canonical entity is exposed.
- **Staged/non-atomic backend:** return `STAGED/UNRESOLVED` with an explicit migration transaction and required reconciliation. Do not mark two regions as authoritative owners simultaneously. A stronger all-or-nothing request must be refused.
- **Remote/distributed backend:** out of scope until separately designed and qualified. An API returning `202 Accepted` is not a completed move.

For the initial in-memory profile, the first profile is the only supported publication mode. Read-only scene instances in multiple regions may refer to the moving entity without duplicating authoritative state.

## 7. Region lifecycle, streaming and materialization

### 7.1 State machine for a region working set

```text
UNRESOLVED --request--> REQUESTED --dispatch--> FETCHING
    |                                    |           |
    |                                  cancel       v
    |                                             VALIDATING
    |                                                |
    |                                    invalid ----+---- valid
    |                                    |                 |
    v                                    v                 v
METADATA_ONLY                         FAILED         RESIDENT_INACTIVE
                                                        |
                                                     admit use
                                                        v
                                                    ACTIVE_VIEW
                                                        |
                                                    suspend/evict
                                                        v
                                                    EVICT_PENDING
                                                        |
                                                release confirmed
                                                        v
                                                     EVICTED
```

This diagram denotes **working-set observation**, not the authoritative entity's existence. `FAILED`, `STALE_RESULT` and `UNKNOWN_RELEASE` are typed side outcomes (not drawn as every arrow). A `cancel` call sets a request state, not proof that an external generator stopped or that all owned resources were freed. `ACTIVE_VIEW` means the region may be accessed by the qualified host for its declared purpose, **not** that D02 owns the game loop or runs physics.

### 7.2 Request and receipt contracts

```text
RegionDemand {
  demand_ref, world_anchor_ref, region_ref,
  requesting_scope_ref, purpose: OBSERVE | EDIT | SIMULATE |
                MATERIALIZE | PREFETCH,
  required_detail_ref, observation_policy_ref,
  priority_class, deadline_class_optional,
  expected_manifest_generation, budget_ref,
  dependency_closure_claim, cancellation_token_ref_optional
}
RegionWorkReceipt {
  demand_ref, request_generation, stage, region_ref,
  input_revision_locks[], resource_usage,
  last_observed_backend_status, data_completeness,
  ready_capability_set[], visibility_scope,
  release_status, failures[], evidence_scope
}
```

A demand may be coalesced with an identical **qualified** request only when all world/branch/model/generator/overlay/detail/observer permissions and input locks are compatible. Two observers with different confidentiality scopes cannot automatically share one unfiltered cache result. Cache reuse is denied when a dependency could modify authoritative semantics and no independent reusable-output witness exists.

### 7.3 Working set policy

A `WorkingSetPolicy` declares `max_resident_regions`, `max_region_bytes`, `max_outstanding_requests`, `max_staged_bytes`, `max_concurrent_backend_jobs`, `retention_pins`, `eviction_priority`, `fairness`, `observation_budget` and `backpressure_policy`. Each quota is attached to a measured resource/capability; unsupported enforcement cannot be claimed from metadata alone.

Demand sources may include camera/viewport, actor location, editor focus, scripted travel destination or background simulation interest, but the **host chooses the sources**. A future teleport target can be prefetched by its explicit region reference. D02 never infers that the user's screen visibility is identical to the world's causal influence radius.

- Priorities and ties require a stable, documented ordering oracle; naïve high-priority-first can starve low-priority regions. A fairness policy is opt-in and independently tested.
- Memory pressure causes typed `BACKPRESSURE/DEGRADED_DETAIL/WAITING_FOR_BUDGET`, not a fabricated empty world.
- Pending cancellation/timeout cannot reclaim native resources until the selected backend confirms its lease release; unknown release remains charged/pinned conservatively.
- A high-detail proxy must never become the source of truth for gameplay collision, resource counts, causal relationships or entity lifetimes merely because it was rendered.
- Prefetch can fail without invalidating the authoritative world. Gameplay requiring a missing region must declare a `BLOCK/DEFER/FAIL_SAFE/EXPLICIT_PARTIAL` accessibility policy.

### 7.4 Eviction and resurrection

Eviction removes **resident/materialized representation**, subject to retention and dependency pins. It does **not** delete canonical state, historical ancestry, admitted world deltas, causal dependency metadata or pending effects. Before releasing a resident region, require either: (a) all authoritative changes already published to the selected store, or (b) a refusal to evict while an uncommitted canonical delta exists. A mere asynchronous flush attempt is not an observed durable acknowledgment.

On revisit, the region must be reconstructed from an admissible canonical anchor, generator+overlay revision locks and independently qualified retrieval/replay capability. If a necessary source revision was discarded, return `RECONSTRUCTION_UNAVAILABLE` and do not reconstruct an invented equivalent using the newest generator.

## 8. Materialization pipeline and P09 boundary

### 8.1 Meaning of materialization

**Materialization is the act of turning a qualified region request and pinned data into a particular usable representation.** It may load authored data, compose a scene, instantiate assets, compute a procedural chunk, or produce a proxy. It does **not** decide what intervention happened and does **not** commit a new world consequence.

```text
D02 RegionDemand + canonical WorldAnchor
  -> strict scope, revision and permission validation
  -> work-set admission and resource reservation
  -> materialization port request (data-only)
  -> backend response + provenance receipt
  -> validate region, identity, overlay, model, schema, completeness
  -> reject if stale (no blind retry or rebinding)
  -> optionally publish resident *view cache* with generation check
  -> expose observer-scoped view to qualified game host
```

There is a **semantic firewall** between a generated *region representation* and authoritative *world-state mutations*. If a procedural output needs to affect resource quantities, entity properties, newly admissible passages or causal laws, P08-D01 (or a separately qualified authoring admission mode) must govern that change.

### 8.2 Revision-locked request/response

D02 uses the **P08-D01 materialization contract**, not an incompatible duplicate. The handoff binds `world_ref`, `world_model_ref`, `branch_ref_optional`, `generator_profile_ref`, `generator_revision`, `seed_or_input_ref`, `space_scope`, `world_time_ref`, `causal_overlay_revision`, `desired_detail`, `budget_ref`, and `expected_world_generation`.

P09 returns `region_ref`, `generated_artifact_ref`, `input_locks[]`, `semantic_validation_result`, `world_generation`, `uncertainty_or_failure`, and `provenance_receipt`. D02 adds a **local demand/request generation** for cancellation/replacement/ABA protection when associating the response with a specific outstanding request.

D02 validates the exact response contract, pinned generation and viewer permission; it cannot assert generated geometric quality, flora realism, noise statistics or optimal performance. Those are P09's algorithm-specific claims.

### 8.3 Three backend choices, without bias

| Profile | Selected by P09 when | D02 requirement |
|---|---|---|
| **A — Pure Gerbil** | A reference or target algorithm meets exact correctness/performance limits | Same typed request and receipt; no special code path privileges |
| **B — Gerbil + native acceleration** | Measured hot paths justify FFI/native/GPU with qualified ABI and ownership | Explicit batch/region boundary, resource leases, affinity and verified release |
| **C — Gerbil + external service** | Solver, distributed computation or scale justifies remote processing | Request identity, bounded outstanding jobs, timeouts, ambiguous acknowledgment and version locks |

**No performance inference:** A/B/C selection and comparative benchmarks belong to P09-D01. The D02 **reference test uses a fake materializer**, so neither Gerbil-native voxel throughput nor remote scaling is claimed qualified. A backend can support only some purposes/detail levels; D02 must refuse unsupported profile combinations instead of guessing fallback semantics.

### 8.4 Layer and generator revisions

Define a materialized cache key with exact **semantic** fields, not merely `(seed, x, y, z)`:

```text
MaterializationCacheKey {
  world_ref, branch_ref_optional, region_ref,
  world_model_revision, generator_profile_revision,
  canonicalizer_revision, input_or_seed_ref,
  causal_overlay_revision, time_scope_ref,
  requested_detail_profile, declared_view_scope,
  backend_equivalence_profile_optional
}
```

Hashing this key is an index aid, not proof of byte equivalence. A new generator revision can coexist with an older generator revision in adjacent regions if **world policy explicitly admits mixed-generation boundaries**, records seam/migration requirements and validates cross-region continuity. Otherwise reject a mixed-revision publish or stage a named migration; do not silently replace previously visited world content.

A view-only change in LOD must not alter region identity or semantic state; any intentionally detail-dependent simulation outcome is a different **qualified model** and cannot masquerade as mere visual degradation.

## 9. World-state publication, consistency and persistence

### 9.1 WorldStatePort fidelity to P08-D01

```text
WorldStatePort {
  read_snapshot(anchor, scope),
  validate_delta(base, delta),
  publish_if_generation(expected_anchor, proposed_state),
  observe_publication(receipt_ref),
  capability_manifest
}
```

D02 fulfills this exact public port in the Gerbil in-memory reference profile. It does not add a new authority to select among D01 consequences. A `proposed_state` must carry a D01 admission receipt (or the separately qualified initial-authoring fixture capability). It must be validated against the expected `WorldAnchor`, model/schema/region/entity generations, world time domain, and the structural invariants below.

### 9.2 One atomic in-memory publication

Initial profile: one writer, immutable values, no mixed-visible partial updates. The proposed algorithm is:

1. Read `WorldSnapshot` under `expected_anchor` and pin its structural/model/overlay revisions.
2. Validate the D01 candidate admission or authoring provenance; reject unrecognized mutation authority.
3. Construct a **new** candidate manifest, entity/region state indexes and structural relations off to the side. Never mutate the live snapshot in place.
4. Validate entity identity, region owner uniqueness, scene relation constraints, revision locks, no stale partial materialization and field ownership.
5. Compare the expected world-generation value and all required named read-set version locks against the single-writer authoritative state.
6. Publish one replacement snapshot reference as an atomic **in-process** state change; record a typed receipt with the exact new generation.
7. Notify observers after publication through a separate bounded view/notification port. A failed notification does not retroactively undo the successful in-memory world commit, nor can it be reported as delivered.

No external filesystem/database/network/GPU effect participates in this in-memory transaction. An optional persistent store must negotiate separate durability and atomicity capabilities. If a multi-region transaction cannot be atomic under its backend, it must refuse strong guarantees or explicitly report partial/unknown state and reconciliation obligations.

### 9.3 Status taxonomy

```text
PublicationStatus = PREPARED | COMMITTED_IN_MEMORY |
                    DURABLY_CONFIRMED_BY_ADAPTER |
                    REJECTED | CONFLICT | UNKNOWN
RegionDataStatus = AUTHORITATIVE_COMPLETE | AUTHORITATIVE_PARTIAL |
                   REPRESENTATION_ONLY | UNMATERIALIZED | UNVERIFIED
```

`DURABLY_CONFIRMED_BY_ADAPTER` is unavailable in pure reference mode. `AUTHORITATIVE_PARTIAL` requires an explicit valid partial-state semantics; it is **not** the unqualified assumption that missing fields are zero, null or nonexistent. Returning a successful CAS does not imply the renderer observed the new state, nor that the data is on disk.

### 9.4 Retention / saved worlds

The in-memory baseline can expose `snapshot_export` and `snapshot_import` as **pure canonical value operations**. It does not expose a claimed save-game file driver. A saved-world profile requires format/version migration, world+asset+generator+overlay closure, atomic commit/durability/backup evidence as supported by a selected backend, and a reader that does not execute serialized Scheme.

A retained future branch, historical event or alternative world must **not** be faked by one global world pointer. P06 owns its history model, while D02 may bind one or more selected `WorldAnchor` views through an explicit optional adapter. Retention of P06 history does not automatically retain all D02 region/artifact/materialization files, so dependency-closure checks are mandatory for reconstruction claims.

## 10. Observer access, visibility and derived world views

D02 exposes **viewer-scoped projections**, not raw world truth by default:

```text
WorldViewRequest {
  observer_ref, observation_scope_ref, world_anchor_ref,
  region_query, scene_query_optional, detail_profile_ref,
  max_entities, byte_budget, privacy_policy_ref,
  expected_view_revision_optional
}
WorldViewReceipt {
  world_anchor_ref, observer_ref, visible_region_refs[],
  visible_entity_refs[], filtered_scene_refs[],
  completeness: COMPLETE | PARTIAL | NOT_ESTABLISHED,
  unavailable_scopes[], redaction_report,
  state_staleness, view_revision, evidence_scope
}
```

- A client may know a place exists without seeing its contents, or know a character's public name without gaining hidden causal variables.
- A scene definition/asset may be public while specific world instances, discovered paths or hidden region contents remain scoped.
- `not returned` means nothing definite about existence unless the response explicitly certifies completeness for the observer's allowed query scope.
- Cache keys include observer/privacy scope and visibility policy revision; an editor/debugger projection must not leak through the ordinary player view.
- Visible LOD may represent unloaded content, but an impostor/proxy cannot certify interaction physics, line-of-sight, occupancy or model causality.
- Rendering timing and sizes may create side channels. Pure projection tests cover explicit field leaks, **not a blanket side-channel or security proof** of deployed game engines.

## 11. Gerbil Scheme package architecture

### 11.1 Independent Gerbil reference packages

```text
editions/game-world-scene/
  src/
    contract/{identity,manifest,scene,entity,region,layer}.ss
    spatial/{profiles,frames,partition,transforms}.ss
    semantic/{relation-checks,composition,field-ownership}.ss
    store/{snapshot,candidate,cas,receipt}.ss
    residency/{requests,working-set,leases,projection}.ss
    ports/{decision,materialize,history,host}.ss
    authoring/{schema-ir,diagnostics}.ss
  tests/
    fixtures/{tiny-town,scene-instances,grid-world,
              sparse-planet,hidden-town,stale-materializer}/
    identity/ spatial/ scene/ entity/ region/ publication/
    materialization/ observation/ failure/ integration/
  qualification/
    matrix.md
    executed-evidence/
```

The tree is an **implementation proposal**. No package or executable has been created by this document. Avoid requiring `gerbil` macros that have not been checked against an exact Gerbil/Gambit release, and never imply unchecked interface calls are an authority boundary.

### 11.2 Implementation mechanisms

- Use documented immutable/simple record types, maps, vectors and pure functions initially; copy-on-write or persistent data structures may be selected after profiling, not assumed to be efficient.
- Trusted authoring macros may lower into validated inert `SceneIR`/`WorldIR`. Untrusted input remains data, cannot call `eval`, choose arbitrary FFI, override trust roots, or inject generated operations.
- An optional ECS adapter can translate canonical world snapshots to an ECS-specific storage format. The adapter owns field/writeback declarations and independently qualifies coherence.
- In-memory mock is the first conformance target; P01 FFI/native lifetime, P09 generation and P05 durability are separate qualification profiles.
- Dependence on external hashes, binary formats and native codecs must carry algorithm/encoding/version locks. Do not assume object identity from a hash alone.

### 11.3 Illustrative authoring DSL

```scheme
(define-game-world tiny-frontier
  (space (grid-2d (cell-size 16) (integer-coordinates)))
  (partition (exclusive-region-owner))
  (scene town (definition town-v1))
  (scene frontier (definition frontier-v1))
  (entities (merchant (generation 1)) (bridge (generation 1)))
  (residency (max-resident-regions 8) (request-budget 16))
  (materialization (profile fake-region-generator))
  (publication (single-writer-in-memory)))
```

The example illustrates *semantic data*: it is **not** a claim that `define-game-world` is built or exported by Gerbil. A later trusted macro can be considered after the corresponding inert schema and reference oracle exist. Scope validation and live-world mutation admission do not follow from successful macro expansion.

## 12. Worked fixtures and adversarial examples

### F1 — Tiny frontier, zero renderer

Two disjoint cells `A` and `B`, one merchant entity, one bridge scene definition instantiated twice, one rule-governed world snapshot. A P08-D01 admitted decision moves the merchant from A to B. D02 checks the entity's generation, removes authoritative membership from A, inserts it into B, and publishes one new immutable world snapshot. Both the old and new structural references remain inspectable, but only the new world generation is authoritative. **No renderer, P09, P06 or P03 runtime is required.**

**Falsification:** unloading A must not delete the merchant in B; reloading A must not resurrect its old canonical owner; repeating the stale move must fail with `STALE_ANCHOR`.

### F2 — One asset, multiple scene instances

A bridge definition is placed in two towns with different instance transforms and render skins. One bridge is structurally broken in canonical world state after a D01 decision; the other remains intact. Reusing one shared asset must not copy the causal breakage to the other bridge. A scene visual replacement must not restore the broken bridge in the authoritative world.

**Falsification:** `asset equality` must not imply `entity identity`; shared renderable components must not become shared mutable health state.

### F3 — Sparse world with hidden region

A world has 10,000 theoretical region addresses, but only 12 have authoritative state and 4 resident representations. A private region contains undiscovered resources. The observer sees an allowed low-detail map proxy, but no resource count. The host makes an authorized `RegionDemand`, validates an explicit scope and an input-locked materializer response, and may activate one new view. The generator's existence does not upgrade an observer's privileges.

**Falsification:** requesting a visual proxy cannot leak private region materialization results, hidden entity IDs or privileged audit metadata.

### F4 — Stale asynchronous generator response

A request for region R under world generation 41, generator revision g1, overlay o5 is dispatched. Before it returns, a D01 consequence publishes generation 42 with overlay o6. The stale response arrives. D02 refuses its publication as a current region view. A new request must pin 42/o6 (or explicitly qualify an immutable representation reuse across overlays). Backend success alone is not sufficient.

**Falsification:** late completion cannot overwrite current world state; a canceled request ID reused with a new generation must not accept the old result.

### F5 — Fictional history without duplicate P06

Two distinct historical heads refer to apparently identical visual towns. D02 may expose a typed world-view anchor for each via an optional P06 adapter, but it must not infer they are the same historical world. The P06 backend remains authoritative for lineage, and D02 never writes history edges or performs synthesis. Without P06, a forward-only world still loads its own local snapshot.

**Falsification:** equality of rendered meshes/asset lists cannot erase branch identity; absence of P06 cannot block normal D02 world-store qualification.

### F6 — Nonlocal causal influence and unloaded data

A distant settlement is unloaded, but D01's causal rule graph includes a trade route from the player's city to that settlement. D02 can report the distant region as `NOT_RESIDENT` with a valid authoritative region anchor. It must not claim the distant region is causally unaffected because it is outside the camera range. D01 may return `INFLUENCE_CLOSURE_UNVERIFIED`, and D02 must preserve the pending semantic dependency instead of pruning it with visual working-set logic.

### F7 — Planetary address precision

A locally charted planetary tile at large conceptual coordinates receives a close camera/view transform. A rounded floating-point render position is valid for presentation but is not canonical entity identity, frame identity or a gameplay exact-distance oracle. Moving between two chart revisions requires explicit conversion and error reporting. A different backend producing the same pixel position does not establish equal world coordinates.

### F8 — Scene composition cycle

Scene definition D refers to asset E, which refers to D via a nested definition. Under strict structural recursion bounds, D02 refuses an unbounded instantiation cycle before materialization. A legal authored scene DAG may have reusable references without creating two canonical world entities by accident. The scene graph's admissible cycles are defined independently from causal dependency cycles and P06 ancestry.

## 13. Core invariants and typed failures

### 13.1 Invariants (I01–I24)

1. **I01 — Disjoint identity kinds.** World, region, entity, scene, scene instance, asset, layer and history refs remain distinct even with identical labels/payloads.
2. **I02 — Generation lock.** A stale world, entity, region, scene-instance or materialization-request generation cannot mutate a newer instance.
3. **I03 — World-state primacy.** Canonical entity/region state is not inferred from active scene nodes, loaded visuals or render proxies.
4. **I04 — Decision ownership.** D01 decides interventions and consequences; D02 publishes structurally valid admitted state but never declares causal legality by itself.
5. **I05 — Historical non-ownership.** D02 stores optional P06 references but cannot publish its own P06 history graph edges or synthesis.
6. **I06 — Generator non-ownership.** Materializer output describes representation; semantic changes require separately admitted world mutation.
7. **I07 — Scope-aware topology.** Spatial comparisons, adjacency, transforms and region ownership require an explicit qualified spatial profile.
8. **I08 — No implicit metric.** Scene hierarchy, graph adjacency and coordinate proximity are different relations.
9. **I09 — Relation typing.** Scene parenting, serialization ownership, component membership, history, causality and resource lease are not the same edge.
10. **I10 — Scene-instance isolation.** Distinct scene instances do not accidentally share canonical mutable state through one asset definition.
11. **I11 — Layer separation.** Presentation/authoring override order cannot silently override canonical causal world state.
12. **I12 — Structural closure.** All committed refs and selected scene/region dependencies are validated at the exact lock revisions; missing required refs fail explicitly.
13. **I13 — Existence/residency distinction.** Unloaded, hidden or unrendered does not mean nonexistent or causally irrelevant.
14. **I14 — Region ownership.** A selected exclusive partition permits one authoritative owner for each canonical entity-state field at a world generation.
15. **I15 — Boundary migration.** Cross-region movement cannot expose a half-moved authoritative entity in the single-writer reference profile.
16. **I16 — View safety.** Observer projection enforces named scope and reports partial/unknown without asserting unauthorized existence facts.
17. **I17 — Materialization exactness.** A materializer response binds world/region/model/generator/overlay/time/detail and request generation, with an exact validation receipt.
18. **I18 — No stale reactivation.** Late backend completion or failed cancellation cannot rebind old content to new world state.
19. **I19 — Bounded operations.** Region requests, view queries, scene traversal, instantiation, resident bytes and traces honor quotas and typed exhaustion.
20. **I20 — No false durability.** In-memory publication, runtime residency and durable saved-world confirmation are different claims.
21. **I21 — Atomic reference publication.** An accepted single-writer world successor becomes visible as one immutable snapshot under expected-generation checking.
22. **I22 — Safe eviction.** Evicting a resident representation cannot discard uncommitted authoritative deltas, necessary retention pins or unverified native leases.
23. **I23 — Replay and revision honesty.** Region reconstruction needs the exact admitted versions/input closure and reports incompatibility or unknown instead of silently regenerating under new rules.
24. **I24 — Source and runtime independence.** Pure D02 builds/tests with mock ports without P06, P09, Bevy, Godot, Unreal, OpenUSD, a renderer, native driver, network service or database.

### 13.2 Failure taxonomy

| Failure / uncertainty | Normative handling |
|---|---|
| `INVALID_WORLD_REF`, `SCHEMA_INCOMPATIBLE`, `STALE_WORLD_GENERATION` | Reject before any committed mutation |
| `STALE_ENTITY_GENERATION`, `STALE_REGION_GENERATION`, `STALE_INSTANCE_GENERATION` | Do not rebind to reused object slot or renamed label |
| `PARTITION_AMBIGUOUS`, `REGION_OWNER_CONFLICT`, `FRAME_UNSUPPORTED`, `COORDINATE_PRECISION_LOSS` | Refuse strong spatial claim or return typed incomplete result |
| `SCENE_CYCLE`, `SCENE_OWNER_INVALID`, `MISSING_ASSET`, `LAYER_PRECEDENCE_CONFLICT` | Refuse scene resolution/instantiation; no silent default |
| `MATERIALIZATION_STALE`, `GENERATOR_REVISION_MISMATCH`, `INPUT_LOCK_MISSING` | Discard or quarantine stale result; preserve world anchor |
| `REQUEST_CANCEL_UNCONFIRMED`, `LEASE_RELEASE_UNKNOWN`, `WORKSET_BUDGET_EXCEEDED` | Preserve conservative resource charges/pins; no invented cleanup |
| `AUTHORITATIVE_STATE_PARTIAL`, `RECONSTRUCTION_UNAVAILABLE`, `RETENTION_CLOSURE_MISSING` | Report uncertainty; never claim region empty or world recovered |
| `OBSERVER_SCOPE_VIOLATION`, `HIDDEN_ENTITY_LEAK_BLOCKED`, `VIEW_STALE` | Fail closed for hidden fields, preserve redacted receipt |
| `UNQUALIFIED_WORLD_MUTATION`, `STALE_ANCHOR`, `CROSS_REGION_ATOMICITY_UNAVAILABLE` | Reject publication or explicitly report supported weaker profile |
| `DURABILITY_NOT_ESTABLISHED`, `UNKNOWN_AFTER_DISPATCH`, `BACKEND_CAPABILITY_UNSUPPORTED` | Keep in-memory/observed/durable statuses separate; reconcile |

## 14. Qualification: 72 proposed tests, 12 gates

**Important:** Every test below is a **future executable requirement**, not a passed test. Runtime claims require pinning the actual Gerbil/Gambit toolchain, validators, source revisions, fixture data and selected adapter/backend versions. A design-level oracle is not an executable verification result.

### 14.1 Qualification gates (Q0–Q11)

| Gate | Required evidence | Claim ceiling |
|---|---|---|
| **Q0 — Toolchain and source locks** | Exact Gerbil/Gambit compiler/build/test probes, dependency closure, pinned D01/K0 refs | Tested toolchain/profile only |
| **Q1 — Ontology and identity** | Distinct world/entity/instance/region kinds; ABA, version, schema and namespace tests | Typed reference semantics only |
| **Q2 — Spatial contracts** | Topology, frame/transform, partition ownership, precision and migration oracles | Named spatial profiles only |
| **Q3 — Scene composition** | Tree/DAG checks, owner-vs-parent, instances, layers, overrides and asset pinning | Selected scene schema only |
| **Q4 — Entity and world state** | Existence/residency, moved entities, field ownership and canonical state invariants | Pure/reference state model |
| **Q5 — Region working sets** | Loads, cancellations, stale results, bounded demand, eviction and release uncertainty | Fake driver or exact driver tested |
| **Q6 — P09 materialization contract** | Typed A/B/C-neutral fake adapter, revision/key exactness, negative compatibility cases | Port only, no generation quality/speed |
| **Q7 — D01 publication parity** | Same read/validate/CAS port as D01; admitted vs invalid mutation; stale world races | One-writer in-memory atomicity |
| **Q8 — Observation and layer isolation** | Actor/view masking, completeness, LOD truth separation, no privileged view leaks | Explicit data flow; not timing-side-channel proof |
| **Q9 — Reconstruction/retention** | Pin preservation, stale generator, canonical export/import, durability refusal | In-memory reconstruction if qualified |
| **Q10 — Optional integrations** | Opt-in P06/scene engine/materializer adapters and independent capabilities; no core import leakage | Only individually tested adapters |
| **Q11 — Independent audit** | Property/reference oracles, mutation testing, fault injection, performance budgets, negative controls and evidence report | Selected fixed fixture/revision |

### 14.2 Test register — identity and ontology (T01–T08)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T01** | World and entity share the same human label | Typed refs are never equal |
| **P08-D02-T02** | Two worlds have byte-identical terrain/scene content | Independent world and model identities preserved |
| **P08-D02-T03** | Entity slot reused after tombstone | Old generational ref rejected |
| **P08-D02-T04** | Scene definition instantiated twice | Independent instance identity and mutable state |
| **P08-D02-T05** | Asset digest matches a canonical entity-state digest | No cross-kind identity inference |
| **P08-D02-T06** | Unknown required schema field/version | Typed refusal; no implicit default |
| **P08-D02-T07** | World anchor references missing region generation | Refusal before publication |
| **P08-D02-T08** | History ref optional and P06 absent | Forward-only world runs normally |

### 14.3 Test register — space and partitions (T09–T16)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T09** | Graph-only world without Cartesian coordinates | Valid region query and adjacency |
| **P08-D02-T10** | 2D integer grid edges and signed coordinates | Deterministic cell ownership |
| **P08-D02-T11** | Two exclusive partitions claim one field owner | Reject `REGION_OWNER_CONFLICT` |
| **P08-D02-T12** | Same coordinate tuple in two frames | No implicit spatial equivalence |
| **P08-D02-T13** | Unsupported/inexact frame transformation | Typed failure or bounded precision statement |
| **P08-D02-T14** | NaN/Infinity/out-of-range spatial input | Reject before index or state mutation |
| **P08-D02-T15** | Region across partition revision boundary | Explicit migration/refusal, not alias |
| **P08-D02-T16** | Extreme-distance local-origin view | Canonical region identity unchanged by render rounding |

### 14.4 Test register — scene/layer composition (T17–T24)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T17** | Scene graph cycle under strict-tree profile | `SCENE_CYCLE` before instantiation |
| **P08-D02-T18** | Scene child is not owned for serialization | Not silently included in packed definition |
| **P08-D02-T19** | Shared asset, two differently overridden instances | Independent resolved views and field provenance |
| **P08-D02-T20** | Two conflicting layer opinions without precedence policy | Explicit `LAYER_PRECEDENCE_CONFLICT` |
| **P08-D02-T21** | Visual override attempts to change canonical resource count | Block world-state mutation |
| **P08-D02-T22** | Referenced asset deleted between validation and instantiation | Stale/missing asset refusal |
| **P08-D02-T23** | Live instance bound to definition revision n; n+1 published | No silent instance migration |
| **P08-D02-T24** | Scene composition traversal under recursion/size quota | Typed bounded termination |

### 14.5 Test register — entity lifecycle (T25–T32)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T25** | Unload resident region containing extant merchant | Merchant remains canonical `PRESENT` |
| **P08-D02-T26** | Entity tombstoned; visual proxy still cached | Proxy not treated as live entity |
| **P08-D02-T27** | Same entity displayed in two scenes | One authoritative state, two view instances |
| **P08-D02-T28** | ECS adapter tries direct canonical health write | Refused without admitted mutation protocol |
| **P08-D02-T29** | Move A-to-B in one world CAS | No half-owner state observed |
| **P08-D02-T30** | Nonatomic backend requested for atomic move | Fail closed or explicit supported weaker semantics |
| **P08-D02-T31** | Replayed old movement on new world generation | `STALE_ANCHOR`; no duplicate move |
| **P08-D02-T32** | Structural authoring mode used as live gameplay bypass | Admission/authority separation enforced |

### 14.6 Test register — region working set (T33–T40)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T33** | Request one valid region from metadata-only state | Exact request generation and typed stages |
| **P08-D02-T34** | Two observers have incompatible visibility scope | No unsafe shared unfiltered resident view |
| **P08-D02-T35** | Region demand exceeds resident-byte cap | Backpressure with honest usage receipt |
| **P08-D02-T36** | Cancel during backend fetch | `CANCEL_REQUESTED` not false `CANCELLED_RELEASED` |
| **P08-D02-T37** | Native lease release confirmation missing | Conservative pinned/unknown resource state |
| **P08-D02-T38** | Eviction with uncommitted authoritative delta | Eviction refused, canonical delta preserved |
| **P08-D02-T39** | Prefetched region fails to materialize | Existing world truth unchanged |
| **P08-D02-T40** | High-priority stream requests starve others | Declared fairness policy observable or starvation explicitly reported |

### 14.7 Test register — materialization and revision locks (T41–T48)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T41** | Fake materializer with complete input/revision closure | Accepted resident representation |
| **P08-D02-T42** | Response from stale world generation | Rejected before activation |
| **P08-D02-T43** | Same request ID recycled, old response arrives | Request-generation ABA rejection |
| **P08-D02-T44** | Generator revision g2 offered for g1-locked region | Mismatch refusal or qualified migration |
| **P08-D02-T45** | Same seed but changed causal overlay | Cache cannot misrepresent current world |
| **P08-D02-T46** | LOD proxy contains different resource number | Proxy has no authority to change gameplay truth |
| **P08-D02-T47** | A/B/C materializer mocks implement same wire contract | Version/receipt conformance without performance claim |
| **P08-D02-T48** | Generator output changes a causal passage without D01 | Rejected as unadmitted world mutation |

### 14.8 Test register — publication and persistence (T49–T56)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T49** | Valid D01 admitted candidate and matching expected anchor | One atomic in-memory successor and receipt |
| **P08-D02-T50** | Structurally well-formed but D01-unadmitted candidate | Publication refused |
| **P08-D02-T51** | Two writes start from one expected world revision | Exactly one CAS wins; other conflicts |
| **P08-D02-T52** | Multiple region edits inside one snapshot CAS | No exposed intermediate state |
| **P08-D02-T53** | Stale actor or model revision at publication | Refused, no stale side effect |
| **P08-D02-T54** | In-memory snapshot exported then canonical reimported | Exact schema/revision/scoped state parity |
| **P08-D02-T55** | In-memory commit requested to be crash durable | `DURABILITY_NOT_ESTABLISHED` |
| **P08-D02-T56** | Observer notification fails after in-memory CAS | Commit remains true; notification marked failed |

### 14.9 Test register — observer, retention, and epistemics (T57–T64)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T57** | Player queries hidden entity in private region | No secret fields, no false completeness |
| **P08-D02-T58** | Debug observer and ordinary player share cache namespace | Privileged data cannot leak to player |
| **P08-D02-T59** | View stale after a new world revision | Staleness flagged, not labeled current |
| **P08-D02-T60** | Region unmaterialized but canonical state exists | Not reported nonexistent |
| **P08-D02-T61** | Unloaded region has nonlocal D01 causal edge | Not classified causally unaffected |
| **P08-D02-T62** | Required generator revision pruned after eviction | `RECONSTRUCTION_UNAVAILABLE` |
| **P08-D02-T63** | P06 history retained but D02 region anchor unavailable | Honest recovery failure, no invented world |
| **P08-D02-T64** | Display proxy differs from high-detail rendering | Stable gameplay/world identity and semantic state |

### 14.10 Test register — integrations and independent falsification (T65–T72)

| ID | Fixture / challenge | Required outcome |
|---|---|---|
| **P08-D02-T65** | Pure Gerbil test with no P06/P09/graphics imports | Core compiles and reference behavior passes |
| **P08-D02-T66** | P06 optional adapter with distinct but visually equal branches | Branch identity preserved, no copied history DAG |
| **P08-D02-T67** | P09 port lacks required detail/capability | Explicit unsupported profile |
| **P08-D02-T68** | Scene engine adapter reports success but canonical anchor stale | Reject current-view publication |
| **P08-D02-T69** | Mutate region owner checker to accept duplicates | Independent oracle catches mutation |
| **P08-D02-T70** | Inject fault between candidate construction and CAS | No partial live world snapshot |
| **P08-D02-T71** | Run tiny town and independent sparse scientific scene fixture | Shared structural contracts with domain-specific semantics intact |
| **P08-D02-T72** | Independent examiner reviews exact toolchain, fixtures, failures and report | Scoped qualification only; no unearned engine/performance claim |

### 14.11 Qualification matrix and evidence limits

- `P08_D02_DESIGN_REVIEWED`: written contracts and counterexamples inspected; no executable claim.
- `P08_D02_PURE_REFERENCE_QUALIFIED`: Q0–Q4 plus relevant Q7–Q9 and Q11, with exact Gerbil version/oracles; does not qualify streaming/native backends.
- `P08_D02_MOCK_STREAMING_QUALIFIED`: Q0–Q9 and Q11 on a specified fake materializer and resource model; no real disk/network/GPU performance guarantee.
- `P08_D02_ADAPTER_QUALIFIED`: Q0–Q11 and selected real-backend fault tests for exact adapter/capability/OS/runtime profile; not all engine families.
- `P08_D02_INTEGRATED_GAME_QUALIFIED`: requires independent end-to-end gameplay, scene, streaming, memory/recovery/performance tests against a named game host. Not achieved by D02's 72 specification cases alone.

A failed gate remains `FAILED`, `DEFERRED`, `SPEC_BLOCKED`, `UNSUPPORTED_BACKEND` or `UNQUALIFIED`, with evidence. Never convert a design proposal, static source inspection or a favorable mock into a production qualification claim.

## 15. Proposed implementation sequence

1. **D02-00 — Source and API lock:** freeze D01's `WorldAnchor`, `SpaceScope`, `WorldStatePort` and materialization envelope; pin Gerbil/Gambit, K0 and selected K1 profile versions.
2. **D02-01 — Pure typed world:** implement world, region, entity and scene/instance refs; structural relation checker and one small immutable in-memory snapshot.
3. **D02-02 — Spatial profiles:** bounded grid and graph-oracle profiles; disjoint partition and a tiny coordinate-frame registry. No octree or planetary renderer.
4. **D02-03 — Scene composition:** asset definition, instance, owner/parent separation, typed layers and independent resolved views with negative cycle tests.
5. **D02-04 — World publication:** `read_snapshot`, `validate_delta`, `publish_if_generation`, `observe_publication` with a D01 mock admission and fault-injected CAS.
6. **D02-05 — Region working set:** fake demand/response driver, quotas, cancellation, eviction, stale result and observer-scoped views; prove no canonical loss.
7. **D02-06 — P09 port conformance:** fake A/B/C adapter envelopes and exact revisions; **do not implement generators or select language by intuition**.
8. **D02-07 — Independent qualification:** execute T01–T72, cross-check D01 port parity and dependency closure, publish only actual evidence if a future code implementation is authorized.
9. **Separate future authorization:** integrate one game/renderer/ECS/native adapter or persistent store only if a real product requirement and qualification budget justify it. P09-D01 retains procedural algorithms; no DLC/time-travel mechanics are designed here.

The staging prioritizes one usable, finite, testable world model. It does not require implementing every world-space topology, all K1 mechanisms or every engine convention before the first reference result.

## 16. Open decisions and final disposition

| Issue | Initial decision | What must be proved or specified later |
|---|---|---|
| Core representation | Immutable, single-writer Gerbil maps and typed records | Memory/performance and precise Gerbil APIs under pinned toolchain |
| ECS vs scene graph | Neither mandated; typed structural relations first | Adapter invariants and actual host needs |
| Scene hierarchy | Strict-tree reference mode, versioned DAG profile optional | Exact cycle/ownership/instancing checks |
| Region partition | Exclusive deterministic grid and graph fixtures | Overlap/multi-resolution/planetary policy independently qualified |
| World coordinates | Frame-typed, units/precision declared | Transform error bounds and extreme-scale tests |
| Region lifecycle | Semantic existence independent of residency | Exact backend lease/cancellation semantics |
| Streaming | Bounded fake reference working set | Real host scheduler, performance and race tests later |
| P09 generation | Strict request/response port | A/B/C benchmark decisions owned by P09 |
| Authoritative state | In-memory snapshot + expected-version publication | Backend-specific durability and distributed semantics later |
| P06 history | Optional public adapter, no DAG import | History/view reconstruction and retention closure if selected |
| D01 world decisions | Required admission for live gameplay mutations | End-to-end fixture validates causal vs structural authority |
| Observer visibility | Scoped projections with explicit unknowns | Side channels and deployed authorization outside pure reference |
| External assets | Reference/definition/instance semantics | Actual format/codecs, licensing and asset pipeline outside D02 |
| Other application domains | May reuse D02 structural contract as research input | Separate specialization designs select adapters and qualification |

**Final design disposition:**

```text
P08_D02_GAME_WORLD_SCENE_DESIGN_PROPOSED /
GAMING_TRACK / D01_WORLDSTATEPORT_EXACT_PARITY /
WORLD_ENTITY_REGION_SCENE_ASSET_INSTANCE_IDENTITIES_DISTINCT /
SPATIAL_FRAMES_TYPED / NO_MANDATORY_ECS /
SCENE_PARENTS_NOT_SERIALIZATION_OWNERS /
AUTHORITATIVE_WORLD_STATE_NOT_RENDER_RESIDENCY /
CONSERVATIVE_NONLOCAL_CAUSAL_DEPENDENCIES /
REGION_STREAMING_BOUNDED / EVICTION_NOT_WORLD_DELETION /
P09_GENERATION_PORT_ONLY / A_B_C_BACKEND_NEUTRAL /
P06_HISTORY_OPTIONAL / IN_MEMORY_CAS_FIRST /
72_FUTURE_TESTS / Q0_TO_Q11 /
NO_IMPL / NO_RUNTIME_PROOF / DOCUMENT_PUBLISHED
```

> **Closing principle:** A game world must remain identifiable, structurally consistent, and causally meaningful even when no camera points at it, no scene is active, and no procedural region has been rendered. D01 determines *what the world can become*; D02 keeps *what the world currently is* accessible and coherent; P09 supplies representations when they are needed. The histories of how it became so remain P06's separate concern.