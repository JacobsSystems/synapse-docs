---
document_id: EWO-008
title: "Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution"
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
    - ARCH-004 (v0.1.0 — architecture/ARCH-004-Local-Actor-Supervision-Architecture.md) — component-placement and identity-model precedent this EWO follows; not itself amended or implemented by this EWO
    - ARCH-005 (v0.1.0 — architecture/ARCH-005-Temporal-Runtime-Architecture.md) — the sole architectural authority this EWO implements
  adrs:
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
    - ADR-0017 (Approved)
  predecessor: EWO-007 (work-orders/EWO-007-Local-Actor-Supervision.md) — prior Runtime Integration milestone; this EWO is the first to implement ARCH-005 rather than ARCH-004
  reported_by: ER-008 (engineering-reports/ER-008-Temporal-Runtime.md, not yet created)
  base_state:
    runtime_head: 5ccc7f9083a71adc6ee704b2322a701935765679
    docs_head: e90404baa5140ce9004839bc51921c789777e003
---

# EWO-008 — Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution

Registered per STD-001 §46 (Engineering Work Orders). This is the first Engineering Work Order authorizing implementation of ARCH-005 — Temporal Runtime Architecture. It is the second Runtime Integration EWO to introduce a new service component (`Temporal Runtime`, parallel to `Scheduler` and `Supervisor`) rather than extending an existing one. This document authorizes engineering work only. It does not itself constitute approval, implementation, or completion.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-008 |
| Title | Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution |
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
| Predecessor milestone | EWO-007 — Runtime Integration: Local Actor Supervision, Failure Escalation, and Restart Ownership |
| Reported by | ER-008 (engineering-reports/ER-008-Temporal-Runtime.md, not yet created) |

No implementation exists yet against this EWO. No approval act has occurred. This document authorizes a bounded scope of future engineering work; it does not report on work already done.

---

## Engineering Authority

This implementation is governed by, in descending order:

1. ARCH-001 — Constitutional Architecture — the four constitutional guarantees and §6's naming of "time observation" among the fixed set of foundational Runtime mechanisms outside the Actor model.
2. ARCH-002 — Runtime Architecture, **v0.2.0** — the Trusted Runtime Core mechanism table (§5), the Runtime Component Model (§6), the Mailbox Model (§13), the Runtime Lifecycle state model (§15), the minimum audit-event set (§18), and the Extension and Replaceability Model (§19).
3. ARCH-003 — Runtime Integration Architecture, **v0.4.0** — the current, verified integration baseline this EWO builds on without altering.
4. ARCH-004 — Local Actor Supervision Architecture, **v0.1.0** — the component-placement, identity-model, and isolation precedent this EWO follows directly (§9, §10, §12). Not itself amended, implemented, or extended in scope by this EWO; ARCH-004's own deferred restart-backoff policy (§4, §21) remains untouched.
5. **ARCH-005 — Temporal Runtime Architecture, v0.1.0 — the specific and sole architectural authority for this milestone's scope, component placement, responsibility boundaries, timer identity model, timer lifecycle, delayed-execution requirements, admission interaction, capability interaction, mailbox interaction, causation, supervision interaction, failure semantics, clock architecture, and audit requirements.** This EWO implements ARCH-005; it does not reinterpret, extend, or narrow it.
6. ADR-0015 — Audit Emitter Failure Semantics (Approved).
7. ADR-0016 — Trusted Core Interaction Rule (Approved).
8. ADR-0017 — Bootstrap Capability Trust Root (Approved).
9. STD-001 — Documentation Standards (§46, Engineering Work Orders).

These documents are authoritative. This task implements them. It does not reinterpret or modify them. Where this EWO makes an implementation-level decision ARCH-005 deliberately left open ("Bounded Design Decisions," below), that decision is bounded, disclosed, and does not amend ARCH-005 itself.

---

## Purpose

This EWO authorizes the first implementation of the Temporal Runtime: a new `synapse-timer` service component; the Runtime-level orchestration connecting a fired timer to the existing, single admission pipeline; an `ActorId`-keyed timer identity and registration model; the full timer lifecycle ARCH-005 §12 defines; fire-time-only capability validation; ordinary mailbox and causation treatment for timer-generated messages; a monotonic clock requirement with a deterministic test seam; and truthful timer audit events — exactly as ARCH-005 establishes, and no further.

---

## Problem Statement

