---
document_id: EWO-012
title: "Effect Timeout Integration"
version: 0.2.1
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
    - ARCH-005 (Temporal Runtime Architecture) — governs the Timer Service this EWO reuses unmodified
    - ARCH-002 (Runtime Architecture) — governs the admission pipeline and audit-event model this EWO reuses unmodified
  adrs:
    - ADR-0015 (Approved — Audit Emitter Failure Semantics)
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  predecessor: "EWO-002 / ER-012 (Provider Actor Integration) — the milestone this EWO extends; its correlation-hook pattern (`by_message`/`attempt_for_message`, the `matches!(attempt_status, Terminal(_))` late-signal guard) is directly reused, not redesigned"
  reported_by: "The next sequentially available Engineering Report identifier per STD-001 §7 at the time this EWO is implemented — not assumed in advance"
  base_state:
    runtime_head: 2bcc81fc000780a92893e3ec21b29542bb906258
    docs_head: 4aa2f6bd5a5ffb401d1b987caa18d7de78336cea
---

# EWO-012 — Effect Timeout Integration

Registered per STD-001 §46 (Engineering Work Orders). This authorizes implementation work only. It does not itself constitute approval, and does not amend ARCH-008 or any other architecture, standards, or governance document.

**Numbering and Traceability Disclosure.** The informal engineering-effort label for this milestone is "EWO-003 — Effect Timeout Integration," per the Effect Runtime Roadmap that identified it as the next milestone. That label cannot be this document's real identifier: `work-orders/EWO-003-Message-Gateway.md` already exists, permanently assigned to an entirely unrelated, pre-existing milestone (predecessor `EWO-002-Actor-Host.md`, created 2026-07-12, governed by ARCH-001/ARCH-002, with no relationship to the Effect Runtime whatsoever) — the identical situation ER-011 and ER-012 already disclosed for their own numbering. The highest existing Engineering Work Order on record is `EWO-011-Local-Actor-Supervision-Hardening.md`; no EWO-012 exists yet. This document is therefore **EWO-012**, derived directly from the repository's own contents (STD-001 §7), not from the informal label. This disclosure is made explicitly, not concealed, consistent with this engineering effort's own established practice.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-012 |
| Title | Effect Timeout Integration |
| Version | 0.2.1 |
| Status | Draft |
| Author | Denver Jacobs |
| Created | 2026-07-27 |
| Predecessor milestone | EWO-002 / ER-012 — Provider Actor Integration |
| Governing architecture | ARCH-008 v0.3.2 (Approved) |

---

## 1. Purpose

Implement ARCH-008 §20 (Timeout Architecture): wire the Effect Coordinator's existing, isolated `record_timed_out` primitive (present, unit-tested, and entirely unwired since ER-011) to a genuine per-Attempt Timer Service registration, so that an Effect Attempt which runs longer than its own caller-supplied timeout is truthfully driven to `TimedOut` rather than remaining `Dispatched`/`Executing` indefinitely — closing one of the two remaining unimplemented Attempt-level terminal outcomes ARCH-008 §16.2 names (the other, `RetryScheduled`'s own re-dispatch loop, is out of scope; see §7, Exclusions).

This EWO does not redesign ARCH-008, the Timer Service, or the Effect Coordinator's existing lifecycle model. It extends EWO-002's own established correlation-hook pattern (message-based correlation, late-signal discard via `matches!(attempt_status, Terminal(_))`) to a second correlation axis (timer-based), reusing the identical discipline.

**Revision note (v0.2.0).** An Independent API Design Review of v0.1.0's proposed public API (a separate `request_effect_with_timeout` method alongside an unchanged `request_effect`) concluded `EWO-012 API DESIGN REQUIRES REVISION`, on the grounds that ARCH-008 itself already names, in currently-Approved text, a second caller-supplied per-request policy axis due in the near future (§19, retry policy — the Effect Runtime Roadmap's own next-but-one milestone), making a one-method-per-axis pattern an evidenced, not speculative, long-term liability. This revision replaces that proposal with a single, extensible request parameter (§5.4), consistent with this codebase's own already-established convention (`Message`'s reserved-field shape; `submit_message`'s object-plus-authorization signature). No ARCH-008 requirement changes as a result — this is a pure API-shape (implementation-design) revision, squarely within this EWO's own authority (ARCH-008 §4 excludes Rust API shape from its own scope).

**Correction note (v0.2.1).** A Final Review of v0.2.0 concluded `EWO-012 REQUIRES FINAL CORRECTION`, identifying concrete specification defects in §5.5/§5.6 (an undefined variable, a referenced-but-undefined audit helper with an incorrect cross-reference, and a helper parameter that would fail the mandatory clippy gate), one stale source-line citation (§4), and an undisclosed-but-inherited interpretation of ARCH-008 §20's reply-propagation requirement. This revision corrects each defect exactly as identified — see §5.5, §5.6, §4, and the new disclosure below — reopening no architecture, no API-shape decision, and no scope beyond what the review itself identified.

## 2. Repository Baseline

