---
document_id: EWO-013
title: "Effect Cancellation on Actor Termination"
version: 0.2.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-27
last_updated: 2026-07-27
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-008 (v0.3.2, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md) — the sole architectural authority this EWO implements; not amended by this EWO
    - ARCH-004 (Actor Lifecycle and Supervision Architecture) — governs Restart/Stop mechanics this EWO integrates with, unmodified; §14's Mailbox-discarded-on-every-termination precedent and §12's Restart Identity Semantics are directly relied on (§4, §5.1 below)
    - ARCH-007 (Persistent Actor Architecture) — §17's deletion-coordination-ordering pattern (validate authority → cancel dependent state → audit → attempt the definitive act → audit outcome) is reused, not redesigned, per ARCH-008 §21
    - ARCH-005 (Temporal Runtime Architecture) — §23's `cancel_all_for_actor`/Stop-cancels-Restart-preserves precedent is the direct analogy this EWO follows for Effects (§4, §5.1)
  adrs:
    - ADR-0015 (Approved — Audit Emitter Failure Semantics)
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  predecessor: "EWO-012 / ER-013 (Effect Timeout Integration) — its `cancel_effect`/`cancel_correlated_timer_if_any` pattern is reused unmodified, not redesigned; EWO-008's own existing `execute_stop_decision` doc comment (Restart does not cancel pending Timer registrations, only Stop does) is the direct, already-published precedent this EWO follows for Effects"
  reported_by: "The next sequentially available Engineering Report identifier per STD-001 §7 at the time this EWO is implemented — not assumed in advance"
  base_state:
    runtime_head: d4542406fa72f04d4d4f70313f31902366993c74
    docs_head: cfd82e4db5ce90eb8ed800ec3f66f1300b173cc5
---

# EWO-013 — Effect Cancellation on Actor Termination

Registered per STD-001 §46 (Engineering Work Orders). This authorizes implementation work only. It does not itself constitute approval, and does not amend ARCH-008 or any other architecture, standards, or governance document.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-013 |
| Title | Effect Cancellation on Actor Termination |
| Version | 0.2.0 |
| Status | Draft |
| Author | Denver Jacobs |
| Created | 2026-07-27 |
| Predecessor milestone | EWO-012 / ER-013 — Effect Timeout Integration |
| Governing architecture | ARCH-008 v0.3.2 (Approved) |

---

## 1. Purpose

Implement two of the cancellation triggers ARCH-008 §21 (Cancellation Architecture) already, constitutionally names: **requesting-actor termination** and **Persistent Actor durable deletion**. Today, neither `Runtime`'s Supervisor-driven `Stop` path nor its durable-deletion path (`delete_actor_state`) touches the Effect Coordinator at all — a pending Effect Attempt belonging to an actor that is permanently gone remains indefinitely `Dispatched`/`Executing`, silently orphaned, unless it happens to carry its own timeout (EWO-012) that eventually fires. This EWO closes that gap by wiring the existing, already-implemented, already-tested `cancel_effect` outcome method (ARCH-008 §16.2, §21) to fire automatically at the two points ARCH-008 §21 names, rather than requiring an explicit, separate caller-issued cancellation for every pending Effect an about-to-be-removed actor happens to hold.

This EWO does not introduce a new lifecycle state, a new terminal outcome, or a new audit event type. `Cancelled` (already defined, already implemented, already exercised by explicit-cancellation tests since EWO-001) is the outcome this trigger produces — ARCH-008 §21 lists "requesting-actor termination" and "explicit actor cancellation" side by side as parallel triggers of the same existing mechanism, not as producing distinct outcomes.

