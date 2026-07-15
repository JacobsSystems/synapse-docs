---
document_id: EWO-009
title: "Runtime Integration: Genuine Actor Execution, Capability-Authorized Actor-to-Actor Messaging, and Bootstrap Grants"
version: 0.1.3
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-15
last_updated: 2026-07-15
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.4.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
    - ARCH-006 (v0.1.3 — architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md) — the sole architectural authority this EWO records conformance to; itself a historical reconstruction, authored after this milestone's implementation
  adrs:
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
    - ADR-0017 (Approved)
  predecessor: EWO-006 (work-orders/EWO-006-Bounded-Actor-Mailboxes.md) — prior Runtime Integration milestone, independently confirmed to precede this one (ARCH-006 §15)
  successor: EWO-007 (work-orders/EWO-007-Local-Actor-Supervision.md) — the first later milestone independently confirmed to depend on this one; EWO-008 (work-orders/EWO-008-Temporal-Runtime.md) — the second, confirmed to reuse this milestone's own admission pattern verbatim
  reported_by: ER-009 (engineering-reports/ER-009-Runtime-Actor-Execution.md, not yet created)
  base_state:
    runtime_head: 5ccc7f9083a71adc6ee704b2322a701935765679
    docs_head: e90404baa5140ce9004839bc51921c789777e003
---

# EWO-009 — Runtime Integration: Genuine Actor Execution, Capability-Authorized Actor-to-Actor Messaging, and Bootstrap Grants

Registered per STD-001 §46 (Engineering Work Orders). **This document is a historical reconstruction — see "Historical Reconstruction Notice," immediately below — authored to restore the missing engineering authority for a milestone whose implementation already exists, already runs, and has already been independently reviewed three times.**

## Historical Reconstruction Notice

**This Engineering Work Order is a historical reconstruction, not a prospective authorization.**

