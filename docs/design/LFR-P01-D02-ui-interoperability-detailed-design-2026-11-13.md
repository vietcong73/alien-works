# LFR-P01-D02 | Lisp Metaprogramming & Interoperability / Typed UI & Interoperability

**Document class:** Detailed design proposal, one specialist edition  
**Date:** 13 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Edition:** P01 — Lisp Metaprogramming & Interoperability  
**Design slice:** D02 — Typed UI & Interoperability  
**Implementation target:** Independent Gerbil Scheme; transport-neutral core with optional HTTP and JavaScript bindings  
**Primary source:** `Glow-Lang/gloui@bac31fab2ff8394cf8d5e727df49baf2bb3b66a1`  
**Source review:** `docs/reviews/gerbil-projects/03-gloui-gerbil-source-code-review.md` in `vietcong73/alien-works@audit/lisp-foundation-reserve`  
**Status:** `DETAILED_DESIGN_PROPOSED / DIRECT_SOURCE_GROUNDED / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Scope isolation:** No modifications to source specimens, LFR-C02, P01-D01–P03-D02, their designs, or their runtime. Documentation publication is owner-approved; runtime implementation still requires separate authorization.

---

## 0. Decision and useful product

Create a **Gerbil-native, contract-driven UI interoperability reserve** for applications that expose typed entities, read-only projections, asynchronous work, and specifically authorized operations to an interface. P01-D02 is **not** a UI framework, an Ethereum wallet, a React/Vue replacement, a full-stack web application generator, a general scheduler, or an authority server. It is the reliable seam between application semantics and an external presentation environment.

The target product is reusable by a research interface, local simulator, desktop tool, diagnostic dashboard, workflow client, or a web app without installing blockchain software, Quasar/Vue, or any mandatory JavaScript runtime.

Three independently testable, jointly required capabilities for a **full P01-D02 reserve**:

- **P01-D02-A — Typed Boundary & Entity Identity:** schema-versioned message envelopes, checked serialization/deserialization, entity-scoped projections, exact typed errors and unit-aware numeric values.
- **P01-D02-B — Async Task Presentation:** request acceptance separated from execution and effect outcome, durable-optional task observations, sequencing, cancellation acknowledgments, bounded subscriptions, client reconciliation and per-entity isolation.
- **P01-D02-C — Governed Code–Data Bridge:** declarative Gerbil interface specifications lowered to validated, version-pinned intermediate representation (IR), producing protocol validators and opt-in target bindings with parity tests; no trusted code execution from untrusted serialized declarations.

A and B may be imported independently in limited deployments. A complete P01-D02 release requires A, B, and C, with at least one **qualified end-to-end adapter pair** (Gerbil process-to-transport-to-client reference), not merely isolated macro expansion tests. An effectful adapter additionally requires the independent authority gate. A pure read-only consumer need not have an effectful adapter.

**Falsifiable thesis:** given the same validated source entity, revision, and authorized observable, all admitted P01-D02 adapters produce equivalent user-visible claims, preserve distinct entity identities, reject malformed or unauthorized commands before their effectful handler, and never reclassify an accepted/unknown task as a completed success.

## 1. Research lineage and evidentiary boundaries

Classification: `S` directly inspected pinned code; `R` static review finding; `E` separately reproduced interop semantics; `D` design response, not implemented; `X` external standard.

| ID | Source evidence | Useful mechanism | Limitation or defect and strength | P01-D02 design response | Regression obligation |
|---|---|---|---|---|---|
| GLO-01 | `server/ftw.ss:36–47` (`define-json-endpoint`) | Scheme macro can express HTTP endpoint registration concisely (S) | Generated endpoint declaration does not itself enforce input, output, auth and effect contracts (S/D) | `define-interface` declares typed operation, schema, outcome and authority profile; static IR validation before adapter codegen | Generated route cannot bypass type/effect checks |
| GLO-02 | `server/ethereum/process.ss:25–32`; `server/ethereum/balance.ss` | Resource-keyed handler registry makes operations extensible (S) | Unregistered type can yield `#f` invoked as a function (S/R) | Typed `HandlerDescriptor`, registration collision checks, lookup returns tagged `UNSUPPORTED_CAPABILITY` | Unknown type never invokes handler |
| GLO-03 | `server/process.ss:9–63` | Async worker IDs, process status, collected output (S) | Normal exit symbol misspelled in `case`; no observed eviction logic (S/R) | Closed task-state ADT and explicit bounded retention; type-checked dispatch | Normal termination maps to one success outcome; terminal task cleanup |
| GLO-04 | `server/ethereum.ss:194–218,302–311` | Transfer planning and worker execution separated in part (S) | Rejected transfer lacks worker but later code dereferences worker thread (S/R) | Rejection terminal before worker creation; worker handle optional and guarded | Rejection never dereferences missing process |
| GLO-05 | `server/ethereum.ss:293–325` | Multi-operation records and async transaction coordinator (S) | Empty list reaches `car` and unguarded current index (S/R) | Validate cardinality and domain constraints before task admission | Empty/oversized work list rejected without a worker |
| GLO-06 | `server/ethereum.ss:172–179` | Distinct output and error streams (S) | Error stream appended to `outputString`, not `errorString` (S/R) | Typed stdout/stderr/error slots and output-size quotas | Independent stream test catches field swaps |
| GLO-07 | `server/erc20.ss:32–38` | Specialized endpoint extension (S) | Handler unconditionally raises a diagnostic error (S/R) | Contracts distinguish `NOT_IMPLEMENTED`, recoverable failure and success | No handler may serialize a diagnostic exception as successful data |
| GLO-08 | `server/erc20.ss:53–66` | Explicit conversion at a boundary (S) | Hard-coded `10^17`; asset decimals are not specified by boundary (S/R) | Unit-qualified integers/decimal-string wire values and domain-owned scaling | Different scales, very large values, fractions and overflow rejected or exact |
| GLO-09 | `server/ftw.ss:5–37` | Simple deployable HTTP server and CORS handling (S) | Binds `0.0.0.0` and reflects Origin with credentials; inspected effect routes lack visible per-handler authorization (S/R; risk, not exploit proof) | Secure deployment profile, origin allowlist, authN/authZ, CSRF policy, deny-by-default effects | Unauthorized cross-origin and cross-principal requests refused |
| GLO-10 | `server/ethereum.ss:92–125` | `try/finally` unregisters sensitive resources (S) | Concurrent registry use may conflict depending on external implementation (R; unproven runtime race) | Scoped resource-owner, no globally shared mutable credential handle, concurrency tests | Parallel operations cannot cross-use handles |
| GLO-11 | `server/process.ss:9–32`; `server/ethereum.ss:344–368` | Task/transaction lookup by ID (S) | No visible eviction from in-memory registries (R; unmeasured accumulation) | TTL and quota-aware task index; tombstone/expired semantics | Saturation, cleanup and lookup-after-eviction tests |
| GLO-12 | `src/assets/Entity.ss:4–77` with `js#decl` | Scheme-owned JavaScript entity DSL and serialization (S) | `this.$entity.self=this` mutates prototype-shared function state (E: reduced JS reproduction; no full-app run) | Value-only normalized payloads and per-instance metadata; generated bindings cannot hold entity identity on shared prototype function | Interleaved A/B and concurrent serialization isolation tests |
| GLO-13 | `js/entity.ss:1–41` (`define-entity`) | Gerbil macros generate JavaScript constructors/interop declarations (S) | Expansion lacks independently checked IR/schema parity; bespoke embedded JS complicates audit (D) | Staged pipeline `Gerbil DSL -> normalized IR -> validators/adapters`; golden parity | Source/IR/compiler digest binding and cross-target fixture parity |