**Preceding Architecture Readiness Review.** A Milestone Architecture Readiness Review of this milestone identified that ARCH-008 §21's "requesting-actor termination" trigger is stated at the instance-termination level, while Effect ownership itself is deliberately `ActorId`-level (durable), per §15 — and, unlike the Provider side (§18, which explicitly treats a Restart, Stop, or Escalate of the instance handling an attempt alike, all producing `ProviderLost`), ARCH-008 contains no equally explicit statement of whether a *requesting* actor's ordinary Supervisor-driven Restart should also cancel that actor's own pending Effects, or only a genuinely permanent loss. The review found this resolvable by direct, already-published, in-repository precedent rather than by a new architectural decision: `execute_stop_decision` already cancels every pending Timer registration for the terminated actor (EWO-008), while `execute_restart_decision` deliberately does not — an already-approved choice that ActorId-durable state (Timers) survives an ordinary Restart and is only torn down by genuine, permanent loss (Stop). Effects are placed in the identical ActorId-durable category by ARCH-008 §15 itself (explicitly, "for the identical durability reason" §11's Timer model and other ARCH documents' own identity models already establish), not in the instance-scoped Mailbox category ARCH-004 §14 already discards unconditionally on every termination including Restart. This reading is independently reinforced by ARCH-004 §12 (Restart Identity Semantics) directly, in already-published, already-approved text: "Capability bindings attach to `ActorId`... and therefore remain applicable to the replacement incarnation automatically, without a new ambient grant," and "Supervision relationships... attach to `ActorId`, never to `ActorInstanceId` — a supervised logical actor remains supervised across any number of restarts." Effect ownership sits in the same `ActorId`-attached category these two already-decided cases occupy, not in the `ActorInstanceId`-attached category §14 places Mailbox contents in. §3 below and §5.1 record this resolution explicitly, as a disclosed interpretation this EWO adopts rather than invents, consistent with this engineering effort's own established practice of disclosing a consequential interpretive call rather than silently assuming it or blocking on it.

**Correction note (v0.2.0).** An Independent Review of v0.1.0 concluded `EWO-013 REQUIRES CORRECTION`, identifying three specification defects (not architectural defects): an internal contradiction in §5.3's own pseudocode, citing `record.actor` at an insertion point in `record_dispatched` where that binding does not yet exist; undefined failure semantics for `cancel_effects_for_actor` when `cancel_effect` returns an error mid-loop; and a missing mandatory test category (a late provider signal arriving after an automatic, Stop- or deletion-triggered cancellation) — the same class of race EWO-012 itself already treats as the most architecturally significant. This revision corrects each exactly as identified (§5.3, §5.4, §6), adds the review's requested minor test coverage (Stop with multiple outstanding Effects; repeated Stop; repeated durable deletion — §6), and strengthens two citations the review found merely adequate rather than strong (ARCH-004 §12's direct textual precedent for `ActorId`-attached state surviving Restart, §1 and §3; an explicit statement of `attempts_for_actor`'s dispatch-order guarantee, §5.3). No design, scope, or architectural decision changes as a result — the Restart-exclusion interpretation, the reuse of `Cancelled`/`cancel_effect` unmodified, and every integration point (§5.5, §5.6) are exactly as v0.1.0 specified.

## 2. Repository Baseline

- `synapse-runtime` @ `d4542406fa72f04d4d4f70313f31902366993c74` (published; EWO-012/ER-013 Effect Timeout Integration).
- `synapse-docs` @ `cfd82e4db5ce90eb8ed800ec3f66f1300b173cc5` (published; ARCH-008 v0.3.2 Approved, ER-013).

An implementer MUST re-verify both repositories are still at these exact commits (or a documented, reconciled later state) before beginning implementation, per this engineering effort's own established repository-verification discipline.

## 3. Architectural Scope — Requirements Extracted from ARCH-008, with Evidence

| Requirement | ARCH-008 citation |
|---|---|
| Requesting-actor termination is a cancellation trigger | §21, second bullet |
| Persistent Actor durable deletion MUST cancel or terminally resolve every pending Effect associated with the deleted `ActorId` before deletion is reported complete | §21, fourth bullet; §31 invariant 20 |
| Durable deletion reuses, without redesigning, ARCH-007 §17's ordering pattern (validate authority → cancel dependent state → audit → attempt the definitive act → audit outcome), extended to pending Effects exactly as it was already extended to pending timers | §21, fourth bullet |
| A cancelled attempt MUST NOT later become successfully completed in Runtime state; a late provider completion after cancellation is discarded and truthfully audited, never applied | §21, fifth paragraph; identical to §20's existing late-signal discipline, already implemented (EWO-002/EWO-012), reused unmodified |
| Provider cancellation support MAY be best-effort; the Effect Coordinator's own bookkeeping obligation (recording `Cancelled`) is unaffected either way | §21, fifth paragraph |
| `Cancelled` does not, by itself, mean the external system performed no operation ("External outcome uncertainty") | §21 |
| Effect ownership is tracked by the durable `ActorId`, never the ephemeral `ActorInstanceId` | §15 |
| No new lifecycle state, terminal outcome, or `Unknown`/`Indeterminate` state is introduced by cancellation | §21, "External outcome uncertainty" closing paragraph |
| **Disclosed interpretation (this EWO, §1 above):** requesting-actor termination cancels pending Effects only on genuine, permanent instance loss (Supervisor `Stop`) and on durable deletion — not on an ordinary Supervisor `Restart` — mirroring the already-published, already-approved Timer precedent (`execute_stop_decision` cancels pending timers; `execute_restart_decision` does not) and the ActorId-durable category ARCH-008 §15 already places Effect ownership in | ARCH-008 §21 (silent on Restart specifically); ARCH-004 §12 (capability bindings and supervision relationships explicitly "remain applicable"/"remain supervised" across Restart — the direct, already-approved precedent for `ActorId`-attached state surviving Restart); ARCH-004 §14 (Mailbox contrast — `ActorInstanceId`-attached state, discarded unconditionally); this EWO's own §5.1, adopted as a disclosed implementation-design decision, not an architecture amendment |

