# LFR-P03-D02 | Symbolic State Machines & Causal Histories / Reversible State, Causal Histories & Bounded Search

**Document class:** Detailed design proposal for the P03-D02 specialist edition  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve, LFR-C02  
**Edition:** P03 — Symbolic State Machines & Causal Histories  
**Design slice:** D02 — Reversible State, Causal Histories & Bounded Search  
**Implementation target:** Independent Gerbil Scheme; no Guix, Guile, Shepherd, Git, Python, or external database runtime dependency in the in-memory core  
**Design scope:** Reversible state and bounded search; immutable causal histories; multi-source synthesis; revision-pinned reconstruction; rule and meta-policy governance.  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; concept `docs/concept/LFR-C02-source-lineage-gerbil-reserve-concept-2026-11-12.md`; P01-D01–P04-D01 remain unchanged.  
**Disposition:** `RESEARCH_GROUNDED / ARCHITECTURAL_DESIGN_PROPOSED / UNIMPLEMENTED / UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  

> **Core decision:** History is immutable; futures are plural; admissible convergence is constructive and non-destructive. A user may revisit retained ancestors, create branches under changed state or rules, or conditionally synthesize **two or more** histories into **one or more new candidate presents**. Synthesis never erases source heads and never occurs merely because the data structures can be combined. Rule specifications, reconciliation policies, and the rules governing those policies are first-class, versioned, inspectable data, with a bounded trusted kernel controlling admission.

This design places **N-ary causal synthesis with meta-governance** at the center of P03-D02-A. It preserves the P03-D02-B bounded-search architecture and three distinct recovery mechanisms: scoped delta undo, immutable generation selection, and revision-pinned reconstruction. It specifies causal-topology behavior and qualification requirements without claiming that an implementation or any of the proposed tests has passed. Domain admission, search correctness, chess stress fixtures, delta unwind, budget semantics, and provenance remain independently testable obligations of the P03-D02 specialist edition.

---

## 1. Research findings: what is actually transferred

| Source | Source-grounded observation | Transfer to P03-D02 | Nontransfer / failure to avoid |
|---|---|---|---|
| Gerbil chess AI, reviewed `mark-watson/gerbil_scheme_book@1335cb2708926213ba62afd2998754358f655db1`, `engine.ss` and `ai.ss` | Mutable board and scoped make/unmake are efficient for exploring and unwinding candidate moves | Checked `DeltaToken`, local undo frames, bounded traversal | LIFO undo restores an execution frame; it is **not** a persistent independent history or proof that every arbitrary operation is invertible |
| GNU Guix `guix time-machine`, reviewed `guix-mirror/guix@18f545c14eb36c7c588d2cf356e387e17e53cd51` | An older pinned channel revision can supply historical package/tool definitions | Historical `RevisionLock` and reconstructed evaluation environment | Running an old definition does **not** restore the application state or reverse service, database, filesystem, or network effects |
| GNU Guix profile/system generations | Immutable store targets and selectable generation pointers provide practical retained versions | Immutable generation images plus independently validated active pointer changes | Ordinary Guix **profile generation history is linear**; installing after rollback overwrites future generations. P03-D02 must not inherit this behavior |
| Git commit DAG and branches | Commits identify snapshots and parents; branches are movable references to heads; merge tracks multiple parents | Immutable causal commits and branch-head map, compact sharing between histories | Git merge's three-way textual/file semantics do not prove domain-state semantic compatibility; do not import Git as a runtime requirement |
| Gerbil Scheme hygienic macros, phases and custom preludes | Compile-time syntax transformations and language definitions can be represented explicitly | Generate typed state/rule/reconciliation policy IR with expander/compiler provenance, then validate before execution | Code-as-data and macros do not inherently enforce type safety, determinism, authorization, termination, or semantic correctness |

**Verification sources:**

1. GNU Guix manual, `guix time-machine`: https://guix-home.trop.in/Invoking-guix-time_002dmachine.html
2. GNU Guix manual, profile rollback and linear future-history overwrite: https://guix-home.trop.in/Invoking-guix-package.html
3. Git: https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell
4. Git merge-base: https://git-scm.com/docs/git-merge-base
5. Git true merge: https://git-scm.com/docs/git-merge
6. Gerbil introduction, macro phases and preludes: https://gerbil.scheme.org/guide/intro.html
7. Gerbil custom languages: https://gerbil.scheme.org/tutorials/languages.html
8. Direct source reviews: `docs/reviews/gerbil-projects/04-chess-ai-gerbil-source-code-review.md` and `docs/reviews/gnu-guix-system-and-package-manager-scheme-source-review.md` on the above repository/branch.

**Research verdict:** The proposed P03-D02 architecture is a synthesis, **not a feature already delivered by Guix, Git, or the chess AI**. N-ary synthesis, typed causal events, policy evolution, and meta-governance are **new design proposals** and must not be attributed to those upstream systems as implemented features. In particular, Guix's generation mechanism is an important *counterexample* to the requirement of retaining independent future heads after history divergence.

---

## 2. System definition: a programmable causal topology

P03-D02's primary object is a **versioned transformation universe**:

```text
U = (S, R, A, N, H, B, M, E, P)
```

- `S`: validated domain states and all future-relevant context.
- `R`: immutable domain-rule, evaluator, compiler, canonicalization, and schema revisions.
- `A`: admitted actions and explicitly sealed nondeterministic inputs.
- `N`: immutable state-transition/causal-history nodes in an ancestry DAG.
- `H`: **synthesis events** with an ordered or canonicalized *set of source heads* and separate candidate outputs; semantically N-ary hyperedges, implemented as typed incidence nodes in an ordinary DAG.
- `B`: named heads that identify retained historical presents without owning their underlying state.
- `M`: immutable **reconciliation-policy and meta-policy revisions** plus governance decisions.
- `E`: transition, reconstruction, migration, synthesis, and qualification evidence.
- `P`: retention, materialization, resource, and authority policies.

A state transition is governed by a pinned rule revision:

```text
T[r](s, a, i) -> (s_next, witness)
              | Reject(reason)