Verified directly against `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (`runtime/src/lib.rs`, `common/src/lib.rs`, `services/scheduler/src/lib.rs`, `services/supervisor/src/lib.rs`, `core/capability-authority/src/internal.rs`), not assumed — restating and independently re-confirming ARCH-005 §7's own verified facts:

- No timer, clock, or time-observation component exists anywhere in the workspace. An exhaustive grep for `timer|clock|Instant|SystemTime|std::time` across every `.rs` file in the workspace returns zero matches outside comments disclaiming their absence (`runtime/src/lib.rs:1252`: "No thread, timer, sleep, or... returns control to the caller").
- `Runtime::step()`/`run_until_idle` are synchronous, bounded, and caller-driven (`runtime/src/lib.rs:1107`, `schedule_next_message`); `Scheduler::select_next` (`services/scheduler/src/lib.rs`) only ever returns already-ready work. No mechanism anywhere holds work back until a future moment and later makes it ready. **This EWO is the first to introduce any notion of "not yet" into the Runtime.**
- `Message.deadline: Option<u64>` and `ExecutionContext.deadline: Option<u64>` (`common/src/lib.rs:88,144`) are opaque, copied through unread (`core/execution-coordinator/src/internal.rs:118`), and never compared or enforced anywhere. **This EWO does not read, enforce, or otherwise give meaning to either field** (ARCH-005 §3 explicitly distinguishes the Temporal Runtime from this pre-existing, unimplemented gap; closing it is out of scope here).
- Capability expiry (`core/capability-authority/src/internal.rs:204-212`, `mark_expired_for_testing`) has no production, clock-triggered mechanism — it is test-only. **This EWO does not add one.** A timer's own firing condition is Temporal Runtime's own internal state, never routed through, or confused with, Capability Authority's `Expired` concept.
- `Runtime` currently composes exactly two replaceable services alongside the Trusted Core (`runtime/src/lib.rs:585-590`: `struct Runtime { state, core, scheduler, supervisor }`). Sixteen audit-event constructors exist (`runtime/src/lib.rs:92-310`: eleven pre-existing plus five added by EWO-007); none represents a timer registration, firing, cancellation, expiry, or discard (ARCH-005 §20).
- The existing, single admission pipeline (`Runtime::admit_message`, `runtime/src/lib.rs:895-917`) performs, in order: envelope/send-authority validation, capability validity, `live_instance` resolution, Message Gateway `admit`, Actor Host `enqueue`, Scheduler `mark_ready`. **This EWO introduces no second such pipeline** — every timer-generated message passes through this exact, unmodified sequence (ARCH-005 §13, §14).

None of the above requires correction beyond what this EWO itself authorizes; each fact is cited because it directly bounds and justifies this EWO's scope.

---

## Architectural Authority

| Concern | Owner | Authority |
|---|---|---|
| Timer registration, lifecycle state, monotonic-time observation, firing proposal | **Temporal Runtime (new)** | ARCH-005 §8, §9, §10.1 |
| Capability validation for a timer-generated message (fire time only) | Capability Authority (unchanged) | ARCH-005 §14 |
| Mailbox enqueue, overflow, ordering for a timer-generated message | Actor Host, Message Gateway (unchanged) | ARCH-005 §13, §15 |
| Scheduler readiness for a timer-generated message's destination instance | Scheduler (unchanged, time-unaware) | ARCH-005 §9.2, §10.3 |
| Supervision (no interaction with Temporal Runtime of any kind) | Supervisor (unchanged) | ARCH-005 §18 |
| Causation of a timer-generated message | Runtime (never Temporal Runtime itself) | ARCH-005 §16 |
| Cross-component orchestration of the whole sequence; the only caller of Temporal Runtime | Runtime | ARCH-005 §10.2, §14; ADR-0016 Rule 1 |
| Audit-event emission (new event categories, existing emission mechanism) | Audit Emitter (unchanged trait/crate); Runtime (sole caller, as today) | ARCH-005 §20; ARCH-002 §18; ADR-0015 |
| Bootstrap Capability — never a timer-delivery fallback | Capability Authority (untouched) | ARCH-005 §14; ADR-0017 |

STD-001 §46 governs this document's own form and authority: an EWO "MAY authorize implementation... MUST NOT redefine Architecture... If implementation... reveals an apparent architectural contradiction, the EWO MUST require engineering to stop and return the issue for architectural review rather than resolve it unilaterally." This EWO's own Stop Conditions ("Definition of Failure / Stop Conditions," below) apply that rule concretely.

---

## Objective

Implement the Temporal Runtime exactly as ARCH-005 defines it: a new, narrow `synapse-timer` service component, composed by Runtime alongside Scheduler and Supervisor, outside Trusted Core; an `ActorId`-keyed timer registration and lifecycle model (`Registered → Waiting → Fired → {Discarded | Completed}`, plus `Cancelled` and `Expired`); Runtime-mediated construction and submission of a fired timer's resulting message through the existing, unmodified admission pipeline; fire-time-only capability validation; ordinary mailbox, overflow, and causation treatment; a monotonic-clock requirement with a deterministic test seam; and truthful timer audit events. No cron, calendar, backoff, retry, persistence, or distributed mechanism of any kind.

---

## Bounded Design Decisions

ARCH-005 fixes architecture; it does not select implementation-level values or mechanisms it explicitly deferred. This EWO resolves exactly the following, and no other open decision is authorized:

### 1. Temporal Runtime crate placement and dependency set — resolved, not open

`Temporal Runtime` is a new workspace member, `services/timer` (`synapse-timer`), mirroring `services/scheduler` (`synapse-scheduler`) exactly: a Runtime-composed service, outside Trusted Core, depending on `synapse-common` only. Unlike `synapse-supervisor` (which required a narrow `synapse-api` dependency for its fresh-behavior-on-restart factory, ER-007), Temporal Runtime constructs no `Box<dyn Actor>` value and requires no dependency beyond `synapse-common` for the `ActorId` and `Message` types it needs. This is not a genuinely open choice — it is the direct, mechanical consequence of ARCH-005 §9.1/§24's own reasoning that no dependency beyond `synapse-common` is architecturally required.

### 2. Expiry condition — resolved, not open

ARCH-005 §12 describes `Expired` as covering both "target already unreachable at registration time" and, more loosely, "some other bounded, implementation-defined validity-window rule." This EWO resolves this to the narrower of the two: **a registration enters `Expired` only when its target `ActorId`'s live instance cannot be resolved at the moment of registration** (mirroring the existing `live_instance` resolution failure `admit_message` already performs, `runtime/src/lib.rs:905`). No periodic re-check, timeout duration, or validity-window mechanism of any kind is introduced by this milestone — doing so would risk introducing exactly the numeric timing policy ARCH-005 §4/§22 and this EWO's own "Explicit Exclusions" prohibit. A registration whose target becomes unreachable only *after* registration (for example, its actor Stops before the timer fires) is handled by §12's own Stop-removal rule (Bounded Design Decision 4), not by this expiry path.

### 3. Firing-check invocation — open, bounded

ARCH-005 does not, and this EWO does not, modify `Runtime::step()` or `Runtime::run_until_idle()`'s existing signatures or behavior (Architecture Constraints, below) — both remain byte-for-byte unchanged. **This EWO requires exactly one new, narrowly scoped public `Runtime` method** (name left to implementation, mirroring `register_supervision`'s own precedent, EWO-007) that: queries Temporal Runtime for every registration whose condition is currently satisfied, given the Runtime-supplied current monotonic time; for each, constructs the resulting `Message` (Runtime-owned causation, ARCH-005 §16) and submits it through the existing, unmodified admission pipeline; and emits the audit events "Audit Semantics" (below) fixes. This method is the sole point at which Temporal Runtime's proposals become Runtime action — it is never invoked automatically by `step()` or `run_until_idle()` themselves, on the same "no existing public method signature changes" discipline EWO-007 already established.

### 4. Stop-removal mechanism — resolved, not open

Per ARCH-005 §23's own normative decision ("Stop / Terminated MUST cause Runtime to remove the corresponding timer registrations proactively"): the existing `Runtime::terminate_actor_instance` and the Stop-decision path inside Supervisor-mediated restart handling (`runtime/src/lib.rs`, EWO-007's own `execute_stop_decision`) each additionally invoke Temporal Runtime's own cancellation operation for the terminated `ActorId`, on the same best-effort basis `terminate_actor_instance` already uses for `scheduler.remove` (`runtime/src/lib.rs:833`, `let _ = self.scheduler.remove(instance)`). This is resolved, not open, because ARCH-005 §23 already states the requirement normatively; only the mechanical wiring (which existing Runtime call sites invoke it) is implementation detail.

### 5. Clock seam — open, bounded

ARCH-005 §19 requires "a single, substitutable notion of 'now'" without specifying its shape. **This EWO requires**: production behavior sourced from genuine monotonic host time; a deterministic seam allowing tests to control perceived elapsed time without any real waiting or sleeping (mirroring this workspace's existing wholly synchronous, instantaneous test character — no test anywhere in this workspace sleeps or waits on a real clock, and this EWO introduces none); and that wall-clock time never determines firing or firing order (ARCH-005 §19, §23). The concrete mechanism (an injected clock abstraction, a caller-supplied "now" value per call, or another substitutable seam) is left to implementation.

No other design decision is authorized to remain open. If implementation determines a materially different decision is required, this is a stop condition ("Definition of Failure / Stop Conditions," below), not something to resolve unilaterally.

---

## Scope

Implement only the following, each directly required by ARCH-005:

### Temporal Runtime component (ARCH-005 §8, §9, §10)

- A new crate, `synapse-timer` (`services/timer`), depending on `synapse-common` only.
- Owns, and exposes only: registration of a timer against an `ActorId` (with whatever data is needed to determine firing and to later construct the resulting `Message`, Bounded Design Decision 5); cancellation of an existing registration; a query, given the current monotonic time, for every registration whose condition is now satisfied; and Temporal Runtime's own internal lifecycle-state bookkeeping (§12).
- Holds no dependency on `synapse-actor-host`, `synapse-lifecycle-guardian`, `synapse-capability-authority`, `synapse-execution-coordinator`, `synapse-message-gateway`, `synapse-scheduler`, or `synapse-supervisor` — mirroring Scheduler's and Supervisor's own existing, analogous isolation.

### Runtime coordination (ARCH-005 §10.2, §14; ADR-0016)

- `Runtime` gains exactly one new field (a Temporal Runtime handle), constructed alongside the existing `scheduler` and `supervisor` fields in `Runtime::bootstrap_with_config`.
- Runtime remains the only caller of Temporal Runtime's operations, and the only component that constructs and submits a fired timer's resulting message. No new direct component-to-component call is introduced anywhere (see "Component Boundaries and Prohibited Interactions," below).

### Timer identity (ARCH-005 §11)

- Registrations keyed by `ActorId`, never `ActorInstanceId`.
- Restart preserves registrations as a free consequence of `ActorId`-keying — no restart-specific timer code is introduced or required.
- Stop/Terminated causes proactive removal (Bounded Design Decision 4).

### Timer lifecycle (ARCH-005 §12)

- Exactly the seven states ARCH-005 §12 defines: `Registered`, `Waiting`, `Fired`, `Cancelled`, `Expired`, `Discarded`, `Completed` — no additional state.
- Exactly the legal transitions ARCH-005 §12 defines; every other transition is rejected.

### Delayed execution (ARCH-005 §13)

- A fired timer never executes an actor and never dispatches — it produces an ordinary `Message`, constructed and submitted by Runtime through the existing, unmodified admission pipeline (Bounded Design Decision 3).

### Capability interaction (ARCH-005 §14)

- Capability validation for a timer-generated message occurs only as part of the ordinary admission sequence, at fire time — never at registration time, never cached, never independently performed by Temporal Runtime.

### Mailbox interaction (ARCH-005 §15)

- A fired timer's resulting message is subject to the same bounded mailbox capacity, overflow rejection, and FIFO ordering as any other message — no exemption, no timer-specific mailbox.

### Causation (ARCH-005 §16)

- Runtime, never Temporal Runtime, sets the resulting message's causation, identifying the firing registration.

### Audit (ARCH-005 §20)

- New `event_type` values, emitted only by Runtime (the existing, sole caller of Audit Emitter), for: timer registered; timer cancelled; timer fired; timer expired; timer discarded; timer completed. Reuse the existing `message.admitted`/`message.rejected` events for delivery outcome (ARCH-005 §20 — "not a new, parallel pair").

---

## Explicit Exclusions

Do NOT implement, and do not let any of the following creep into this milestone's scope:

- Persistence, durable timers, or any timer state surviving a whole Runtime-process restart.
- A workflow engine or generalized effect-scheduling system.
- Message retry, redelivery, or acknowledgement protocols; retry policies of any kind.
- Delayed or scheduled restart, or any Supervisor restart-backoff policy (ARCH-004 §4, §21 remains deferred, unaffected by this EWO).
- Backoff algorithms, timing, or jitter of any kind.
- Cron scheduling, calendar scheduling, or recurring/repeating timers.
- Time-zone handling of any kind.
- Distributed clocks, remote timers, or clustering.
- Deadline propagation, or any change to the existing, unimplemented `Message.deadline`/`ExecutionContext.deadline` fields.
- Mailbox persistence, dead-letter queues, event sourcing, or checkpoint/snapshot recovery of any kind.
- Any Rust API redesign of `Actor`, `ActorHost`, `ExecutionCoordinator`, `LifecycleGuardian`, `MessageGateway`, `CapabilityAuthority`, `Scheduler`, `Supervisor`, or any `synapse-common` shared type.
- A public scheduling DSL, or any generic, configurable timing-policy framework of any kind.
- Distributed time synchronization of any kind.
- Any modification to `synapse-lifecycle-guardian`, `synapse-actor-host`, `synapse-capability-authority`, `synapse-execution-coordinator`, `synapse-message-gateway`, `synapse-audit-emitter`, `synapse-host-adapter`, `synapse-scheduler`, or `synapse-supervisor` (Problem Statement already establishes none is required; see Repository Constraints).
- Any change to `AuditEvent`'s field shape in `synapse-common`.
- A new `RuntimeError` variant, unless implementation proves one is genuinely unavoidable — in which case, prefer reusing an existing variant (`UnknownTarget` for an unreachable registration target, `IllegalTransition` for an illegal timer-lifecycle transition) before introducing anything new, on the same basis EWO-007's own `RuntimeError::AmbiguousAuthority` precedent required narrow justification.
- SDK or host-specific implementation work.
- Any architecture, ADR, or STD-001 document change, including ARCH-005 itself.

---

## Trusted Core

The seven Trusted Core components (ARCH-002 §6) are unchanged in count, responsibility, and boundary. `Temporal Runtime` is explicitly **not** Trusted Core — it is a Runtime-composed service, positioned exactly as `Scheduler` and `Supervisor` already are (ARCH-005 §9.1). No existing Trusted Core component's responsibility is transferred, absorbed, or duplicated. Per the Problem Statement's own verification, this milestone requires **zero source changes** to `synapse-lifecycle-guardian`, `synapse-actor-host`, `synapse-capability-authority`, `synapse-execution-coordinator`, `synapse-message-gateway`, `synapse-audit-emitter`, or `synapse-host-adapter` — every one of the seven Trusted Core crates is touched by this EWO in exactly zero lines. `synapse-scheduler` and `synapse-supervisor` likewise require zero source changes. Only `runtime/src/lib.rs` (composition root) and the new `services/timer` crate change.

---

## Component Boundaries and Prohibited Interactions

The following direct interactions MUST NOT be introduced by this milestone (ADR-0016 Rule 2, extended without exception to the new participant):

| Prohibited direct call | Why |
|---|---|
| Temporal Runtime → Scheduler | A fired timer's resulting message becomes Scheduler-ready only through the existing, unmodified admission pipeline (`admit_message`), which Runtime invokes — never a direct call from Temporal Runtime. |
| Temporal Runtime → Lifecycle Guardian | Temporal Runtime has no concept of, or path to, actor lifecycle state; the existing `live_instance`/admission-time resolution already handles a target actor's current reachability. |
| Temporal Runtime → Actor Host | Mailbox enqueue and instance resolution occur only through the existing admission pipeline, invoked by Runtime — Temporal Runtime never touches Actor Host directly. |
| Temporal Runtime → Execution Coordinator | Temporal Runtime never dispatches or executes; it only proposes that a moment has arrived (ARCH-005 §13). |
| Temporal Runtime → Capability Authority | Capability validation for a timer-generated message occurs only through the existing admission pipeline, at fire time (ARCH-005 §14) — Temporal Runtime holds no capability state and never validates anything itself. |
| Scheduler → Temporal Runtime | Scheduler remains time-unaware, unchanged (`services/scheduler/src/lib.rs`); this milestone does not add this dependency in either direction. |
| Supervisor → Temporal Runtime | No interaction exists or is required between the two (ARCH-005 §18); timer continuity across restart arises solely from shared `ActorId`-keying, never coordination. |
| Lifecycle Guardian → Temporal Runtime | Lifecycle Guardian remains unaware Temporal Runtime exists, on the same basis it remains unaware Supervisor exists (ADR-0016). |
| Any actor (via `Actor::handle()`'s own emitted messages) → Temporal Runtime | Ordinary actor messaging MUST NOT become a channel for registering, cancelling, or otherwise manipulating a timer directly — timer registration is reachable only through Runtime's own internal orchestration, never through a message an actor's own handler emits (mirroring ARCH-004 §10.2/§13's identical prohibition for supervision). |

Runtime remains the only bridge for every one of these pairs, exactly as it already is for every existing Trusted-Core-to-Trusted-Core sequence and for Scheduler/Supervisor (ADR-0016 Rule 1).

---

## Architecture Constraints

Do not:

- introduce new Runtime concepts beyond `Temporal Runtime` itself and the orchestration ARCH-005 already specifies;
- invent lifecycle states beyond the seven ARCH-005 §12 defines, or a new `ActorState`/`RuntimeState` variant;
- modify constitutional concepts;
- reinterpret ARCH-001, ARCH-002, ARCH-003, ARCH-004, or ARCH-005;
- redesign `ActorHost`, `LifecycleGuardian`, `ExecutionCoordinator`, `MessageGateway`, `CapabilityAuthority`, `Scheduler`, or `Supervisor`'s public trait in any way;
- redesign `Runtime`'s existing public method signatures (`submit_message`, `execute_message`, `step`, `run_until_idle`, `create_actor_instance_with_behavior`, `terminate_actor_instance`, `register_supervision`, and every other existing public method keep their exact current signatures);
- introduce external dependencies;
- use unsafe Rust;
- give `synapse-timer` a dependency on anything beyond `synapse-common`;
- give any existing Trusted Core crate, `synapse-scheduler`, or `synapse-supervisor` a new dependency on `synapse-timer`, or the reverse, in either direction (only `runtime` may depend on `synapse-timer`, exactly as only `runtime` depends on `synapse-scheduler` and `synapse-supervisor` today);
- introduce a generic, configurable scheduling-policy, backoff, or timing-threshold mechanism of any kind;
- give the resulting `Message`/`ExecutionContext.deadline` fields any new meaning or enforcement.

If implementation requires any of the above: STOP. Produce an Engineering Report explaining the issue. Do not invent a solution.

---

## Repository Constraints

Permitted changes, and only these:

- New crate: `services/timer/` (`synapse-timer`), its `Cargo.toml`, `src/lib.rs`, `src/internal.rs` (mirroring `services/scheduler`'s and `services/supervisor`'s own file layout), and a new `README.md`.
- Root `Cargo.toml`: add `services/timer` to `[workspace] members` and `synapse-timer = { path = "services/timer" }` to `[workspace.dependencies]`, mirroring the existing `synapse-scheduler`/`synapse-supervisor` entries exactly.
- `runtime/Cargo.toml`: add `synapse-timer = { workspace = true }`, mirroring the existing `synapse-scheduler`/`synapse-supervisor` entries.
- `runtime/src/lib.rs`: the new `Runtime` field, its construction in `bootstrap_with_config`, the new firing-check public method (Bounded Design Decision 3), the Stop-path cancellation wiring (Bounded Design Decision 4), the new audit-event constructors, and any new, narrowly scoped private helper methods "Required Interface Evolution" (below) requires.
- `runtime/README.md`: documentation of the newly implemented Temporal Runtime behavior only.
- New tests: within `synapse-timer`'s own test module, and within `runtime`'s existing `#[cfg(test)]` module and/or `runtime/tests/`, matching current convention.