No behavior beyond this table is inferred. Where ARCH-008 is silent on the Restart-specific question (above), this EWO resolves it by direct precedent and discloses the resolution, per §21's own established practice (ARCH-008 itself defers analogous implementation-level questions to Engineering Work Orders — e.g. the "concrete notification API" for Provider lifecycle loss, §21 third bullet).

## 4. Existing Implementation Analysis — Components to Be Reused

- **`Runtime::cancel_effect`** (`runtime/src/lib.rs`) — resolves the requesting actor, records `Cancelled` (auto-accepting it, since `Cancelled` is never retry-eligible), cancels any correlated timeout timer, and emits the existing `effect.cancelled` audit event. Reused **entirely unchanged** — this EWO adds a new automatic *caller* of this existing method; it does not modify the method itself.
- **`Runtime::execute_stop_decision`** (`runtime/src/lib.rs:2539`) — already cancels every pending Timer registration for the terminated `actor` via `self.timer.cancel_all_for_actor(actor)`, in a loop, each iteration emitting `timer_cancelled_event`. Its own existing doc comment explicitly states this is "the one call site this milestone \[EWO-008\] wires for Stop-triggered removal: it is reachable with the owning `ActorId` already in scope, unlike `Runtime::terminate_actor_instance`... which takes only an `ActorInstanceId` and has no path to the logical `ActorId`... no reverse lookup exists anywhere in this workspace." This is the direct precedent and the direct integration point for Effect cancellation (§5.2) — `actor: &ActorId` is already a parameter of this function.
- **`Runtime::execute_restart_decision`** (`runtime/src/lib.rs:2488`) — does **not** call `cancel_all_for_actor`; pending Timer registrations for `actor` survive an ordinary Restart unchanged. This EWO's disclosed interpretation (§1, §3) follows this exact precedent: Effect cancellation is likewise **not** wired here.
- **`Runtime::delete_actor_state`** (`runtime/src/lib.rs:2910`) — already has `actor: &ActorId` in scope, and already cancels every pending Timer registration for `actor` (identical `cancel_all_for_actor` loop) strictly before calling `self.persistence.delete(actor)` — the exact ARCH-007 §17 ordering position ("cancel dependent state" before "attempt the definitive act") this EWO's new Effect-cancellation step reuses for the identical position.
- **Effect Coordinator (`synapse-effect-coordinator`)** — `EffectRecord.actor: ActorId` is already stored per Effect (`internal.rs`); `AttemptRecord.effect: EffectId` links each attempt back to its owning Effect. No `ActorId`-keyed reverse index exists today — confirmed directly: `actor_of(effect: &EffectId) -> Option<ActorId>` is the only ActorId-related lookup, forward-only. The `by_message: HashMap<MessageId, EffectAttemptId>` (EWO-002) and `by_timer`/`attempt_timer` (EWO-012) maps are the direct, reusable shape precedent for the new reverse index this EWO adds (§5.2) — populated once, at the point an attempt is minted, never evicted, on the same basis every other correlation map in this crate is never evicted.
- **`Runtime::effect_requester`**, **`cancel_correlated_timer_if_any`**, **`AttemptStatus::is_terminal`** (all pre-existing, unmodified) — reused inside `cancel_effect` exactly as today; this EWO's new code never duplicates their logic, only supplies the new list of attempt ids to iterate and call `cancel_effect` against.
- **`ActorHost` public trait** (`core/actor-host/src/lib.rs`) — confirmed to expose no `ActorInstanceId → ActorId` reverse lookup (only the forward `live_instance(actor) -> ActorInstanceId`); the internal `instances: HashMap<ActorInstanceId, ActorId>` field is private. This is why `Runtime::terminate_actor_instance` cannot be wired by this EWO without either a new `Runtime`-owned reverse-index field or a new Actor Host public method — both a larger change than this milestone's own scope authorizes (§7).

