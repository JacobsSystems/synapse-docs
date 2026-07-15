---
document_id: EWO-006
title: "Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection"
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-13
last_updated: 2026-07-13
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.4.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
  adrs:
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
    - ADR-0017 (Approved)
  predecessor: EWO-005 (work-orders/EWO-005-Truthful-Execution-State-Tracking.md) — prior Runtime Integration milestone; this EWO implements the mandatory "bounded mailboxes" conformance gap ARCH-002 §13/§22 and ARCH-003 §5/§18 have disclosed since EWO-001
  reported_by: ER-006 (engineering-reports/ER-006-Bounded-Actor-Mailboxes.md, not yet created)
  base_state:
    runtime_head: 5ccc7f9083a71adc6ee704b2322a701935765679
    docs_head: e90404baa5140ce9004839bc51921c789777e003
---

# EWO-006 — Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection

Registered per STD-001 §46 (Engineering Work Orders). This is the third Engineering Work Order issued under the Runtime Integration phase ARCH-003 opens, and the first to close a mandatory ARCH-002 conformance gap disclosed, unchanged, since EWO-001. This document authorizes engineering work only. It does not itself constitute approval, implementation, or completion.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-006 |
| Title | Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection |
| Version | 0.1.0 |
| Status | **Draft** — not yet approved, not yet implemented |
| Author | Denver Jacobs |
| Owner | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-13 |
| Last Updated | 2026-07-13 |
| Classification | Public |
| Applicable repository | `synapse-runtime` |
| Target branch | `main` |
| Runtime base HEAD | `5ccc7f9083a71adc6ee704b2322a701935765679` |
| Documentation base HEAD | `e90404baa5140ce9004839bc51921c789777e003` |
| Predecessor milestone | EWO-005 — Runtime Integration: Truthful Actor Execution-State Tracking |
| Reported by | ER-006 (engineering-reports/ER-006-Bounded-Actor-Mailboxes.md, not yet created) |

No implementation exists yet against this EWO. No approval act has occurred. This document authorizes a bounded scope of future engineering work; it does not report on work already done.

---

## Engineering Authority

This implementation is governed by, in descending order:

1. ARCH-001 — Constitutional Architecture
2. ARCH-002 — Runtime Architecture, **v0.2.0** (architecture/ARCH-002-Runtime-Architecture.md) — the authority for the Mailbox Model (§13: bounded, finite capacity as a mechanism-level MUST; overflow as a mandatory, non-silent, audited response) and for bounded mailboxes' status among Mandatory conformance requirements (§22)
3. ARCH-003 — Runtime Integration Architecture, **v0.4.0** (architecture/ARCH-003-Runtime-Integration-Architecture.md) — the specific authority for this milestone's scope and boundaries, in particular §5 (Current Implementation Baseline, disclosing unbounded mailboxes since EWO-001) and §18 (Deferred Integration Work, listing "bounded mailbox capacity and audited overflow handling")
4. ADR-0015 — Audit Emitter Failure Semantics (Approved)
5. ADR-0016 — Trusted Core Interaction Rule (Approved)
6. ADR-0017 — Bootstrap Capability Trust Root (Approved)
7. STD-001 — Documentation Standards (§46, Engineering Work Orders)

These documents are authoritative. This task implements them. It does not reinterpret or modify them.

---

## Purpose

This EWO exists to close the one mandatory ARCH-002 conformance gap that has remained open, unchanged and explicitly disclosed, since Actor Host's own first implementation (EWO-002/SRP-002/SRP-003): actor-instance mailboxes are currently unbounded, and mailbox overflow can never occur or be rejected. ARCH-002 §13 states plainly that "capacity is bounded and finite" is a mechanism-level MUST and that overflow "MUST produce a defined, non-silent response"; §22 lists bounded mailboxes among its Mandatory conformance requirements. ARCH-003 §5 has recorded this gap in every one of its revisions (v0.1.0 through v0.4.0) as "a mandatory ARCH-002 conformance item, not merely a convenience gap."

This is the first implementation milestone arising from the EWO-006 scope-determination review, which independently confirmed this is the smallest, fully unblocked, architecture-mandatory Runtime Integration item remaining — unlike every other deferred item ARCH-003 §18 lists, which is blocked either by the absence of an actor-defined handler contract, by the absence of a genuinely asynchronous execution mechanism, or by requiring a new Trusted Core interaction path.

---

## Problem Statement

