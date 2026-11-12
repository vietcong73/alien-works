# LFR-P05-D01 | Retrieval, Persistence & Evidence Reserve

**Document class:** Detailed Design Proposal, specialized edition P05 only  
**Date:** 12 November 2026  
**Programme:** Lisp Foundation Reserve, current architecture LFR-C02  
**Edition:** P05, Retrieval, Persistence & Evidence  
**Target implementation:** Independent Gerbil Scheme, without Python retrieval or obligatory model API  
**Primary source lineage:** AutoContext and Personal AI Assistant from `mark-watson/gerbil_scheme_book@1335cb2708926213ba62afd2998754358f655db1` (11 November 2026)  
**Research baseline:** LFR-C02 and published source audits `08-autocontext-gerbil-source-code-review.md` and `05-personal-ai-assistant-gerbil-source-code-review.md`  
**Status:** `DETAILED_DESIGN_PROPOSED / SOURCE_LINEAGE_MAPPED / DURABLE_MEMORY_REQUIRED / STORAGE_ADAPTERS_UNQUALIFIED / RETRIEVAL_UNBENCHMARKED`  
**Publication:** Owner-approved documentation publication on 12 November 2026 under `docs/design/` in `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`. This publication does not authorize source implementation, deployment, indexing private files, or live model access.  
**Isolation:** Does not modify P01–P04, the minimal Meta-Foundation, existing reviews, original sources, or isolated historical concept 04.  
**Design correction:** Durable memory and evidence persistence are core P05 capabilities; particular database, logging and cloud integrations are interchangeable qualified adapters. The corrected D01 was approved for publication.

---

## 0. Design decision and product purpose

P05 is a **small, independently usable Gerbil retrieval and integrity-aware memory reserve**. It exists so applications can find bounded relevant passages in an owned document corpus, persist memory reliably, and explain exactly which source revision produced each piece of returned context. It does not assume a chat model; applications can search, audit and reuse factual extracts without creating prompts.

Two **separately testable but jointly required P05 capabilities**:

- **P05-A — Pure Retrieval:** tokenize; chunk with source locations; build in-memory BM25 and sparse TF-IDF indexes; rank/fuse results; construct evidence-bearing and budget-bounded context bundles. The pure retrieval library is independently importable and testable from immutable fixtures without a database or network.
- **P05-B — Durable Memory & Evidence:** admit and durably commit validated source/memory/evidence records, preserve lineage and revisions, expose recoverable state, manage tombstones and retention, and rebuild indexes. A **complete P05 deployment MUST configure at least one qualified authoritative durable backend**. The backend can be file-based, a transactional database, an event store, or a qualifying cloud service; no specific implementation or provider is mandatory.

**Qualification distinction:** `P05-A_PURE_RETRIEVAL_QUALIFIED` is possible before storage is built, but is *not* equivalent to `P05_FULL_RESERVE_QUALIFIED`. A running full P05 instance with no verified authoritative store must refuse durable writes and must not claim persistent memory. Conversely, P05-B can be tested without a model. Durable memory is required; **storage-provider choice is optional**.

Evidence schema is common to A and B but **does not become a third all-purpose governance engine**. A P05 `EvidenceRef` records where text came from, not whether a claim is scientifically proven. Evaluation of truth and permission to act are out of scope; P04 remains the independent agent-authority design.

**Consumer examples:** an offline searchable research notebook; a source-linked documentation searcher; a bounded assistant memory store; and a deterministic local RAG context-builder. **Non-goals:** neural semantic embedding engine, knowledge graph database, universal data lake, autonomous agent, unrestricted web crawler, distributed database, LLM provider, general security monitor, or the Meta-Foundation itself.

**Default architecture:** classical lexical retrieval over a **durably committed authoritative record store**, with in-memory or persisted **rebuildable** indexes. Choose one qualified backend according to the consumer environment: local file generations, SQLite, PostgreSQL, an existing event store, or a verified cloud-backed database/object store. Logging/provenance integration is first-class, but a telemetry log does not itself establish recoverable memory. Optional model API remains **outside** P05. Local-first is an implementation and qualification starting point, not an architectural constraint.

## 1. Source-lineage and defect-resolution register

Source status `S`: directly inspected pinned source; `R`: prior static review; `D`: newly proposed Gerbil design, **not tested**. A source failure mode is not necessarily a failure reproduced in a live deployment.

