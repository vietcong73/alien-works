# LFR-P09-D04 | Distributed Systems & Coordination Architecture

**Document class:** Source-grounded specialist detailed-design proposal  
**Date:** 14 November 2026  
**Programme:** Lisp Foundation Reserve (LFR-C02)  
**Domain track:** `ADAPTIVE SYSTEMS` — Adaptive Intelligence & Autonomous Systems  
**Edition:** P09 — Adaptive Intelligence & Autonomous Systems  
**Design slice:** D04 — Distributed Systems & Coordination  
**Primary design language:** Gerbil Scheme for message contracts, bounded simulation, actor coordination and reference algorithms; measured and qualified external backends for stronger durability, consensus and scale  
**Repository context:** `vietcong73/alien-works`, branch `audit/lisp-foundation-reserve`; path `docs/design/LFR-P09-D04-distributed-systems-coordination-architecture-2026-11-14.md`  
**Status:** `SOURCE_GROUNDED_DESIGN_PROPOSED / 80_FUTURE_TESTS / 12_FUTURE_GATES / UNIMPLEMENTED / RUNTIME_UNQUALIFIED / DOCUMENT_PUBLISHED_TO_GITHUB`  
**Publication:** Owner-approved publication of this design document to `docs/design/` on `audit/lisp-foundation-reserve`. Runtime implementation, external service installation, credentials and deployment require separate approval.

> **Design decision:** P09-D04 defines versioned distributed message exchange, coordination state, failure-aware workflow progress and evidence of remote effects. It starts with a finite, fault-injectable Gerbil reference and a testable single-owner task ledger. Transport, broker persistence, consensus, effect admission and application authority are selected as independently qualified interfaces. Reliability claims bind to precise scopes, fault assumptions and backend revisions.

## 0. Objectives, first product and boundaries

**Research question:** How can Gerbil-centered applications coordinate actors, services, workers and autonomous components across process and network failures while making message delivery, state consistency, scheduling, durability, retry, ordering and authority independently testable?

**First useful product:** `PartitionedWorkLedger`, a deterministic two-worker/two-host simulation that injects partition, delayed acknowledgement, duplicate delivery, leader/lease turnover, stale-worker completion, crash and recovery. Its oracle checks committed operations, task ownership, fencing and unresolved effects over bounded event schedules. The initial implementation can run on one machine without a broker or remote service.

**In scope:** actor identities and generations; protocol/schema negotiation; command/event envelopes; bounded mailboxes and credit-based backpressure; asynchronous ordering; delivery and deduplication; membership and liveness; coordinator epochs; leases and fencing; compare-and-swap state; optional durable queue/log/outbox/inbox; saga/workflow progress; network partitions; retries and recovery; consistency profiles; authenticated endpoints; end-to-end receipts; distributed qualification and benchmark tours.

**System ownership:** the application or domain host owns state truth and the meaning of effects. D04 owns coordination semantics and provides an explicit remote-effect boundary. K0 supplies portable identity/validation/effect/result vocabulary. Qualified K1 mechanisms are optional; D04 defines concrete distributed behavior and corresponding evidence.

**Ports into adjacent designs:** P09-D01 can supply frozen policies and evaluation tasks; P09-D02 supplies agent proposals, team roles and joint-action decisions; P09-D03 supplies time-bounded control intents and device-host acknowledgements. P05 owns domain memory/evidence policy and P06 owns historical branching/reconstruction. P02 owns local service/process supervision. P04 owns operational permission decisions. D04 defines delivery, remote reconciliation and coordination between those owners.

**Deployment classes:** `FINITE_MODEL`, `LOCAL_ACTOR`, `REMOTE_TRANSPORT`, `DURABLE_WORKFLOW`, `QUORUM_COORDINATION`, and `HOST_EFFECT_INTEGRATION`. Each class has an independent qualification boundary. Live external financial, robot, operating-system or network changes require their own host authorization.

### 0.1 Claim types