Verified directly against `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (`core/actor-host/src/lib.rs`, `core/actor-host/src/internal.rs`, `core/message-gateway/src/lib.rs`, `runtime/src/lib.rs`, `common/src/lib.rs`), not assumed:

- `ActorHostImpl`'s mailbox storage is `mailboxes: HashMap<ActorInstanceId, Vec<Message>>`. `enqueue` unconditionally pushes onto the target instance's `Vec` if the instance exists, and fails only with `RuntimeError::UnknownTarget` if it does not. No capacity field, constant, or placeholder exists anywhere in this crate.
- `Message Gateway::admit`'s own doc comment already, explicitly disclaims capacity responsibility: "it does not itself place the message into a mailbox or enforce bounded capacity... mailbox placement and capacity are Actor Host's mechanism." This confirms directly, from the crate whose boundary is most often confused with Actor Host's, that capacity ownership already, unambiguously belongs to Actor Host.
- `RuntimeError::Overflow` already exists in `synapse-common`'s shared error enum, and is used today **only** inside `runtime/src/lib.rs`'s own test module (the `FailingAuditEmitter` test double and its assertions) — it is completely unused in any production code path in this workspace. Its name and existing usage pattern (representing "a bounded resource has no remaining capacity") already, honestly fit mailbox overflow with no reinterpretation required.
- `Runtime::submit_message`'s step 6 already, unconditionally, routes any error `self.core.actor_host.enqueue(&instance, message.clone())` returns to `self.reject_message(&message, reason)`, which emits the existing `message.rejected` audit event (ARCH-002 §18) and returns the triggering error unchanged. This handling requires no modification to accommodate a new Actor Host rejection cause — it was already generic across every possible `enqueue` error before this EWO.
- No dequeue or drain mechanism exists anywhere in the current implementation: nothing removes an individual message from a mailbox once admitted. The only removal is whole-mailbox deletion, performed by `terminate_instance`. This is consistent with ARCH-003 §5's own disclosure that "no actor logic is actually invoked" at this milestone — nothing currently consumes mailbox contents.

None of the above requires correction beyond what this EWO itself authorizes; each fact above is cited because it directly bounds and justifies this EWO's own minimal scope.

---

## Architectural Authority

Distinguishing the concerns this EWO touches, per already-published architecture:

| Concern | Owner | Authority |
|---|---|---|
| Mailbox existence, one per live instance | Actor Host | ARCH-002 §7, §13; ARCH-001 §5.1 (Actor's constitutional mailbox ownership) |
| Mailbox capacity and overflow rejection | Actor Host | ARCH-002 §13 ("mechanism-level MUST"); confirmed disclaimed by Message Gateway's own `admit` documentation |
| Overflow audit obligation | Audit Emitter, via Runtime's existing `message.rejected` path | ARCH-002 §13 ("reject-with-audit-event is the mandatory mechanism-level contract"), §18 (message rejection already named a minimum audit event) |
| Connecting components | Runtime, exclusively | ADR-0016 — unaffected by this EWO, since no new cross-component connection is introduced |
| Overflow policy variant (reject-newest/reject-oldest/backpressure), priority reordering, deduplication | Deferred, not this milestone | ARCH-002 §13 (explicitly named policy, deferred) |

STD-001 §46 governs this document's own form and authority: an EWO "MAY authorize implementation... MUST NOT redefine Architecture... If implementation... reveals an apparent architectural contradiction, the EWO MUST require engineering to stop and return the issue for architectural review rather than resolve it unilaterally." This EWO's own Stop Conditions (below) apply that rule concretely.

---

## Objective

Introduce bounded actor-instance mailboxes with deterministic overflow rejection inside Actor Host, while preserving existing Runtime composition, existing audit behaviour, existing public interfaces, and existing architectural ownership. Enqueue succeeds while the target mailbox's current length is below capacity; enqueue is rejected with the existing `RuntimeError::Overflow` once capacity is reached. No other Runtime, Message Gateway, or shared-type behaviour changes.

---

## Bounded Design Decision (the only decision this EWO leaves open)

ARCH-002 §13 requires that mailbox capacity be "bounded and finite" — a mechanism-level MUST — but explicitly names "the specific number" a **deployment parameter (policy)**, not an architectural guarantee. No numeric default exists anywhere in the published architecture, in any prior EWO, or anywhere in the current implementation.

**This EWO authorizes exactly one open engineering decision:** the implementer shall select a small, conservative, fixed mailbox-capacity constant. This EWO does not select that value, for want of any evidentiary basis to prefer one number over another — inventing a specific figure here would be exactly the kind of unjustified, unevidenced decision this engagement's own conventions (established across EWO-001 through EWO-005) consistently avoid.

Requirements on the choice, all binding:

- The value SHALL be small and conservative — sufficient for the current single-process, non-distributed, no-real-actor-execution Minimal Runtime Profile, not tuned for any anticipated production load (none is architecturally specified).
- The value SHALL be a fixed, documented, crate-internal constant — not a constructor parameter, not a per-actor-instance or per-actor-definition value, and not backed by any configuration mechanism. See "Architecture Constraints" below for why.
- The value and its rationale SHALL be recorded in ER-006.
- The value is an implementation policy choice, explicitly authorized by ARCH-002 §13's own "deployment parameter" language — it is not, and must not be treated as, an architectural guarantee. A future milestone may revisit it without requiring an ADR, provided the revisit does not change capacity's ownership, mechanism, or the reject-only (non-backpressure) policy this EWO fixes.

No other design decision is authorized to remain open. If implementation determines a second, materially different decision is required, this is a stop condition (see below), not something to resolve unilaterally.

---

## Scope

Implement only:

- A capacity check inside `ActorHostImpl::enqueue` (`core/actor-host/src/internal.rs`), comparing the target mailbox's current length against the fixed capacity constant before pushing, rejecting with `RuntimeError::Overflow` if the mailbox is already at capacity.
- The fixed capacity constant itself, defined within `synapse-actor-host`, per "Bounded Design Decision" above.
- Actor Host unit tests proving the required behaviour (see "Required Tests" below).
- Runtime tests proving the existing `submit_message` → `reject_message` → `message.rejected` path correctly, unconditionally, handles this new rejection cause — using only the existing public API (no test-only seam is required or authorized; see "Required Tests").
- Documentation updates to `core/actor-host/README.md`, strictly limited to describing the newly implemented capacity bound and its disclosed dequeue-related limitation.

---

## Explicit Exclusions

Do NOT implement, and do not let any of the following creep into this milestone's scope:

- Dequeue, drain, or any mechanism to remove an individual message from a mailbox once admitted.
- Mailbox draining on shutdown or termination beyond the existing whole-mailbox removal `terminate_instance` already performs.
- Real actor-defined handler invocation of any kind.
- Scheduler work of any kind.
- Capability revalidation at invocation or during restoration.
- Lifecycle Guardian work of any kind, including suspend/restore reachability.
- Asynchronous, concurrent, or otherwise interruptible execution of any kind.
- Blocking enqueue of any kind.
- Retry of any kind.
- Dead-letter queues or any dead-letter storage mechanism beyond the existing mandatory rejection-audit event.
- Persistence of any kind.
- Priority queuing or reordering of any kind.
- A generic configuration framework, or any mechanism to make capacity runtime-configurable, per-actor-definition-configurable, or per-deployment-configurable.
- Distributed queues or cross-host mailbox behaviour of any kind.
- SDK or host-specific implementation work.
- Any architecture, ADR, or STD-001 document change.
- A new audit event type of any kind — the existing `message.rejected` event is sufficient and this EWO does not add another.
- A new `RuntimeError` variant of any kind — the existing `Overflow` variant is sufficient.
- Any change to `enqueue`'s public signature, to `Runtime`'s public interface, to Message Gateway's public interface, or to any shared type in `synapse-common`.
- A new or redesigned `ActorHostHandle`/`ActorHostImpl` constructor.
- Any Runtime construction or composition-root plumbing of any kind.

---

## Trusted Core

Actor Host remains one of exactly seven Trusted Core components (ARCH-002 §6). No new Trusted Core component is introduced. No responsibility is transferred: Actor Host continues to own only actor-instance existence, liveness, and mailbox mechanics, exactly as ARCH-002 §6 already assigns. No other Trusted Core component's responsibility, ownership, or prohibition is touched. Runtime's role remains exactly what it already is — composing and sequencing, per ARCH-003 §6 ("Runtime composes and sequences; it does not decide") — since no new cross-component connection is introduced by this milestone at all; the existing `submit_message` → `reject_message` path already, generically, handles this new rejection cause without modification.

---

## Architecture Constraints

Do not:

- introduce new Runtime concepts;
- invent lifecycle states, audit events, or capability checks;
- modify constitutional concepts;
- reinterpret ARCH-001, ARCH-002, or ARCH-003;
- redesign the `ActorHost` trait or `ActorHostHandle`'s public interface in any way;
- redesign `Runtime`'s or Message Gateway's public interface in any way;
- introduce new crates;
- introduce external dependencies;
- use unsafe Rust;
- give `synapse-actor-host` a dependency on any other Trusted Core crate, or the reverse, beyond what already exists (`synapse-common` only);
- introduce a generic configuration mechanism of any kind.

If implementation requires any of the above: STOP. Produce an Engineering Report explaining the issue. Do not invent a solution.

---

## Repository Constraints

Do not modify: governance documents; architecture documents; standards; work orders other than this one (including, but not limited to, `EWO-003-Message-Gateway.md`, which remains an untracked, protected file this EWO does not touch); repository structure beyond `core/actor-host/` and its manifest/README. Do not modify `synapse-execution-coordinator`, `synapse-host-adapter`, `synapse-lifecycle-guardian`, `synapse-message-gateway`, `synapse-capability-authority`, `synapse-audit-emitter`, `runtime/`, or `common/` in any way — none of them requires any change to realize this milestone. If implementation determines any of them genuinely requires a change, STOP per Stop Conditions rather than proceeding.

---

## Required Interface Evolution

**None.** This is a deliberate, verified property of this milestone, not an oversight: `enqueue`'s existing signature — `fn enqueue(&mut self, instance: &ActorInstanceId, message: Message) -> Result<(), RuntimeError>` — already returns a `Result<(), RuntimeError>` capable of representing the new rejection cause without any change. `RuntimeError::Overflow` already exists. No new public method, type, parameter, or constructor is authorized or required. This mirrors, in miniature, EWO-004's own "minimum interface evolution" discipline — except here, the minimum evolution required is genuinely zero at the public-interface level, since both the error type and the Runtime-level handling path already, generically, existed before this EWO.

---

## Runtime Behaviour

`Runtime::submit_message` requires **no code change**. Its existing step 6 — `self.core.actor_host.enqueue(&instance, message.clone())`, routed to `self.reject_message(&message, reason)` on any `Err` — already, unconditionally, handles `Err(RuntimeError::Overflow)` exactly as it already handles `Err(RuntimeError::UnknownTarget)`: no subsequent step runs, `message.rejected` is emitted (ARCH-002 §18), and the triggering error (`Overflow`) is returned to the caller — unless that rejection-audit emission itself fails, in which case the audit failure is what the caller observes, exactly as ADR-0015 already, generally requires. Runtime remains only the composition root; it decides nothing about capacity, and this milestone gives it no new decision to make.

---

## Actor Host Behaviour

`ActorHostImpl::enqueue`'s required behaviour, in order:

1. Reject with `RuntimeError::UnknownTarget` if `instance` does not exist — unchanged from today.
2. If `instance` exists, compare the target mailbox's current length (`mailbox.len()`) against the fixed capacity constant.
3. If the current length is below capacity: push the message, exactly as today; return `Ok(())`.
4. If the current length equals capacity: do not push the message; return `Err(RuntimeError::Overflow)`.

No other method (`define`, `create_instance`, `terminate_instance`, `live_instance`) changes in any way.

---

## Overflow Semantics

- Mailbox length below capacity → enqueue succeeds, message is admitted in FIFO position, exactly as today.
- Mailbox length equals capacity → enqueue fails with `RuntimeError::Overflow`; the message is never inserted.
- Overflow is reject-only. No backpressure, no blocking, no retry, no reject-oldest/reject-newest alternative policy is implemented — ARCH-002 §13 names these as deferred policy choices among which this EWO selects the simplest, already-implied one (reject the incoming message; leave the queue as-is).
- Overflow does not affect any other actor instance's mailbox — capacity and length are already, structurally, tracked per instance (`HashMap<ActorInstanceId, Vec<Message>>`), so isolation is a direct, pre-existing consequence of the existing storage shape, not new work this EWO performs.
- Overflow does not terminate, suspend, or otherwise affect the target actor instance's lifecycle state in any way — Lifecycle Guardian is not called, is not aware, and its own tracked state is untouched by this milestone.

---

## Failure Semantics

- **Overflow rejection:** handled entirely by the existing `Runtime::reject_message` path (see "Runtime Behaviour" above) — no new failure-handling code exists or is required anywhere outside `ActorHostImpl::enqueue`'s own capacity check.
- **Audit-emission failure on a rejection caused by overflow:** identical to audit-emission failure on any other `submit_message` rejection today — per ADR-0015, the audit failure is what the caller observes instead of the triggering `Overflow` error, with no rollback of anything (there is nothing to roll back: the message was never inserted).
- **No new failure mode exists.** This EWO introduces exactly one new way an existing method can fail (`enqueue` returning `Overflow` instead of always succeeding for a known instance), handled by exactly the pre-existing, generic mechanism every other `enqueue` failure already uses.

---

## State Invariants

The following hold both before and after this EWO, and must continue to hold:

1. Exactly one mailbox exists per live actor instance — unchanged.
2. Actor creation starts with an empty mailbox — unchanged; this EWO adds a capacity ceiling, not a different starting state.
3. Mailbox ownership never leaves Actor Host — unchanged; no other component gains a path to mailbox contents or capacity.
4. Mailbox capacity and length are isolated per actor instance — a full mailbox for one instance has no effect on any other instance's capacity or contents.
5. A rejected enqueue does not modify mailbox contents in any way — the mailbox after a rejected enqueue is byte-for-byte identical to the mailbox immediately before the attempt.
6. FIFO ordering of already-admitted messages is unchanged by this EWO, and unaffected by any rejection.

**Honest, explicitly disclosed limitation — not solved by this EWO:** no dequeue or drain mechanism exists anywhere in the current implementation (see "Problem Statement" above). Consequently, once a mailbox reaches capacity, it remains at capacity — permanently full, permanently rejecting further `enqueue` calls with `Overflow` — until the owning actor instance is terminated (which removes the mailbox entirely via `terminate_instance`, exactly as today). This is an honest, disclosed consequence of the current Minimal Runtime Profile not yet invoking real actor logic (ARCH-002 §21; ARCH-003 §5, §10) — nothing currently consumes mailbox contents, so nothing currently frees capacity short of termination. This EWO does not propose, and is not scoped to propose, any solution to this — it is recorded here so it is never silently assumed away, exactly as ARCH-003 itself never silently assumes away a disclosed gap.

---

## Audit Semantics

- No new audit event type is authorized by this EWO.
- The existing `message.rejected` event (ARCH-002 §18) is the sole audit consequence of an overflow rejection, emitted via the existing, unmodified `Runtime::reject_message` path.
- Overflow is not distinguished from any other `submit_message` rejection cause by a separate event or a separate field — `AuditEvent`'s existing shape (`event_type`, `actor`, `capability`, `message`) already carries exactly what every other rejection already carries, and this EWO does not extend it.
- Audit emission continues to occur only where ARCH-002 §18 already authorizes it, per ARCH-003 §6's own Integration Principle ("no integration described here introduces an audit obligation ARCH-002 does not already name") — this EWO does not introduce an integration in the ARCH-003 sense at all, since no new cross-component connection is created.

---

## Definition of Done

The task is complete only if all of the following are true:

- `ActorHostImpl::enqueue` rejects with `RuntimeError::Overflow` when, and only when, the target mailbox is already at the chosen fixed capacity.
- Enqueue below capacity behaves exactly as it did before this EWO.
- A rejected enqueue never inserts the message and never alters existing mailbox contents or order.
- Mailbox capacity and overflow are isolated per actor instance.
- `Runtime::submit_message`'s own code is unchanged; overflow is handled entirely by its existing rejection path.
- `message.rejected` is emitted on overflow, via the existing mechanism, with no new audit event anywhere.
- No public interface — `ActorHost`, `Runtime`, `MessageGateway`, or any `synapse-common` type — changed in any way.
- The chosen capacity constant and its rationale are recorded in ER-006.
- `synapse-actor-host`'s dependency set remains exactly `synapse-common`. No new dependency of any kind, in any crate.
- All pre-existing Actor Host and Runtime tests continue to pass unmodified in behaviour.
- All tests pass. No warnings. No unsafe.
- Trusted Core boundary unchanged (still exactly seven components). Architecture unchanged.

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

All must pass. Additionally verify: workspace member count unchanged; zero `unsafe`; no dependency cycle; `cargo tree -p synapse-actor-host` shows exactly `synapse-common`; `Runtime::submit_message`'s own public signature and `ActorHost::enqueue`'s own public signature are byte-for-byte unchanged.

---

## Data Constraints

No new state is required beyond what already exists:

- `ActorHostImpl` requires exactly one addition: the fixed capacity constant (or, if implementation determines a `const` alone cannot satisfy the capacity check cleanly, a single, non-configurable, compile-time-fixed value stored however is idiomatic — no runtime-mutable field, no constructor parameter).
- No new field is added to `Message`, `ActorInstanceId`, `ActorId`, or any other shared type.
- No new field is added to `Runtime`, `TrustedCore`, or any other Runtime-level type.
- Do not add: a capacity registry, a per-instance override map, a configuration struct, or any bookkeeping beyond the single constant this EWO authorizes.

---

## Definition of Failure / Stop Conditions

Stop immediately, and produce an Engineering Report rather than resolving the issue unilaterally, if any of the following occurs:

1. Bounded mailbox capacity is found to require a change to any public interface (`ActorHost`, `Runtime`, `MessageGateway`, or any `synapse-common` type).
2. Overflow rejection is found to require a new `RuntimeError` variant, because `Overflow` proves insufficient or already semantically committed elsewhere in a way that conflicts with this use.
3. Overflow rejection is found to require a new audit event, because the existing `message.rejected` event proves insufficient.
4. `Runtime::submit_message` is found to require any code change to correctly handle overflow.
5. Capacity is found to require per-actor-instance or per-actor-definition differentiation to satisfy any architectural requirement (none is currently known to exist).
6. A configuration mechanism is found to be architecturally required, rather than merely a plausible future enhancement.
7. Isolating capacity per actor instance is found to require anything beyond the existing per-instance `HashMap` keying already in place.
8. Any Trusted Core component other than Actor Host is found to require modification.
9. A direct dependency between `synapse-actor-host` and any other Trusted Core crate is found to be required.
10. Any ADR guarantee or ARCH-002/ARCH-003 architectural boundary is found to require change to complete this milestone.
11. The dequeue-related limitation disclosed under "State Invariants" is found to require resolution to satisfy this EWO's own Definition of Done (it does not — resolving it is explicitly out of scope).

Do not resolve these issues yourself. Report them.

---

## Implementation Sequence

1. Baseline verification: re-confirm the current `core/actor-host/` state matches this EWO's "Problem Statement" section exactly.
2. Select the fixed capacity constant, per "Bounded Design Decision" above; record the choice and rationale for ER-006.
3. Implement the capacity check inside `ActorHostImpl::enqueue`, per "Actor Host Behaviour" above.
4. Component-local tests: unit tests within `synapse-actor-host` proving the required behaviour, per "Required Tests" below.
5. Runtime tests: within `runtime`'s existing test module (matching current convention), prove the existing rejection path correctly handles overflow, per "Required Tests" below.
6. Regression check: confirm all pre-existing Actor Host and Runtime tests still pass unmodified in outcome.
7. Documentation updates: `core/actor-host/README.md`, reflecting the real behaviour now present, including explicit disclosure of the dequeue-related limitation named under "State Invariants."
8. Complete quality validation, per Mandatory Validation above.
9. ER-006 preparation after implementation and validation are complete — not before, and not as part of this EWO.

Do not include work from later milestones.

---

## Acceptance Criteria

Every criterion is objective and testable:

- Actor Host alone changes (`core/actor-host/src/internal.rs`, its tests, and its README) — no other crate's source changes.
- `Runtime`'s public behaviour is unchanged; its existing rejection path is exercised, not modified.
- `RuntimeError::Overflow` is reused, not redefined or duplicated.
- `message.rejected` is reused; no new audit event exists anywhere in the implementation.
- No public interface — `ActorHost`, `Runtime`, `MessageGateway`, or any `synapse-common` type — changes in any way.
- All pre-existing tests pass unmodified in outcome.
- New tests, per "Required Tests" below, are added and pass.
- `cargo fmt --all -- --check`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`, `cargo build --workspace`, and `cargo test --workspace` all pass with zero warnings.
- `git status` after implementation shows changes confined to `core/actor-host/` and its manifest/README — no unrelated file touched.
- Architecture and ADR documents remain byte-for-byte untouched.

