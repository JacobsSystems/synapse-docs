---
document_id: EWO-007
title: "Runtime Integration: Local Actor Supervision, Failure Escalation, and Restart Ownership"
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-14
last_updated: 2026-07-14
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.4.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
    - ARCH-004 (v0.1.0 — architecture/ARCH-004-Local-Actor-Supervision-Architecture.md) — the sole architectural authority this EWO implements
  adrs:
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
    - ADR-0017 (Approved)
  predecessor: EWO-006 (work-orders/EWO-006-Bounded-Actor-Mailboxes.md) — prior Runtime Integration milestone; this EWO is the first to implement ARCH-004 rather than closing an ARCH-002/ARCH-003 conformance gap
  reported_by: ER-007 (engineering-reports/ER-007-Local-Actor-Supervision.md, not yet created)
  base_state:
    runtime_head: 5ccc7f9083a71adc6ee704b2322a701935765679
    docs_head: e90404baa5140ce9004839bc51921c789777e003
---

# EWO-007 — Runtime Integration: Local Actor Supervision, Failure Escalation, and Restart Ownership

Registered per STD-001 §46 (Engineering Work Orders). This is the first Engineering Work Order authorizing implementation of ARCH-004 — Local Actor Supervision Architecture. It is also the first Runtime Integration EWO to introduce a new service component (`Supervisor`, parallel to `Scheduler`) rather than extending an existing one. This document authorizes engineering work only. It does not itself constitute approval, implementation, or completion.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-007 |
| Title | Runtime Integration: Local Actor Supervision, Failure Escalation, and Restart Ownership |
| Version | 0.1.0 |
| Status | **Draft** — not yet approved, not yet implemented |
| Author | Denver Jacobs |
| Owner | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-14 |
| Last Updated | 2026-07-14 |
| Classification | Public |
| Applicable repository | `synapse-runtime` |
| Target branch | `main` |
| Runtime base HEAD | `5ccc7f9083a71adc6ee704b2322a701935765679` |
| Documentation base HEAD | `e90404baa5140ce9004839bc51921c789777e003` |
| Predecessor milestone | EWO-006 — Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection |
| Reported by | ER-007 (engineering-reports/ER-007-Local-Actor-Supervision.md, not yet created) |

No implementation exists yet against this EWO. No approval act has occurred. This document authorizes a bounded scope of future engineering work; it does not report on work already done.

---

## Engineering Authority

This implementation is governed by, in descending order:

1. ARCH-001 — Constitutional Architecture — the four constitutional guarantees and the host-process/Runtime-internal supervision boundary (`ARCH-001 §5.4`).
2. ARCH-002 — Runtime Architecture, **v0.2.0** — the authority for actor/instance identity (§7), the Mailbox Model (§13), Dispatch and Routing (§14), the Runtime Lifecycle state model (§15), the minimum audit-event set (§18), Runtime Interfaces (§20), and the explicit deferral of "supervision/restart policy" to a later Lifecycle Architecture document (§15, §23).
3. ARCH-003 — Runtime Integration Architecture, **v0.4.0** — the current, verified integration baseline this EWO builds on without altering.
4. **ARCH-004 — Local Actor Supervision Architecture, v0.1.0 — the specific and sole architectural authority for this milestone's scope, component placement, responsibility boundaries, failure taxonomy, restart identity semantics, failed-instance scheduling semantics, mailbox semantics, parent/child model, escalation model, capability semantics, and audit requirements.** This EWO implements ARCH-004; it does not reinterpret, extend, or narrow it.
5. ADR-0015 — Audit Emitter Failure Semantics (Approved).
6. ADR-0016 — Trusted Core Interaction Rule (Approved).
7. ADR-0017 — Bootstrap Capability Trust Root (Approved).
8. STD-001 — Documentation Standards (§46, Engineering Work Orders).

These documents are authoritative. This task implements them. It does not reinterpret or modify them. Where this EWO makes an implementation-level decision ARCH-004 deliberately left open ("Bounded Design Decisions," below), that decision is bounded, disclosed, and does not amend ARCH-004 itself.

---

## Purpose

This EWO authorizes the first implementation of local actor supervision: a new `Supervisor` service component; the Runtime-level orchestration connecting it to the existing failure path; a failure taxonomy enforced in code (only `Actor::handle()` errors reach supervision policy); restart mechanics preserving `ActorId` and replacing `ActorInstanceId`; withdrawal of Scheduler dispatch-eligibility for a `Failed` instance; a logical, `ActorId`-keyed parent/child hierarchy; a minimal, non-algorithmic Restart/Stop/Escalate/Ignore decision flow; and truthful supervision audit events — exactly as ARCH-004 establishes, and no further.

---

## Problem Statement