**Do not enlarge the evidence:** Source concerns are from static review except the reduced JavaScript object-identity reproduction. No live Gerbil/GloUI end-to-end test, transaction exploit, worker race, or current-production safety assertion is claimed. Standalone `.vue` or unrelated Quasar defects are deliberately outside the lineage scope.

### External standards and reference semantics

- Gerbil guide, hygienic `syntax-case`, contracts and web/JSON facilities: https://gerbil.scheme.org/guide/intro.html
- Gerbil custom language/prelude and staged expansion: https://gerbil.scheme.org/tutorials/languages.html
- HTTP semantics including `202 Accepted`: https://www.rfc-editor.org/rfc/rfc9110.html
- RFC 9457, `application/problem+json`: https://www.rfc-editor.org/rfc/rfc9457.html
- JSON Schema Draft 2020-12 vocabulary for compatible validation adapters: https://json-schema.org/draft/2020-12/
- OWASP API Security Top 10 for object-level/function-level authorization and resource-consumption threats: https://owasp.org/API-Security/

The HTTP and JSON Schema standards inform **optional adapters**, not the ontology of a mandatory network server. P01-D02's core `Result`/`Failure` algebra is transport-neutral; an HTTP adapter maps failures to suitable status and optionally RFC 9457 problem documents. A JSON Schema document does not, by itself, certify authorization or business semantics.

## 2. System boundaries and source-derived design principles

**Core invariant:** an interface is an observation-and-command boundary, **not** the source of truth for the domain entity or the authority to execute an effect.

| Boundary | P01-D02 owns | Explicitly outside P01-D02 |
|---|---|---|
| Domain entities | Validated identity references, projections, schema versions and revision checks | Domain calculations, object truth and database transactions |
| UI | Data/action contracts, per-entity view model, client reconciliation | Widget toolkit, layout engine, browser renderer, native GUI rendering |
| Commands | Validation, classification, authorization-gate invocation, typed dispatch and task receipt | General policy governance, signing secrets, privileged effect implementation |
| Async work | Task observation protocol and bounded local reference dispatcher | Universal job scheduler, distributed queue, process tree supervisor |
| Metaprogramming | Compile-time declarations -> typed IR -> validators and bindings | Arbitrary dynamic eval of external declarations, untrusted compiler plug-ins |
| Persistence | Retention protocol, task recovery claims and receipts | Mandatory database, P04-D01 durable storage implementation |
| Cross-edition | Explicit adapters/contracts only | P02-D02 authority engine, P04-D01 provenance DB, P03-D02 branch/synthesis implementation |