| Claim | Minimum evidence | Ceiling |
|---|---|---|
| `PROTOCOL_CONFORMANT` | Schema validation, serialization fixtures, duplicate/conflict semantics | Named protocol and version |
| `MODEL_SAFETY_QUALIFIED` | Bounded schedules, invariant oracle, fault assumptions, counterexample traces | Declared finite model |
| `TRANSPORT_QUALIFIED` | Actual send/ack/drop/delay/reconnect tests on pinned adapter | Transport and topology under test |
| `DURABILITY_QUALIFIED` | Crash/restart, persistent storage, checksum/replay, retention and recovery evidence | Named store and failure model |
| `COORDINATION_QUALIFIED` | CAS/lease/epoch/fencing safety and quorum fault tests | Selected consistency domain |
| `HOST_EFFECT_ADMISSIBLE` | Separate host approval, operation IDs, dedup/receipt and unknown-outcome reconciliation | Named host/effect profile |

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

## 2. Architecture and responsibility contracts

```text
           Application / task owner
                   |
       CommandIntent / Event / WorkSpec
                   |
            Admission adapter <---- K0 + P04 authority (when effects)
                   |
          Protocol gateway / schema
                   |
    +------- D04 COORDINATION PLANE --------+
    | Routing + membership + epochs         |
    | Mailboxes + backpressure + deadlines  |
    | Delivery + ACK + dedup + retries      |
    | CAS / lease / fencing / reservation   |
    | Outbox/inbox + workflow reconciliation|
    +-------+----------------+--------------+
            |                |
       Local actors     Qualified external adapter
        (:std/actor)    NATS / RabbitMQ / Kafka / etcd / Temporal
            |                |
            +--------+-------+
                     |
         Host receives bounded proposal
                     |
         Host admit -> effect -> receipt
                     |
         Trace + independent evaluator
```

**Control plane versus data plane:** the control plane handles schema registries, membership, leases, routing profiles and admission configurations. The data plane carries work messages, streamed records, delivery receipts and effect status. Persistent state is delegated to a declared storage owner; mock storage is sufficient for model qualification, not durability claims.

### 2.1 Owners and integration ports

| Owner | D04 receives | D04 returns |
|---|---|---|
| K0 | Identity/version, validation, authority and effect envelopes | Typed transport/coordination receipts |
| K1 (selected) | M05 time, M09 lifetime, M10 effects, M11 reconciliation, M12 checkpoint, M13 interop, M16 provenance | Profile-bound mechanism evidence |
| P09-D01 | Policy/dataset/experiment work requests and version locks | Delivery/scheduling receipts, run provenance |
| P09-D02 | Agent/team proposals, group membership and joint-work requests | Message/coordination status, ordered observations and conflict outcomes |
| P09-D03 | Controller work deadlines, frozen command intents, host interface identity | Fresh/stale/unknown remote-execution evidence |
| P02/P05/P06 | Local process health, durable storage/history adapters | Remote failure/commit metadata; storage/history ownership unchanged |
| P04 | Authorized effect descriptor for a specific host and scope | Execution reconciliation, never an inferred permission |
| P08 | World adapter event/region/task requests | Delivery semantics; P08 maintains world truth |

**Coordinator identity:** `WorkerId` names a logical participant, `WorkerGeneration` an incarnation, `ClusterMembershipRevision` a membership change, `CoordinatorEpoch` the current lease/fencing generation, and `PolicyRef` an application decision function. Equality of these fields is never inferred from matching display names or network addresses.

### 2.2 Timing, ordering and truth

- `logical_step` is a test-harness event order or a domain-owned step ID.
- `message_sequence` is scoped to a sender/channel/partition and does not establish a global order.
- `causal_parent_refs` records declared event antecedents; observed order alone cannot establish all causality.
- `clock_time` is a source-labelled timestamp with offset/accuracy bound; remote timestamps need uncertainty accounting.
- `deadline` is evaluated by the responsible host against its designated clock and validity profile.
- `commit_revision` is assigned by the authoritative store or host; it is not a broker sequence number by default.

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

**Fencing rule:** for a serialized resource, the authoritative host stores `last_accepted_epoch` and atomically validates `candidate_epoch >= last_accepted_epoch`, expected state revision and semantic operation ID. For exclusive leadership, accept writes from exactly the currently granted owner/epoch, and reject all superseded epochs. A stale client with a still-running process has no implicit authority after lease turnover.

**Lease rule:** a lease is evidence of coordination service liveness during its validity window. A delayed process may continue after lease expiry; effect hosts enforce the epoch. TTL and clock uncertainty belong to the grantor and host contract, not the worker's wall clock estimate.

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

