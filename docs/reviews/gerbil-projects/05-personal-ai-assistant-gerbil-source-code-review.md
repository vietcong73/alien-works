# 05. Personal AI Assistant CLI: Gerbil Scheme Source Code Audit

**Project:** [mark-watson/gerbil_scheme_book — daily_use](https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use)  
**Upstream revision:** [`1335cb2708926213ba62afd2998754358f655db1`](https://github.com/mark-watson/gerbil_scheme_book/commit/1335cb2708926213ba62afd2998754358f655db1), November 11, 2026  
**Review date:** November 12, 2026  
**Scope:** Direct inspection of the Gerbil Scheme CLI client, persistence/retrieval implementation, tests and build script; not a review of the Gemini model or Gerbil language itself.  
**Method:** Static review of complete relevant source bodies, connecting error, cache, retrieval, prompt-construction and HTTP response call paths. No Gerbil `gxi` or live Google API test was executed.  
**Disposition:** `GERBIL_SOURCE_REVIEW_COMPLETE / PERSISTENCE_AND_RESULT_CLASSIFICATION_DEFECTS / RAG_LIMITATIONS_DOCUMENTED / RUNTIME_VALIDATION_PENDING`

## 1. What was implemented

`daily-use` is a Gerbil Scheme command-line assistant that calls the Google Gemini Interactions API. The interactive loop allows normal queries, search-grounded queries, manual persistence of the last answer, and manual pruning of cached answers. It is a **retrieval-augmented LLM client**, not an autonomous tool-using agent and not a locally trained language model.

- [`daily_use/main.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss): 239 lines (REPL, tokenization, prompt assembly, HTTP/JSON request and response processing).
- [`daily_use/cache.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss): 111 lines (s-expression read/write, ranking, clearing).
- [`test-cache.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/test-cache.ss): 36 lines (keyword-rank assertions).
- [`build.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/build.ss): 8 lines (`defbuild-script` for a compiled `daily-use` executable).

### Effective call path

```text
REPL input
    ├── !<query> -> enable google_search tool in Gemini request
    ├── <query>  -> no search tool
    ├── >        -> persist *last-answer*
    └── !        -> explicitly prune cache older than seven days

Query -> extract-keywords -> cache-lookup -> prepend up to 10 answers
      -> HTTP POST Gemini Interactions API -> last model_output/first content block
      -> display-answer -> *last-answer* -> optional disk write
```

The client stores **answers only** as `(timestamp . answer-string)` pairs. It does not preserve the original question, source identity, model response provenance, conversation ID, references, or complete interaction history. No `previous_interaction_id` is sent. Therefore, the persistence mechanism is a lightweight reusable answer cache, not replayable conversation memory.

## 2. Positive design observations

1. **Straightforward module boundaries.** `main.ss` owns interaction, model communication and prompt construction; `cache.ss` owns storage and lookup. The call graph is small enough to inspect end to end.
2. **Real native Gerbil application.** Gerbil standard libraries handle HTTP and JSON, while `defbuild-script` builds the CLI binary. No Python orchestration is concealed behind the Scheme frontend.
3. **Explicit manual persistence.** Users must enter `>` to store the last response; normal queries do not automatically populate durable memory.
4. **Controlled search option.** The `!<query>` path conditionally includes a Google Search tool declaration rather than sending every request through search.
5. **A correctly failing test entry point.** Unlike a test that merely prints `FAIL`, `test-cache.ss:32–35` exits nonzero when its suite does not report `OK`.
6. **API key supplied through an environment variable.** `main.ss:59–68` uses `GOOGLE_API_KEY` instead of embedding a secret in the repository.

## 3. Source-grounded findings

### PA-01 — Read errors become silent empty caches, later overwriting old data

**Severity:** High (persistence integrity; conditional on file read failure).  
**Evidence:** [`cache.ss:32–58`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss#L32-L58), [`cache.ss:109–110`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss#L109-L110), [`main.ss:183–190`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L183-L190).

`load-cache-entries` catches every read exception and returns `()`, indistinguishable from an intentionally empty file. `make-cache` accepts the resulting empty list. `main` always invokes `cache-close` after a normal REPL exit; `cache-close` calls `save-cache-entries`, which writes the in-memory entries back to the same file.

**Counterexample call path:** malformed/truncated s-expression cache -> `read` raises -> `with-catch` returns `()` -> normal startup with zero entries -> user exits -> empty cache replaces formerly recoverable file contents. The write is not guarded by an indication that the original load failed.

This is a source-confirmed destructive-recovery path, not a claim that data loss has occurred on a particular user's machine. Distinguish `not found` from `read failed`; preserve and back up unreadable files rather than overwriting them.

### PA-02 — API failures are returned as answer strings and can be persisted

**Severity:** Medium (memory integrity).  
**Evidence:** [`main.ss:87–108`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L87-L108), [`main.ss:126–160`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L126-L160).

`ask-gemini` catches exceptions and returns a formatted `"[Error calling Gemini API: ...]"` **string**. `display-answer` treats any non-false string as a valid response and stores it in `*last-answer*`. The `>` command can then add it to persistent cache; `build-context-from-cache` may later prepend that error to the prompt.

**Proof path:** failed HTTP request -> string error -> `*last-answer*` -> `>` -> `cache-add` -> future context. The `>` operation remains user-initiated; the application does not cache every error automatically.

The interface should return a structured success/error result, and only successful model outputs should be eligible for durable memory.

### PA-03 — Retrieval scores arbitrary case-insensitive substrings

**Severity:** Medium (retrieval relevance).  
**Evidence:** [`cache.ss:63–94`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss#L63-L94), [`main.ss:35–44`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L35-L44).

The lookup routine `string-contains-ci?` searches for a keyword within the **entire answer text**, not within token boundaries. For example, the keyword `cat` matches the unrelated word `education`. Repeated query terms are not deduplicated before `count-matching-keywords`, so a repeated keyword can receive disproportionate weight.

This is a demonstrated scoring property, not proof of a specific harmful model answer. The cache is a simple keyword-overlap retriever; it should not be described as semantic retrieval, BM25, or document search with independent term statistics.

### PA-04 — No total context/token budget

**Severity:** Medium (robustness, latency and API cost).  
**Evidence:** [`main.ss:46–57`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L46-L57).

`cache-lookup` returns up to ten matching answers, but **individual answer lengths are unrestricted**. `build-context-from-cache` concatenates all returned strings into the request prompt. Ten sufficiently long entries can exceed an API context limit or produce avoidable cost and latency. No truncation, token estimation or maximum aggregate byte budget is visible.

### PA-05 — Cached model text is promoted into prompt instructions

**Classification:** Trust-boundary/design risk, not a demonstrated exploit.  
**Evidence:** [`main.ss:46–57`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L46-L57), [`cache.ss:54–58`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss#L54-L58).

The saved answers are plain model-generated text. They are concatenated after an imperative preamble: `Use the following context from previous conversations when answering:`. No explicit delimiting, source attribution, or trusted-versus-untrusted content representation exists. A saved answer containing bogus instructions can enter future prompts as if it were authorized context. The cache is manually curated but still contains unverified AI output; correctness and trust should not be equated with persistence.

### PA-06 — The cache is neither crash-atomic nor concurrency-safe

**Classification:** Durability/operational risk.  
**Evidence:** [`cache.ss:46–58`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss#L46-L58).

`save-cache-entries` uses `call-with-output-file` on the destination itself. The code shows no temporary file/atomic rename, fsync discipline, or lock shared across multiple running assistant processes. A partial write or concurrent writers may leave incomplete or stale cache contents. No live write-interruption test was performed; this is a bounded design observation.

### PA-07 — Cache age is ignored during ordinary retrieval

**Classification:** Expected behavior with stale-context implications.  
**Evidence:** [`cache.ss:82–107`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/cache.ss#L82-L107), [`main.ss:162–175`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L162-L175).

Timestamps are retained in the file but ignored for ranking and ordinary lookup. Entries older than seven days persist in context until the user executes the bare `!` prune command. This matches the documented command design and should **not** be labeled an implementation bug, but it limits suitability for rapidly changing facts.

### PA-08 — Only the first block of the last model output is returned

**Classification:** Conditional response-completeness limitation.  
**Evidence:** [`main.ss:69–85`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/daily_use/main.ss#L69-L85).

The HTTP parser locates the last step with `type = "model_output"`, then reads the `text` field of **only `(car content)`**. If the response contains several text content blocks, later blocks are discarded; if the first is a non-text block, the function returns `""` despite possible subsequent text. The Gemini Interactions API documents a `steps` sequence and a `content` array, so multi-block handling should not be assumed to be unnecessary. This is not a claim of a reproduced Google API failure.

### PA-09 — Unit tests do not qualify persistence or API handling

**Classification:** Test coverage gap.  
**Evidence:** [`test-cache.ss:1–35`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/daily_use/test-cache.ss#L1-L35).

The single suite exercises keyword ordering on three strings. It does not independently reload serialized entries from a fresh cache object, test malformed cache recovery, empty keywords, time-based pruning, API exception classification, multi-block responses, or maximum prompt size. The test uses a hard-coded `/tmp/test-daily-cache.sexp` path and deletes that file, so parallel runs also have a collision/interference risk. Crucially, however, the test entry point *does* exit nonzero on suite failure.

## 4. Design and implementation trade-offs

| Area | Strength | Weakness |
| --- | --- | --- |
| Gerbil module structure | Small, auditable modules; standard-library integration | Informal data contracts (`hash` and untyped strings) |
| CLI | Very low overhead and simple commands | No structured session state or provenance |
| API integration | Environmental credentials, JSON HTTP, optional search | Error string conflated with successful model answer; partial content extraction |
| Retrieval | Transparent, deterministic, explainable | Substring overlap, repeated-term bias, no semantic retrieval |
| Cache persistence | Simple s-expression format and manual save | Silent recovery failure, destructive rewrite risk, non-atomic writes |
| Context construction | Easy to inspect | No trust separation, no token budget, no freshness rule |
| Tests | Proper nonzero failure status | Only elementary ranking cases covered |

## 5. Qualification and conclusion

This is a genuine, compact **Gerbil Scheme LLM client**. It demonstrates how little infrastructure is necessary to build a CLI, call a modern LLM API, and persist a small amount of reusable context. The most instructive Gerbil code is the `:std/net/request` + JSON integration and the use of Scheme datums for local persistence.

The important engineering weaknesses are **state and evidence integrity**, not syntax or lack of language features. In particular, a read failure can silently erase accumulated memory on subsequent save, an API error can become cached context, and the retriever treats raw generated text as both searchable history and trusted prompt material.

**Recommended priority for remediation:** (1) failure-aware/atomic cache loads and saves; (2) explicit success/error result type; (3) bounded context construction; (4) token-aware retrieval with provenance; (5) end-to-end persistence and API tests. These are recommendations for the upstream implementation, not modifications performed in this review.

**Final disposition:** `SOURCE_REVIEW_COMPLETE / DATA_INTEGRITY_DEFECTS_IDENTIFIED / RUNTIME_TESTS_NOT_RUN`.