**Negative transfer:** no `:mukn/ethereum`, blockchain address, ERC-20 decimals, wallet, keypair, Quasar, Vue, browser `window` object, native HTTP daemon, or event-source transport in the P01-D02 core.

## 3. Architecture: describe -> validate -> generate -> operate -> observe

```text
        GERBIL AUTHORING PLANE  (trusted source descriptions)
  InterfaceSpec / EntitySpec / ActionSpec / ViewSpec / EffectSpec
                         |
                normalize / typecheck
                         v
             VERSION-PINNED CONTRACT IR
          schema + identity + effects + lifecycle
          + method/capability + projection + limits
                /           |              \
               v            v               v
        Validators      Gerbil codecs    Client binding metadata
               \            |               /
                \           v              /
              QUALIFIED ADAPTER PROFILE (optional)
             local IPC | HTTP/JSON | JS client | native client
                              |
                    REQUEST ADMISSION GATE
    parse -> authenticate -> entity-scope -> validate -> authorize
             -> dedup/preconditions -> invoke owned handler
                              |
                TASK / OBSERVATION JOURNAL
             typed outcome + monotone event sequence
                              |
                PER-ENTITY VIEW RECONCILER
          snapshot + events + stale/error/unknown status
                              |
                APP-SUPPLIED RENDERER / CONTROLLER
```

The declarative IR is **data to inspect**, not executable authority. The only way it becomes an operation is through verified generated adapter code and a separately installed qualified handler with an authorization boundary. Source and generated bindings carry a revision lock; a changed macro expander, compiler, schema or capability registry revision cannot silently inherit an old qualification receipt.

### Source topology (proposed, not created)

```text
editions/interface-interop/
  gerbil.pkg
  README.md
  lineage/source-pins.md
  lineage/defect-register.md
  src/spec/{entity,interface,operation,projection,effect,limits}.ss
  src/ir/{normalize,validate,canonicalize,identity,provenance}.ss
  src/wire/{envelope,codec,number,version,error,receipt}.ss
  src/entity/{ref,projection,ownership,isolation,revision}.ss
  src/task/{state,event,dispatcher,observe,retention,reconcile}.ss
  src/security/{principal,scope,precondition,gate,idempotency}.ss
  src/generate/{gerbil,openapi,js-metadata,fixtures}.ss
  adapters/{in-process,http-json,js-client,storage-protocol}.ss
  tests/{spec,wire,identity,task,authority,generation,adapter,faults}/
```

## 4. Type system: identities, messages and transport contracts

### 4.1 Entity identity and observation scope

```text
EntityRef {
  domain_id, entity_type, entity_id, identity_generation,
  tenant_or_scope, owner_visibility_policy, schema_revision
}

ProjectionRef {
  entity_ref, projection_id, projection_revision,
  observable_fields, redaction_policy_digest
}

EntityObservation {
  projection_ref, entity_revision, observed_at, sequence,
  value, consistency_class, completeness, evidence_ref_optional
}
```

- Two objects with equal display names or equal JSON payloads are **not** equal entities.
- The same entity projected for two principals may disclose different fields and **must not share** a permissive cache entry.
- `identity_generation` prevents reuse of a deleted object identifier from reviving stale UI state.
- A client must not infer continued authorization from a cached observation after a scope or principal change.
- Projections only expose explicit allowlisted fields; no automatic `Object.assign` of server-controlled or untrusted fields onto privileged instances.

### 4.2 The message envelope

```text
Envelope {
  protocol_id, protocol_revision, message_kind,
  message_id, correlation_id, causation_id_optional,
  principal_scope_ref, entity_ref_optional, operation_ref_optional,
  schema_digest, payload, sent_at_optional, extensions
}
```

`message_kind` is one of `QUERY`, `QUERY_RESULT`, `COMMAND`, `COMMAND_RECEIPT`, `TASK_EVENT`, `ERROR`, `SUBSCRIBE`, `UNSUBSCRIBE`, `CANCEL_REQUEST`, `CANCEL_ACK`. An envelope is validated **before** dispatch, and its principal is established from the trusted transport/session context, not from a forgeable JSON field. `principal_scope_ref` in the envelope is a *binding* for cross-checking, not an authentication credential.