---

## Required Tests

### Actor Host tests (`synapse-actor-host`)

At minimum:

- Enqueue succeeds while the mailbox's current length is below the chosen capacity.
- Enqueue succeeds when it brings the mailbox exactly to capacity.
- The next enqueue attempt, once the mailbox is at capacity, is rejected with `RuntimeError::Overflow`.
- The rejected message is not present in the mailbox afterward (using the existing `mailbox_message_ids` same-crate test seam, matching established precedent).
- Existing, already-admitted messages remain in unchanged FIFO order after a rejected enqueue.
- Capacity and overflow are isolated per actor instance: filling one instance's mailbox to capacity does not affect a second instance's ability to enqueue.
- Enqueue against an unknown instance continues to fail with `RuntimeError::UnknownTarget`, unaffected by this change.
- A newly created actor instance starts with an empty mailbox (capacity available from zero).
- All pre-existing Actor Host tests continue to pass unmodified in outcome.

### Runtime tests (`runtime`)

At minimum:

- `Runtime::submit_message` returns `Err(RuntimeError::Overflow)` once the target instance's mailbox is filled to capacity through repeated, ordinary `submit_message` calls via the existing public API alone — no test-only seam is required or authorized, since this scenario is genuinely reachable through public, sequential use (unlike EWO-005's internally-seeded rejection paths).
- The existing `message.rejected` event is emitted on the overflow rejection, exactly as it is for any other `submit_message` rejection cause.
- Existing `submit_message` tests (envelope, send-authority, capability-validity, existence rejections; successful admission) continue to pass unmodified in outcome.