No reusable pattern from EWO-001/EWO-002/EWO-012 is bypassed; every new piece of state mirrors an already-established shape, and no new terminal-outcome or audit-event logic is introduced anywhere.

## 5. Implementation Design

### 5.1 Implementation Objectives

1. Give the Effect Coordinator a way to enumerate every Effect Attempt ever dispatched for a given `ActorId` (all attempts, terminal and non-terminal alike — the caller filters, exactly as the `by_message`/`by_timer` callers already do).
2. Add a Runtime-internal helper that, given an `ActorId`, cancels every currently non-terminal attempt for that actor by calling the existing, unmodified `cancel_effect` — introducing no new terminal-state logic, no new audit event, and no new public API surface.
3. Call that helper from `execute_stop_decision`, immediately after its existing Timer-cancellation loop.
4. Call that helper from `delete_actor_state`, immediately after its existing Timer-cancellation loop and strictly before `self.persistence.delete(actor)` — the ARCH-007 §17 ordering position.
5. Do **not** call it from `execute_restart_decision` or from the public `Runtime::terminate_actor_instance` — both exclusions disclosed explicitly (§1, §3, §7), not silently omitted.

### 5.2 Components Affected

- `services/effect-coordinator/src/internal.rs` / `lib.rs` — one new field (`by_actor: HashMap<ActorId, Vec<EffectAttemptId>>`), populated inside the existing `record_dispatched` body at the specific point §5.3 identifies (the owning `ActorId` is already resolved there via the existing `self.effects.get_mut(effect)` lookup; capturing it for the new map requires no new parameter and no signature change to `record_dispatched`); one new trait method, `attempts_for_actor`.
- `runtime/src/lib.rs` — one new private helper, `cancel_effects_for_actor`; one new call site inside `execute_stop_decision`; one new call site inside `delete_actor_state`. No change to `cancel_effect`, `execute_restart_decision`, `terminate_actor_instance`, or any audit-event helper.
- No change to `synapse-effect-coordinator`'s `Cargo.toml` — `ActorId` is already an existing dependency (`synapse-common`); no new crate dependency is introduced by this EWO, unlike EWO-012's `synapse-timer` addition.
- No change to Actor Host, Timer Service, Capability Authority, Persistence Service, Supervisor, or Scheduler.

### 5.3 Effect Coordinator Changes

Add to the `EffectCoordinator` trait and `EffectCoordinatorImpl`/`EffectCoordinatorHandle`:

```text
fn attempts_for_actor(&self, actor: &ActorId) -> Vec<EffectAttemptId>;
```

Internal state: one new field on `EffectCoordinatorImpl`:

```text
by_actor: HashMap<ActorId, Vec<EffectAttemptId>>,
```

**Insertion point, corrected.** `record_dispatched`'s existing body resolves the owning `ActorId` only once it reaches its own final block, where it already looks up `let record = self.effects.get_mut(effect).expect(...)` to perform the existing `record.status = EffectStatus::InProgress; record.current_attempt = Some(attempt_id.clone());` update — `record.actor` is not in scope any earlier than this point (in particular, it is not yet bound at the point the new `AttemptRecord` is inserted, several lines earlier). The `by_actor` push MUST therefore be added immediately after that existing `record.current_attempt = Some(attempt_id.clone());` line, reading `record.actor.clone()` from the same, already-obtained `record` binding, in the same statement block — not earlier in the function:

```text
record.status = EffectStatus::InProgress;
record.current_attempt = Some(attempt_id.clone());
self.by_actor.entry(record.actor.clone()).or_default().push(attempt_id.clone());
```

This is the only point in `record_dispatched` where both the freshly minted `attempt_id` and the owning `record.actor` are simultaneously, genuinely in scope. No new parameter is added to `record_dispatched`'s own signature — the owning `ActorId` is already in scope inside its existing body, at this specific point.

`attempts_for_actor` returns a clone of the stored `Vec<EffectAttemptId>` (or an empty `Vec` if `actor` has never had an Effect dispatched) — it does not filter by status; exactly as `attempt_for_message`/`attempt_for_timer` return raw correlation facts and leave any terminal-state check to the caller, `attempts_for_actor` does the same, consistent with this crate's own established convention that the Effect Coordinator never itself decides what to do with a correlation fact. Because `by_actor`'s value is a `Vec`, not a `HashSet`, and it is only ever appended to (never reordered, never removed from), iteration over a returned `attempts_for_actor` result is deterministic and always reflects dispatch order — the same order-preservation guarantee `EffectCoordinatorImpl`'s own `next_attempt_sequence` counter already establishes for attempt minting generally, not a new one this EWO invents.