- Reject malformed JSON, depth/size overflows, duplicate ambiguous fields, wrong enum tags, invalid UTF-8, schema digests not admitted, and unexpected protocol major versions.
- Explicit compatibility profile controls unknown optional fields; never silently accept unknown fields that might expand permissions, select handlers or alter amounts.
- For sensitive values and large integers, use canonical **decimal strings or integer minor-unit records**, never implicitly round through JavaScript IEEE-754 Number.
- Number conversions specify `unit_id`, `scale`, `rounding_policy`, `source_precision`, `range` and whether fractional input is admitted. Asset decimal parameters belong to the domain adapter.
- Every cross-language message has an independent read/write and normalized round-trip oracle. Object equality is defined by canonical fields, not a single hash.

### 4.3 Failure and receipt algebra

```text
Result<T> = Ok<T, Receipt> | Err<Failure>

Failure {
  code, class, message_public, diagnostics_private_ref_optional,
  retryability, request_ref, effect_state,
  violated_contracts[], correlation_id
}

EffectState = NOT_ATTEMPTED | ADMITTED_NOT_STARTED | IN_PROGRESS |
              CONFIRMED_SUCCEEDED | CONFIRMED_FAILED |
              UNKNOWN_AFTER_DISPATCH
```

`Err` is not a successful `Ok` containing an error string. Diagnostic exceptions, stack traces, secrets, and internal handles never become public `message_public`. `FAILED` alone does not imply an external effect was reversed; `UNKNOWN_AFTER_DISPATCH` is preserved when the outcome cannot be established. A standard HTTP adapter maps its problem details according to RFC 9457 and keeps the actual HTTP status consistent with any body `status` field.

## 5. Async semantics: separate job truth from client observation

### 5.1 Task truth state machine

```text
VALIDATING --reject--> REJECTED (no worker)
    |
    v
ADMITTED --> QUEUED --> RUNNING --> SUCCEEDED
                |          |    \--> FAILED
                |          \-----> CANCEL_REQUESTED
                |                        |
                +------------------------+--> CANCELLED (only if verified)
                                         \--> FAILED / SUCCEEDED /
                                              UNKNOWN_AFTER_DISPATCH
```

The diagram describes **allowed categories**, not every transition. The actual allowed-transition table is checked as a total relation with explicit error handling. Cancellation is a **request**, not a rollback or proof of nonexecution. A terminal `SUCCEEDED` requires an independently defined completion/effect witness; a worker thread's normal termination is insufficient when the external operation has unknown status. If a transport drops after dispatch, do not automatically replay a side-effecting request without an idempotency contract.

Task truth comprises `TaskId`, `OwnerPrincipal`, `OperationRef`, `EntityRef`, `AcceptedRevision`, `TaskState`, `EffectState`, `Sequence`, `CreatedAt`, `Deadline`, `Receipt`, and `RetentionRef`. Each observed update has a monotone `(task_id, generation, sequence)` and a typed state transition proof. Invalid transitions are rejected, not silently patched on the UI.

### 5.2 View observation is an independent axis

```text
ViewObservation = FRESH | STALE | DISCONNECTED | RECONNECTING |
                  PARTIAL | FORBIDDEN | UNVERIFIED
```

The UI may show **`RUNNING / DISCONNECTED`**, **`SUCCEEDED / STALE`**, or **`UNKNOWN_AFTER_DISPATCH / RECONNECTING`** without pretending task truth changed when the client lost connectivity. A cache or event stream is not the authoritative worker state. A reconnect obtains the latest qualified snapshot, then replays only events with strictly newer sequence numbers, checking for gaps.

Per-entity state is indexed by the full identity and principal projection scope. Switching entities must not allow an in-flight response for the previous entity to paint the new entity's screen. Terminal task observations must not be overwritten by delayed earlier nonterminal events.

### 5.3 HTTP/transport profiles (opt-in)

- `202 Accepted` means a request was accepted for processing, **not completed**. The response contains task identity and a status-monitor reference when applicable.
- `201 Created` may be appropriate for a created resource when that resource creation has actually completed; `200 OK` for completed synchronous responses; error codes map to typed failures.
- Event delivery may use polling first. SSE/WebSocket are possible adapters, not mandatory requirements; all must share the same typed event and reconciliation semantics.
- Subscription protocols declare bounded replay window, cursor semantics, page limits, backpressure, slow-consumer policy and terminal tombstones.
- Task registry never grows without bound. Eviction requires TTL, memory/byte quotas, pinned outstanding observations and explicit `TASK_EXPIRED` / `HISTORY_UNAVAILABLE` results; never fabricate missing history.
- Crash durability requires a separately qualified storage adapter. P01-D02 in-memory mode cannot claim that tasks are recoverable after a process exits.

## 6. Security and effectful operation admission

The UI is an **untrusted requester**. Disabled buttons, hidden routes, localStorage flags, schema-valid JSON, same-origin controls and a DSL declaration do not confer authorization.

**Admission sequence:**