| ID | Original mechanism/problem | Strong part to learn | Source-grounded limitation | New P05 response | Required falsification evidence |
|---|---|---|---|---|---|
| **AC-01** | AutoContext `bm25.ss:46–65`: lexical scoring with term and length statistics | Transparent BM25 calculations (S) | Negative IDF combined with strictly-positive-score filtering in `autocontext.ss:125–135` can drop matching documents (S/R) | Explicit nonnegative IDF variant; separate `matched_terms` from numeric score | One-document exact match; common-term corpus; invariant that a lexical hit is not lost through sign filtering |
| **AC-02** | `bm25.ss:32–65`: corpus-length normalization | Efficient per-document statistics (S) | All-zero-token corpus yields average document length zero and invalid denominator on scoring (S/R) | Empty-index guard; finite-parameter and denominator checks; no result on zero-token/no-match corpus | Empty, stop-word-only, zero-length and all-unknown-term fixtures |
| **AC-03** | `tfidf.ss`: sparse vectors and cosine similarity | Independent lexical vector-space retriever (S) | Zero vectors require safeguards; scores from BM25 and cosine are not automatically comparable | Finite vector norms, explicit sparse TF-IDF, no semantic embedding claim; use rank-based fusion | Hand-calculated golden vectors and zero-vector cases |
| **AC-04** | `autocontext.ss:125–146`: take top-N from BM25 then TF-IDF, concatenate and deduplicate | Stable chunk index makes union easy (S) | Output may contain up to 2N; concat-order is not a calibrated hybrid score (S) | Explicit candidate pools + deterministic reciprocal-rank fusion; final top-K globally | Test exactly K cap; compare RRF to union on labeled corpus |
| **AC-05** | `autocontext.ss:80–88`: sentence-count chunking | Readable source-centered chunks (S) | `chunk-size=0` can recur without consuming input (S/R); huge sentence evades size limit | Validate positive limits; enforce byte/character cap; monotonic progress proof | Zero/negative size, giant sentence, punctuation and Unicode fixtures |
| **AC-06** | `bm25.ss` and `tfidf.ss`: separate top-N loops | Simple ranking interfaces (S) | Negative N handled inconsistently; TF-IDF may consume entire list (S/R) | One checked `TopKRequest` with `0 <= K <= K_MAX`, zero means empty | Negative, zero, excessive and fractional limits |
| **AC-07** | `test.ss`: constructs a prompt and optionally calls Ollama | Small end-to-end demonstration (S) | Print-based demo does not assert ranking/correctness (S/R) | Fixed golden vectors, invariants, differential and metamorphic ranking tests | `make test`/CI nonzero on any failed assertion, no model required |
| **AC-08** | `autocontext.ss:29–118`: sentence chunking/tokenization | Complete Gerbil text pipeline (S) | Chunks lose filename, source offset and revision identity; stop words may remove all tokens | `ChunkRef` binds source ID/revision, original span, tokenizer/chunker versions and chunk digest | Extracted text round-trips to correct source revision/span |
| **PA-01** | `daily_use/cache.ss:32–58`: s-expression file cache | Compact, inspectable storage (S) | Parse failure returns empty `()`, then normal close can overwrite existing content (S/R) | `Absent`, `LoadedEmpty`, `Loaded`, `Corrupt`, `UnsupportedVersion`, `IOError` separate; failed load enters read-only quarantine | Corrupt source then close: original bytes unchanged; recovery tests |
| **PA-02** | `daily_use/main.ss:87–160`: model request and manual memory save | Explicit user-initiated save (S) | API failure becomes ordinary response string and is eligible for later persistence (S/R) | Typed ingress success/error; only explicit accepted records may be saved | Failed API and multi-block/empty response tests; no error saved as answer |
| **PA-03** | `cache.ss:63–94`: keyword-overlap ranking | Easy-to-understand lexical retrieval (S) | Substring `cat` may match `education`; repeated query tokens inflate matches (S/R) | Unicode-aware token boundaries and default unique-term query scoring | Substring trap, repeated-token fixtures |
| **PA-04** | `main.ss:46–57`: reuse up to ten cached answers | Simple useful context injection (S) | Number-of-records limit does not constrain total characters/tokens (S/R) | Query-specific aggregate byte/character/token-estimate quotas with overhead reservation | Giant records, encoding, no overruns |
| **PA-05** | `main.ss:46–57`: prepend cached answers to model prompt | Useful context reuse (S) | Saved model text is not distinguished from instructions or ground-truth (R) | `TrustClass`, `EvidenceRef`, quoted data envelope and no authority elevation | Malicious instructions in retrieved chunk remain untrusted data |
| **PA-06** | `cache.ss:46–58`: direct destination rewrite | Minimal persistent implementation (S) | No temp/rename/fsync/lock; truncation and concurrent writes possible (S/R) | **Required durable-store contract**, implemented by qualified filesystem, transactional DB or cloud adapter; backend-specific commit/CAS/recovery protocol | Crash, disk-full, database transaction, network-ack ambiguity, concurrency and recovery tests |
| **PA-07** | `cache.ss:82–107`: timestamps and manual seven-day prune | Explicit user retention control (S) | Age not used in ordinary retrieval; older answers can appear as current (design limitation) | Freshness/expiry filters separate from ranking; staleness visible even when retention permits | Clock/freshness edge cases; stale result not presented as current |
| **PA-08** | `main.ss:69–85`: parse response content | Simple HTTP/JSON handling (S) | Only first content block of last model output is retained (S/R) | Optional consumer adapter validates all relevant blocks and typed status | Multi-block, non-text-first, empty and error-only responses |
| **PA-09** | `test-cache.ss`: properly failing test harness | Test returns nonzero on failure (S) | Only basic ranking scenarios; hard-coded temp path can collide (S/R) | Per-test isolated directories, crash-injection and independent index/property tests | Parallel tests, persistence reload, corrupted bytes, external state |

**Legal note:** Source reuse licensing must be reviewed per actual file and dependency; no assumption that all examples share the Coding Harness's AGPL header. Prefer independently designed Gerbil contracts and original code, with source pins and attribution. No automatic copying of source into P05.

## 2. Module architecture: retrieval plus mandatory durable evidence

```text
P05 CORE [REQUIRED CONTRACTS]
  SourceRef / MemoryRecord / EvidenceRef / TrustClass / Revision
               |                                |
          P05-A RETRIEVAL                 P05-B DURABLE MEMORY
        tokenizer / chunker              admission / transaction / CAS
        BM25 / TF-IDF / RRF              authoritative record/event store
        bounded ContextBundle            recovery / retention / replay
               |                                |
               +---- index from exact committed generation ----+
                                                |
                        StoreDriver [ONE QUALIFIED PRIMARY REQUIRED]
                           |          |          |          |
                       filesystem   SQLite   PostgreSQL   cloud/event store
                           +------ optional replicas, audit sinks -------+

OUTSIDE: model API, project-specific log owners, deployment credentials,
         OS permission boundary, cloud accounts and approval interfaces
```

An **authoritative durable store** holds accepted records and their commit history. **Operational logs** report attempted/observed actions and may be exported to existing project logging systems. **Retrieval indexes** are derived and reconstructible. These are three different responsibilities, even when a single transactional database stores tables for all of them.

### 2.1 Provider-neutral contract, not provider-neutral guarantees

A `StoreDriver` advertises its supported semantics; there is no weakest-common-denominator fiction. A full P05 deployment selects one authoritative driver and a **DurabilityProfile** specifying `PROCESS_CRASH_SAFE`, `POWER_LOSS_SAFE`, `REMOTE_ACKNOWLEDGED`, and required read consistency only when those properties have been qualified. The driver must refuse a profile it cannot fulfill.

Candidate adapters:

- **Local immutable generations + journal:** portable conceptual baseline and crash-test oracle; filesystem-qualified implementation required for durability claims.
- **SQLite:** local transactional storage for a single-host consumer; qualify its transaction mode, synchronous/journal configuration, file system, backup and concurrent writer behavior; an SQLite label alone guarantees nothing about power-loss protection.
- **PostgreSQL / managed relational DB:** suitable for concurrently accessing clients and richer queries; qualify transaction isolation, version checks, acknowledgements, recovery and security configuration.
- **Append-only event store / existing logging infrastructure:** adapter may persist accepted record events and reconstruct snapshots **if** it exposes reliable record ownership, ordering, integrity, durable acknowledgment, recovery, retention and idempotency semantics. A best-effort JSONL audit file or process log alone is *not* automatically the memory database.
- **Cloud object store / managed database:** conditional on capability contracts for object versioning or compare-and-swap, conditional writes, read consistency, persistence acknowledgment, encryption, access policy and recovery. Async replicas and backups remain separate statuses rather than assumed synchronized.

**Backend plurality is supported at the interface level, not promised as one universal distributed transaction.** Optional secondary sinks can receive events via a durable outbox and track independent acknowledgement/cursor watermarks. Memory remains locally or remotely committed only at the authoritative store's defined commit boundary. A secondary cloud copy being behind is `REPLICATION_LAG`, not loss of the primary commit. When the authoritative store itself is remote and unavailable, P05 must not acknowledge persistence merely because a local queue contains a proposal.

### 2.2 Logging integration, existing precedent, and ownership

The user's existing project process-ledger precedent, e.g. `projects/usd-centric-cross-asset-risk-analysis/evidence/process/project_events.jsonl` in the separate `quant-risk-analytics-portfolio` repository, is a **project-owned research process log**. P05 may learn its append-only event, checkpoint, hash, and remote-readback discipline or provide a *consumer-configured adapter*; it must **not** open, append to, relocate, replace, or duplicate that canonical project log by default. Likewise, the previously discussed event/checkpoint/artifact ideas for other projects are precedents, not an already-installed generic P05 engine.

