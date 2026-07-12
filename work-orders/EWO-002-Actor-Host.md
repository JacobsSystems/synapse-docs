---
document_id: EWO-002
title: Actor Host
version: 0.3.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-12
last_updated: 2026-07-12
classification: Public
related_documents:
  standards:
    - STD-001
    - STD-002
    - STD-004
    - STD-011
  architecture:
    - ARCH-001
    - ARCH-002
  adrs:
    - ADR-0013
    - ADR-0014
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
  predecessor: EWO-001 (work-orders/EWO-001-Runtime-Bootstrap.md)
  reported_by: ER-002 (engineering-reports/ER-002-Actor-Host.md)
---

# EWO-002 — Actor Host

Registered per STD-001 §46 (Engineering Work Orders). Revised to integrate ADR-0015 and ADR-0016, both approved and integrated into the committed ARCH-002 (commit `2629241`); see Revision History.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-002 |
| Title | Actor Host |
| Version | 0.3.0 |
| Status | Draft |
| Author | Denver Jacobs |
| Created | 2026-07-12 |
| Last Updated | 2026-07-12 |
| Classification | Public |
| Predecessor milestone | EWO-001 — Runtime Bootstrap |
| Reported by | ER-002 (engineering-reports/ER-002-Actor-Host.md) |

---

## Engineering Authority

This implementation is governed by:

### Standards

- STD-001
- STD-002
- STD-004
- STD-011

### Architecture Decisions

- ADR-0013
- ADR-0014
- ADR-0015 — Audit Emitter Failure Semantics (Approved)
- ADR-0016 — Trusted Core Interaction Model (Approved)

### Architecture

- ARCH-001
- ARCH-002 (as amended at commit `2629241`, integrating ADR-0015 and ADR-0016)

These documents are authoritative. This task implements them. It does not reinterpret or modify them.

---

## Objective

Implement the Actor Host trusted-core component's identity-assignment and instance-lifecycle behaviour exactly as ARCH-002 defines it: `define`, `create_instance`, and `terminate_instance`, and nothing else.

Completion means: the `synapse-actor-host` crate's existing `ActorHost` trait has a real, non-opaque implementation satisfying every invariant ARCH-002 §3, §6, §7, §11 (steps 2–3, 17), §15, §17, §18, §20, and §22 state for those three operations, backed by tests that exercise each invariant directly.

---

## Existing Baseline (verified against the source tree, not assumed)

`core/actor-host/` already exists (established under EWO-001, SRP-0). Its current state:

- `src/lib.rs` defines `pub trait ActorHost` with exactly three methods — `define`, `create_instance`, `terminate_instance` — already citing ARCH-002 §11 (steps 2–3) and §20 in its own doc comments. No default method bodies exist.
- `src/lib.rs` also defines `pub struct ActorHostHandle`, an opaque construction handle introduced by EWO-001 for Runtime Bootstrap purposes only. It implements no trait.
- `src/internal.rs` defines `pub(crate) struct ActorHostImpl`, a unit struct with only a `new()` constructor. Its doc comment states isolation-enforcement mechanics are "deliberately left undecided" and that "the trait itself remains unimplemented."
- The crate depends only on `synapse-common` (`ActorId`, `ActorInstanceId`, `RuntimeError`).
- The Runtime composition root (`runtime/`) currently constructs `ActorHostHandle::new()` only, to prove construction succeeds. It does not call any `ActorHost` trait method. It already depends on `synapse-audit-emitter` (for its own Runtime-level start/shutdown events, EWO-001) and on `synapse-actor-host`.
- No test in the crate exercises actual behaviour — only `trait_is_object_safe` and `handle_is_constructible`.

This EWO's job is to give `ActorHostImpl` a real implementation of the already-correctly-scoped trait, and to make that behaviour reachable in the one place this milestone genuinely requires it to be reachable (the audit interaction, below) — not to redesign the trait, the crate boundary, or the composition root beyond what that requires.

---

## Scope

Implement only:

- Real behaviour for `define(&mut self, definition_name: &str) -> Result<ActorId, RuntimeError>`: assigns a logical identity to a newly defined actor (ARCH-002 §11, step 2; §7). See "Identity Generation Constraints" and "Meaning of 'Stable'" below for the exact bounds of this behaviour.
- Real behaviour for `create_instance(&mut self, actor: &ActorId) -> Result<ActorInstanceId, RuntimeError>`: creates a new, distinct instance identity for a previously defined actor, establishing per-instance state isolation before any execution occurs (ARCH-002 §11, step 3; §7). See "Define-Before-Create" below for the authoritative basis.
- Real behaviour for `terminate_instance(&mut self, instance: &ActorInstanceId) -> Result<(), RuntimeError>`: tears down an instance without leaking its isolated state (ARCH-002 §20, "Actor termination... Must not leak isolated state").
- The invariant that `create_instance` fails for an `ActorId` that was never defined.
- The invariant that `terminate_instance` fails for an `ActorInstanceId` that does not exist or was already terminated.
- Private, per-instance actor-state storage, held internally and exposed through no public accessor (ARCH-001 §5.1; ARCH-002 §7: "Actor state is private, held per-instance, enforced by Actor Host"). Because message handling is out of scope for this milestone, this storage holds no actor-defined payload yet — it exists only to the extent needed to prove isolation is structurally enforced.
- Isolation delivered by ordinary language-level enforcement (private fields, no shared mutable state, no public mutation path) — one of the three mechanisms ARCH-002 §17 explicitly leaves open. Language-level enforcement is the only one of the three this single-process Rust workspace, with no process-management code anywhere in it, can currently realize; selecting it is not a new architectural decision.
- The two audit events ARCH-002 §18 requires at this milestone — actor creation, actor termination (not actor definition, which §18 does not list) — being caused to reach Audit Emitter, exactly as "Audit Interaction" below defines, including whatever minimal, narrowly-bounded construction-surface and Runtime-composition-root change that section authorizes.
- The observable rule that `create_instance` and `terminate_instance` are not reported successful unless their required audit emission succeeds, exactly as "Audit Failure Semantics" below defines.
- Unit tests, within the `synapse-actor-host` crate, and the minimum integration test described in "Audit Interaction," for every Definition-of-Done item.

---

## Out of Scope

Do NOT implement:

- Message Gateway behaviour (message validation, admission, mailboxes)
- Capability Authority behaviour (capability issuance, validation, binding) — ARCH-002 §7 is explicit that "capability bindings are held by Capability Authority," not Actor Host
- Execution Coordinator behaviour (Execution Context construction, dispatch mechanics)
- Lifecycle Guardian behaviour, including the full `ActorState` transition set (`Idle ⇄ Ready → Executing → {Idle | Suspended | Failed | Stopping}`) — this milestone handles only the boundary between "instance does not exist," "instance exists," and "instance terminated." Suspension, restoration, and execution-driven transitions remain Lifecycle Guardian's domain and are not implemented here.
- Host Adapter behaviour beyond what already exists
- Scheduling or dispatch order
- Message delivery of any kind
- Persistence, snapshotting, or restoration
- Networking or distributed execution (remote actors, cross-host routing, location transparency mechanics)
- Any replaceable service (Scheduler, Actor Directory, Audit Pipeline, Persistence)
- Actor-authored message-handling logic (`synapse-api`'s `Actor` trait is untouched)
- A direct dependency from `synapse-actor-host` on `synapse-audit-emitter`, or on any other Trusted Core crate. This is a standing prohibition under ADR-0016 and the now-integrated ARCH-002 §6, not a question implementation may revisit.
- Any Runtime role beyond the narrow, bounded coordination "Audit Interaction" below authorizes — the Runtime does not become a scheduler, general workflow engine, or owner of Actor Host's own behavioural semantics (ADR-0016; ARCH-002 §3).
- Any construction-surface or public-API change beyond the minimum "Audit Interaction" below authorizes.
- Any later SRP milestone

---

## Trusted Core

Actor Host remains one of exactly seven Trusted Core components (ARCH-002 §6). No new Trusted Core component is introduced. No responsibility is transferred from another Trusted Core component into Actor Host, or from Actor Host into another component. Actor Host does not absorb any replaceable-service responsibility. No architectural boundary is weakened for implementation convenience.

---

## Architecture Constraints

Do not:

- introduce new Runtime concepts;
- invent actor lifecycle states beyond the defined/exists/terminated boundary this EWO scopes;
- modify constitutional concepts;
- reinterpret ARCH-001;
- reinterpret ARCH-002;
- redesign the Actor Host trait or the crate boundary;
- introduce new crates;
- introduce external dependencies;
- use unsafe Rust;
- give `synapse-actor-host` a dependency on `synapse-audit-emitter` or any other Trusted Core crate. ADR-0016 and the now-integrated ARCH-002 §3/§6 make the Runtime the sole entity accountable for establishing and coordinating Trusted Core interaction; Actor Host independently owning a path to Audit Emitter would itself be exercising that accountability, which is not Actor Host's to exercise.

If implementation requires any of the above: STOP. Produce an Engineering Report explaining the issue. Do not invent a solution.

---

## Repository Constraints

Do not modify: governance documents; architecture documents; standards; repository structure beyond `core/actor-host` and the minimum `runtime/` change "Audit Interaction" authorizes. Only modify source files necessary for Actor Host and that one integration point.

---

## Identity Generation Constraints

`define`'s identity-generation mechanism is implementation freedom only within these bounds, all derived from this milestone's own scope boundary rather than asserted independently:

- MUST NOT depend on distributed coordination or cross-host uniqueness — distributed execution is out of scope (Out of Scope, above; ARCH-002 §21 defers it).
- MUST NOT depend on persistence or durable storage — persistence is out of scope (ARCH-002 §6, Persistence/Restoration Service is a replaceable service, out of scope here).
- MUST NOT depend on an external randomness source or any external crate — no external dependency is authorized (Architecture Constraints, above).
- MAY use any in-process, deterministic-or-not mechanism (for example, a monotonic counter) that guarantees uniqueness within one running instance of the Runtime, since that is the only scope this milestone's tests can actually exercise.

## Meaning of "Stable" ActorId

ARCH-002 §7 states a logical identity is "assigned once at definition, stable across suspension, resumption, and restart." Suspension, resumption, and restart are Lifecycle Guardian behaviour and are out of scope for this milestone (Out of Scope, above). Therefore, for this milestone specifically: "stable" means only that once `define` returns an `ActorId`, no Actor Host operation implemented under this EWO subsequently changes, reassigns, or invalidates that value for as long as the defining actor's record exists. This milestone's tests may demonstrate that narrower claim only. They must not claim to demonstrate stability across suspension, resumption, or restart, since none of those are implemented here — that fuller guarantee is necessarily deferred to whichever milestone implements Lifecycle Guardian.

## Define-Before-Create — Authoritative Basis

Two independent textual bases, both in ARCH-002, establish this ordering; implementation must not treat it as inferred or assumed:

- §11's Execution Cycle table lists "Actor definition" as step 2 and "Actor creation" as step 3, with step 3's failure semantics stated as "Creation failure → remains Defined" — which presupposes the actor is already in a Defined state at the point creation is attempted.
- §15's Runtime Lifecycle explicitly opens the actor state machine at `Defined`, before any instance-related state: "Defined → Initializing... → Idle ⇄ Ready → Executing...".

## Existence Records vs. Lifecycle State Ownership

The internal records this EWO requires (which `ActorId`s are defined; which `ActorInstanceId`s exist, and are alive or terminated) are a narrow bookkeeping mechanism serving only Actor Host's own operational invariants (rejecting creation-before-definition, rejecting double-termination). They are not, and must not be implemented as, the authoritative `ActorState` lifecycle model ARCH-002 §15 defines, and they do not constitute lifecycle-transition ownership, which ARCH-002 §6 assigns to Lifecycle Guardian alone ("Enforces legal lifecycle-state transitions"). Implementation must keep these two concepts structurally separate — for example, an existence/liveness record is not a substitute for, and must not later be silently promoted into, an `ActorState` field or transition-validation logic.

---

## Audit Interaction

Resolved by ADR-0016, now integrated into ARCH-002 §3 and §6. This section states the resulting requirement and its bound; it does not reopen the question.

- **Actor Host owns the content of its own required audit events.** `create_instance` and `terminate_instance` already return the exact facts a creation or termination audit event needs — the resulting `ActorId`/`ActorInstanceId` on success, or an `Err` on rejection. Actor Host is not required to construct an `AuditEvent` value itself, reference `synapse-common`'s `AuditEvent` type, or know anything about `AuditEmitter` — its own return values are the content.
- **The Runtime owns establishing and coordinating the interaction.** Per ARCH-002 §3, the Runtime is the sole entity accountable for connecting Actor Host's operations to Audit Emitter. The Runtime composition root already depends on both `synapse-actor-host` and `synapse-audit-emitter` (established under EWO-001, for its own Runtime-level events) — no new dependency is required anywhere to realize this.
- **Actor Host must not establish or independently own a direct peer interaction path to Audit Emitter.** This is the standing prohibition stated in Architecture Constraints and Out of Scope above, restated here for the one place it is most directly relevant.
- **The Runtime must not absorb Actor Host's behavioural responsibility.** Coordinating the call is not deciding whether an operation succeeded, what identifier resulted, or what the operation means — those remain entirely Actor Host's, per its own `define`/`create_instance`/`terminate_instance` return values. The Runtime only constructs the audit event from those already-produced values and causes it to be emitted.
- **No engineering mechanism is prescribed.** This EWO does not authorize, and implementation must not introduce, a direct crate dependency from Actor Host to Audit Emitter, a new generic audit abstraction, an event bus, callbacks, service location, global state, a mediator framework, or a trait-object design adopted merely for hypothetical future flexibility. How the Runtime obtains and invokes Actor Host's behaviour is engineering discretion, bound only by the next paragraph.
- **Minimum authorized construction-surface change.** The existing `ActorHostHandle` implements no trait and cannot be invoked by anything outside the crate — it was sufficient for EWO-001's construction-only bootstrap, but is objectively insufficient now that the Runtime must actually invoke `define`/`create_instance`/`terminate_instance` to coordinate the audit interaction. This EWO authorizes the smallest change to `synapse-actor-host`'s public construction surface necessary to make its real behaviour invocable by the Runtime, and nothing beyond that minimum — no generic factory, no additional public type beyond what is strictly needed, no extension point, no premature abstraction for milestones this EWO does not cover. Implementation chooses the concrete form; this EWO does not prescribe it.
- **Minimum authorized Runtime-composition-root change.** The Runtime may construct Actor Host via its (possibly adjusted) public surface, invoke `define`/`create_instance`/`terminate_instance` where this milestone's own tests require exercising them, and, on success, construct and cause the corresponding audit event to be emitted via its existing Audit Emitter instance. This is the entire extent of the authorized Runtime change. It does not authorize the Runtime to orchestrate actor lifecycle generally, to become a caller of Actor Host for any purpose beyond satisfying and testing this audit obligation, or to acquire any role Out of Scope above excludes.

If more than one architecturally plausible way to satisfy this section's requirements would require deciding something this section does not already resolve, that is the stop condition (Definition of Failure, below) — but the coupling model itself (Runtime-mediated, no direct Actor Host dependency) is not open for reconsideration.

## Audit Failure Semantics

Resolved by ADR-0015, now integrated into ARCH-002 §11 (step 17) and §22. This section states the resulting requirement; it does not reopen the question.

- `create_instance` and `terminate_instance` each carry a mandatory audit obligation under ARCH-002 §18. Neither may be reported successful to its own caller unless the corresponding required audit event was successfully caused to be emitted.
- If required audit emission fails, the triggering operation (`create_instance` or `terminate_instance`) fails. This is the operation's own outcome, observable through its existing `Result<_, RuntimeError>` return type — no new error-reporting channel is introduced.
- Audit-emission failure does not cause Runtime-wide failure. It does not transition `RuntimeState`, and it does not affect any other actor's or operation's outcome.
- Downstream Audit Pipeline failure remains governed exactly as ARCH-002 §11 step 17 and §16 already state — it must not block or fail the triggering operation. This is a distinct failure domain from Audit Emitter's own emission failing, per the now-integrated ARCH-002 §11.
- No rollback, transaction model, persistence guarantee, or internal mutation ordering is required or authorized by this rule. Whether Actor Host's internal existence/liveness records are updated before, after, or as part of a failed audit emission is implementation freedom; the only required, testable outcome is that the operation as a whole reports failure when its mandatory audit emission fails.

---

## Definition of Done

The task is complete only if all of the following are true:

- `define` assigns a unique `ActorId` per call, meeting "Meaning of 'Stable' ActorId" above.
- `create_instance` succeeds only for a previously defined `ActorId`, per "Define-Before-Create — Authoritative Basis," and fails with `RuntimeError` otherwise.
- `create_instance` assigns a unique `ActorInstanceId` distinct from every other instance, including other instances of the same defined actor.
- `terminate_instance` succeeds exactly once per instance and fails with `RuntimeError` on an unknown or already-terminated instance.
- No public accessor exposes an instance's private state.
- Actor creation and actor termination each result in the required audit event (ARCH-002 §18) being caused to reach Audit Emitter, via the Runtime-mediated interaction "Audit Interaction" defines — no direct Actor Host dependency on Audit Emitter exists.
- `create_instance` and `terminate_instance` are not reported successful unless their required audit emission succeeded, per "Audit Failure Semantics."
- EWO-001's existing bootstrap/shutdown tests continue to pass unmodified in behaviour.
- All tests pass.
- No warnings.
- No unsafe.
- No external dependencies.
- No new dependency of any kind — `synapse-actor-host`'s dependency set remains exactly `synapse-common`.
- The `synapse-actor-host` public construction surface reflects only the minimum change "Audit Interaction" authorizes.
- Trusted Core boundary unchanged (still exactly seven components).
- Architecture unchanged.

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

All must pass. Additionally verify: workspace member count unchanged (14); zero `unsafe`; `synapse-common` unchanged in behaviour; no dependency cycle; no replaceable-service crate depended on by `synapse-actor-host`; no crate depends back on `synapse-actor-host` other than the Runtime composition root; `cargo tree -p synapse-actor-host` shows exactly `synapse-common`.

---

## Data and State Constraints

Required state, per ARCH-002 §7 and §11 (no more, no less):

- A record of which logical `ActorId`s have been defined.
- A record of which `ActorInstanceId`s currently exist and are alive, and which defined `ActorId` each belongs to.
- A private, per-instance state slot, opaque at this milestone (no actor-defined payload exists yet to store).
- A record of which instances have been terminated, sufficient to reject a second termination.

These are existence/liveness records only — see "Existence Records vs. Lifecycle State Ownership" above; they are not an `ActorState` implementation. The exact data structures (map, set, or other representation) are implementation freedom, not prescribed here. Do not add: message queues or mailboxes; capability registries; scheduling queues; lifecycle-policy state beyond the defined/exists/terminated boundary; synchronization primitives beyond what `&mut self` already provides (concurrent access is out of scope — ARCH-002 §12 governs execution, not construction/teardown bookkeeping).

---

## Failure and Error Behaviour

Per ARCH-002 §16 (Failure Model) and the now-integrated §11 step 17:

- "Actor initialization failure" → the actor instance fails to become Ready; isolated, audited. At this milestone, this corresponds to `create_instance` returning `Err`; no instance is left partially constructed.
- Creating an instance for an undefined `ActorId` is a rejected operation (`Err(RuntimeError)`), not a panic or a partially-constructed instance.
- Terminating an unknown or already-terminated instance is a rejected operation (`Err(RuntimeError)`), not a panic.
- Audit-emission failure causes the triggering operation to fail, per "Audit Failure Semantics" above. It does not cause Runtime-wide failure and does not affect any other operation.
- No distinct "component failure" or "Runtime failure" tier applies to Actor Host at this milestone — nothing in this scope can compromise trusted-core integrity beyond a single rejected operation, since no shared mutable state, execution, or capability logic exists here yet.

---

## Definition of Failure

Stop immediately if:

- a constitutional contradiction is discovered;
- ARCH-002 cannot be implemented as written;
- Trusted Core expansion becomes necessary;
- a new Runtime abstraction appears necessary;
- an architectural decision is required;
- a new interaction not already named by ARCH-002 §11 or §20 appears necessary;
- ownership between Actor Host and another Trusted Core component becomes ambiguous;
- actor identity semantics prove insufficiently defined by ARCH-002 §7 for the operation being implemented, beyond the bounds "Identity Generation Constraints" and "Meaning of 'Stable' ActorId" already set;
- a state transition beyond defined/exists/terminated must be invented;
- a failure or recovery policy beyond §16's existing entries, and beyond what "Audit Failure Semantics" and "Failure and Error Behaviour" above already resolve, must be invented;
- audit semantics beyond the two events §18 lists must be invented;
- the construction-surface or Runtime-composition-root change objectively required exceeds the minimum "Audit Interaction" authorizes;
- a replaceable service appears necessary but is not architecturally assigned;
- an external dependency appears necessary;
- the existing Runtime lifecycle (EWO-001) would need reinterpretation;
- satisfying this milestone would require work from a later SRP milestone (message handling, capability validation, execution, scheduling, persistence, networking).

Do not resolve these issues yourself. Report them.

---

## Implementation Sequence

1. Baseline verification: re-confirm the current `core/actor-host` and `runtime/` state matches this EWO's "Existing Baseline" section exactly.
2. Component-local implementation: implement `ActorHostImpl`'s internal state and the `ActorHost` trait for it, per Scope, "Identity Generation Constraints," "Meaning of 'Stable' ActorId," and "Existence Records vs. Lifecycle State Ownership."
3. Component-local tests: unit tests for each Definition-of-Done item not concerning the audit interaction, run within the `synapse-actor-host` crate.
4. Construction-surface change: apply the minimum change "Audit Interaction" authorizes, and no more.
5. Runtime integration: apply the minimum Runtime-composition-root change "Audit Interaction" authorizes — constructing Actor Host, invoking its operations, and causing the resulting audit events to reach Audit Emitter.
6. Integration test: confirm actor creation and termination each result in the required audit event reaching Audit Emitter, and that a failed required emission fails the triggering operation, per "Audit Failure Semantics."
7. Regression check: confirm EWO-001's existing Runtime bootstrap/shutdown tests still pass unmodified.
8. Documentation updates: update `core/actor-host/README.md` and `runtime/README.md` to reflect the real behaviour and the minimum integration now present.
9. Complete quality validation, per Mandatory Validation above.
10. ER-002 preparation after implementation and validation are complete — not before, and not as part of this EWO.

Do not include work from later milestones.

---

## Acceptance Criteria

Every criterion is objective and testable:

- `define` called twice with different names returns two different `ActorId`s (uniqueness, direct test).
- `create_instance` on an undefined `ActorId` returns `Err` (direct test).
- `create_instance` called twice on the same defined `ActorId` returns two different `ActorInstanceId`s (direct test).
- `terminate_instance` on an instance created in the same test returns `Ok` exactly once; a second call on the same instance returns `Err` (direct test).
- `terminate_instance` on a never-created `ActorInstanceId` returns `Err` (direct test).
- No `pub` item in the crate exposes per-instance state directly (verified by code review).
- `cargo tree -p synapse-actor-host` shows exactly `synapse-common` — no other dependency, unconditionally.
- Workspace member count is unchanged at 14 (`cargo metadata`).
- `cargo test --workspace` shows all of EWO-001's existing tests still passing, unmodified in outcome.
- An integration test demonstrates that a successful `create_instance` and a successful `terminate_instance`, invoked through the Runtime, each result in the corresponding audit event reaching Audit Emitter.
- A test demonstrates that if required audit emission fails, the triggering operation reports failure, and no other operation or the Runtime's own state is affected.
- Every item in Out of Scope has zero corresponding code.
- All Mandatory Validation gates pass with zero warnings.
- `git status` after implementation shows changes confined to `core/actor-host/`, the minimum touched parts of `runtime/`, and their manifests/READMEs — no unrelated file touched.

---

## Required Tests

- Unit tests (within `synapse-actor-host`): identity uniqueness for `define`; instance uniqueness for `create_instance`; define-before-create invariant; terminate success and double-terminate rejection; terminate-unknown rejection; no public state-access path (a structural/compile-time check).
- Boundary tests: none invented for a `define` failure mode, since ARCH-002 does not define one at this milestone.
- Invalid-operation tests: covered by the `Err` cases above.
- Audit-interaction integration tests (within `runtime/tests/`): creation and termination each cause the required audit event to reach Audit Emitter; a failed required emission fails the triggering operation without affecting Runtime state or any other operation.
- Regression tests for EWO-001: the existing `runtime` crate unit and integration tests must continue to pass unmodified in outcome.
- No test is required for any Out-of-Scope behaviour, and no test is required to prove suspension/resumption/restart stability, which this milestone does not implement (see "Meaning of 'Stable' ActorId").

---

## Engineering Decision Log

Record:

- implementation decisions;
- repository decisions;
- deferred decisions;
- architectural decisions (expected: None);
- constitutional decisions (expected: None);
- future work enabled;
- future work deferred.

---

## Completion Report

ER-002 must provide, after implementation:

1. Files modified.
2. Files created.
3. Actor Host behaviour implemented.
4. Invariants enforced (define-before-create, single termination, state privacy, audit-failure-fails-operation).
5. Tests added.
6. Validation results.
7. Dependency changes (expected: none).
8. Trusted Core changes (expected: none — same seven components, same boundaries).
9. Architecture changes (expected: none).
10. Construction-surface and Runtime-integration changes made, exactly as authorized by "Audit Interaction."
11. Engineering Decision Log.
12. Any issues requiring architectural review.

Stop after Actor Host. Do not begin the next engineering milestone.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-12 | Denver Jacobs | Initial EWO, registered per STD-001 §46. Derived exclusively from ARCH-001, ARCH-002, STD-001, STD-002, STD-004, STD-011, ADR-0013, ADR-0014, and the verified current state of `core/actor-host` and `runtime/`. |
| 0.2.0 | 2026-07-12 | Denver Jacobs | Returned for revision following Architecture Review. Removed the blanket authorization for a direct `synapse-actor-host → synapse-audit-emitter` dependency, converting the coupling-mechanism question into an explicit stop condition. Removed the prescribed `Box<dyn ActorHost>` replacement; retained the existing opaque construction mechanism unless implementation objectively proved it insufficient. Added "Identity Generation Constraints," "Meaning of 'Stable' ActorId," "Define-Before-Create — Authoritative Basis," and "Existence Records vs. Lifecycle State Ownership." |
| 0.3.0 | 2026-07-12 | Denver Jacobs | Revised to integrate ADR-0015 and ADR-0016, both approved and integrated into ARCH-002 at commit `2629241`. Replaced the "Audit Obligations" stop-condition section with two definite sections: "Audit Interaction" (Runtime-mediated coordination per ADR-0016 — Actor Host owns event content via its own return values, Runtime owns causing emission, no direct dependency is authorized or ever will be, and the previously-opaque `ActorHostHandle` is now objectively insufficient, so this EWO authorizes the minimum necessary construction-surface change and a correspondingly minimum Runtime-composition-root change, both without prescribing engineering form) and "Audit Failure Semantics" (per ADR-0015 — mandatory-audit operations fail when their emission fails, no Runtime-wide failure, no rollback/transaction/persistence/ordering prescribed, Audit Pipeline failure remains a distinct, already-governed failure domain). Updated Out of Scope, Architecture Constraints, Definition of Done, Failure and Error Behaviour, Definition of Failure, Implementation Sequence, Acceptance Criteria, and Required Tests throughout to remove every reference to the two now-resolved ambiguities and to add the new, definite audit-interaction and audit-failure requirements and their tests. The milestone objective, Actor Host's own responsibilities, identity constraints, define-before-create basis, and existence-records/lifecycle-state distinction are unchanged from 0.2.0. |

## Disposition

Not yet dispositioned.