```

Here `i` closes over recorded seeds, input and clock observations; `w` is an independently verifiable witness or receipt. `T_r` need not be invertible. An immutable node records *how* a state was produced; equal state payloads need not imply equal historical identities. A branch is a mutable pointer to an immutable node. Neither a common ancestor nor hash equality makes two histories semantically mergeable.

### 2.1 Four distinct axes, no artificial coordinates

1. **Execution time:** a local search routine's forward/undo stack. `DeltaToken` lives only here.
2. **Causal/logical time:** the ancestry partial order. Siblings are not temporally ordered just because they share a timestamp.
3. **Structural/configuration space:** alternative states, dependency configurations, constraints, branch arrangements, and possible paths. It is not assumed to be physical X/Y/Z space.
4. **Meta-transformation space:** evolution of domain rules, reconciliation policies, DSL definitions, and admissibility methods, each with its own identity and provenance.

Multiple retained branch heads count as multiple *presents* in the model. They are not claims about physically existing universes or independent concurrently deployed operating systems.

### 2.2 Branching, multi-source synthesis, and multiple outcomes

```text
                        A3 (head A, R1) -------\
                       /                        \
           ancestor ---*---- B3 (head B, R1) ----[SynthEvent E, policy rho2]
                       \                        /          |             |
                        C3 (head C, R2) -------/           v             v
                                                    D1 (candidate)  D2 (candidate)
                                                       |               |
                                               new head D1      new head D2

         The original A3, B3, C3 heads are not modified.
         Both D1 and D2 are new histories only after separate qualification/commit.
```

The graph is **not limited to binary merge**. A synthesis event may depend jointly on `n >= 2` immutable source heads and yield zero, one, or several candidate descendants. No maximum source count is hard-coded into the semantics; practical requests are explicitly budgeted. Distinct candidates from one event must never be silently collapsed into one winner. If the sources do not share a common ancestor, synthesis requires a stronger domain-supplied bridge or must be refused.

Mathematically, synthesis is a **partial, potentially multivalued relation**:

```text
Synth[rho](H1, ..., Hn) ~> P(H_candidate)    (n >= 2)
```

The power set notation describes semantic possibilities, **not** an instruction to enumerate the whole set. An execution returns a bounded subset plus typed completeness status, evidence, and budget accounting. It may return `REJECTED`, `UNVERIFIED`, or `BUDGET_EXHAUSTED` without any committed output.

### 2.3 Three different relations, not one generalized equality

- **Ancestry:** `X` caused or is a historical parent of `Y` in the model. Source identifiers and event receipts establish *recorded model lineage*, not physical-world causation.
- **Comparability:** a declared observer, projection, or semantic bridge can meaningfully compare `X` and `Y`; it does not imply synthesis is admissible.
- **Synthesizability:** a *versioned* policy, necessary migration, global constraints, authority, and evidence jointly admit construction of a new result.

Synthesizability may be policy-relative, directional with respect to a chosen target ruleset, and **neither commutative nor associative**. A two-source-compatible collection need not be jointly synthesizable. Pairwise results are insufficient when global invariants constrain all sources together.

## 3. Data architecture: typed causal incidence graph and versioned policy graphs

### 3.1 Proposed logical contracts (language-neutral, not implemented)

```text
RuleRevision {
  rule_id, parent_rule_ids[], domain_id, source_digest, normalized_ir_digest,
  expanded_ast_digest, compiler_macro_dependency_lock,
  schema_version, transition_evaluator_contract_digest,
  deterministic_input_contract, capability_manifest, qualification_ref, provenance_ref
}

ReconciliationPolicy {
  policy_id, parent_policy_ids[], normalized_ir_digest, expander_lock,
  accepted_source_cardinality, order_semantics, target_rule_selection,
  migration_requirements[], global_invariants[], conflict_handlers[],
  candidate_ordering, effect_manifest, resource_limits, authority_requirements,
  qualification_ref, provenance_ref
}

MetaPolicyRevision {
  meta_policy_id, parent_meta_policy_ids[], normalized_ir_digest,
  allowed_rule_and_policy_transformations[], admission_contract,
  authority_requirements, capability_ceiling, qualification_ref, provenance_ref
}

StateImage {
  image_id, domain_id, schema_version, canonical_payload,
  canonicalizer_revision, state_digest, validation_receipt
}

CausalNode {
  node_id, kind: ROOT | STEP | RULE_SWITCH | SYNTHESIS_OUTPUT,
  parent_event_or_node_ids[], state_ref, rule_revision_id,
  transition_or_migration_receipt_ref, input_receipt_ref,
  logical_generation, validity_status, provenance_ref
}

SynthesisEvent {
  event_id, source_head_ids[], frozen_source_ref_versions[],
  source_order_contract, reconciliation_policy_id, meta_policy_id,
  semantic_bridge_refs[], shared_or_distinct_ancestors[], global_constraint_receipts[],
  candidate_output_ids[], search_completeness, budget_usage,
  qualification_receipt, provenance_ref
}

SynthesisCandidate {
  candidate_id, event_id, output_state_ref, output_rule_revision_id,
  construction_witness, invariant_receipt, semantic_status,
  conflict_resolution_receipt, candidate_rank_optional, publication_status
}

BranchRef {
  branch_id, head_node_id, expected_ref_version,
  owner_ref, retention_class, status
}

GovernanceDecision {
  proposal_id, policy_or_meta_policy_lock, requested_capabilities,
  static_admission_result, proof_or_test_refs[], authority_receipt,
  decision_status, effective_revision, denial_reason_optional
}

UniverseManifest {
  universe_id, domain_id, branch_refs[], pinned_roots[],
  state_rule_policy_registries[], retention_policy_revision,
  active_meta_policy_ref, version, canonical_digest
}

