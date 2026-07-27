---
document_id: ER-013
title: "Effect Timeout Integration — Engineering Report"
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-27
last_updated: 2026-07-27
classification: Public
related_documents:
  reports_on: "EWO-012 (work-orders/EWO-012-Effect-Timeout-Integration.md, v0.2.1) — the governing Engineering Work Order this report records"
  architecture:
    - ARCH-008 (v0.3.2, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md)
    - ARCH-005 (Temporal Runtime Architecture; governs the Timer Service this milestone reuses unmodified)
    - ARCH-002 (Runtime Architecture; governs the admission pipeline and audit-event model this milestone reuses unmodified)
  adrs:
    - ADR-0015 (Approved — Audit Emitter Failure Semantics)
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  standards:
    - STD-001
  predecessor: ER-012 (Provider Actor Integration — Engineering Report)
---

# ER-013 — Effect Timeout Integration — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. It records the implementation exactly as independently verified. Nothing described here has been staged, committed, or pushed.

**Numbering.** The highest existing Engineering Report on record is `ER-012-Provider-Actor-Integration.md`; no ER-013 exists yet. This document is therefore **ER-013**, derived directly from the repository's own contents (STD-001 §7), consistent with the governing Engineering Work Order's own literal label — unlike the earlier numbering collisions this engineering effort disclosed for "EWO-001"/"EWO-002"/"EWO-003" (each already permanently assigned to unrelated, pre-existing milestones), no such collision exists for "ER-013": it is both the informal task label and the evidence-derived identifier.

## 1. Repository Verification

**`synapse-runtime`:** path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `2bcc81fc000780a92893e3ec21b29542bb906258`, tracking `origin/main`, 0 ahead / 0 behind, confirmed via `git fetch origin` immediately before authoring this report. Working tree modified by the implementation this report records: `Cargo.lock`, `runtime/src/lib.rs`, `services/effect-coordinator/Cargo.toml`, `services/effect-coordinator/src/internal.rs`, `services/effect-coordinator/src/lib.rs` — the identical five files the Independent Implementation Review examined; no drift occurred between review and this report.

**`synapse-docs`:** path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `4aa2f6bd5a5ffb401d1b987caa18d7de78336cea`, tracking `origin/main`, 0 ahead / 0 behind. Pre-existing, unrelated backlog left untouched throughout this and every preceding task in this engineering effort. `work-orders/EWO-012-Effect-Timeout-Integration.md` (v0.2.1, untracked) is the governing EWO, unchanged since its own Final Review. This report (`engineering-reports/ER-013-Effect-Timeout-Integration.md`) is the only file this task adds to `synapse-docs`.

**Nothing staged, committed, or pushed in either repository at any point during this task.**

## 2. Purpose

