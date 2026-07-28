---
document_id: ER-014
title: "Effect Cancellation on Actor Termination — Engineering Report"
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-27
last_updated: 2026-07-27
classification: Public
related_documents:
  reports_on: "EWO-013 (work-orders/EWO-013-Effect-Cancellation-on-Actor-Termination.md, v0.2.0) — the governing Engineering Work Order this report records"
  architecture:
    - ARCH-008 (v0.3.2, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md)
    - ARCH-004 (Actor Lifecycle and Supervision Architecture; governs Restart/Stop mechanics this milestone integrates with, unmodified)
    - ARCH-007 (Persistent Actor Architecture; §17's deletion-coordination-ordering pattern is reused, not redesigned)
    - ARCH-005 (Temporal Runtime Architecture; §23's Stop-cancels-Restart-preserves precedent is the direct analogy this milestone follows for Effects)
  adrs:
    - ADR-0015 (Approved — Audit Emitter Failure Semantics)
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  standards:
    - STD-001
  predecessor: ER-013 (Effect Timeout Integration — Engineering Report)
---

# ER-014 — Effect Cancellation on Actor Termination — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. It records the implementation exactly as independently verified, including the correction cycle this milestone required. Nothing described here has been staged, committed, or pushed.

**Numbering.** The highest existing Engineering Report on record is `ER-013-Effect-Timeout-Integration.md`; no ER-014 exists yet. This document is therefore **ER-014**, derived directly from the repository's own contents (STD-001 §7), consistent with the governing Engineering Work Order's own literal label — no numbering collision exists here, unlike the earlier "EWO-001"/"EWO-002"/"EWO-003" collisions this engineering effort has separately disclosed.

## 1. Repository Verification

**`synapse-runtime`:** path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `d4542406fa72f04d4d4f70313f31902366993c74`, tracking `origin/main`, 0 ahead / 0 behind, confirmed via `git fetch origin` immediately before authoring this report. Working tree modified by the implementation and its correction, both of which this report records: `runtime/src/lib.rs`, `services/effect-coordinator/src/internal.rs`, `services/effect-coordinator/src/lib.rs` — the identical three files the post-correction Independent Implementation Review examined; no drift occurred between that review and this report.

**`synapse-docs`:** path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `cfd82e4db5ce90eb8ed800ec3f66f1300b173cc5`, tracking `origin/main`, 0 ahead / 0 behind. Pre-existing, unrelated backlog (a modified `STD-001`, and untracked RES/RSS/ACR/GOV-002/EMO-001/`EWO-003-Message-Gateway`/`.ai` content) left completely untouched throughout every task in this milestone's own history. `work-orders/EWO-013-Effect-Cancellation-on-Actor-Termination.md` (v0.2.0, untracked) is the governing, frozen EWO, unchanged since its own Final Review. This report (`engineering-reports/ER-014-Effect-Cancellation-on-Actor-Termination.md`) is the only file this task adds to `synapse-docs`.

**Nothing staged, committed, or pushed in either repository at any point during this milestone.**

## 2. Purpose

ER-013 (Effect Timeout Integration) closed one of ARCH-008 §16.2's five Attempt-level terminal outcomes (`TimedOut`) but left ARCH-008 §21 (Cancellation Architecture) only partially wired: `cancel_effect` existed and was exercised by explicit, caller-issued cancellation, but neither of Runtime's two automatic termination paths — the Supervisor-driven `Stop` decision, and Persistent Actor durable deletion — ever called it. A pending Effect Attempt belonging to an actor that was permanently Stopped, or whose durable state was deleted, remained indefinitely `Dispatched`/`Executing`, silently orphaned, unless it happened to carry its own timeout (ER-013) that eventually fired. EWO-013 existed to close that gap: wire the existing, already-implemented, already-tested `cancel_effect` outcome method to fire automatically at the two points ARCH-008 §21 names — "requesting-actor termination" and "Persistent Actor durable deletion" — without introducing any new lifecycle state, terminal outcome, or audit event.

## 3. Engineering Summary

### 3.1 Implemented Architecture

ARCH-008 §21 for the two triggers this milestone addresses: requesting-actor termination and Persistent Actor durable deletion each cancel every pending Effect Attempt the terminated/deleted `ActorId` owns; cancellation reuses the existing `Cancelled` outcome (§16.2) exactly as explicit actor-issued cancellation already does — §21 itself lists these as parallel triggers of the identical mechanism, not as producing distinct outcomes; durable deletion's cancellation occurs strictly before `persistence.delete` is attempted, satisfying §21's explicit "before deletion is reported complete" requirement (§31 invariant 20) via the same ARCH-007 §17 ordering pattern (validate → cancel dependent state → audit → act → audit outcome) already reused for pending Timer registrations; a late provider signal after cancellation is discarded and truthfully audited, reusing the identical late-signal discipline §20/EWO-002/EWO-013 already established; ordinary Supervisor-driven Restart, and the direct public `terminate_actor_instance`, are deliberately excluded (§5, below).

### 3.2 Effect Coordinator Ownership Index

```rust
by_actor: HashMap<ActorId, Vec<EffectAttemptId>>
```

A new reverse index on `EffectCoordinatorImpl`, mirroring the exact shape precedent `by_message` (EWO-002) and `by_timer`/`attempt_timer` (ER-013) already established. Populated by a single insertion inside `record_dispatched`, immediately after the existing `record.current_attempt = Some(attempt_id.clone())` line — the one point in that function where both the freshly minted `attempt_id` and the owning `record.actor` are simultaneously in scope. Entries are appended only, never removed or reordered, on the same no-eviction basis every other correlation map in this crate already uses. Exposed via one new trait method:

```rust
fn attempts_for_actor(&self, actor: &ActorId) -> Vec<EffectAttemptId>;
```

returning every attempt ever dispatched for `actor`, in dispatch order, regardless of terminal status — an owned `Vec` clone, never a live reference, on the identical raw-correlation-fact convention `attempt_for_message`/`attempt_for_timer` already establish. No new crate dependency was required (`ActorId` was already an existing dependency via `synapse-common`).

### 3.3 Runtime Cancellation Helper

```rust
fn cancel_effects_for_actor(&mut self, actor: &ActorId) -> Result<(), RuntimeError> {
    for attempt in self.effect_coordinator.attempts_for_actor(actor) {
        if !matches!(
            self.effect_coordinator.attempt_status(&attempt),
            Some(AttemptStatus::Terminal(_))
        ) {
            self.cancel_effect(&attempt)?;
        }
    }
    Ok(())
}
```

A new private Runtime method introducing no new terminal-state logic, no new audit event, and no new public API surface: every attempt it actually cancels goes through the existing, entirely unmodified `cancel_effect`. Already-terminal attempts are left untouched by the guard.

### 3.4 Stop Integration

`execute_stop_decision` gained exactly one new line — `self.cancel_effects_for_actor(actor)?;` — immediately after its existing Timer-cancellation loop, before its own `Ok(())`.

### 3.5 Durable-Deletion Integration

`delete_actor_state` gained exactly one new call — `self.cancel_effects_for_actor(actor)?;` — immediately after its existing Timer-cancellation loop and strictly before `match self.persistence.delete(actor)`, satisfying ARCH-008 §21/§31 invariant 20's explicit ordering requirement directly from source position, not merely by inference.

### 3.6 Restart and Low-Level Termination — Deliberately Unwired

Neither `execute_restart_decision` nor the public `Runtime::terminate_actor_instance` calls `cancel_effects_for_actor`. Both exclusions are disclosed, reviewed design decisions, not oversights (§5).

## 4. Significant Engineering Decisions

1. **Restart preserves pending Effects; only Stop and durable deletion cancel them.** ARCH-008 §21 states "requesting-actor termination" as a trigger without ARCH-008 §18's own explicit symmetry for the Provider side (which names Restart, Stop, and Escalate alike as producing `ProviderLost`). This milestone resolved the question by direct, already-published, in-repository precedent rather than inventing a new architectural rule: `execute_stop_decision` already cancels every pending Timer registration for a terminated actor (EWO-008), while `execute_restart_decision` deliberately does not — Timer state is `ActorId`-durable and survives an ordinary Restart, torn down only by genuine, permanent loss. ARCH-008 §15 places Effect ownership in the identical `ActorId`-durable category, and ARCH-004 §12 independently confirms `ActorId`-attached state (capability bindings, supervision relationships) "remain[s] applicable"/"remain[s] supervised" across Restart — in contrast to ARCH-004 §14's `ActorInstanceId`-attached Mailbox contents, discarded unconditionally on every termination including Restart. This is a disclosed interpretation, not an explicit architectural rule stated for Effects specifically; it is recorded as a load-bearing decision in EWO-013 itself, not silently assumed.
2. **The public `terminate_actor_instance` is deliberately left unwired.** This function takes only an `ActorInstanceId`, with no path to the owning `ActorId` — confirmed directly: the `ActorHost` public trait exposes no `ActorInstanceId → ActorId` reverse lookup, and adding one would require either a new `Runtime`-owned reverse-index field or a new Actor Host public method, either a larger change than this milestone authorizes. This is the identical, already-accepted limitation EWO-008 already disclosed for Timer cancellation through this same call: an Effect requested by an actor terminated only through this direct call remains Runtime-tracked, non-terminal state until it separately resolves.
3. **Stop-on-first-error, not best-effort continuation.** `cancel_effects_for_actor` stops at the first error `cancel_effect` returns, leaving any later attempts in the same actor's list untouched. This mirrors the pre-existing Timer-cancellation loop's own failure shape in both call sites it is wired into, but is genuinely broader: Timer cancellation (`cancel_all_for_actor`) completes every state change atomically before its own audit loop even begins, so a Timer audit failure only ever creates an audit gap, never an uncancelled timer; Effect cancellation bundles the state change and the audit emission into one fallible call per attempt (`cancel_effect`'s own `record_cancelled` executes before its own `audit_emitter.emit`), so a failure can leave a later attempt genuinely uncancelled, not merely unaudited. This distinction was verified directly from source during independent review, not merely asserted, and is accepted as the frozen, disclosed failure model — no retry, rollback, or best-effort continuation was introduced to compensate for it.
4. **No new lifecycle state, terminal outcome, or audit event.** `Cancelled` and `effect.cancelled` (both pre-existing since EWO-001) are reused exactly as an explicit, caller-issued cancellation already produces them — ARCH-008 §21 itself lists "requesting-actor termination" and "explicit actor cancellation" as parallel triggers of the identical mechanism.

## 5. Validation Evidence

Independently re-run at implementation completion, again during each of two independent reviews, and a final time immediately before authoring this report, from the actual working tree:

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean, exit 0 |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace --all-targets --all-features` | Clean |
| `cargo test --workspace --all-targets --all-features` | **695 passed, 0 failed, 0 ignored** |

**Test totals:** 672 (ER-013 baseline) + 23 new = **695**, confirmed by direct summation of every `test result:` line across all 26 test binaries, independently re-derived at each of three separate points in this milestone's history (initial implementation, post-correction implementation, this report), not asserted from a single aggregate figure. Breakdown: 6 new in `synapse-effect-coordinator` (55 → 61), 17 new in `synapse-runtime` (269 → 286). No existing test was removed, ignored, renamed, or had an existing assertion weakened; the complete diff across all three modified files remains additive (941 insertions, 0 deletions).

## 6. Independent Review and Correction History

This milestone's own engineering history includes a genuine correction cycle, recorded truthfully rather than smoothed over:

1. The initial implementation completed, reporting `EWO-013 EFFECT CANCELLATION ON ACTOR TERMINATION IMPLEMENTED` with 691 passing tests.
2. A first Independent Implementation Review re-derived every conclusion directly from source rather than trusting that report. It confirmed the production integration itself was architecturally correct — `delete_actor_state` genuinely called the new cancellation helper immediately after Timer cancellation and strictly before `persistence.delete`, verified by direct source reading — but found that **four mandatory durable-deletion tests the implementation report explicitly claimed to exist were absent from the repository**: a basic cancellation test, an audit-ordering test, a mixed terminal/non-terminal test, and a repeated-deletion test. This review concluded:

   ```text
   EWO-013 IMPLEMENTATION REQUIRES CORRECTION
   ```

3. A targeted, narrowly scoped correction added exactly the four missing tests, exercising the real `delete_actor_state` path in each case (never calling the private cancellation helper directly). No production code was found to be defective, and none was changed.
4. Validation increased from 691 to 695 passing tests, with the correction's own diff confirmed purely additive (`runtime/src/lib.rs` alone, +211 lines, 0 deletions).
5. A second, complete, fresh Independent Implementation Review was performed — not limited to the four new tests, but re-evaluating the full implementation, specification compliance, architecture, failure semantics, and repository integrity from scratch. That review independently re-derived the same 695-test total directly from the diff itself (17 new Runtime test names, 6 new Coordinator test names, enumerated from `git diff`, not inferred from a count delta) and concluded:

   ```text
   EWO-013 IMPLEMENTATION VERIFIED — READY FOR ENGINEERING REPORT
   ```

This sequence — a genuine failing review, a narrow correction, and a genuine passing re-review — is recorded here as evidence that this engineering effort's independent-review discipline functioned as intended: an inaccurate claim in an implementation report was caught by re-deriving evidence from source rather than trusting the report, and the correction was verified rather than assumed.

## 7. Requirement Traceability

| EWO-013 requirement | Implemented in |
|---|---|
| `by_actor` reverse index | `services/effect-coordinator/src/internal.rs` |
| Insertion point (both `record.actor`/`attempt_id` genuinely in scope) | `record_dispatched`, immediately after `record.current_attempt = Some(...)` |
| `attempts_for_actor` query | `services/effect-coordinator/src/lib.rs` — trait, `EffectCoordinatorImpl`, `EffectCoordinatorHandle` |
| `cancel_effects_for_actor` helper | `runtime/src/lib.rs` |
| Stop integration | `execute_stop_decision` — one new call, after Timer cancellation |
| Durable-deletion integration, ordered before `persistence.delete` | `delete_actor_state` — one new call, after Timer cancellation, before `persistence.delete` |
| Restart exclusion (disclosed) | `execute_restart_decision` — unchanged |
| Low-level termination exclusion (disclosed) | `terminate_actor_instance` — unchanged |
| Reuse of `Cancelled`/`cancel_effect`/`record_cancelled` | `runtime/src/lib.rs` — unmodified |
| Correlated Timer cleanup | reused via `cancel_effect`'s own existing `cancel_correlated_timer_if_any` |
| Late-signal discard, truthfully audited | reused via `execute_message_capturing`'s existing, unmodified guards |
| Stop-on-first-error, no retry/rollback/best-effort | `cancel_effects_for_actor` — confirmed by direct source reading |
| Mandatory tests (23) | `services/effect-coordinator/src/lib.rs` (6), `runtime/src/lib.rs` (17) |

## 8. Architectural Assessment

Runtime remains the sole orchestrator of Actor lifecycle and durable-deletion policy (ARCH-008 §10, §21); the Effect Coordinator gained only a raw ownership-fact query (`attempts_for_actor`) — it makes no lifecycle decision of its own, exactly as it already made none for `attempt_for_message`/`attempt_for_timer`. Runtime alone decides what to do with that fact, dispatching to the existing, unmodified `cancel_effect`. No capability boundary was bypassed, no lifecycle authority moved into the Effect Coordinator, and no universal termination hook was introduced — confirmed directly: `execute_restart_decision` and `terminate_actor_instance` are both byte-for-byte unchanged. No new crate dependency, no Persistence/Timer/Capability/Actor-Host/Scheduler/Supervisor/Mailbox redesign, and no circular coupling were introduced anywhere in the diff.

## 9. Known Limitations and Accepted Behaviour

- The `by_actor` index retains every historical `EffectAttemptId` ever dispatched for an actor, including terminal ones — append-only, with no eviction mechanism, on the same already-accepted unbounded-growth basis `effects`/`attempts`/`by_message`/`by_timer` already use.
- Ordinary Supervisor-driven Restart does not automatically cancel a restarting actor's own pending Effects (§4, item 1) — a disclosed, precedented interpretation, not an explicit ARCH-008 rule stated for Effects specifically.
- The direct public `terminate_actor_instance` does not automatically cancel pending Effects (§4, item 2) — the same disclosed limitation already accepted for Timer cancellation.
- Cancellation is stop-on-first-error, not best-effort: a failure partway through cancelling a single actor's multiple pending Effects leaves later attempts genuinely uncancelled, a broader partial-failure surface than the analogous Timer-cancellation precedent (§4, item 3).
- Retry Architecture, Idempotency Metadata, Provider business/operation failure signalling, and Resource Governance remain untouched, each its own separate, future milestone per the Effect Runtime Roadmap.

None of the above is a defect of this milestone; each is a frozen, independently reviewed EWO-013 design decision or an explicitly deferred future scope item.

## 10. Conclusion

EWO-013 (Effect Cancellation on Actor Termination) successfully implemented the two cancellation triggers ARCH-008 §21 names that remained unwired after ER-013: requesting-actor termination (via the Supervisor `Stop` decision) and Persistent Actor durable deletion, each reusing the existing `Cancelled` outcome, the existing `cancel_effect` method, existing correlated-Timer cleanup, and existing late-signal discard discipline without introducing any new lifecycle state, terminal outcome, or audit event. The implementation required one genuine correction cycle — an Independent Implementation Review found that four mandatory durable-deletion tests an implementation report claimed to exist were in fact absent, a narrow correction added them without any production change, and a full, independent re-review verified the corrected state from scratch. Validation now passes completely: 695 tests, 0 failures, 0 warnings, a purely additive 941-line diff across exactly three files. Both the Restart exclusion and the low-level `terminate_actor_instance` exclusion are disclosed, precedented design decisions, not oversights. The implementation is ready for publication, subject to whatever governance disposition Denver Jacobs applies to this and every other Draft document in this corpus.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-27 | Claude (AI-assisted) | Initial report. Records the Effect Cancellation on Actor Termination implementation exactly as completed and independently verified: the `by_actor` reverse index, `attempts_for_actor`, `cancel_effects_for_actor`, Stop and durable-deletion integration, and the disclosed Restart/low-level-termination exclusions — independently re-verified against source across implementation, a first review that correctly identified a genuine test-coverage gap, a targeted correction, and a second, complete post-correction review (695 tests, 0 failures). Records the full correction-cycle history truthfully, without minimizing the first review's `REQUIRES CORRECTION` finding. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Claude (AI-assisted) | Drafted | 2026-07-27 |
| Reviewer | Independent Implementation Review (this engineering effort) — two review cycles, the first returning `REQUIRES CORRECTION`, the second returning `VERIFIED` | Approved, no rework outstanding | 2026-07-27 |
| Approval Authority | Denver Jacobs | Pending | |