P05 records two related but separate event classes: `MemoryMutationEvent` (accepted durable state changes) and `AuditObservation` (requests, denials, failures, read/access traces). Auditing a failed write does **not** make that write committed; a committed record should have a traceable event and generation. A DB transaction can commit the record and outbox event atomically; an external logging sink is normally updated **afterward** with at-least-once delivery plus idempotent event IDs. If audit delivery fails, mark `AUDIT_DELIVERY_PENDING` without claiming either that the memory commit failed or that the audit was delivered.

### 2.3 Proposed source structure (not created)

```text
editions/retrieval-persistence/
  gerbil.pkg
  README.md
  lineage/
    source-lock.md
    defect-register.md
  src/
    records/
      identity.ss
      sources.ss
      evidence.ss
      trust.ss
      errors.ss
      versioning.ss
    retrieval/
      tokenize.ss
      chunk.ss
      corpus.ss
      bm25.ss
      tfidf.ss
      rank.ss
      fusion.ss
      filters.ss
      context.ss
      diagnostics.ss
    memory/                        # P05-B: required capability
      admission.ss
      journal.ss
      snapshot.ss
      transaction.ss
      consistency.ss
      commit.ss
      recovery.ss
      retention.ss
      reconcile.ss
      outbox.ss
      audit-event.ss
    storage-contract/
      store-driver.ss
      capabilities.ss
      durability-profile.ss
      result.ss
    adapters/
      fixture-corpus.ss
      mock-store.ss
      local-generation-store.ss    # initial qualification target
      sqlite-store.ss              # candidate
      postgres-store.ss            # candidate
      event-store.ss               # candidate
      cloud-object-store.ss        # candidate
      audit-log-sink.ss            # optional integration
  tests/
    retrieval/
    memory/
    crash-recovery/
    backend-conformance/
    replication/
    audit-integration/
    privacy/
  qualification/
    test-corpus/
    relevance-goldens/
    driver-evidence/
```

The directory is a proposed topology; no new source code or integration exists yet. The first qualified primary backend is chosen during implementation; other adapters remain `NOT_IMPLEMENTED` or `UNQUALIFIED` until tested, without weakening the mandatory P05-B contract.

## 3. Data and evidence contracts

Each serialized object has `schema_version`, canonical encoding policy and a size limit; unknown required versions must fail closed.

| Record | Essential fields | Invariant |
|---|---|---|
| `SourceRef` | source ID, origin URI/path label, owner, revision/generation, content digest, media type, policy flags | Stable reference identifies a specific source revision, never a vague filename alone |
| `SourceSnapshot` | source ref, immutable bytes/text, encoding, ingestion status, length, digest | Decode/normalization failure is not silently an empty document |
| `ChunkRef` | chunk ID, source revision, byte/span offsets, chunker/tokenizer revision, content digest | Returned span resolves to the exact ingested text, or has an explicit unverifiable status |
| `TokenStream` | normalized tokens, token positions, normalization profile, stopword policy, empty flag | No empty-token division by zero; deterministic normalization |
| `IndexSnapshot` | corpus generation, tokenizer revision, ranking configuration, ordered chunk IDs, statistics digest | Scoring never mixes incompatible corpus/index generations |
| `QuerySpec` | text, top-K, filters, scoring/fusion config, quotas, caller context | Invalid K or quotas are rejected before retrieval |
| `CandidateHit` | chunk ID, matched terms, score, method, rank, diagnostic status | Lexical match is not inferred from `score > 0` alone |
| `RetrievalHit` | chunk ID/ref, fusion rank, method evidence, source span, trust/freshness labels | Every hit names its source and exact version |
| `ContextBundle` | query identity, selected hits, rendered fragments, byte/character estimates, truncation, provenance | Strict aggregate budget; contains only data, never authority instructions |
| `MemoryRecord` | record ID, origin, content/source ref, observed time, saved time, status, trust class, revision | API errors and unvalidated responses cannot masquerade as accepted content |
| `StoreManifest` | format version, authoritative store ID, generation, snapshot digest, prior generation pointer, commit state | Readers can distinguish committed good generations from incomplete writes |
| `DurabilityProfile` | requested failure tolerance, commit/ack rule, read consistency, provider capabilities, retention | Required guarantees must be admitted by an independently qualified driver |
| `StoreDriver` | backend ID/version, namespace ownership, `open/read/commit-if-version/recover` contract, capability declaration | The selected primary store is authoritative; unsupported capabilities fail closed |
| `MemoryMutationEvent` | event ID, record revision, predecessor, mutation type, durable generation, provenance, commit status | Accepted memory changes are replayable/idempotently identifiable |
| `AuditObservation` | event/correlation ID, operation, outcome, redaction class, observed generation, delivery state | Observational logging never substitutes for a confirmed memory commit |
| `OutboxEntry` | event ID, committed generation, sink ID, retry count, ack/watermark, delivery status | External delivery is independently observed; duplicate sends are deduplicable |
| `StorageResult` | tagged success/error/uncertain state, revision, committed/unknown effects, durability/replica acknowledgement, diagnostics | Corruption is never indistinguishable from valid empty storage; ack semantics are explicit |

Source IDs and chunk IDs must not be conflated. Two identical chunks in distinct documents may share a content digest but retain distinct source-location identities. Stable IDs may be derived from `(source-revision, byte-span, chunker-version, text-digest)` using a canonical encoding; canonicalization is deliberately versioned. Collisions in an identifier implementation must be detected, not treated as permission to overwrite unrelated content.

### Trust and factual status

Separate labels:

- **Origin class:** `LOCAL_USER`, `IMPORTED_DOCUMENT`, `TOOL_OBSERVATION`, `MODEL_OUTPUT`, `DERIVED_SUMMARY`.
- **Integrity status:** `HASH_VERIFIED`, `DECODED`, `CORRUPT`, `UNKNOWN`.
- **Truth/evidence status:** `UNCHECKED_CLAIM`, `SOURCE_QUOTATION`, `EXTERNALLY_CHECKED`, `CONTRADICTED`, `WITHDRAWN` (if a consumer supplies genuine verification evidence).
- **Freshness status:** `CURRENT_AS_OF`, `STALE_AS_OF`, `UNDATED`, `UNKNOWN` with an explicit evaluation timestamp.
- **Visibility classification:** consumer-provided access label; P05 is *not* a kernel security boundary, and does not infer access rights from text.

`HASH_VERIFIED` means bytes match the snapshot, **not** that the text is true. `SOURCE_QUOTATION` establishes attribution to a record, **not** scientific or legal validity. A text fragment saying “ignore instructions” remains source data even when retrieved with high score.

## 4. Ingestion, text normalization and chunking

Pipeline:

```text
bytes + SourceRef
 -> bounded decode [or explicit error]
 -> preserve original content snapshot and encoding
 -> text extraction [verified adapter, if needed]
 -> normalization with source offset map
 -> bounded sentence/paragraph or fixed-size chunking
 -> ChunkRef + token statistics
 -> immutable CorpusSnapshot
```

**Text policy:** start with plain UTF-8 text only. PDFs, scanned images, HTML and binary formats are out of scope unless a separately qualified extractor supplies accurate spans and provenance. Never claim a byte offset in the original PDF when it really locates only a post-extraction string.

**Tokenizer:** versioned Unicode casefold/normalization policy; deterministic token-boundary rules; strict maximum bytes, code points, tokens and token length. A default stopword list may improve ranking but is not universal. Store original spans so raw text can be reconstructed independently of the token stream. Repeated query tokens are unique by default; a separately declared query-term-frequency mode may use multiplicity intentionally. Language-specific tokenization is optional and must be benchmarked before claiming multilingual relevance.

**Chunker:** positive `max_chars`, `max_bytes`, `max_tokens_estimate`, `max_chunks_per_source`, and nonnegative overlap with `overlap < max_chunk_size`. For a giant sentence, split on bounded boundaries and mark the split reason, rather than emitting an unbounded chunk. Every iteration consumes original text; reject configurations with no guaranteed progress. Empty or stopword-only text creates no scoreable term vector but may remain in preservation records.

**Source spans:** maintain mapping to original decoded text offsets, including any Unicode transformations. When a mapping cannot be reconstructed, represent `OFFSET_UNVERIFIED` explicitly; a fabricated source location is worse than an honest unknown.

## 5. P05-A: lexical scoring and ranking semantics

### 5.1 BM25 baseline

Choose a declared Okapi BM25 variant with a nonnegative smoothed IDF to avoid conflating matches with negative weights. For each term with `1 <= df(t) <= N`:

```text
idf(t) = ln(1 + (N - df(t) + 0.5) / (df(t) + 0.5))
score(D,Q) = SUM over distinct matched query terms t:
  idf(t) * [tf(t,D)*(k1+1)] / [tf(t,D)+k1*(1-b+b*len(D)/avgdl)]
```

Default candidate parameters `k1=1.5`, `b=0.75`, **configuration values, not empirical optima**. Require finite `k1 > 0` and `0 <= b <= 1`. Missing terms contribute zero by design; no unnecessary calculation for `tf=0`. For `N=0`, `avgdl=0`, no usable terms, or empty query, return an empty hit set instead of manufacturing a denominator. For terms unseen in the corpus, no matching result.

A `CandidateHit` exists when a chunk shares at least one indexed query token, **not** because some arbitrary numerical threshold is positive. A matched rare/common term must not vanish due to an IDF sign. The exact IDF variant must be stored in the index schema so scoring changes cannot silently alter old results.

For one document containing a single matching term, `N=df=1`, the selected IDF is `ln(1+1/3)`, positive; its BM25 score is positive and finite. This is a design oracle; a runtime pass is not asserted.

### 5.2 TF-IDF and cosine

Sparse term frequency and smoothed document frequency computed against one immutable index generation. Suggested weight `w(t,d)=(1+ln(tf(t,d)))*(1+ln((N+1)/(df(t)+1)))` when `tf>0`. Cosine similarity returns zero for either zero-norm vector; it never returns NaN/Infinity. Numeric overflows, nonfinite weights and damaged index tables must be explicit errors or safe refusal, not silently promoted.

TF-IDF is **lexical vector-space retrieval**, not a neural semantic embedding. The original AutoContext documentation likewise distinguishes this; P05 must not market it as semantic search. Scores from TF-IDF and BM25 are different scales and should not be directly added without a calibrated method.

### 5.3 Hybrid ranking via reciprocal rank fusion

Default candidate: compute a bounded candidate pool per retriever (`pool_size` >= `top_k`; default proposal `min(corpus_size, 3*K)`), remove nonmatching/zero-vector results, deduplicate by **chunk identity**, and fuse via deterministic reciprocal rank fusion:

```text
RRF(chunk) = SUM over participating ranked lists m: 1 / (c + rank_m(chunk))
```

`rank_m` is 1-based; `c` is a documented positive parameter, default candidate `60` (not a calibrated optimum). Results tied on RRF sort by best individual rank, then canonical chunk ID. Apply a **single final `top_k`**, never top-K per source accidentally yielding 2K. Preserve per-method score/rank diagnostics; the RRF sum is a *ranking heuristic*, not a probability or confidence interval.

Other fusion modes (`BM25_ONLY`, `TFIDF_ONLY`, `UNION_LEGACY`) may be included as explicit reproducible comparators, not implicit defaults. Relevance comparison with the published union baseline is mandatory before claiming improvement.

### 5.4 Filtering and deterministic ordering

Apply identity/access/freshness constraints before context release. A stable ranking sort must not depend on hash-table iteration or file listing order. Frozen corpus and query with the same version/configuration must reproduce ordered results. Updates or deletions create a new index generation. If a snapshot is stale relative to store state, report its generation rather than silently mixing documents from different revisions.

**Candidate complexity:** a naïve baseline may score every chunk, with worst-case operations proportional to corpus size and query terms; no claim of large-corpus performance is justified. Inverted indexes or incremental index maintenance are later optimizations requiring correctness equivalence and memory benchmarks. No GPU or neural database dependency is necessary.

## 6. Context assembly and evidence preservation

Retrieval is not the same as packing context for another system. `assemble_context(QuerySpec, hits, quota)` must reserve space for required delimiters, query, source labels and consumer overhead before choosing text. Require `max_items`, `max_bytes`, `max_chars`, and optional versioned token estimator; if a remote provider's *actual* tokenizer is unknown, an estimate must not be advertised as a hard provider-token guarantee.

Algorithmic contract:

1. Validate all requested quotas, source access and index generation.
2. Order hits by fused rank. Group or diversify only if explicitly configured and benchmarked.
3. Deduplicate exact `ChunkRef`s; nearby overlapping chunks may be coalesced **only with traceable source spans**.
4. Allocate remaining budget after fixed/metadata overhead; use a deterministic truncation policy that never cuts UTF-8 bytes mid-codepoint and records omissions.
5. Emit `ContextBundle` as structured data: each fragment includes source ID, revision, original span, trust/freshness label, content digest and reason for inclusion.
6. Render for a consumer as clearly delimited **untrusted quoted source content**. Delimiters and prompt wording do not create a security boundary; the consuming agent must maintain its own instructions/policy.
7. If no hit can fit the budget, return `NO_FITTING_CONTEXT` and a receipt; never silently exceed it.

An optional external model renderer or local chat front end may use the bundle. P05's default tests require **no** model. The answer's factual support remains the responsibility of the consumer: a high-ranking passage is not evidence that its claims are true.

## 7. P05-B: required durable record admission and storage

### 7.1 Admission boundary