1. Authenticate the request at the actual channel/session; reject absent/untrusted identity.
2. Bind the principal to `EntityRef`, domain, tenant/scope and operation; enforce object-level access.
3. Parse and validate envelope, schema revision, capabilities, units and quotas.
4. Check operation-specific preconditions against an authoritative source with explicit stale revision handling.
5. Obtain independently supplied authorization decision for `EffectDescriptor`; default deny for mutation, process, network and financial operations.
6. Bind idempotency key to principal, operation, canonical payload and precondition revision. Replays return the prior receipt or typed conflict, not an unverified second effect.
7. Execute only through a registered handler with an allowlisted effect scope; record acknowledgment, evidence and outcome separately.
8. Revalidate permissions and safety at dispatch time for queued actions if policy could have changed after admission. Never silently promote read-only permission into mutation permission.

A policy decision provider may be supplied by P02-D02 or an application-defined gate; **P01-D02 always enforces its own admission contract** even when P02-D02 is absent. Neither a generated policy macro nor a client-provided capability tag may approve itself.

For optional HTTP, the default listener is loopback-only, and any non-loopback exposure needs an explicitly qualified deployment profile with authentication, TLS as applicable, origin allowlist, CSRF defense for cookie-authenticated mutating routes, request quotas and logs. CORS only governs browser-origin sharing and is not authentication. An effectful route cannot be activated merely by importing an endpoint module. Do not echo secrets into error messages or task output.

## 7. Gerbil metaprogramming: code and data without authority confusion

**P01-D02-C uses code–data duality as the engine of interoperability**, not as an excuse to run arbitrary code embedded in a message.

```scheme
;; Declarative design notation: NOT an implemented Gerbil macro.
(define-interface diagnostic-console
  (revision "diag.v1")
  (entity job
    (identity job-id generation owner-scope)
    (projection status (fields state progress updated-at)))
  (query read-job
    (input JobLookup)
    (output JobView)
    (effect READ))
  (command retry-job
    (input RetryRequest)
    (output TaskReceipt)
    (effect MUTATION)
    (authority retry-job-policy)
    (idempotency REQUIRED)))
```

### Staging and generation pipeline

```text
Trusted Gerbil source + pinned expander/compiler
     -> hygienic expansion and parse-only declarative AST
     -> normalized typed IR (no implicit handlers)
     -> static checks (entity identity, fields, effect class,
                       schema closure, operation ownership, quotas)
     -> stable IR digest and capability registry revision
     -> generated Gerbil codec/validator and client metadata
     -> independent fixture evaluation / schema-parity checks
     -> deployment admission of a qualified adapter
```

A generated JS binding is an **adapter** and may be plain JavaScript/TypeScript metadata; its source is not the foundation's runtime. Every entity instance owns its own data and serialization view. No function object shared on a prototype carries `self`, current selection, tenant or task mutable state. Generated JS must not use `eval`, `Function`, implicit code loading, or prototype keys controlled by wire payloads.

### Compiler/DSL invariants

- `G1`: source and normalized IR identities are revision pinned; friendly version strings are not enough.
- `G2`: schema, effect, authority and entity ownership metadata survive lowering and serialization.
- `G3`: generated serializer and parser are round-trip compatible for admitted values and reject malformed/over-range values.
- `G4`: macro expansion cannot silently turn a read-only operation into an effectful handler.
- `G5`: two client targets agree on canonical fixtures or the adapter remains unqualified.
- `G6`: generated code cannot self-register privileged handlers; deployment registry admits them separately.
- `G7`: old client schema against new server schema follows declared compatibility policy, never a guess.
- `G8`: changes to the compiler/expander, external schema vocabulary or canonicalizer invalidate affected qualification receipts until rechecked.

## 8. Exact interfaces and ownership contracts (proposed)

```text
register_interface(spec, source_lock) -> Result<InterfaceRevision>
normalize_interface(spec) -> Result<TypedIR>
compile_adapter(typed_ir, target_profile) -> Result<GeneratedArtifact>
validate_envelope(envelope, negotiated_contract) -> Result<ValidatedMessage>
project_entity(entity, principal, projection_ref) -> Result<EntityObservation>
register_handler(op_ref, signature, effect_descriptor, handler) -> Result<Handle>
invoke_query(request, authority_context) -> Result<Observation>
submit_command(request, admission_context) -> Result<TaskReceipt>
observe_task(task_ref, principal, last_sequence) -> Result<TaskObservation>
request_cancel(task_ref, principal) -> Result<CancelRequestReceipt>
reconcile_view(view_ref, snapshot, ordered_events) -> Result<ViewState>
expire_task(task_ref, retention_policy) -> Result<RetireReceipt>
```

The `register_handler` interface refuses duplicate or conflicting operation revisions. Every handler owns an explicit lifetime; deregistration while tasks are running has a typed drain/refusal policy. Reconciliation never calls the effectful handler. Observations and command outcomes are separate types, so an observation cannot accidentally be promoted into a new mutation command.

## 9. Resource, failure and lifecycle rules

