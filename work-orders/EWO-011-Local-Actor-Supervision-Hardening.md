---
document_id: EWO-011
title: "Runtime Integration: Local Actor Supervision Hardening"
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-20
last_updated: 2026-07-20
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.1 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.5.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
    - ARCH-004 (v0.1.0 — architecture/ARCH-004-Local-Actor-Supervision-Architecture.md) — the sole architectural authority this EWO hardens the realization of; not amended by this EWO
    - ARCH-006 (v0.1.4 — architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md)
  adrs:
    - ADR-0015
    - ADR-0016
    - ADR-0017
  predecessor: EWO-007 (work-orders/EWO-007-Local-Actor-Supervision.md) — the milestone whose implementation this EWO hardens; EWO-008 (Temporal Runtime) and EWO-009 (Runtime Actor Execution) both integrate with EWO-007's own supervision routing and are left unmodified
  reported_by: ER-010 (engineering-reports/ER-010-Local-Actor-Supervision-Hardening.md) — the next sequentially available Engineering Report identifier per STD-001 §7 (independently, sequentially numbered families); not "ER-011," which would incorrectly assume ER and EWO numbering stay paired — EWO-010 (documentation-only) required no ER at all, so the two sequences have already diverged by one
  base_state:
    runtime_head: 0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7
    docs_head: c4be0432ebc334f668932d0fb780c37352918035
---

# EWO-011 — Runtime Integration: Local Actor Supervision Hardening

Registered per STD-001 §46 (Engineering Work Orders — "reconciliation" and ordinary bounded engineering work are both authorized purposes; this EWO is the latter: hardening, not reconciliation). This is the second Engineering Work Order touching local actor supervision, following an independent architecture review (conducted immediately prior to this EWO, within the same engineering effort) that confirmed ARCH-004 is already fully, correctly implemented by EWO-007, independently re-verified against source and against a full workspace test run (517 tests, 0 failures, at this EWO's own base state) — and that no architectural redesign is warranted. This EWO authorizes implementation work only. It does not itself constitute approval.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-011 |
| Title | Runtime Integration: Local Actor Supervision Hardening |
| Version | 0.1.0 |
| Status | **Draft** — not yet approved |
| Author | Denver Jacobs |
| Owner | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-20 |
| Last Updated | 2026-07-20 |
| Classification | Public |
| Applicable repository | `synapse-runtime` |
| Target branch | `main` |
| Runtime base HEAD | `0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7` |
| Documentation base HEAD | `c4be0432ebc334f668932d0fb780c37352918035` |
| Predecessor milestone | EWO-007 — Local Actor Supervision |
| Reported by | ER-010 (engineering-reports/ER-010-Local-Actor-Supervision-Hardening.md) |

This document is authored alongside, not strictly before, the implementation it authorizes: the engineering work it describes was carried out within the same continuous task as this filing, per this Founder-directed engineering effort's own established practice for narrowly bounded, immediately-reported milestones. Its own `created`/`last_updated` dates reflect that. It authorizes exactly the scope below; it does not authorize, and this task does not perform, any work beyond it.

---

## Engineering Authority

Governed by, in descending order:

1. ARCH-001 — Constitutional Architecture — unaffected; every action this EWO authorizes remains subject to all four constitutional guarantees.
2. ARCH-002 — Runtime Architecture, **v0.2.1** — `RuntimeState` and the Runtime shutdown sequence (§15, EWO-001 "Runtime Requirements" steps 5–7), unaffected in their own text; this EWO adds no new `RuntimeState` value and no new transition.
3. ARCH-003 — Runtime Integration Architecture, **v0.5.0** — the current, verified integration baseline; unaffected.
4. **ARCH-004 — Local Actor Supervision Architecture, v0.1.0 — the sole architectural authority for supervision itself.** This EWO implements no new architectural decision beyond what ARCH-004 already establishes; every change below is either (a) already implied by ARCH-004's own text (§18's truthfulness requirement — "no lifecycle state may be held or represented as other than what it currently, genuinely is" — extended here to the Runtime's own shutdown state, not merely an actor's) or (b) a bounded, disclosed implementation-policy choice of the same kind EWO-007's own "Bounded Design Decisions" already precedent (§4, §16 of ARCH-004 explicitly reserve exactly this class of decision to implementation).
5. ARCH-006 — Runtime Actor Execution Architecture, v0.1.4 — unaffected; the dispatch-`Err` call site this EWO's own guard sits immediately downstream of is ARCH-006's own realized admission pipeline, untouched by this EWO.
6. ADR-0015 — Audit Emitter Failure Semantics (Approved) — the authority this EWO's entire "Objective 2" investigation is measured against: every restart-failure path this EWO reviewed was found already consistent with ADR-0015's own established rule (a mandatory audit-emission failure fails the reporting operation without rolling back already-committed component-level state), and none required correction.
7. ADR-0016 — Trusted Core Interaction Rule (Approved) — unaffected; this EWO adds no new cross-component call edge.
8. STD-001 — Documentation Standards (§46, Engineering Work Orders; §47, Engineering Reports).

