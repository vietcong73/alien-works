# 01. Glow: Gerbil Scheme Source Code Review

**Project:** [Glow-Lang/glow](https://github.com/Glow-Lang/glow)  
**Reviewed commit:** [`47753e6777ebe933be319f117c00401745eaeb2f`](https://github.com/Glow-Lang/glow/tree/47753e6777ebe933be319f117c00401745eaeb2f)  
**Review focus:** Gerbil Scheme project implementation (compiler, state/protocol analysis, EVM code generation, participant runtime, tests)  
**Method:** Static code inspection and control-flow analysis. No independent end-to-end execution was performed.  
**Disposition:** `SOURCE_ARCHITECTURE_STRONG / COMPILER_CORRECTNESS_GAPS_IDENTIFIED / RUNTIME_QUALIFICATION_NOT_ESTABLISHED`

## Purpose and scope

Glow is a language and runtime for describing multi-party decentralized applications. Its Gerbil implementation compiles a high-level interaction into participant-specific execution and EVM consensus code. This is a *project-code* review, not an audit of Gerbil Scheme itself. Unlike mere LLM integrations, its main implementation includes compiler transformations, checkpoint analysis, on-chain code generation, and off-chain participant execution.

## Architecture

`Glow source → parse/alpha-convert/desugar/typecheck/ANF → checkpoint/participant analysis → projection → EVM consensus code + participant runtime`.

The compiler uses registered passes and intermediate representation layers, rather than a single monolithic compiler function. Key source: [`compiler/passes.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/passes.ss) and [`compiler/multipass.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/multipass.ss). This is useful for transformation isolation, intermediate output inspection, and regression testing.

The checkpoint model tracks participants, transitions, publicly used/introduced variables, and persistence boundaries. [`compiler/checkpointify/checkpointify.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/checkpointify/checkpointify.ss) and [`compiler/project/project.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/project/project.ss) illustrate genuinely substantial Scheme use: macros, pattern matching, program analysis, and endpoint projection.

## Strong implementation decisions

1. **Pass-by-pass transformations:** intermediate layers are first-class and testable.
2. **Checkpoint-aware protocol representation:** the compiler identifies active participants and state needed across transitions.
3. **Endpoint projection:** one protocol is transformed into participant-specific views, reducing manual duplication.
4. **Test vectors:** `t/passdata/` and compiler tests contain source and intermediate artifacts; [`compiler/t/passes-test.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/t/passes-test.ss) exercises passes.
5. **Typed agreement and runtime objects:** `InteractionAgreement`, `AgreementHandshake`, and participant runtime are represented explicitly in Gerbil Scheme.

## Source findings (static review)

### GL-F01: Backward checkpoint-liveness propagation does not establish a fixed point

**Source:** [`compiler/liveness/checkpoint-liveness.ss:28–62`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/compiler/liveness/checkpoint-liveness.ss#L28-L62). The backward traversal accumulates live variables but excludes checkpoints present in a `seen` list. On a graph with a join and unequal predecessor path lengths, a checkpoint may acquire additional live variables after it has been processed, without re-enqueuing that checkpoint's predecessors. This is a **source-grounded algorithmic counterexample** to general fixed-point dataflow semantics. A regression case expressed in valid Glow control flow is still required to establish end-to-end reachability and impact. **High priority** because state saved between blockchain transitions depends on liveness.

### GL-F02: Shell escaping fails when agreements contain a single quote

**Source:** [`runtime/participant-runtime.ss:1246–1257`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/participant-runtime.ss#L1246-L1257). `escape-agreement-string/shell` returns the input unchanged if it contains `'`, otherwise wraps it in single quotes. The generated command is displayed by `IoChannel.send-agreement` for copy/paste by a participant ([`runtime/participant-runtime.ss:894–900`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/participant-runtime.ss#L894-L900)). The source comments themselves note the error. **Command-injection risk for untrusted input when copied into a shell**, not demonstrated automatic remote execution.

### GL-F03: Code digest is carried but not visibly verified along received-agreement path

**Source:** [`runtime/participant-runtime.ss:88–151`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/participant-runtime.ss#L88-L151) and [`cli/interaction.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/cli/interaction.ss). `InteractionAgreement` has a `code-digest`, and the runtime comments contain a TODO to check agreement/source binding. The inspected code does not establish that all participants compile a source matching the received digest. This is a **source-binding assurance gap**, not evidence of a successful financial exploit.

### GL-F04: Consensus timeout helper assumes exactly two participants

**Source:** [`runtime/consensus-code-generator.ss:249–258`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/consensus-code-generator.ss#L249-L258) and [`311–319`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/consensus-code-generator.ss#L311-L319). The compiler chooses one `other-participant` for timeout handling, explicitly noting the limitation. Sound for intended two-party paths, not a generalized multiparty implementation.

### GL-F05: Arithmetic-semantic equivalence remains unqualified

**Source:** [`runtime/consensus-code-generator.ss`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/consensus-code-generator.ss). Generated EVM integer operations and local runtime operations must agree on overflow and numerical bounds. Source TODOs identify unfinished semantics. This is a **verification gap**, not a reproduced divergent test case.

### GL-F06: Missing-function guard occurs after a dereference

**Source:** [`runtime/participant-runtime.ss:702–714`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/participant-runtime.ss#L702-L714). The code extracts `small-function.arguments` before checking `unless small-function`. For a missing function, an object access may fail before the intended `Unknown function` error.

### GL-F07: libp2p subsystem is disabled in this snapshot

**Source:** [`runtime/participant-runtime.ss:1039–1040`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/participant-runtime.ss#L1039-L1040) and [`1220–1240`](https://github.com/Glow-Lang/glow/blob/47753e6777ebe933be319f117c00401745eaeb2f/runtime/participant-runtime.ss#L1220-L1240). `Libp2pChannel` and its selection path are disabled. Working IO and TCP channel implementations exist separately; this is **an incomplete networking backend**, not a claim that all off-chain communication is absent.

## Verdict

The compiler pipeline, checkpoint-based protocol model, and endpoint projection are strong examples of what a substantive project can implement in Gerbil Scheme. Weaknesses concern dataflow fixed-point correctness, source/agreements binding, arithmetic semantics, and incomplete multiparty/networking support. **No blanket safety or production qualification is established.** The review is source-based and read-only.