ReconstructionReceipt {
  target_node, anchor_image, replayed_edges[], state_rule_policy_locks[],
  input_locks[], resulting_digest, comparison_oracle, verification_level
}
```

*Invariants for identities:* content IDs, canonical serialization, unique event IDs, and exact semantic equality are different notions. `source_head_ids[]` is a sequence with declared ordering semantics, or a canonically serialized set if a **qualified** policy declares permutation invariance. Neither ordering nor idempotence is implicit. Duplicate source IDs are rejected unless the policy explicitly defines and qualifies multiplicity semantics. Candidate output nodes are distinguished from a synthesis event so one event may yield more than one result without inventing ambiguous multi-head ancestry.

### 3.2 Typed hyperedges without cyclic implementation graphs

At the semantic level, a synthesis is one **hyperedge** from `{H1,...,Hn}` to `{C1,...,Ck}`. The implementation can use a typed **event node** with incoming references to committed source nodes and outgoing references to output nodes. Source nodes must predate event publication, and output nodes must not exist as committed ancestors of the event. This preserves acyclicity and makes the entire source group jointly relevant to one operation.

The causal DAG, the rule-revision DAG, and the meta-policy DAG remain **logically separate**. Cross-graph links are version references, not arbitrary mutation paths. A domain `RULE_SWITCH` is an explicit transition, not an implicit rewriting of earlier historical states. Changing a synthesis policy creates a new immutable policy revision, never retroactive changes to an old synthesis event.

### 3.3 Core invariants I01–I24

1. **I01 — Acyclicity:** New events refer only to previously committed sources; outputs are newly created descendants.
2. **I02 — Immutability:** Committed state, event, rule, policy, and evidence records are append-only; correction creates new records.
3. **I03 — Non-destructive branching:** Forking never rewrites a source head or descendants.
4. **I04 — Atomic head movement:** Head promotion requires expected ref version and fails closed on conflict.
5. **I05 — Exact rule binding:** Transition provenance pins executable, evaluator, schema, canonicalizer, and dependencies.
6. **I06 — Input closure:** Replay claims require sealed nondeterministic inputs and verified environmental assumptions.
7. **I07 — No false inverse:** Checkout, restoration, reconstruction, and synthesis do not claim external effects were reversed.
8. **I08 — Retention closure:** Pinned heads, committed synthesis events, source heads needed for proof, rule/policy locks, and recovery anchors remain recoverable for their stated retention guarantee.
9. **I09 — Search-frame isolation:** Delta tokens are never valid across a changed session, active head, generation, or ruleset.
10. **I10 — Semantic cache isolation:** Identity incorporates rules, policies where relevant, objective, history context, and canonicalization; hashes never suffice as equality proof.
11. **I11 — Typed synthesis:** Comparability and pairwise compatibility are not proofs of N-ary admissibility.
12. **I12 — No effect escalation:** P03-D02 creates scenarios and receipts, not live-world actuation.
13. **I13 — Source preservation:** Synthesizing never mutates, consumes, or silently advances the immutable source histories and their existing branch references.
14. **I14 — Joint global validation:** All-source constraints and output invariants are checked together, not inferred from pairwise tests.
15. **I15 — Plural outputs:** A multi-candidate synthesis is represented as separate identified outputs with distinct evidence and no unrequested implicit selection.
16. **I16 — Policy relativity:** Synthesis admissibility is bound to a pinned reconciliation policy, target rule revision, named observer and evidence revision.
17. **I17 — No algebraic assumptions:** Associativity, commutativity, idempotence, monotonicity, or convergence require domain/policy-specific evidence.
18. **I18 — Fail-closed commits:** Unqualified/partial candidate sets are not automatically committed or promoted to a head.
19. **I19 — No unbounded exploration:** Source enumeration, subset search, candidate generation, rule compilation, recursion, and retained graph expansion have explicit quotas.
20. **I20 — Staged rule safety:** Macro/code-data transformations generate inspectable IR; syntax acceptance is not authorization, proof, or permission to execute arbitrary code.
21. **I21 — Meta-policy separation:** A policy may propose transformations of rules and policies but cannot unilaterally expand its own authority or alter the trusted kernel.
22. **I22 — Monotone governance receipts:** An accepted policy upgrade records old/new revisions, authorization, tests/proofs, activation point and rollback reference; old policy histories remain inspectable.
23. **I23 — Stable failure semantics:** Refusal, conflict, uncertainty, partial exploration, and budget exhaustion are distinct typed outcomes, never silently converted to success.
24. **I24 — Schema-aware migration:** A rule/policy upgrade cannot reuse old caches or replay proofs across incompatible semantics without an independently qualified bridge.

## 4. Public operations, internal lifecycle and atomicity

| Operation | Contract and observable effect | Explicit constraint |
|---|---|---|
| `fork_at(ancestor, branch)` | Creates a distinct branch ref at a retained immutable node | Never modifies old branch heads |
| `checkout(head, session)` | Moves a **session cursor** to a verified state | Invalidates stale search frames/cache, no physical rollback |
| `commit_step(branch, expected_head, action, inputs)` | Validated new immutable STEP + CAS head | No partially published head |
| `fork_rule(rule_ref, change_spec)` | New rule-revision child | Old rule remains available |
| `transition_rule(branch, new_rule, migration)` | Explicit RULE_SWITCH node | Rejects absent required bridge |
| `undo_checked!(frame, token)` | Local LIFO frame restoration | Not history branching or durable recovery |
| `restore_generation!(generation)` | Selects retained immutable state | Not inverse transition |
| `reconstruct_at_revision(node)` | Qualified replay under **original** version locks | Not counterfactual reinterpretation |
| `reinterpret(node, new_rule)` | Constructs a **new** historically distinct scenario | Cannot claim original past was changed |
| `compare(heads[], observer)` | Bounded comparison over one or more histories | Does not imply synthesis |
| `synthesis_propose(heads[], policy, budget)` | Freezes sources, validates admission, explores a bounded candidate set | No committed outputs, no default binary fold |
| `synthesis_qualify(proposal, evidence)` | Validates candidates, all-source constraints, rule bridges, and provenance | May reject or qualify a proper subset of candidates |
| `synthesis_commit(qualified_proposal, expected_refs)` | Atomically publishes event + selected qualified output node(s), optional new branch refs | Existing source heads unaffected; only explicit target refs moved |
| `merge_propose(A,B,policy)` | **Compatibility adapter** to `synthesis_propose([A,B],...)` | No special binary semantics or automatic qualification |
| `fork_reconciliation_policy(policy, change_spec)` | New policy revision | Cannot redefine old events retrospectively |
| `propose_meta_upgrade(meta_policy, change_spec)` | Inspectable, non-executing governance proposal | Self-authorization prohibited |
| `admit_meta_upgrade(proposal, independent_authority)` | Checks static proofs, budgets, separate authorization and migration | Cannot raise kernel capability ceiling |
| `activate_meta_revision(qualified_upgrade)` | Moves an explicit policy reference with CAS, retains prior revision | Failure keeps old meta-policy active |
| `prune_unpinned(retention_policy)` | Collects only reclaimable unreachable objects | Never breaks stated recovery promises |

### 4.1 Two different types of transaction

**Exploration transaction:** speculative search and DSL interpretation operate on isolated, quota-bound ephemeral values. An exhaustive search is not required: `PARTIAL`, `BUDGET_EXHAUSTED`, `UNVERIFIED`, and `NO_CANDIDATE_FOUND_WITHIN_BUDGET` are honest noncommittal outcomes.

**Publication transaction:** freeze source-head version vector; stage new immutable event, candidate nodes, state images, rule/policy locks, proofs, and recovery anchors; verify all dependencies; durably commit staging when supported; then atomically publish a new event/head mapping. Source refs need not be advanced. If multiple new target refs must publish atomically, require a backend transaction or one manifest-level CAS. A series of independent ref CAS operations must not claim all-or-nothing publication.

### 4.2 Three modes of backward navigation, plus branching and synthesis

- `undo_checked!`: independent restoration check of one owned mutable execution frame.
- `restore_generation!`: verified selection of a retained immutable checkpoint.
- `reconstruct_at_revision`: original-code replay of an event chain with pinned inputs.
- `fork_at`: construct a *new* descendant history from any retained ancestor.
- `synthesis_*`: construct *new*, policy-relative candidates from multiple histories, which can fail or yield more than one candidate.

None of these five operations unwinds arbitrary real-world side effects. If a consumer later deploys anything, authorization and effect compensation are separately owned responsibilities outside P03-D02.

## 5. Metaprogramming and code–data duality as management mechanisms

This is not merely a DSL that generates transitions. It is a **versioned rule-and-policy management architecture** that uses Gerbil's macro and code–data affordances to make state rules, synthesis rules, and *rules for changing those rules* inspectable objects.

### 5.1 Explicit compilation and qualification pipeline

```text
Quoted/structured declarative specification (untrusted data)
  -> parser/schema/identity checks
  -> normalized immutable RuleIR / SynthesisIR / MetaPolicyIR
  -> effect, capability, resource and dependency analysis
  -> hygienic macro expansion under pinned Gerbil toolchain
  -> inspectable expanded IR + generated artifact digest
  -> independent tests/proofs and qualification receipt
  -> admission by separately controlled governance policy
  -> versioned, capability-limited runtime execution
  -> immutable transition or synthesis receipts
