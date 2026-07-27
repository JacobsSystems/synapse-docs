---
document_id: EMR-002
title: Persistent Actor Durable-Deletion Timer Cleanup
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-27
last_updated: 2026-07-27
classification: Public
related_documents:
  standards:
    - STD-001 (§48, §49 — Engineering Maintenance Orders/Reports; this report's own governing sections)
  architecture:
    - ARCH-005 (v0.2.0 — Temporal Runtime Architecture; §23, the corrected requirement this maintenance restores implementation conformance to)
    - ARCH-007 (v0.4.0 — Persistent Actor Architecture; §17, the corrected requirement this maintenance restores implementation conformance to)
  work_orders:
    - EWO-008 (Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution; governs the `synapse-timer`/`Timer::cancel_all_for_actor` mechanism this maintenance calls into — see §5, the disclosed traceability asymmetry)
  reports:
    - ER-008 (Temporal Runtime — Engineering Report; the closest related ER; predates ARCH-007 and does not itself cover deletion or persistence)
  maintenance:
    - EMO-001 (maintenance/EMO-001-ActorHost-Single-Live-Instance.md; unrelated prior EMO, cited only to explain this document's own EMR-002 numbering — see §2)
  implements: "Engineering Modification Order — Persistent Actor Durable Deletion Timer Cleanup" (supplied and executed as this correction's own authorizing instruction within this engineering effort; not committed to this repository as a separate file — see §3)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# EMR-002 — Persistent Actor Durable-Deletion Timer Cleanup

Registered per STD-001 §49 (Engineering Maintenance Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or retroactively expand the scope of the EMO it reports against. It records the maintenance work exactly as independently verified; it does not redesign the implementation or revisit the architecture. Nothing described here has been committed, staged, or pushed.

## 1. Purpose

This report closes the completed maintenance correction restoring `Runtime::delete_actor_state` to conformance with ARCH-005 v0.2.0 §23 and ARCH-007 v0.4.0 §17: durable deletion of a Persistent Actor's domain state now proactively cancels every pending Temporal Runtime timer registration for the same `ActorId`, in the architecturally required order, before durable deletion is attempted.

## 2. Scope and Document Numbering

This report covers only the narrow maintenance scope authorized by the governing EMO (§3): timer cleanup inside `delete_actor_state`. It does not cover, and explicitly excludes, timer re-arming, durable timer persistence, timer quotas, logical timer keys, workflow orchestration, or any change to `terminate_actor_instance`.

**Numbering.** The `maintenance/` directory's only existing document at the time of this report is `EMO-001-ActorHost-Single-Live-Instance.md` (Draft, unrelated ActorHost work from 2026-07-12), whose own frontmatter already forward-references `EMR-001 (maintenance/EMR-001-ActorHost-Single-Live-Instance.md, not yet created)`. No `EMR-NNN` file exists anywhere in the repository. Rather than claim the `EMR-001` identifier — which EMO-001 has already, explicitly, on the record, reserved for its own eventual completion report on unrelated work — this report is numbered **EMR-002**, the next identifier that does not collide with an existing forward-reference. STD-001 §7 confirms EMO/EMR numbering is independent of any EWO/ER/milestone label; this reasoning is disclosed here because STD-001's own §7 registry table (line ~798) is itself stale, still stating "EMO-NNN (none issued yet)" despite EMO-001 already existing on disk — a pre-existing repository inconsistency, noted for completeness, not corrected here (STD-001 is explicitly out of scope for this maintenance task).

## 3. Governing EMO

The authorizing instrument — "Engineering Modification Order — Persistent Actor Durable Deletion Timer Cleanup" — was supplied and executed as this correction's own authorizing instruction within this engineering effort's demonstrated lifecycle (Integration Architecture Review → Targeted Architecture Correction → Narrow Architecture Re-Review → EMO → Implementation → Independent Implementation Review → this closure report). Consistent with how the preceding Integration Architecture Review and Narrow Architecture Re-Review in this same correction chain were treated, the EMO was not itself committed to `synapse-docs` as a separate file. This report cites it by its full scope and content, verified directly against the resulting implementation (§6–§11), rather than by a file path that would not exist. This is a process observation, not a defect this report has authority to correct.

## 4. Repository Baseline

`synapse-runtime`: branch `main`, HEAD `0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7`, unchanged throughout this entire maintenance effort. `synapse-docs`: branch `main`, HEAD `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6`, 2 ahead / 0 behind `origin/main`. `git fetch origin` fails with `Permission denied (publickey)` in this environment; verification throughout this effort has proceeded from local state only. Nothing staged in either repository at any point in this maintenance effort.

## 5. Traceability Disclosure — the EWO/ER Asymmetry

STD-001 §48/§49 require every EMO/EMR to identify "the EWO it maintains" and "the ER it relates to, if one exists." This maintenance work does not fit that pattern cleanly, and this report discloses the mismatch rather than forcing a misleading single citation: `Runtime::delete_actor_state` — the method this maintenance modifies — was itself introduced during this engineering effort's Persistent Actors implementation, which had **no EWO** (an explicitly disclosed deviation at the time). The mechanism the new code calls into, `Timer::cancel_all_for_actor`, traces to **EWO-008** (Temporal Runtime). No Persistent Actor Engineering Report exists to relate to; the closest ER is **ER-008**, which predates ARCH-007 and does not itself address deletion or persistence. This maintenance therefore restores conformance to corrected **architecture** (ARCH-005 §23, ARCH-007 §17) spanning a boundary between an EWO-covered component and a non-EWO-covered one — a case STD-001 §48's own text did not precisely anticipate. No architectural deficiency was found requiring ADR-process escalation; this is a traceability-citation gap only, disclosed per this report's own governing "truthful engineering history" priority.

## 6. Implementation Summary

Exactly six lines (plus a doc comment) were added inside `Runtime::delete_actor_state` (`runtime/src/lib.rs`), between the existing `persistence_deletion_authorized_event` emission and the existing `self.persistence.delete(actor)` call:

```rust
for cancelled in self.timer.cancel_all_for_actor(actor) {
    let _ = cancelled;
    self.core.audit_emitter.emit(timer_cancelled_event(actor))?;
}
```

This reuses, unmodified, the identical pattern already established in `Runtime::execute_stop_decision` — no new timer-cleanup mechanism, no new `Timer` or `Persistence` trait method, no new audit-event constructor, and no public method signature change. `delete_actor_state`'s signature — `pub fn delete_actor_state(&mut self, actor: &ActorId, authorization: &Capability) -> Result<(), RuntimeError>` — is byte-identical to before this maintenance.

## 7. Authorization Ordering

Confirmed unchanged and preserved: `RuntimeState::Running` validation → `deletion_requested` audit → authorization check → on denial, `deletion_denied` audit and immediate return (before the cancellation loop can be reached) → on success, `deletion_authorized` audit → **only then** the new cancellation loop → `persistence.delete` → outcome audit. An unauthorized deletion attempt cannot cancel any timer; this is structural (the early `return` is positioned before the loop in source), not merely test-observed.

## 8. Timer-Cleanup Behaviour

Cancellation uses the `ActorId` `delete_actor_state` already received — no reverse `ActorInstanceId`-to-`ActorId` lookup was added or is required, since this call site (unlike `terminate_actor_instance`) already carries `ActorId` directly. Every currently `Pending` registration belonging to the deleted `ActorId` is cancelled; registrations belonging to any other `ActorId` are structurally untouched (`Timer::cancel_all_for_actor`'s own `&node.actor == actor` comparison).

## 9. Audit Ordering

Confirmed by source and by dedicated tests: `timer.cancelled` (one per cancelled registration) is emitted strictly after `persistence.deletion_authorized` and strictly before `persistence.deletion_completed` or `persistence.deletion_failed`. When no timer is pending, no `timer.cancelled` event is emitted, and the pre-existing three-event deletion sequence is unchanged.

## 10. Persistence-Success Semantics

Deletion completion is reported only after `Persistence::delete` genuinely returns `Ok`, and only after cancellation and its audit have already, genuinely completed. No completion is ever claimed early.

## 11. Persistence-Failure Semantics

When durable-state deletion subsequently fails, the durable state remains, the already-cancelled timers remain cancelled (not rolled back), the failure is reported truthfully via `persistence.deletion_failed`, and no `persistence.deletion_completed` event exists in that path. No timer is recreated and no compensation of any kind is attempted — this correction does not claim, and does not implement, transactional atomicity across Temporal Runtime and Persistence Service; it implements a truthful, ordered, independently-auditable sequence of steps.

## 12. Audit-Infrastructure-Failure Disclosure

The Independent Implementation Review traced `Timer::cancel_all_for_actor`'s actual implementation and established the following, precisely: **all matching timers are mutated to `Cancelled` synchronously, inside that single call, before it returns** — the subsequent audit loop only reports cancellations that have already, unconditionally, occurred. Consequently, if the audit emitter itself fails partway through the loop: the deletion operation returns an error; durable-state deletion is never attempted; every matching timer remains genuinely cancelled; only the cancellation audit events emitted before the failure exist; and — because `cancel_all_for_actor` never again returns an already-`Cancelled` registration — **a later retry cannot regenerate the missing cancellation events**. This is consistent with, not a violation of, ADR-0015's already-established policy that a mandatory audit emission's failure causes the reporting operation to fail without rolling back already-committed component-level state. The identical structural characteristic already exists, unmodified, in the pre-existing `execute_stop_decision`, and is not newly introduced by this maintenance. Audit completeness is **not** guaranteed during audit-infrastructure failure — this report does not claim otherwise, in either direction. The Independent Implementation Review classified this as an **OBSERVATION**, and separately identified, as a **MINOR** finding, that no dedicated regression test currently exercises this scenario or the related retry scenarios. Redesigning audit durability, compensation, or event granularity to close this gap is explicitly outside this maintenance's authorized scope.

## 13. Tests Added

Nine tests, added to `runtime/src/lib.rs`'s existing test module, none modifying any pre-existing test:

1. `delete_actor_state_cancels_a_pending_timer_for_the_same_actor`
2. `delete_actor_state_cancels_every_pending_timer_for_the_actor`
3. `delete_actor_state_does_not_touch_a_different_actors_timers`
4. `delete_actor_state_denial_does_not_cancel_timers`
5. `delete_actor_state_succeeds_and_emits_no_cancellation_event_when_no_timers_are_pending`
6. `delete_actor_state_failure_still_leaves_pending_timers_cancelled`
7. `delete_actor_state_audit_events_are_emitted_in_truthful_order_with_a_pending_timer`
8. `checkpointing_does_not_cancel_pending_timers`
9. `restoring_an_actor_does_not_automatically_cancel_or_rearm_timers`

Not covered by any dedicated test, disclosed rather than overstated: audit-emitter failure partway through the new cancellation loop; retry behaviour following persistence-deletion failure; retry behaviour following a cancellation-audit failure. Per the Independent Implementation Review, these are approved, non-blocking coverage observations.

## 14. Fresh Validation Results

Independently re-run for this closure report:

| Command | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | **569 passed, 0 failed, 0 ignored** |
| Narrow filtered run (`delete_actor_state`, `checkpointing_does_not_cancel`, `restoring_an_actor_does_not_automatically`) | 13 matched, all passed |

569 is reproduced independently, not assumed from any prior report, and matches every prior report in this correction chain exactly.

## 15. Independent Implementation Review Outcome

The Independent Implementation Review (conducted immediately prior to this closure report, within this same engineering effort) concluded:

**IMPLEMENTATION APPROVED** — "The Persistent Actor durable-deletion timer-cleanup implementation is ready for milestone closure." No Blocker or Major finding was recorded. Two findings were recorded and are carried forward into §12–§13 above: an OBSERVATION on audit-infrastructure-failure behaviour, and a MINOR finding on missing regression-test coverage for that scenario and its retry variants.

## 16. API and Component-Boundary Verification

Confirmed unchanged throughout this maintenance: no public Runtime method signature changed; `services/timer/`, `services/persistence/` (beyond its pre-existing, unrelated state from earlier in this engineering effort), Actor Host, Lifecycle Guardian, Capability Authority, and Audit Emitter crates were not modified; no new crate dependency was introduced; Temporal Runtime and Persistence Service remain mutually unaware of one another — neither gained any dependency on, or call path to, the other.

## 17. Deferred Issues

Preserved, exactly as scoped by every document in this correction chain:

- **Direct termination cleanup** — `terminate_actor_instance` still lacks direct `ActorId` context for proactive timer cleanup; any signature or caller-contract change remains separately authorized future work.
- **Timer re-arming** — Persistent Actor restoration still has no approved coordination mechanism for reconstructing actor-owned temporal intent after a complete Runtime-process restart.
- **Duplicate prevention** — no logical timer key, reconciliation mechanism, or idempotency safeguard exists for any future re-arming work.
- **Timer quotas** — no per-actor or global timer registration limit is introduced by this correction.
- **Audit-failure regression coverage** — dedicated tests for audit-emitter failure and retry behaviour (§12–§13) remain a future coverage improvement.

None of these issues blocks this milestone's closure.

## 18. Known Non-Blocking Observations

1. The per-timer audit-emission-in-a-loop pattern, inherited unmodified from `execute_stop_decision`, means audit completeness for timer cancellation is not guaranteed under a genuine audit-infrastructure failure, and any such gap is not recoverable by retry (§12). Consistent with existing ADR-0015 policy; not newly introduced.
2. STD-001 §7's own registry table is stale relative to the actual repository contents (`EMO-001` already exists; the table still says "none issued yet") — noted for completeness, outside this report's authority to correct.
3. This maintenance's own EWO/ER traceability is asymmetric across the component boundary it touches (§5) — disclosed, not resolved, since resolving it would require either authoring a retroactive EWO for the EWO-less Persistent Actors implementation or amending STD-001's own traceability requirement, neither of which is within this report's scope.

## 19. Final Completion Status

Scope completed exactly as authorized, with no deviation from the governing EMO's own scope (§3). All required behaviour is implemented, all nine required tests pass, full workspace validation is clean, and the Independent Implementation Review's verdict is recorded accurately (§15). No unauthorized file changed. No pre-existing work was disturbed. Nothing is staged or committed.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-27 | Claude (AI-assisted) | Initial report closing the Persistent Actor durable-deletion timer-cleanup maintenance correction, following its Independent Implementation Review (verdict: IMPLEMENTATION APPROVED). Records the implementation, its authorization/cancellation/audit ordering, its persistence-success and persistence-failure semantics, a full disclosure of audit-infrastructure-failure behaviour under the existing, unmodified `execute_stop_decision`-derived pattern, the nine tests added, independently reproduced validation (569 passed, 0 failed), and every deferred issue and non-blocking observation carried forward from the review chain. Discloses this report's own EMR-002 numbering (reserving EMR-001 for EMO-001's own already-forward-referenced completion report) and the EWO/ER traceability asymmetry inherent to this maintenance's cross-component scope. |

## Disposition

Draft. Not yet reviewed for publication. Records completed, independently reviewed maintenance work. Does not itself authorize, approve, publish, commit, or push anything, consistent with STD-001 §49's informational-only status for Engineering Maintenance Reports.
