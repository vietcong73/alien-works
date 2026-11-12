# 03. GloUI: Gerbil Scheme Source Code Audit (Revised)

**Project:** [Glow-Lang/gloui](https://github.com/Glow-Lang/gloui)  
**Reviewed commit:** [`bac31fab2ff8394cf8d5e727df49baf2bb3b66a1`](https://github.com/Glow-Lang/gloui/tree/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1), September 9, 2021  
**Review focus:** First-party Gerbil Scheme (`.ss`) including HTTP endpoints, capability dispatch, transaction supervision, error handling, object macros, and code generated/embedded by Scheme. **Standalone Vue/Quasar/JavaScript defects are excluded.**  
**Method:** Direct static source inspection and call-path tracing. A reduced JavaScript semantic reproduction was separately run for the Gerbil-embedded entity model; full application execution was not performed.  
**Disposition:** `GERBIL_SOURCE_REVIEW_COMPLETE / CODE_DEFECTS_IDENTIFIED / INTEROP_SEMANTICS_REPRODUCED / INTEGRATION_UNVERIFIED`

## Architecture and positive design elements

GloUI is a hybrid application using **Gerbil Scheme for HTTP handling and blockchain transaction orchestration** with a separate Vue/Quasar frontend. The main code path is `server/ftw.ss` → `server.ss` / `server/erc20.ss` → `server/ethereum.ss` → `server/ethereum/{balance,process,erc20}.ss` → external Ethereum implementation.

The best Scheme abstraction is **capability-based dispatch**. `server/ethereum/process.ss` uses a hash-table registry keyed by resource type, and `server/ethereum/balance.ss` provides a parallel balance-handler registry. ERC-20 handlers are installed in `server/ethereum/erc20.ss`. This is modular and extensible without requiring a large conditional handler. The weakness is the absence of an enforced handler-result/error contract; an absent registry key may lead to calling `#f`.

The code also exhibits `try/finally`-based cleanup around keypair registration, explicit transaction records, Gambit threads for asynchronous processing, and Scheme macros for the JSON endpoint DSL (`define-json-endpoint`). `entity.ss` and `js/entity.ss` show a separate entity-object DSL and a Gerbil-to-JavaScript compilation path.

## Source-grounded findings

### GLO-S01: Invalid transfer leads to an invalid process dereference

**Source:** [`server/ethereum.ss:194–218`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/ethereum.ss#L194-L218) and [`302–311`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/ethereum.ss#L302-L311). On insufficient balance, `process-transfer!` sets `.error` without starting a process (`.process` remains `#f`). `process-transaction!` unconditionally accesses `(.@ tran process thread)` and calls `thread-join!`. This is a **control-flow defect** rather than a graceful failed-transfer result. Note also that `valid-transfer?` uses strict `<` rather than `<=`; spending the exact available balance is rejected by the local check, though fee handling can require additional funds.

### GLO-S02: ERC-20 scaling is hardcoded

**Source:** [`server/erc20.ss:53–66`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/erc20.ss#L53-L66). The endpoint computes `amount * 10^17`, not `amount * 10^token_decimals`. For an 18-decimal token, requesting 1 yields 0.1 token in base units. The general `make-transfer` path also derives `value` from the chain's `nativeCurrency`, requiring careful review for ERC-20 unit semantics.

### GLO-S03: ERC-20 balance handler deliberately fails

**Source:** [`server/erc20.ss:32–38`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/erc20.ss#L32-L38). `erc20-balance/POST` unconditionally evaluates `(error "ADdress" address)` instead of returning a balance. This appears to be leftover diagnostic code and has no successful route through the handler.

### GLO-S04: Error output stored in wrong field

**Source:** [`server/ethereum.ss:172–179`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/ethereum.ss#L172-L179). `transfer-process-error-output` obtains the error stream and `errorString` but writes the result to `.outputString`. This is a direct field-selection error. No separate call site was established in the inspected repository, so demonstrated runtime impact is limited.

### GLO-S05: Thread state is misspelled in dispatch

**Source:** [`server/process.ss:34–52`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/process.ss#L34-L52). The producer yields `'normally-terminated`, while the `case` branch expects `'normally-terminataed`; that branch cannot match a normal termination state.

### GLO-S06: Empty transaction violates supervisor precondition

**Source:** [`server/ethereum.ss:293–325`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/ethereum.ss#L293-L325). Empty `assets` creates an empty transfer list. The `process-transaction!` loop immediately uses `(car trans)` and `transaction-state` also indexes the list without guarding an empty transaction. The request needs validation before starting the worker.

### GLO-S07: Broad HTTP exposure at the execution boundary

**Source:** [`server/ftw.ss:5–37`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/ftw.ss#L5-L37). Default bind is `0.0.0.0:6741`; CORS reflects the request's Origin and returns `Access-Control-Allow-Credentials: true`. The inspected transaction route accepts JSON and can initiate financial operations; no explicit per-handler authorization appears in the inspected code. This is a **conditional security design risk**, not a proven remote exploit, since deployment and external library protections have not been established.

### GLO-S08: Keypair registry needs concurrency qualification

**Source:** [`server/ethereum.ss:92–125`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/ethereum.ss#L92-L125). The source correctly uses `try/finally` to unregister the named keypair after an operation. Overlapping threads using the same key name may interact depending on the external key registry's semantics. **Risk only**, not a demonstrated race.

### GLO-S09: Process and transaction records have no visible eviction

**Source:** [`server/process.ss:9–32`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/process.ss#L9-L32) and [`server/ethereum.ss:344–368`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/server/ethereum.ss#L344-L368). Both maintain mutable hash tables keyed by UUID/transaction ID. Repository search did not reveal corresponding eviction operations, so completed records may accumulate during server lifetime. The scale and impact are unmeasured.

## Interoperability-specific finding (Scheme-owned embedded JavaScript)

### GLO-I01: Shared prototype function causes cross-instance entity state corruption

**Source:** [`src/assets/Entity.ss:4–77`](https://github.com/Glow-Lang/gloui/blob/bac31fab2ff8394cf8d5e727df49baf2bb3b66a1/src/assets/Entity.ss#L4-L77). This JavaScript is embedded with Gerbil `js#decl`, so the fault belongs to the Scheme-defined interop model rather than an unrelated Vue file. The constructor sets `this.$entity.self = this`, but `$entity` is a function shared via prototype. Its `JSON` getter serializes `this.self`, allowing constructing entity B to alter serialization of A. A reduced Node.js reproduction of these exact expressions yielded `A.JSON={"name":"Alice"}` before construction of B and `A.JSON={"name":"Bob"}` afterward. The reduced test demonstrates JavaScript object semantics, not end-to-end app execution.

## Review disposition

**Strengths:** cohesive small Scheme modules, extensible asset-type dispatch, endpoint macros, asynchronous execution machinery, `try/finally` for resource cleanup, and genuine Scheme-to-JavaScript metaprogramming.

**Weaknesses:** unvalidated state-machine preconditions, unit-conversion inconsistencies, weak failure propagation, mutable shared registries without established lifecycle contracts, broad HTTP authority, and interop object-state leakage.

The main conclusion is **local modularity is better than end-to-end contract coherence**. GloUI is a valuable *Gerbil application engineering case study*, not a qualified production reference for financial transactions. Independent `.vue`/Quasar bugs and frontend polling findings from the earlier broader review are explicitly excluded.