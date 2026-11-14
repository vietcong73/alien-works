# Lisp Foundation Reserve — Specialist Edition Reindex

**Date:** 14 November 2026  
**Repository:** `vietcong73/alien-works`  
**Branch:** `audit/lisp-foundation-reserve`  
**Status:** Owner-approved design taxonomy migration; no implementation qualification.

## Scope and design boundaries

**Sixteen detailed designs** are organized in **seven specialist editions**. A shared edition denotes a family of Lisp/Scheme capabilities; each detailed design retains its own runtime independence, technical contracts, authority owner, test register and qualification obligations.

The **K0 minimal contract kernel** and **K1 optional mechanism portfolio** remain separate foundations. Their links and internal references are aligned to the new identifiers; their role and qualification policy are preserved.

The source review and concept files under `docs/reviews/` and `docs/concept/` are outside this taxonomy and remain untouched. Original filenames and original design IDs can be resolved at the [pre-reindex commit](https://github.com/vietcong73/alien-works/tree/b0f14617af7f9ede2771f4469b974216ab87c3a9/docs/design).

## Seven specialist editions

| New P | Lisp/Scheme capability group | Designs |
|---|---|---|
| **P01** | Lisp Metaprogramming & Interoperability | P01-D01, P01-D02 |
| **P02** | Declarative Scheme Systems & Governed Execution | P02-D01, P02-D02 |
| **P03** | Symbolic State Machines & Causal Histories | P03-D01, P03-D02 |
| **P04** | Retrieval, Persistent Memory & Evidence | P04-D01 |
| **P05** | Symbolic Computation & Interactive Worlds | P05-D00 (reserved), P05-D01, P05-D02, P05-D03 |
| **P06** | Adaptive Intelligence & Autonomous Systems | P06-D01, P06-D02, P06-D03, P06-D04 |
| **P07** | Scientific Simulation & Experimental Systems | P07-D01, P07-D02 |

**P05-D00 — Formal & Symbolic Computing** is reserved for the final specialist design. Its intended cross-domain scope includes formal semantics, symbolic reasoning, constraint solving, model checking, proof artifacts and program synthesis. Its placement in P05 does not make interactive worlds or game engines a dependency. This **D00 does not yet exist as a reviewed design**.

## Exact old-to-new mapping

| Original ID | New ID | Design | Published Markdown |
|---|---|---|---|
| `P01-D01` | **`P01-D01`** | Native Integration & Metaprogramming | [`LFR-P01-D01-native-metaprogramming-detailed-design-2026-11-12.md`](./LFR-P01-D01-native-metaprogramming-detailed-design-2026-11-12.md) |
| `P02-D01` | **`P02-D01`** | Declarative Systems & Process Supervision | [`LFR-P02-D01-declarative-systems-process-supervision-detailed-design-2026-11-12.md`](./LFR-P02-D01-declarative-systems-process-supervision-detailed-design-2026-11-12.md) |
| `P03-D01` | **`P03-D01`** | Semantic Compilation & Temporal State DSL | [`LFR-P03-D01-semantic-state-dsl-detailed-design-2026-11-12.md`](./LFR-P03-D01-semantic-state-dsl-detailed-design-2026-11-12.md) |
| `P04-D01` | **`P02-D02`** | Agent Authority & Controlled Execution | [`LFR-P02-D02-agent-authority-controlled-execution-detailed-design-2026-11-12.md`](./LFR-P02-D02-agent-authority-controlled-execution-detailed-design-2026-11-12.md) |
| `P05-D01` | **`P04-D01`** | Retrieval, Persistence & Evidence | [`LFR-P04-D01-retrieval-persistence-evidence-detailed-design-2026-11-12.md`](./LFR-P04-D01-retrieval-persistence-evidence-detailed-design-2026-11-12.md) |
| `P06-D01` | **`P03-D02`** | Reversible State, Causal Histories & Bounded Search | [`LFR-P03-D02-branching-causal-reversibility-design-2026-11-13.md`](./LFR-P03-D02-branching-causal-reversibility-design-2026-11-13.md) |
| `P07-D01` | **`P01-D02`** | Typed UI & Interoperability | [`LFR-P01-D02-ui-interoperability-detailed-design-2026-11-13.md`](./LFR-P01-D02-ui-interoperability-detailed-design-2026-11-13.md) |
| `P08-D01` | **`P05-D01`** | Spacetime Decision & Causal Consequence | [`LFR-P05-D01-spacetime-decision-causal-consequence-design-2026-11-13.md`](./LFR-P05-D01-spacetime-decision-causal-consequence-design-2026-11-13.md) |
| `P08-D02` | **`P05-D02`** | Game World & Scene Architecture | [`LFR-P05-D02-game-world-scene-architecture-2026-11-13.md`](./LFR-P05-D02-game-world-scene-architecture-2026-11-13.md) |
| `P08-D03` | **`P05-D03`** | Game World Procedural Generation | [`LFR-P05-D03-game-world-procedural-generation-architecture-2026-11-13.md`](./LFR-P05-D03-game-world-procedural-generation-architecture-2026-11-13.md) |
| `P09-D01` | **`P06-D01`** | Reinforcement Learning & Adaptive Intelligence | [`LFR-P06-D01-reinforcement-learning-adaptive-intelligence-architecture-2026-11-13.md`](./LFR-P06-D01-reinforcement-learning-adaptive-intelligence-architecture-2026-11-13.md) |
| `P09-D02` | **`P06-D02`** | Multi-Agent & Behavioral Systems | [`LFR-P06-D02-multi-agent-behavioral-systems-architecture-2026-11-13.md`](./LFR-P06-D02-multi-agent-behavioral-systems-architecture-2026-11-13.md) |
| `P09-D03` | **`P06-D03`** | Control & Autonomous Systems | [`LFR-P06-D03-control-autonomous-systems-architecture-2026-11-13.md`](./LFR-P06-D03-control-autonomous-systems-architecture-2026-11-13.md) |
| `P09-D04` | **`P06-D04`** | Distributed Systems & Coordination | [`LFR-P06-D04-distributed-systems-coordination-architecture-2026-11-14.md`](./LFR-P06-D04-distributed-systems-coordination-architecture-2026-11-14.md) |
| `P10-D01` | **`P07-D01`** | Scientific & Physical Simulation | [`LFR-P07-D01-scientific-physical-simulation-architecture-2026-11-14.md`](./LFR-P07-D01-scientific-physical-simulation-architecture-2026-11-14.md) |
| `P10-D02` | **`P07-D02`** | Experimental & Research Infrastructure | [`LFR-P07-D02-experimental-research-infrastructure-architecture-2026-11-14.md`](./LFR-P07-D02-experimental-research-infrastructure-architecture-2026-11-14.md) |

## Publication and technical provenance

- Updated first-level titles take the form **`LFR-Pxx-Dyy | Lisp/Scheme group / Design`**. Edition and design-slice metadata use the canonical naming.
- The canonical IDs were applied to file paths, inter-design references, test prefixes and proposed Gerbil namespace examples. **Test numbers, conditions, invariants, qualification gates and technical models remain the same.**
- Previous P08-D01/D02's provisional P09 procedural generator reference now points to **P05-D03**. The adaptive intelligence edition now occupies **P06**.
- P01-D01's original hypothetical native `P01-D02` is now recorded as a **P01-D01 follow-up**; the new **P01-D02** uniquely identifies Typed UI & Interoperability.
- Original design files are preserved by Git history at [`b0f14617af7f`](https://github.com/vietcong73/alien-works/tree/b0f14617af7f9ede2771f4469b974216ab87c3a9/docs/design). Legacy references in unchanged research/concept records can be resolved with this table.
- This reindex makes **no change to ownership policies, runtime dependencies, authorization, implementation state or qualification claims**.

## Next design

**P05-D00 — Formal & Symbolic Computing** is next for independent research and design review, after the specialist corpus has been renumbered.

`16_SPECIALIST_DESIGNS / 7_EDITIONS / K0_K1_FOUNDATIONS / P05_D00_RESERVED / DOCUMENTATION_ONLY`