Verified directly against `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (`runtime/src/lib.rs`, `core/lifecycle-guardian/src/internal.rs`, `core/actor-host/src/internal.rs`, `core/capability-authority/src/internal.rs`, `services/scheduler/src/lib.rs`, `common/src/lib.rs`), not assumed — restating and independently re-confirming ARCH-004 §6's own verified facts:

- No supervision component, restart counter, escalation path, parent/child field, or supervision-specific audit event exists anywhere in the workspace.
- A failed actor instance (`Actor::handle()` returned `Err`, surfaced by `Runtime::fail_active_execution`, `runtime/src/lib.rs:1552`) remains fully resident in Actor Host, is never terminated by the failure path, and — whenever its mailbox still holds further messages — remains Scheduler-ready, because `Runtime::schedule_next_message` (`runtime/src/lib.rs:1021-1034`) re-marks an instance ready before the message about to execute is even attempted. Confirmed by the existing test `repeated_steps_after_handler_errors_do_not_panic_and_eventually_reach_idle` (`runtime/src/lib.rs:5685`): a second queued message for a permanently failing actor is dequeued and discarded via a second, individually truthful `execution.failed` event, with no coordination, retry, or halt. **This is the defect this EWO eliminates** ("Dispatch eligibility," below).
- `core/lifecycle-guardian/src/internal.rs:47-69`'s `is_legal_transition` already includes `(Failed, Initializing)` as legal, but this EWO's own re-verification (going beyond ARCH-004's own text) establishes a precise, load-bearing clarification: **this edge is never mechanically driven on the same tracked `ActorInstanceId`.** `ActorHostImpl` never reuses an `ActorInstanceId` after `terminate_instance` removes it (`core/actor-host/src/internal.rs:183-193`); every `create_instance`/`create_instance_with_behavior` call mints a new, sequence-suffixed id (`internal.rs:141-144`). Restart, per ARCH-004 §12, replaces the incarnation entirely — the new `ActorInstanceId` is simply absent from Lifecycle Guardian's tracked map at creation, which that crate's own existing default already treats as `Idle` (`current_state`, `internal.rs:88-96`). **Consequently, this EWO requires no new Lifecycle Guardian method and no modification to `synapse-lifecycle-guardian` at all** — restart is realized entirely by Actor Host instance replacement ("Restart Mechanics," below), never by mutating the terminated instance's own stale tracked entry. The old instance's `Failed` entry, if still present in Lifecycle Guardian's map after termination, remains an orphaned historical fact; freeing it is not required by ARCH-004 and is explicitly out of scope ("Explicit Exclusions," below).
- `CapabilityAuthorityImpl.bindings: HashMap<ActorId, HashSet<CapabilityId>>` (`core/capability-authority/src/internal.rs:127`) already keys on logical `ActorId`. **This EWO requires no modification to `synapse-capability-authority` at all** — a replacement instance sharing its predecessor's `ActorId` is automatically, freely capability-continuous.
- `ActorHostImpl::create_instance_with_behavior` (`core/actor-host/src/internal.rs:166-174`) accepts an already-constructed `behavior: Box<dyn Actor>` value supplied once, by the caller, at creation time — Actor Host "does not itself clone or re-invoke anything on the caller's behalf" (existing trait doc, `core/actor-host/src/lib.rs:75-85`). **This surfaces a genuine, concrete gap this EWO must close, not merely disclose:** nothing in the current architecture supplies a *fresh* behavior value for a replacement instance once the original, one-shot value has been consumed and later dropped by `terminate_instance` (`internal.rs:183-193` removes `self.behaviors`). Restart therefore requires a new capability — obtaining a fresh initial behavior value for a supervised `ActorId`, on demand, at restart time — that exists nowhere today. "Required Interface Evolution," below, resolves this as a capability owned entirely by the new `Supervisor` crate, touching no existing crate's public interface.
- Exactly eleven audit-event constructors exist in `runtime/src/lib.rs` (lines 91-222); none represents a supervision decision, a restart, or an escalation (ARCH-004 §18, §22).

None of the above requires correction beyond what this EWO itself authorizes; each fact is cited because it directly bounds and justifies this EWO's scope.

---

## Architectural Authority

| Concern | Owner | Authority |
|---|---|---|
| Supervision policy, restart accounting, failure history, parent/child registry, restart/stop/escalate/ignore decision | **Supervisor (new)** | ARCH-004 §9, §10.1 |
| Lifecycle-transition legality (unchanged; not newly exercised on a reused instance — see Problem Statement) | Lifecycle Guardian | ARCH-004 §9.2; ARCH-002 §6, §15 |
| Instance/mailbox/behavior creation and termination (the mechanics restart invokes) | Actor Host | ARCH-004 §10.2, §14, §19; ARCH-002 §6 |
| Capability continuity across restart (free, via existing `ActorId` keying) | Capability Authority | ARCH-004 §17; ARCH-002 §9 |
| Ready-order among ready actors; withdrawal of a `Failed` instance's own eligibility, performed by Runtime through Scheduler's existing interface | Scheduler (lifecycle-unaware, unchanged) | ARCH-004 §13, §19; `services/scheduler/src/lib.rs:14-23` |
| Execution mechanics and its own existing failure cleanup (unaffected) | Execution Coordinator | ARCH-004 §9.2, §19; ARCH-002 §6 |
| Cross-component orchestration of the whole sequence; the only caller of Supervisor | Runtime | ARCH-004 §10.2, §19; ADR-0016 Rule 1 |
| Audit-event emission (new event categories, existing emission mechanism) | Audit Emitter (unchanged trait/crate); Runtime (sole caller, as today) | ARCH-004 §18; ARCH-002 §18; ADR-0015 |
| Bootstrap Capability — never a restart fallback | Capability Authority (untouched) | ARCH-004 §17; ADR-0017 |

STD-001 §46 governs this document's own form and authority: an EWO "MAY authorize implementation... MUST NOT redefine Architecture... If implementation... reveals an apparent architectural contradiction, the EWO MUST require engineering to stop and return the issue for architectural review rather than resolve it unilaterally." This EWO's own Stop Conditions ("Definition of Failure / Stop Conditions," below) apply that rule concretely.

---

## Objective

Implement local actor supervision exactly as ARCH-004 defines it: a new, narrow `Supervisor` service component, composed by Runtime alongside Scheduler, outside Trusted Core; Runtime-mediated observation of `Actor::handle()` failures and execution of Supervisor's resulting decision through existing component operations; a `Failed` instance that is no longer Scheduler-eligible; restart that preserves `ActorId`, replaces `ActorInstanceId`, inherits capability bindings for free, and does not preserve mailbox contents; a logical, cycle-free, `ActorId`-keyed, single-parent hierarchy; and truthful, correctly ordered supervision audit events. No numeric restart-limit policy, backoff algorithm, or timing logic beyond the one bounded, disclosed constant this EWO itself fixes ("Bounded Design Decisions," below).

---

## Bounded Design Decisions

ARCH-004 fixes architecture; it does not select implementation-level values or mechanisms it explicitly deferred. This EWO resolves exactly the following, and no other open decision is authorized:

### 1. Supervisor crate placement — resolved, not open

`Supervisor` is a new workspace member, `services/supervisor` (`synapse-supervisor`), mirroring `services/scheduler` (`synapse-scheduler`) exactly: a Runtime-composed service, outside Trusted Core, depending on `synapse-common` only. This is not a genuinely open choice — it is the direct, mechanical consequence of the one existing precedent for exactly this architectural placement (ARCH-004 §9.1: "positioned architecturally parallel to Scheduler").

### 2. Restart-allowance constant — open, bounded

ARCH-004 §4/§16 defers numeric restart-limit thresholds as policy. This EWO nonetheless requires *some* fixed, minimal, non-configurable bound for Supervisor's own restart accounting to produce a deterministic, testable Restart vs. Escalate/Stop outcome — mirroring EWO-006's own "Bounded Design Decision" for `MAILBOX_CAPACITY` exactly. **The implementer shall select a small, conservative, fixed, crate-internal restart-allowance constant** (the number of restarts a supervised `ActorId` may receive before its next failure produces Escalate/Stop instead of Restart). Requirements:

- SHALL be a fixed, documented, crate-internal constant in `synapse-supervisor` — not a constructor parameter, not per-`ActorId`-configurable, not backed by any configuration mechanism.
- SHALL NOT involve timing, backoff, or jitter of any kind — a plain integer comparison against a monotonically incrementing per-`ActorId` counter only.
- The value and its rationale SHALL be recorded in ER-007.
- This is an implementation policy choice, not an architectural guarantee (ARCH-004 §4, §16 already exclude numeric policy from architecture); a future milestone may revisit the number without requiring an ADR, provided the revisit does not introduce backoff, timing, or per-actor configurability.

### 3. Fresh-behavior-on-restart capability — open, bounded

Per the Problem Statement's own discovery: restart requires obtaining a fresh, freshly constructed `Box<dyn Actor>`-equivalent value for a supervised `ActorId` at the moment a restart is authorized, and nothing today supplies one. **This EWO requires `Supervisor`'s own registration operation to accept, alongside the `ActorId` (and optional parent), a caller-supplied means of producing a fresh initial behavior value on demand** — for example, a boxed factory closure (`Box<dyn Fn() -> Box<dyn Actor>>`) stored by Supervisor and invoked only when Runtime has already decided to restart. The exact Rust shape is left to implementation; the required *capability* — a fresh value obtainable repeatedly, without reinvoking or cloning a previously consumed instance, and without requiring the original external caller to be present again at restart time — is fixed. This capability lives entirely inside the new `synapse-supervisor` crate's own public interface; it touches no existing crate (`Actor`, `ActorHost`, `ExecutionCoordinator` remain byte-for-byte unchanged).

### 4. Escalation mechanics — resolved, not open

Escalating a child's exhausted restart budget to its parent means exactly, and only: (a) the child's own failed instance is left as Lifecycle Guardian's existing failure cleanup already leaves it — `Failed`, non-dispatchable ("Dispatch eligibility," above) — never replaced and never additionally terminated by this act alone; (b) the failure is recorded against the *parent's own* restart accounting, exactly as if the parent itself had newly incurred one failure; (c) a truthful `supervision.escalated` audit event is emitted. This EWO does **not** cascade any further automatic action against the parent's own live instance merely because a child escalated to it — see "Explicit Exclusions" and "Engineering Decision Log," below, for why this is a deliberate, disclosed boundary, not an oversight.

### 5. Audit event shape — resolved, not open

No field is added to `AuditEvent` (`common/src/lib.rs`). Every new supervision fact is represented by a distinct `event_type` string value on the existing `AuditEvent` shape (`event_type`, `actor`, `capability`, `message`), exactly as every one of the eleven existing events already does. The specific decision (Restart/Stop/Escalate/Ignore) is carried by which event type is emitted, not by a new payload field. The parent `ActorId` an escalation targets is **not** separately carried by `AuditEvent` — this is a disclosed limitation (mirroring EWO-005's own disclosed Exception B precedent), not a defect this EWO is scoped to fix; cross-referencing which parent a given escalation reached requires consulting Supervisor's own tracked hierarchy, out of the audit trail's own scope for this milestone (see ARCH-004 §22, which already names this exact limitation).

No other design decision is authorized to remain open. If implementation determines a materially different decision is required, this is a stop condition ("Definition of Failure / Stop Conditions," below), not something to resolve unilaterally.

---

## Scope

Implement only the following, each directly required by ARCH-004:

### Supervisor component (ARCH-004 §9, §10)

- A new crate, `synapse-supervisor` (`services/supervisor`), depending on `synapse-common` only.
- Owns, and exposes only: registration of a supervised `ActorId` (with an optional parent `ActorId` and a fresh-behavior-supplying capability, Bounded Design Decision 3); a lookup of whether, and by whom, an `ActorId` is supervised; observation of one actor-execution failure for a registered `ActorId`, returning a decision among Restart, Stop, Escalate, Ignore (see "Runtime Sequencing," below); and Supervisor's own internal restart-accounting and parent/child bookkeeping.
- Holds no dependency on `synapse-actor-host`, `synapse-lifecycle-guardian`, `synapse-capability-authority`, `synapse-execution-coordinator`, or `synapse-scheduler` — mirroring Scheduler's own existing, analogous isolation (`services/scheduler/src/lib.rs:14-23`).

### Runtime coordination (ARCH-004 §10.2, §19; ADR-0016)

- `Runtime` gains exactly one new field (a `Supervisor` handle), constructed alongside the existing `scheduler` field in `Runtime::bootstrap_with_config`.
- Runtime remains the only caller of Supervisor's operations, and the only component that executes a Supervisor decision through Actor Host, Scheduler, or Audit Emitter. No new direct component-to-component call is introduced anywhere (see "Component Boundaries and Prohibited Interactions," below).

### Failure routing (ARCH-004 §11)

- Only the dispatch-`Err` path inside `Runtime::execute_message_capturing` (the branch currently calling `fail_active_execution`, `runtime/src/lib.rs:1552`) consults Supervisor. No other existing failure path (`fail_execution`, `release_and_fail_execution` reached from live-instance resolution, lifecycle prevalidation, handle allocation, context construction, or Lifecycle Guardian's own post-dispatch mutation failures) is modified to consult Supervisor.
- Regression tests ("Required Tests," below) MUST prove that admission failures, authorization failures, mailbox overflow, unknown destination, illegal lifecycle transitions, and Runtime/audit-infrastructure failures never reach Supervisor and never appear in Supervisor's own tracked failure history.

### Restart semantics (ARCH-004 §12, §17)

- Same `ActorId`, new `ActorInstanceId` — realized entirely by Actor Host's existing, unmodified `terminate_instance` followed by `create_instance_with_behavior`, using the fresh behavior value Supervisor's own registration-time capability supplies (Bounded Design Decision 3).
- Capability bindings require no action — already `ActorId`-keyed (Problem Statement).
- Mailbox and pending messages are not preserved — the replacement instance's mailbox is the ordinary new, empty mailbox `create_instance_with_behavior` already creates; no transfer mechanism is introduced ("Explicit Exclusions," below).
- Lifecycle restarts fresh — the new instance is absent from Lifecycle Guardian's tracked map at creation, hence `Idle` by that crate's own existing default (Problem Statement). No Lifecycle Guardian code changes.

### Dispatch eligibility (ARCH-004 §13)

- Immediately after the existing `LifecycleGuardian::fail_execution` transition and `execution.failed` audit (unchanged), Runtime withdraws the failed instance's Scheduler dispatch-eligibility, using Scheduler's existing `remove` operation (`services/scheduler/src/lib.rs:48`), on the same best-effort basis `terminate_actor_instance` already uses (`runtime/src/lib.rs:747`, `let _ = self.scheduler.remove(instance)`).
- This withdrawal occurs for **every** dispatch failure, regardless of whether the failing `ActorId` is registered with Supervisor — a `Failed` instance MUST become non-dispatchable unconditionally (ARCH-004 §13, §23); supervision registration governs only whether a *restart* is later attempted, never whether the failed incarnation stops being proposed by Scheduler.
- **This is the specific defect this EWO eliminates:** before this EWO, a `Failed` instance with remaining mailbox work is dequeued and rejected repeatedly (Problem Statement, `repeated_steps_after_handler_errors_do_not_panic_and_eventually_reach_idle`); after this EWO, it is withdrawn from the ready set at the moment of failure and never proposed again unless a replacement instance is later created and separately, legitimately marked ready through the existing, unmodified admission path.

### Parent/child supervision (ARCH-004 §15)

- Registration of a logical `ActorId` for supervision, optionally naming a parent `ActorId`.
- Cycle prevention at registration/parent-assignment time: walking the proposed parent's own ancestry chain and rejecting if it already includes the child's own `ActorId`.
- Lookup of an `ActorId`'s current parent, if any.
- A logical `ActorId` has at most one active parent under this milestone.
- Reassignment and removal of an existing supervision relationship are **not** implemented by this EWO ("Explicit Exclusions," below) — ARCH-004 §15 requires that *if* they exist they must be explicit and Runtime-mediated, but does not require this milestone to implement them, and the task authorizing this EWO explicitly limits scope to "registration, parent assignment, supervision lookup, failure notification, escalation."

### Escalation (ARCH-004 §16, §18)

- Exactly four decision outcomes: Restart, Stop, Escalate, Ignore (see "Runtime Sequencing," below).
- No restart-strategy algorithm, backoff, timing, or publicly exposed retry counter ("Explicit Exclusions," below) — the one bounded restart-allowance constant (Bounded Design Decision 2) is the sole quantitative element, and it is never exposed as a public, per-call parameter.
- Escalation mechanics exactly as fixed in Bounded Design Decision 4.

### Audit (ARCH-004 §18)

- New `event_type` values, emitted only by Runtime (the existing, sole caller of Audit Emitter), for: failure observed by Supervisor; each of the four decisions; restart initiated; restart completed. Reuse the existing `actor.terminated` event for the Stop path's own instance termination (already truthful and sufficient) and the existing `actor.created` event for a restart's own replacement-instance creation (already truthful and sufficient) — these are not new events; they are the same, unmodified constructors, invoked at a new call site.
- Truthful ordering exactly as ARCH-004 §18 fixes it ("Audit Semantics," below).

---

## Explicit Exclusions

Do NOT implement, and do not let any of the following creep into this milestone's scope:

- Restart backoff algorithms, timing, jitter, or delayed/scheduled restart.
- Message retry, redelivery, or acknowledgement protocols.
- Dead-letter queues or dead-letter storage of any kind.
- Mailbox content transfer, draining, or preservation across restart — the replacement instance's mailbox is always new and empty.
- Durable or persistent mailboxes.
- State snapshots, event sourcing, or checkpoint recovery.
- Distributed supervision, remote failure detection, node monitoring, or clustering.
- Workflow compensation or generalized effect-retry frameworks.
- Any Rust API redesign of `Actor`, `ActorHost`, `ExecutionCoordinator`, `LifecycleGuardian`, `MessageGateway`, `CapabilityAuthority`, or any `synapse-common` shared type.
- A publicly exposed restart counter, restart-limit parameter, or any per-`ActorId`-configurable policy value.
- A generic, configurable restart-strategy or supervision-policy framework of any kind.
- Reassignment or removal of an existing supervision relationship ("Parent/child supervision," above).
- Any cascading action taken against a parent's own live instance solely because a child escalated to it (Bounded Design Decision 4).
- Recovery, replay, or reconciliation of the terminated instance's own orphaned Lifecycle Guardian entry (Problem Statement) — it is left exactly as today's implementation already leaves any terminated instance's entry: unreferenced, never freed, architecturally harmless.
- Any modification to `synapse-lifecycle-guardian`, `synapse-actor-host`, `synapse-capability-authority`, `synapse-execution-coordinator`, `synapse-message-gateway`, `synapse-audit-emitter`, or `synapse-host-adapter` (Problem Statement already establishes none is required; see Repository Constraints).
- Any change to `AuditEvent`'s field shape in `synapse-common` (Bounded Design Decision 5).
- A new `RuntimeError` variant, unless implementation proves one is genuinely unavoidable for Supervisor's own registration/cycle-rejection outcome — in which case, prefer reusing `RuntimeError::IllegalTransition` (already used elsewhere for "this operation does not follow the current tracked state") before introducing anything new, and if a new variant genuinely proves necessary, it must be exactly one, narrow, and justified on the same basis this session's own `RuntimeError::AmbiguousAuthority` addition already was.
- SDK or host-specific implementation work.
- Any architecture, ADR, or STD-001 document change.

---

## Trusted Core

The seven Trusted Core components (ARCH-002 §6) are unchanged in count, responsibility, and boundary. `Supervisor` is explicitly **not** Trusted Core — it is a Runtime-composed service, positioned exactly as `Scheduler` already is (ARCH-004 §9.1). No existing Trusted Core component's responsibility is transferred, absorbed, or duplicated. Per the Problem Statement's own verification, this milestone requires **zero source changes** to `synapse-lifecycle-guardian`, `synapse-actor-host`, `synapse-capability-authority`, `synapse-execution-coordinator`, `synapse-message-gateway`, `synapse-audit-emitter`, or `synapse-host-adapter` — every one of the seven Trusted Core crates is touched by this EWO in exactly zero lines. Only `runtime/src/lib.rs` (composition root) and the new `services/supervisor` crate change.

---

## Component Boundaries and Prohibited Interactions

The following direct interactions MUST NOT be introduced by this milestone (ADR-0016 Rule 2, extended without exception to the new participant):

| Prohibited direct call | Why |
|---|---|
| Lifecycle Guardian → Supervisor | Lifecycle Guardian has no path to any component but its own tracked state (ADR-0016); it must remain unaware Supervisor exists. |
| Supervisor → Actor Host | Instance creation/termination is exclusively invoked by Runtime, on Supervisor's returned decision, never by Supervisor reaching into Actor Host itself (ARCH-004 §10.2). |
| Supervisor → Scheduler | Dispatch-eligibility withdrawal is exclusively performed by Runtime (ARCH-004 §13, §19); Supervisor never touches Scheduler's tracked state. |
| Scheduler → Lifecycle Guardian | Scheduler remains lifecycle-unaware, unchanged (`services/scheduler/src/lib.rs:14-23`); this milestone does not add this dependency in either direction. |
| Execution Coordinator → (creating actors) | Execution Coordinator's own responsibility remains context construction, dispatch, and completion only (ARCH-002 §6); it never creates or destroys an actor instance, restart included. |
| Actor Host → (making supervision decisions) | Actor Host executes exactly what Runtime instructs (`terminate_instance`, `create_instance_with_behavior`) and decides nothing about *whether* to do so. |
| Capability Authority → (participating in supervision policy) | Capability Authority's role in restart is entirely passive and automatic (its existing `ActorId`-keyed binding map); it is never consulted by, or aware of, Supervisor's decision-making. |
| Any actor (via `Actor::handle()`'s own emitted messages) → Supervisor | Ordinary actor messaging (this session's own admission pipeline, `admit_message`/`resolve_emitted_message_authority`/`process_emitted_messages`, `runtime/src/lib.rs`) MUST NOT become a second, privileged supervision-control channel (ARCH-004 §10.2, §13). Supervision registration, decisions, and execution are reachable only through Runtime's own internal orchestration, never through a message an actor's own handler emits. |

Runtime remains the only bridge for every one of these pairs, exactly as it already is for every existing Trusted-Core-to-Trusted-Core sequence (ADR-0016 Rule 1).

---

## Architecture Constraints

Do not:

- introduce new Runtime concepts beyond `Supervisor` itself and the orchestration ARCH-004 already specifies;
- invent lifecycle states, capability checks, or a new `ActorState`/`RuntimeState` variant;
- modify constitutional concepts;
- reinterpret ARCH-001, ARCH-002, ARCH-003, or ARCH-004;
- redesign `ActorHost`, `LifecycleGuardian`, `ExecutionCoordinator`, `MessageGateway`, `CapabilityAuthority`, or `Scheduler`'s public trait in any way;
- redesign `Runtime`'s existing public method signatures (`submit_message`, `execute_message`, `step`, `run_until_idle`, `create_actor_instance_with_behavior`, `terminate_actor_instance`, and every other existing public method keep their exact current signatures);
- introduce external dependencies;
- use unsafe Rust;
- give `synapse-supervisor` a dependency on anything beyond `synapse-common`;
- give any existing Trusted Core crate a new dependency on `synapse-supervisor`, or the reverse, in either direction (only `runtime` may depend on `synapse-supervisor`, exactly as only `runtime` depends on `synapse-scheduler` today);
- introduce a generic, configurable supervision-policy or restart-strategy mechanism of any kind.

If implementation requires any of the above: STOP. Produce an Engineering Report explaining the issue. Do not invent a solution.

---

## Repository Constraints

Permitted changes, and only these:

- New crate: `services/supervisor/` (`synapse-supervisor`), its `Cargo.toml`, `src/lib.rs`, `src/internal.rs` (mirroring `services/scheduler`'s own file layout), and a new `README.md`.
- Root `Cargo.toml`: add `services/supervisor` to `[workspace] members` and `synapse-supervisor = { path = "services/supervisor" }` to `[workspace.dependencies]`, mirroring the existing `synapse-scheduler` entries exactly.
- `runtime/Cargo.toml`: add `synapse-supervisor = { workspace = true }`, mirroring the existing `synapse-scheduler` entry.
- `runtime/src/lib.rs`: the new `Runtime` field, its construction in `bootstrap_with_config`, the new failure-routing sequence inside `execute_message_capturing`'s dispatch-`Err` branch, the new audit-event constructors, and any new, narrowly scoped public or private Runtime methods "Required Interface Evolution" (below) requires.
- `runtime/README.md`: documentation of the newly implemented supervision behavior only.
- New tests: within `synapse-supervisor`'s own test module, and within `runtime`'s existing `#[cfg(test)]` module and/or `runtime/tests/`, matching current convention.