```

The narrow trusted kernel interprets validated declarative records and protects authority boundaries. A macro can author and generate code; it does **not** automatically gain execution permission merely because source forms are data. The default must prohibit arbitrary `eval`, arbitrary historical-code execution, ambient I/O and unrestricted syntax extensions during `checkout`, `compare`, or static `synthesis_propose`.

**Proposed DSL, illustrative only (not executable Gerbil syntax):**

```scheme
(define-universe scenario
  (state-schema state-v1)
  (rule-revision evolve-v1)
  (meta-policy meta-v1)
  (invariants (valid-state? bounded-resources?)))

(define-reconciliation-policy joint-v3
  (sources (at-least 2))
  (ordering explicit)
  (target-rules rules-v3)
  (constraints (joint-capacity-check) (invariants-hold?))
  (on-conflict (return-qualified-candidates))
  (outputs (candidate-histories))
  (budget (max-candidates 8) (max-combinations 5000)))

(define-meta-policy meta-v2
  (parent meta-v1)
  (permit (fork-rule) (fork-reconciliation-policy))
  (require (independent-qualification) (authorization-receipt))
  (forbid (self-authorization) (kernel-privilege-change)))

(define proposal
  (synthesis-propose scenario '(head-A head-B head-C)
                     joint-v3 bounded-request))

;; The next operations require explicit qualification and admission.
(synthesis-qualify proposal evidence-bundle)
(synthesis-commit qualified-proposal expected-source-versions)
```

### 5.2 Policy recursion and the trusted base

There are *three distinct kinds of change*:

1. **Domain rules:** how an admitted state evolves under a given `RuleRevision`.
2. **Reconciliation rules:** when multiple histories may jointly produce candidates and how those candidates are constructed.
3. **Meta-rules:** which changes to domain or reconciliation rules are allowed, admitted, and activated.

A meta-policy can itself have parents, branches, comparisons and qualified successors. That **does not imply an unbounded chain of autonomous self-modification**. A minimal trusted kernel fixes serialization, identity, authority checks, capability ceilings, immutable publication, and qualification receipt verification. Changing that kernel is a separately authorized **kernel upgrade protocol**, never a side effect of evaluating a policy it is supposed to validate.

There is no claim that such a kernel is mathematically complete or that testing arbitrary generated programs is sufficient for safety. The kernel simply defines the *current trust boundary* and refuses unsupported capabilities.

### 5.3 Required meta-transformation properties

- **M1 — Identity:** every source form, normalized IR, macro-expander/compiler version, policy revision, and generated artifact is referenced by a stable lock.
- **M2 — Staging:** syntax transform, static admission, qualification, and effectful runtime execution are distinct phases.
- **M3 — Historical honesty:** new rules acting on an old anchor create a fresh counterfactual branch, not a reconstructed original event.
- **M4 — Explicit bridges:** changes in schemas, dimensions, units, invariants, action meaning, and policy scope require migrations or refusal.
- **M5 — Introspection:** `inspect-rule`, `inspect-policy`, `diff-policy`, and `explain-denial` read data without granting execution authority.
- **M6 — Cache isolation:** rule and policy revisions invalidate incompatible transformation caches and partial-result indexes.
- **M7 — Independent qualification:** a generated transformation must pass declared oracles before being promoted to trusted use.
- **M8 — No self-endorsement:** a proposed policy revision cannot produce the authority receipt that admits itself.
- **M9 — Bounded recursion:** policy transformations, expansions, and meta-policy ancestry traversals have explicit depth, size, gas and cycle guards.
- **M10 — Non-monotone policy changes:** expanding or restricting permitted synthesis transformations creates a new auditable revision; no implicit retroactive relabeling of earlier candidate validity.

## 6. N-ary synthesis semantics, admissibility, and conflict resolution

**Comparison, admission, candidate construction, qualification, and publication are distinct operations.** A set of histories can be comparable without having any admissible shared descendant.

### 6.1 Comparison levels

```text
EXACT_SAME_NODE
IDENTICAL_STATE_SAME_SEMANTICS
OBSERVATION_EQUIVALENT (named observer + bounds + witness)
STRUCTURALLY_DIFFERENT_COMPARABLE
DIFFERENT_SEMANTIC_REVISION (requires explicit bridge)
INCOMPARABLE (no qualified observer/migration)
UNVERIFIED (insufficient evidence)
```

A match in a hash, package name, GUI appearance, or projected state is never enough to assert equality of historical identities or future behavior.

### 6.2 Admissibility is policy-relative, N-ary, and multivalued

```text
Synth[rho](H1, ..., Hn) ~> {C1, ..., Ck}
(n >= 2; k >= 0; "~>" denotes a partial, multivalued relation)
```

A result has **at least two separate dimensions of status**:

- *Admission:* `DIRECT_ADMISSIBLE`, `CONDITIONAL`, `META_RECONCILIATION_REQUIRED`, `REJECTED`, or `UNVERIFIED`.
- *Exploration/publication:* `COMPLETE_CANDIDATE_SET`, `PARTIAL_CANDIDATE_SET`, `BUDGET_EXHAUSTED`, `NO_CANDIDATE_FOUND_WITHIN_BUDGET`, `QUALIFIED`, `COMMITTED`, or `FAILED`.

`NO_CANDIDATE_FOUND_WITHIN_BUDGET` is *not* proof that no admissible candidate exists. `COMPLETE_CANDIDATE_SET` requires a finite, explicitly bounded search domain and proof the scope was exhausted. `REJECTED` needs a witnessed violated constraint or unsupported admitted policy, not mere search failure. `META_RECONCILIATION_REQUIRED` indicates that existing policies cannot yet establish a bridge; it does not imply that some future meta-policy is guaranteed to succeed.

### 6.3 Two-source checks cannot replace joint checks

Assume a policy with global capacity 10 and three source histories each claiming independent consumption of 4 capacity units. All three pairs total 8 and may be pairwise admissible, but the triple totals 12, violating the global invariant. This illustrates why qualification must consider the entire source set simultaneously. If source effects overlap, the policy must declare double-counting and resource-ownership semantics first; the arithmetic fixture assumes disjoint consumers.

A valid N-way result is not generally equivalent to any chosen binary fold:

```text
Direct = Synth[rho](A, B, C)
Folded = Union(Synth[rho](X, C) for X in Synth[rho](A, B))
Direct != Folded   (in general)
```

In this comparison, a binary-fold intermediate `X` must first be qualified and admitted as a valid source for the second synthesis; rejected or partial outcomes cannot be silently substituted. Even under common rules, information loss, migration order, conflict policy, and inter-source constraints can change the answer. Commutativity, associativity, idempotence, monotonicity, target-rule symmetry, and convergence are **not defaults** and must be qualified separately.

### 6.4 Rule asymmetry and many candidate outputs

A bridge from `R1` into `R2` may exist while the reverse does not, for example when conversion discards information. One set of source heads may admit different outputs under policies `rho1` and `rho2`, or multiple outcomes under the **same** nondeterministic-but-qualified policy. Each outcome is a new node with its own construction witness, final ruleset and provenance. A policy may use a deterministic ordering **only to rank proposals**, not silently declare one candidate the unique correct present.

### 6.5 Required admission sequence

1. Freeze the full list of `n` source head IDs, their current ref versions, and the chosen policy/meta-policy locks.
2. Validate every source, state/rule revision, retention closure and independent source ownership requirement.
3. Determine whether a common ancestor exists and whether any cross-domain/ruleset bridge is required.
4. Check **joint** domain invariants, effect/capability constraints, units, identities, and global resource budgets.
5. Construct candidates in isolated bounded execution; never mutate source histories.
6. Verify candidate state oracles, trace receipts, policy semantics and absence of conflicting effect claims.
7. Return a set of qualified proposals (or typed failure/partial result) without silently publishing anything.
8. Only after explicit admission: commit the synthesis event and selected qualified outputs. Fork named new heads for each selected output; source heads retain their original IDs.

CRDT-style convergence or three-way Git merge may be used **only** as domain-specific qualified solvers. Neither provides general N-way semantic synthesis. Physical causal counterfactual validity is never inferred from simulated lineage alone.

## 7. P03-D02-B bounded search plus lifecycle and combinatorial governance

Retain original P03-D02-B DFS/BFS/IDDFS, Dijkstra/A* subject to objective/heuristic admission, and qualified adversarial search. These work on a committed read-only anchor plus **private ephemeral execution frames**. Speculative search steps are not automatically published as histories. A verified promotion step is required to turn a speculative outcome into a retained state or synthesis candidate.

### 7.1 Combinatorial explosion is a first-class failure mode

For `n` distinct history heads, the number of possible source subsets with cardinality >= 2 is:

```text
N_groups(n) = sum(C(n, k) for k = 2..n)
            = 2^n - n - 1