**Atomic publication contract:** when a domain event is generated by a committed domain-state change, store the state mutation and `OutboxRecord` in one qualified transaction if atomic event publication is required. A publisher relays pending outbox records; a consumer writes an `InboxRecord` and domain effect in one atomic unit when that storage model permits. Cross-store pipelines retain explicit `UNKNOWN` and reconciliation semantics.

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

The initial oracle uses a single authoritative state machine. Its abstract state is `S = (rev, next_epoch, current_owner, tasks, applied_ops, pending, trace)` with typed event transitions.

```text
claim(worker, task, expected_rev):
  if expected_rev != rev: return STALE_REVISION
  if task.claimed && lease_valid: return CONFLICT
  epoch := next_epoch + 1
  next_epoch := epoch
  current_owner[task] := (worker, generation, epoch)
  rev := rev + 1
  return GRANT(task, epoch, rev)

commit(worker, task, epoch, op_id, effect_digest, expected_rev):
  if applied_ops contains op_id:
     if matching effect_digest: return prior_receipt
     else: return OPERATION_CONFLICT
  if expected_rev != rev: return STALE_REVISION
  if owner[task] != (worker, generation, epoch): return FENCED
  rev := rev + 1
  tasks[task] := COMMITTED
  applied_ops[op_id] := (effect_digest, rev)
  return APPLIED(op_id, rev)
```

**Oracle conditions:** these transitions are atomic *inside the test model*. A real adapter must implement the same effect-commit atomicity using a qualified host/store operation, otherwise the claim ceiling is lower. A separate action `expire_and_regrant` advances the epoch under the authoritative grantor's clock and creates a new owner generation.

**Required safety properties:** `S1` an operation ID commits with one immutable digest; `S2` a stale epoch cannot commit; `S3` a task terminal state is committed at most once; `S4` revision increases on accepted mutations; `S5` different active owners never commit the same exclusive task; `S6` unmatched ACK cannot prove host effect; `S7` leaked private payload is rejected by scope validation; `S8` bounded queues respect limits; `S9` replay preserves committed state in the declared durable model; `S10` no quorum-backed write succeeds during loss of quorum.

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
  adapters/{nats,rabbitmq,kafka,etcd,temporal,remote-host}.ss
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

The reference API is a design-level contract. Actual Gerbil standard-library forms, OS integration and serialized representation must be compiled and checked against a pinned Gerbil/Gambit toolchain. `:std/actor` is a concrete documented candidate for local messaging and remote actor research, not a substitute for durable storage.

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
26. **I26 Role:** D02 joint proposals are transported without changing arbitration semantics.
27. **I27 Control:** D03 deadlines and host authority survive transport boundaries.
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
| **Q0 — Source closure** | Dependencies, licensing, versions and runtime pinned | Source digest, toolchain manifest, ABI/protocol version |
| **Q1 — Schema and identity** | Wire semantics, trust scoping and dedup identities validated | Golden vectors, malformed payload rejection, collision traces |
| **Q2 — Finite-model safety** | Bounded event enumerator checks S1-S10 | State-hash coverage, counterexamples, model configuration |
| **Q3 — Ordering and delivery** | Declared send/ACK/duplicate/reorder properties pass | Broker-independent simulator and chosen adapter traces |
| **Q4 — Capacity and liveness** | Budgets respected, fairness-scoped progress shown | Load model, deadlines, queue growth, L1-L3 traces |
| **Q5 — Lease/CAS/fencing** | Old leaders rejected, monotonic revisions and CAS work | Partition/regrant schedules, host-side fencing trace |
| **Q6 — Durable recovery** | Recovery anchors and outbox/inbox atomicity qualified | Crash matrix, restart, checksum, retention and replay closure |
| **Q7 — Workflow reconciliation** | Duplicate/timeout/compensation paths preserve truth | Event history, activity IDs, external-host response cases |
| **Q8 — Remote backend parity** | Candidate adapter implements its claimed semantics | Cross-process tests and exact configuration/version locks |
| **Q9 — Security and isolation** | Auth, tenant scoping, schema and payload boundaries validated | Negative replay/forgery tests and security configuration |
| **Q10 — Cross-domain integration** | D01/D02/D03 and P04/P05 ports preserve scope | Frozen policies, host-controlled effects, independent oracle |
| **Q11 — Independent audit and release** | Claim level matches observed evidence and budgets | Reproducible report, artifact hashes, reviewer disposition |

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
| **P09-D04-T25** | Two workers contend for task | One authoritative owner per epoch |
| **P09-D04-T26** | Partitioned owner replaced | Stale owner fenced after regrant |
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
| **P09-D04-T73** | P09-D01 frozen policy dispatch | Policy revision unchanged in flight |
| **P09-D04-T74** | P09-D02 joint-action transport | Coordination decision preserved |
| **P09-D04-T75** | P09-D03 stale remote control command | Deadline/admission checked by host |
| **P09-D04-T76** | P04 authorization revocation | Revoked authority rejected at effect host |
| **P09-D04-T77** | P05 durable evidence store port | Receipt durability uses explicit adapter |
| **P09-D04-T78** | P06 history event handoff | No implicit change to ancestry semantics |
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
| Cross-domain remote effects | Matching prior gates + Q10 + host-specific independent gate | `HOST_EFFECT_ADMISSIBLE` within approved host scope |