| Failure | Required behavior |
|---|---|
| Unknown operation/resource type | `UNSUPPORTED_CAPABILITY`, no attempt to call `#f` |
| Invalid entity ID or scope | Reject before lookup/handler; no cross-principal disclosure |
| Invalid schema/numeric scale | Typed refusal, no rounding or command creation |
| Worker absent after early rejection | Return rejected receipt; no thread dereference |
| Lost response after dispatch | Preserve `UNKNOWN_AFTER_DISPATCH` until reconciled; idempotency rules govern retries |
| Task event out of order | Ignore or buffer boundedly; never regress terminal truth |
| Entity A request returns while entity B visible | Update A's scoped state only; no B contamination |
| Cancel request arrives after completion | Report actual terminal state; cannot rewrite history to `CANCELLED` |
| Unregistered/mutated handler while in use | Reject or drain under scoped lifetime contract |
| Output/error log overflow | Bounded truncation with explicit marker; no unbounded retention |
| Disconnected client | Separate view observation state from task truth |
| Unavailable durable backend | No claim of crash-safe task recovery |
| Change to macro/compiler | Revision-lock mismatch or explicit requalification |
| Privileged operation with no authority provider | Deny; read-only views may still be used |

Recommended initial resource limits are **configuration fields** (`max_payload_bytes`, `max_depth`, `max_entities`, `max_tasks`, `max_events_per_task`, `max_output_bytes`, `max_subscriptions`, `max_pending_commands`, `max_poll_rate`, `task_ttl`, `max_codegen_nodes`). Choose numeric defaults after bounded reference benchmarks and threat modeling; no arbitrary performance promises.

## 10. Qualification: 64 proposed tests and 11 gates

### T01–T12: schema and exact serialization

| Test | Fixture | Required oracle |
|---|---|---|
| P01-D02-T01 | Valid typed query and response | Round-trip canonical fields and exact schema revision |
| P01-D02-T02 | Unknown major protocol version | Reject without attempting a handler |
| P01-D02-T03 | Unknown optional field | Version-policy-controlled, no automatic effect expansion |
| P01-D02-T04 | Malformed JSON and duplicate key ambiguity | Deterministic rejection |
| P01-D02-T05 | Deep/nested/large payload | Quota refusal before materialization |
| P01-D02-T06 | Numeric value above JS safe integer | Decimal string retains exact value |
| P01-D02-T07 | Decimal with wrong scale/fraction | No silent rounding or guessed unit |
| P01-D02-T08 | NaN, Infinity, malformed numeric strings | Strict rejection |
| P01-D02-T09 | Unicode normalization edge cases | Canonicalization policy explicit and stable |
| P01-D02-T10 | Canonical hash collision injection | Exact canonical equality prevents false match |
| P01-D02-T11 | Schema version kept but shape changed | Digest and admitted compatibility detect mismatch |
| P01-D02-T12 | Failure serialized through HTTP | Actual status and problem detail agree; no diagnostic leak |

### T13–T22: entity identity and isolation

| Test | Fixture | Required oracle |
|---|---|---|
| P01-D02-T13 | A and B created from same generated prototype | A serialization unchanged when B created |
| P01-D02-T14 | A/B interleaved updates | Per-instance state and projection preserved |
| P01-D02-T15 | Same display name, different IDs | Distinct cache keys |
| P01-D02-T16 | Same ID after deletion/recreation | Generation prevents stale revival |
| P01-D02-T17 | Same entity, two principals | Projection and cache cannot leak privileged fields |
| P01-D02-T18 | Switch selection A -> B during A request | A response never paints B |
| P01-D02-T19 | Malicious `__proto__`/constructor fields | No prototype or privileged-instance pollution |
| P01-D02-T20 | Hidden private field in projection | No over-disclosure in JSON or logs |
| P01-D02-T21 | Read-only projected state mistaken for command | Type system refuses effect invocation |
| P01-D02-T22 | Concurrent serializers of many entities | No cross-instance, cross-tenant contamination |

### T23–T38: async truth, observation and bounded lifecycle

| Test | Fixture | Required oracle |
|---|---|---|
| P01-D02-T23 | Reject before worker construction | No worker dereference, rejected receipt |
| P01-D02-T24 | Worker starts and completes normally | Success only with completion/effect witness |
| P01-D02-T25 | Worker exits abnormally | Typed failure and separate diagnostics |
| P01-D02-T26 | Empty list of tasks requested | Precondition failure before dispatch |
| P01-D02-T27 | `202 Accepted` without task completion | UI does not display success |
| P01-D02-T28 | Task completion event delayed beyond newer update | No state regression |
| P01-D02-T29 | Duplicate event sequence received | Idempotent reconciliation |
| P01-D02-T30 | Gap in sequence during reconnect | Resnapshot or explicit history-unavailable state |
| P01-D02-T31 | Disconnect while running | Truth remains running; view says disconnected |
| P01-D02-T32 | Cancel requested but effect already succeeded | Actual success retained, not fake cancelled |
| P01-D02-T33 | Cancel requested and confirmed before execution | `CANCELLED` only with verified nonexecution |
| P01-D02-T34 | Response lost after external dispatch | Unknown effect status, no blind retry |
| P01-D02-T35 | Server restarts without durable adapter | No fake restored task or completion claim |
| P01-D02-T36 | Task output grows past quota | Bounded truncation with explicit receipt |
| P01-D02-T37 | Registry fills and TTL expires | Quota enforcement; `TASK_EXPIRED` semantics |
| P01-D02-T38 | Task lookup by another principal | Object-level access refusal |