```

This counts only candidate **groups**; adding policy revisions, transformation order, output multiplicity, rule migrations, and recursively synthesized heads makes the explored space larger. No exhaustive enumeration should be presumed feasible for arbitrary `n`.

### 7.2 Resource and retention controls

Mandatory bounded request fields:

```text
SynthesisBudget {
  max_sources, max_source_subsets_examined, max_candidate_count,
  max_candidate_depth, max_rule_migrations, max_meta_rule_depth,
  max_ir_nodes, max_expansion_steps, max_cpu_or_step_budget,
  max_snapshot_bytes, max_event_bytes, max_live_materializations,
  max_retained_heads, max_reconstruction_steps,
  cancellation_policy, partial_result_policy
}
```

`max_cpu_or_step_budget` is a logical work bound first; wall-clock deadlines are useful diagnostics but cannot safely interrupt uncooperative arbitrary code. No hidden default unlimited budget. The admission manager can decline a request before compilation if the upper-bound estimate exceeds configured limits. Quotas apply across **the entire synthesis request**, not independently per source in a way that accidentally multiplies the allowance.

### 7.3 Lazy universes and shared history

Branch heads, potential joins, policy-derived candidate descriptions, and delta event chains may be represented **symbolically**; only selected states need full materialization. Structural sharing and content-addressed immutable ancestors reduce duplication, but **materialization is never a claim of deterministic reconstruction** without sufficient anchors, pinned rules, and input closure. Eviction must respect protected source heads, synthesis receipts, and the ability to reconstruct every promised candidate.

### 7.4 Management loop

```text
REGISTER -> INDEX -> ADMIT -> PLAN -> MATERIALIZE/EXPLORE
     -> QUALIFY -> PROPOSE -> (COMMIT | DEFER | REJECT)
     -> PIN/RETAIN -> AUDIT -> RECLAIM_ELIGIBLE