- `synapse-runtime` @ `2bcc81fc000780a92893e3ec21b29542bb906258` (published; EWO-001/ER-011 Effect Runtime Foundation + EWO-002/ER-012 Provider Actor Integration).
- `synapse-docs` @ `4aa2f6bd5a5ffb401d1b987caa18d7de78336cea` (published; ARCH-008 v0.3.2 Approved, ER-011, ER-012).

An implementer MUST re-verify both repositories are still at these exact commits (or a documented, reconciled later state) before beginning implementation, per this engineering effort's own established repository-verification discipline.

## 3. Architectural Scope — Requirements Extracted from ARCH-008, with Evidence

| Requirement | ARCH-008 citation |
|---|---|
| Timeout policy (the numeric duration) is owned by the requesting actor or a capability-declared constraint — never Runtime, never the Effect Coordinator | §20, ownership table, row 1 |
| Timeout scheduling reuses the existing Timer Service directly, per attempt — no new mechanism | §20, ownership table, row 2; §15 ("timeout applicable to this attempt") |
| Timeout enforcement: upon the scheduled timer firing, the Effect Coordinator transitions the specific attempt to `TimedOut` if it has not already reached a terminal state | §20, ownership table, row 3 |
| Timeout cancellation: if an attempt completes before its own timeout fires, the Effect Coordinator cancels the corresponding timer registration for that attempt | §20, ownership table, row 4 |
| Timeout audit: `effect.timeout`, attributed to the specific Effect Attempt ID | §20, ownership table, row 5; §17 (mandatory, attempt-level) |
| Timeout propagation: the requesting actor receives an ordinary reply message carrying the truthful `TimedOut` outcome — never a silent drop | §20, ownership table, row 6 |
| The Timer Service exposes two cancellation operations with different failure behavior; a race between an attempt's own completion and its timeout independently firing is expected and MUST be treated as harmless — resolved by discarding whichever signal loses the race, never treating the cancellation attempt itself as an error | §20, "Timer cancellation and the completion/firing race" |
| A late provider completion after timeout MUST NOT overwrite the timeout terminal state; the Effect Coordinator MUST discard it and MUST truthfully audit that a late result was received and discarded | §20, "A late provider completion after timeout..." |
| `TimedOut` means Runtime stopped waiting under the applicable timeout; it does not, by itself, prove the provider or external system actually stopped | §20, "External outcome uncertainty" |
| `TimedOut` is retry-eligible (leaves the owning Effect `InProgress`, no auto-accept) | §19; §16.2 terminal-outcomes list |
| A given Effect Attempt reaches at most one terminal outcome; a late signal (including a stale timeout) is evaluated and, if applicable, discarded against the specific Attempt it is attributed to, never permitted to overwrite a different outcome | §16.2; §31 invariants 8–9 |
| Every Effect dispatch, including the reuse of Timer Service scheduling, undergoes no second admission or authorization pipeline | §13; §31 invariant 18 |
| In-flight Effects (and, by direct extension, their own pending timer registrations) are Runtime execution state, never actor domain state — MUST NOT be included in actor checkpoints, MUST NOT be automatically recreated on restore | §22 |
| No numeric timeout-duration default is fixed by architecture | §4, §30 (Explicit Non-Goals) |

No behavior beyond this table is inferred. Where ARCH-008 is silent on a mechanism (e.g., the exact concrete audit event name for a discarded late signal), §17 itself explicitly defers the concrete identifier to implementation while still requiring the underlying fact be truthfully, distinctly auditable — this EWO fixes a concrete name in §5 below, consistent with that deferral, not in place of it.

## 4. Existing Implementation Analysis — Components to Be Reused