Only typed, validated content is eligible to become a `MemoryRecord`. A `MODEL_OUTPUT` may be stored as **unverified historical output** with explicit owner acceptance; it may not automatically be converted into `VERIFIED_FACT`. A failed API call, timeout or transport error must result in `INGEST_FAILED` and may be kept in an error log, **not** in normal answer/context records. Explicit manual save is an acceptable consumer policy and should be preserved as an option.

Input files must have a declared owner and import scope. P05 is not an unbounded scanner of a user's home directory. This design never requests collecting private files, credentials or conversation history into a new store without a consuming application's authority and user's own configuration.

### 7.2 Store open states

```text
ABSENT          -> optionally initialize new empty generation by explicit command
LOADED_EMPTY    -> verified committed empty store
LOADED          -> verified committed records
CORRUPT         -> quarantine / read-only; preserve bytes; no overwrite
UNSUPPORTED     -> read-only or refuse until migration approved
IO_ERROR        -> refuse writes, report operating-system error
CONFLICT        -> another writer owns a different committed generation
RECOVERY_NEEDED -> inspect valid committed snapshots; no blind empty fallback
```

**Never** map `CORRUPT`, `UNSUPPORTED`, `IO_ERROR` or `RECOVERY_NEEDED` to `LOADED_EMPTY`. `close()` must not rewrite a store unless an actual approved mutation was successfully committed. An empty cache after a parse failure is not evidence that the user deliberately cleared data.

### 7.3 Authoritative commit protocol (provider-neutral)

**A full P05 has one selected authoritative durable store per namespace**. All accepted memories and evidential revisions must be committed there before a successful `MemoryWriteReceipt` can be returned. The abstract protocol is:

1. Validate owner scope, record schema, origin/trust status, current revision and store capabilities.
2. Begin an operation with a **stable idempotency key** and expected generation (optimistic CAS or an equivalent database transaction).
3. Persist the new record revision, its `MemoryMutationEvent`, and the durable outbox entry as one authoritative commit **when the backend supports atomic grouping**. Otherwise mark the weaker guarantee explicitly and refuse a stronger requested profile.
4. Obtain the backend-specific durability acknowledgment and committed generation. `UNKNOWN_ACK` requires reconciliation by idempotency key rather than blind replay.
5. Update/rebuild indexes against exactly the committed generation. Export observation events to logging or cloud replicas after the primary commit, tracking each sink's delivery independently.

**Filesystem baseline example:** an initial local driver may use immutable *generation snapshots* plus a small current-manifest pointer on the same filesystem, with one writer per store. The following steps apply specifically to this file-backed adapter, **not** to PostgreSQL, SQLite, or cloud backends:

1. Open known-good current manifest; check schema, digest, generation and writable state.
2. Obtain an exclusive writer lock (or a store-specific capability implementing equivalent mutual exclusion), and re-check current generation as a compare-and-swap precondition.
3. Build a new immutable snapshot and index metadata from validated records, in a uniquely named private sibling staging location.
4. Flush file contents and metadata according to the selected durability policy; verify snapshot digest and record count.
5. Write a new manifest pointing to the complete snapshot, and atomically replace the current-manifest name through a qualified same-filesystem primitive. Flush parent-directory metadata if crash-durable profile requires it.
6. Release lock only after recording a definite outcome or declaring `COMMIT_UNCERTAIN` where the filesystem did not provide a decisive result.
7. Readers must verify current manifest + snapshot digest before exposing records; old complete generations remain available according to retention policy.

A rename can offer **namespace atomicity for one manifest entry**, not automatically an ACID transaction across external files, devices, databases or sudden power loss. POSIX fsync semantics vary by filesystem/driver; do not assert durability before real fault-injection qualification. If a suitable filesystem cannot provide the specified guarantees, report `UNSUPPORTED_DURABILITY` or explicitly run a weaker documented profile.

**Recovery:** inspect manifest candidates and prior complete generations. Never overwrite an unreadable generation merely because parsing failed. A recovery operation can propose promoting the last verified generation, but it must preserve the corrupt candidate and report the resulting data-loss window. Do not reapply ambiguous writes or silently change histories.

**Concurrent writers:** lock acquisition, current-generation compare-and-swap, and no lost-update checks are mandatory for the multi-process profile. Do not rely on a lock file's mere existence or on an in-process mutex as a multi-process guarantee. If OS-level concurrency semantics cannot be qualified, reject multiple-writer operation.

### 7.4 Database, logging and cloud backend semantics

- **SQLite**: use a transaction for record/event/outbox writes; enforce expected revision, unique event IDs, and schema migration. Configure actual journal/synchronous settings intentionally. Test concurrent processes, crash recovery, backup consistency and the exact durability class.
- **PostgreSQL**: use transactional writes and compare-and-swap predicates or serializable operations where justified; event/outbox identity and row versions must prevent lost updates. Verify client acknowledgment/connection-loss ambiguity, failover consistency and authorized read filtering.
- **Existing append-only event ledger**: first define ownership, one canonical path/stream per project, event ordering, schema and checkpoint strategy. A generic log may be an audit sink or a valid primary event store only if commit and replay guarantees meet `StoreDriver`. Never create a second canonical research log without owner direction.
- **Cloud object store**: use immutable versioned objects and conditional manifest update (where supported); verify read-after-write/consistency semantics and recover when acknowledgement was lost. Encryption, object retention, keys, credentials and deletion obligations belong to an explicit deployment policy, not implicit defaults.
- **Cloud managed database**: defer durability/transaction semantics to the selected managed DB's verified contract; distinguish provider acknowledgement from cross-region replication. Avoid pretending a remote API call and local audit append form one transaction.
- **Secondary sinks / telemetry**: use transactional outbox + idempotent consumers or a documented equivalent. `OUTBOX_PENDING`, `AUDIT_DELIVERY_PENDING`, `REPLICATION_LAG`, and `REPLICA_UNVERIFIED` are not memory-commit failures if the authoritative generation is verified, but they must remain observable.

**Consistency choice:** a primary read can require `READ_COMMITTED_GENERATION` and read-your-own-writes by recorded generation. An eventually consistent replica can be served only when the caller explicitly accepts a watermark; the system must not silently return stale data as current. No unconditional multi-region or cross-backend ACID guarantee is made.

### 7.5 Index-to-store consistency

The selected **authoritative durable store** is the source of truth; a project's external audit log is not automatically that store. Lexical index files, if persisted, are rebuildable derived artifacts. `IndexSnapshot.corpus_generation` must exactly equal `StoreManifest.generation`; a mismatch produces `INDEX_STALE`, not mixed results. On recovery, rebuild in memory from the last verified source snapshot, then validate corpus and tokenizer versions before exposing search.

### 7.6 Retention, correction, deletion and privacy

A record may be expired for ordinary retrieval without being deleted; retention and freshness are separate. Corrected claims receive a new revision and a link to the superseded record. Deletion is an explicit owner-authorized event producing a tombstone and index invalidation; real byte erasure and backup purge are **not** automatically proved by a tombstone. A user-facing privacy workflow must separately define snapshot retention, backup deletion and encryption where applicable. No default indefinite retention of private data is assumed.