```

A `Meta-Control Plane` owns this lifecycle for P03-D02's state/rule/policy topology, not application actuation or external agent authority. Its service boundaries are:

| Service | Owned function |
|---|---|
| `HistoryRegistry` | Causal identities, ancestry, branch heads, protected roots |
| `TransformationRegistry` | Rule versions, normalized IR, compiler/macro lock and migrations |
| `PolicyRegistry` | Reconciliation/meta-policy revisions, status and upgrade receipts |
| `SynthesisPlanner` | Source selection, N-ary constraint dependencies and candidate planning |
| `ConstraintVerifier` | Joint global invariants, typed conflicts and witness checking |
| `MetaCompiler` | Staged DSL parsing, expansion provenance and generated artifact validation |
| `QualificationEngine` | Policy/candidate admissibility and claim-level evidence |
| `LifecycleManager` | Quotas, lazy materialization, retention, garbage collection, cancellation |

Only the minimal kernel has authority to publish committed records and change admitted policy references. The registry/planner cannot authorize its own new capabilities.

## 8. Persistence and P04-D01 integration: durable evidence, no ownership confusion

P03-D02 owns **causal semantics**, including immutable identities, branch heads, multi-source synthesis events, policy and meta-policy locks, retained recovery roots, and admission receipts. It may initially retain these for one process lifetime; no crash-durable claim follows. P04-D01-B (or another independently qualified adapter such as a transactional database/event store) may provide durable object storage, journal integrity, tamper detection, and recoverability. P03-D02 has no compulsory dependency on P04-D01 or a particular database vendor.

The minimal data-only interchange now includes `SynthesisEvent`, `SynthesisCandidate`, `ReconciliationPolicy`, `MetaPolicyRevision`, `GovernanceDecision`, and retention/authority receipts in addition to existing `StateImage`, `CommitBundle`, `BranchRef`, `RuleRevision`, and `RecoveryReceipt`. An independent storage adapter must protect the **entire dependency closure** of every committed synthesis output: all source heads, event records, required policies, semantic bridges, and reconstructed state anchors.

**Durable sequence:** stage objects -> validate digests/references and quotas -> fsync/transactionally seal all reachable dependencies -> commit one manifest or atomic multi-ref update -> record publication receipt -> recover and recheck closure. If multiple new heads share one event, publication must be all-or-nothing where claimed. Otherwise the interface must explicitly support **independently published outputs with partial success receipts**, never pretend an atomic batch existed.

Recovery tests must inject failure before/after staging, before/after manifest CAS, during proof/reference creation, and during candidate selection. Garbage collection cannot evict a policy revision required to verify a retained candidate or a source node required to reconstruct an advertised synthesis result. `prune` may remove unqualified ephemeral candidates if their retention contract permits it.

## 9. Trust, security, and graduated verification claims

State data, declarative policies, macro source, compiler expansions, external inputs, and effect authority are distinct trust domains. A hash proves payload integrity relative to the checked bytes, not safety, totality, determinism, or semantic validity. Neither `checkout` nor `compare` must execute untrusted historical code; `synthesis_propose` must default to static inspection and approved interpretation rather than ambient arbitrary evaluation.

### 9.1 Minimal trusted kernel (not a universal formal proof)

The trusted kernel fixes: typed manifest parsing, canonical identity, append-only/atomic publication, authority and capability ceilings, qualification receipt verification, version-lock compatibility, and failure classifications. Above it, domain rules and reconciliation/meta-policies can evolve, branch, and be proposed as data. A meta-rule may **request** new powers but cannot grant them to itself. Kernel changes require an independent kernel-versioning/upgrade process, out of scope for automatic meta-policy execution.

### 9.2 Claim levels

- `STRUCTURAL_HISTORY_VERIFIED`: graph, identities and references satisfy invariants.
- `CHECKPOINT_RESTORED`: independently validated retained state image selected.
- `REPLAY_VERIFIED`: original rules, inputs and independent oracle verify a specific replay.
- `OBSERVATION_EQUIVALENT`: named observer and tolerances support a bounded equivalence claim.
- `COUNTERFACTUAL_GENERATED`: new model branch, not identified physical causation.
- `SYNTHESIS_PROPOSED`: uncommitted, bounded candidate set with policy and source locks.
- `NARY_SYNTHESIS_QUALIFIED`: specific candidates jointly validate under pinned policy.
- `SYNTHESIS_COMMITTED`: named new histories published under atomicity contract; sources retained.
- `META_POLICY_UPGRADE_ADMITTED`: independently authorized, version-bound upgrade; does not imply universal safety.
- `DURABLE_RECOVERY_VERIFIED`: only after qualified backend, crash injection, and closure audits.
- `UNVERIFIED` / `REJECTED` / `PARTIAL`: reasons preserved without claim inflation.

Every claim identifies exact input head set, policy, rules, qualifications, and scope. Partial exploration cannot be relabeled as complete absence of viable synthesis. Generated counterfactuals do not constitute empirical causal identification absent domain-level identification evidence.

## 10. Extended test corpus and qualification gates

The P03-D02 test register reserves **P03-D02-T01–T80** and **Q0–Q10** for the reversible-state and bounded-search core. The following **32 proposed causal-branching tests (T81–T112)** specify additional obligations. No test or gate is recorded as passed:

| ID | Falsification fixture | Pass condition |
|---|---|---|
| T81 | Fork at a non-tip ancestor | Old head unchanged, new branch valid |
| T82 | Advance original and alternate branches | Both retained heads individually reconstructible |
| T83 | Checkout historical node without fork | No branch-head mutation |
| T84 | Fork multiple times at same ancestor | Deterministic independent provenance for each path |
| T85 | Same rule and inputs, identical replay | Exact observer/digest oracle passes; receipt pins revision |
| T86 | Different rule revisions, same initial state/action | Separate result/revision claims; no cache alias |
| T87 | Modified rule code but unchanged friendly version string | Identity binding detects different expanded semantics |
| T88 | Rule migration changes unit/schema | Reject absent validated converter |
| T89 | Nondeterministic external read omitted from event | Cannot claim `REPLAY_VERIFIED` |
| T90 | Forced truncated replay chain | No fabricated reconstruction |
| T91 | Remove active branch ref while root pinned | Retention/recovery respects declared policy |
| T92 | Simulated GC evicts unpinned object | Protected heads unaffected; unprotected restore explicitly fails |
| T93 | Two writers advance same head concurrently | Exactly one CAS wins; no lost update |
| T94 | Crash before durable head CAS | Existing head intact; staged orphan handling correct |
| T95 | Crash after durable head CAS | Every reachable object recoverable |
| T96 | Checkout while local delta frame active | Frame rejected/closed; stale token cannot mutate other branch |
| T97 | Compare different rules without semantic bridge | `INCOMPARABLE`/explicit differential, not `EQUAL` |
| T98 | Merge conflicting field edits under incompatible rules | Refused without certified resolver/migration |
| T99 | Merge independent commuting actions under qualified resolver | Deterministic receipt, invariant success, both parents retained |
| T100 | Multiple best common ancestors in merge graph | Resolver handles ambiguity or refuses explicitly |
| T101 | Same hashes, different canonical state objects | Equality guard rejects false equality |
| T102 | Distinct branch heads at same wall-clock timestamp | Logical ancestry unaffected |
| T103 | Same content state at distinct historical nodes | Nodes retain distinct lineage even if state images deduplicate |
| T104 | Macro-expansion/compiler revision drift | Replay lock mismatch; no silent substitution |
| T105 | Untrusted rule load during inspect/checkout | No implicit execution |
| T106 | Budget exhausted during speculative search | No materialization beyond quota; committed heads intact |
| T107 | Rule change invalidates search transpositions | Cache entries never cross incompatible contexts |
| T108 | Reinterpret historical state using new rules | Classified as alternative run, not historical replay |
| T109 | Compare via declared lossy observer | Qualified observation equivalence only, not state equality |
| T110 | Failure while preparing transition commit | No half-published head, no false success |
| T111 | Branch names changed or deleted | Immutable historical node and provenance identity unaffected |
| T112 | Counterfactual lacks real-world causal model | No claim of empirical causal identification |

**Causal-branching qualification gates (proposed):**

- **Q11 — Causal graph and non-destructive branches:** T81–T84, T91–T93, T101–T103, T111. Required for any `BRANCHING_CAUSAL_REVERSIBILITY` claim.
- **Q12 — Meta-revision and reconstruction semantics:** T85–T90, T104–T105, T107–T109, T112. Required for cross-rule branching and verified replay claims.
- **Q13 — Comparison, merge and conflict governance:** T97–T100, T109. Required only for any *merge* capability claim; compare-only may qualify independently with stricter explicit qualifiers.
- **Q14 — Durable graph retention/recovery:** T91–T95 and T110 with a real qualified backend and injected crash boundaries. Required only for persistence across process exits.

### 10.1 N-ary synthesis and meta-governance tests: T113–T152

| ID | Falsification fixture | Required result |
|---|---|---|
| **T113** | Three disjoint consumers each use 4 units from capacity 10; every pair is valid | N-ary request for all three rejected on global invariant; pairwise evidence not sufficient |
| **T114** | Three compatible sources under one qualified joint policy | Exactly scoped, replay-verifiable N-ary candidate admitted |
| **T115** | Publish result from three source heads | All three original heads preserved and independently reconstructible |
| **T116** | Order-sensitive policy with permuted input sources | Policy-declared order affects outcome with explicit provenance; no false commutativity |
| **T117** | Qualified permutation-invariant policy with permuted sources | Canonicalization yields semantically equivalent candidate set and stable receipts |
| **T118** | Direct three-way synthesis compared with `(A⊕B)⊕C` | Distinct results/eligibility accurately reported; no assumed associativity |
| **T119** | Same source supplied twice to a policy without multiplicity contract | Duplicate rejected, not silently double-counted |
| **T120** | Zero or one source passed to synthesis API | Typed arity error; no event committed |
| **T121** | Source count exceeds configured bounded request limit | Request refused before expensive rule expansion |
| **T122** | One qualified synthesis event yields two legitimate outputs | Two separate candidate IDs, states, rule locks and receipts; neither auto-selected |
| **T123** | Candidate search stops with none discovered but unexplored search space remains | `NO_CANDIDATE_FOUND_WITHIN_BUDGET`, not impossibility or exhaustive rejection |
| **T124** | Multi-source event persisted with multiple outputs | Event stores frozen full source group; each output can trace through same event to all sources |
| **T125** | Sources have unrelated roots or incompatible rules without bridge | Typed refusal; no invented synthetic ancestor or semantic equivalence |
| **T126** | R1→R2 migration exists, R2→R1 loses essential data | Directional compatibility honored; reverse synthesis not fabricated |
| **T127** | Candidate violates joint output invariant after valid local migrations | Candidate rejected even when every source was individually valid |
| **T128** | Failure after staging two output candidates but before publication | Neither committed; original heads unchanged and staged orphans handled |
| **T129** | One of three source refs changes during proposal evaluation | Expected version-vector check rejects stale publication or restarts explicitly |
| **T130** | Three histories comparable via projection but synthesis incompatible | Comparison succeeds with scope; synthesis refuses |
| **T131** | Quota truncates multivalued candidate set | Result labeled `PARTIAL_CANDIDATE_SET`; unique candidate receipts retained |
| **T132** | Conflicting claims to one resource across three histories | Constraint verifier reports conflict graph and witnesses, not arbitrary last-writer-wins |
| **T133** | Proposal generated under policy rho1, qualified under rho2 by mistake | Policy-lock mismatch rejected |
| **T134** | Same heads processed under two qualified reconciliation policies | Distinct outcomes and policy provenance preserved; no global canonical 'best present' claim |
| **T135** | DSL text changes while friendly policy version remains identical | Immutable normalized IR/source/expander digest distinguishes revisions |
| **T136** | Untrusted code-shaped data included in policy declaration | Parse/inspect has zero incidental eval, file/network effects or privileged access |
| **T137** | Equivalent well-formed declarative IR inputs | Stable canonical representation across pinned toolchain runs |
| **T138** | Macro/expander drift changes generated semantics | Requalification required; no silent reuse of previous receipt |
| **T139** | Newly generated meta-policy attempts to approve itself | Authority boundary rejects self-endorsement |
| **T140** | Recursive meta-policy transformation expands indefinitely | Depth/node/step quotas stop expansion with typed status |
| **T141** | Meta-policy change has valid schema but no independent authority receipt | Cannot activate; existing policy remains current |
| **T142** | Actor requests policy capability beyond assigned authority | Refused regardless of source/DSL validity |
| **T143** | Policy mutation attempts to replace trusted kernel capability checks | Rejected as outside policy scope; only independent kernel-upgrade protocol could change kernel |
| **T144** | Two meta-policy branches diverge from one policy ancestor | Both retained; old synthesis decisions still resolve to original policy |
| **T145** | Two actors activate competing admitted meta-policy versions | One CAS succeeds; other gets version conflict; no torn policy manifest |
| **T146** | Previously qualified event inspected after a stricter policy becomes active | Historical receipt remains attached to old policy, not silently reinterpreted |
| **T147** | Generated policy asks to qualify an unverified branch set by default | Refused: qualification receipt and explicit admission are still required |
| **T148** | Large number of retained heads requests all possible subsets | Enumerator respects global subset budget and reports partial coverage |
| **T149** | Fixed `n` fixture for source groups of size ≥2 | Count equals `2^n - n - 1` when unrestricted enumeration is explicitly performed |
| **T150** | Symbolic candidate graph with lazy materialization | No uncontrolled state/DSL execution until approved; quota and anchor checks enforced |
| **T151** | GC attempts to evict sources, rules, meta-policy or bridge needed by committed candidate | Dependency closure protects promised reconstructibility |
| **T152** | Crash during transaction publishing multiple new heads | Either all-or-nothing under transactional manifest claim, or accurately recorded per-head partial success; never falsely claims atomic batch |

### 10.2 N-ary synthesis and governance qualification gates

- **Q15 — N-ary causal synthesis:** T113–T127, T130–T134, plus carried-forward T97–T100. Required for any `NARY_SYNTHESIS_QUALIFIED` or `SYNTHESIS_COMMITTED` claim. Includes pairwise-insufficiency, multiple candidates, conditional compatibility, and non-associativity fixtures.
- **Q16 — Metaprogrammed policy governance:** T133–T147, with compiler provenance and independent authority audits. Required for evolving reconciliation/meta-policies and the `META_POLICY_UPGRADE_ADMITTED` claim.
- **Q17 — Combinatorial budgets and lazy lifecycle:** T121, T123, T131, T140, T148–T151. Required for bounded N-ary exploration, symbolic candidate retention, and quota claims.
- **Q18 — Atomic multi-source/multi-output publication:** T115, T124, T128–T129, T151–T152, plus retained crash tests T94–T95 and T110. Required for any claimed atomic batch publication or crash-durable synthesis retention; storage guarantees remain adapter-specific.

**Test accounting:** 80 reversible-state/search core proposals + 32 causal-branching proposals + 40 N-ary synthesis/governance proposals = **152 proposed test cases (T01–T152)**. Gates Q0–Q18 total **19 proposed qualification gates**. None is marked passed by this design document.

**Revised qualification statement:** P03-D02-A (in-memory reversible-state/search companion) still requires Q0–Q3 and Q9. The stronger **P03-D02-A causal-branching profile** additionally requires Q11. A **cross-rule replay and reinterpretation profile** requires Q12. Binary merge, if ever exposed independently, still needs Q13. N-ary synthesis adds Q15 and Q17, and any metaprogrammed reconciliation or self-evolving policy management adds Q16. Atomic multi-output publication adds Q18. Durable causal recovery separately adds Q14 and, for synthesis with multiple heads, Q18 under a qualified backend. P03-D02-B retains algorithm-specific Q4–Q8. These gates are intentionally composable: a two-head reversible search engine does not claim all N-ary/meta-governance capabilities by association.

---

## 11. Implementation sequencing (no implementation authorized)

The release strategy separates usable core functionality from speculative metaprogramming and multi-source synthesis. All steps are **proposed work**, not instructions to silently commit or run external code.

1. **C0 — Formal skeleton:** freeze contracts for states, rules, branch/node IDs, event types, qualification receipts, and no-side-effect oracles.
2. **C1 — Branching DAG:** immutable nodes, fork/checkout/CAS, retention roots, session checkpoint recoverability. Qualify Q11 on toy domains.
3. **C2 — Delta search frames:** preserve original `DeltaToken` semantics; verify independent exact undo, exception unwind, hash-collision cases. Re-run original Q2/Q3/Q9.
4. **C3 — Revision-pinned replay:** historical rule/input closure, RULE_SWITCH migrations, replay vs reinterpretation distinction. Qualify Q10/Q12.
5. **C4 — Declarative rule IR:** inspectable normalized `RuleRevision` without arbitrary `eval`; separate generation from execution and enforce lock consistency.
6. **C5 — Original bounded search:** DFS/BFS/IDDFS and admitted optimization profiles. Search speculative nodes are not permanent history without commit.
7. **C6 — Pure comparison:** explicit observer, projection and semantic-bridge claims; qualify comparison without assuming synth admissibility.
8. **C7 — N-ary synthesis reference:** build `SynthesisEvent`, source-set freezing, globally checked constraints, two-or-more-source candidate proposals, and multiple candidate outcomes. Keep publication disabled until Q15/Q17.
9. **C8 — Reconciliation DSL:** add narrow Gerbil macros over the checked declarative data interface, generated IR provenance, static capability analysis, and audited policy registry. Qualify Q16.
10. **C9 — Meta-policy governance:** rule/policy branching, independent admission, CAS policy activation, no self-authorization; include bounded recursion and kernel-upgrade separation. Re-run Q16 adversarial tests.
11. **C10 — Publication transaction:** staged event/candidate atomicity, per-output qualification, expected source version vector, namespace-safe caches and retention closure. Qualify Q18 before batch-publication claims.
12. **C11 — Durable adapter:** integrate P04-D01-B or another qualified storage backend without importing external runtime into the Gerbil core; inject crash/GC faults and qualify Q14 + Q18 as appropriate.
13. **C12 — Independent audit:** reproduce reference oracles, policy-drift differential tests, nested fork/synthesis histories, bounded resource use, and incompatible-semantic refusals. Explicitly publish which profiles remain unqualified.

### Proposed source topology (not created)

```text
editions/reversible-state-search/
  src/state/                       # original state engine and owned DeltaTokens
  src/causal/node.ss
  src/causal/graph.ss
  src/causal/branch-ref.ss
  src/causal/fork.ss
  src/causal/checkout.ss
  src/causal/commit.ss
  src/causal/retention.ss
  src/causal/synthesis-event.ss    # typed N-ary event incidence
  src/rules/revision.ss
  src/rules/transition.ss
  src/rules/migration.ss
  src/rules/expansion-provenance.ss
  src/policies/reconciliation.ss  # immutable policy registry, not authority
  src/policies/meta-revision.ss
  src/policies/admission.ss
  src/policies/governance-receipt.ss
  src/meta/ir.ss                   # data-only canonical IR
  src/meta/compile.ss              # controlled DSL pipeline
  src/meta/qualification.ss
  src/synthesis/request.ss
  src/synthesis/plan.ss
  src/synthesis/constraints.ss
  src/synthesis/candidate.ss
  src/synthesis/qualification.ss
  src/synthesis/publication.ss
  src/synthesis/budget.ss
  src/replay/reconstruct.ss
  src/replay/reinterpret.ss
  src/compare/projection.ss
  src/compare/difference.ss
  src/search/                      # original P03-D02-B algorithm families
  adapters/storage-protocol.ss
  tests/causal/
  tests/synthesis/
  tests/metarules/
  tests/recovery/
  tests/budgets/