No existing method changes behavior. `record_dispatched`'s own existing return value, error conditions, and all other side effects are unchanged.

### 5.4 Runtime Changes — the Cancellation Helper

Add one new private helper:

```text
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

This introduces no new terminal-outcome recording, no new audit event, and no new error variant: every attempt this loop actually cancels goes through the existing, unmodified `cancel_effect`, which already resolves the requester, records `Cancelled`, cancels any correlated timeout timer, and emits the existing `effect.cancelled` audit event, exactly as an explicit caller-issued cancellation already does today. An attempt already terminal (whether `Completed`, `Failed`, `Cancelled`, `TimedOut`, or `ProviderLost`) is left untouched — mirroring the identical "already terminal, do nothing further" discipline `execute_message_capturing`'s own late-signal guards, and `process_due_timers`'s own EWO-012 branch, already establish. An actor with no dispatched Effects at all produces an empty list and this function is a no-op, exactly as `cancel_all_for_actor` already behaves for an actor with no pending timers.

Unlike `cancel_correlated_timer_if_any` (which silently discards a harmless race, per §20), this helper's own iteration does not itself race against anything new: `attempt_status` is read and `cancel_effect` is called within the same synchronous Runtime call, with no intervening mutation possible — the same execution-model guarantee every other Runtime method already relies on.

**Failure semantics, explicitly specified.** The `?` after `self.cancel_effect(&attempt)?` means iteration stops at the first error `cancel_effect` returns: any attempts later in the list (for that same actor) are left exactly as they were — non-terminal, uncancelled — and the error propagates unchanged to `cancel_effects_for_actor`'s own caller (`execute_stop_decision` or `delete_actor_state`), which itself propagates it unchanged to its own caller in turn. This means a failure here can surface after the actor's instance has already been irreversibly terminated (`execute_stop_decision`) or after its persistent state has already been deleted (`delete_actor_state`), with some — not necessarily all — of that actor's pending Effects left uncancelled. This is not a new failure shape this EWO invents: it is the identical shape the existing Timer-cancellation loop in each of those same two functions already has today (`self.core.audit_emitter.emit(timer_cancelled_event(actor))?` inside a `for` loop, stopping at the first audit-emitter failure and leaving any remaining timers uncancelled, with the instance already terminated or the state already deleted regardless). `cancel_effect`'s own realistic failure mode here is the same one — an `AuditEvent` emission failure (ADR-0015) — since `effect_requester` and `record_cancelled` are structurally expected to succeed given the immediately preceding `attempt_status` check. This behavior is accepted on the same basis the pre-existing Timer-cancellation shape already is: it is not made worse by this EWO, only extended to a second, independent dependent-state category in the same already-accepted way.

### 5.5 Integration — `execute_stop_decision`

Immediately after the existing Timer-cancellation loop (`for cancelled in self.timer.cancel_all_for_actor(actor) { ... }`) and before the function's existing `Ok(())`, add:

```text
self.cancel_effects_for_actor(actor)?;
```

No other change to `execute_stop_decision`. Ordering: `terminate_instance` → `actor_terminated_event` → Timer cancellation (existing) → Effect cancellation (new) → `Ok(())`. This mirrors the existing Timer-cancellation position exactly — both are "dependent-state cleanup following a confirmed, permanent instance loss," performed in the same step, with the Effect cancellation ordered after Timer cancellation only because it is being added second, not because either ordering is architecturally required relative to the other (§21 does not distinguish an ordering between these two independent dependent-state categories).

### 5.6 Integration — `delete_actor_state`

Immediately after the existing Timer-cancellation loop and strictly before `match self.persistence.delete(actor) { ... }`, add:

```text
self.cancel_effects_for_actor(actor)?;
```

This is the ARCH-007 §17 ordering position ARCH-008 §21 requires explicitly: "Durable deletion MUST cancel or terminally resolve every pending Effect associated with the deleted `ActorId` before deletion is reported complete." Every `effect.cancelled` audit event this produces is therefore emitted before `persistence_deletion_completed_event` (or `persistence_deletion_failed_event`), satisfying the ordering requirement directly and verifiably from the audit sequence, not merely from code position.

### 5.7 Explicitly Not Integrated — `execute_restart_decision` and `terminate_actor_instance`

- **`execute_restart_decision`** — no call to `cancel_effects_for_actor` is added. Per §1/§3's disclosed interpretation: an ordinary Supervisor-driven Restart does not cancel the restarting actor's own pending Effects, on the same already-approved basis it does not cancel that actor's own pending Timer registrations (EWO-008). A pending Effect Attempt survives a Restart exactly as a pending Timer registration already does, remaining associated with the same, unchanged `ActorId` (§15) across the instance replacement.
- **`Runtime::terminate_actor_instance`** — no call to `cancel_effects_for_actor` is added. This function takes only an `ActorInstanceId`, with no path to the owning `ActorId` (§4) — the identical, already-disclosed, already-accepted limitation `execute_stop_decision`'s own existing doc comment states for Timer cancellation applies here without modification: an Effect requested by an actor terminated only through this direct public call (never through Supervisor's `Stop` decision, and never through durable deletion) is not proactively cancelled by this EWO. It remains Runtime-tracked, non-terminal state until it separately resolves — by its own timeout (EWO-012), by an explicit `cancel_effect` call, or by a later genuine `Stop`/deletion of the same actor. This is a disclosed, narrow, already-precedented limitation, not a silently unresolved gap.

### 5.8 Determinism

No new non-determinism. `cancel_effects_for_actor` performs no clock read, no I/O, and no randomness — it is a pure, synchronous iteration over already-in-memory state, calling an already-deterministic existing method (`cancel_effect`).

## 6. Testing Strategy

Every behavior in §3/§5 maps to at least one test. No test relies on real elapsed time.

**Unit tests (Effect Coordinator, `services/effect-coordinator`):**
- `attempts_for_actor` returns every attempt dispatched for that actor, across multiple Effects, in dispatch order.
- `attempts_for_actor` returns an empty `Vec` for an actor with no dispatched Effects.
- `attempts_for_actor` for actor A does not include any attempt dispatched for actor B.
- `record_dispatched`'s own existing behavior (return value, `EffectStatus`/`AttemptStatus` transitions, error conditions) is unchanged — re-confirm the full existing `record_dispatched` test set still passes unmodified.

**Integration tests (Runtime, `runtime/src/lib.rs`):**
- `execute_stop_decision` on an actor with one non-terminal (`Dispatched`) Effect Attempt: the attempt transitions to `Cancelled`; `effect.cancelled` is emitted, attributed to the actor.
- `execute_stop_decision` on an actor with **multiple** non-terminal Effect Attempts (across more than one Effect): every one transitions to `Cancelled`; one `effect.cancelled` event is emitted per attempt, in dispatch order (§5.3's ordering guarantee).
- `execute_stop_decision` on an actor whose pending attempt also has a registered EWO-012 timeout timer: the timer is cancelled as a side effect of the reused `cancel_effect` call (via its own existing `cancel_correlated_timer_if_any`); no double-cancellation error.
- `execute_stop_decision` on an actor with an already-`Completed` attempt: left untouched; no error; no additional audit event beyond what completion already produced.
- `execute_stop_decision` on an actor with no dispatched Effects at all: unaffected, no panic, no spurious audit events.
- `execute_stop_decision` invoked a second time against an actor already Stopped (its Effects already `Cancelled` from the first call): the second call's own `cancel_effects_for_actor` finds every attempt already terminal and cancels nothing further; no error, no duplicate audit event.
- `execute_restart_decision` on an actor with a non-terminal Effect Attempt: the attempt is explicitly **not** cancelled — remains `Dispatched`/`Executing` after the restart completes, proving §5.7's disclosed exclusion is genuinely implemented as decided.
- `delete_actor_state` on an actor with a non-terminal Effect Attempt: the attempt transitions to `Cancelled`, and the resulting `effect.cancelled` audit event is confirmed to precede `persistence_deletion_completed_event` in the audit sequence — the direct verification of §5.6's ordering requirement.
- `delete_actor_state` on an actor with multiple Effects in a mix of terminal and non-terminal states: only the non-terminal ones are cancelled; terminal ones are left exactly as they were.
- `delete_actor_state` invoked a second time for the same, already-deleted actor: exercises the same pre-existing repeated-deletion path `persistence.delete` already defines (unmodified by this EWO); `cancel_effects_for_actor` finds every attempt already terminal from the first call and cancels nothing further, contributing no new failure mode.
- Direct `Runtime::terminate_actor_instance` (not via Supervisor `Stop`/`Restart`, not via `delete_actor_state`) on an actor with a non-terminal Effect Attempt: the attempt is **not** cancelled — confirming §5.7's disclosed limitation is the actual, observed behavior, not merely a stated intention.
- Cancelling actor A's effects (via any of the above triggers) never affects actor B's own pending Effect Attempts — the direct analogue of the existing `cancel_all_for_actor_only_touches_that_actors_own_pending_timers` Timer test.

**Late-signal race tests (mandatory — the most architecturally significant category, mirroring EWO-012's own treatment of the equivalent Timer-race tests):**
- A provider genuinely completes (`execute_message_capturing`'s success path) **after** its own attempt was already cancelled by `execute_stop_decision`: the late completion is discarded by the existing, unmodified `Terminal(_)` guard; the attempt's own final status remains `Cancelled`, never overwritten to `Completed`; `effect.late_signal_discarded` (not `effect.completed`) is the resulting audit fact.
- A provider genuinely fails (`execute_message_capturing`'s failure path) **after** its own attempt was already cancelled by `execute_stop_decision`: identical assertion — status remains `Cancelled`, `effect.late_signal_discarded` is emitted, never `effect.failed`.
- The identical pair of tests, repeated against `delete_actor_state`-triggered cancellation in place of `execute_stop_decision`-triggered cancellation, confirming the same existing guard behaves identically regardless of which of this EWO's two new triggers produced the `Cancelled` outcome.

**Regression tests:**
- Full existing EWO-001/EWO-002/EWO-012 test suite passing unmodified in behavior — this EWO adds calls at two existing integration points and reads existing state; it changes no existing method's own signature or behavior.
- Existing `execute_stop_decision`/`execute_restart_decision`/`delete_actor_state` tests continue to pass with their existing assertions intact, extended only where this EWO's own new tests (above) are additive.

**Negative tests:**
- `attempts_for_actor` on a fabricated/unknown `ActorId` returns an empty `Vec`, not a panic or error.
- `cancel_effects_for_actor` (exercised indirectly via `execute_stop_decision`/`delete_actor_state`) on an actor with a mix of many terminal and few non-terminal attempts touches only the non-terminal ones, verified by asserting the exact count of `effect.cancelled` events emitted equals the exact count of non-terminal attempts, not the total.

## 7. Explicit Exclusions

This EWO MUST NOT:

- introduce a new lifecycle state, terminal outcome, or audit event type — `Cancelled` and `effect.cancelled` (both pre-existing) are reused exactly as they are;
- wire `cancel_effects_for_actor` (or any equivalent) into `execute_restart_decision` — restart-preserves-pending-Effects is this EWO's own disclosed, precedented design decision (§5.7), not left open for silent future reinterpretation without its own review;
- wire `cancel_effects_for_actor` into the public `Runtime::terminate_actor_instance` — doing so would require either a new `Runtime`-owned `ActorInstanceId → ActorId` reverse-index field or a new Actor Host public method, either of which is a larger change than this milestone authorizes; that remains a disclosed, narrow, separately-decidable limitation (§5.7), on the same basis the identical Timer limitation is already accepted;
- implement retry scheduling, retry-decision logic, Idempotency Metadata, Provider business/operation failure signalling, or Resource Governance (each its own, separate, future milestone per the Effect Runtime Roadmap; none is a prerequisite for this milestone, and none is touched by it);
- add a new field to `EffectRequest`, `AttemptRecord`, `EffectRecord`, or `AuditEvent`;
- change `cancel_effect`'s, `cancel_correlated_timer_if_any`'s, `effect_requester`'s, or `process_due_timers`'s own existing internal logic in any way — each is reused exactly as published by EWO-002/EWO-012;
- add any eviction, expiry, or size-bounding mechanism to `by_actor` — consistent with the fact that `effects`, `attempts`, `by_message`, and `by_timer` are never evicted either;
- change the Actor Host, Timer Service, Capability Authority, Persistence Service, Supervisor, or Scheduler in any way;
- perform any opportunistic refactor, rename, or reformatting outside this milestone's own stated scope.

## 8. Validation Requirements

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace
cargo test --workspace
```