This EWO amends none of the above. Where it makes an implementation-level determination the independent architecture review itself identified as safely resolvable "during EWO drafting" rather than requiring separate governance work, that determination is bounded and disclosed below ("Bounded Design Decisions"), exactly as EWO-007's own precedent established for `RESTART_ALLOWANCE`.

---

## Purpose

Harden EWO-007's already-implemented, already-independently-reviewed local actor supervision realization against two specific, evidence-identified gaps — the one genuine open question the immediately preceding architecture review surfaced (shutdown-aware restart suppression, not previously guarded), and comprehensive regression coverage for restart's own failure paths (already believed safe by design, but not yet regression-tested) — without redesigning supervision, its topology, its identity model, or its lifecycle ownership in any way.

---

## Problem Statement

Verified directly against `synapse-runtime` @ `0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7` (this EWO's own base HEAD; the working tree also carries EWO-007/008/009's own realized supervision, timer, and actor-execution code, all committed at this same HEAD via the repository's own consolidated "publish reconstructed Act 1 foundation" commit) — not assumed from the architecture review's own report alone:

- **`Runtime::shutdown` consumes `self` by value** (`runtime/src/lib.rs`, doc comment: "a terminated Runtime cannot be used again, enforced at compile time rather than by a runtime check alone"). Through the public API as it exists today, no caller-visible `Runtime` value can ever have a non-`Running` state and still be callable — this is a genuine, structural, compile-time guarantee, not an assumption.
- **`Runtime::step` already carries a defensive `RuntimeState::Running` guard** for exactly this "structurally unreachable through ordinary sequential use, guarded anyway" reason, with its own doc comment stating plainly: "No other Runtime method in this crate currently gates on `RuntimeState` (only `bootstrap`, `shutdown`, and the private `transition` touch it)." Independently re-confirmed by direct source inspection: `route_actor_execution_failure` (EWO-007) — the sole entry point through which a new actor incarnation can be created outside ordinary `create_actor_instance*` calls — carried no such guard before this EWO. This is the one genuine gap the immediately preceding architecture review identified (its own §9, §12): not a reachable defect today, but an implicit, ownership-based guarantee rather than an explicit, testable, invariant-encoding one, and the sole place besides `step` where a defensive guard of this kind is architecturally warranted.
- **Every restart-execution failure path was traced directly against source** (`execute_restart_decision`, `execute_stop_decision`, `runtime/src/lib.rs`) for this EWO's own "Objective 2" investigation. Each of the four sequential mandatory audit emissions inside `execute_restart_decision` (`supervision.restart_initiated`, `actor.terminated`, `actor.created`, `supervision.restart_completed`) was independently confirmed to fail safely, per ADR-0015's own established, corpus-wide rule, with no partial or corrupting Actor Host, Scheduler, Capability Authority, or Lifecycle Guardian state possible on any failure branch. **No code defect was found in any restart-failure path.** ER-007 §11 had already disclosed, and this EWO's own investigation independently reconfirmed, that the specific case of `terminate_instance` succeeding immediately followed by `create_instance_with_behavior` itself failing (as opposed to a subsequent audit-emission failure) is not exercisable through any test double currently available to `runtime`'s own test module — Actor Host is a concrete, non-swappable field (`ActorHostHandle`, not a trait object slot), unlike `AuditEmitter` (`Box<dyn AuditEmitter>`) — and constructing one would require either a Trusted Core modification or a Runtime-internal structural change, both outside this EWO's own bounded scope ("Explicit Exclusions," below).
- This EWO's own predecessor gap (regression coverage for the above) was, until this EWO, genuinely absent: no test in the current workspace exercised `route_actor_execution_failure` against a non-`Running` `RuntimeState`, nor against a targeted audit-emission failure at each of the four restart-sequence event types individually.

None of the above requires correction beyond what this EWO itself authorizes.

---

## Architectural Authority

| Concern | Owner | Authority | Touched by this EWO? |
|---|---|---|---|
| Whether a new actor incarnation may begin during/after shutdown | Runtime (the sole place this determination can be made — ADR-0016 Rule 1) | ARCH-004 §18 (truthfulness), ARCH-002 §15 (`RuntimeState`) | **Yes** — one new guard clause |
| Supervision policy, restart accounting, decision rule | Supervisor | ARCH-004 §9, §10.1 | No |
| Instance/mailbox/behavior mechanics | Actor Host | ARCH-004 §10.2, §14 | No |
| Lifecycle-transition legality | Lifecycle Guardian | ARCH-004 §9.2 | No |
| Capability continuity | Capability Authority | ARCH-004 §17 | No |
| Ready-order, lifecycle-unaware | Scheduler | ARCH-004 §13, §19 | No |
| Mandatory audit-emission failure semantics | Audit Emitter / ADR-0015 (unchanged rule) | ADR-0015 | No — this EWO tests conformance to it, does not alter it |

STD-001 §46 governs this document's own form. This EWO's own Stop Conditions (below) apply STD-001 §46's escalation rule exactly as EWO-007's own did.

---

## Objective

Add one defensive `RuntimeState` guard to the single Runtime method through which a new actor incarnation can originate outside ordinary explicit creation calls, mirroring `Runtime::step`'s own already-established precedent exactly; independently trace every restart-execution failure branch against ADR-0015's own established rule; and add comprehensive, deterministic regression coverage for both. No new architecture, no new component, no new `RuntimeState` value, no new cross-component call edge, no numeric-policy change.

---

## Bounded Design Decisions

### 1. Guard placement — resolved, not open

The guard is placed as the first statement of `route_actor_execution_failure`, before even the pre-existing unconditional `Scheduler::remove` call — mirroring `Runtime::step`'s own placement ("before any scheduling or execution is attempted"). This is not a genuinely open choice: `route_actor_execution_failure` is the one and only entry point EWO-007 itself established for new-incarnation creation outside `create_actor_instance`/`create_actor_instance_with_behavior`, so no alternative placement was considered.

### 2. Scope of the guard — resolved, not open

The guard refuses the entire decision — Restart, Stop, and Escalate alike — not Restart specifically. A narrower guard admitting Stop/Escalate but refusing only Restart would create an asymmetry with no basis in ARCH-004 (which treats all three as outcomes of one `observe_failure` decision, §16) and would still permit Actor Host/Scheduler/Timer mutation after shutdown for two of the three outcomes — inconsistent with "shutdown never creates new actor incarnations" *and* the broader, already-established principle that nothing should act through Runtime once it is no longer `Running`.

### 3. Restart-failure code changes — resolved: none required

Per the Problem Statement's own investigation, every restart-execution failure path was found already correct under ADR-0015. No production code change beyond the shutdown guard (Decision 1–2) is authorized or performed by this EWO. This is itself a normal, expected EWO outcome, not a deviation — mirroring the precedent already set when an investigation finds existing behavior sound rather than defective.

### 4. Untestable failure branch — resolved: disclosed, not forced

The `create_instance_with_behavior`-fails-after-`terminate_instance`-succeeds branch remains code-reasoned-safe (traced directly against source; degrades honestly to a Stop-equivalent end state, per ARCH-004 §18) but not executable-test-covered, because Actor Host is not swappable in `runtime`'s own test harness. Making it swappable would be a Runtime-internal structural change and is explicitly excluded from this EWO's own bounded scope (below). This EWO instead provides equivalent coverage for the adjacent, genuinely testable case — an audit-emission failure occurring between successful termination and attempted replacement creation (`actor.terminated`'s own emission failing) — which exercises the identical "actor left with no live instance, honestly" outcome via a mechanism this EWO's own test infrastructure can actually drive.

---

## Scope

### Shutdown-aware restart suppression (Objective 1)

- One new `if self.state != RuntimeState::Running { return Err(RuntimeError::IllegalTransition); }` guard, as the first statement of `route_actor_execution_failure` (`runtime/src/lib.rs`).
- No new `RuntimeError` variant (reuses `IllegalTransition`, the same variant `Runtime::step`'s own guard already uses for an identical situation).
- No change to `Runtime::shutdown`, `RuntimeState`, or `validate_transition`.

### Restart failure hardening (Objective 2)

- No production code change (Bounded Design Decision 3).
- Direct source-level verification of every mandatory audit-emission failure branch inside `execute_restart_decision` and `execute_stop_decision` against ADR-0015.

### Regression testing (Objective 3)

- Nine new tests in `runtime/src/lib.rs`'s own `mod tests`, added under a new `// --- Local actor supervision hardening (EWO-011) ---` section immediately following the existing EWO-007 supervision tests: shutdown-refusal (Stopping and Stopped states), repeated-refusal statelessness, Scheduler non-interference on refusal, an explicit Running-state control confirming normal restart behavior is unaffected, no-duplicate-live-incarnation, and three audit-emission-failure-targeted tests (`restart_initiated`, `actor.terminated`, `restart_completed`) using a new, minimal, event-type-targeted test double (`FailFromEventType`) rather than a fragile call-count budget.

---

## Explicit Exclusions

Per the governing task's own instruction, none of the following is implemented, and no new capability toward any of them is introduced: distributed supervision; persistence; mailbox transfer; restart backoff, timing, or jitter; configurable restart limits; multiple supervisors; parent reassignment; dead-letter queues; service discovery; workflow recovery; clustering; making Actor Host swappable in the test harness (Bounded Design Decision 4); any change to `RESTART_ALLOWANCE`, supervision topology, restart identity semantics, or lifecycle ownership.

---

## Trusted Core

No `core/*` crate is modified. No `services/scheduler`, `services/supervisor`, or `services/timer` file is modified. This EWO's entire diff is confined to `runtime/src/lib.rs` (one guard clause, one doc-comment update, nine new tests, one new test-only struct).

---

## Component Boundaries and Prohibited Interactions

Unchanged from EWO-007 (ARCH-004 §10.2, §19; ADR-0016 Rule 2): Supervisor still reaches no other component directly; Runtime remains the sole composer; the new guard adds a check against Runtime's own already-owned `state` field and introduces no new call edge to or from any component.

---

## Definition of Done

- The shutdown-aware guard is present, is the first statement of `route_actor_execution_failure`, and is exercised by at least one direct test.
- `cargo fmt --all -- --check`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`, `cargo build --workspace`, and `cargo test --workspace` all pass with zero warnings and zero failures.
- No `core/*`, `services/scheduler`, `services/supervisor`, or `services/timer` file is modified.
- Both existing demonstrations (`cargo run --example worker_pool`, `cargo run --example actor_to_actor_messaging`) continue to run unchanged.
- ER-010 records the completed work, independently re-verified against source.

---

## Completion Report

Implementation complete. See ER-010 for full validation results, test counts, and independent verification.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-20 | Denver Jacobs | Initial Draft, authored alongside this EWO's own implementation. Authorizes the shutdown-aware restart-suppression guard (Objective 1), records that Objective 2's own investigation required no production code change (Bounded Design Decision 3), and authorizes the nine new regression tests (Objective 3). |

## Disposition

Not yet reviewed. Not yet approved.