## 8. Failure and uncertainty taxonomy

| Failure | Meaning and required handling |
|---|---|
| `BAD_QUERY`, `BAD_TOP_K`, `BAD_CHUNK_CONFIG` | Reject before ranking or chunking |
| `TOKENIZATION_UNSUPPORTED`, `SOURCE_DECODE_ERROR` | Preserve source identity; no misleading empty document |
| `EMPTY_CORPUS`, `NO_MATCH`, `NO_FITTING_CONTEXT` | Valid nonerror outcomes, distinct from failure |
| `NONFINITE_SCORE`, `INDEX_CORRUPT`, `INDEX_STALE` | Do not return an apparently valid rank from broken statistics |
| `SOURCE_SPAN_UNVERIFIED` | Return or omit according to caller policy, no fabricated citation |
| `RECORD_REJECTED`, `UNVERIFIED_MODEL_OUTPUT` | No accidental promotion to trusted factual evidence |
| `STORE_ABSENT`, `STORE_CORRUPT`, `STORE_VERSION_UNSUPPORTED` | Distinct storage opening states |
| `LOCK_CONFLICT`, `STALE_GENERATION`, `PARTIAL_WRITE`, `COMMIT_UNCERTAIN` | Do not assume write succeeded, retry without reconciliation, or lose prior generation |
| `DURABILITY_UNQUALIFIED`, `UNSUPPORTED_DURABILITY`, `NO_AUTHORITATIVE_STORE` | Never overclaim persistence; refuse full-reserve write mode without a qualified primary backend |
| `UNKNOWN_ACK`, `OUTBOX_PENDING`, `AUDIT_DELIVERY_PENDING`, `REPLICATION_LAG` | Reconcile authoritative generation and secondary sink separately; no blind retry |
| `REPLICA_UNVERIFIED`, `READ_CONSISTENCY_UNMET`, `BACKUP_PURGE_UNVERIFIED` | Do not overclaim freshness, verification or erasure |
| `SOURCE_WITHDRAWN`, `STALE_SOURCE`, `ACCESS_DENIED` | Filter and label according to current context policy |

`NO_MATCH` is normal. `STORE_CORRUPT` is not. Both may yield zero returned hits, but collapsing them destroys the diagnostic meaning of the system.

## 9. Example vertical slices (proposed, not implemented)

### Slice A — Pure retrieval on an immutable fixture

Corpus has (a) a single exact common-term match, (b) a zero-token document, (c) duplicate text across two independently identified sources, (d) a very long sentence, and (e) a query with repeated terms. P05-A produces finite scored hits, stable source-located citations, and a global top-K. No database, network or model call.

### Slice B — Crash-safe memory lifecycle (mock store first)

Start from a committed three-record generation. Inject malformed data, read failure, interrupted write, stale writer and simulated disk-full/rename uncertainty. A normal close must not erase the original. New generation visibility is all-or-nothing under the **mocked manifest protocol**; real filesystem claims remain unqualified until OS tests.

### Slice D — Logging, DB and cloud convergence (injected drivers)

Commit `MemoryRecord` revisions N and N+1 to a mock transactional primary and emit outbox events. Simulate audit delivery failure, duplicate event delivery, delayed secondary cloud replica, and a lost primary acknowledgement. Recover by event id and committed generation; confirm no duplicate memory write, no false `AUDIT_DELIVERED`, and a bounded stale-replica read labeled by watermark. Then run the same fixture against each independently admitted backend.

**Pass:** evidence is durably committed at the selected primary before a success receipt; audit/replica health remains separately observable. A disconnected remote primary cannot be misreported as durable merely because a request was queued.

### Slice C — Bounded context and trust propagation

Retrieval finds one user-authored source and one old model output containing malicious instructions. A strict 4 KiB context quota caps selection; both fragments retain origin/trust labels; the rendering does not grant the model output policy authority. The example is *not* a proof that a future LLM cannot be prompt-injected.

## 10. Behavioral API proposals (not executable Gerbil syntax)

```text
validate_source(SourceRef, SourceSnapshot)     -> SourceResult
chunk_source(SourceSnapshot, ChunkerConfig)    -> ChunkResult
build_corpus(chunks, TokenizerConfig)          -> CorpusSnapshot | Error
build_index(CorpusSnapshot, RankConfig)       -> IndexSnapshot | Error
search(IndexSnapshot, QuerySpec)               -> [RetrievalHit] | Error
explain_hit(RetrievalHit)                      -> ScoreAndProvenance
assemble_context(hits, Quota, RenderPolicy)    -> ContextBundle | Error

register_store_driver(DriverSpec, qualified_profile)   -> DriverRegistration | Refusal
open_memory(namespace, StoreDriver, DurabilityProfile)    -> StoreOpenResult | Refusal
admit_record(snapshot, CandidateRecord)                   -> AdmissionResult
prepare_mutation(snapshot, ChangeSet, event_id)           -> PreparedMutation
commit_if_generation(prepared, expected_generation)       -> MemoryWriteReceipt | UnknownAck
read_committed(namespace, generation, consistency)         -> VerifiedSnapshot | Error
reconcile_event(namespace, event_id, primary_observations) -> CommitState | Unknown
rebuild_index(VerifiedSnapshot, RankConfig)              -> IndexSnapshot | Error
queue_audit_delivery(committed_event, sink)               -> OutboxResult
observe_sink_watermark(sink, event_id)                    -> DeliveryStatus
expire_records(store, RetentionPolicy)                  -> ChangeSet
withdraw_record(store, exact_record_ref)                 -> ChangeSet
```

`admit_record` should not pretend a model's answer has been fact-checked. `commit_if_generation` cannot claim a durability level that its selected backend has not demonstrated. `queue_audit_delivery` does not prove external sink acknowledgement. `assemble_context` never executes instructions contained in retrieved text. The operation signatures are behavioral pseudocode, **not** real Gerbil APIs.

## 11. Test matrix: 58 exact scenarios

These are specification targets, not completed results. CI must exit nonzero on assertion failure. Use hand-calculated numeric goldens, small human-labeled corpora and independent reference implementations where needed.