- **Timer Service (`synapse-timer`)** — `Timer` trait / `TimerHandle`: `register`, `cancel`, `cancel_all_for_actor`, `query_due`, `mark_completed`, `mark_discarded`, `mark_expired`. Entirely unmodified. Already proven deterministic (caller-supplied `Instant`, no real-time wait) and already reused unmodified by Persistent Actor timers (ARCH-005).
- **`Runtime::register_timer`/`cancel_timer`** (`runtime/src/lib.rs`) — the existing, narrow public delegation pattern to `Timer::register`/`cancel`, with truthful `timer.registered`/`timer.cancelled`/`timer.expired` audit emission. Reused directly for the underlying timer registration/cancellation mechanics; no new Timer Service wrapper is introduced.
- **`Runtime::process_due_timers`** (`runtime/src/lib.rs:1839`) — the existing, caller-driven (`Instant`-parameterized) function that queries `Timer::query_due` and, for every due entry, currently constructs and admits an ordinary `Message` to the timer's own owning actor. **This function requires one, minimal, additive modification** (§5 below) — it is the only possible integration point, because `Timer::query_due` is a one-shot, destructive query: a due timer is returned exactly once, to whichever caller asks first (confirmed directly by the existing test `querying_due_twice_only_reports_each_timer_once`). A second, parallel polling function cannot coexist with it against the same `Timer` instance.
- **Effect Coordinator (`synapse-effect-coordinator`)** — `record_timed_out` (present since ER-011, calls the shared `record_terminal` helper, fully unit-tested in isolation, never called by Runtime) is reused **entirely unchanged**. The `by_message`/`attempt_for_message` correlation pattern (EWO-002) is the direct design precedent for the new timer-based correlation this EWO adds (§5).
- **`execute_message_capturing`'s existing late-signal guards** (`matches!(attempt_status, Some(AttemptStatus::Terminal(_)))`, EWO-002) — require **no modification**. `TimedOut` is already one of the `AttemptOutcome::Terminal` variants those guards match against; a late provider completion arriving after a `TimedOut` attempt is already correctly discarded by the existing, unmodified guard. This is confirmed directly from source, not assumed.
- **`delete_actor_state`'s existing pending-timer-cancellation ordering** (ARCH-007 §17 pattern, already reused by ARCH-005) — the direct precedent for "cancel a pending registration before reporting a terminal outcome complete," reused conceptually for the new per-attempt cancel-on-early-termination step (§5).
- **`Runtime::effect_requester`** (`runtime/src/lib.rs` — a private method; a specific line number is not cited here since it is fragile against ordinary source movement, per this correction's own finding that the previously cited line 2977 was already stale and actually fell inside `fail_effect`'s own body) — the existing shared private helper every outcome method (`complete_effect`/`fail_effect`/`cancel_effect`/`provider_lost_effect`) already uses to resolve the requesting `ActorId`. Reused unchanged by the new `timeout_effect` method, and newly reused by the late-signal-discard audit call (§5.5, §5.6) to resolve the actor at each of the three now-guarded points.
- **`Message`'s own reserved-field convention and `submit_message`'s own signature shape** (`common/src/lib.rs`, `runtime/src/lib.rs:1224`) — `submit_message(&mut self, message: Message, presented: &Capability)` already establishes the exact precedent this EWO's revised API design (§5.4) follows: request data bundled into one object, authorization proof kept as a separate, trailing parameter. `Message` itself already carries fields whose "internal representation is deferred" (`delivery_constraints`, `durability_classification`, `information_classification`) — direct, already-shipped, already-approved evidence that this codebase's own convention is to shape a request as an extensible struct rather than grow a family of similarly-named methods.

No reusable pattern from EWO-001/EWO-002 is bypassed; every new piece of state and every new audit event mirrors an already-established shape.

## 5. Implementation Design

### 5.1 Implementation Objectives

1. Allow a caller to optionally request a timeout when requesting an Effect.
2. Register exactly one Timer Service timer per Effect Attempt that requested one, at dispatch time.
3. Correlate a fired timer back to its Effect Attempt, exactly as EWO-002 correlates a dispatched message back to its Effect Attempt.
4. Truthfully transition a still-non-terminal attempt to `TimedOut` when its own timer genuinely fires; discard (never error, never re-apply) a fired timer whose attempt already reached a different terminal outcome.
5. Cancel a still-pending timeout timer whenever its own attempt reaches any terminal outcome first, treating "the timer had already fired" as an expected, harmless race, never an error.
6. Truthfully audit every new fact this introduces, including the previously-undisclosed late-signal-discard audit gap this EWO also closes (§5.6).

### 5.2 Components Affected

- `services/effect-coordinator/src/lib.rs` / `internal.rs` — new correlation state and three new trait methods (§5.3). `record_timed_out` itself is unchanged.
- `runtime/src/lib.rs` — a new public `EffectRequest` struct (§5.4); `request_effect`'s existing signature **is changed** to accept it (§5.4 — see Migration Impact below); one new outcome method (`timeout_effect`), one new shared private helper (`cancel_correlated_timer_if_any`), one minimal, additive branch inside the existing `process_due_timers` loop, two new audit-event helper functions — `effect_timeout_event` (defined in §5.5, alongside `timeout_effect`) and `effect_late_signal_discarded_event` (defined in §5.6) — and calls to the new shared helper from the four existing outcome methods.
- No change to `record_dispatched`, the Timer Service crate itself, or any Persistence, Supervisor, or Capability Authority code.

**Migration impact, disclosed explicitly.** Every existing call site of `request_effect` in the EWO-001/EWO-002 test suite (`runtime/src/lib.rs`) must be updated to construct an `EffectRequest` in place of the three positional arguments it replaces (`provider`, `operation`, `payload`). This is a one-time, compiler-checked, internal-only migration — every affected call site is this same codebase's own test code; no external consumer exists yet to break. This is a deliberate revision of v0.1.0's own stated goal of zero call-site churn: the Independent API Design Review concluded that goal was the wrong criterion to optimize for, given ARCH-008's own already-approved text anticipates a second caller-supplied per-request axis (retry policy, §19) in the near future. Paying this migration cost once, now, while the call-site surface is still small, is judged preferable to either repeating it later against a larger surface or compounding a telescoping-method antipattern.

### 5.3 Effect Coordinator Changes

