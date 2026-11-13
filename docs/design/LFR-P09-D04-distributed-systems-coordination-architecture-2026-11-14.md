# LFR-P09-D04 | Distributed Systems & Coordination Architecture

**Document class:** Source-grounded specialist detailed-design proposal, architecture revision v2  
**Date:** 14 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `ADAPTIVE SYSTEMS` — Adaptive Intelligence & Autonomous Systems  
**Edition:** P09 — Adaptive Intelligence & Autonomous Systems  
**Design slice:** D04 — Distributed Systems & Coordination  
**Primary design language:** Gerbil Scheme for message contracts, bounded simulation, actor coordination and reference algorithms; measured and qualified external backends for stronger durability, consensus and scale  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P09-D04-distributed-systems-coordination-architecture-2026-11-14.md`  
**Status:** `ARCHITECTURE_REVISION_V2_PROPOSED / 80_FUTURE_TESTS / 12_FUTURE_GATES / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved v2 architecture revision published to `docs/design/` on `audit/lisp-foundation-reserve`, replacing the prior v1 document. Implementation, installation, credentials and deployment require separate authorization.

> **Design decision:** P09-D04 is a standalone distributed coordination subsystem. Its Gerbil reference defines messaging, bounded work ownership, delivery evidence and recovery semantics on local ports. Transport, durable storage, consensus and host effects attach through separately qualified capability adapters. K0 supplies the shared contract vocabulary; K1 mechanisms and specialist integrations attach when selected. Reliability claims bind to the tested backend, topology, fault assumptions and authority host.

## 0. Objectives, first product and boundaries

**Research question:** How can Gerbil-centered applications coordinate actors, services, workers and autonomous components across process and network failures while making message delivery, state consistency, scheduling, durability, retry, ordering and authority independently testable?

**First useful product:** `PartitionedWorkLedger`, a deterministic two-worker, one-authoritative-ledger reference with simulated client links. It injects partition, delayed acknowledgement, duplicate delivery, grant turnover, stale-worker completion, crash and recovery. Its oracle checks committed operations, task ownership, fencing and unresolved effects over bounded schedules on one machine. Selected multi-host implementations follow only after backend qualification.

**In scope:** actor identities and generations; protocol/schema negotiation; command/event envelopes; bounded mailboxes and credit-based backpressure; asynchronous ordering; delivery and deduplication; membership and liveness; coordinator epochs; leases and fencing; compare-and-swap state; optional durable queue/log/outbox/inbox; saga/workflow progress; network partitions; retries and recovery; consistency profiles; authenticated endpoints; end-to-end receipts; distributed qualification and benchmark tours.

**Ownership:** D04 owns coordination state, message-delivery contracts, work leases, retry/reconciliation protocols and its internal recovery records. An integrating application owns domain facts, agent objectives, policy decisions and effects. The effect host decides admission and records actual commits. K0 supplies portable identity, validation and receipt vocabulary; K1 supplies selected reusable mechanisms.

**Integration surface:** optional integrations may include P09-D01 (learning jobs), P09-D02 (agent interaction), P09-D03 (controller commands), P08 (world events), P02 (process health), P04 (effect policy), P05 (durable evidence), P06 (history) or unrelated applications. Each adapter translates an application-specific request into the D04 protocol and back. The core reference runs with zero specialist integration adapters.

**Deployment classes:** `FINITE_MODEL`, `LOCAL_ACTOR`, `REMOTE_TRANSPORT`, `DURABLE_WORKFLOW`, `QUORUM_COORDINATION`, and `HOST_EFFECT_INTEGRATION`. Each class has an independent qualification boundary. Live external financial, robot, operating-system or network changes require their own host authorization.

### 0.1 Claim types

| Claim | Minimum evidence | Ceiling |
|---|---|---|
| `PROTOCOL_CONFORMANT` | Schema validation, serialization fixtures, duplicate/conflict semantics | Named protocol and version |
| `MODEL_SAFETY_QUALIFIED` | Bounded schedules, invariant oracle, fault assumptions, counterexample traces | Declared finite model |
| `TRANSPORT_QUALIFIED` | Actual send/ack/drop/delay/reconnect tests on pinned adapter | Transport and topology under test |
| `DURABILITY_QUALIFIED` | Crash/restart, persistent storage, checksum/replay, retention and recovery evidence | Named store and failure model |
| `COORDINATION_QUALIFIED` | CAS/lease/epoch/fencing safety and quorum fault tests | Selected consistency domain |
| `HOST_EFFECT_INTEGRATION_CONFORMANT` | Host approval interface, operation IDs, dedup/receipt and unknown-outcome reconciliation | Named adapter and host/effect profile; admission claim belongs to host |

**Default classification:** `DESIGN_PROPOSED`. No method or backend below has been executed or performance-qualified for LFR.

## 1. Research and implementation evidence ledger

Sources distinguish documented API behavior from proposed LFR semantics. `DOC` is official documentation, `SOURCE` is inspectable implementation, `RESEARCH` is an algorithm/specification, `LFR` is a prior LFR design, and `PROPOSED` is this D04 contract.