Do not modify: governance documents; architecture documents (including ARCH-004 itself); standards; ADRs; work orders other than this one; `common/src/lib.rs` (unless Stop Condition 2, below, is genuinely triggered); any of the seven Trusted Core crates; `services/scheduler`, `services/actor-directory`, `services/audit-pipeline`, or `services/persistence`; `api/`; any existing example or existing test file's own assertions (existing tests may gain new, additive test *functions* proving regression, but no existing test's own body or expected outcome may change — see "Required Tests," below).

---

## Required Interface Evolution

**`synapse-supervisor` (new crate) — minimum public surface, names recommended, not mandated:**

1. **Register/adopt.** Accepts a supervised `ActorId`, an optional parent `ActorId`, and the fresh-behavior-supplying capability (Bounded Design Decision 3). Rejects if the proposed parent's ancestry already includes the child (cycle prevention, per "Parent/child supervision," above) — reuse `RuntimeError::IllegalTransition` for this rejection unless proven insufficient (Explicit Exclusions).
2. **Lookup.** Reports whether an `ActorId` is currently registered and, if so, its parent (if any).
3. **Observe failure.** Accepts the failing `ActorId`; returns one of Restart, Stop, Escalate, Ignore, per the exact decision rule fixed in "Runtime Sequencing," below; for Restart, also yields the stored fresh-behavior-producing capability so Runtime can invoke it.