### Not required

- Any test proving a later enqueue succeeds after a dequeue, drain, or capacity-freeing event — no such mechanism exists to test, per "State Invariants" above. Do not attempt to manufacture one via an unauthorized test seam.
- Any Message Gateway test change — `admit`'s own behaviour is untouched by this EWO.

---

## Engineering Decision Log

Record:

- implementation decisions (expected: the chosen capacity constant and its rationale, per "Bounded Design Decision");
- repository decisions;
- deferred decisions (expected: none beyond what this EWO itself already defers, per "Explicit Exclusions");
- architectural decisions (expected: None);
- constitutional decisions (expected: None);
- future work enabled (expected: a genuinely conformant, ARCH-002 §22-satisfying mailbox mechanism that later milestones — real actor execution, in particular — can build a dequeue/drain mechanism against, once actor logic exists to consume messages);
- future work deferred (expected: dequeue/drain mechanism design; overflow policy alternatives beyond reject-only; capacity configurability; every other item ARCH-003 §18 still lists, unaffected in status by this EWO).

---

## Completion Report

ER-006 must provide, after implementation:

1. Files modified.
2. Files created (expected: none beyond test files within `synapse-actor-host`).
3. The chosen capacity constant and its rationale.
4. Actor Host behaviour implemented, confirmed against "Actor Host Behaviour" and "Overflow Semantics" above.
5. Confirmation that `Runtime::submit_message` required no code change, with the exact call sequence re-verified against the source.
6. Tests added, mapped against "Required Tests" above.
7. Validation results (Mandatory Validation, in full).
8. Dependency changes (expected: none).
9. Trusted Core changes (expected: none — same seven components, same boundaries).
10. Architecture changes (expected: none).
11. Explicit re-confirmation of the disclosed dequeue-related limitation named under "State Invariants," stating whether it remains accurate post-implementation.
12. Engineering Decision Log.
13. Any Stop Condition encountered, and its resolution status.
14. Confirmation that ARCH-003 itself should be updated (per its own §20 conformance requirement) to reflect this item's new implemented status, per the same pattern already used for EWO-004 and EWO-005.

Stop after this milestone. Do not begin the next Runtime Integration engineering milestone.

---

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-13 | Denver Jacobs | Initial EWO, registered per STD-001 §46. Authorizes the first implementation milestone closing ARCH-002 §13/§22's mandatory bounded-mailbox conformance gap, disclosed unchanged in ARCH-003 §5 since EWO-001, per the approved EWO-006 scope-determination review. Derived exclusively from ARCH-001, ARCH-002 (v0.2.0), ARCH-003 (v0.4.0), ADR-0015 through ADR-0017, STD-001, and the verified current state of `core/actor-host/`, `core/message-gateway/`, `runtime/`, and `common/` at commit `5ccc7f9083a71adc6ee704b2322a701935765679`. Resolves the one bounded design decision the scope-determination review identified (fixed conservative capacity value) by deferring the literal numeric choice to implementation, per ARCH-002 §13's own "deployment parameter" framing. |

## Disposition

Draft. Not yet reviewed. Not yet approved. Not yet implemented.