| ID | Scenario | Expected contract |
|---|---|---|
| **P05-T01** | Build retrieval-only profile without store, model, network or Python | Pure dependency isolation |
| **P05-T02** | Empty corpus and valid nonempty query | `EMPTY_CORPUS`, zero hits, no invalid arithmetic |
| **P05-T03** | One document, one matching term | Positive, finite BM25 hit |
| **P05-T04** | A term occurs in every document | Matches preserved, no negative-IDF filtering loss |
| **P05-T05** | All documents tokenize to empty | No division by zero; zero lexical hits |
| **P05-T06** | Query contains only stop words | Normal empty-query/no-match outcome |
| **P05-T07** | Query term absent from corpus | No false match, no NaN |
| **P05-T08** | BM25 score hand-worked two-document fixture | Formula/parameter golden |
| **P05-T09** | Invalid `k1`, `b`, Infinity/NaN parameters | Reject before index creation |
| **P05-T10** | Sparse TF-IDF hand-calculated document/query | Exact approximate cosine golden within tolerance |
| **P05-T11** | Zero TF-IDF query/document norm | Zero similarity, never NaN/Infinity |
| **P05-T12** | Query repeats identical token | Unique-term default prevents unintentional amplification |
| **P05-T13** | Token `cat` vs word `education` | No arbitrary substring match |
| **P05-T14** | `top_k=-1`, fraction or enormous value | Checked refusal |
| **P05-T15** | `top_k=0` | Valid empty result without running unneeded ranking |
| **P05-T16** | One retriever finds K, second finds disjoint K | Final fused output at most K, not 2K |
| **P05-T17** | Tied RRF scores | Deterministic secondary ordering |
| **P05-T18** | Duplicate chunk indices in multiple lists | One hit with both method receipts |
| **P05-T19** | Identical text at different source locations | Distinct provenance identities |
| **P05-T20** | Reordered source loading of same canonical corpus | Identical ordered results |
| **P05-T21** | Single huge sentence | Chunk bounded and source text consumption progresses |
| **P05-T22** | Zero/negative chunk size and illegal overlap | Fail before chunker loop |
| **P05-T23** | Unicode casefold, multibyte offsets | Correct versioned normalization and source span mapping |
| **P05-T24** | Decode failure / unsupported media type | Error, not valid empty document |
| **P05-T25** | Retrieving old index after source revision | Explicit generation mismatch or labeled old snapshot |
| **P05-T26** | Source span/revision digest tampered | Citation rejected or marked unverifiable |
| **P05-T27** | Giant context hits under aggregate quota | Selected bundle stays below byte/char limits |
| **P05-T28** | Rendering metadata overhead exceeds quota | `NO_FITTING_CONTEXT` with evidence |
| **P05-T29** | Context truncation at Unicode boundary | Valid UTF-8 and recorded truncation |
| **P05-T30** | Model-generated malicious instructions in a retrieved chunk | Trust label retained as data; no P05 policy elevation |
| **P05-T31** | Expired source retained but stale | Correct freshness label; policy filter works |
| **P05-T32** | Store absent vs committed zero-record snapshot | Distinct opening states |
| **P05-T33** | Malformed s-expression/persisted snapshot then ordinary close | Bytes unchanged; quarantine/read-only state |
| **P05-T34** | Format revision unsupported | Refuse writes until migration approved |
| **P05-T35** | Failed API request proposed as memory | `RECORD_REJECTED`, not accepted answer |
| **P05-T36** | Valid model response stored at owner request | `MODEL_OUTPUT` origin, unverified status, not trusted fact |
| **P05-T37** | Prepare new generation then crash before manifest | Prior committed snapshot remains readable |
| **P05-T38** | Interrupted manifest update | Recover last verified generation or report uncertainty |
| **P05-T39** | Disk full during staged write | No silent replacement of prior valid data |
| **P05-T40** | Two writers race on same generation | No lost update; one conflict or serialized commit |
| **P05-T41** | Reopen after successful committed snapshot | Same records, hashes and manifest generation |
| **P05-T42** | Index generation differs from store generation | Rebuild/refuse, never mix records |
| **P05-T43** | Record corrected with superseding revision | Both lineage and current selection preserved |
| **P05-T44** | Withdraw/delete record | Tombstone and index invalidation, no unsupported erasure claim |
| **P05-T45** | Real Linux process is terminated during file/dir sync | Report honest durability result after recovery testing |
| **P05-T46** | Labeled retrieval corpus compares BM25, TF-IDF, union, RRF | Reproducible MRR/nDCG/Recall@K with no unsupported superiority claim |
| **P05-T47** | Full P05 starts with no authoritative backend | Refuse durable mode; pure retrieval remains usable |
| **P05-T48** | An ordinary JSONL audit log is offered as the sole memory backend | Refuse unless qualified commit/replay capabilities exist |
| **P05-T49** | SQLite primary commits record + event + outbox then reopens | One coherent committed generation, verified on qualified configuration |
| **P05-T50** | PostgreSQL primary concurrent compare-and-swap writers | No lost revision or duplicate event; tested isolation contract |
| **P05-T51** | Cloud primary loses response after conditional manifest write | Reconcile by event ID/object version; no blind duplicate commit |
| **P05-T52** | Same idempotency key retried after unknown acknowledgement | Resolve original outcome or report unknown; never silently double-write |
| **P05-T53** | Primary commits, external logging sink crashes before delivery | Memory committed; durable outbox remains pending and traceable |
| **P05-T54** | Audit/cloud sink receives same event twice | Idempotent event identity, one logical observation at consumer |
| **P05-T55** | Cloud replica lags authoritative generation | Read watermark and staleness visible; strong read routed or refused |
| **P05-T56** | Replay committed event log after index loss | Recover correct record revisions and deterministic index generation |
| **P05-T57** | PII/secrets present in memory records but audit sink enabled | Audit payload excludes restricted contents; policy failures visible |
| **P05-T58** | Same conformance fixture across two qualified store drivers | Matching logical revisions/results; provider-specific durability separately reported |

## 12. Qualification gates

| Gate | Scope | Minimum evidence |
|---|---|---|
| **Q0 — Source / legal provenance** | Verified source pins, review findings, file-level licensing decision | Exact repository revision, source anchors, no automatic source port |
| **Q1 — Pure corpus construction** | Normalization, bounded chunking, provenance | T01, T21–T24 and source-span oracles |
| **Q2 — Ranking correctness** | BM25, TF-IDF, fusion, deterministic ordering | T02–T20; hand-computed and differential goldens |
| **Q3 — Evidence/context** | Trust and truth labels, source integrity, quotas, freshness | T25–T31; negative injection and oversized-content tests |
| **Q4 — Durable store model** | Core admission, revision, event identities, CAS, recovery and outbox | T32–T44 plus T47–T48 and T52–T56 in fault-injection model |
| **Q5 — One real primary backend (mandatory for full P05)** | Real DB/filesystem/cloud durability, authoritative acknowledgment and corruption handling | T37–T45 plus the applicable backend conformance cases (T49–T51); `QUALIFIED` only for tested profile |
| **Q6 — Adapter/interoperability (per backend)** | Logging, database/cloud sync, consistency watermarks, idempotent outbox, privacy | T49–T58; each optional adapter qualifies independently; no unqualified multi-backend claim |
| **Q7 — Retrieval quality & budget** | Relevance and performance vs source baselines | Labeled corpora T46, memory/latency bounds, regression analysis |
| **Q8 — Independent audit** | Reproducibility, negative tests, licensing, statuses | Independent rerun and PASS/REJECTED/UNQUALIFIED by component |