All four MUST pass cleanly, from a forced-clean state, before this milestone is considered complete. The final test count MUST be independently re-derived by direct summation, not asserted from a single aggregate figure, consistent with this engineering effort's own established validation discipline.

## 9. Reporting Requirements

An Engineering Report SHALL be authored recording: objective; implementation summary; every file modified; validation performed (all four gates, with actual, independently-derived test counts); test coverage mapped against §6; architectural conformance against §3, including the disclosed Restart-exclusion interpretation (§1, §5.7) as an explicit, factual part of engineering history, not minimized or concealed; the two disclosed, narrow, unwired limitations (`execute_restart_decision`, `Runtime::terminate_actor_instance`) stated plainly as accepted scope boundaries, not defects; deviations from this EWO, if any; and recommendations. Its own document identifier SHALL be derived from repository evidence at authoring time (STD-001 §7), not assumed in advance.

## 10. Definition of Done

- Both items in §3 (requesting-actor termination via Stop; durable deletion) are implemented and independently verifiable against source.
- `execute_restart_decision` and `Runtime::terminate_actor_instance` remain unmodified by this EWO, exactly as §5.7 specifies.
- No new lifecycle state, terminal outcome, or audit event type was introduced.
- Every test in §6 exists and passes.
- All four validation gates in §8 pass cleanly.
- No item listed in §7 (Explicit Exclusions) was implemented.
- An Independent Implementation Review has been conducted and has not identified a BLOCKER or unresolved MAJOR finding.