### T39–T48: capability and authority enforcement

| Test | Fixture | Required oracle |
|---|---|---|
| P01-D02-T39 | Unknown handler registry key | Typed unsupported, not procedure invocation |
| P01-D02-T40 | Duplicate handler registration | Deterministic conflict refusal |
| P01-D02-T41 | Handler unregistered while task running | Drain/deny per lifetime policy |
| P01-D02-T42 | Effectful route missing authority provider | Denied by default |
| P01-D02-T43 | Authenticated principal accesses unrelated entity | Refused on object-level authorization |
| P01-D02-T44 | Client-supplied fake principal/capability | Untrusted tag cannot grant access |
| P01-D02-T45 | Cookie-authenticated cross-origin mutation | Origin/CSRF policy refuses unauthorized request |
| P01-D02-T46 | Same idempotency key, exact same request | One effect, same receipt when qualified |
| P01-D02-T47 | Same key with different payload/revision | Typed conflict, no extra effect |
| P01-D02-T48 | Permission revoked between enqueue and dispatch | Final gate denies stale permission |

### T49–T60: code–data generation and adapter parity

| Test | Fixture | Required oracle |
|---|---|---|
| P01-D02-T49 | Declarative interface produces typed IR | All fields/effects/authority policies retained |
| P01-D02-T50 | Invalid DSL schema or unknown type | Compile-time checked refusal |
| P01-D02-T51 | Generated Gerbil validator vs reference oracle | Same admissibility on fixture corpus |
| P01-D02-T52 | JS/client binding vs Gerbil codec | Round-trip and error parity |
| P01-D02-T53 | Macro/compiler revision drifts | Revision lock mismatch forces requalification |
| P01-D02-T54 | Generated READ operation tries to mutate | Static/effect admission rejects |
| P01-D02-T55 | Untrusted code-shaped field in wire data | Parsing causes no evaluation or privileged side effect |
| P01-D02-T56 | Generated entity uses prototype-shared mutable `self` | Static or runtime isolation test fails |
| P01-D02-T57 | Source IR digest collision or friendly version reused | Distinct semantics not silently aliased |
| P01-D02-T58 | Old client with newer schema | Declared compatibility/refusal, no guessing |
| P01-D02-T59 | Generated binding attempts privileged self-registration | Registry admission prevents activation |
| P01-D02-T60 | One mock app using local adapter vs HTTP/JSON | Same qualified entity/task outcomes |

### T61–T64: fault and independence

| Test | Fixture | Required oracle |
|---|---|---|
| P01-D02-T61 | Partial transport write/failure | No false success; typed uncertainty and replay rules |
| P01-D02-T62 | Invalid CORS/deployment settings | Secure profile refuses activation |
| P01-D02-T63 | Optional P02-D02/P04-D01/P03-D02 unavailable | Pure P01-D02-A operations still work without hidden imports |
| P01-D02-T64 | Deliberate failed assertion in native test runner | Nonzero process exit; no print-only `FAIL` |

### Qualification gates

| Gate | Claim admitted only after evidence |
|---|---|
| **Q0 — Source provenance/license** | Pinned files, compatible source license, original-source/non-port distinction, review classification |
| **Q1 — Wire/schema correctness** | T01–T12: canonical round-trip, schema compatibility, numeric precision, error mapping |
| **Q2 — Entity isolation** | T13–T22: per-instance/per-principal state and scope, JS prototype adversarial tests |
| **Q3 — Async task semantics** | T23–T35: acceptance/status separation, event ordering, cancellation and uncertainty |
| **Q4 — Resource bounds** | T05, T26, T30, T36–T37 and T61: tasks/subscriptions/outputs/requests stay bounded |
| **Q5 — Effectful authority** | T38–T48 plus deployment threat model and independent policy/handler tests |
| **Q6 — Metaprogramming/codegen** | T49–T59: IR provenance, generated parity, safe staging, no self-authorization |
| **Q7 — Adapter parity** | T12, T27, T52, T58, T60–T62 under at least one complete adapter pair |
| **Q8 — Fault tolerance** | T23, T25, T30–T35, T41, T48, T61 and injected failures |
| **Q9 — Cross-edition isolation** | T63: no dependency on optional P02-D02/P04-D01/P03-D02, no mandatory HTTP/JS dependency in core |
| **Q10 — Independent acceptance** | T64, randomized property tests, differential reference harness, recorded reproducible evidence |