Q2 can pass for **P05-A alone** while durable storage remains unimplemented. **Full P05 may not pass without Q5**, at least one real qualified authoritative durable backend. Q4 mock correctness does not qualify Q5 power-loss durability. Q6 can remain partial for unimplemented PostgreSQL or cloud providers without blocking a fully qualified single-backend P05; nevertheless those adapters may not be advertised as implemented. Q7 may reject RRF if the evidence supports a simpler retriever. Neither richer algorithms nor more storage providers inherently improve reliability.

### Quality metrics (explicit evaluation definitions)

For a fixed relevance-labeled corpus, report `Recall@K`, `MRR@K` and `nDCG@K` with exact corpus version, query list, relevance grades and adjudication instructions. For speed, report memory footprint, index-build wall time and retrieval latency distribution over fixed hardware and warm/cold conditions. No target metrics or dominance claims are invented prior to measurement. Include separate fixtures for English and any claimed other language; do not generalize English tokenization quality to multilingual use.

## 13. Proposed implementation sequence if later authorized

1. **P05-00:** pin source/licensing evidence; freeze corpus fixtures, durability contract, and numerical reference oracles.
2. **P05-01:** implement common identity/source/evidence records, `StoreDriver` capability descriptions, mutation event IDs and typed storage results.
3. **P05-02:** implement pure tokenizer/chunker, guarded BM25, sparse TF-IDF, deterministic fusion and context quotas. Qualify **P05-A only**.
4. **P05-03:** implement the mandatory P05-B in-memory state/transaction model with revisions, idempotency keys, tombstones, a mock authoritative store and fake crash/replay tests.
5. **P05-04:** implement and qualify **one** real durable primary backend, initially selected by deployment need (local generations or SQLite are natural low-complexity candidates, not architectural mandates).
6. **P05-05:** verify coherent retrieval over exact committed generations, indexing/rebuild/recovery, and read-your-writes semantics.
7. **P05-06:** add durable outbox, audit-logging adapter contract and export receipts; preserve existing project-owned canonical logging schemes without unauthorized mutation.
8. **P05-07:** add PostgreSQL, event-store or cloud adapters **only as applicable**, with individual conformance, privacy, consistency and failure-injection gates.
9. **P05-08:** run relevance, latency, memory, storage and recovery benchmarks, including negative tests. Select actual default ranking and storage profiles on evidence.
10. **P05-Q01:** independent qualification report, separately distinguishing P05-A, full P05, each backend and every replication/audit adapter.

Only **P05** is in scope. No LLM integration is a prerequisite; no changes to P01–P04, Meta-Foundation design, source-preservation vault, system configuration, or other GitHub files are authorized by this design document. Documentation publication was separately approved.

## 14. Open questions and decisions not to conceal

1. **Tokenizer semantics:** exact Unicode casefold/normalization rules and language-specific handling must be pinned and reproducible in actual Gerbil/Gambit runtime.
2. **Source formats:** first release is UTF-8 text; PDFs/HTML and extracted offsets need independently qualified adapters.
3. **RRF vs union:** RRF is the chosen default *candidate*, subject to rejection if the relevance corpus supports a simpler option.
4. **Query policy:** should stopword-only queries be reported as `EMPTY_QUERY_AFTER_FILTER` or trigger an explicitly configured non-stopword fallback? Default is transparent empty.
5. **Source access filtering:** P05 relies on consumer-provided access labels, not a security kernel. A mixed-trust multi-tenant service would require a separate security boundary.
6. **Store format and drivers:** use a strictly versioned data-only logical schema and canonical digest grammar. SQLite/PostgreSQL/cloud physical formats differ; never execute imported serialized code.
7. **Durability and consistency:** define precisely the acknowledged commit boundary, process-crash/power-loss/remote-ack classes, reader consistency and reconciliation semantics for each driver and deployment.
8. **Backup and privacy:** deletion of active indexes does not prove secure erasure of old generations, backups or flash devices.
9. **Freshness:** ordinary retrieval must not silently present historical snapshots as live facts; timestamps are labels and filters, not proofs.
10. **Evidence truth:** source identity + hash is provenance/integrity, not factual validation; any independent fact-checking workflow is outside P05.
11. **Model outputs:** whether any should enter persistent memory by default must be a consumer policy; design default is explicit owner acceptance and unverified trust label.
12. **Logging integration ownership:** existing project process logs/checkpoints remain under their project governance; P05 has no implicit permission to change a canonical event log. An adapter contract must name the owner, schema, sink cursor and permitted mutations.
13. **Cloud deployment:** choose remote primary vs optional replica explicitly. Network failure, lag, privacy retention, encryption, access controls and data-residency policies require consumer configuration and qualification.
14. **Backend scope:** qualifying local durable memory does not make SQLite, PostgreSQL, event-store or cloud adapters automatically functional; each gets a separate verdict.
15. **Cross-edition promotion:** P05 records may inspire shared K0 `Identity`, `Evidence`, and `Result` interfaces only after genuine independent cross-domain tests; no premature Meta-Foundation dependency.

## 15. Verdict

`P05_D01_DETAILED_DESIGN_PROPOSED / AUTOCONTEXT_BM25_TFIDF_LINEAGE / PERSONAL_AI_MEMORY_LINEAGE / GERBIL_NATIVE / PURE_RETRIEVAL_INDEPENDENT / DURABLE_MEMORY_REQUIRED / PLUGGABLE_STORAGE_AND_LOGGING / EXPLICIT_PROVENANCE_AND_TRUST / SCORE_AND_CORRUPTION_GUARDS / MODEL_OPTIONAL / NO_RUNTIME_QUALIFICATION / P01_P02_P03_P04_UNTOUCHED / DOCUMENT_PUBLISHED`

P05's distinct role is to **remember durably and retrieve without losing, fabricating or silently elevating evidence**. Full P05 requires a qualified authoritative persistent store; logging, databases and cloud providers can be integrated through validated adapters, with explicit distinctions between authoritative commits and observability/replication. AutoContext contributes lexical retrieval architecture and its identifiable edge cases; Personal AI contributes lightweight explicit memory and a concrete failure path demonstrating why persistence needs correctness contracts. The result is a proposed specialized Gerbil edition, not a claim that existing source code is safe, ported or benchmarked.

---

## Source references

- [Pinned AutoContext](https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext)
- [AutoContext BM25 implementation](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/bm25.ss)
- [AutoContext TF-IDF implementation](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/tfidf.ss)
- [AutoContext ingestion and hybrid assembly](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/autocontext.ss)
- [Pinned Personal AI Assistant](https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use)
- [Personal AI cache implementation](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss)
- [Personal AI model and context processing](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss)
- [Existing AutoContext review](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects/08-autocontext-gerbil-source-code-review.md)
- [Existing Personal AI review](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/reviews/gerbil-projects/05-personal-ai-assistant-gerbil-source-code-review.md)
- [LFR-C02 concept](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/concept/LFR-C02-source-lineage-gerbil-reserve-concept-2026-11-12.md)
- [Previous P04 design](https://github.com/vietcong73/alien-works/blob/audit/lisp-foundation-reserve/docs/design/LFR-P04-D01-agent-authority-controlled-execution-detailed-design-2026-11-12.md)