Do not modify: governance documents; architecture documents (including ARCH-005 itself); standards; ADRs; work orders other than this one; `common/src/lib.rs` (unless Stop Condition 2, below, is genuinely triggered); any of the seven Trusted Core crates; `services/scheduler`, `services/supervisor`, `services/actor-directory`, `services/audit-pipeline`, or `services/persistence`; `api/`; any existing example or existing test file's own assertions (existing tests may gain new, additive test *functions* proving regression, but no existing test's own body or expected outcome may change — see "Required Tests," below).

---

## Required Interface Evolution

**`synapse-timer` (new crate) — minimum public surface, names recommended, not mandated:**

1. **Register.** Accepts a target `ActorId` and whatever data is needed to determine firing (a monotonic-time-relative condition, per Bounded Design Decision 5) and to later construct the resulting `Message`. Enters `Registered`/`Waiting`, or `Expired` immediately if the target is unreachable at registration time (Bounded Design Decision 2).
2. **Cancel.** Accepts a previously registered timer's identity (or its target `ActorId`, if this milestone's own registration model ties one timer per `ActorId` — implementation's choice, provided it satisfies ARCH-005 §11's identity model); transitions the registration to `Cancelled` if it has not already fired.
3. **Query due.** Accepts the current monotonic time (Bounded Design Decision 5); returns every registration whose condition is now satisfied, transitioning each to `Fired`.