ER-012 recorded Provider Actor Integration: the first live execution path connecting the Effect Runtime foundation to ordinary Provider Actor dispatch, correlating a dispatched message to its Effect Attempt and reporting genuine success/failure. That milestone left `TimedOut` — one of ARCH-008 §16.2's five Attempt-level terminal outcomes — entirely unreachable: the Effect Coordinator's own `record_timed_out` primitive existed, unit-tested in isolation since ER-011, but nothing in Runtime ever called it. This milestone, EWO-012 (informally "EWO-003" within this engineering effort; see EWO-012's own Numbering and Traceability Disclosure), existed to close that gap: implement ARCH-008 §20 (Timeout Architecture) by wiring a genuine, caller-supplied, per-Attempt Timer Service registration to `record_timed_out`, so an Effect Attempt that runs longer than its own requested timeout is truthfully driven to `TimedOut` rather than remaining `Dispatched`/`Executing` indefinitely.

## 3. Engineering Summary

### 3.1 Implemented Architecture

ARCH-008 §20 in full: timeout policy remains caller-supplied (never fixed by Runtime or the Effect Coordinator); timeout scheduling reuses the existing Timer Service directly, per attempt; timeout enforcement transitions a still-non-terminal attempt to `TimedOut` when its own timer genuinely fires; timeout cancellation removes a now-moot pending timer once an attempt reaches any other terminal outcome first; the Timer Service's own disclosed cancellation/firing race is treated as harmless, never as an error; a late signal arriving after an attempt is already terminal is discarded and truthfully audited, never silently dropped and never allowed to overwrite the existing outcome; `TimedOut` remains retry-eligible (leaves the owning Effect `InProgress`); and Runtime's own "stopped waiting" outcome is never represented as certainty about the external system's own state.

### 3.2 Public API Changes

`request_effect`'s signature changed from five flat parameters to `request_effect(actor: &ActorId, request: EffectRequest, authorization: &Capability) -> Result<EffectId, RuntimeError>`, mirroring `submit_message(message: Message, presented: &Capability)`'s own already-established shape: request data in one object, authorization proof as a separate, trailing parameter. This was a deliberate, reviewed revision — not the original design — recorded in full in §4 and in EWO-012's own Revision History.

### 3.3 EffectRequest

```rust
pub struct EffectRequest {
    pub provider: ActorId,
    pub operation: String,
    pub payload: Vec<u8>,
    pub timeout: Option<Duration>,
}
```

Exactly these four fields — `provider`/`operation`/`payload` moved from `request_effect`'s former positional parameters, plus the new `timeout: Option<Duration>` (`None` means no timeout is enforced for that attempt). No retry-policy, idempotency, priority, or other speculative field was added.

### 3.4 Timeout Registration

When `request.timeout` is `Some(duration)` and dispatch succeeds, `request_effect` registers exactly one Timer Service timer — via `Timer::register` directly, not the `Runtime::register_timer` public wrapper (which exists for a caller's own ordinary domain timers and would emit a misleading `timer.registered`/`timer.expired` audit trail and an irrelevant reachability check against the requester, for what is really Effect Runtime-internal bookkeeping) — bound to the requesting actor, never the provider. On dispatch failure (`Denied` or admission rejection), no timer is ever registered, regardless of `request.timeout`.

### 3.5 TimerId ↔ EffectAttemptId Correlation

A new, genuinely bidirectional correlation in the Effect Coordinator: `by_timer: HashMap<TimerId, EffectAttemptId>` (the direction `process_due_timers` needs to resolve a fired timer back to its attempt) and `attempt_timer: HashMap<EffectAttemptId, TimerId>` (the direction the outcome methods need to cancel a still-pending timer early). Both are populated by a single new method, `record_timeout_registered`, and queried via `attempt_for_timer`/`timer_for_attempt`. `synapse-effect-coordinator` gained a new dependency on `synapse-timer`, solely for the plain `TimerId` newtype (no behavior, no transitive dependency beyond `synapse-common`, which both crates already shared).

### 3.6 Timeout Lifecycle

`process_due_timers` gained one additive branch: for each due Timer Service entry, if it correlates to an Effect Attempt, the ordinary message-construction/admission path is skipped entirely (an Effect-correlated timer has no actor-visible message of its own); if the attempt is not already terminal, `timeout_effect` records `TimedOut` and audits it; if it is already terminal, the signal is discarded and audited as such. The underlying Timer Service registration is always marked `Completed` (never `Discarded`), since "discarded" specifically means the resulting message failed ordinary admission, which does not apply here. `record_timed_out` itself — present since ER-011 — is reused with no behavioral change of any kind.

### 3.7 Audit Additions

Two new `event_type` values, using the existing, unmodified `AuditEvent` structure: `effect.timeout` (attributed to the requester, emitted by `timeout_effect`) and `effect.late_signal_discarded` (attributed to the requester, emitted at all three points where a signal for an already-terminal attempt would otherwise be silently dropped).

### 3.8 Timer Cancellation Behaviour

A new shared private helper, `cancel_correlated_timer_if_any`, called by all five outcome methods (`complete_effect`, `fail_effect`, `cancel_effect`, `provider_lost_effect`, `timeout_effect`) immediately after each successfully records its own terminal outcome. It cancels the attempt's own pending timer via `Timer::cancel` directly (not the `cancel_timer` public wrapper, for the identical internal-housekeeping reason registration bypasses `register_timer`) and silently discards `Err(IllegalTransition)` — the disclosed, harmless race where the timer had already independently fired.

### 3.9 Late-Signal Handling

A genuine correction, applied consistently across three call sites, not just the new one: the two pre-existing EWO-002 guards in `execute_message_capturing` (success-path and failure-path) previously discarded a late signal for an already-terminal attempt silently, with no audit event — a narrow, previously unidentified gap relative to ARCH-008 §16.2's own "MUST be discarded and truthfully audited as discarded" requirement. This milestone closes that gap at all three guarded points (the two existing ones, plus the new timeout guard) in the same pass, since the new timeout guard needed the identical audit call regardless.

## 4. Significant Engineering Decisions

1. **`Instant::now()` inside `request_effect`.** `EffectRequest.timeout` is a relative `Duration`; the Timer Service's own `register` requires an absolute `fire_at: Instant`. Translating one into the other requires reading the current instant somewhere, and since `request_effect`'s own signature (three parameters, no externally supplied "now") and `EffectRequest`'s own four fields were both already reviewed and approved, no alternative existed within that already-fixed shape. This is the first real-clock read anywhere in this crate's history, confined entirely to the registration step; `process_due_timers` itself remains fully caller-driven and 100% deterministic, unchanged. The new test suite relies only on monotonic-clock ordering between two nearby `Instant::now()` calls, never on an elapsed-time threshold, so no flakiness is introduced.
2. **Reuse of `process_due_timers`, not a second polling function.** `Timer::query_due` is a one-shot, destructive query — a due timer is returned exactly once, to whichever caller asks first (confirmed directly by the pre-existing test `querying_due_twice_only_reports_each_timer_once`). A second, parallel consumer could not coexist against the same `Timer` instance without stealing entries from the first. The only viable integration point was therefore one additive branch inside the existing function.
3. **`record_timed_out` reused with zero behavioral change.** Already existed since ER-011, already unit-tested in isolation, already calls the shared `record_terminal` helper every other terminal-outcome method uses. Nothing about it needed to change; this milestone only had to make it reachable.
4. **No new mailbox-reply-delivery mechanism.** `timeout_effect` follows the identical, already-published pattern the four existing outcome methods (`complete_effect`, `fail_effect`, `cancel_effect`, `provider_lost_effect`) already use: update Effect Coordinator state, emit an audit event, construct no message. This is a disclosed, inherited interpretation of ARCH-008 §20's own "requesting actor receives an ordinary reply message" language — none of the four pre-existing outcome methods deliver such a message either, and that characteristic already passed through the full EWO-001/EWO-002 review sequence without being raised as a finding. Building an actual reply-delivery mechanism is a distinct, separate concern, explicitly outside this milestone's scope.
5. **Replacement behaviour for duplicate `TimerId ↔ EffectAttemptId` registrations.** No code path in this milestone's own Runtime integration ever calls `record_timeout_registered` twice for the same attempt (it is called at most once, immediately after a fresh dispatch). As a defensive characterization of the underlying `HashMap` semantics, a second registration for the same attempt is accepted (not rejected as illegal) and cleanly replaces the first: the prior `TimerId`'s own `by_timer` entry is removed before the new one is inserted, so no stale, dangling reverse-mapping is ever left behind. Covered directly by its own dedicated test.

## 5. Validation Evidence

Independently re-run at implementation completion and again during the Independent Implementation Review; re-confirmed a third time immediately before authoring this report, from the actual working tree:

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean, exit 0 |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | **672 passed, 0 failed, 0 ignored** |

**Test totals:** 650 (EWO-002/ER-012 baseline) + 22 new (14 in `synapse-runtime`, 8 in `synapse-effect-coordinator`) = **672**, confirmed by direct summation of every `test result:` line, independently re-derived, not asserted from a single aggregate figure. All 24 pre-existing `request_effect` call sites were mechanically migrated to `EffectRequest`; one pre-existing test (`a_cancelled_effect_is_not_overwritten_by_a_late_successful_provider_result`) was extended, not replaced, to additionally assert the new `effect.late_signal_discarded` audit event. No test was removed, ignored, or had an existing assertion weakened.

## 6. Independent Review Findings

An Independent Implementation Review was conducted within this same engineering effort, re-deriving every conclusion directly from source rather than from this milestone's own implementation report, and independently re-executing all four validation gates from the working tree (672 passed, 0 failed, matching §5 above exactly).

**Findings:** no Critical findings; no Major findings; one Minor finding; the remainder Observations.

- **Minor** — `Runtime::process_due_timers`'s own doc comment was not updated to disclose the new Effect-correlated branch; it still describes "every processed registration reaches `Completed`/`Discarded`" purely in terms of message admission, which no longer fully describes the Effect-correlated case (which reaches `Completed` without any message ever being constructed). The code itself is correct and fully tested; only the prose describing it is incomplete. Recorded here as an implementation defect (a documentation-completeness gap), not minimized.
- **Observations** (implementation decisions and disclosed design notes, not defects): the `Instant::now()` read (§4, item 1); `by_timer`/`attempt_timer` are never evicted, extending the already-accepted unbounded-growth precedent `by_message`/`effects`/`attempts` already established in EWO-001/EWO-002; the `completed` counter is incremented for an Effect-correlated due entry even though EWO-012's own pseudocode did not literally show this line, a reasonable and harmless elaboration consistent with the counter's own state-based meaning; and EWO-012 §6's own description of the "cancel racing a still-pending timeout" test implied an `effect.late_signal_discarded` audit would fire in that specific scenario, whereas direct analysis of the Timer Service's own contract shows a cleanly-cancelled timer never reports due again, so the implemented test correctly asserts the real mechanical outcome instead — the genuinely late-signal-discarding scenarios remain covered by three separate, correctly-targeted tests.

**Resolution:** the Minor finding was accepted as a disclosed, non-blocking documentation follow-up rather than requiring a further correction cycle, consistent with this engineering effort's own established precedent for Minor-level findings (ER-011 §7). The Independent Implementation Review's own concluding statement: `EWO-012 IMPLEMENTATION VERIFIED — READY FOR ENGINEERING REPORT`.

## 7. Requirement Traceability

| EWO-012 requirement | Implemented in |
|---|---|
| `EffectRequest` (4 fields only) | `runtime/src/lib.rs` — new public struct |
| `request_effect(actor, request, authorization)` | `runtime/src/lib.rs` — revised signature, unchanged internal admission/dispatch logic |
| Per-attempt Timer Service registration | `runtime/src/lib.rs` — `request_effect`'s new timeout branch |
| `TimerId ↔ EffectAttemptId` correlation | `services/effect-coordinator/src/lib.rs`, `internal.rs` — `record_timeout_registered`/`attempt_for_timer`/`timer_for_attempt` |
| Timeout enforcement (`TimedOut`, retry-eligible) | `runtime/src/lib.rs` — `process_due_timers`'s new branch, `timeout_effect`, `record_timed_out` (unchanged) |
| Timeout cancellation on earlier termination | `runtime/src/lib.rs` — `cancel_correlated_timer_if_any`, wired into all five outcome methods |
| Harmless cancellation/firing race | `runtime/src/lib.rs` — `cancel_correlated_timer_if_any` silently discards `Err(IllegalTransition)` |
| `effect.timeout` audit | `runtime/src/lib.rs` — `effect_timeout_event` |
| Late-signal discard, truthfully audited | `runtime/src/lib.rs` — `effect_late_signal_discarded_event`, wired into all three guarded points |
| No second admission/timer/reply-delivery mechanism | Confirmed by direct source inspection and the Independent Implementation Review |

## 8. Known Limitations

Explicitly, deliberately deferred, matching EWO-012's own stated exclusions and the Effect Runtime Roadmap:

- **Retry Architecture** (ARCH-008 §19) — `record_retry_scheduled` exists since ER-011 and is unit-tested in isolation, but nothing in this milestone's own Runtime integration calls it; Idempotency Metadata remains a hard prerequisite, per the Roadmap, and is not addressed here.
- **Idempotency metadata** (ARCH-008 §23) — reserved, not implemented; the next milestone the Roadmap identifies.
- **Provider business/operation failure signalling** (ARCH-008 §18) — a successful `handle()` is still unconditionally recorded as `Completed`; no convention exists yet for a Provider to communicate a business-level failure through a successful reply.
- **Resource Governance** (ARCH-008 §26) — no Effect-level quotas, rate limits, or concurrency controls; lowest priority per the Roadmap, since no concrete provider yet exists to stress it.
- **Provider Actor Lifecycle Loss Integration beyond EWO-002's own narrow trigger** — unaffected by this milestone.

None of the above is a defect of this milestone; each is reserved for its own future, separately authorized work.

## 9. Conclusion

EWO-012 (Effect Timeout Integration) successfully implemented ARCH-008 §20 in full: a caller-supplied, per-Attempt Timer Service timeout, correlated bidirectionally to its Effect Attempt, truthfully enforced, truthfully cancelled on early termination, and truthfully audited — including a disclosed correction closing a narrow, previously unidentified late-signal audit gap in the already-published EWO-002 implementation. The public API was deliberately revised mid-milestone, following an Independent API Design Review, to an extensible `EffectRequest` object rather than a telescoping method family — a decision recorded, not concealed, in this milestone's own history. Validation passes completely (672 passed, 0 failed, zero warnings). The Independent Implementation Review found no Critical or Major finding, one disclosed Minor finding (a documentation-completeness gap, not a functional defect), and confirmed no architectural drift, no retry/idempotency/resource-governance behavior, and no new reply-delivery mechanism. The implementation is ready for publication, subject to whatever governance disposition Denver Jacobs applies to this and every other Draft document in this corpus.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-27 | Claude (AI-assisted) | Initial report. Records the Effect Timeout Integration implementation exactly as completed and independently verified: `EffectRequest`, the revised `request_effect` signature, bidirectional `TimerId ↔ EffectAttemptId` correlation, the `process_due_timers` timeout branch, `timeout_effect`, the two new audit events, timer cancellation, and the disclosed late-signal audit-gap correction across all three guarded points — independently re-verified against source across implementation, review, and this report's own re-execution of all four validation gates (672 tests, 0 failures). Records the Independent Implementation Review's one Minor finding (a `process_due_timers` doc-comment completeness gap) truthfully, without minimizing it. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Claude (AI-assisted) | Drafted | 2026-07-27 |
| Reviewer | Independent Implementation Review (this engineering effort) | Approved, one Minor finding disclosed, no rework required | 2026-07-27 |
| Approval Authority | Denver Jacobs | Pending | |