Add, to the `EffectCoordinator` trait and `EffectCoordinatorImpl`/`EffectCoordinatorHandle` (mirroring `by_message`/`attempt_for_message`'s own exact shape):

```text
fn record_timeout_registered(&mut self, attempt: &EffectAttemptId, timer: TimerId) -> Result<(), RuntimeError>;
fn attempt_for_timer(&self, timer: &TimerId) -> Option<EffectAttemptId>;
fn timer_for_attempt(&self, attempt: &EffectAttemptId) -> Option<TimerId>;
```

Internal state: two new maps on `EffectCoordinatorImpl` — `by_timer: HashMap<TimerId, EffectAttemptId>` (fired-timer → attempt, the reverse-lookup direction `process_due_timers` needs) and `attempt_timer: HashMap<EffectAttemptId, TimerId>` (attempt → its own pending timer, the direction the four existing outcome methods need for early cancellation). Both directions are required — unlike `by_message` (which only ever needs message → attempt), timeout correlation is genuinely bidirectional. `record_timeout_registered` populates both maps atomically; population is unconditional (an attempt either has a registered timer or it does not — no partial state).

`record_timed_out` itself requires **no change** — it already exists, already calls the shared `record_terminal` helper, and is already unit-tested in isolation since ER-011.

**New, disclosed dependency**: `synapse-effect-coordinator`'s `Cargo.toml` gains a dependency on `synapse-timer`, solely to reference the plain `TimerId` newtype (`pub struct TimerId(pub String)`, no behavior) as a correlation key — directly analogous to its existing dependency on `synapse-common` for the identical purpose with `MessageId`. `synapse-timer` itself depends only on `synapse-common`, so this introduces no transitive dependency beyond what the Effect Coordinator already shares, and does not cross ARCH-008 §10's "depends on nothing but identity/state" boundary (`TimerId` is pure identity).

### 5.4 Runtime API — Revised Request Shape and Timer Registration Flow

**This section supersedes v0.1.0's own `request_effect_with_timeout` proposal entirely**, per the Independent API Design Review (`EWO-012 API DESIGN REQUIRES REVISION`).

Add a new public struct, `EffectRequest`, defined directly in `runtime/src/lib.rs` (it is a parameter-bundle for `Runtime::request_effect` alone; nothing else needs it, so it does not belong in `synapse-common` alongside cross-crate primitives like `Message`):

```text
pub struct EffectRequest {
    pub provider: ActorId,
    pub operation: String,
    pub payload: Vec<u8>,
    pub timeout: Option<Duration>,
}
```

Containing **only** the four fields this EWO itself requires — `provider`/`operation`/`payload` (already existing `request_effect` parameters, moved onto the struct) and `timeout: Option<Duration>` (new; `None` means no timeout is enforced for this attempt, consistent with ARCH-008 §20's own framing that a timeout is not mandatory for every Effect). No retry-policy, idempotency, priority, or other future field is added — the review's own recommendation was to secure the struct's *extensibility*, not to pre-populate its *content*; a future milestone (Retry Architecture, per the Effect Runtime Roadmap) adds its own field to this struct when, and only when, it is actually authorized to do so.

`request_effect`'s existing signature **changes** to accept it, mirroring `submit_message(message: Message, presented: &Capability)`'s own already-established shape exactly — request data in one object, authorization proof as a separate, trailing parameter:

```text
pub fn request_effect(
    &mut self,
    actor: &ActorId,
    request: EffectRequest,
    authorization: &Capability,
) -> Result<EffectId, RuntimeError>
```

Implementation: dispatch exactly as v0.1.0's own `request_effect` already does (unchanged internal logic — admission, `EffectId` minting, `Denied`/`InProgress` transition), reading `provider`/`operation`/`payload` from `request` instead of separate parameters. If `request.timeout` is `Some(duration)` **and** dispatch succeeds, resolve the attempt via `current_attempt_of`, register a timer via the existing `self.timer.register(actor.clone(), now + duration, "effect.timeout".into(), Vec::new())` (owning `ActorId` is the **requesting** actor — consistent with every existing Effect audit attribution, never the provider), then call `record_timeout_registered`. If `request.timeout` is `None`, no timer is registered at all — the no-timeout path is unaffected by this milestone in every observable respect. On dispatch failure (`Denied` or admission rejection), no timer is ever registered regardless of `request.timeout` — mirroring `Denied`'s own existing "no Attempt, no orphaned state" guarantee exactly.

The `message_type` string (`"effect.timeout"`) is never inspected by anything — correlation is entirely via `TimerId`, not string matching — recorded here only because the `Timer` trait's own `register` signature requires one; it will never reach `process_due_timers`'s ordinary message-construction path (§5.5) for a correlated timer.

### 5.5 Timeout Processing Flow — the One Modification to `process_due_timers`

Inside the existing per-due-entry loop, immediately after the existing, unconditional `timer_fired_event` emission and before the existing message-construction step, insert:

```text
if let Some(attempt) = self.effect_coordinator.attempt_for_timer(&entry.id) {
    if !matches!(
        self.effect_coordinator.attempt_status(&attempt),
        Some(AttemptStatus::Terminal(_))
    ) {
        self.timeout_effect(&attempt)?;
    } else {
        // The requesting actor is not otherwise in scope here —
        // resolved explicitly via the existing shared helper, exactly
        // as every outcome method already does, never left implicit.
        let actor = self.effect_requester(&attempt)?;
        self.core.audit_emitter.emit(effect_late_signal_discarded_event(&actor))?; // see §5.6
    }
    self.timer.mark_completed(&entry.id)?; // the underlying Timer Service registration is genuinely, truthfully resolved either way
    continue; // an Effect-correlated timer never constructs or admits an ordinary actor message
}
```

An Effect-correlated due entry never reaches the existing message-construction/`admit_message` logic — it has no actor-visible message of its own, exactly as `provider_lost_effect`/`complete_effect` already have none. `mark_completed` (never `mark_discarded`) is used unconditionally here, since "discarded" specifically means "the resulting message failed ordinary admission" (§20's own Timer Service semantics), which does not apply — the underlying Timer Service's own job (firing truthfully, at the right time) is accomplished whether or not the correlated Effect Attempt was already terminal.

New method, mirroring `provider_lost_effect`'s exact shape, and its accompanying audit-event helper, mirroring `effect_provider_lost_event`'s exact shape:

```text
pub fn timeout_effect(&mut self, attempt: &EffectAttemptId) -> Result<(), RuntimeError> {
    let actor = self.effect_requester(attempt)?;
    self.effect_coordinator.record_timed_out(attempt)?;
    self.core.audit_emitter.emit(effect_timeout_event(&actor))
}

fn effect_timeout_event(actor: &ActorId) -> AuditEvent {
    AuditEvent {
        event_type: "effect.timeout".into(),
        actor: Some(actor.clone()),
        capability: None,
        message: None,
    }
}
```

**Disclosed interpretation of ARCH-008 §20's reply-propagation requirement.** ARCH-008 §20's own "Timeout propagation" row states that "the requesting actor receives an ordinary reply message carrying the truthful `TimedOut` outcome — never a silent drop." Directly confirmed against the published source: none of the existing outcome methods (`complete_effect`, `fail_effect`, `cancel_effect`, `provider_lost_effect`) construct or admit any message to the requester's own mailbox — each already, only, updates Effect Coordinator state and emits an audit event; this has already passed through the full EWO-001/EWO-002 review sequence (Independent Implementation Review, Correction, Re-review, Engineering Report Review) without being raised as a finding. `timeout_effect`, above, follows this identical, already-published, already-reviewed pattern — it is not a new interpretation this milestone invents. Within the current implementation baseline: the timeout outcome is recorded in queryable Effect Coordinator state (`effect_status`/`attempt_status`), truthfully audited (`effect.timeout`), and observable by any caller that queries or consumes the audit stream — but no additional mailbox-reply-delivery mechanism exists for any outcome, timeout included. Building one is a distinct, separate concern from wiring the Timer Service to `record_timed_out`, and is explicitly outside this milestone's own scope (§7) — changing the broader reply-delivery interpretation, if ever warranted, is a decision for a future, separately authorized milestone, not something this EWO introduces or resolves.

### 5.6 Cancellation Behaviour — and Closing an Existing, Disclosed Audit Gap

Add one new shared private helper, called from `complete_effect`, `fail_effect`, `cancel_effect`, and `provider_lost_effect` (each, immediately after successfully recording its own terminal outcome) and from the new `timeout_effect` (for symmetry, even though a just-timed-out attempt cancelling its own already-fired timer is itself the harmless race case below):

```text
fn cancel_correlated_timer_if_any(&mut self, attempt: &EffectAttemptId) {
    if let Some(timer) = self.effect_coordinator.timer_for_attempt(attempt) {
        // `Err(IllegalTransition)` here means the timer had already
        // independently fired — the exact harmless race ARCH-008 §20
        // names explicitly. Never propagated as an error; never
        // escalated. `Err(UnknownTarget)` is structurally unreachable
        // (the timer's own existence is guaranteed by `record_
        // timeout_registered` having been called for this same
        // attempt) and is likewise discarded defensively rather than
        // panicking.
        let _ = self.timer.cancel(&timer);
    }
}
```

**Disclosed correction to EWO-002's own existing guards, in scope for this milestone.** ARCH-008 §16.2 requires generally — not only for the timeout case — that a discarded late signal "MUST be discarded and truthfully audited as discarded, never silently applied and never silently dropped without record." EWO-002's own `execute_message_capturing` guards (the success-path and failure-path hooks) currently discard a late signal for an already-terminal attempt **silently**, with no audit event at all — confirmed directly from the published source; this is a genuine, if narrow, gap relative to §16.2's own already-approved text, not previously identified as its own finding. Since this milestone touches the identical guard shape a third time (§5.5) and must add the discard-audit event there regardless, closing the same gap at the two existing EWO-002 call sites in the same pass is the minimal, consistent choice — leaving one guard audited and two silent would be a worse, inconsistent end state. Add:

```text
fn effect_late_signal_discarded_event(actor: &ActorId) -> AuditEvent {
    AuditEvent {
        event_type: "effect.late_signal_discarded".into(),
        actor: Some(actor.clone()),
        capability: None,
        message: None,
    }
}
```

No `reason` parameter is included: `AuditEvent`'s own shape, reused unmodified per ARCH-008 §17, carries no freeform payload field — only `event_type`, `actor`, `capability`, `message` exist — so a parameter nothing in the function body could ever use would be dead on arrival, and would itself cause `cargo clippy --workspace --all-targets --all-features -- -D warnings` (§8) to fail on an unused-parameter lint. `event_type: "effect.late_signal_discarded"` alone already truthfully distinguishes this fact from every other Effect audit event; do not add a new field to `AuditEvent` to carry a discard reason — that would be a new architectural decision this EWO does not authorize.

Emit `effect_late_signal_discarded_event(&actor)` at all three now-guarded points (the two existing EWO-002 hooks in `execute_message_capturing`, plus the new §5.5 timeout hook), each resolving `actor` via `self.effect_requester(&attempt)?` first exactly as §5.5 now shows explicitly, whenever the `matches!(..., Terminal(_))` check finds the attempt already terminal, in place of doing nothing.

### 5.7 Error Handling

- `request_effect`'s own failure modes are unchanged by this revision (`IllegalTransition` if Runtime is not `Running`; ordinary admission/denial errors) — a timer is registered if, and only if, both `request.timeout` is `Some` and dispatch itself succeeds.
- `timeout_effect` returns `RuntimeError::UnknownTarget` if the attempt does not exist (mirrors `complete_effect`/`fail_effect`/`provider_lost_effect` exactly) and propagates `RuntimeError::IllegalTransition` if it has already reached a terminal outcome — structurally unreachable given §5.5's own guard, but the underlying `record_timed_out` call retains this protection regardless, exactly as every other outcome method already does.
- `cancel_correlated_timer_if_any` never returns an error and never panics — an already-fired timer's `IllegalTransition` and (structurally unreachable) `UnknownTarget` are both discarded, per §5.6.

### 5.8 Determinism

No new non-determinism is introduced. Timer registration continues to use the existing, caller-supplied `Instant` model (`ARCH-005 §19`, `synapse-timer`'s own established precedent) — no real-clock read, no real sleep, anywhere in this milestone's own code. `process_due_timers` remains entirely caller-driven; the new branch inside it introduces no new timing dependency beyond the `now: Instant` already supplied by its existing caller.

## 6. Testing Strategy

Every behavior in §3/§5 maps to at least one test. No test may rely on real elapsed time — all timing uses synthetic `Instant` arithmetic, per §5.8 and this project's own established Timer Service test precedent.

**Unit tests (Effect Coordinator, `services/effect-coordinator`):**
- `record_timeout_registered` populates both `by_timer` and `attempt_timer`, queryable in both directions.
- `attempt_for_timer`/`timer_for_attempt` return `None` for an unrelated `TimerId`/`EffectAttemptId`.
- `record_timed_out` (unchanged, but re-confirm still correct): leaves the owning Effect `InProgress`; rejects a second terminal recording.

**Integration tests (Runtime, `runtime/src/lib.rs`):**
- A dispatched Effect Attempt requested with `EffectRequest.timeout = Some(duration)`, whose provider never responds, genuinely reaches `TimedOut` when `process_due_timers` is called at or after the registered `fire_at`.
- The requesting actor's own `effect.timeout` audit event is emitted, attributed to the requester, never the provider.
- An Effect Attempt requested with `EffectRequest.timeout = None` is entirely unaffected by `process_due_timers` — no timer was ever registered for it. This is the direct replacement for v0.1.0's own "`request_effect`, unchanged" test framing, now expressed as the `None` branch of the same, single `request_effect` entry point rather than a separate method.
- `request_effect`'s own denial path (capability mismatch, unreachable provider) registers no timer regardless of `EffectRequest.timeout` — no orphaned Timer Service registration.
- Every existing EWO-001/EWO-002 test that constructs an Effect request is updated to build an `EffectRequest` (with `timeout: None` unless the test specifically exercises timeout behavior) and continues to pass unmodified in every other respect — proving the migration is purely mechanical, not behavior-changing.

**Race-condition tests (the most architecturally important category, mirroring EWO-002's own most-important test):**
- A provider that genuinely completes **before** its own timeout fires: the timer is cancelled (`Timer::state_of` reports `Cancelled`, not left `Pending`); a subsequent `process_due_timers` call at or after the original `fire_at` does not, and cannot, transition the (already-`Completed`) attempt to `TimedOut`.
- A provider that genuinely completes **after** its own timeout has already fired (timeout wins the race): `TimedOut` is recorded first; the late provider completion is discarded by the existing, unmodified `execute_message_capturing` guard — confirmed by asserting the attempt's own final status remains `TimedOut`, never overwritten, and that `effect.late_signal_discarded` (not `effect.completed`) is the resulting audit fact.
- Cancelling a timer that has already independently fired (the harmless race §20 names explicitly): `cancel_correlated_timer_if_any` does not propagate an error and does not panic.
- An explicit `cancel_effect` call racing ahead of a still-pending timeout: the timer is cancelled; a later `process_due_timers` call at the original `fire_at` finds the attempt already `Cancelled` and emits `effect.late_signal_discarded`, never overwriting `Cancelled`.

**Regression tests:**
- Full existing EWO-001/EWO-002 test suite (255 `synapse-runtime`, 47 `synapse-effect-coordinator`, 650 workspace) passing after the mechanical `EffectRequest` migration (§5.2) — every call site's own *source* is updated to construct an `EffectRequest`, but every test's own *assertions and behavior* are unchanged; this distinction (source touched, behavior identical) MUST be explicit in the eventual Engineering Report, not conflated with "nothing changed."
- The two existing EWO-002 guard sites, now emitting `effect.late_signal_discarded` where they previously emitted nothing: re-confirm every existing late-signal test (`a_cancelled_effect_is_not_overwritten_by_a_late_successful_provider_result`, etc.) still asserts the correct terminal *status*; extend each to additionally assert the new audit event now accompanies the discard.

**Determinism tests:**
- A timer registered far in the future is not reported due, and its attempt is not affected, when `process_due_timers` is called with an earlier synthetic `Instant` — mirroring `a_timer_due_in_the_future_is_not_yet_reported_as_due` exactly.
- Two Effects with timeouts due at different synthetic instants resolve in `fire_at` order, never registration order — mirroring `multiple_due_timers_are_returned_in_ascending_fire_at_order`.

**Negative tests:**
- `timeout_effect` on a fabricated/unknown `EffectAttemptId` returns `UnknownTarget`.
- `attempt_for_timer`/`timer_for_attempt` on a fabricated id return `None`, not a panic.
- `request_effect` with `EffectRequest.timeout = Some(duration)` while Runtime is not `Running` fails with the identical, existing `IllegalTransition` — no timer is registered.

## 7. Explicit Exclusions

This EWO MUST NOT:

- implement retry scheduling, retry-decision logic, or any consumer of `record_retry_scheduled` (ARCH-008 §19 — a distinct, future milestone; Idempotency Metadata is a hard prerequisite for it, per the Effect Runtime Roadmap, and is not addressed here);
- fix, define, or default any numeric timeout duration — the `Duration` inside `EffectRequest.timeout` is always caller/capability-supplied (§4, §30 Explicit Non-Goals);
- change `request_effect`'s own internal admission/dispatch logic beyond reading `provider`/`operation`/`payload` from `EffectRequest` instead of separate parameters — the signature change itself (§5.4) is authorized by this revision, but nothing about *how* a request is admitted, authorized, or dispatched may change;
- add any field to `EffectRequest` beyond the four this EWO specifies (`provider`, `operation`, `payload`, `timeout`) — in particular, no retry-policy, idempotency, priority, or tracing field, however plausible a future placeholder might seem; a future milestone adds its own field when, and only when, it is separately authorized to do so;
- modify `record_dispatched`, the message-based (`by_message`) correlation mechanism, or any other EWO-002 behavior beyond the single, disclosed audit addition in §5.6;
- introduce Idempotency metadata, Effect Classification, Compensation, Resource Governance, or Provider business/operation failure signalling (each its own, separate, future milestone per the Effect Runtime Roadmap);
- change the Timer Service (`synapse-timer`) crate itself in any way;
- add a new field to the existing `AuditEvent` structure;
- build a new mailbox-reply-delivery mechanism for any Effect outcome, timeout included — `timeout_effect` follows the identical, already-published, already-reviewed pattern the four existing outcome methods already use (state update plus audit, no message construction); changing that broader interpretation of ARCH-008 §20's reply-propagation requirement (§5.5, disclosed) is a distinct, future, separately authorized concern;
- perform any opportunistic refactor, rename, or reformatting outside this milestone's own stated scope.

## 8. Validation Requirements

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace
cargo test --workspace
```

All four MUST pass cleanly, from a forced-clean state, before this milestone is considered complete. The baseline to compare against is 650 passed, 0 failed (this EWO's own base state, §2); the final count MUST be reported and independently re-derived by direct summation, not asserted from a single aggregate figure, consistent with this engineering effort's own established validation discipline.

## 9. Reporting Requirements

An Engineering Report SHALL be authored recording: objective; implementation summary; every file modified; validation performed (all four gates, with actual, independently-derived test counts); test coverage mapped against §6; architectural conformance against §3; the disclosed EWO-002 audit-gap correction (§5.6) as an explicit, factual part of engineering history, not a defect to be minimized or concealed; the `request_effect` signature change and the resulting call-site migration (§5.2, §5.4) as engineering history, including that it was a deliberate, reviewed revision of this EWO's own v0.1.0 design, not an oversight; deviations from this EWO, if any; and recommendations. Its own document identifier SHALL be derived from repository evidence at authoring time (STD-001 §7), not assumed to be "ER-003" merely because this milestone's informal label is "EWO-003."

## 10. Definition of Done

- All items in §3 are implemented and independently verifiable against source.
- The public API matches §5.4 exactly — a single `request_effect(actor, request: EffectRequest, authorization)` entry point; no `request_effect_with_timeout` or other second method was introduced.
- Every test in §6 exists and passes.
- All four validation gates in §8 pass cleanly.
- No item listed in §7 (Explicit Exclusions) was implemented.
- An Independent Implementation Review has been conducted and has not identified a BLOCKER or unresolved MAJOR finding.

## Critical Safety Rule (restated for the implementer)

ARCH-008 is Approved and frozen. Implementation MUST conform to it, never silently reinterpret it. If any behavior this EWO requires proves genuinely underspecified or contradictory once implementation begins, stop and report the exact gap rather than inventing semantics — the same discipline this engineering effort has already applied consistently across EWO-001 and EWO-002.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Initial Draft. Authored from the Effect Runtime Roadmap's own identification of Effect Timeout Integration as the next milestone, ARCH-008 v0.3.2 (Approved) §20, and direct inspection of the published EWO-001/EWO-002 implementation. Specifies: optional per-request timeout via a new `request_effect_with_timeout` method (existing `request_effect` unchanged); a new, bidirectional `TimerId ↔ EffectAttemptId` correlation in the Effect Coordinator (new, minimal, disclosed dependency on `synapse-timer` for the plain `TimerId` type); one additive branch inside the existing `process_due_timers`, required because `Timer::query_due` is a one-shot query no second consumer can share; reuse of `record_timed_out` (present, unwired, since ER-011) entirely unchanged; a new `timeout_effect` method mirroring `provider_lost_effect`'s exact shape; a shared timer-cancellation-on-early-termination helper treating an already-fired timer as the explicitly harmless race ARCH-008 §20 names; and a disclosed, in-scope correction adding the `effect.late_signal_discarded` audit event ARCH-008 §16.2 already requires generally, closing a narrow, previously unidentified silent-discard gap in EWO-002's own existing guards at the same time as adding the analogous new timeout guard. Discloses that "EWO-003" (the informal engineering-effort label) collides with the pre-existing, unrelated `EWO-003-Message-Gateway.md`, and that this document is therefore EWO-012. |
| 0.2.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Revision following an Independent API Design Review of v0.1.0's proposed public API, which concluded `EWO-012 API DESIGN REQUIRES REVISION`: the review found that a separate `request_effect_with_timeout` method alongside an unchanged `request_effect` is a telescoping-method antipattern that ARCH-008 §19/§20 (both already Approved) and the Effect Runtime Roadmap's own next-but-one milestone (Retry Architecture) already show will need a second caller-supplied per-request axis in the near future, and that this codebase's own established convention (`Message`'s reserved-field shape, `submit_message`'s object-plus-authorization signature) already favors an extensible request object over a method family. Replaces the v0.1.0 proposal with a new `EffectRequest` struct (`provider`, `operation`, `payload`, `timeout: Option<Duration>` — exactly these four fields, no speculative placeholder field for retry policy, idempotency, or anything else) and changes `request_effect`'s own signature to `request_effect(actor, request: EffectRequest, authorization)`, mirroring `submit_message` exactly. Discloses, rather than conceals, the resulting migration impact: every existing EWO-001/EWO-002 call site of `request_effect` requires a mechanical, compiler-checked update to construct an `EffectRequest`, reversing v0.1.0's own stated "zero call-site churn" design goal, which the review found to be the wrong criterion to optimize for. No ARCH-008 requirement, and no behavior of `request_effect`'s own internal admission/dispatch logic, changed as a result — this is a pure implementation-design (API-shape) revision, within this EWO's own authority per ARCH-008 §4. Updated: §1 (revision note), §4 (added the `Message`/`submit_message` precedent), §5.2 (Components Affected, Migration Impact), §5.4 (rewritten in full), §5.7, §6 (Testing Strategy call-site and negative-test wording), §7 (Explicit Exclusions — reversed the "no signature change" prohibition into a precisely scoped "no field beyond these four, no internal-logic change" constraint), §9 (Reporting Requirements), §10 (Definition of Done). |
| 0.2.1 | 2026-07-27 | Denver Jacobs (AI-assisted) | Corrective revision following a Final Review of v0.2.0, which concluded `EWO-012 REQUIRES FINAL CORRECTION`: the review found an undefined variable (`attempt_requester`) in §5.5's own code, a referenced-but-never-defined `effect_timeout_event` helper (with §5.2 incorrectly claiming both new audit helpers were defined in §5.6), an unused `reason` parameter on `effect_late_signal_discarded_event` that would fail the mandatory `cargo clippy -D warnings` gate (§8) exactly as shown, a stale source-line citation for `effect_requester` in §4 (cited at line 2977; actually at line 3029), and an undisclosed reliance on the already-published EWO-001/EWO-002 outcome-recording pattern to satisfy ARCH-008 §20's own reply-propagation requirement. Corrects each defect exactly as identified: §5.5 now resolves `actor` explicitly via `self.effect_requester(&attempt)?` before the discard-audit call, and explicitly defines `effect_timeout_event`; §5.6's `effect_late_signal_discarded_event` drops the unused `reason` parameter; §5.2 correctly attributes each of the two new audit helpers to its own actual section; §4's `effect_requester` citation is replaced with a stable symbol reference rather than a line number; §5.5 gains an explicit disclosure of the inherited reply-propagation interpretation, and §7 gains a matching exclusion against building a new reply-delivery mechanism. No architecture, no API shape (`EffectRequest`, `request_effect`'s signature), and no scope beyond what the review itself identified was reopened. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-27 |
| Technical Review | TBD | Pending | |
| Approval Authority | TBD | Pending | |