**`Runtime` (existing crate) — minimum change:**

- One new field on the `Runtime` struct (a Temporal Runtime handle), constructed in `bootstrap_with_config` alongside the existing `scheduler: SchedulerHandle` and `supervisor: SupervisorHandle` fields. `Runtime`'s own public constructors (`bootstrap`, `bootstrap_with_config`) keep their exact existing signatures — this is an internal composition change only.
- One new public method (Bounded Design Decision 3) that queries Temporal Runtime for due registrations and, for each, constructs the resulting `Message` (Runtime-owned causation, ARCH-005 §16) and submits it through the existing, unmodified `admit_message`/`submit_message` path, emitting the audit events "Audit Semantics" (below) fixes.
- The existing `Runtime::terminate_actor_instance` and the existing Stop-decision execution path (`execute_stop_decision`, introduced by EWO-007) each additionally invoke Temporal Runtime's cancellation operation for the terminated `ActorId` (Bounded Design Decision 4), on a best-effort basis mirroring `scheduler.remove`'s own existing pattern.
- No other existing public Runtime method changes signature.

No change to `synapse-common`, `synapse-actor-host`, `synapse-lifecycle-guardian`, `synapse-capability-authority`, `synapse-execution-coordinator`, `synapse-message-gateway`, `synapse-audit-emitter`, `synapse-host-adapter`, `synapse-scheduler`, or `synapse-supervisor` is required or authorized.