**Qualification policy:** `P01-D02-A_QUALIFIED` requires Q0–Q2. `P01-D02-B_QUALIFIED` requires Q0, Q3–Q4, Q8. `P01-D02-C_QUALIFIED` requires Q0, Q1 and Q6. `P07_FULL_QUALIFIED` requires Q0–Q4, Q6–Q10 and a complete adapter pair. Q5 is **additionally mandatory for every effectful deployment**; a purely read-only P01-D02 release can state clearly that effectful operations are disabled and unqualified. A design document passes none of these gates.

## 11. Implementation sequence if separately authorized

1. **P01-D02-00:** freeze pinned source reviews, licenses, failure classification and reference fixtures; explicitly disallow source port by default.
2. **P01-D02-01:** implement canonical schema/envelope/result algebra with immutable reference codec.
3. **P01-D02-02:** implement `EntityRef`, scoped projection and per-instance/per-principal state; reproduce and block GloUI prototype leakage.
4. **P01-D02-03:** implement reference task-state machine, sequence/event log and read-only observation reconcile.
5. **P01-D02-04:** implement strict registry, bounded retention, idempotency and authorization-admission contracts in no-effect mock mode.
6. **P01-D02-05:** implement Gerbil DSL parser/IR normalization and generated Gerbil validation code; independently check macro expansion.
7. **P01-D02-06:** add minimal JavaScript/client metadata generator and parity fixtures. Avoid prescribing a UI toolkit.
8. **P01-D02-07:** add in-process reference adapter, then optional loopback HTTP/JSON adapter with RFC semantics, secure defaults and authenticated effect test harness.
9. **P01-D02-08:** run adversarial tests, independent differential audit, memory quotas, malformed payload fuzzing and cancellation/disconnect fault injections.
10. **P01-D02-Q01:** report admitted profiles, refused claims and benchmark data; only then consider UI toolkit integrations.

## 12. Open decisions and explicitly retained limits

1. Which declarative IR subset is expressive enough for entity projections without becoming a universal UI description language? Begin deliberately small.
2. Canonical JSON normalization and schema compatibility must be defined and tested; JSON Schema interoperability is an adapter, not automatically an exact substitute for Gerbil domain validation.
3. Dynamic capability discovery can leak operation inventory; require principal-filtered views of registry capabilities.
4. Idempotency records may need persistence across crashes. Without a durable adapter, external retry safety across restarts is **not established**.
5. Exactly-once side effects are not promised by an in-memory key; use domain transaction IDs and independently verified external receipts.
6. A task can finish its worker while external commit remains uncertain; report truth and observational confidence separately.
7. Cross-tenant projection caching should be disabled until scope isolation is proven; no universal permission to deduplicate by payload hash.
8. Cancel/stop semantics differ by domain; `CANCEL_REQUESTED` is an observation, not evidence that an external effect was reversed.
9. Real-time and streaming transports require additional backpressure/event-order qualification beyond polling fixtures.
10. Generated client code is a convenience, not a security boundary; all admission is enforced again on the trusted server/adaptor.
11. Admitted HTTP exposure may need CSP, TLS termination, authentication and CSRF strategy. P01-D02 will not presume loopback implies security from every local actor.
12. The design does not require choosing JavaScript, Vue or React; a Rust GUI, terminal UI or native client can consume the same typed contract when individually qualified.
13. Code generation correctness cannot be inferred from macro hygiene alone: semantic parity and negative tests remain mandatory.
14. Cross-edition integration is contract-only. P02-D02 may supply authorization, P04-D01 durable evidence and task records, P03-D02 versioned simulation branches, but none is a mandatory runtime import.
15. Performance, security and correctness remain unqualified until exercised with a real Gerbil toolchain, client adapters, and actual failure-injection evidence.

## 13. Design verdict

`P07_D01_DETAILED_DESIGN_PROPOSED / GLOUI_SOURCE_GROUNDED / GERBIL_NATIVE_OPTIONAL_SPECIALIST / TYPED_MESSAGE_CONTRACTS / PER_ENTITY_SCOPE_ISOLATION / ASYNC_TRUTH_OBSERVATION_SEPARATION / METAPROGRAMMED_VERSIONED_IR / EFFECTS_DENIED_BY_DEFAULT / BOUNDED_TASK_LIFECYCLE / CROSS_ADAPTER_PARITY / 64_FUTURE_TESTS / 11_FUTURE_GATES / UNIMPLEMENTED / UNQUALIFIED / DOCUMENT_PUBLISHED / P01_TO_P06_UNTOUCHED`

**Final design principle:** *The interface may describe, request and observe; it must never silently become the entity, the worker, the permission system, or the proof of success.*

P01-D02 is intentionally a specialist reserve. It can make a Gerbil application pleasantly usable without turning UI rendering or network exposure into a compulsory part of the Meta-Foundation.