- **The implementation predates this document.** Every capability this EWO records — genuine `Actor::handle()` execution, the shared Runtime admission pipeline, Runtime-owned causation and authority resolution, and the bootstrap-grant mechanism — already exists in `synapse-runtime`'s current working tree and was implemented before this document was authored.
- **The implementation has already been independently verified.** Three successive, independent reviews (the Publication Recovery Review; the Architecture Reconstruction Review, "Capability-Authorized Actor-to-Actor Messaging Runtime"; and the Runtime Actor Execution Architecture Review) each verified this milestone's behaviour directly against source, tests, and runtime execution — none against this document.
- **This document restores the missing engineering record around an existing implementation.** It documents, retrospectively, exactly what was built, why, and against what authority — it does not propose, and has never proposed, anything not already present in source.
- **This document does not authorize additional implementation.** No engineering work remains to be performed under this EWO. Its "Scope," "Deliverables," and "Acceptance Criteria" sections record what was already, verifiably, delivered — they are not a forward work list.
- **This document does not imply that implementation occurred after its own publication.** The opposite is true throughout: every date, every commit reference, and every test-count figure below reflects the state of the implementation as it already stood before this document existed. Where this EWO uses forward-looking language inherited from this repository's own EWO template (e.g., "Scope," "Required Tests," "Definition of Done"), each such section is understood, throughout, as a **retrospective record of what was done and verified**, never as an unmet requirement.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-009 |
| Title | Runtime Integration: Genuine Actor Execution, Capability-Authorized Actor-to-Actor Messaging, and Bootstrap Grants |
| Version | 0.1.3 |
| Status | **Draft** — this document's own governance status; the implementation it records is already complete |
| Author | Denver Jacobs |
| Owner | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-15 |
| Last Updated | 2026-07-15 |
| Classification | Public |
| Applicable repository | `synapse-runtime` |
| Target branch | `main` |
| Runtime base HEAD | `5ccc7f9083a71adc6ee704b2322a701935765679` |
| Documentation base HEAD | `e90404baa5140ce9004839bc51921c789777e003` |
| Predecessor milestone | EWO-006 — Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection |
| Successor milestones | EWO-007 — Local Actor Supervision (confirmed dependent); EWO-008 — Temporal Runtime (confirmed to reuse this milestone's own admission pattern) |
| Reported by | ER-009 (engineering-reports/ER-009-Runtime-Actor-Execution.md, not yet created) |

Implementation already exists against this EWO, already independently reviewed. This document records it; it does not report on future work, and it does not itself constitute a fresh approval act for new engineering — see the Historical Reconstruction Notice, above.

---

## Engineering Authority

This implementation is governed by, in descending order — recorded here in the same form every other EWO in this corpus states its authority, though every citation below is confirmed retrospectively rather than relied upon prospectively:

1. ARCH-001 — Constitutional Architecture — the four constitutional guarantees, in particular non-forgery (§5.3), extended by this milestone from actor identity to actor authority claims.
2. ARCH-002 — Runtime Architecture, **v0.2.0** — the authority for the Trusted Core component table (§6), the Constitutional Execution Cycle (§11), the message model's content/envelope/authority/transfer distinction (§8), and the Minimal Runtime Profile's own conformance requirements (§21, §22) — in particular "one-message actor execution" and "private actor-state transition," left unsatisfied until this milestone.
3. ARCH-003 — Runtime Integration Architecture, **v0.4.0** — the specific authority disclosing, unchanged across four of its own revisions, that "no actor-defined message-handling logic exists anywhere in the workspace" — the exact gap this milestone closes.
4. **ARCH-006 — Runtime Actor Execution Architecture, v0.1.3 — the specific and sole architectural authority for this milestone's scope, component placement, responsibility boundaries, execution model, shared admission architecture, bootstrap architecture, and constitutional invariants.** ARCH-006 is itself a historical reconstruction, authored after this milestone's implementation, on the same basis this EWO is. This EWO records conformance to it; neither reinterprets, extends, nor narrows it.
5. ADR-0015 — Audit Emitter Failure Semantics (Approved).
6. ADR-0016 — Trusted Core Interaction Rule (Approved).
7. ADR-0017 — Bootstrap Capability Trust Root (Approved).
8. STD-001 — Documentation Standards (§46, Engineering Work Orders).

These documents are authoritative. This EWO records conformance to them; it does not reinterpret or modify any of them.

---

## Purpose

This EWO documents the first implementation of genuine actor execution in SynapseOS: real, non-simulated `Actor::handle()` invocation; the treatment of an actor's own emitted messages as fresh, independently authorized admission requests rather than already-sent facts; Runtime-owned causation and authority resolution for those requests; the single, shared admission pipeline every message origin converges through; and a bounded, disclosed bootstrap-grant mechanism making all of the above reachable by a genuinely external, public-API-only caller — exactly as ARCH-006 establishes, and no further.

---

## Problem Statement

Verified directly against `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679`, and independently re-confirmed across three prior reviews before this document was authored — restating ARCH-003's own disclosed facts, unchanged across four of its own revisions (v0.1.0 through v0.4.0):

- *"No claim is made anywhere in this section that the current runtime performs actor logic. It performs exactly the mechanism ARCH-002 assigns Execution Coordinator... and nothing beyond it."*
- *"End-to-end actor execution is not demonstrated. No actor-defined message-handling logic exists anywhere in the workspace... `Execution Coordinator::dispatch` and `::complete` enforce the legal construct → dispatch → complete sequence and nothing more; no actor logic is actually invoked."*
- Nothing in the prior implementation treated an actor's own returned output as further Runtime work — `Actor::handle` itself did not yet exist as a genuinely invoked contract, so the question of what to do with its output had never been reached.
- No genuinely external, public-API-only caller had any route to a first `Capability` — `Capability` is constructible only by Capability Authority, and the Bootstrap Capability itself is never exposed (ADR-0017) — confining every prior demonstration to same-crate test-only shortcuts.

None of the above required correction beyond what this milestone's own implementation resolved; each fact is restated here because it directly bounds and justifies the scope this EWO records.

---

## Architectural Authority

| Concern | Owner | Authority |
|---|---|---|
| Genuine dispatch mechanics | Execution Coordinator | ARCH-006 §9.1; ARCH-002 §6 |
| Truthful cleanup of a genuinely failed dispatch (new: `fail`) | Execution Coordinator | ARCH-006 §9.1; closes EWO-005/ER-005's own disclosed gap |
| Supplies the mutable behavior value for dispatch (new: `behavior_mut`) | Actor Host | ARCH-006 §9.1, §15 |
| Attaches genuine behavior to a new instance at creation (new: `create_instance_with_behavior`) | Actor Host | ARCH-006 §9.1 |
| Retrieves a ready instance's next message for dispatch (new: `dequeue`) | Actor Host | ARCH-006 §9.1, §10 |
| Fresh, per-emission capability resolution (new: `bound_capabilities`) | Capability Authority | ARCH-006 §9.1, §13 |
| Envelope/structural authority validation, admission | Message Gateway (unchanged) | ARCH-006 §11 |
| Ready-order among ready actors; Scheduler's own first genuine implementation (trait redesign and FIFO ready set) | Scheduler | ARCH-006 §9.1 |
| Cross-component orchestration; sole caller of every new operation | Runtime | ARCH-006 §9.3; ADR-0016 Rule 1 |
| Primary caller-facing execution-driving surface (new: `step()`, `run_until_idle()`) | Runtime | ARCH-006 §10 |
| Audit-event emission (existing events, new call sites only) | Audit Emitter (unchanged trait/crate); Runtime (sole caller) | ARCH-006 §10; ARCH-002 §18; ADR-0015 |
| Bootstrap grant issuance | Runtime, via the existing `issue_capability` path | ARCH-006 §12; ADR-0017 |

---

## Objective

Implement genuine actor execution exactly as ARCH-006 records it: real `Actor::handle()` invocation via Execution Coordinator's own dispatch mechanics; treatment of an actor's own emitted messages as admission requests processed through the identical, single, Runtime-owned pipeline external submission already uses; deterministic, non-arbitrary capability resolution for those requests (`resolve_emitted_message_authority`, introducing `RuntimeError::AmbiguousAuthority`); Runtime-owned, non-forgeable causation; and a bounded, one-time bootstrap-grant mechanism giving a genuinely external caller its first capability. No supervision, no timers, no persistence, no distributed or networked execution of any kind — see "Explicit Exclusions," below.

**Every objective below is stated as achieved, independently verified, not as authorized for future work.**

- Genuine `Actor::handle()` invocation occurs, confirmed by direct source inspection (`runtime/src/lib.rs:1697-1815`) and by the passing `worker_pool` and `actor_to_actor_messaging` demonstrations.
- An actor's own emitted messages are treated as admission requests, independently resolved and admitted, never as already-sent facts.
- One shared admission pipeline (`admit_message`) serves both external submission and actor-emitted origin, confirmed by direct code reuse.
- Capability resolution is deterministic — exactly one match required — confirmed by the `AmbiguousAuthority` rejection path.
- Bootstrap grants are minted once, through the existing issuance path, before `Running` is entered, confirmed by direct source inspection.

---

## Bounded Design Decisions (reconstructed from evidence)

ARCH-006 fixes architecture; the implementation resolved the following implementation-level decisions, each reconstructed here from the evidence the implementation itself leaves — not authored prospectively, and not now subject to being reopened by this document.

### 1. Admission-pipeline consolidation — resolved, evidenced

A single private helper, `Runtime::admit_message`, was introduced and used identically by both `submit_message` (external origin) and the new `process_emitted_messages` (actor-emitted origin) — confirmed by direct reading, `runtime/src/lib.rs:979-1001`. This was not a genuinely open choice by the time of implementation: ARCH-002 §8's own pre-existing content/envelope/authority distinction, and ADR-0016 Rule 1's own composition discipline, jointly leave no architecturally defensible alternative to one shared pipeline.

### 2. Capability-resolution ambiguity handling — resolved, evidenced

Zero matching bound capabilities is rejected `EnforcementDenied` (an existing variant, reused); more than one matching candidate is rejected with a new variant, `RuntimeError::AmbiguousAuthority` (`common/src/lib.rs`), rather than arbitrarily selecting the first match. This is the **one new `RuntimeError` variant this milestone introduced** — confirmed to be the sole such addition, and confirmed narrow: it exists only to represent "more than one currently valid, bound capability structurally matches," a condition no prior variant could truthfully represent.

### 3. Bootstrap-grant bound and shape — resolved, evidenced

`MAX_BOOTSTRAP_GRANTS = 16` — a fixed, documented, non-configurable implementation constant, confirmed by its own doc comment to be deliberately modeled on `core/actor-host`'s own `MAILBOX_CAPACITY` precedent ("the same basis... a policy choice, not an architectural guarantee... Not configurable at this milestone, on the same basis `MAILBOX_CAPACITY` is not"). `BootstrapGrant`, `RuntimeBootstrapConfig`, and `BootstrapGrantSet` are each private-field, constructor-validated types (`runtime/src/lib.rs:440-596`), rejecting an empty operations set, more than 16 grants (`Overflow`, reused), or a duplicate grant name (`IntegrityViolation`, reused) before any capability is minted.

### 4. Actor Host's new query surface — resolved, evidenced

`ActorHost::behavior_mut(&mut self, instance: &ActorInstanceId) -> Option<&mut (dyn Actor + '_)>` was added to Actor Host's own trait and implementation (`core/actor-host/src/{lib,internal}.rs`) — the **narrow, disclosed exception to "zero Trusted Core change"** later milestones (EWO-007, EWO-008) each explicitly claim for themselves and could claim only because this milestone had already, once, made the one addition genuine dispatch structurally required: a way to obtain the actor's own stored, mutable behavior value at the moment of dispatch.

### 5. Scheduler's own trait redesign — resolved, evidenced

Per the Independent Implementation Review's own finding: this milestone did not merely reuse an existing Scheduler — it gave Scheduler its own first genuine implementation, redesigning the trait from a single, stateless `select_next(ready: &[ActorInstanceId])` method to three stateful operations (`mark_ready`, `remove`, `select_next()`) backed by a real FIFO ready set, confirmed by ER-006's own recorded test count for this crate immediately prior (1 test, and an empty `#[allow(dead_code)]` placeholder struct with no trait implementation). This was not a genuinely open design choice by the time genuine dispatch existed: without Scheduler tracking real per-instance readiness, no dispatch order could be proposed at all — the redesign is the direct, structural consequence of genuine actor execution requiring something to actually schedule.

### 6. Actor Host's further new surface — resolved, evidenced

Beyond `behavior_mut` (Decision 4), Actor Host also gained `create_instance_with_behavior`, the mechanism by which a new instance is given real, genuine behavior at creation (rather than a placeholder later replaced), and `dequeue`, retrieving a ready instance's own next mailbox message for dispatch. Both are confirmed genuinely new — absent from the committed predecessor state, present only in this milestone's own implementation (`core/actor-host/src/{lib,internal}.rs`). Neither was a genuinely open design choice by the time genuine dispatch existed: without a way to attach real behavior at creation, `behavior_mut` would have nothing genuine to return; without `dequeue`, no mechanism existed to retrieve the specific message a scheduled instance was ready to process. `dequeue`'s own further-work signal (`mailbox_has_more`) is what this milestone's own execution-model ordering guarantees depend on (ARCH-006 §10).

### 7. Execution Coordinator's truthful failure path — resolved, evidenced

`ExecutionCoordinator::fail` was added as the truthful counterpart to the existing `complete`, for a dispatch that genuinely failed (`Actor::handle()` returned `Err`) rather than genuinely completed. This closes the exact Execution-Coordinator-side cleanup gap EWO-005/ER-005 explicitly disclosed and left open at that milestone's own completion — confirmed by direct citation of that report's own text. Its introduction was not a genuinely open choice: once dispatch was genuine and could genuinely fail, treating a failure as though it were a completion (or omitting cleanup entirely) would have been a truthfulness violation this milestone's own admission and audit discipline could not tolerate.

### 8. Runtime's own caller-facing execution-driving surface — resolved, evidenced

`Runtime::step()` and `Runtime::run_until_idle()` were introduced as this milestone's own primary caller-facing entry points into the entire execution model — confirmed genuinely new, absent from the committed predecessor state. Supporting this surface: `schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, and `execute_next_scheduled_message_with_outcome` (internal-facing mechanics `step`/`run_until_idle` are themselves built from), and `bind_capability`, a further new public method. Confirmed by direct source inspection: no other public method causes `Actor::handle()` to be invoked — these are the sole entry points. Their introduction was not a genuinely open design choice: genuine dispatch, once it existed at all beneath the surface, structurally required some caller-facing way to actually drive it; the demonstrations and integration tests this milestone's own "Scope" records could not otherwise have been written.

### 9. Dependency disclosure — resolved, evidenced

Actor Host's and Execution Coordinator's own crate manifests gained an additional dependency on `synapse-api`, required by the new methods above. This is disclosed here explicitly as an engineering-level fact, on the same basis Decision 3's `MAILBOX_CAPACITY` precedent is disclosed — it introduces no new external (outside-workspace) dependency, and does not alter either component's own architectural boundary (ARCH-006 §9.2).

No other design decision is evidenced as having remained open at implementation time; where this document's own reconstruction cannot determine why a specific choice was made, this is disclosed explicitly under "Risks," below, rather than asserted as intentional.

---

## Scope (as implemented)

The following was implemented, each directly required by ARCH-006:

### Genuine dispatch (ARCH-006 §9, §10)

- Execution Coordinator's own `dispatch` signature evolved in three respects, confirmed by direct diff against the committed predecessor state: it gained a `message: &Message` parameter (the full message, not merely its identity, needed to genuinely invoke behaviour); it gained an `actor: Option<&mut dyn Actor>` parameter (`Option`, not a bare reference — a mechanical, behaviour-free instance still dispatches, invoking nothing); and its return type changed from `Result<(), RuntimeError>` to `Result<Vec<Message>, RuntimeError>`, the exact mechanism by which an actor's own emitted messages are captured for `process_emitted_messages`, below. `dispatch` genuinely invokes the supplied behaviour when present (`actor.handle(&context, message)`) and returns its emitted messages; construction, one-owner enforcement, and completion mechanics are confirmed unchanged. An earlier revision of this document described this as gaining only "a `behavior: &mut dyn Actor` parameter," omitting the `message` parameter and the return-type change; both are engineering-responsibility changes, not merely a naming correction — the changed return type is what makes emitted-message capture possible at all.
- Actor Host gained `behavior_mut`, supplying that value.
- `ActorExecutionOutcome` was introduced as the truthful, engineering-level record of a single dispatch's genuine result (success or failure), consumed internally by the caller-facing surface below.

### Execution Coordinator's truthful failure path (ARCH-006 §9.1)

- `ExecutionCoordinator::fail` was added as the truthful counterpart to `complete`, for a dispatch that genuinely failed — closing the cleanup gap EWO-005/ER-005 explicitly disclosed and left open.

### Actor Host's further new surface (ARCH-006 §9.1, §10)

- `ActorHost::create_instance_with_behavior` attaches real, genuine behavior to a new instance at creation.
- `ActorHost::dequeue` retrieves a ready instance's own next mailbox message for dispatch, returning a `DequeuedMessage` and the mailbox's own further-work signal (`mailbox_has_more`) this milestone's own execution-model ordering guarantees depend on (ARCH-006 §10).
- `Runtime::create_actor_instance_with_behavior` — the Runtime-level public wrapper that reaches `ActorHost::create_instance_with_behavior`, exactly as `Runtime::create_actor_instance` already reaches `ActorHost::create_instance`. Confirmed genuinely new to this milestone (absent from the committed predecessor state entirely) — not, as an earlier revision of this document stated, a pre-existing method retaining a stable signature. `Runtime::create_actor_instance` remains fully supported and unchanged: a mechanical, behaviour-free instance is still a first-class, deliberately retained path this new method does not silently replace.

### Runtime's own caller-facing execution-driving surface (ARCH-006 §10)

- `Runtime::step()` and `Runtime::run_until_idle()` — the sole caller-facing entry points into this milestone's entire execution model, confirmed by direct source inspection: no other public method causes `Actor::handle()` to be invoked. `step()` returns a `RuntimeStepOutcome`; `run_until_idle()` returns a `RuntimeRunOutcome`.
- `schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, `execute_next_scheduled_message_with_outcome` — the internal-facing mechanics `step()`/`run_until_idle()` are built from.
- `Runtime::bind_capability` — a further new public method, disclosed here for completeness.

### Scheduler's own first genuine implementation (ARCH-006 §9.1)

- `Scheduler`'s public trait was redesigned from a single, stateless `select_next(&mut self, ready: &[ActorInstanceId]) -> Option<ActorInstanceId>` method to three stateful operations: `mark_ready(&mut self, instance: ActorInstanceId) -> Result<(), RuntimeError>`, `remove(&mut self, instance: &ActorInstanceId) -> Result<(), RuntimeError>`, and `select_next(&mut self) -> Option<ActorInstanceId>` (no parameter — the ready set is now Scheduler's own tracked state, not caller-supplied on every call).
- `SchedulerImpl` gained a genuine FIFO implementation (`VecDeque`-backed order, `HashSet`-backed membership index) — previously an empty, `#[allow(dead_code)] pub(crate) struct SchedulerImpl;` placeholder with zero fields and no trait implementation.
- This is this milestone's own work, not a pre-existing capability reused: confirmed by ER-006's own recorded test count for this crate immediately prior (1 test) against its own count immediately after (19 tests) — the entire working Scheduler, not an incremental addition to one.
- Scheduler's own architectural role is unchanged by this addition — ready-order policy only (ARCH-002 §6), no lifecycle awareness, no capability awareness, no dependency beyond `synapse-common` — genuine actor execution is simply what first required a real dispatch-order mechanism to exist at all.

### Emitted-message processing (ARCH-006 §8, §10)

- `Runtime::process_emitted_messages` — processes `Actor::handle`'s own `Ok(Vec<Message>)` sequentially, non-atomically, each message independently, returning a `Vec<EmittedMessageOutcome>`.
- `EmittedMessageOutcome` — the truthful, per-message engineering record of each emitted message's own independent admission result (admitted or rejected, and if rejected, why); recorded here as a supporting engineering type, not an architectural concept in its own right.
- `Runtime::resolve_emitted_message_authority` — resolves send authority purely from the emitting actor's own currently bound, currently valid capability set, requiring exactly one structural match.
- Causation is unconditionally overwritten by Runtime with the truthful, Runtime-established triggering-message id.

### Shared admission (ARCH-006 §11)

- `Runtime::admit_message` — the single, private helper both `submit_message` and `process_emitted_messages` call identically.
- Capability Authority gained exactly one new public accessor, `bound_capabilities`, returning an actor's current bindings in deterministic (sorted-by-id) order.

### Bootstrap grants (ARCH-006 §12)

- `BootstrapGrantName`, `BootstrapGrant`, `RuntimeBootstrapConfig`, `BootstrapGrantSet`, and `Runtime::bootstrap_with_config` — a bounded, one-time, disclosed mechanism minting declared grants through the existing `issue_capability` path during the one-time bootstrap act.

### Demonstrations and tests

- `runtime/examples/worker_pool.rs`, `runtime/examples/actor_to_actor_messaging.rs` — public-API-only demonstrations of genuine, deterministic, round-robin dispatch and capability-authorized actor-to-actor messaging respectively.
- `runtime/tests/{bootstrap_grant,worker_pool,actor_to_actor_messaging}.rs` — public-API-only integration test suites.
- Extensive unit-test growth within `core/actor-host`, `core/capability-authority`, `core/execution-coordinator`, `services/scheduler`, and `runtime/src/lib.rs` itself — see "Validation Performed," below, for the exact, independently-reconciled figures.

---

## Explicit Exclusions

Confirmed absent from this milestone's own implementation by direct inspection — none of the following exists anywhere in the code this EWO records, and ARCH-006 §14 independently confirms each exclusion's architectural basis:

- **Supervision** — no failure-routing, restart, or parent/child concept exists; a failing `Actor::handle()` returns `Err`, handled by `ExecutionCoordinator::fail` (this milestone's own new, truthful counterpart to `complete` — see "Failure Semantics" — not a pre-existing cleanup path), with no restart, escalation, or policy decision applied to it. Introduced later, cleanly, by ARCH-004/EWO-007.
- **Timers** — zero `Instant`/`SystemTime`/clock reference anywhere in this milestone. Introduced later by ARCH-005/EWO-008.
- **Retries** — rejection is terminal per message; no redelivery mechanism exists.
- **Restart strategies** — no restart concept, and no reference to the `ActorId`/`ActorInstanceId` distinction, exists in this milestone's own code.
- **Persistence** — nothing in `RuntimeBootstrapConfig`, `BootstrapGrantSet`, or emitted-message outcomes survives beyond the in-process `Runtime` value.
- **Durable mailboxes** — mailbox contents remain exactly as EWO-006 left them: in-memory, lost on termination.
- **Workflow runtime** — no cross-message orchestration or compensation concept exists.
- **Effect runtime** — no generalized effect-scheduling abstraction; this milestone's own logic is specific to "a message `Actor::handle` itself returned."
- **Networking** — entirely in-process; no I/O anywhere in the reviewed code paths.
- **Distributed runtime** — `admit_message` resolves purely local `ActorInstanceId`s; no location or transport concept exists.
- **Clustering** — not addressed, not partially designed.
- **Service discovery** — Actor Directory (a separate, pre-existing replaceable service) is untouched; no new lookup mechanism was introduced.
- **Remote execution** — dispatch remains entirely local; `behavior_mut` returns an in-process reference, never a remote handle.

None of the above is supported by, or a candidate reading of, ARCH-006 — each is excluded for the specific architectural reason ARCH-006 §14 states, restated here identically, not independently re-derived.

---

## Trusted Core

**Unlike every later Runtime Integration milestone (EWO-007, EWO-008), this milestone does not claim zero Trusted Core source change.** Three of the seven Trusted Core components (ARCH-002 §6) gained a narrow, disclosed addition:

| Component | Addition | Scope |
|---|---|---|
| Actor Host | `behavior_mut`, `create_instance_with_behavior`, `dequeue` (new public methods); new dependency on `synapse-api` | Returns/attaches the stored, mutable behavior value and retrieves a ready instance's own next mailbox message; no new stored field beyond `behaviors` (see "Data and State Constraints") |
| Capability Authority | `bound_capabilities` (new public method) | Returns an actor's current bindings, deterministically ordered; no new dependency, no new field |
| Execution Coordinator | `dispatch`'s own signature gained a `message: &Message` parameter and an `actor: Option<&mut dyn Actor>` parameter (genuinely invoked when present) and its return type changed from `Result<(), RuntimeError>` to `Result<Vec<Message>, RuntimeError>`, capturing the invoked behaviour's own emitted messages; `fail` (new public method); new dependency on `synapse-api` | The truthful counterpart to `complete` for a genuinely failed dispatch, closing EWO-005/ER-005's own disclosed gap; no new stored field |

No new Trusted Core component was introduced; the count remains exactly seven. No responsibility was transferred between components — each addition is a narrow query or mechanics extension of a responsibility ARCH-002 §6 already assigned that same component, never a responsibility moved from one component to another. `synapse-lifecycle-guardian`, `synapse-message-gateway`, `synapse-audit-emitter`, and `synapse-host-adapter` required, and received, zero source changes. **`synapse-scheduler` is not a Trusted Core component (it is a replaceable service, ARCH-002 §6) and is therefore not counted among the seven above — but it is not unaffected by this milestone: it received its own first genuine implementation, disclosed in full under "Scope," "Required Interface Evolution," and Bounded Design Decision 5.**

---

## Component Boundaries and Prohibited Interactions

The following, confirmed absent by direct dependency-graph and call-site inspection (ADR-0016 Rule 2):

| Prohibited direct call | Confirmed absent |
|---|---|
| Execution Coordinator → Capability Authority | Dispatch never validates capability itself; validation occurs only inside Runtime's own `admit_message`, before dispatch is ever reached for a *subsequent* message |
| Actor Host → Capability Authority | `behavior_mut`, `create_instance_with_behavior`, and `dequeue` each operate only over Actor Host's own stored state (instances, behaviors, mailboxes); none reads or reaches into Capability Authority — no capability awareness of any kind |
| Capability Authority → Actor Host | `bound_capabilities` reads only Capability Authority's own registry; no reach into Actor Host |
| Scheduler → Actor Host, Capability Authority, or Lifecycle Guardian | Scheduler gained its own first genuine implementation as part of this milestone (`mark_ready`/`remove`/`select_next`, a real FIFO ready set), but remains fully isolated from every other component — confirmed by its own unchanged `synapse-common`-only dependency set; `mark_ready`/`remove`/`select_next` are each called only by Runtime |
| Any actor (via its own emitted messages) → a privileged admission shortcut | Every emitted message passes through the identical `admit_message` pipeline external submission uses — no actor-controlled bypass exists |

Runtime remains the only bridge for every one of these pairs (ADR-0016 Rule 1).

---

## Architecture Constraints (confirmed observed)

The implementation, as it stands, is confirmed **not** to have done any of the following:

- introduce a new Trusted Core component;
- invent a new lifecycle state, `ActorState`, or `RuntimeState` variant;
- modify any constitutional concept;
- reinterpret ARCH-001, ARCH-002, or ARCH-003;
- redesign `ActorHost`, `LifecycleGuardian`, `ExecutionCoordinator`, `MessageGateway`, or `CapabilityAuthority`'s public trait beyond the specific narrow additions §"Trusted Core" names in full (`behavior_mut`, `create_instance_with_behavior`, `dequeue` on Actor Host; `bound_capabilities` on Capability Authority; the `dispatch` parameter and `fail` on Execution Coordinator) — each is an additive extension of a responsibility ARCH-002 §6 already assigned that component, never a redesign of the trait's own existing shape. **Scheduler's own public trait is the one confirmed exception to "additive only"**: it was substantially redesigned by this milestone (§"Required Interface Evolution," §"Scope"), from a single stateless method to three stateful ones, because no genuine dispatch order could be tracked without one. This is disclosed here explicitly, not silently folded into the additive-only language above, which refers only to Actor Host, Capability Authority, and Execution Coordinator;
- redesign any existing `Runtime` public method's signature (`bootstrap`, `submit_message`, `execute_message`, `terminate_actor_instance`, and every other pre-existing public method retain their exact prior signatures — `bootstrap_with_config` and the bootstrap-grant types are additive, not replacements). **`Runtime::step()`, `Runtime::run_until_idle()`, and `Runtime::create_actor_instance_with_behavior` are not pre-existing methods and are not part of this "no redesign" claim — all three are confirmed genuinely new to this milestone (absent from the committed predecessor state entirely, not merely signature-stable) and are disclosed in full under "Scope" and "Required Interface Evolution," below;**
- introduce external dependencies;
- use unsafe Rust (confirmed: zero `unsafe` anywhere in the workspace);
- introduce more than the one new `RuntimeError` variant (`AmbiguousAuthority`);
- introduce a generic, configurable admission or authorization framework of any kind.

---

## Required Interface Evolution (as implemented)

**`Runtime` (existing crate):**

- `bootstrap_with_config(config: RuntimeBootstrapConfig) -> Result<(Self, BootstrapGrantSet), RuntimeError>` — new public constructor; `bootstrap()` delegates to it with an empty configuration, preserving every existing caller's own behaviour unchanged.
- `create_actor_instance_with_behavior<A>(&mut self, actor: &ActorId, behavior: A) -> Result<ActorInstanceId, RuntimeError>` — new public method, the Runtime-level wrapper reaching `ActorHost::create_instance_with_behavior`; confirmed genuinely new (absent from the committed predecessor state), not a pre-existing method whose signature merely remained stable. `create_actor_instance` (the mechanical, behaviour-free constructor) retains its own exact prior signature, unaffected.
- `admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages` — new private methods; no public signature change.
- `execute_message_capturing` — the existing dispatch flow, now genuinely invoking actor behaviour and routing its emitted output through the above.
- `step() -> RuntimeStepOutcome` and `run_until_idle() -> RuntimeRunOutcome` — new public methods; this milestone's own sole caller-facing entry points into genuine dispatch, confirmed genuinely new (absent from the committed predecessor state), not pre-existing methods whose signature merely remained stable.
- `schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, `execute_next_scheduled_message_with_outcome` — new private/internal methods supporting `step()`/`run_until_idle()`; no public signature change beyond the two entry points above.
- `bind_capability` — new public method.

**`core/actor-host` (Trusted Core):** `behavior_mut`, `create_instance_with_behavior`, and `dequeue` added to the `ActorHost` trait and its implementation. New dependency: `synapse-api`.

**`core/capability-authority` (Trusted Core):** `bound_capabilities` added to the `CapabilityAuthority` trait and its implementation — the sole new public method.

**`core/execution-coordinator` (Trusted Core):** `dispatch(&mut self, context: ExecutionContext) -> Result<(), RuntimeError>` (committed predecessor state) became `dispatch(&mut self, context: ExecutionContext, message: &Message, actor: Option<&mut dyn Actor>) -> Result<Vec<Message>, RuntimeError>` — two new parameters, not one, and a changed return type: `message` supplies the full message `Actor::handle` requires; `actor` is genuinely invoked when present (a mechanical, behaviour-free instance still dispatches, invoking nothing); the return type change from `()` to `Vec<Message>` is the mechanism by which an actor's own emitted messages are captured for `process_emitted_messages`. `fail` was also added to the `ExecutionCoordinator` trait and its implementation, the truthful counterpart to `complete` for a genuinely failed dispatch. New dependency: `synapse-api`.

**`services/scheduler` (existing crate, replaceable service — not Trusted Core):** the `Scheduler` trait was redesigned from `select_next(&mut self, ready: &[ActorInstanceId]) -> Option<ActorInstanceId>` to `mark_ready(&mut self, instance: ActorInstanceId) -> Result<(), RuntimeError>`, `remove(&mut self, instance: &ActorInstanceId) -> Result<(), RuntimeError>`, and `select_next(&mut self) -> Option<ActorInstanceId>`; `SchedulerImpl` gained a genuine FIFO implementation. This is the one component-level trait redesign this milestone performs — disclosed explicitly, not folded into the "no other existing crate's public interface changed" statement below, which excludes Scheduler for this reason.

**`common/src/lib.rs`:** `RuntimeError::AmbiguousAuthority` — the sole new variant.

**Supporting engineering types (new, non-architectural):** `ActorExecutionOutcome` (Execution Coordinator's genuine per-dispatch result), `RuntimeStepOutcome` (the result of a single `step()` call), `RuntimeRunOutcome` (the result of a full `run_until_idle()` call), `EmittedMessageOutcome` (a single emitted message's own admission result, returned by `process_emitted_messages`), `DequeuedMessage` (Actor Host's `dequeue` return value, carrying the message and the `mailbox_has_more` signal).

No other existing crate's public interface changed beyond Scheduler's own redesign and the Actor Host / Execution Coordinator / Runtime additions disclosed above.

---

## Runtime Sequencing (as implemented)

Confirmed by direct source inspection, `runtime/src/lib.rs:1697-1815` (`execute_message_capturing`):

1. (existing, EWO-005) Actor Host `live_instance`; Lifecycle Guardian `validate_transition`; Host Adapter `allocate_execution_handle`; Execution Coordinator `construct_context`; Lifecycle Guardian `begin_execution`.
2. **(this milestone)** Actor Host `behavior_mut(&instance)` obtains the actor's own stored, mutable behavior value.
3. **(this milestone)** Execution Coordinator `dispatch(context, &message, behavior)` — genuine invocation of `Actor::handle()`.
4. On genuine success (existing, EWO-005): Execution Coordinator `complete`; Lifecycle Guardian `complete_execution`; Host Adapter `release_execution_handle`; Audit Emitter `execution.completed`. On genuine failure, **(this milestone)**: Execution Coordinator `fail` — the truthful counterpart to `complete`, closing EWO-005/ER-005's own disclosed cleanup gap — followed by the equivalent Lifecycle Guardian / Host Adapter / Audit Emitter cleanup for the failed outcome.
5. **(this milestone)** `Runtime::process_emitted_messages(&sender, &triggering_message, emitted)` — reached only after step 4 has genuinely, truthfully completed. For each emitted message, independently: `resolve_emitted_message_authority`, then `admit_message` (Message Gateway → Capability Authority → Actor Host → Scheduler), then the existing `message.admitted`/`message.rejected` audit event.

Steps 2–3, the failure branch of step 4, and step 5 are this milestone's own contribution to this function; every step marked "existing" is EWO-005's own unmodified sequence, reused, not touched.

---

## Failure Semantics (as implemented)

- A genuine `Actor::handle()` failure (`Err`) is the one and only dispatch-rejection path this milestone's own dispatch step can produce; it is handled by `ExecutionCoordinator::fail`, this milestone's own new, truthful counterpart to `complete` — not, as an earlier revision of this document stated, a pre-existing cleanup sequence. `fail` closes the exact cleanup gap EWO-005/ER-005 explicitly disclosed and left open at that milestone's own completion (Bounded Design Decision 7).
- Each emitted message's own admission outcome is independent — one rejection never prevents or affects a later message's own attempt, confirmed by direct reading of `process_emitted_messages`'s own sequential loop.
- `process_emitted_messages` returns `Err` only if a mandatory audit emission itself fails (ADR-0015's existing rule) — never merely because an individual emitted message was rejected, which is recorded in the returned `Vec<EmittedMessageOutcome>` as a truthful, expected outcome, not an operation failure.
- No new failure mode exists beyond what this section records.

---

## Audit Semantics (as implemented)

- No new audit event type was introduced. The existing `message.admitted`, `message.rejected`, and `execution.completed` events (ARCH-002 §18) are reused, unmodified, for this milestone's own new call sites.
- Truthful ordering, confirmed by direct source inspection: `execution.completed` is emitted before any emitted message is processed; each emitted message's own `message.admitted`/`message.rejected` fires as that message's own outcome becomes truthfully known, never batched or deferred.
- Audit-emission failure follows ADR-0015 unchanged: the reporting operation fails if the emission fails, with no rollback of already-committed component-level state.

---

## Definition of Done (confirmed satisfied)

- Genuine `Actor::handle()` invocation occurs, confirmed by passing demonstrations and dedicated tests. ✅
- An actor's own emitted messages are treated as admission requests, independently resolved and admitted through the existing, shared pipeline. ✅
- Capability resolution for an emitted message is deterministic — exactly one match required, ambiguity refused rather than arbitrated. ✅
- Causation is Runtime-established, never left to an actor's own self-declared claim. ✅
- Bootstrap grants are minted once, through the existing issuance path, before `Running` is entered; the Bootstrap Capability itself is never exposed. ✅
- No Trusted Core component beyond the three narrowly, disclosedly extended (Actor Host, Capability Authority, Execution Coordinator) required change; the Trusted Core boundary remains exactly seven components. ✅
- Scheduler received its own first genuine implementation (trait redesign, FIFO ready set) — disclosed explicitly as this milestone's own work, not folded into "no other change." ✅
- `Runtime::step()` and `Runtime::run_until_idle()` are disclosed as this milestone's own new, sole caller-facing entry points into genuine dispatch — not grouped with pre-existing, signature-stable methods. ✅
- `ActorHost::create_instance_with_behavior`, `ActorHost::dequeue`, and `ExecutionCoordinator::fail` are each disclosed as this milestone's own new engineering deliverables, not omitted or folded into "no other change." ✅
- No existing public method signature changed beyond Scheduler's own disclosed redesign and the Trusted Core additions disclosed under "Trusted Core," above; only additive new constructors/methods exist otherwise. ✅
- Exactly one new `RuntimeError` variant (`AmbiguousAuthority`) exists anywhere in the implementation. ✅
- All tests pass; zero warnings; zero `unsafe`. ✅ (see "Validation Performed," below)

---

## Validation Performed

The following gates were run and confirmed passing, both at the time of this milestone's own implementation (evidenced by every later milestone's own regression suite continuing to pass against this milestone's own contribution unmodified) and independently re-confirmed in this session's own prior reviews:

```text
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace
cargo test --workspace
cargo tree --workspace
```

**Test-count reconciliation — reconstructed, cross-checked against two independently-authored Engineering Reports:**

| Crate/target | Before this milestone (ER-006's own final count) | After this milestone (ER-007's own stated starting baseline / reconciled) | This milestone's own delta |
|---|---|---|---|
| `synapse-actor-host` | 31 | 48 | **+17** (`behavior_mut` and its supporting tests) |
| `synapse-capability-authority` | 48 | 53 | **+5** (`bound_capabilities` and its supporting tests) |
| `synapse-execution-coordinator` | 15 | 30 | **+15** (genuine-dispatch tests) |
| `synapse-scheduler` | 1 | 19 | **+18** (real-dispatch/fairness regression under genuine execution) |
| `synapse-runtime` unit tests (`src/lib.rs`) | 54 | 151 | **+97** (admission pipeline, bootstrap grants, emitted-message processing) |
| `runtime/tests/actor_to_actor_messaging.rs` (new) | 0 | 2 | **+2** |
| `runtime/tests/bootstrap_grant.rs` (new) | 0 | 8 | **+8** |
| `runtime/tests/worker_pool.rs` (new) | 0 | 29 | **+29** |
| `runtime/tests/bootstrap.rs` | 16 | 16 | 0 (unaffected regression) |
| All other crates (`actor-directory`, `api`, `audit-emitter`, `audit-pipeline`, `common`, `host-adapter`, `lifecycle-guardian`, `message-gateway`, `persistence`) | unchanged | unchanged | 0 |
| **Workspace total** | **232** (ER-006's own final figure) | **423** (ER-007's own stated starting baseline) | **+191** |

The per-crate deltas above sum exactly to the workspace-level delta (17+5+15+18+97+2+8+29 = 191), independently reconciling ER-006's and ER-007's own, separately-authored figures against one another for the first time under this document.

**Demonstrations, confirmed run to completion, output consistent with deterministic, correct behaviour:**

```text
cargo run --example worker_pool
cargo run --example actor_to_actor_messaging
```

---

## Data and State Constraints (as implemented)

- `Runtime` gained no new field for this milestone specifically — the capability is realized entirely as new methods on the existing struct.
- `ActorHostImpl` gained one new stored field, `behaviors: HashMap<ActorInstanceId, Box<dyn Actor>>` — the storage `create_instance_with_behavior` populates and `behavior_mut`/`dequeue` read from. This corrects an earlier revision of this document, which stated `ActorHostImpl` gained no new stored field; that statement held only for `behavior_mut` in isolation and did not account for `create_instance_with_behavior`'s own storage requirement.
- `CapabilityAuthorityImpl` gained no new stored field — `bound_capabilities` is a query method over state the crate already held.
- `ExecutionCoordinatorImpl` gained no new stored field — `dispatch`'s own new `message` and `actor` parameters are passed through, not stored, and its emitted-message return value is constructed fresh, not stored; `fail` operates over state already held.
- `SchedulerImpl` gained its own new state (`order: VecDeque<ActorInstanceId>`, `members: HashSet<ActorInstanceId>`) — the ready set itself, previously nonexistent (the prior placeholder held no field of any kind).
- No new field was added to `Message`, `ActorId`, `ActorInstanceId`, `ExecutionContext`, `AuditEvent`, or any other `synapse-common` type beyond the one new `RuntimeError` variant.

---

## Acceptance Criteria (objectively verified)

Every criterion below is independently verifiable against the current source tree and was confirmed, not assumed, before this document was authored:

- `Execution Coordinator::dispatch` genuinely invokes the supplied `Actor::handle()` implementation — confirmed by direct reading and by both passing demonstrations.
- `Runtime::step()` and `Runtime::run_until_idle()` are confirmed, by direct source inspection, to be the sole public methods that cause `Actor::handle()` to be invoked — no other public method reaches genuine dispatch.
- `ActorHost::create_instance_with_behavior` and `ActorHost::dequeue`, and `ExecutionCoordinator::fail`, are each confirmed present, genuinely new, and disclosed in this document's own "Scope," "Required Interface Evolution," and "Trusted Core" sections.
- Every emitted message is processed through `Runtime::process_emitted_messages`, independently, non-atomically.
- Capability resolution for an emitted message requires exactly one structurally-matching, currently-valid, currently-bound candidate.
- Causation for an emitted message is set by Runtime, never left as the emitting actor's own unverified claim.
- A fired admission attempt for an emitted message reaches admission only through `admit_message` — the identical function `submit_message` uses.
- Bootstrap grants are minted only once, before `Running` is entered, through the existing `issue_capability` path; the Bootstrap Capability itself is never returned or exposed.
- Exactly three Trusted Core components gained a narrow, disclosed addition (Actor Host, Capability Authority, Execution Coordinator); the remaining four required, and received, zero change.
- Scheduler's own trait redesign and FIFO implementation are disclosed explicitly as this milestone's own engineering work, not described as "unaffected" or "unchanged."
- Exactly one new `RuntimeError` variant exists in the entire implementation.
- All pre-existing tests (EWO-001 through EWO-006) pass unmodified in outcome.
- `cargo fmt`, `cargo clippy -D warnings`, `cargo build`, and `cargo test --workspace` all pass with zero warnings.
- Architecture and ADR documents remain byte-for-byte untouched by this milestone's own implementation.

---

## Implementation Sequence (historical)

Reconstructed, in evidenced logical order, from the dependency structure the implementation itself exhibits — not a prescription, a record:

1. Actor Host gained `create_instance_with_behavior`, the mechanism by which a new instance is given real, genuine behavior at creation — a precondition for `behavior_mut` or `dequeue` to have anything genuine to return.
2. Capability Authority gained `bound_capabilities`, the query surface fresh authority resolution requires.
3. Actor Host gained `behavior_mut` and `dequeue`, the query surfaces genuine dispatch requires.
4. Execution Coordinator's `dispatch` gained its `message` and `actor` parameters, genuinely invoking the latter when present and returning its emitted messages; Execution Coordinator gained `fail`, the truthful counterpart to `complete` a genuinely failing dispatch now required.
5. `common/src/lib.rs` gained `RuntimeError::AmbiguousAuthority`, required once capability resolution against a *set* of candidates became possible.
6. `Runtime::resolve_emitted_message_authority` and `Runtime::admit_message` were introduced, consolidating admission into one shared helper.
7. `Runtime::process_emitted_messages` was introduced, wired into `execute_message_capturing` immediately after EWO-005's own existing completion sequence.
8. `BootstrapGrantName`, `BootstrapGrant`, `RuntimeBootstrapConfig`, `BootstrapGrantSet`, and `Runtime::bootstrap_with_config` were introduced, giving the whole capability external reachability.
9. `Runtime::schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, `execute_next_scheduled_message_with_outcome`, `bind_capability`, and finally the caller-facing `step()`/`run_until_idle()` were introduced, giving genuine dispatch its first caller-facing entry points.
10. Extensive component-local and Runtime-level tests were added, per the reconciled figures above.
11. `runtime/examples/{worker_pool,actor_to_actor_messaging}.rs` and their corresponding `runtime/tests/` integration suites were added as public-API-only demonstrations.
12. `examples/README.md` was updated to describe the new demonstrations.

This ordering is a reconstruction from structural dependency, not a verified historical timeline — no commit-level record exists to confirm the literal sequence in which these steps occurred (Publication Recovery Review, §4–§5).

---

## Engineering Decision Log

- Admission was consolidated into one shared helper rather than duplicated across two call sites — the direct, evidenced consequence of ADR-0016 Rule 1 and ARCH-002 §8, not an independently invented convenience.
- Capability-resolution ambiguity was refused rather than arbitrated — a new invariant, evidenced by the introduction of `AmbiguousAuthority` specifically to represent it, rather than reusing `EnforcementDenied` (which means "no authority exists," a materially different condition).
- The bootstrap-grant bound (`16`) was fixed and modeled explicitly on `MAILBOX_CAPACITY`'s own precedent, evidenced by direct doc-comment citation.
- Actor Host's and Capability Authority's new methods were each kept to the narrowest possible query surface — no setter, no mutation beyond what already existed, confirmed by direct reading of both new methods' own bodies.
- Future work enabled (confirmed, not merely anticipated): a genuine, testable local-execution mechanism EWO-007's supervision and EWO-008's Temporal Runtime both directly built upon, confirmed by citation in each of their own governing text.
- Future work deferred (confirmed unaffected by this milestone): supervision, timers, persistence, durable mailboxes, workflow, effect, distributed, and clustered execution — see "Explicit Exclusions," above.

---

## Required Completion Report Contents

ER-009, if and when authored, should provide, consistent with STD-001 §47 and this repository's own established precedent (ER-006 through ER-008):

1. Files changed, reconciled against this EWO's own "Scope" and "Trusted Core" sections.
2. Confirmation, re-verified against source, of the exact Trusted Core boundary this EWO records (three components narrowly extended, four untouched).
3. The exact test-count reconciliation this EWO reconstructs (§"Validation Performed"), re-verified independently rather than restated from this document alone.
4. Explicit confirmation of every item in "Explicit Exclusions," stating whether each remains accurately excluded as of the report's own authoring.
5. An independent implementation review summary, on the same basis EWO-006/007/008's own reports each provide — even though the implementation itself was already independently reviewed three times before this EWO existed, a dedicated ER should still record that review chain explicitly, for this milestone's own permanent record.
6. Deviations from this EWO, if any are found upon closer, independent re-verification.
7. Explicit acknowledgement that this EWO, and any ER reporting on it, are both historical reconstructions — per the Historical Reconstruction Notice this document itself establishes and requires any dependent document to preserve.

---

## Independent Implementation Review Expectations

Any future ER authored against this EWO should, at minimum, independently re-verify — not merely restate — every figure and claim in this document's own "Validation Performed" and "Trusted Core" sections, exactly as the three prior reviews already did for the architecture this EWO records. A reconstruction's own authority is only as strong as the independence of its own re-verification; this EWO's own claims should be treated as a starting hypothesis for that re-verification, not as ground truth exempt from it.

---

## Traceability Requirements

Every future document referencing this milestone MUST cite:

- **ARCH-006** as the architectural authority this EWO implements;
- **this EWO (EWO-009)** as the engineering authorization record;
- **ER-009** (once authored) as the completion report;
- the three prior reviews (Publication Recovery Review; Architecture Reconstruction Review; Runtime Actor Execution Architecture Review) as the analytical basis from which both ARCH-006 and this EWO were themselves reconstructed;
- **ER-006** and **ER-007** specifically, as the two independently-authored sources whose own stated test-count figures (232 and 423 respectively) jointly evidence this milestone's own historical placement and scope.

No future document may cite this milestone's own capabilities (`admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`, bootstrap grants) without tracing back to this EWO and ARCH-006, exactly as EWO-007 and EWO-008 already, independently, do in their own text (§"References," below).

---

## Historical Reconstruction Considerations

- This EWO's own "Bounded Design Decisions," "Implementation Sequence," and parts of its "Engineering Decision Log" are **reconstructions from evidence**, not records of decisions as they were actually deliberated at the time — where the evidence does not permit a confident reconstruction of *why* a specific choice was made (as opposed to *what* was chosen), this document says so explicitly (see "Risks," below) rather than inventing a rationale.
- This EWO does not, and cannot, retroactively grant itself the same real-time governance value an EWO authored *before* its own implementation would have had — it restores traceability and record-keeping completeness; it does not restore the counterfactual opportunity for prospective review that a contemporaneous EWO would have provided.
- Every section of this document that uses the present-authorizing tense common to this repository's own EWO template ("Scope," "Definition of Done," "Acceptance Criteria") is to be read, throughout, as recording an already-settled fact, never as granting new authorization — consistent with the Historical Reconstruction Notice this document opens with.

---

## Risks

- **This EWO's own reconstructed "Implementation Sequence" and parts of its "Engineering Decision Log" cannot be independently verified against a real-time record** — no commit-level history exists for this milestone (Publication Recovery Review, §4–§5); the ordering and rationale recorded above are the most evidence-consistent reconstruction available, not a verified timeline. **Classification: Major** (a historical-completeness limitation, not an engineering-correctness one).
- **This milestone remains, after this EWO, still without a dedicated Engineering Report.** ER-009 does not yet exist; this EWO's own "Required Completion Report Contents" section anticipates one but does not itself satisfy that requirement. **Classification: Major.**
- **Every later EWO (EWO-006 in its own predecessor role, EWO-007, EWO-008) already cites this milestone's own artifacts as established precedent, predating this EWO's own formal existence.** This EWO closes that citation gap retroactively; it cannot, and does not claim to, alter what those three EWOs' own already-existing text already says. **Classification: Observation** (a closed, not an open, risk — recorded for completeness).
- **No engineering-correctness risk was identified.** Every risk named above concerns the completeness of the historical record, not the correctness, safety, or behaviour of the implementation itself, which three independent prior reviews each found sound.

---

## References

- ARCH-001 — Constitutional Architecture
- ARCH-002 — Runtime Architecture (v0.2.0; §6, §8, §11, §21, §22)
- ARCH-003 — Runtime Integration Architecture (v0.4.0; §5, §18, all four revisions)
- ARCH-006 — Runtime Actor Execution Architecture (v0.1.3) — the sole architectural authority this EWO implements
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- EWO-006 — Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection (predecessor)
- ER-006 — Bounded Actor Mailboxes — Engineering Report (§4, §8, §11, §12 — direct evidentiary basis for this EWO's own test-count reconciliation)
- EWO-007 — Runtime Integration: Local Actor Supervision, Failure Escalation, and Restart Ownership (successor; its own "Problem Statement"/"Component Boundaries" sections cite this milestone's own artifacts directly)
- ER-007 — Local Actor Supervision — Engineering Report (§5, §9 — direct evidentiary basis for this EWO's own test-count reconciliation)
- EWO-008 — Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution (successor; reuses this milestone's own admission pattern verbatim)
- ER-008 — Temporal Runtime — Engineering Report
- "SynapseOS — Publication Recovery Architecture Review" (this engineering effort)
- "SynapseOS Architecture Review — Capability-Authorized Actor-to-Actor Messaging Runtime" (this engineering effort)
- "SynapseOS Architecture Review — Runtime Actor Execution Architecture" (this engineering effort)
- `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree): `runtime/src/lib.rs` (including `step`, `run_until_idle`, `schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, `execute_next_scheduled_message_with_outcome`, `bind_capability`), `core/actor-host/src/{lib,internal}.rs` (including `create_instance_with_behavior`, `dequeue`, and the `behaviors` field), `core/actor-host/Cargo.toml` (new `synapse-api` dependency), `core/capability-authority/src/{lib,internal}.rs`, `core/execution-coordinator/src/{lib,internal}.rs` (including `fail`), `core/execution-coordinator/Cargo.toml` (new `synapse-api` dependency), `services/scheduler/src/{lib,internal}.rs`, `common/src/lib.rs`, `runtime/examples/{worker_pool,actor_to_actor_messaging}.rs`, `runtime/tests/{bootstrap_grant,worker_pool,actor_to_actor_messaging}.rs`

---

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-15 | Denver Jacobs | Initial Draft. Historical reconstruction of the Engineering Work Order for genuine Runtime actor execution, capability-authorized actor-to-actor messaging, and bootstrap grants — restoring the missing engineering-authorization record for an already-completed, already-independently-reviewed (three times) implementation milestone. Derived exclusively from ARCH-006, the three prior governing reviews, ER-006's and ER-007's own independently-reconciled test-count figures (232 and 423 respectively, jointly evidencing a +191 test delta this document reconciles per-crate for the first time), and direct source inspection of `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679`. Authorizes no implementation — the work it records is already fully complete. |
| 0.1.1 | 2026-07-15 | Denver Jacobs | Corrective revision following the Independent Implementation Review of this milestone (verdict: CHANGES REQUIRED), which found this document incorrectly described Scheduler as unchanged/unaware of this milestone in "Architectural Authority," "Component Boundaries and Prohibited Interactions," and "Architecture Constraints." Direct implementation inspection showed Scheduler's own public trait was redesigned (from a single stateless `select_next(ready: &[ActorInstanceId])` method to three stateful operations) and its FIFO implementation written from scratch by this milestone — confirmed by ER-006's own recorded test count for this crate immediately prior (1 test, empty placeholder struct) against its own count immediately after (19 tests). Corrected: "Architectural Authority" table; added a new "Scheduler's own first genuine implementation" subsection to "Scope"; added a new Bounded Design Decision (#5); corrected "Component Boundaries and Prohibited Interactions," "Architecture Constraints," "Definition of Done," and "Acceptance Criteria"; added a Scheduler subsection to "Required Interface Evolution"; added `SchedulerImpl`'s own new state to "Data and State Constraints"; added `services/scheduler/src/{lib,internal}.rs` to "References." No other section changed. Scheduler's own architectural role (ready-order policy only, ARCH-002 §6, fully isolated from every other component) is unchanged by this correction — only the prior claim that this milestone left it untouched is corrected. No new engineering decision is introduced; no other claim in this document is affected. |
| 0.1.2 | 2026-07-15 | Denver Jacobs | Revision applying the approved Governance Coverage Reconciliation. A second, fully independent Re-Review (which explicitly did not assume the 0.1.1 correction was sufficient) found four further undisclosed items: `Runtime::step()`/`run_until_idle()` (and their supporting internals `schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, `execute_next_scheduled_message_with_outcome`, `bind_capability`), `ActorHost::create_instance_with_behavior`, `ActorHost::dequeue`, and `ExecutionCoordinator::fail`. A subsequent forensic Implementation Inventory and Governance Coverage Reconciliation classified each: `step()`/`run_until_idle()` as Category A (architectural — also updates ARCH-006 §10, in that document's own 0.1.2 revision), and `create_instance_with_behavior`/`dequeue`/`fail` as Category B (engineering deliverables — this document only, no architectural principle at stake). Applied: corrected the "Architecture Constraints" bullet that had incorrectly grouped `step`/`run_until_idle` with genuinely pre-existing, signature-stable `Runtime` methods; added three new Bounded Design Decisions (#6 Actor Host's further new surface, #7 Execution Coordinator's `fail`, #8 Runtime's own caller-facing execution-driving surface) and one dependency disclosure (#9, `synapse-api` added to Actor Host and Execution Coordinator); expanded "Scope" with three new subsections and named `EmittedMessageOutcome` explicitly; updated the "Trusted Core" table's Actor Host and Execution Coordinator rows; expanded "Required Interface Evolution" with the full Runtime/Actor Host/Execution Coordinator additions and five new supporting engineering types (`ActorExecutionOutcome`, `RuntimeStepOutcome`, `RuntimeRunOutcome`, `EmittedMessageOutcome`, `DequeuedMessage`); corrected "Runtime Sequencing" step 4 and "Failure Semantics" to disclose `fail` rather than describing failure cleanup as pre-existing and unmodified; corrected "Data and State Constraints" to disclose `ActorHostImpl.behaviors: HashMap<ActorInstanceId, Box<dyn Actor>>`, superseding the 0.1.0/0.1.1 statement that `ActorHostImpl` gained no new stored field; added corresponding "Definition of Done" and "Acceptance Criteria" bullets; updated "References" source evidence. No architectural principle changed, no Trusted Core component count changed (still exactly seven), no engineering decision already recorded was reopened or reversed — this revision only completes disclosure of what the second Re-Review and the Reconciliation independently confirmed was already, genuinely, part of this milestone's own implementation. |
| 0.1.3 | 2026-07-15 | Denver Jacobs | Applied the three remaining corrections identified by a dedicated Historical Provenance Audit (distinct from, and more exhaustive than, the two prior Independent Reviews — it swept every historical-claim statement in this document and ARCH-006 against direct source evidence). **Correction 1:** removed `Runtime::create_actor_instance_with_behavior` from "Architecture Constraints"' list of methods "retaining their exact prior signatures" — confirmed absent from the committed predecessor state entirely, not signature-stable; disclosed it in full under "Scope" (new bullet, "Actor Host's further new surface") and "Required Interface Evolution" (new `Runtime` bullet). **Correction 2:** corrected "Explicit Exclusions"' Supervision bullet, which still described failure cleanup as "the pre-existing cleanup path alone" — directly contradicting this document's own "Failure Semantics" section; both now consistently attribute failure cleanup to `ExecutionCoordinator::fail`, genuinely new to this milestone. **Correction 3:** corrected three locations ("Scope," "Trusted Core," "Required Interface Evolution") that understated `dispatch`'s own signature evolution as gaining only "a `behavior: &mut dyn Actor` parameter." Direct diff against the committed predecessor state shows `dispatch` gained two parameters (`message: &Message`, `actor: Option<&mut dyn Actor>`) and its return type changed from `Result<(), RuntimeError>` to `Result<Vec<Message>, RuntimeError>` — the mechanism by which emitted messages are captured; also corrected two secondary references ("Data and State Constraints," "Implementation Sequence" step 4) that inherited the same singular-parameter phrasing. Also updated every citation of ARCH-006's own version (v0.1.2 → v0.1.3, reflecting that document's own corresponding corrections). No architectural principle changed, no Trusted Core boundary changed, no engineering decision already recorded was reopened or reversed — this revision only completes disclosure of facts already true of the implementation. |

## Disposition

Draft. Not yet reviewed for publication. The implementation this document records is already complete, already tested, and already independently reviewed three times; this document's own governance disposition is independent of that fact, per STD-001's own document lifecycle (§12).