---

## Runtime Sequencing

The new firing-check method (Bounded Design Decision 3) performs exactly this sequence, and only when explicitly invoked by a caller:

1. Runtime supplies the current monotonic time (Bounded Design Decision 5) to Temporal Runtime's query-due operation, which transitions every satisfied registration to `Fired` and returns them.
2. For each `Fired` registration, in the order Temporal Runtime returns them: emit `timer.fired` (audited before any admission attempt, ARCH-005 §20); construct the resulting `Message`, with causation set by Runtime to identify the firing registration (ARCH-005 §16, never the registration's own unverified self-assertion); submit it through the existing, unmodified `admit_message`/`submit_message` sequence.
3. On successful admission: the existing `message.admitted` event fires (unmodified); the registration transitions to `Completed`; emit `timer.completed`.
4. On admission failure (capability denial, mailbox overflow, unknown destination, or any other admission-time rejection): the existing `message.rejected` event fires (unmodified); the registration transitions to `Discarded`; emit `timer.discarded`. This is **not** routed to Supervisor under any circumstance (ARCH-005 §17; EWO-007's own established "admission failures never reach supervision" invariant, extended without modification to this new message origin).
5. Registrations processed independently — one registration's admission outcome never affects another's (mirroring `process_emitted_messages`'s own existing "each message is processed sequentially, in vector order, fully independently" discipline, `runtime/src/lib.rs`).

---

## Failure Semantics

- **Temporal Runtime's own registration, cancellation, and query-due operations do not fail under ordinary operation** beyond the one legitimate case Bounded Design Decision 2 already resolves (registration against an unreachable target, entering `Expired` immediately rather than returning an error) — if implementation finds a genuine failure mode beyond this, it is a Stop Condition (below), not something to resolve by inventing a new variant.
- **A fired registration's resulting message retains the existing admission pipeline's own failure semantics unchanged.** Whatever `RuntimeError` `admit_message`/`submit_message` already returns for a given rejection cause is exactly what is observed here; no new failure mode is introduced by the fact that the message originated from a timer rather than an external caller or an actor emission.
- **Audit-emission failure for any new timer event follows ADR-0015 unchanged:** the operation reporting that event fails if the emission fails, with no rollback of already-committed component-level state (the same rule every existing mandatory audit obligation in this codebase already follows).
- **No new failure mode is introduced beyond what this section defines.**

---

## Delayed Execution Mechanics

Restated precisely, binding on implementation (ARCH-005 §11, §13, §14, §15, §16):

1. A timer registration is keyed by `ActorId`. Restart preserves it automatically; no restart-specific timer code exists (ARCH-005 §11).
2. A fired timer never executes an actor and never dispatches. It produces exactly one ordinary `Message`, submitted by Runtime through the existing, unmodified admission pipeline (ARCH-005 §13).
3. Capability validation for that message occurs only as part of that ordinary admission sequence, at fire time — never cached, never performed by Temporal Runtime (ARCH-005 §14).
4. Mailbox enqueue, bounded capacity, overflow rejection, and FIFO ordering apply exactly as for any other message — no exemption (ARCH-005 §15).
5. Causation is set by Runtime, identifying the firing registration — never self-asserted by Temporal Runtime (ARCH-005 §16).
6. Stop/Terminated causes Runtime to proactively cancel the corresponding registration (Bounded Design Decision 4) — a registration is never left to fail silently, indefinitely, at every future (non-existent) firing attempt against a stopped actor.

---

## Audit Semantics

Truthful ordering, exactly as ARCH-005 §20 fixes it, and not reorderable:

```text
1. Timer registered (new event, at registration)
        |
        v
2. Timer cancelled / expired, OR:
        |
        v
3. Timer fired (new event — observation, before any admission attempt)
        |
        v
4. Resulting Message submitted through the existing, unmodified
   admission pipeline (message.admitted / message.rejected — existing,
   unmodified by this EWO)
        |
        v
5. Timer completed (successful admission), OR timer discarded
   (admission failure) — new events, naming the truthful outcome
```

No audit record may claim a timer has fired before it genuinely has, and no record may claim delivery succeeded before admission genuinely returns success (ARCH-005 §20). No new `AuditEvent` field is introduced — every new fact is a distinct `event_type` string value on the existing, unmodified shape.

---

## Definition of Done

The task is complete only if all of the following are true:

- A new `synapse-timer` crate exists, depending on `synapse-common` only, exposing registration, cancellation, and query-due operations per "Required Interface Evolution."
- `Runtime` composes Temporal Runtime alongside Scheduler and Supervisor, with no other existing public Runtime method signature changed.
- A fired timer's resulting message is constructed and submitted only through the existing, unmodified admission pipeline — no alternate execution, dispatch, mailbox-insertion, or admission path exists anywhere.
- Timer registrations are keyed by `ActorId`; restart preserves them without any restart-specific code; Stop/Terminated causes proactive removal.
- Exactly the seven lifecycle states ARCH-005 §12 defines exist, with exactly its legal transitions; every other transition is rejected.
- Capability validation for a timer-generated message occurs only at fire time, through the existing pipeline; Temporal Runtime owns no capability state.
- Timer delivery failures are ordinary admission failures and never reach Supervisor.
- Timer registration, firing, cancellation, expiry, completion, and discard each produce truthful, correctly ordered audit events, using the existing, unmodified `AuditEvent` shape.
- Firing is determined by monotonic time only; a deterministic test seam exists; wall-clock time never determines firing or ordering.
- No pre-existing actor, test, or demonstration (`runtime/tests/worker_pool.rs`, `runtime/tests/actor_to_actor_messaging.rs`, `runtime/tests/actor_supervision.rs`, `runtime/examples/*`) is required to register a timer, and none of their existing assertions change.
- No Trusted Core crate's source changes. Trusted Core remains exactly seven components. `synapse-scheduler` and `synapse-supervisor` remain byte-for-byte unchanged.
- No public interface of any existing crate changes signature.
- `synapse-timer`'s dependency set is exactly `synapse-common`.
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

All must pass. Additionally verify: workspace member count increases by exactly one (`services/timer`); zero `unsafe`; no dependency cycle; `cargo tree -p synapse-timer` shows exactly `synapse-common`; no existing crate other than `runtime` depends on `synapse-timer`; every existing public method signature named in "Architecture Constraints" is byte-for-byte unchanged; the existing demonstrations (`cargo run --example worker_pool`, `cargo run --example actor_to_actor_messaging`) continue to run to completion with unchanged output. If a new Temporal Runtime demonstration is added, it must be deterministic (using the required test-clock seam, Bounded Design Decision 5, never a real wall-clock sleep), minimal, and limited to this EWO's own approved scope.

---

## Data and State Constraints

- Temporal Runtime's own internal state: a per-registration target `ActorId`; a per-registration firing condition (monotonic-time-relative); a per-registration lifecycle state (§12's seven states); whatever data is needed to construct the resulting `Message` at fire time. No other state.
- No new field is added to `Message`, `ActorId`, `ActorInstanceId`, `ExecutionContext`, `AuditEvent`, or any other `synapse-common` type.
- No new field is added to `TrustedCore`. Exactly one new field is added to `Runtime` itself (the Temporal Runtime handle), alongside the existing `scheduler` and `supervisor` fields.
- Do not add: a persistence layer, a configuration struct, a generic scheduling-policy registry, or any bookkeeping beyond what this section names.

---

## Definition of Failure / Stop Conditions

Stop immediately, and produce an Engineering Report rather than resolving the issue unilaterally, if any of the following occurs:

1. Delayed execution is found to require any modification to `synapse-lifecycle-guardian`, `synapse-actor-host`, `synapse-capability-authority`, `synapse-scheduler`, or `synapse-supervisor` beyond the existing, unmodified operations this EWO already names.
2. `AuditEvent`'s existing shape is found to be insufficient to truthfully represent a required timer fact without inventing a new field.
3. Any existing public method signature (`Runtime`, `ActorHost`, `LifecycleGuardian`, `ExecutionCoordinator`, `MessageGateway`, `CapabilityAuthority`, `Scheduler`, `Supervisor`) is found to require a change, including `step()` or `run_until_idle()`.
4. A direct dependency between `synapse-timer` and any Trusted Core crate, `synapse-scheduler`, or `synapse-supervisor` is found to be required in either direction.
5. Firing is found to require any interaction with Supervisor, or Supervisor is found to require any interaction with Temporal Runtime, to satisfy this EWO's own Definition of Done (it does not — ARCH-005 §18 already establishes no such interaction exists or is required).
6. The clock seam (Bounded Design Decision 5) is found to require exposing wall-clock time as a basis for firing or ordering.
7. Any existing test's own assertions are found to require modification (only additive new tests are authorized).
8. Any ADR guarantee or ARCH-002/ARCH-003/ARCH-004/ARCH-005 architectural boundary is found to require change to complete this milestone.
9. The expiry condition (Bounded Design Decision 2) is found to require a genuine timing/validity-window mechanism beyond registration-time unreachability to satisfy this EWO's own Definition of Done.

Do not resolve these issues yourself. Report them.

---

## Implementation Sequence

1. Baseline verification: re-confirm the current workspace state matches this EWO's Problem Statement exactly.
2. Create `services/timer` (`synapse-timer`); register it in root `Cargo.toml`.
3. Implement Temporal Runtime's registration, cancellation, and query-due operations, per "Required Interface Evolution," including the expiry condition (Bounded Design Decision 2) and the clock seam (Bounded Design Decision 5); record the chosen clock-seam shape and its rationale for ER-008.
4. Component-local tests within `synapse-timer` proving registration, cancellation, expiry, lifecycle-transition legality, and due-query determinism in isolation ("Required Tests," below).
5. Add the new `Runtime` field and its construction in `bootstrap_with_config`; add the new firing-check public method (Bounded Design Decision 3) and the Stop-path cancellation wiring (Bounded Design Decision 4).
6. Implement the new firing-check sequence, per "Runtime Sequencing," reusing the existing, unmodified `admit_message`/`submit_message` path.
7. Implement the new audit-event constructors in `runtime/src/lib.rs`, reusing `message.admitted`/`message.rejected` where "Runtime Sequencing" and "Audit Semantics" specify.
8. Runtime-level tests proving: identity/restart/stop semantics; ordinary-message-generation and admission-pipeline reuse; fire-time-only capability validation; ordinary mailbox/overflow/causation treatment; audit ordering; and the absence of any Supervisor interaction ("Required Tests," below).
9. Regression pass: existing actor-to-actor messaging, scheduler, supervision, capability, and worker-pool tests and demonstrations, unmodified, all passing.
10. Documentation updates: `services/timer/README.md` (new) and `runtime/README.md`, strictly limited to describing the newly implemented behavior.
11. Complete quality validation, per "Mandatory Validation."
12. ER-008 preparation after implementation and validation are complete — not before, and not as part of this EWO.

Do not include work from later milestones.

---

## Acceptance Criteria

Every criterion is objective and testable:

- Exactly one new crate (`synapse-timer`) and `runtime/`'s own composition-root file change; no Trusted Core crate's source changes; `synapse-scheduler` and `synapse-supervisor` remain byte-for-byte unchanged.
- `synapse-timer`'s dependency set is exactly `synapse-common`.
- No existing public method signature changes anywhere in the workspace, including `step()` and `run_until_idle()`.
- A fired timer's resulting message reaches admission only through the existing, unmodified pipeline — proven by regression and dedicated tests.
- Timer delivery failures never reach Supervisor — proven by regression test.
- Restart preserves timer registrations with no restart-specific code; Stop/Terminated removes them, proven by dedicated tests.
- Fire-time-only capability validation matches "Delayed Execution Mechanics" exactly.
- Timer registration, firing, cancellation, expiry, completion, and discard each produce the exact audit ordering "Audit Semantics" fixes.
- Firing is deterministic under the required test-clock seam; no test sleeps or waits on a real clock.
- All pre-existing tests pass unmodified in outcome; all pre-existing demonstrations run unchanged.
- New tests, per "Required Tests," are added and pass.
- `cargo fmt --all -- --check`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`, `cargo build --workspace`, and `cargo test --workspace` all pass with zero warnings.
- `git status` after implementation shows changes confined to `services/timer/`, root `Cargo.toml`, `runtime/Cargo.toml`, `runtime/src/lib.rs`, `runtime/README.md`, and new test files — no unrelated file touched.
- Architecture and ADR documents remain byte-for-byte untouched.

---

## Required Tests

### Timer registration (`synapse-timer`)

- Registration succeeds for a target `ActorId` with a live instance.
- Registration against an unreachable target immediately enters `Expired` (Bounded Design Decision 2), never `Registered`/`Waiting`.
- Cancellation of a `Waiting` registration transitions it to `Cancelled`; a `Cancelled` registration never later reports as due.
- Illegal lifecycle transitions (for example, attempting to cancel an already-`Fired`, `Discarded`, or `Completed` registration) are rejected without mutating existing state.
- Querying due registrations against a supplied monotonic time returns exactly those whose condition is satisfied, transitioning each to `Fired`, and leaves every other registration's state untouched.

### Identity

- A timer registered against an `ActorId` remains valid, unaffected, after that actor is restarted (same `ActorId`, new `ActorInstanceId`) — proven by a Runtime-level test combining supervision-triggered restart with a still-pending timer.
- A timer's eventual firing, after restart, delivers to whichever instance is currently live for its `ActorId` — proving replacement continuity.
- Stopping/terminating the target actor removes the corresponding registration (Bounded Design Decision 4) — proven by confirming it no longer reports as due even if its original condition would otherwise be satisfied.

### Message generation

- A fired timer's resulting message is constructed and submitted through the same `admit_message`/`submit_message` sequence any other message uses — proven by a test asserting identical admission behavior (same rejection causes, same acceptance path) for a timer-generated message as for an externally submitted one.
- Causation on the resulting message identifies the firing registration, set by Runtime, never left as an unverified self-assertion.
- Mailbox ordering (FIFO admission order) is preserved for a fired timer's message relative to concurrently admitted ordinary messages.

### Capability

- A fired timer whose target's bound capability remains valid is admitted successfully.
- A fired timer whose target's bound capability was revoked before firing is discarded via the ordinary admission-rejection path — proving capability validation is fire-time-only, never cached from registration time.
- A fired timer's own registration succeeds even before any capability exists for its target, proving no registration-time validation occurs.
- No test seeds or asserts any capability check performed directly by Temporal Runtime itself — Temporal Runtime holds no capability-validation logic at all.

### Mailbox

- A fired timer's message is rejected with the existing `Overflow` cause when its target's mailbox is already at capacity — proving no exemption exists.
- No direct mailbox insertion occurs — proven by confirming a fired timer's message is only ever observed in the mailbox after passing through Message Gateway's `admit` and Actor Host's `enqueue`, never appearing before that sequence completes.

### Runtime composition

- Runtime remains the sole caller of every Temporal Runtime operation — proven by the crate's own isolation (no dependency on any component crate) plus a Runtime-level test confirming Temporal Runtime is reachable only through the new firing-check method.
- Scheduler's own isolation is unaffected: `synapse-scheduler`'s dependency set remains unchanged, and it gains no dependency on `synapse-timer`.
- Supervisor's own isolation is unaffected: `synapse-supervisor`'s dependency set remains unchanged, and it gains no dependency on `synapse-timer`; no test proves or requires any direct interaction between the two.

### Audit

- Timer registered, fired, cancelled, expired, completed, and discarded each produce their own distinct, correctly ordered event — provable via `RecordingAuditEmitter`, the same test double this workspace's existing tests already use.
- Ordering: for a successfully delivered timer, the recorded sequence matches "Audit Semantics" exactly (registered, then fired, then the existing message.admitted, then completed).
- Ordering: for a discarded timer, the recorded sequence matches "Audit Semantics" exactly (registered, then fired, then the existing message.rejected, then discarded).

### Clock

- Firing order is proven deterministic and monotonic under the required test-clock seam — two registrations with different conditions fire in the order their conditions are satisfied, never in wall-clock creation order if that would differ.
- The test-clock seam allows a test to assert "not yet due" and "now due" deterministically, without any real sleep or wait.
- No test exercises or depends on real wall-clock time in any way.

### Regression

- All Scheduler regressions (`services/scheduler`'s own test module) continue to pass unmodified; its dependency set is unchanged.
- All Supervision regressions (`services/supervisor`'s own test module, `runtime/tests/actor_supervision.rs`) continue to pass unmodified.
- All Actor-to-Actor Messaging regressions (`runtime/tests/actor_to_actor_messaging.rs`) continue to pass unmodified.
- All Capability regressions (`core/capability-authority`'s own test module) continue to pass unmodified; `synapse-capability-authority`'s own source is byte-for-byte unchanged.
- All Admission regressions (existing `admit_message`/`submit_message` tests within `runtime`) continue to pass unmodified.
- All Worker Pool regressions (`runtime/tests/worker_pool.rs`, `runtime/examples/worker_pool.rs`) continue to pass/run unmodified, with identical output.

### Not required

- Any test proving cron, calendar, or recurring-timer behavior — no such mechanism is implemented.
- Any test proving persistence or durability of a timer registration across a Runtime-process restart — explicitly out of scope.
- Any test proving distributed or cross-host timer behavior — explicitly out of scope.
- Any test proving numeric backoff or timing-policy behaviour beyond the single fixed expiry condition (Bounded Design Decision 2) — none exists to test.

---

## Engineering Decision Log

Record:

- implementation decisions (expected: the chosen clock-seam shape and its rationale; the concrete Rust shape chosen for a timer's firing condition and resulting-message-construction data);
- repository decisions (expected: `services/timer` file layout, mirroring `services/scheduler` and `services/supervisor`);
- deferred decisions (expected: none beyond what this EWO itself already defers, per "Explicit Exclusions");
- architectural decisions (expected: None — ARCH-005 is implemented, not amended);
- constitutional decisions (expected: None);
- future work enabled (expected: a genuine, testable local-timer mechanism a future durable-timer, workflow, or Supervisor restart-backoff milestone can build on, per ARCH-005 §21);
- future work deferred (expected: persistence/durable timers; workflow-engine and effect-scheduling implementation; cron/calendar scheduling; distributed/remote timers; Supervisor restart-backoff policy; every item ARCH-005 §4/§22/§24 already lists, unaffected in status by this EWO).

---

## Completion Report

ER-008 must provide, after implementation:

1. Files created (expected: `services/timer/` in full, plus new tests within `runtime`).
2. Files modified (expected: root `Cargo.toml`, `runtime/Cargo.toml`, `runtime/src/lib.rs`, `runtime/README.md`).
3. The chosen clock-seam shape and its rationale.
4. The concrete Rust shape chosen for a timer's firing condition and resulting-message-construction data, and why.
5. Confirmation, re-verified against source, that all seven Trusted Core crates, `synapse-scheduler`, and `synapse-supervisor` remain byte-for-byte unchanged.
6. Temporal Runtime and Runtime behaviour implemented, confirmed against "Delayed Execution Mechanics," "Runtime Sequencing," and "Audit Semantics."
7. Tests added, mapped against "Required Tests."
8. Validation results (Mandatory Validation, in full).
9. Dependency changes (expected: exactly one new crate, `synapse-timer`, depended on by `runtime` only).
10. Trusted Core changes (expected: none — same seven components, same boundaries).
11. Architecture changes (expected: none).
12. Explicit confirmation of every deferred item named in "Explicit Exclusions," stating whether each remains accurately deferred post-implementation.
13. Engineering Decision Log.
14. Any Stop Condition encountered, and its resolution status.
15. Confirmation that ARCH-005 itself needs no amendment, or, if implementation revealed a genuine gap, a precise description routed for architectural review rather than resolved unilaterally.
16. A requirement-by-requirement traceability matrix linking ARCH-005 → this EWO → implementation → tests → the independent implementation review → ER-008, mirroring EWO-007/ER-007's own precedent.

An independent implementation review, on the same basis EWO-007's own implementation received, is expected before ER-008 is finalized — this EWO does not itself authorize or perform that review.

Stop after this milestone. Do not begin the next Runtime Integration engineering milestone.

---

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-14 | Denver Jacobs | Initial EWO, registered per STD-001 §46. Authorizes the first implementation milestone realizing ARCH-005 — Temporal Runtime Architecture: a new `synapse-timer` service component; Runtime-mediated firing-check orchestration reusing the existing, unmodified admission pipeline; `ActorId`-keyed timer identity with restart preservation and Stop-triggered removal; the full seven-state timer lifecycle; fire-time-only capability validation; ordinary mailbox, causation, and audit treatment; and a monotonic-clock requirement with a deterministic test seam. Derived exclusively from ARCH-001 through ARCH-005, ADR-0015 through ADR-0017, STD-001, ER-007, and the verified current state of `runtime/`, every Trusted Core crate, `services/scheduler`, and `services/supervisor` at commit `5ccc7f9083a71adc6ee704b2322a701935765679`. Resolves five bounded design decisions this EWO itself identifies (crate placement and dependency set; the expiry condition; the firing-check invocation method; the Stop-removal mechanism; the clock seam), leaving open only the clock seam's and firing-condition's concrete Rust shape, per ARCH-005's own explicit deferral of implementation-level policy. |

## Disposition

Draft. Not yet reviewed. Not yet approved. Not yet implemented.
