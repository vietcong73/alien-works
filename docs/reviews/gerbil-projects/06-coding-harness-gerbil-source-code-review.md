# 06. Coding Harness: Gerbil Scheme Source Code Audit

**Repository:** [mark-watson/gerbil_scheme_book/source_code/coding-harness](https://github.com/mark-watson/gerbil_scheme_book/tree/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness)  
**Pinned revision:** `1335cb2708926213ba62afd2998754358f655db1` (November 11, 2026)  
**Review date:** November 12, 2026  
**Scope:** Gerbil implementation of the model/tool loop, file and command tools, edit approval, language dispatch, syntax checker, subprocesses, and tests. This is not a review of the supported programming languages or LLM model quality.  
**Method:** Static source-code review using direct access to function bodies. No `gxi` execution, live model call, or full integration test. Severity reflects potential consequences within the intended local-agent operating model.  
**Disposition:** `STATIC_SOURCE_AUDITED / LOCAL_AUTHORITY_BOUNDARIES_IDENTIFIED / CHECK_RESULT_SEMANTICS_WEAK / RUNTIME_QUALIFICATION_PENDING`.

## Implementation profile and design

This is an actual coding agent written in Gerbil, not a simple chat API wrapper. It supports multi-turn tool calls, per-language evaluation and syntax checking, proposed source edits, human diff approval, post-edit checks, and a syntax-repair handoff. The modules have clear responsibilities:

| Module | Implementation responsibility |
|---|---|
| [`harness.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/harness.ss) | REPL, conversation state, model loop, compacting, exit statuses |
| [`llm.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/llm.ss) | OpenAI-compatible requests, iteration cap, tool-call repetition detection |
| [`tools.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/tools.ss) | Tool registry, file access, eval, command execution, proposed edits |
| [`checker.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/checker.ss) | Build/check gate, syntax diagnostics, user-repair loop |
| [`lang.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/lang.ss) | Language profiles, per-file checks, command prefixes |
| [`parens.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/parens.ss) | Multi-lexicon delimiter scanner and diagnostics |
| [`approval.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/approval.ss) | Unified diff and human approval interface |
| [`util.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/util.ss) | argv-based subprocesses, filesystem helpers |

**Good design features:** Tool schemas and dispatch are centralized; the model's `propose_edit` tool performs stale-base matching and displays a diff; the checker separates read/syntax failures from other build failures. `run-command` uses `open-process` with explicit `path` and `arguments` rather than passing a command string to a shell. Thus shell metacharacter injection should **not** be alleged solely from `run_shell` input. The delimiter scanner is sophisticated enough to warrant separate tests. The language profile allows reuse beyond Gerbil while remaining implemented in Gerbil.

## Findings

### CH-01 — No canonical project-root boundary in filesystem tools (high priority for unattended agents)

[`tools.ss:134–150`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/tools.ss#L134-L150) checks whether a basename is hidden and whether the path exists. It does **not** resolve and constrain the target to the project root. For instance, a readable absolute path outside the project and a relative path containing `../` can pass the read tool's visible guards if their final basename is not hidden. [`tools.ss:299–354`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/tools.ss#L299-L354) similarly lacks root containment checks before an approved/auto-approved write. `list_dir` and `grep` likewise do not establish project isolation. This is **scope/authority exposure**, not a claim of operating-system privilege escalation: access still depends on the process account and editing generally requires approval except under `-y`.

A sibling issue occurs in [`checker.ss:131–132`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/checker.ss#L131-L132): the function `inside-project?` uses string-prefix comparison on expanded paths. A path under `/work/project-copy/` can share the prefix `/work/project` while not being a descendant. This function only gates diagnostic source excerpts, but illustrates the boundary problem.

**Needed validation:** allow-only project-root paths with canonicalization and symlink resolution; reject absolute/outside paths before touching disk; test traversal, sibling-prefix and symlink escapes.

### CH-02 — Auto-approval collapses manual execution gates (intentional but high-impact)

[`tools.ss:288–294`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/tools.ss#L288-L294) allows `eval_code` when `(auto-approve?)`, bypassing the prompt. [`tools.ss:320–344`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/tools.ss#L320-L344) similarly auto-applies proposed edits (with an extra delimiter guard for strict languages). Because `eval_code` executes user-provided/model-proposed source with local process rights, auto-approval is **not** a sandbox. This is a documented mode choice, not an accidentally missing approval prompt; it needs an explicit deployment trust policy.

### CH-03 — Failed checks are not generally transactional (medium)

[`tools.ss:346–354`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/tools.ss#L346-L354) writes the new file and only then checks. [`checker.ss:220–255`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/checker.ss#L220-L255) can return `'failed`, `'handed-to-model`, or `'aborted` without invoking the revert thunk. Revert is an explicit option in one interactive syntax-error branch. Consequently, "check failed" does not imply "filesystem restored"; this may be useful for iterative repair, but the result should not be represented as an atomic validated edit. In addition, external effects of a build/test command are outside the single-file revert mechanism.

### CH-04 — One-shot exit status is inferred from historical output text (medium, code-confirmed)

[`harness.ss:372–395`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/harness.ss#L372-L395) scans **all** recorded tool messages for substrings `SYNTAX ERROR`, `FAILED`, `user rejected`, etc. A failure that has subsequently been corrected can remain in conversation history and still produce a failure exit code. Conversely, operational failures expressed with other wording need not match. This is a direct mismatch between **structured execution result** and **natural-language history search**; a last-gate outcome record would be more robust.

### CH-05 — Timeout is not an absolute execution deadline (conditional)

[`util.ss:54–67`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/util.ss#L54-L67) attaches an input-port timeout, then waits for process status. This protects against inactivity or completion stalls according to the port/runtime semantics, not necessarily a monotonic wall-clock deadline. A continuously producing process is a counterexample to treating an idle timeout as a hard total runtime budget. In addition, the visible cleanup sends a signal to a PID, not a managed process tree. Runtime behavior must be tested with processes that spawn descendants and continuously write output.

### CH-06 — Repeated tool calls are treated as stalled even when repeat might be meaningful (design tradeoff)

[`llm.ss:170–220`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/llm.ss#L170-L220) terminates the agent loop on an identical tool-call signature found in a recent window. This is a useful runaway defense but can reject legitimate repeated read/poll operations. This is a **policy tradeoff**, not a confirmed implementation bug.

## Tests and limitations

[`test/parens-test.ss`](https://github.com/mark-watson/gerbil_scheme_book/blob/1335cb2708926213ba62afd2998754358f655db1/source_code/coding-harness/test/parens-test.ss) contains actual assertions for delimiters and syntax categorization. It is not evidence that root containment, approval paths, failed-check recovery, model/tool protocol sequencing, or process lifecycle were integration-tested. No independent native Gerbil execution was performed in this review.

## Source-code disposition

**Study/reuse:** Tool registration, per-language dispatch, argv-based subprocess invocation, stale-base compare, diff display, and rich syntax diagnostics.  
**Do not assume without remediation:** Project-root containment, read-only operation under general code eval, atomic post-check edits, structured one-shot result semantics, hard resource limits.  
**Overall:** Substantial Gerbil application code with useful practical engineering. The major concern is **authority governance**, not whether Scheme is capable of hosting an agent.