```

### Explicit exclusions and integration boundary

- `Meta-Control Plane` here manages P03-D02's causal topology only; it does not seize the responsibilities of P02-D02 agent authority, P04-D01 evidence/database, P03-D01 temporal runtime, or the LFR minimal Meta-Foundation.
- The trusted kernel's current capability ceiling is **not editable by the rules it evaluates**. A kernel upgrade has a distinct independent process.
- No Git dependency, no Guix/Guile runtime bridge, no Python metacompiler, no arbitrary code execution, no assumed real-world time reversal.
- Preservation and analysis of Alien-Works, Chess AI and Guix source remain separate from the new Gerbil edition and from any eventual optional application integration.
- Existing P01-D01–P04-D01, reviews, C02 and upstream repositories are unchanged. This design document is published to the `audit/lisp-foundation-reserve` branch; documentation publication does not authorize runtime implementation.

## 12. Consolidated design verdict

**Disposition:** `DETAILED_DESIGN_PROPOSED / NARY_CAUSAL_SYNTHESIS / CONDITIONAL_NONDESTRUCTIVE_CONVERGENCE / TYPED_CAUSAL_INCIDENCE_DAG / PLURAL_PRESENT_CANDIDATES / RULES_AND_POLICIES_AS_VERSIONED_DATA / STAGED_GERBIL_METAPROGRAMMING / BOUNDED_META_CONTROL_PLANE / TRUSTED_KERNEL_AUTHORITY_CEILING / POLICY_RELATIVE_ADMISSIBILITY / PAIRWISE_NOT_SUFFICIENT / NO_DEFAULT_ASSOCIATIVITY_OR_COMMUTATIVITY / LAZY_BRANCH_MATERIALIZATION / P05_STORAGE_BOUNDARY / 152_FUTURE_TESTS / 19_FUTURE_GATES / UNIMPLEMENTED / UNQUALIFIED / DOCUMENT_PUBLISHED / P01_TO_P05_UNTOUCHED`.

The principal invariant is no longer merely *"going backward must be possible"*. It is:

> **Retained histories are immutable; futures may diverge; admissible histories may converge only into new, separately qualified descendants; the rules governing divergence and convergence are versioned and governable, not self-authorizing.**

Reversibility (`undo`/restore/reconstruct), branching (`fork`), synthesis (`n >= 2`), and metaprogramming (rule/policy evolution) are separate, composable capabilities with separate evidence and qualification. This is a proposed **programmable causal topology**, not evidence of a fully implemented multiverse engine.