| Source | Observed engineering contract | Proposed transfer | Evidence ceiling |
|---|---|---|---|
| [Gerbil `:std/actor`](https://gerbil.scheme.org/reference/std/actor.html) (`DOC/SOURCE`) | Actors, envelopes, `->`, `<-`, `->>`, handles, remote actor servers and ensembles | A native actor adapter with explicit identity, routing, expiry, supervision and reply correlation | API availability is not proof of bounded queues, durable sends or network isolation |
| [Gerbil actor introduction](https://gerbil.scheme.org/guide/intro.html) (`DOC`) | Local actors extend to remote ensembles with registered actors and server connections | Local reference and future remote-profile feasibility | Server cookies/TLS require deployment-specific security audit |
| [NATS Core vs JetStream](https://docs.nats.io/concepts/jetstream) (`DOC/SOURCE`) | Core publish is at-most-once to live subscribers; JetStream stores events and redelivers unacknowledged messages | Alternative ephemeral versus persisted messaging profiles | Broker ACK and consumer ACK have distinct scopes |
| [NATS JetStream delivery](https://docs.nats.io/learn/jetstream/) (`DOC`) | Stream persistence, consumer state, acknowledgement and replay options | Broker adapter with retention, cursor, dedup-window and redelivery tests | Exactly-once claims depend on configured dedup/ack mechanisms and window |
| [RabbitMQ acknowledgements](https://www.rabbitmq.com/docs/confirms) (`DOC/SOURCE`) | Publisher confirms and consumer acknowledgements are independent; redelivery follows lost consumer ACK | Explicit message lifecycle and channel-scoped receipt mapping | Publisher confirm is no proof of downstream effect completion |
| [Apache Kafka delivery semantics](https://kafka.apache.org/40/design/design/) (`DOC/SOURCE`) | Partition order, committed offsets and transactions can support exactly-once topic processing | Optional ordered durable stream adapter and producer/consumer transaction tests | External effects need cooperative atomicity/idempotency |
| [etcd API guarantees](https://etcd.io/docs/v3.4/learning/api_guarantees/) (`DOC/SOURCE`) | KV operations default to linearizable; watch is revision-ordered but not a linearizable read | Coordination backend with CAS, revision, lease, election and watch recovery | Snapshot and watch freshness are separate from quorum read semantics |
| [etcd v3.6 API](https://etcd.io/docs/v3.6/learning/api/) (`DOC`) | KV mod revisions, leases, watch stream resume and compaction boundaries | Concrete key/revision/lease mapping and watch-compacted repair | A leased lock needs host-side fencing of old owners |
| [Temporal Event History](https://docs.temporal.io/encyclopedia/event-history) (`DOC/SOURCE`) | Durable workflow commands and replay of event history restore workflow state after worker crash | Optional orchestration with deterministic workflow progress and activity receipts | Workflow replay does not prove an arbitrary external effect reversible |
| [Temporal retry policies](https://docs.temporal.io/encyclopedia/retry-policies) (`DOC`) | Retry semantics differ for workflows and activities; activity idempotency matters | Typed retry, deadline and ambiguity handling for activity adapters | Retried external call requires effect-specific dedup/reconciliation |
| [CloudEvents v1.0.2](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md) (`SPEC`) | `source + id` identifies a distinct event; versioned event metadata | Event interchange adapter and identity collision negative tests | Event syntax does not grant transport security, authority or processing order |
| [Raft original paper](https://www.usenix.org/conference/atc14/technical-sessions/presentation/ongaro) (`RESEARCH`) | Consensus via leader election, replicated log and safe membership transition | Safety oracle/reference for quorum-backed profile selection | A paper alone is not a production consensus implementation |
| [Tokio bounded MPSC](https://docs.rs/tokio/latest/tokio/sync/mpsc/fn.channel.html) (`DOC/SOURCE`) | Finite channel capacity introduces backpressure | Measured comparator for capacity, bounded waiting and send failure | Rust channel guarantees apply to that channel, not arbitrary distributed mailboxes |
| LFR [K0-D01](LFR-K0-D01-gerbil-meta-foundation-minimal-contract-kernel-2026-11-13.md), [K1-D01](LFR-K1-D01-opt-in-generalized-mechanisms-detailed-design-2026-11-13.md) (`LFR`) | Typed identity, effects, authority, results; optional causal/time/reconciliation mechanisms | Portable boundary vocabulary and selected opt-in mechanisms | Both designs require independent runtime qualification |
| LFR [P09-D01](LFR-P09-D01-reinforcement-learning-adaptive-intelligence-architecture-2026-11-13.md), [D02](LFR-P09-D02-multi-agent-behavioral-systems-architecture-2026-11-13.md), [D03](LFR-P09-D03-control-autonomous-systems-architecture-2026-11-13.md) (`LFR`) | Learner lifecycle, agent coordination, controller/supervisor timing and host effects have separate owners | Concrete request/receipt interoperability | A distributed adapter has no automatic learner, coordination policy or device authority |

**Source qualification for later implementation:** pin source commit/tag, licensed components, protocol version, topology, server and client releases, config, persistence/fsync semantics, TLS/mTLS configuration, fault model, load profile and test harness. Compare actual execution results with the declared oracle.

## 2. Architecture, dependency closure and authority

```text
  Application / domain owner
        |  typed WorkSpec / Event / CommandIntent             P08, P09, P02...
        v
  +---------------- D04 PUBLIC PORT -----------------+
  | validate -> submit -> route -> claim -> reconcile|
  +-----------------------+--------------------------+
                          v
  +---------------- D04 GERBIL CORE -----------------+
  | identity / message / protocol / bounded mailbox |
  | delivery states / attempts / backpressure       |
  | task-ledger / CAS / lease / epoch / fencing      |
  | local workflow progress / receipt reconciliation|
  | finite fault scheduler / independent oracle     |
  +-----------------------+--------------------------+
                          | capability ports
       +----------+--------+---------+----------+
       |          |                  |          |
    ClockPort TransportPort     StateStorePort  HostEffectPort
       |          |                  |          |
  virtual /   local actor /      in-memory /    mock host /
  qualified   NATS / RabbitMQ    transactional   authorized real
  clock       / Kafka            store / etcd   host adapter
                          |
                   Trace / qualification

  K0 contract vocabulary: shared foundation
  K1 mechanisms: selected, qualified extensions
```

**Dependency direction:** the D04 core directly uses only the versioned K0 contract vocabulary and its own Gerbil reference implementations. K1 mechanisms, adjacent specialists, brokers, consensus services and external workflow products enter through declared ports. Profile A uses local reference ports. Profile B or C replaces selected ports after a capability/semantic qualification; an integration choice does not rewrite the public D04 messages.

### 2.1 Dependency classes and independent startup

| Class | Component | Required for the finite D04 reference | Binding contract |
|---|---|---|---|
| Shared foundation | K0 identity/schema/effect/result envelope | Yes: agreed semantics and version lock | Exact identity, validation, version and receipts |
| D04-owned core | Message, mailbox, local scheduler, task ledger, CAS, finite oracle, trace | Yes | Local deterministic reference and versioned state |
| Capability provider | `ClockPort`, `TransportPort`, `StateStorePort`, `HostEffectPort` | Yes as interfaces; local mocks are sufficient | Separate clocks, retries, atomicity and effect truth |
| Qualified extension | K1 M05/M09/M10/M11/M12/M13/M16 | No | Explicit import and independent qualification per mechanism |
| Optional integration | P09-D01/D02/D03, P02/P04/P05/P06/P08, independent apps | No | Translate application requests/receipts while preserving domain ownership |
| Optional infrastructure | NATS, RabbitMQ, Kafka, etcd, Temporal, native libraries | No | Selected configuration and observed capability/claim ceiling |

**Standalone acceptance:** the finite `PartitionedWorkLedger` must compile and execute its model tests with zero imports from P02–P09 specialist packages, zero K1 runtime mechanism imports, zero network access and zero external broker. It still uses the shared K0 contract surface. Failing this test is an architectural coupling defect even if every integration test passes.

**Control plane:** D04 owns routing tables, its coordinator membership and lease/epoch state. **Data plane:** D04 owns typed message movement and the evidence of delivery. **Coordination persistence:** D04 defines its own ledger/outbox/inbox/recovery formats and depends on the selected `StateStorePort` for actual persistence guarantees. An application's domain database or P05 evidence repository can integrate as a separate store provider under an explicit atomicity/transaction claim.

### 2.2 Capability ports

```text
ClockPort.now(clock_domain) -> TimeObservation | Unknown
ClockPort.deadline_verdict(deadline_ref, clock_profile) -> Valid | Expired | Unknown
TransportPort.send(envelope, delivery_profile) -> SendReceipt | Unknown
TransportPort.receive(consumer_ref, budget) -> DeliveryAttempt | Empty | Error
StateStorePort.cas(key, expected_revision, mutation, atomicity_profile)
  -> StoreCommitReceipt | RevisionConflict | Unknown
StateStorePort.snapshot(anchor_spec) -> SnapshotReceipt | Unavailable
HostEffectPort.admit_and_apply(operation, auth_context, expected_revision)
  -> AuthoritativeEffectReceipt | Rejected | Unknown
HostEffectPort.query(host_ref, operation_id) -> AuthoritativeEffectReceipt | Unknown
```

Each capability record carries a provider identity, revision, scope, expiry and qualification evidence. `StateStorePort` explicitly distinguishes ephemeral, single-host transactional and quorum-linearizable stores; `HostEffectPort` declares deduplication atomicity and query capability. A local mock proves the finite-model contract, while a real provider's claims require fault injection against that provider.

A `HostEffectPort` controls admission and effects. If an integration selects P04, its policy decision becomes one input to the host; otherwise the host may use its own qualified admission system. D04 transports the result without gaining the host's authority.

### 2.3 Integration boundaries and ownership

| Integration | Its authoritative decision | D04 responsibility | Evidence handoff |
|---|---|---|---|
| P09-D01 | Policy identity, learning lifecycle and experiment evaluation | Deliver version-locked training/evaluation work | Scheduling, delivery and transport trace |
| P09-D02 | Agent goals, joint-action arbitration, domain resource conflict | Deliver proposals and coordinate infrastructure task ownership | Delivery, stale claim and reservation receipts |
| P09-D03 | Control feasibility, supervision, command validity | Route deadline-bound command to selected host | Fresh/late/unknown delivery and host receipts |
| P08 | World truth, causal admission and generation | Route world-domain events or work | Message/host-effect evidence only |
| P02/P04/P05/P06 | Local process supervision / permission / evidence / historical state respectively | Adapt to lifecycle, authority, storage or event interfaces when chosen | Scoped compatibility witnesses |
| Independent application | Domain truth and authorization | Standard work/task/message coordination | Same D04 protocol and test gates |

**Conflict distinction:** an infrastructure `LEASE_CONFLICT`, `STALE_EPOCH` or `CAS_CONFLICT` describes task ownership and store revision. A domain `JOINT_ACTION_REJECTED` or disputed NPC resource belongs to the P09-D02 host and returns as an opaque domain outcome. D04 carries that outcome, without re-running behavioral arbitration.

**Coordinator identity:** `WorkerId` names a logical participant, `WorkerGeneration` its incarnation, `ClusterMembershipRevision` a configuration change, `CoordinatorEpoch` a grant/fencing revision, and `PolicyRef` an external decision-function reference. Equal display names or addresses never establish identity equality.

### 2.4 Time, ordering and source of truth

- `logical_step`: finite-model event order or declared application-owned step.
- `message_sequence`: sender/channel/partition ordering profile.
- `causal_parent_refs`: explicit event antecedents, with separate correlation identity.
- `clock_time`: source-labelled timestamp and uncertainty/accuracy bound.
- `deadline`: checked in the responsible host's declared clock domain.
- `commit_revision`: assigned by the authoritative state store or effect host.

A transport acknowledgement establishes only its declared delivery milestone. A valid domain effect requires a host admission and authoritative effect receipt. Simulated time, broker sequence and host commit order remain separately identified.

## 3. Message, event and receipt contracts

```text
WireEnvelope {
  protocol_id, schema_version, producer_ref, producer_generation,
  message_id, operation_id_optional, correlation_id_optional,
  causation_refs[], source_scope, destination_scope,
  message_kind: COMMAND | EVENT | REPLY | CONTROL,
  payload_schema_ref, payload_digest, payload_bytes,
  intended_ordering_key_optional, partition_key_optional,
  sent_clock_ref, deadline_ref_optional, expiry_profile_optional,
  authentication_context_ref, trace_context_ref,
  delivery_profile_ref, expected_host_revision_optional
}
DeliveryReceipt {
  message_id, attempt_no, transport_ref, transport_revision,
  accepted_at_transport, persisted_revision_optional,
  consumer_delivery_ref_optional, consumer_ack_ref_optional,
  redelivery_count, observed_lag, disposition,
  evidence_ref, status
}
EffectReceipt {
  operation_id, host_ref, host_revision, actor_ref, authority_ref,
  coordinator_epoch, execution_generation,
  status: REJECTED | ADMITTED | APPLIED | UNKNOWN | COMPENSATED,
  host_commit_revision_optional, observed_effect_ref_optional,
  reconciliation_ref_optional, evidence_ref
}
```

**Port binding:** each `WireEnvelope` carries `delivery_profile_ref` and a destination scope. A dispatched command also carries an exact `HostEffectPort` binding in the destination application profile, including host revision, authorization scheme and idempotency guarantee. The binding is resolved and revalidated at the host; the wire representation cannot mint an effect permission.

**Message identity:** a replayed message retains `message_id` and `payload_digest`. A new semantically distinct message receives a new identity. A logical operation may generate multiple delivery attempts, each with distinct `attempt_no`; semantic dedup keys are `(host, operation_id)` or a declared equivalent, rather than an unscoped UUID alone.

**Event identity:** for CloudEvents interop map `source`, `id`, `type` and `specversion` into a versioned adapter. The required `source+id` uniqueness is an event-level property; operation dedup and host commit identity have independent contracts.

**Validation pipeline:** length/canonical-format checks; schema and version negotiation; credentials and source binding; payload digest; causation/expiry; target host and expected revision; domain-specific admission. Untrusted payloads are decoded as bounded data. A typed error retains the correlation record without executing a command.

### 3.1 Lifecycle table

| Status | Established evidence | Remaining question |
|---|---|---|
| `CREATED` | Producer generated envelope | Transport acceptance |
| `TRANSPORT_ACCEPTED` | Send accepted locally or by broker API | Persistence, peer processing |
| `PERSISTED` | Named storage durability receipt | Consumer handling |
| `DELIVERED` | Consumer obtained this attempt | Business processing |
| `CONSUMER_ACKED` | Broker accepted consumption acknowledgement | Host application effect |
| `HOST_ADMITTED` | Host checked authority and revision | Effect completion |
| `HOST_APPLIED` | Authoritative host committed named operation | Subsequent physical consequences, if any |
| `UNKNOWN` | Request/ACK gap left outcome uncertain | Reconciliation against host state |
| `REJECTED` | Explicit boundary validation/admission rejection | Producer recovery policy |

Publishing, acknowledging consumption and applying a real effect generate distinct receipts. Host-authoritative status is the sole source for `HOST_APPLIED`.

## 4. Delivery modes, ordering and backpressure

**Delivery profiles:** `EPHEMERAL_AT_MOST_ONCE`; `RETRY_AT_LEAST_ONCE` with durable or bounded retention as configured; `LOG_TRANSACTIONAL` within a documented broker transaction; `APPLICATION_IDEMPOTENT_EFFECT` where the host makes operation dedup atomic with effect commit. Select per message category. Telemetry may tolerate loss; a task ledger or device command demands stricter evidence.

**Ordering profiles:** per-sender FIFO, per-partition order, causal-predecessor-ready, or unordered. Any adapter claiming a stronger profile must demonstrate actual behavior under rebalance, reconnect, retry and concurrent producers. Reordering across independent partitions is valid under a partition-scoped guarantee.

**Consumer bookkeeping:** acknowledgements, processing offsets and host commit records have separate references. Ack happens at the configured ownership handoff. `ACK_BEFORE_COMMIT` permits loss of the effect on crash; `COMMIT_BEFORE_ACK` permits redelivery; deduplication makes the second arrangement safe for repeatable host operations.

**Backpressure:** finite mailbox capacity, `in_flight_limit`, credits/prefetch, retry queue budget, batching and bounded queue time. Saturation returns a typed `BACKPRESSURE` or `EXPIRED` outcome or awaits a declared deadline. Work admission never relies on unlimited memory growth.

### 4.1 Duplicate and retry rules

1. Same message ID and identical digest: recognize replay according to configured dedup window/retained record.
2. Same message ID with different digest: `IDENTITY_COLLISION` and quarantine.
3. Same operation ID with matching effect digest and committed host record: return prior host receipt.
4. Same operation ID with a different effect digest: `OPERATION_CONFLICT` and retain both candidates for audit.
5. Timeout without authoritative receipt: `UNKNOWN`, then query host by operation ID; retry only under a qualified idempotency profile.
6. Retry policy: bounded attempts, exponential or task-specific backoff, jitter, deadline, poison-message retirement and dead-letter evidence.

## 5. Membership, leases, elections and fencing

```text
MembershipRecord {
  cluster_ref, membership_revision, node_id, generation,
  role, capability_set, endpoint_set, status,
  health_observation_time, lease_ref_optional, evidence_ref
}
LeaseGrant {
  resource_ref, grantor_ref, owner_ref, owner_generation,
  coordinator_epoch, grant_revision, ttl_clock_domain,
  expires_at_grantor_clock, capability_scope, status
}
FencedWriteRequest {
  operation_id, effect_digest, resource_ref, expected_state_revision,
  owner_ref, owner_generation, coordinator_epoch,
  authority_ref, requested_transition
}
```

**Fencing rule:** the authoritative ledger/host validates `(resource, owner, worker_generation, coordinator_epoch)` against the *current granted tuple* in the same atomic operation as the target state transition and operation-id deduplication. It validates the expected state revision and the grant validity at the relevant host clock. Mere comparison against `last_accepted_epoch` is insufficient if the host has not observed a newer grant. A real multi-host profile must show a transactional grant/commit boundary or a qualified host-side fencing protocol whose new-grant visibility precedes writes under the stated consistency assumptions.

**Lease rule:** a grant records owner generation, epoch, scope, grantor revision and clock domain. Expiration and regrant update the authoritative grant state. The host checks the currently granted tuple before accepting a serialized mutation; when that authority cannot be established, the outcome is `UNKNOWN` or `QUORUM_UNAVAILABLE` under the selected profile. Lease TTL, clock uncertainty and fencing are separately tested.

**Election/consensus:** candidate choices include etcd-backed leases/election for a small control plane and an independently qualified replicated-log service for stronger state machine replication. If a quorum is unavailable, the `QUORUM_COORDINATION` profile refuses writes that require linearizability. Disconnected read-only observations may continue under an explicitly stale-read profile.

**Membership change:** group configuration carries a revision; leader/worker map changes are recorded and validated. Real consensus membership changes require the selected backend's safe transition semantics; a test harness may model membership without pretending to implement Raft production behavior.

### 5.1 Consistency profile matrix

| Workload | Minimum consistency | Suitable candidate | Adverse fixture |
|---|---|---|---|
| Fire-and-forget UI telemetry | Loss-tolerant ephemeral delivery | Gerbil local actor or NATS Core | Offline consumer |
| Multi-agent shared inbox | Ordered per entity, bounded duplicates | Local actor or JetStream/RabbitMQ profile | Reconnect and reorder |
| Durable task allocation | CAS + operation dedup + durable task record | Transactional store + broker/JetStream | Crash before ACK |
| Single-owner resource control | Lease with fencing + host-side epoch validation | etcd and qualified host | Old owner after partition |
| Stream analytics | Partition ordering and offset transactions | Kafka or corresponding stream provider | Rebalance and aborted transaction |
| Long-running workflow | Durable progress + replay + idempotent activities | Temporal or selected workflow engine | Worker crash and activity timeout |
| Convergent counters/sets | Associative/commutative/idempotent merge when specified | Domain-qualified CRDT profile | Concurrent updates and tombstones |
| Financial/robotics effects | Host-authoritative admission and reconciliation | External authority adapter; separately approved | Lost ACK after real effect |

**CRDT boundary:** commutative merge is useful for certain replicated data structures. Unique allocation of a scarce resource, lease ownership or external-actuator command requires a serializing/admission authority unless a specific algorithm proves the requested invariant under its fault model. Last-writer-wins is a conflict-resolution policy with possible information loss.

## 6. Durable queues, outbox/inbox and recovery

**Atomic publication contract:** D04 owns its coordination ledger and outbox/inbox schema. When a domain event is generated by a committed domain-state change, store the state mutation and `OutboxRecord` in one qualified transaction if atomic event publication is required. A publisher relays pending outbox records; a consumer writes an `InboxRecord` and domain effect in one atomic unit when that storage model permits. Cross-store pipelines retain explicit `UNKNOWN` and reconciliation semantics.

```text
OutboxRecord {
  operation_id, domain_revision, event_ref, payload_digest,
  outbox_revision, delivery_attempts, broker_receipt_optional,
  status: PENDING | SENT | ACKED | UNKNOWN
}
InboxRecord {
  host_ref, operation_id, effect_digest, expected_revision,
  result_digest, host_commit_revision, consumed_at,
  status: APPLIED | REJECTED | UNKNOWN
}
RecoveryAnchor {
  storage_ref, revision, checksum, schema_lock,
  snapshot_ref, log_position, retention_policy_ref,
  verification_status
}
```

**Crash matrix:** (i) before source transaction, no outbox record; (ii) after source transaction but before publish, relay resumes; (iii) after publish before broker ACK, duplicate may appear; (iv) after consumer commit before consumer ACK, delivery repeats; (v) after external host effect before client receipt, reconcile with host. Every case requires its own repeatable fixture.

**Snapshots:** distinguish local checkpoint, committed durable snapshot, broker cursor, domain event history and P06 causal history. A recovery claim specifies fsync/replication/retention conditions and the replay closure. Compacted watch streams trigger a fresh authoritative snapshot followed by a verified revision bridge, not silent use of a missing log segment.

## 7. Workflow orchestration, sagas and uncertain effects

**Work unit:** `WorkflowId`, immutable `WorkflowDefinitionRevision`, current `WorkflowStateRevision`, `ActivityId`, `AttemptId` and `OperationId`. A workflow state machine evaluates commands against event history, schedules activities and records retries. Every activity has a budget and idempotency class.

**Saga:** a workflow may schedule independently admitted domain effects and their compensating operations. Compensation is a distinct forward effect with new authority, identity and failure possibilities; it does not guarantee restoration of physical or financial reality. The ledger records `COMPENSATION_REQUIRED`, `COMPENSATING`, `COMPENSATED`, `COMPENSATION_FAILED` or `UNKNOWN` where appropriate.

**Unknown receipt protocol:** maintain `operation_id`, `host_ref`, digest and expected revision; issue a read-only status query; reconcile against an authoritative host receipt; mark explicit conflict if payload or revision differs. If the host lacks a query/dedup interface, the operation remains `UNRESOLVED_EXTERNAL_EFFECT` until a human/host-specific policy determines a safe next step.

**Temporal adapter:** workflows and activities have different retry/replay semantics. The adapter maps workflow history and activity IDs to D04 receipts, pins SDK/server versions and proves model/actual replay consistency for a named workflow definition. D04's minimal reference remains testable without installing Temporal.

## 8. Fault model and invariant specification

**Fault classes:** message drop, duplication, delay, reordering, partition/heal, actor crash/restart, fail-stop store, broker restart, clock skew, stale lease owner, membership turnover, read staleness, data corruption, schema drift, credential rejection and overload. Byzantine behavior and arbitrary hardware corruption are separately scoped advanced profiles.

**Fairness assumption for liveness:** bounded fault schedules or an eventually healed network, restarted workers and an available authoritative quorum as declared. Permanent isolation or unavailable quorum may prevent progress while safety remains intact. Each assertion states its fault scope.

### 8.1 Linearizable task-ledger oracle

The reference oracle is a single authoritative state machine used by two simulated workers. Its state is `S = (rev, next_epoch, grants, tasks, applied_ops, pending, trace, clock)`; `applied_ops` is keyed by `(host_ref, operation_id)`. Each transition is atomic within the model and records the exact source of any clock or delivery fact.

```text
claim(worker, generation, host, task, expected_rev):
  if expected_rev != rev: return STALE_REVISION
  if tasks[task] is terminal: return TASK_TERMINAL
  if grants[task] is currently valid: return CONFLICT
  epoch := next_epoch + 1
  next_epoch := epoch
  grants[task] := current_grant(worker, generation, host, epoch, expiry)
  rev := rev + 1
  return GRANT(task, worker, generation, epoch, rev, expiry)

commit(host, worker, generation, task, epoch, op_id, digest, expected_rev):
  key := (host, op_id)
  if applied_ops contains key:
     if applied_ops[key].digest == digest: return prior_receipt
     else: return OPERATION_CONFLICT
  if profile requires quorum && quorum_unavailable: return QUORUM_UNAVAILABLE
  if expected_rev != rev: return STALE_REVISION
  if tasks[task] is terminal: return TASK_TERMINAL
  if grants[task] != current_grant(worker, generation, host, epoch): return FENCED
  if grant_expired_at_authoritative_clock(grants[task]): return FENCED
  rev := rev + 1
  tasks[task] := COMMITTED
  applied_ops[key] := (digest, rev, task, terminal_receipt)
  return APPLIED(host, op_id, rev)

expire_and_regrant(task, new_worker, new_generation):
  verify expiry against authoritative grantor clock
  atomically invalidate old grant and install epoch := next_epoch + 1
  advance rev; return new current grant
```

**Oracle and implementation boundary:** operation deduplication, current-grant validation and effect commit are one atomic state transition in the finite model. A production host must establish equivalent atomicity or publish a narrower claim. An external effect that cannot share this atomic boundary requires host-side idempotency/reconciliation and cannot inherit the oracle's at-most-once commit guarantee. Multi-node fencing qualification additionally checks that a superseding grant is visible to the enforcing host before it accepts subsequent writes.

**Required safety properties:** `S1` a host-scoped operation ID commits with one immutable digest; `S2` an outdated or expired grant cannot commit; `S3` a task terminal state is committed at most once; `S4` revision increases on accepted mutations; `S5` different active owners never commit the same exclusive task; `S6` unmatched ACK cannot prove host effect; `S7` leaked private payload is rejected by scope validation; `S8` bounded queues respect limits; `S9` replay preserves committed state in the declared durable model; `S10` no quorum-backed write succeeds during loss of quorum.

**Liveness properties, conditional:** `L1` an eligible queued task eventually completes under healthy workers, sufficient retries and eventually available owner; `L2` an unresolved operation eventually transitions to known if the authoritative host becomes queryable; `L3` a backpressured request eventually progresses if consumers drain and its deadline remains valid. These are verified only under matching fairness and retention assumptions.

### 8.2 Enumerated fault schedules

- **F01:** worker A holds epoch 41; network partitions A; lease expires; worker B obtains epoch 42; A's delayed epoch-41 commit is rejected; B commits.
- **F02:** B commits operation `op-7` at revision 9; response is lost; same operation and digest returns original revision-9 receipt with no second transition.
- **F03:** replay of `op-7` carries a different effect digest; host responds `OPERATION_CONFLICT`.
- **F04:** publisher ACK lost after durable publish; outbox retry may produce duplicate delivery; inbox/host dedup yields one applied effect.
- **F05:** consumer process crashes after host commit, before broker ACK; redelivery returns original effect receipt.
- **F06:** cluster loses quorum; linearizable task claims return unavailable; stale-read observation remains explicitly marked.
- **F07:** watch resumes from compacted revision; adapter obtains new snapshot and checks revision bridge before re-enabling decisions.
- **F08:** bounded queue saturates; sender receives backpressure/timeout without unbounded allocation.
- **F09:** schema version mismatch or forged origin is rejected at the boundary.
- **F10:** workflow activity times out after remote execution; activity status remains unknown pending host reconciliation.

## 9. Gerbil implementation profiles and package structure

Three profiles are selected by job, with distinct qualification tests and migration costs.

| Profile | Runtime ownership | Appropriate first workload | Evidence gate |
|---|---|---|---|
| **A — Gerbil reference/local actors** | Gerbil finite model and `:std/actor` transport for local/mocked cases | Two workers, deterministic fault schedules, finite work ledger, low-volume local orchestration | Pinned Gerbil/Gambit compile, trace parity, bounded memory and scheduler behavior |
| **B — Gerbil + qualified native/broker/coordination adapter** | Gerbil protocol/host semantics; selected Kafka/NATS/RabbitMQ/etcd or native client performs transport/storage/consensus function | Durable tasks, external worker pools, linearizable ownership and throughput-bound processing | Adapter semantic parity, cross-process failure tests, resource/cost measurements |
| **C — Gerbil + external workflow/cluster service** | Gerbil request/receipt contracts; service owns declared workflow execution and storage consistency | Long-running durable workflows, federated tasks or remote fault recovery | Auth, service-version lock, history/receipt parity, reconciliation and operational controls |

**A/B/C criteria:** prove source compatibility and message semantics first; compare end-to-end throughput, p50/p95/p99 latency, queue growth, memory, storage amplification, recovery time, operator effort and failure dispositions under equivalent workloads. Profile B or C is promoted when its measured capability is needed for a named task. Scope claims to the actual integration and platform.

### 9.1 Proposed namespace

```text
lfr/p09/distributed/
  contract/{identity,message,delivery,effect,membership,lease,task}.ss
  protocol/{schema,codec,negotiation,validation}.ss
  reference/{virtual-clock,event-scheduler,fault-network,ledger,oracle}.ss
  actors/{mailbox,router,worker,supervisor,local-transport}.ss
  coordination/{cas,lease,fencing,allocation,read-profile}.ss
  recovery/{outbox,inbox,checkpoint,reconcile,workflow}.ss
  ports/{clock,transport,state-store,host-effect,trace}.ss
  adapters/{nats,rabbitmq,kafka,etcd,temporal,remote-host}.ss
  integrations/{p08,p09-d01,p09-d02,p09-d03,p02,p04,p05,p06}.ss
  evaluation/{trace,invariants,benchmark,qualification}.ss
  tests/{protocol,delivery,ordering,fencing,crash,security,parity}.ss
```

### 9.2 Minimal reference API

```text
create_simulated_cluster(spec, seed) -> ClusterHandle | Error
register_worker(cluster, worker_spec) -> WorkerRef | Error
submit_work(cluster, work_spec, expected_revision) -> TaskReceipt | Error
inject_fault(cluster, fault_event, logical_step) -> FaultReceipt | Error
advance(cluster, event_budget) -> TraceSegment | Exhausted
claim_task(cluster, worker_ref, task_ref, expected_revision) -> LeaseGrant | Rejection
commit_operation(cluster, fenced_request) -> EffectReceipt | Rejection
reconcile_operation(cluster, host_ref, operation_id) -> EffectReceipt | Unknown
snapshot(cluster) -> SnapshotRef + Digest
check_invariants(cluster, oracle_spec) -> OracleReport
```

The reference API uses D04-owned contracts and in-memory capability-port implementations. Actual Gerbil API forms, OS integration and serialization are qualified against a pinned toolchain. `:std/actor` is the first actor candidate, with queue bounds and failure semantics established by execution. External brokers and integration adapters ship as separate packages.

## 10. Algorithm and backend-to-job qualification

| Job | Reference algorithm / oracle | Candidate implementation | Disqualifying evidence |
|---|---|---|---|
| Local bounded actor messaging | Finite mailbox FIFO/expiry oracle | Gerbil `:std/actor` + explicit capacity wrapper | Loss or unbounded memory against chosen profile |
| Durable work queue | Atomic task ledger + idempotent retry | JetStream or RabbitMQ quorum queue | Unrecoverable accepted task under declared crash model |
| High-volume ordered stream | Per-partition offset oracle | Kafka transactional stream profile | Out-of-order within guaranteed partition; committed offset/output drift |
| Exclusive ownership | Epoch-fenced CAS state machine | etcd KV transaction/lease | Superseded owner commits; stale watch treated as authoritative |
| Workflow recovery | Event-sourced workflow state replay oracle | Temporal workflow/activity adapter | Activity executes twice when exactly-once host effect was claimed |
| Multi-agent task allocation | D02 allocation oracle + D04 delivery/lease trace | Gerbil coordination or durable broker | Behavior decision silently changes during transport |
| Remote control dispatch | D03 deadline/admission oracle | bounded remote command host | Late/unknown command reported applied |
| Event interchange | `source+id`/schema identity oracle | CloudEvents-adapted backend | Message identity collision or unauthorized schema reinterpretation |
| Convergent replicated counter | Algebraic merge properties | Qualified CRDT counter | Nonconvergence under defined merge/replication model |

**Tour policy:** enumerate realistic candidates per job, pin implementation and failure assumptions, run independent oracles before resource benchmarks. A candidate satisfies a job when it meets its declared objective and mandatory invariants; an unrelated academic preference does not decide qualification.

## 11. Security, operational budgets and observability

**Identity and trust:** the gateway binds message source to an authenticated principal, namespace/tenant, producer generation, schema and target scope. Deployments specify TLS/mTLS, credential rotation, secret storage and transport-level authorization. Event metadata, agent role and network address are distinct from authenticated authority to execute an effect.

**Budget model:** maximum message size, decode depth, mailbox bytes, in-flight records, retention duration, retries, outstanding workflows, connections, memory, CPU, disk, queue delay, per-tenant quota and maximum tracing cardinality. Overload yields `BACKPRESSURE`, `EXPIRED` or `REJECTED_BUDGET` with retained provenance.

**Operational signals:** queue length, enqueued/acked/applied counts, duplicates, redeliveries, unknown-effect backlog, CAS conflicts, lease turnover, stale epochs, leader changes, quorum unavailability, durable write lag, watch compaction, workflow retries, p50/p95/p99 end-to-end latency and data-retention pressure. Correlate by operation ID and trace IDs; redact secrets and private payloads.

**Security fault examples:** mismatched message digest, unrecognized schema version, cross-tenant destination, replay from stale producer generation, unauthorized host endpoint, forged authority ref, unsafe recursive decoder and telemetry poisoning. Status returns carry structured evidence with bounded payloads.

## 12. Invariants and typed outcomes

### 12.1 Core invariants I01-I30

1. **I01 Identity:** message, operation, worker generation and host revision have independent namespaces.
2. **I02 Schema:** unsupported payload/version is rejected at the protocol boundary.
3. **I03 Origin:** source scope is bound to authenticated sender identity for qualified remote profiles.
4. **I04 Message digest:** replay with mutated bytes cannot masquerade as the same message.
5. **I05 Dedup:** same committed operation/digest returns the recorded host receipt.
6. **I06 Operation conflict:** same operation ID with different digest is rejected.
7. **I07 Truth:** broker ACK and consumer ACK remain distinct from host application.
8. **I08 Ordering:** ordering assertions are scoped to the selected channel/partition/profile.
9. **I09 Causation:** correlation and causal-parent references are independent.
10. **I10 Deadline:** expired work is rejected against the responsible clock.
11. **I11 Capacity:** mailbox and retry queues remain within configured bounds.
12. **I12 Credits:** in-flight work respects configured credit/prefetch budget.
13. **I13 Lease authority:** a stale owner cannot commit a fenced resource mutation.
14. **I14 Epoch monotonicity:** accepted coordinator epoch never decreases.
15. **I15 CAS:** accepted state transition matches expected authoritative revision.
16. **I16 Quorum:** linearizable operations require available quorum in the selected profile.
17. **I17 Read freshness:** stale watches/reads carry an explicit consistency status.
18. **I18 Compaction:** missing watch history requires snapshot/bridge repair.
19. **I19 Durability:** persistence claim binds to storage and replication receipt.
20. **I20 Outbox:** atomic source-state/outbox profile has no committed state without event record.
21. **I21 Inbox:** committed host effect and idempotency record share an atomicity boundary.
22. **I22 Recovery:** replay from a verified anchor reconstructs declared state.
23. **I23 Workflow:** workflow progress tracks definition version and completed activity evidence.
24. **I24 Unknown effect:** timeout without a host receipt remains unknown.
25. **I25 Compensation:** compensating action is a new forward effect with independent evidence.
26. **I26 Domain isolation:** an integrating application's domain outcome is carried unchanged through transport and recovery; infrastructure claim conflicts retain their own namespace.
27. **I27 Host truth:** only the bound authoritative host reports applied effects; deadlines are enforced by the selected host clock.
28. **I28 Tenant:** cross-namespace processing requires explicit host authorization.
29. **I29 Resource:** retries and retention cannot exceed declared resource policy.
30. **I30 Claim scope:** evidence reports identify tested model, topology, fault profile and runtime.

### 12.2 Typed results

`ACCEPTED`, `PERSISTED`, `DELIVERED`, `ACKED`, `APPLIED`, `ALREADY_APPLIED`, `CONFLICT`, `STALE_REVISION`, `STALE_EPOCH`, `FENCED`, `QUORUM_UNAVAILABLE`, `BACKPRESSURE`, `EXPIRED`, `SCHEMA_REJECTED`, `AUTH_REJECTED`, `OPERATION_CONFLICT`, `RETRY_EXHAUSTED`, `PARTITIONED`, `WATCH_COMPACTED`, `RECONCILIATION_PENDING`, `UNRESOLVED_EXTERNAL_EFFECT`, `COMPENSATION_FAILED`, `BUDGET_EXHAUSTED`, `NOT_QUALIFIED`.

Each result records the relevant authoritative reference, correlation, revision, scope and evidence pointer. `UNKNOWN` is an outcome category, not a substitute for a positive or negative acknowledgment.

## 13. Qualification plan: 80 proposed tests and 12 gates

**Execution rule:** these are future test specifications. Each test must have a deterministic fixture or exact/falsifiable comparator, an observed result, a failure trace and a declared claim ceiling. No test is marked PASS by design publication.

### 13.1 Gates Q0-Q11

| Gate | Condition | Evidence requirement |
|---|---|---|
| **Q0 — Source closure** | Core dependencies and optional adapters are classified and pinned | Source digest, K0/Gerbil locks, zero-specialist-import build and adapter manifest |
| **Q1 — Schema and identity** | Wire semantics, trust scoping and dedup identities validated | Golden vectors, malformed payload rejection, collision traces |
| **Q2 — Finite-model safety** | Bounded event enumerator checks S1-S10 | State-hash coverage, counterexamples, model configuration |
| **Q3 — Ordering and delivery** | Declared send/ACK/duplicate/reorder properties pass | Broker-independent simulator and chosen adapter traces |
| **Q4 — Capacity and liveness** | Budgets respected, fairness-scoped progress shown | Load model, deadlines, queue growth, L1-L3 traces |
| **Q5 — Lease/CAS/fencing** | Old leaders rejected, monotonic revisions and CAS work | Partition/regrant schedules, host-side fencing trace |
| **Q6 — Durable recovery** | Recovery anchors and outbox/inbox atomicity qualified | Crash matrix, restart, checksum, retention and replay closure |
| **Q7 — Workflow reconciliation** | Duplicate/timeout/compensation paths preserve truth | Event history, activity IDs, external-host response cases |
| **Q8 — Remote backend parity** | Candidate adapter implements its claimed semantics | Cross-process tests and exact configuration/version locks |
| **Q9 — Security and isolation** | Auth, tenant scoping, schema and payload boundaries validated | Negative replay/forgery tests and security configuration |
| **Q10 — Integration port qualification** | Selected integration/effect adapters preserve owner decisions and their host scope | Per-adapter fixtures and host evidence; `NOT_APPLICABLE` for a core-only release |
| **Q11 — Independent audit and release** | Claim level matches observed evidence and budgets | Reproducible report, artifact hashes, reviewer disposition |

**Dependency-closure gate:** T73 is mandatory for every D04 core release. T74–T78 are selectively mandatory only for the integration profile to which they apply. A core-only result cannot be rejected for lack of an unselected learner, game, controller, evidence store or historical engine. Conversely, an integrated claim must pass its selected adapter's exact negative fixtures.

### 13.2 Test register

**Family 1: Identity and protocol**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T01** | Wire schema round-trip | Canonical payload and digest preserved |
| **P09-D04-T02** | Unknown schema revision | Typed schema rejection |
| **P09-D04-T03** | Duplicate message identity | Same digest recognized as replay |
| **P09-D04-T04** | Message ID with altered payload | Identity collision rejected |
| **P09-D04-T05** | Operation versus message IDs | Multiple attempts map to one operation |
| **P09-D04-T06** | Producer incarnation turnover | Old generation cannot impersonate current |
| **P09-D04-T07** | Cross-tenant destination | Scope check rejects request |
| **P09-D04-T08** | CloudEvents source plus id | Event identity map is collision-safe |


**Family 2: Delivery and ordering**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T09** | At-most-once disconnected subscriber | Missing delivery recorded within claim scope |
| **P09-D04-T10** | Unacked-at-least-once message | Redelivery and attempt count observed |
| **P09-D04-T11** | Publisher confirmation before consumer | No host-effect claim inferred |
| **P09-D04-T12** | ACK lost after host commit | Redelivery returns prior effect receipt |
| **P09-D04-T13** | Two ordered messages one partition | Per-partition order preserved |
| **P09-D04-T14** | Two producers different partitions | Cross-partition reorder accepted |
| **P09-D04-T15** | Causal predecessor delayed | Causal-ready rule withholds successor |
| **P09-D04-T16** | Message expires in queue | Deadline outcome and no late host apply |


**Family 3: Capacity and timing**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T17** | Mailbox at exact capacity | Bounded enqueue succeeds within budget |
| **P09-D04-T18** | Mailbox overflow | Backpressure is returned |
| **P09-D04-T19** | Credit exhaustion | In-flight count never exceeds credits |
| **P09-D04-T20** | Slow consumer and fast producer | Queue growth bounded |
| **P09-D04-T21** | Expired retry work | No new attempt after deadline |
| **P09-D04-T22** | Retry attempt cap | Final typed exhaustion with trace |
| **P09-D04-T23** | Clock source mismatch | Wrong clock mapping rejected |
| **P09-D04-T24** | Recovered consumer with future deadline | Liveness only under fairness assumptions |


**Family 4: Membership and fencing**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T25** | Two workers contend for task | One current grant tuple, atomic owner revision |
| **P09-D04-T26** | Partitioned owner replaced | Superseded grant rejected by authoritative host |
| **P09-D04-T27** | Lease renewed before expiry | Same owner generation remains valid |
| **P09-D04-T28** | Lease expires during delayed write | Host rejects stale epoch |
| **P09-D04-T29** | Membership revision changes | Old configuration cannot silently write |
| **P09-D04-T30** | Same epoch different worker | Owner binding rejects second worker |
| **P09-D04-T31** | CAS expected revision mismatch | No mutation, conflict evidence |
| **P09-D04-T32** | Epoch monotonicity over 50 turnovers | No decrease, wraparound policy tested |


**Family 5: Consistency and state**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T33** | Quorum healthy linearizable write | Committed revision monotonic |
| **P09-D04-T34** | Quorum unavailable | Required write refused |
| **P09-D04-T35** | Stale serializable read | Freshness label retained |
| **P09-D04-T36** | Watch event ordering | Revisions monotonic in watch |
| **P09-D04-T37** | Watch does not imply read freshness | No linearizable claim via watch |
| **P09-D04-T38** | Compacted watch resume | Snapshot plus verified bridge |
| **P09-D04-T39** | Concurrent CRDT counter updates | Commutative merge reaches same value |
| **P09-D04-T40** | CRDT used for scarce allocation | Invalid invariant configuration rejected |


**Family 6: Durability and outbox**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T41** | Crash before source transaction | No outbox/effect committed |
| **P09-D04-T42** | Crash after state+outbox commit | Relay can resume |
| **P09-D04-T43** | Publish receipt lost | Duplicate publication, one host commit |
| **P09-D04-T44** | Consumer crashes before host commit | Redelivery can apply once |
| **P09-D04-T45** | Consumer crashes after host commit | Idempotency receipt suppresses repeat |
| **P09-D04-T46** | Corrupt recovery snapshot | Checksum rejects anchor |
| **P09-D04-T47** | Outbox replay across restart | No lost committed source record |
| **P09-D04-T48** | Durability unsupported backend | Claim stays below durable-qualified |


**Family 7: Workflow and effects**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T49** | Activity completes with ACK lost | Host reconciliation finds applied |
| **P09-D04-T50** | Activity never executed | Authoritative status shows absent |
| **P09-D04-T51** | External host lacks status query | Unresolved effect remains explicit |
| **P09-D04-T52** | Same operation with different digest | Operation conflict reported |
| **P09-D04-T53** | Workflow code version changed | Replay contract/version mismatch |
| **P09-D04-T54** | Compensation succeeds | New effect receipt recorded |
| **P09-D04-T55** | Compensation fails | Failure preserved without fabricated rollback |
| **P09-D04-T56** | Workflow timeout and retry | Task-specific retry cap obeyed |


**Family 8: Security and isolation**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T57** | Unsigned/unauthenticated remote message | Authentication failure |
| **P09-D04-T58** | Forged actor capability | Host permission check rejects |
| **P09-D04-T59** | Mutated payload digest | Message rejected before host |
| **P09-D04-T60** | Unexpected nested payload depth | Decoder budget rejects |
| **P09-D04-T61** | Expired auth credential | Re-authentication/denial recorded |
| **P09-D04-T62** | Producer generation replay | Old producer fenced by namespace |
| **P09-D04-T63** | Poison message repeated | Quarantine/retirement bounded |
| **P09-D04-T64** | Private data in telemetry trace | Redacted evidence preserved |


**Family 9: Backend and benchmark**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T65** | Gerbil local actor small messages | Actual compiled adapter trace matches model |
| **P09-D04-T66** | Gerbil cross-process restart | Remote failure classification accurate |
| **P09-D04-T67** | NATS ephemeral and durable variants | Distinct loss/retention profiles pass |
| **P09-D04-T68** | RabbitMQ publisher and consumer ACK | Orthogonal receipt mapping preserved |
| **P09-D04-T69** | Kafka transaction and external sink | Transaction scope/ceiling correct |
| **P09-D04-T70** | etcd CAS and lease turnover | Revision/fencing parity with oracle |
| **P09-D04-T71** | Temporal event-history replay | Activity/workflow receipts reconcile |
| **P09-D04-T72** | Backpressure performance tour | p95/p99 and memory measured end-to-end |


**Family 10: Integration and release**

| ID | Fixture / injection | Required observation |
|---|---|---|
| **P09-D04-T73** | D04 core built with only K0 and Gerbil | No specialist, K1 or external-service runtime imports |
| **P09-D04-T74** | Optional P09-D02 joint-action adapter | Domain arbitration and D04 lease conflict remain distinct |
| **P09-D04-T75** | Optional P09-D03 control adapter | Deadline/admission checked by host clock and authority |
| **P09-D04-T76** | HostEffectPort with or without P04 | Host enforces declared authority; P04 adapter optional |
| **P09-D04-T77** | D04 ledger with standalone store port | Recovery works without P05; optional P05 store separately scoped |
| **P09-D04-T78** | D04 recovery with no P06 history engine | Message replay reconstructs ledger without P06 imports |
| **P09-D04-T79** | Independent model checker rerun | Same fixtures reproduce state hashes |
| **P09-D04-T80** | Release matrix claim audit | Only qualified classes marked accepted |



### 13.3 Claim and release matrix

| Release profile | Mandatory gates | Permitted claim |
|---|---|---|
| Finite reference | Q0/Q1/Q2/Q3/Q4, applicable Q9/Q11 | `MODEL_SAFETY_QUALIFIED` for bounded schedules |
| Local Gerbil actor | Above + local Q8 implementation parity | `LOCAL_RUNTIME_QUALIFIED` for chosen local runtime |
| Broker messaging | Q0/Q1/Q3/Q4/Q8/Q9/Q11 | `TRANSPORT_QUALIFIED` for specified broker, topology and delivery mode |
| Durable work pipeline | Q0/Q1/Q2/Q3/Q4/Q6/Q7/Q8/Q9/Q11 | `DURABILITY_QUALIFIED` for pinned store/failure model |
| Distributed exclusive ownership | Q0/Q1/Q2/Q5/Q8/Q9/Q11 | `COORDINATION_QUALIFIED` with host fencing and quorum assumptions |
| Selected integrations/host effects | Matching prior gates + applicable Q10 and independent host gate | `HOST_EFFECT_INTEGRATION_CONFORMANT` for tested adapter; host retains admission authority |

Qualification is modular: the registered 80 cases form a **total design register**. Every core profile includes the zero-specialist-import fixture T73. Q10 applies only when an integration or real-effect adapter is selected; an unselected adapter is `NOT_APPLICABLE`, rather than a failure. A claim requires its applicable tests and all corresponding gates, with unresolved and excluded workloads published explicitly. Model evidence remains scoped to the finite model.

## 14. Implementation sequencing post separate authorization

1. **D04-00 — Evidence and toolchain locks:** pin Gerbil/Gambit, message formats, source versions, benchmark topology and implementation licenses.
2. **D04-01 — Typed contracts and finite oracle:** canonical envelopes, operation/epoch/revision semantics, deterministic virtual network and exhaustive bounded state exploration.
3. **D04-02 — First working distributed fixture:** `PartitionedWorkLedger`, two workers and a single host ledger; force F01-F10 and check invariant S1-S10.
4. **D04-03 — Gerbil local-actor adapter:** implement fixed-capacity mailbox, event/ACK trace, supervision and shutdown; compare against reference schedules.
5. **D04-04 — Durable store integration:** choose transactional outbox/inbox/ledger storage, test crash cuts and replay closure before claiming crash recovery.
6. **D04-05 — Backend candidate tour:** select per job among NATS/RabbitMQ/Kafka/etcd/Temporal, inspect actual maintained client APIs, toolchain feasibility and license, then run matching parity and failure fixtures.
7. **D04-06 — Selected integration conformance:** first run T73 with no specialist imports, then qualify only the selected application/host adapters against their own source-of-truth and effect contracts.
8. **D04-07 — Independent qualification:** audit Q0-Q11, publish evidence/claim matrix and maintain unresolved cases without expanding claim ceilings.

## 15. Open engineering decisions

| Decision | Proposed baseline | Required evidence |
|---|---|---|
| Native message runtime | Pinned Gerbil `:std/actor` for local reference | Compiler/API probes, mailbox bounds, actor restart behavior |
| First delivery profile | Bounded in-memory at-least-once simulation | Retry schedule and exact operation dedup oracle |
| Initial consistency class | Single-owner CAS ledger, linearizable abstract model | Exhaustive fault schedules and real adapter later |
| Durable storage | Qualify one transactional persistence adapter | Atomic state/outbox + crash recovery + fsync scope |
| Consensus backend | etcd service only where exclusive ownership needs it | Safe lease/fencing with real quorum/partition tests |
| Broker candidate | By actual workload and existing operational footprint | Delivery/recovery semantics, throughput and maintenance |
| Workflow engine | Optional Temporal-style durable progress adapter | Replay, idempotent activities, bounded retry and outage tests |
| Cross-language FFI | Native library or service protocol according to measured need | ABI/memory ownership, version skew, error surfaces |
| Multi-tenant support | Optional named security profile | Authentication, isolation, quotas, encrypted transport |
| CRDT applicability | Convergent observables only for initial profile | Algebraic merge properties and domain invariant check |
| Formal model checking | Finite exhaustive enumerator, later TLA+ if useful | State coverage, counterexample replay, tool qualification |
| Distributed physical control | Host-specific design and approval | Command validity, clock, watchdog, safety and reconciliation |

## 16. Final design verdict

**Design maturity:** v2 architecture review with a standalone Gerbil core, classified dependencies, explicit capability ports, a corrected host-grant fencing oracle, ten fault schedules, thirty invariants, eighty proposed tests and twelve qualification gates. Design publication is not implementation or runtime qualification.

```text
P09_D04_DISTRIBUTED_SYSTEMS_COORDINATION_DESIGN_PROPOSED /
GERBIL_FINITE_REFERENCE_FIRST / ACTOR_MESSAGE_AND_EFFECT_BOUNDARIES /
MESSAGE_OPERATION_WORKER_EPOCH_REVISION_DISTINCT /
DELIVERY_ACK_HOST_APPLY_DISTINCT / BOUNDED_BACKPRESSURE /
PARTITION_DUPLICATE_REORDER_CRASH_FAULT_PROFILES /
LEASE_CAS_FENCING / QUORUM_SCOPED_CONSISTENCY /
OUTBOX_INBOX_REPLAY_AND_WORKFLOW_RECONCILIATION /
OPTIONAL_INTEGRATION_ADAPTERS / STANDALONE_CORE_K0_DEPENDENCY /
A_B_C_IMPLEMENTATION_PROFILES_SELECTED_PER_JOB /
80_FUTURE_TESTS / Q0_TO_Q11 / NO_RUNTIME_EVIDENCE /
NO_BENCHMARK_PASS / UNIMPLEMENTED / DOCUMENT_PUBLISHED
```

> **Closing principle:** D04 coordinates the movement and ownership of work; each domain owner defines its own decisions and each host establishes real effects. A reliable implementation connects those contracts through verifiable receipts, current grants and explicit failure outcomes.