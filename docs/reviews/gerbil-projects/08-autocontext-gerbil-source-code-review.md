# 08. AutoContext: Gerbil Scheme Source Code Audit

**Repository:** [mark-watson/gerbil_scheme_book/source_code/autocontext](https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext)  
**Pinned revision:** `1335cb2708926213ba62afd2998754358f655db1` (November 11, 2026)  
**Review date:** November 12, 2026  
**Scope:** Direct inspection of Gerbil tokenization/chunking, BM25, TF-IDF/cosine similarity, fusion, prompt construction, and Ollama bridge.  
**Method:** Static code analysis and numerical counterexamples derived from the source formulas. No native `gxi` execution, external model test, or retrieval benchmark.  
**Disposition:** `REAL_GERBIL_RETRIEVAL_IMPLEMENTATION / RANKING_AND_EDGE_CASE_DEFECTS_IDENTIFIED / BENCHMARK_QUALIFICATION_PENDING`.

## Implementation and strengths

This is a small, fully Gerbil retrieval pipeline, rather than an API wrapper around a Python embedding library. [`autocontext.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/autocontext.ss) tokenizes and chunks `.txt` inputs and constructs context prompts. [`bm25.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/bm25.ss) implements a term-frequency/length-normalized scoring index; [`tfidf.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/tfidf.ss) builds sparse TF-IDF vectors and scores them using cosine similarity. Retrieved positions are maintained as **chunk indices**, enabling deterministic deduplication across retrievers. Ollama calls are optional.

The source documentation accurately identifies TF-IDF as a lexical vector-space retriever, **not** a neural embedding model. The core retrieval functions remain independent of Ollama, which is good separation.

## Findings

### AC-01 — Negative BM25 IDF + positive-score filtering suppresses valid lexical hits (medium, numerical counterexample)

[`bm25.ss:46–65`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/bm25.ss#L46-L65) computes `IDF = log10((N - df + 0.5)/(df + 0.5))`, a variant that becomes negative for sufficiently common terms. Negative IDF itself is a known BM25 variant choice. However, [`autocontext.ss:125–135`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/autocontext.ss#L125-L135) discards any BM25 score not strictly positive.

Direct counterexample under its own formula: one corpus document, one matching term, term frequency 1 and document length 1. Then `N=df=1`, `IDF=log10(0.5/1.5)≈-0.477121`, and BM25 score is ≈-0.477121. The exactly matching document is removed from the **BM25 branch**. The TF-IDF branch can still include it, so this is not necessarily a failure of final hybrid retrieval, but it invalidates the intended lexical-hit behavior. Consider using a nonnegative IDF variant or separating match existence from rank/score filtering.

### AC-02 — Zero-average-length corpus causes division by zero (high for input robustness)

[`bm25.ss:32–36`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/bm25.ss#L32-L36) computes `avgdl` from token counts. A document containing only stop words can produce a nonempty text chunk yet an **empty token list** (see [`autocontext.ss:29–51`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/autocontext.ss#L29-L51)). If every chunk tokenizes to empty, `avgdl=0`. A subsequent nonempty query enters [`bm25.ss:52–65`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/bm25.ss#L52-L65), which evaluates `dl/avgdl`. This path is undefined/division-by-zero. A guarded empty-index strategy should return no lexical hits.

### AC-03 — “Hybrid” output is ordered concatenation, not score fusion (design limitation)

[`autocontext.ss:125–146`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/autocontext.ss#L125-L146) extracts top-N indices from BM25, then top-N from TF-IDF, concatenates them and deduplicates by position. This is a useful simple union, but does **not** normalize scores or implement reciprocal-rank fusion, and ranks BM25-only hits before TF-IDF-only hits irrespective of cross-method evidence. More importantly, final prompt can have **up to 2N chunks**, not N. It should be described as top-N-per-retriever union, not a single calibrated hybrid ranking.

### AC-04 — No aggregate context-size budget (medium operational limitation)

Neither [`get-prompt`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/autocontext.ss#L125-L146) nor `chunk-text` caps context characters/tokens. Chunks are grouped by number of sentences, so one very long sentence can dominate prompt size; selecting 2N results is not a token budget. Missing source filenames/offsets in the stored chunks also makes provenance reconstruction difficult.

### AC-05 — `chunk-text` with `chunk-size = 0` loops without progress (conditional exported-API defect)

[`autocontext.ss:80–88`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/autocontext.ss#L80-L88) has an inner loop that terminates when `k = 0` and then calls the outer loop with **the same unconsumed sentence list**. For a nonempty input and `chunk-size: 0`, recursion makes no progress. The default is 3, so the normal pipeline is not affected; the exported function should validate that chunk size is a positive integer.

### AC-06 — Negative `num-results` gives inconsistent retriever behavior (conditional)

[`bm25.ss:76–77`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/bm25.ss#L76-L77) stops when N≤0, but [`tfidf.ss:59–66`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/tfidf.ss#L59-L66) stops when K=0. For negative K, the TF-IDF loop may run through the entire list, potentially including all documents. `num-results` should be validated before dispatch.

### AC-07 — Test is a demonstration, not an assertion suite (test qualification gap)

[`test.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/autocontext/test.ss) prints a prompt for a single example question, then catches Ollama failures and continues. It does not assert ranking order, invariants, zero-token behavior, deterministic parsing, or retrieval quality. `make test` therefore cannot be treated as proof of retrieval correctness.

## Bounded test recommendations

- Single-document and common-term BM25 scenarios, alongside an explicit decision about negative IDF semantics.
- Empty-directory, all-stop-word corpus, and one-empty-token-document cases.
- Zero/negative N and chunk size input validation.
- Small hand-computed BM25 and cosine similarity goldens, tie-handling and deduplication tests.
- Token-budget accounting and provenance metadata in each retrieved chunk.
- Compare union ordering with score-normalized fusion and reciprocal rank fusion using a fixed labeled corpus.

**Overall:** A clean, understandable Gerbil retrieval implementation with good module separation and a real classical IR core. The key weakness is **retrieval correctness at scoring/boundary conditions**, rather than any inability of Scheme to express ranking algorithms.