## Critical Safety Rule (restated for the implementer)

ARCH-008 is Approved and frozen. Implementation MUST conform to it, never silently reinterpret it. This EWO's own disclosed interpretation of the Restart-vs-Stop question (§1, §3, §5.7) is itself a load-bearing design decision specific to this document — an implementer MUST NOT revisit or reverse it without a fresh review, and MUST NOT extend cancellation to `execute_restart_decision` or `terminate_actor_instance` merely because doing so would appear more "complete." If any behavior this EWO requires proves genuinely underspecified or contradictory once implementation begins, stop and report the exact gap rather than inventing semantics — the same discipline this engineering effort has already applied consistently across EWO-001, EWO-002, and EWO-012.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Initial Draft. Authored from the preceding Milestone Architecture Readiness Review, ARCH-008 v0.3.2 (Approved) §21, and direct inspection of the published EWO-012 implementation. Specifies: a new `attempts_for_actor` Effect Coordinator lookup (`by_actor` reverse index, mirroring `by_message`/`by_timer`'s own shape); a new private Runtime helper `cancel_effects_for_actor` reusing the existing, unmodified `cancel_effect` for every non-terminal attempt found; integration at `execute_stop_decision` and `delete_actor_state` (the latter strictly before `persistence.delete`, per ARCH-007 §17's ordering requirement); and an explicitly disclosed decision not to integrate at `execute_restart_decision` (pending Effects survive Restart, mirroring the already-published Timer precedent) or at the public `Runtime::terminate_actor_instance` (no `ActorId` reverse lookup exists from an `ActorInstanceId`, the identical already-accepted limitation already disclosed for Timer cancellation). No new lifecycle state, terminal outcome, or audit event is introduced. |
| 0.2.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Corrective revision following an Independent Review of v0.1.0, which concluded `EWO-013 REQUIRES CORRECTION`: the review found an internal contradiction in §5.3 (the `by_actor` insertion instruction referenced `record.actor` at a point in `record_dispatched` before that binding exists), unspecified failure semantics for `cancel_effects_for_actor` on a mid-loop `cancel_effect` error, and a missing mandatory test category (late provider signals racing an automatic cancellation). Corrects each exactly as identified: §5.3 now places the `by_actor` push immediately after the existing `record.current_attempt = Some(...)` line, where `record.actor` is genuinely in scope, and states the resulting dispatch-order iteration guarantee explicitly; §5.4 adds an explicit failure-semantics paragraph (stops at first error, mirrors the pre-existing Timer-cancellation-loop shape in the same two functions, not a new failure model); §6 adds the two mandatory late-signal-race tests (success and failure, against both triggers) plus the review's requested minor coverage (Stop with multiple outstanding Effects, repeated Stop, repeated durable deletion). §1 and §3 additionally cite ARCH-004 §12 directly (capability bindings/supervision relationships already, explicitly surviving Restart) alongside the existing Timer-code precedent. No architecture, scope, or design decision reopened: the Restart-exclusion interpretation, the reuse of `Cancelled`/`cancel_effect` unmodified, and both integration points (§5.5, §5.6) are unchanged from v0.1.0. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-27 |
| Technical Review | TBD | Pending | |
| Approval Authority | TBD | Pending | |