Qualification is modular: the registered 80 cases form a **total design register**, while each release claim declares and passes its applicable subset and all mandatory gates. Coverage, unresolved tests and excluded workloads are published explicitly. A profile that passes an exact finite model does not inherit distributed runtime qualification.

## 14. Implementation sequencing post separate authorization

1. **D04-00 — Evidence and toolchain locks:** pin Gerbil/Gambit, message formats, source versions, benchmark topology and implementation licenses.
2. **D04-01 — Typed contracts and finite oracle:** canonical envelopes, operation/epoch/revision semantics, deterministic virtual network and exhaustive bounded state exploration.
3. **D04-02 — First working distributed fixture:** `PartitionedWorkLedger`, two workers and a single host ledger; force F01-F10 and check invariant S1-S10.
4. **D04-03 — Gerbil local-actor adapter:** implement fixed-capacity mailbox, event/ACK trace, supervision and shutdown; compare against reference schedules.
5. **D04-04 — Durable store integration:** choose transactional outbox/inbox/ledger storage, test crash cuts and replay closure before claiming crash recovery.
6. **D04-05 — Backend candidate tour:** select per job among NATS/RabbitMQ/Kafka/etcd/Temporal, inspect actual maintained client APIs, toolchain feasibility and license, then run matching parity and failure fixtures.
7. **D04-06 — D01/D02/D03 interoperability:** execute frozen-policy task, joint action and stale-control deadline fixtures with host-owned admission.
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

**Design maturity:** source-grounded architecture with a finite reference oracle, explicit message and effect receipts, actor/broker/consensus/workflow implementation profiles, ten fault schedules, thirty invariants, eighty proposed tests and twelve qualification gates. No Gerbil code, external service, consensus runtime, durability benchmark or security claim has been qualified in this document.

```text
P09_D04_DISTRIBUTED_SYSTEMS_COORDINATION_DESIGN_PROPOSED /
GERBIL_FINITE_REFERENCE_FIRST / ACTOR_MESSAGE_AND_EFFECT_BOUNDARIES /
MESSAGE_OPERATION_WORKER_EPOCH_REVISION_DISTINCT /
DELIVERY_ACK_HOST_APPLY_DISTINCT / BOUNDED_BACKPRESSURE /
PARTITION_DUPLICATE_REORDER_CRASH_FAULT_PROFILES /
LEASE_CAS_FENCING / QUORUM_SCOPED_CONSISTENCY /
OUTBOX_INBOX_REPLAY_AND_WORKFLOW_RECONCILIATION /
P09_D01_LEARNER_P09_D02_AGENT_P09_D03_CONTROL_SEPARATE /
A_B_C_IMPLEMENTATION_PROFILES_SELECTED_PER_JOB /
80_FUTURE_TESTS / Q0_TO_Q11 / NO_RUNTIME_EVIDENCE /
NO_BENCHMARK_PASS / UNIMPLEMENTED / DOCUMENT_PUBLISHED
```

> **Closing principle:** Distributed correctness is established at the boundary where responsibility changes hands. A message can arrive twice, a worker can outlive its lease, a quorum can disappear and an acknowledgement can be lost. Reliable systems preserve ownership, identity and evidence through those events, then make the next decision from what the authoritative host can actually establish.