**`Runtime` (existing crate) — minimum change:**

- One new field on the `Runtime` struct (a `Supervisor` handle), constructed in `bootstrap_with_config` alongside the existing `scheduler: SchedulerHandle` field. `Runtime`'s own public constructors (`bootstrap`, `bootstrap_with_config`) keep their exact existing signatures — this is an internal composition change only.
- A public registration method allowing an embedder to register a supervised `ActorId` (delegating directly to Supervisor's own registration operation) — the minimum surface required for the tests in "Required Tests" (below) to exercise supervision at all through the public API, mirroring how `Runtime::bind_capability` (this session's own addition) is a thin, direct delegation to Capability Authority's existing `bind`.
- No other existing public Runtime method changes signature. The new failure-routing sequence ("Runtime Sequencing," below) is reached entirely from *inside* the existing `execute_message_capturing`'s existing dispatch-`Err` branch — a private, non-public-API change.

No change to `synapse-common`, `synapse-actor-host`, `synapse-lifecycle-guardian`, `synapse-capability-authority`, `synapse-execution-coordinator`, `synapse-message-gateway`, `synapse-audit-emitter`, `synapse-host-adapter`, or `synapse-scheduler` is required or authorized.

---

## Runtime Sequencing

The existing dispatch-`Err` branch inside `execute_message_capturing` (`runtime/src/lib.rs:1552`, `fail_active_execution`) is followed by exactly this new sequence, and only on this branch:

1. (existing, unchanged) `ExecutionCoordinator::fail`, `LifecycleGuardian::fail_execution` (→ `Failed`), Host Adapter release, `execution.failed` audit.
2. **(new)** Runtime withdraws the failed instance's Scheduler dispatch-eligibility: `let _ = self.scheduler.remove(&instance)` (best-effort, mirroring `terminate_actor_instance`'s own existing pattern, `runtime/src/lib.rs:747`). Occurs unconditionally, regardless of supervision registration.
3. **(new)** Runtime asks Supervisor whether `message.destination` (the emitting `ActorId` — already Runtime-established truth from this same function's own earlier `live_instance` call, exactly as `process_emitted_messages` already relies on for the identical fact, `runtime/src/lib.rs:934`) is registered.
   - **Not registered:** no supervision-audit event is emitted (the existing `execution.failed` event already, fully, audits this failure — unchanged from before this EWO). No further action. This is the Ignore outcome in the sense that Supervisor was never consulted for a decision at all — nothing to decide for an unregistered actor.
   - **Registered:** emit the new "failure observed" audit event; ask Supervisor for its decision (Bounded Design Decision 2's fixed-allowance rule); emit the new decision-specific audit event; then:
     - **Restart:** emit "restart initiated"; Actor Host `terminate_instance` (old instance); Actor Host `create_instance_with_behavior` (new instance, same `ActorId`, fresh behavior value from Supervisor); emit the existing `actor.terminated` and `actor.created` events (already truthful, unmodified) for the two Actor Host operations; emit "restart completed" once the new instance is genuinely live.
     - **Stop:** Actor Host `terminate_instance` (old instance, no replacement); emit the existing `actor.terminated` event.
     - **Escalate:** no action against the child's own (already `Failed`, already non-dispatchable) instance beyond steps 1-2 above; record the failure against the parent's own restart accounting; emit "escalated," naming the child (Bounded Design Decision 5's disclosed limitation: the parent is not separately carried in the event).
4. **(new)** Return the existing dispatch-`Err` result unchanged — this sequence records and acts on Supervisor's decision; it does not alter `execute_message_capturing`'s own existing `Result<Vec<EmittedMessageOutcome>, RuntimeError>` return contract. The triggering execution's own failure and any supervision consequence remain separate truthful facts, exactly as ARCH-004 §16, §23 require.

---

## Failure Semantics

- **Supervisor lookup/decision operations themselves do not fail** under ordinary operation (no I/O, no fallible external call) — if implementation finds a genuine failure mode here beyond what `RuntimeError::IllegalTransition` (registration-time cycle rejection) already covers, this is a Stop Condition (below), not something to resolve by inventing a new variant.
- **Actor Host operations invoked during restart (`terminate_instance`, `create_instance_with_behavior`) retain their own existing failure semantics unchanged.** If `terminate_instance` fails (e.g., the instance was already gone — structurally not expected here, since Runtime just finished executing it, but not assumed away): propagate the existing `RuntimeError` unchanged; do not attempt `create_instance_with_behavior` afterward; emit no "restart completed" event (only whatever existing audit obligation the failing operation itself already carries). If `create_instance_with_behavior` fails after a successful `terminate_instance`: the `ActorId` is left with no live instance (identical to an ordinary `Stop` outcome's own end state) — emit no "restart completed" event; this is a disclosed, honest degradation from Restart to a Stop-equivalent end state, not a silently swallowed error, and MUST be provable by a dedicated test (see "Required Tests," below).
- **Audit-emission failure for any new supervision event follows ADR-0015 unchanged:** the operation reporting that event fails if the emission fails, with no rollback of already-committed component-level state (the same rule every existing mandatory audit obligation in this codebase already follows).
- **No new failure mode is introduced beyond what this section defines.**

---

## Restart Mechanics

Restated precisely, binding on implementation (ARCH-004 §12, §14, §17):

1. Same `ActorId`; new `ActorInstanceId`, minted by Actor Host's own existing, unmodified sequence counter.
2. Capability bindings: no action required; already `ActorId`-keyed.
3. Mailbox: the replacement instance's mailbox is the ordinary new, empty one `create_instance_with_behavior` already creates. The old instance's mailbox is discarded by `terminate_instance`, exactly as today. No transfer of any kind.
4. Lifecycle: the replacement instance begins absent from Lifecycle Guardian's tracked map, hence `Idle`, exactly as any newly created instance today. No Lifecycle Guardian code change (Problem Statement).
5. Behavior: a fresh value obtained from the factory Supervisor stored at registration time (Bounded Design Decision 3) — never the old, already-consumed value, never a clone of it.
6. Scheduler: the replacement instance is **not** automatically marked ready by restart itself — it becomes ready only through the existing, unmodified admission path (`submit_message`, or a future emitted message addressed to it), exactly as any other newly created instance already behaves today.

---

## Escalation Mechanics

Exactly as fixed in Bounded Design Decision 4: recording only, against the parent's own restart accounting; no action against the parent's own live instance; the child's own instance remains `Failed` and non-dispatchable, neither replaced nor further terminated by the act of escalating alone. A future, separately authorized milestone may define what happens when a parent's own accounting is repeatedly exhausted by cascading child escalations — this EWO explicitly does not, and its own tests ("Required Tests," below) must not assume or assert any such cascading behavior.

---

## Audit Semantics

Truthful ordering, exactly as ARCH-004 §18 fixes it, and not reorderable:

```text
1. Execution failure and its existing cleanup are recorded
   (execution.failed — existing, unmodified by this EWO)
        |
        v
2. Runtime withdraws Scheduler dispatch-eligibility (new, "Runtime
   Sequencing" step 2)
        |
        v
3. Supervisor observes the completed failure state
   (only if the ActorId is registered — new, "Runtime Sequencing" step 3)
        |
        v
4. A supervision decision is recorded
   (new audit event, naming the decision)
        |
        v
5. Runtime executes the decision through existing component owners
   (Actor Host terminate/create, reusing existing actor.terminated/
    actor.created events; or recording against a parent for Escalate)
        |
        v
6. The result is recorded
   (restart completed, or the absence of one on a disclosed
    Actor-Host-failure degradation, per Failure Semantics)
```

No audit record may claim a restart completed before the replacement instance is genuinely live (ARCH-004 §18). No new `AuditEvent` field is introduced (Bounded Design Decision 5).

---

## Definition of Done

The task is complete only if all of the following are true:

- A new `synapse-supervisor` crate exists, depending on `synapse-common` only, exposing registration, lookup, and failure-observation operations per "Required Interface Evolution."
- `Runtime` composes `Supervisor` alongside `Scheduler`, with no other existing public Runtime method signature changed.
- Only the dispatch-`Err` path inside `execute_message_capturing` consults Supervisor; every other failure path (admission, authorization, mailbox overflow, unknown destination, illegal lifecycle transition, Runtime/audit-infrastructure failure) never reaches Supervisor and never appears in its tracked failure history — proven by regression tests ("Required Tests," below).
- A `Failed` instance is withdrawn from Scheduler's ready set at the moment of failure, unconditionally, regardless of supervision registration.
- Restart preserves `ActorId`, replaces `ActorInstanceId`, requires no Capability Authority action, does not preserve mailbox contents, and begins the replacement instance's lifecycle fresh.
- Escalation, Stop, and Ignore each produce their own distinct, correctly ordered, truthful audit trail, with no cascading action against a parent's own instance.
- Supervision registration is opt-in; no pre-existing actor, test, or demonstration (`runtime/tests/worker_pool.rs`, `runtime/tests/actor_to_actor_messaging.rs`, `runtime/examples/*`) is required to register a supervisor, and none of their existing assertions change.
- No Trusted Core crate's source changes. Trusted Core remains exactly seven components.
- No public interface of any existing crate changes signature.
- `synapse-supervisor`'s dependency set is exactly `synapse-common`.
- All pre-existing tests across the workspace continue to pass unmodified in behaviour.
- All tests pass. No warnings. No unsafe.

---

## Mandatory Validation

Execute:

```text
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace
cargo test --workspace
cargo tree --workspace
```

All must pass. Additionally verify: workspace member count increases by exactly one (`services/supervisor`); zero `unsafe`; no dependency cycle; `cargo tree -p synapse-supervisor` shows exactly `synapse-common`; no existing crate other than `runtime` depends on `synapse-supervisor`; every existing public method signature named in "Architecture Constraints" is byte-for-byte unchanged; the existing demonstrations (`cargo run --example worker_pool`, `cargo run --example actor_to_actor_messaging`) continue to run to completion with unchanged output.

---

## Data and State Constraints

- `Supervisor`'s own internal state: a per-`ActorId` restart counter; a per-`ActorId` optional parent `ActorId`; a per-`ActorId` stored fresh-behavior-producing capability. No other state.
- No new field is added to `Message`, `ActorId`, `ActorInstanceId`, `ExecutionContext`, `AuditEvent`, or any other `synapse-common` type (Bounded Design Decision 5; "Explicit Exclusions").
- No new field is added to `TrustedCore`. Exactly one new field is added to `Runtime` itself (the Supervisor handle), alongside the existing `scheduler` field.
- Do not add: a persistence layer, a configuration struct, a generic policy registry, or any bookkeeping beyond what this section names.

---

## Definition of Failure / Stop Conditions

Stop immediately, and produce an Engineering Report rather than resolving the issue unilaterally, if any of the following occurs:

1. Restart is found to require any modification to `synapse-lifecycle-guardian`, `synapse-actor-host`, or `synapse-capability-authority` beyond the existing, unmodified operations this EWO already names.
2. Supervision registration's cycle-rejection outcome is found to require a new `RuntimeError` variant because `IllegalTransition` proves insufficient or already semantically committed elsewhere in conflict with this use.
3. The fresh-behavior-on-restart capability (Bounded Design Decision 3) is found to require any change to the `Actor` trait itself (`synapse-api`).
4. `AuditEvent`'s existing shape is found to be insufficient to truthfully represent a required supervision fact without inventing a new field.
5. Any existing public method signature (`Runtime`, `ActorHost`, `LifecycleGuardian`, `ExecutionCoordinator`, `MessageGateway`, `CapabilityAuthority`, `Scheduler`) is found to require a change.
6. A direct dependency between `synapse-supervisor` and any Trusted Core crate, or `synapse-scheduler`, is found to be required in either direction.
7. Escalation is found to require any action against a parent's own live instance to satisfy this EWO's own Definition of Done (it does not — resolving that is explicitly out of scope; see "Escalation Mechanics").
8. Cycle prevention ("Parent/child supervision") is found to require anything beyond a straightforward ancestry-chain walk at registration/parent-assignment time.
9. Any existing test's own assertions are found to require modification (only additive new tests are authorized).
10. Any ADR guarantee or ARCH-002/ARCH-003/ARCH-004 architectural boundary is found to require change to complete this milestone.

Do not resolve these issues yourself. Report them.

---

## Implementation Sequence

1. Baseline verification: re-confirm the current workspace state matches this EWO's Problem Statement exactly.
2. Create `services/supervisor` (`synapse-supervisor`); register it in root `Cargo.toml`.
3. Implement Supervisor's registration, lookup, and failure-observation operations, per "Required Interface Evolution," including the fixed restart-allowance constant (Bounded Design Decision 2) and the fresh-behavior-capability storage (Bounded Design Decision 3); record the chosen constant and its rationale for ER-007.
4. Component-local tests within `synapse-supervisor` proving registration, lookup, cycle rejection, and the Restart/Stop/Escalate/Ignore decision rule in isolation ("Required Tests," below).
5. Add the new `Runtime` field and its construction in `bootstrap_with_config`; add the minimum public registration-delegation method ("Required Interface Evolution").
6. Implement the new failure-routing sequence inside `execute_message_capturing`'s existing dispatch-`Err` branch, per "Runtime Sequencing."
7. Implement the new audit-event constructors in `runtime/src/lib.rs`, reusing `actor.terminated`/`actor.created` where "Runtime Sequencing" specifies.
8. Runtime-level tests proving: dispatch eligibility withdrawal; restart identity/capability/mailbox semantics; Stop; Escalate; Ignore; the failure-routing exclusion of every non-actor-execution failure class; and the elimination of the pre-existing failed-mailbox-drain defect ("Required Tests," below).
9. Regression pass: existing actor-to-actor messaging, scheduler, lifecycle, capability, and worker-pool tests and demonstrations, unmodified, all passing.
10. Documentation updates: `services/supervisor/README.md` (new) and `runtime/README.md`, strictly limited to describing the newly implemented behavior.
11. Complete quality validation, per "Mandatory Validation."
12. ER-007 preparation after implementation and validation are complete — not before, and not as part of this EWO.

Do not include work from later milestones.

---

## Acceptance Criteria

Every criterion is objective and testable:

- Exactly one new crate (`synapse-supervisor`) and `runtime/`'s own composition-root file change; no Trusted Core crate's source changes.
- `synapse-supervisor`'s dependency set is exactly `synapse-common`.
- No existing public method signature changes anywhere in the workspace.
- Only `Actor::handle()` dispatch failures ever reach Supervisor; every other failure class is proven, by regression test, never to reach it.
- A `Failed` instance never becomes Scheduler-eligible again except through a genuine, separately admitted or emitted message to a legitimately created (possibly replacement) instance.
- Restart identity, capability continuity, and mailbox non-preservation each match "Restart Mechanics" exactly.
- Escalation, Stop, and Ignore each produce the exact audit ordering "Audit Semantics" fixes, with no cascading parent-instance action.
- All pre-existing tests pass unmodified in outcome; all pre-existing demonstrations run unchanged.
- New tests, per "Required Tests," are added and pass.
- `cargo fmt --all -- --check`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`, `cargo build --workspace`, and `cargo test --workspace` all pass with zero warnings.
- `git status` after implementation shows changes confined to `services/supervisor/`, root `Cargo.toml`, `runtime/Cargo.toml`, `runtime/src/lib.rs`, `runtime/README.md`, and new test files — no unrelated file touched.
- Architecture and ADR documents remain byte-for-byte untouched.

---

## Required Tests

### Supervisor tests (`synapse-supervisor`)

- Registration succeeds for a previously unregistered `ActorId`.
- Registration with a parent succeeds when the parent is not a descendant of the child.
- Registration/parent-assignment is rejected when it would create a cycle.
- Lookup reports the correct registration and parent state.
- Observing a failure for an unregistered `ActorId` is meaningful only in that Supervisor reports "not registered" — Runtime, not Supervisor, interprets this as Ignore (see "Runtime Sequencing," above).
- Observing repeated failures for a registered `ActorId` with no parent: Restart while under the fixed allowance; Stop once exhausted.
- Observing repeated failures for a registered `ActorId` with a parent: Restart while under the fixed allowance; Escalate once exhausted, with the parent's own accounting incremented.
- The stored fresh-behavior capability is invoked exactly once per authorized Restart decision and produces a new value each time (not a cached, reused one).

### Runtime tests (`runtime`)

- **Dispatch eligibility:** a `Failed` instance with remaining queued mail is not proposed by the Scheduler again; the previously discarded messages are provably never dequeued after the failure (regression covering the exact scenario `repeated_steps_after_handler_errors_do_not_panic_and_eventually_reach_idle` exercises — this existing test's own assertions are not modified; a new, additive test proves the corrected behavior).
- **Restart — successful path:** same `ActorId`, new `ActorInstanceId`; the replacement instance's capability bindings (bound to the shared `ActorId` before the failure) authorize it identically to its predecessor with no re-binding call; the replacement's mailbox is empty even if the failed predecessor had queued mail; the replacement's `Actor::handle()` genuinely executes on a subsequent, ordinarily admitted message.
- **Restart — Actor-Host-failure degradation:** if `create_instance_with_behavior` fails after a successful `terminate_instance`, the `ActorId` ends with no live instance and no "restart completed" event, exactly as Failure Semantics requires.
- **Stop decision:** exhausting a parentless registered `ActorId`'s restart allowance terminates it with no replacement; a subsequent message addressed to it is rejected exactly as any message to an actor with no live instance already is today.
- **Escalate decision:** exhausting a registered `ActorId`'s restart allowance, with a parent registered, leaves the child `Failed` and non-dispatchable, increments the parent's own accounting, and emits the expected audit trail — with no action taken against the parent's own live instance.
- **Ignore:** a dispatch failure for an unregistered `ActorId` produces exactly the same observable behaviour as before this EWO (`execution.failed` audit only) plus the new, unconditional Scheduler dispatch-eligibility withdrawal ("Runtime Sequencing" step 2, which applies regardless of registration).
- **Failure-routing exclusion (regression), one test per class, each proving the class never reaches Supervisor and never appears in its failure history:** invalid envelope; structural send-authority mismatch; capability denial; revoked capability; ambiguous authority; mailbox overflow; unknown destination; illegal lifecycle transition; a mandatory audit-emission failure (Runtime/infrastructure class).
- **Audit ordering:** for a registered `ActorId`'s Restart path, the recorded event sequence matches "Audit Semantics" exactly (execution.failed, then the new failure-observed/decision/restart-initiated events, then actor.terminated and actor.created, then restart-completed) — provable via `RecordingAuditEmitter`, the same test double this session's own existing tests already use.
- **Existing actor-to-actor messaging regression:** all tests in `runtime/tests/actor_to_actor_messaging.rs` and the corresponding in-crate tests continue to pass unmodified, proving supervision introduces no interference with emitted-message admission, causation, or the shared admission pipeline.
- **Existing scheduler regression:** all tests in `services/scheduler`'s own test module continue to pass unmodified, and Scheduler's own dependency set remains unchanged (still no dependency on `synapse-actor-host`, `synapse-capability-authority`, or `synapse-lifecycle-guardian` — this EWO also does not add `synapse-supervisor` as one of Scheduler's dependencies).
- **Existing lifecycle regression:** all tests in `core/lifecycle-guardian`'s own test module continue to pass unmodified; `synapse-lifecycle-guardian`'s own source is byte-for-byte unchanged.
- **Existing capability regression:** all tests in `core/capability-authority`'s own test module continue to pass unmodified; `synapse-capability-authority`'s own source is byte-for-byte unchanged.
- **Existing worker-pool regression:** `runtime/tests/worker_pool.rs` and `runtime/examples/worker_pool.rs` continue to pass/run unmodified, with identical output.

### Not required

- Any test proving cascading action against a parent's own instance upon repeated child escalation — no such mechanism is implemented (see "Escalation Mechanics").
- Any test proving mailbox content preservation across restart — explicitly not implemented (see "Explicit Exclusions"; "Restart Mechanics").
- Any test proving numeric backoff or timing behaviour — none exists to test.

---

## Engineering Decision Log

Record:

- implementation decisions (expected: the chosen restart-allowance constant and its rationale; the concrete shape chosen for the fresh-behavior-supplying capability);
- repository decisions (expected: `services/supervisor` file layout, mirroring `services/scheduler`);
- deferred decisions (expected: none beyond what this EWO itself already defers, per "Explicit Exclusions" and "Escalation Mechanics");
- architectural decisions (expected: None — ARCH-004 is implemented, not amended);
- constitutional decisions (expected: None);
- future work enabled (expected: a genuine, testable local-supervision mechanism a future durable-mailbox or workflow milestone can build on, per ARCH-004 §21);
- future work deferred (expected: reassignment/removal of supervision relationships; cascading parent-instance action on repeated escalation; mailbox transfer; numeric backoff/timing policy; every item ARCH-004 §4/§21/§22 already lists, unaffected in status by this EWO).

---

## Completion Report

ER-007 must provide, after implementation:

1. Files created (expected: `services/supervisor/` in full, plus new tests within `runtime`).
2. Files modified (expected: root `Cargo.toml`, `runtime/Cargo.toml`, `runtime/src/lib.rs`, `runtime/README.md`).
3. The chosen restart-allowance constant and its rationale.
4. The concrete shape chosen for the fresh-behavior-supplying capability, and why.
5. Confirmation, re-verified against source, that all seven Trusted Core crates and `synapse-scheduler` remain byte-for-byte unchanged.
6. Supervisor and Runtime behaviour implemented, confirmed against "Restart Mechanics," "Escalation Mechanics," and "Runtime Sequencing."
7. Tests added, mapped against "Required Tests."
8. Validation results (Mandatory Validation, in full).
9. Dependency changes (expected: exactly one new crate, `synapse-supervisor`, depended on by `runtime` only).
10. Trusted Core changes (expected: none — same seven components, same boundaries).
11. Architecture changes (expected: none).
12. Explicit confirmation of every deferred item named in "Explicit Exclusions" and "Escalation Mechanics," stating whether each remains accurately deferred post-implementation.
13. Engineering Decision Log.
14. Any Stop Condition encountered, and its resolution status.
15. Confirmation that ARCH-004 itself needs no amendment, or, if implementation revealed a genuine gap, a precise description routed for architectural review rather than resolved unilaterally.

Stop after this milestone. Do not begin the next Runtime Integration engineering milestone.

---

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-14 | Denver Jacobs | Initial EWO, registered per STD-001 §46. Authorizes the first implementation milestone realizing ARCH-004 — Local Actor Supervision Architecture: a new `synapse-supervisor` service component; Runtime-mediated failure routing restricted to genuine `Actor::handle()` faults; restart identity/capability/mailbox semantics; unconditional withdrawal of a `Failed` instance's Scheduler eligibility (eliminating the pre-existing failed-mailbox-drain defect); a logical, `ActorId`-keyed, cycle-free, single-parent supervision hierarchy; a minimal, non-algorithmic Restart/Stop/Escalate/Ignore decision flow; and truthful supervision audit events. Derived exclusively from ARCH-001 through ARCH-004, ADR-0015 through ADR-0017, STD-001, and the verified current state of `runtime/`, every Trusted Core crate, and `services/scheduler` at commit `5ccc7f9083a71adc6ee704b2322a701935765679`. Resolves five bounded design decisions this EWO itself identifies (crate placement; the restart-allowance constant; the fresh-behavior-on-restart capability; escalation mechanics; audit event shape), leaving open only the restart-allowance constant's literal value and the fresh-behavior capability's concrete Rust shape, per ARCH-004's own explicit deferral of numeric and implementation-level policy. |

## Disposition

Draft. Not yet reviewed. Not yet approved. Not yet implemented.
