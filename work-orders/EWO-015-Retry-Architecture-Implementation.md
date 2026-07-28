---
document_id: EWO-015
title: "Retry Architecture Implementation"
version: 0.1.1
status: Approved
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-28
last_updated: 2026-07-28
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-008 (v0.4.3, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md) — the sole architectural authority this EWO implements; specifically §19.1–§19.4 and invariant 45, added by the Retry Architecture Completion amendment; not amended by this EWO
    - ARCH-002 (Runtime Architecture) — governs the capability model (§8, §9) this EWO reuses unmodified
  adrs:
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  predecessor: "EWO-014 / ER-015 (Provider Idempotency Registration) — its `idempotency` map (Effect Coordinator field + trait query, populated at the point a fact is established, never evicted) is the direct precedent this EWO's own `operation`/retry-bookkeeping fields follow; EWO-012 (Effect Timeout Integration) — its `by_timer`/`attempt_timer` correlation-map pair and `process_due_timers` integration are the direct, reusable design precedent this EWO follows for the new `by_retry_timer`/`retry_timer` pair"
  reported_by: "The next sequentially available Engineering Report identifier per STD-001 §7 at the time this EWO is implemented — not assumed in advance"
  base_state:
    runtime_head: 4256b4434447fb9ab43d0d901a5baf8476c024e3
    docs_head: 2bd2595912f3ba1acf30d80684501c95bc4903fd
---

# EWO-015 — Retry Architecture Implementation

Registered per STD-001 §46 (Engineering Work Orders). This authorizes implementation work only. It does not itself constitute approval, and does not amend ARCH-008 or any other architecture, standards, or governance document.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-015 |
| Title | Retry Architecture Implementation |
| Version | 0.1.1 |
| Status | **Approved** |
| Author | Denver Jacobs |
| Created | 2026-07-28 |
| Predecessor milestone | EWO-014 / ER-015 — Provider Idempotency Registration |
| Governing architecture | ARCH-008 v0.4.3 (Approved), §19.1–§19.4, invariant 45 |

---

## 1. Purpose

ARCH-008 §19 established Retry Architecture's constitutional principles from the document's own initial approval — Retry Policy/Decision/Scheduling/Execution, policy-mechanism separation, the Effect Coordinator as sole decision-maker — but left several completions explicitly deferred. The v0.4.3 Retry Architecture Completion amendment (§19.1–§19.4) closed those normative gaps: retry eligibility (exactly `Failed`/`TimedOut`/`ProviderLost`), retry authority (an exhaustive, non-overlapping four-role partition), idempotency-class retry permission (an explicit three-row table), and retry-limit ownership (a capability-ceiling/actor-narrowing precedence model by direct analogy to the existing attenuation-only capability model). This EWO closes the remaining gap: it gives the Effect Coordinator and Runtime a real, concrete, tested mechanism realizing §19/§19.1–§19.4 exactly as written, introducing nothing those sections do not already authorize.

This EWO does not define retry policy numerics (maximum-attempt defaults, backoff, jitter) — those remain, per §19's own text, owned by the requesting actor or a capability constraint, never fixed by this or any engineering specification.

## 2. Repository Baseline

- `synapse-runtime` @ `4256b4434447fb9ab43d0d901a5baf8476c024e3` (published; Effect Runtime Foundation through Provider Idempotency Registration implementation).
- `synapse-docs` @ `2bd2595912f3ba1acf30d80684501c95bc4903fd` (published; ARCH-008 v0.4.3 Approved, ER-015).

An implementer MUST re-verify both repositories are still at these exact commits (or a documented, reconciled later state) before beginning implementation, per this engineering effort's own established repository-verification discipline.

## 3. Architectural Scope — Requirements Extracted from ARCH-008, with Evidence

| Requirement | ARCH-008 citation |
|---|---|
| Retry-eligible attempt-level outcomes are exactly `Failed`, `TimedOut`, `ProviderLost`; `Completed`/`Cancelled` are never retry-eligible; `Denied` is categorically excluded (Effect-level, no attempt ever existed) | §19.1 |
| Retry-related responsibility is exhaustively partitioned across Provider Actor, requesting actor, Runtime, and Effect Coordinator; none supplies another's input or performs another's decision | §19.2 |
| Retry intent is requesting-actor-supplied, expressed once at original request time, never re-solicited per attempt | §19.2; §9 (ownership table) |
| `Idempotent` → retry permitted, subject to eligibility and actor intent; `NonIdempotent`/`Unknown` → retry prohibited by default, permitted only with the actor's own explicit, distinct risk acceptance | §19.3; §23 (conservative `Unknown` default) |
| A capability-declared retry-limit constraint, where presented, is a hard ceiling; an actor-supplied limit may narrow, never widen it; absent either, no architecture-level bound exists | §19.4; §14 (attenuation-only analogy) |
| Retry Decisions are a pure function of Effect Coordinator tracked state and the §19.1–§19.4 inputs; never dependent on wall-clock time, process identity, or thread scheduling | Invariant 45 |
| Retry Scheduling reuses the existing Timer Service and Temporal Runtime directly; no Effect-specific timer subsystem | §19; invariant 32 |
| Retry Execution mints a fresh Effect Attempt ID, undergoes fresh capability validation never inherited from the original attempt, and may target a different Provider Actor destination if replaced | §19; §13; §14; §15; invariant 28 |
| Effect Coordinator is the sole Retry Decision-maker; bookkeeping only | §10; §19 |
| No new capability, capability operation string, or authorization step | §19.4; §23.5's identical "no new gate" reasoning |
| No new Effect lifecycle state, terminal outcome, or `AuditEvent` field-layout change | §16; §17 ("new `event_type` string values only, never a new field") |
| No numeric retry policy (maximum attempts, backoff, jitter) | §4; §19; §33 |

No behavior beyond this table is inferred. Where ARCH-008 is silent on a concrete representation specifically, this EWO does not invent architecture — it makes a disclosed, narrow implementation choice consistent with the table above (§5, §7).

## 4. Existing Implementation Analysis — Components to Be Reused

- **Effect Coordinator (`synapse-effect-coordinator`)** — `record_retry_scheduled` and the `RetryScheduled` state already exist as pure bookkeeping (confirmed at the current published baseline: enforces the `Requested`/`RetryScheduled` → `RetryScheduled` transition only, with no retry-decision mechanism wired to it). The `idempotency: HashMap<(ActorId, String), IdempotencyDeclaration>` map (EWO-014) is the direct precedent this EWO's own `operation` field and retry-bookkeeping fields follow — a new field, exposed via new trait methods, populated at the point the underlying fact is established, never evicted.
- **Timer correlation precedent (EWO-012)** — `by_timer: HashMap<TimerId, EffectAttemptId>` / `attempt_timer: HashMap<EffectAttemptId, TimerId>`, and their integration into `Runtime::process_due_timers` (`runtime/src/lib.rs`, confirmed: polls `self.timer.query_due(now)`, resolves each due `TimerId` via `attempt_for_timer`, and calls `timeout_effect`) are the direct, reusable design precedent this EWO follows for a new `EffectId`-keyed pair, `retry_timer`/`by_retry_timer`, and a new `process_due_timers` branch.
- **`admit_message`** (`runtime/src/lib.rs`) — reused unmodified for retry dispatch, exactly as the original dispatch already uses it; performs fresh, uncached capability validation on every call regardless of whether the presented `Capability` was newly supplied or replayed from Runtime's own retained state (§5.7).
- **Crate-dependency discipline, confirmed and preserved** — `services/effect-coordinator/Cargo.toml` currently depends only on `synapse-common` and `synapse-timer`; every other replaceable-service crate (`persistence`, `supervisor`, `timer`) depends only on `synapse-common` (plus `synapse-api` for `supervisor`). Only `runtime`, `message-gateway`, and `capability-authority` itself depend on `synapse-capability-authority` workspace-wide. This EWO's design (§5.5) was corrected during its own engineering review specifically to preserve this pattern: the Effect Coordinator gains no new crate dependency; `Capability` retention is Runtime's own responsibility exclusively.

No reusable pattern from EWO-012/EWO-013/EWO-014 is bypassed; the new state mirrors already-established shapes exactly.

## 5. Implementation Design

### 5.1 Implementation Objectives

1. Give the Effect Coordinator a concrete `RetryIntent` type and `RetryDecision` outcome realizing §19.2's four-role partition and §19's own Retry Decision concept.
2. Give the Effect Coordinator a pure, deterministic `decide_retry` operation combining retry eligibility (§19.1), actor-supplied retry intent (§19.2), idempotency-class permission (§19.3), and retry-limit precedence (§19.4) into `Retry` or `Accept`.
3. Give Runtime the mechanism to act on that decision: schedule a retry via the existing Timer Service, recognize the timer's firing via the existing `process_due_timers` loop, and re-dispatch through the existing, unmodified admission pipeline with fresh capability validation.
4. Introduce no new capability, no new Effect lifecycle state, no new terminal outcome, and no crate dependency beyond what Runtime already, legitimately holds.

### 5.2 Components Affected

- `services/effect-coordinator/src/lib.rs` — new public `RetryIntent` struct, `RetryDecision` enum; new trait methods on `EffectCoordinator` (`set_retry_intent`, `set_operation`, `operation_of`, `dispatch_target_of`, `decide_retry`, `record_retry_timer_registered`, `effect_for_retry_timer`); matching delegating methods on `EffectCoordinatorHandle`.
- `services/effect-coordinator/src/internal.rs` — `EffectRecord` gains `retry_intent: RetryIntent`, `attempt_count: u32`, `operation: String`; `EffectCoordinatorImpl` gains `retry_timer: HashMap<EffectId, TimerId>`, `by_retry_timer: HashMap<TimerId, EffectId>`; new method implementations.
- `runtime/src/lib.rs` — `EffectRequest` gains `pub retry_intent: RetryIntent`; `Runtime` gains a new private field `retry_dispatch_material: HashMap<EffectId, (Capability, Vec<u8>)>`; `request_effect` populates it; `fail_effect`/`provider_lost_effect`/`timeout_effect` each call a new `maybe_schedule_retry` helper; `process_due_timers` gains one new branch; a new `dispatch_retry` method; two new audit-event helpers (`effect_retry_scheduled_event`, `effect_retry_dispatched_event`), both `actor`-keyed, matching every existing sibling helper's shape.
- No change to `services/effect-coordinator/Cargo.toml` (no new crate dependency), the Timer Service's own internals, Capability Authority, Persistence Service, Supervisor, or Scheduler.

### 5.3 The `RetryIntent` and `RetryDecision` Types

```text
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct RetryIntent {
    pub requested: bool,
    pub accept_non_idempotent_risk: bool,
    pub max_attempts: Option<u32>,
}

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum RetryDecision {
    Retry,
    Accept,
}
```

`RetryIntent` is requesting-actor-supplied, domain-level data (§19.2) — never inferred, never re-solicited per attempt. `RetryDecision` is the Effect Coordinator's own output (§19, §19.2): `Retry` requires the caller to invoke `record_retry_scheduled`; `Accept` requires the caller to invoke the existing `accept` method.

### 5.4 Effect Coordinator Changes

New field on `EffectRecord` (Effect-level, per §15's own text that the Effect ID "retains or associates... the capability or operation reference originally presented" and per §19.2's requesting-actor-intent ownership):

```text
struct EffectRecord {
    actor: ActorId,
    status: EffectStatus,
    current_attempt: Option<EffectAttemptId>,
    retry_intent: RetryIntent,
    attempt_count: u32,
    operation: String,
}
```

`attempt_count` is incremented by the existing `record_dispatched` at the point each fresh attempt is minted. `retry_intent`/`operation` are each set exactly once, via two new, additive trait methods — never folded into `record_requested`'s existing signature:

```text
fn set_retry_intent(&mut self, effect: &EffectId, intent: RetryIntent);
fn set_operation(&mut self, effect: &EffectId, operation: String);
fn operation_of(&self, effect: &EffectId) -> Option<String>;
```

This additive approach — rather than changing `record_requested(actor: ActorId) -> EffectId`'s own signature — was adopted specifically because `record_requested` is called 59 times across the workspace (trait definition, handle delegation, internal implementation, and dozens of existing tests, confirmed by direct count during this EWO's own engineering review); the codebase's own established precedent (`record_timeout_registered`, `register_idempotency`) already, consistently adds a new tracked fact via a new, separate method rather than mutating an existing constructor.

New retry-decision accessor, combining per-attempt provider (already tracked) with per-Effect operation (new):

```text
fn dispatch_target_of(&self, attempt: &EffectAttemptId) -> Option<(ActorId, String)>;
```

New timer-correlation state, mirroring `by_timer`/`attempt_timer` exactly, at Effect level (since `RetryScheduled` is itself an Effect-level state, §16.1):

```text
retry_timer: HashMap<EffectId, TimerId>,
by_retry_timer: HashMap<TimerId, EffectId>,
```

```text
fn record_retry_timer_registered(&mut self, effect: &EffectId, timer: TimerId);
fn effect_for_retry_timer(&self, timer: &TimerId) -> Option<EffectId>;
```

The Retry Decision itself:

```text
fn decide_retry(
    &self,
    effect: &EffectId,
    outcome: AttemptOutcome,
    idempotency: IdempotencyDeclaration,
    capability_limit: Option<u32>,
) -> RetryDecision;
```

**Decision logic**, a pure function of Effect Coordinator tracked state and the named parameters (invariant 45):

1. `outcome ∉ {Failed, TimedOut, ProviderLost}` → `Accept` (§19.1).
2. `!retry_intent.requested` → `Accept` (§19.2 — no intent, no retry).
3. `effective_limit = match (capability_limit, retry_intent.max_attempts) { (Some(c), Some(a)) => Some(min(c, a)), (Some(c), None) => Some(c), (None, Some(a)) => Some(a), (None, None) => None }` (§19.4, narrowing-only).
4. `effective_limit == Some(n) && attempt_count >= n` → `Accept`.
5. `idempotency == Idempotent` → `Retry`.
6. `idempotency ∈ {NonIdempotent, Unknown} && !retry_intent.accept_non_idempotent_risk` → `Accept` (invariant 33).
7. Otherwise → `Retry`.

No wall-clock read, no thread-local, no randomness anywhere in this function.

### 5.5 Runtime Changes — Dispatch Material Retention (Regression-Corrected)

An earlier draft of this EWO proposed retaining the originally-presented `Capability` and request `payload` as fields on `EffectRecord`, inside the Effect Coordinator. This was identified, during this EWO's own Independent Engineering Re-review, as a regression: it would have required `services/effect-coordinator` to newly depend on `synapse-capability-authority`, making it the first replaceable-service crate ever to do so, breaking the otherwise-universal pattern confirmed in §4 above. The corrected design instead retains this material in **Runtime**, which already, legitimately depends on `synapse-capability-authority` and is already the sole component in the workspace that ever receives a `Capability` value directly:

```text
pub struct Runtime {
    state: RuntimeState,
    core: TrustedCore,
    scheduler: SchedulerHandle,
    supervisor: SupervisorHandle,
    timer: TimerHandle,
    persistence: PersistenceHandle,
    effect_coordinator: EffectCoordinatorHandle,
    retry_dispatch_material: HashMap<EffectId, (Capability, Vec<u8>)>,
}
```

`EffectRequest` gains one field:

```text
pub struct EffectRequest {
    pub provider: ActorId,
    pub operation: String,
    pub payload: Vec<u8>,
    pub timeout: Option<Duration>,
    pub retry_intent: RetryIntent,
}
```

`request_effect` is extended, immediately after `record_requested` and before `request.payload` is moved into the outgoing `Message`:

```text
let effect = self.effect_coordinator.record_requested(actor.clone());
self.effect_coordinator.set_operation(&effect, request.operation.clone());
self.effect_coordinator.set_retry_intent(&effect, request.retry_intent);
self.retry_dispatch_material.insert(effect.clone(), (authorization.clone(), request.payload.clone()));
```

`retry_dispatch_material` is never evicted — the identical no-eviction precedent every existing Effect Coordinator correlation map already establishes (`by_message`, `by_timer`, `by_actor`, `idempotency`), applied here at the Runtime layer for consistency.

### 5.6 Runtime Changes — Retry Decision Invocation

A new private helper, called from the tail of each of the three existing methods that currently leave a retry-eligible Effect `InProgress` "awaiting a future retry-decision mechanism" (`fail_effect`, `provider_lost_effect`, `timeout_effect`):

```text
fn maybe_schedule_retry(
    &mut self,
    attempt: &EffectAttemptId,
    outcome: AttemptOutcome,
) -> Result<(), RuntimeError> {
    let effect = self.effect_coordinator.effect_of(attempt).ok_or(RuntimeError::UnknownTarget)?;
    let actor = self.effect_coordinator.actor_of(&effect).ok_or(RuntimeError::UnknownTarget)?;
    let (provider, operation) = self.effect_coordinator
        .dispatch_target_of(attempt)
        .ok_or(RuntimeError::UnknownTarget)?;
    let idempotency = self.effect_coordinator.idempotency_of(&provider, &operation);
    let capability_limit = None; // Phase 1 — see §7

    match self.effect_coordinator.decide_retry(&effect, outcome, idempotency, capability_limit) {
        RetryDecision::Retry => {
            self.effect_coordinator.record_retry_scheduled(&effect)?;
            self.core.audit_emitter.emit(effect_retry_scheduled_event(&actor))?;
            let timer = self.timer.register(actor.clone(), Instant::now(), "effect.retry".into(), Vec::new());
            self.effect_coordinator.record_retry_timer_registered(&effect, timer);
        }
        RetryDecision::Accept => {
            self.effect_coordinator.accept(&effect)?;
        }
    }
    Ok(())
}
```

The retry timer is registered at `Instant::now()` (immediate) absent any architecture-fixed default (§4 Non-Goals; §19/§33 explicitly exclude numeric retry policy) — the only value that is itself an absence of policy, not an invented algorithm (§7).

### 5.7 Runtime Changes — Retry Recognition and Re-Dispatch

`process_due_timers` (`runtime/src/lib.rs`) gains one new branch, alongside its existing `attempt_for_timer` branch:

```text
} else if let Some(effect) = self.effect_coordinator.effect_for_retry_timer(&entry.id) {
    if matches!(self.effect_coordinator.effect_status(&effect), Some(EffectStatus::RetryScheduled)) {
        self.dispatch_retry(&effect)?;
    }
    // else: the Effect was already cancelled/resolved before this
    // timer fired (§21) — a harmless race, mirroring the existing
    // already-terminal-attempt branch's own discard-without-error
    // handling for the timeout case, applied here at Effect level.
}
```

New method:

```text
fn dispatch_retry(&mut self, effect: &EffectId) -> Result<(), RuntimeError> {
    let (capability, payload) = self.retry_dispatch_material
        .get(effect)
        .cloned()
        .ok_or(RuntimeError::UnknownTarget)?;
    let operation = self.effect_coordinator.operation_of(effect).ok_or(RuntimeError::UnknownTarget)?;
    let actor = self.effect_coordinator.actor_of(effect).ok_or(RuntimeError::UnknownTarget)?;

    let message = Message {
        id: MessageId(format!("effect-retry#{}", effect.0)),
        message_type: operation,
        sender: actor.clone(),
        destination: capability.target().clone(),
        payload,
        causation: None,
        correlation: Some(effect.0.clone()),
        capability_transfer: None,
        delivery_constraints: DeliveryConstraints,
        durability_classification: DurabilityClassification,
        deadline: None,
        replay_protection: ReplayProtection,
        information_classification: InformationClassification,
        trace_id: format!("effect-retry#{}", effect.0),
    };

    match self.admit_message(&message, &capability) {
        Ok(_instance) => {
            self.effect_coordinator.record_dispatched(effect, capability.target().clone(), message.id.clone())?;
            self.core.audit_emitter.emit(effect_retry_dispatched_event(&actor))?;
            Ok(())
        }
        Err(reason) => {
            self.effect_coordinator.record_denied(effect)?;
            self.core.audit_emitter.emit(effect_denied_event(&actor))?;
            Err(reason)
        }
    }
}
```

**Fresh capability validation is preserved exactly**: the retained `Capability` is presented, unmodified, through the same, unmodified `admit_message` pipeline every ordinary dispatch already uses — no cached authorization *decision* is stored or reused, only the presented *reference* itself. Retry's destination is derived from `capability.target()` directly, never a separately-tracked "last known provider" value: if the capability was not reissued after a Provider Actor replacement, retry naturally targets the original binding, and fresh validation rejects it if stale — exactly matching §14's own already-disclosed, pre-existing consequence, not a new failure mode this EWO introduces. A retry whose fresh validation fails is recorded via the existing `record_denied` path, exactly as `request_effect`'s own initial-dispatch failure branch already does.

### 5.8 Audit Helpers

```text
fn effect_retry_scheduled_event(actor: &ActorId) -> AuditEvent {
    AuditEvent { event_type: "effect.retry_scheduled".into(), actor: Some(actor.clone()), capability: None, message: None }
}

fn effect_retry_dispatched_event(actor: &ActorId) -> AuditEvent {
    AuditEvent { event_type: "effect.retry_dispatched".into(), actor: Some(actor.clone()), capability: None, message: None }
}
```

Both `actor`-keyed, matching every existing sibling helper (`effect_requested_event`, `effect_denied_event`, `effect_dispatched_event`, `effect_completed_event`, `effect_failed_event`) and `AuditEvent`'s own actual fields (`event_type`, `actor: Option<ActorId>`, `capability: Option<CapabilityId>`, `message: Option<MessageId>`) exactly — no new `AuditEvent` field, consistent with §17.

### 5.9 Determinism

`decide_retry` is a pure function of Effect Coordinator tracked state and its named parameters — no wall-clock read, no thread-local state, no randomness. Given identical Runtime history (the same sequence of Effect requests, attempt-level terminal outcomes, requesting-actor-supplied retry intent, Provider-declared idempotency classifications, and applicable capability constraints), retry decisions are identical, satisfying invariant 45 directly.

## 6. Testing Strategy

**Unit tests (Effect Coordinator, `services/effect-coordinator`):**
- Retry eligibility: `Completed`/`Cancelled` outcomes yield `Accept` regardless of intent/idempotency/limit; `Failed`/`TimedOut`/`ProviderLost` proceed past the eligibility gate.
- Retry authority: no code path exists for a Provider Actor to supply `RetryIntent` (structural/API-shape test).
- Idempotency permission: all nine `(class × requested × accept_risk)` combinations relevant to the decision table.
- Retry limits: capability-only ceiling; actor-only limit; both present (minimum taken); neither present (unlimited); exact boundary (`attempt_count == n-1` retries, `== n` accepts).
- Retry decision construction: the full combinatorial matrix across eligibility × intent × idempotency × limit.
- Deterministic scheduling: identical inputs against two independently constructed `EffectCoordinatorHandle` instances yield identical decisions (mirrors the existing idempotency session-scoping test pattern, EWO-014).
- Edge cases: multi-retry chains (retry-of-a-retry); a fabricated/unknown Effect ID; a retry decision immediately after `record_retry_scheduled` (no current attempt exists).

**Integration tests (Runtime, `runtime/src/lib.rs`):**
- Lifecycle integration: `Failed` + `Idempotent` + `requested: true` → new Effect Attempt ID, fresh capability validation, correct provider dispatch.
- Timeout interaction: `TimedOut` is retry-eligible identically to `Failed`; the retry timer and the original timeout timer remain distinct registrations.
- Cancellation interaction: `Cancelled` never reaches `decide_retry`; a `RetryScheduled` Effect remains cancellable via the existing §21 triggers before its new attempt dispatches.
- Persistence recovery: actor restore does not resurrect a `RetryScheduled` Effect or replay a retry.
- Effect Coordinator interaction: `retry_intent`/`operation` are correctly threaded from `EffectRequest` into stored `EffectRecord` state.
- Multiple concurrent retries: two independent Effects retry without interference (per-`EffectId` isolation, exactly as `by_actor`/`idempotency` already demonstrate for their own keys).
- Actor termination: a requesting actor terminated while its Effect is `RetryScheduled` — the pending retry is cancelled, never dispatched.
- Replay determinism: an identical failure-outcome sequence fed into two fresh `Runtime` instances produces bit-for-bit identical retry decisions.

**Regression tests:**
- Full existing workspace test suite (705 tests, per the current published baseline) continues to pass in substance. The one test that is intentionally superseded — `retry_scheduling_remains_unimplemented_by_this_milestone` (`runtime/src/lib.rs`), which explicitly documented the prior absence of a retry-decision mechanism — MUST be replaced with a test asserting the new, correct behavior, disclosed as an intentional, named change in the resulting Engineering Report, never silently deleted.

## 7. Explicit Exclusions

This EWO MUST NOT:

- implement any numeric retry policy — maximum-attempt defaults, backoff calculation, or jitter (§4; §19; §33);
- implement adaptive, probabilistic, or AI-driven retry decisions, or any form of runtime policy inference — every input to `decide_retry` is declarative and caller-supplied, never learned or heuristically adjusted;
- extend `ConstraintSet` (`common/src/lib.rs`, currently a genuinely empty unit struct) to carry a capability-declared retry-limit value — this is **Phase 2**, explicitly deferred; this EWO (**Phase 1**) ships with `capability_limit` always `None` in `maybe_schedule_retry`, which is architecturally legal per §19.4 ("where no capability-declared ceiling is presented, the requesting actor's own... preference alone governs");
- add any new capability, capability operation string, or capability-authorization step (§19.4; §23.5's identical reasoning);
- add any new Effect lifecycle state, terminal outcome, or `AuditEvent` field-layout change;
- add a new crate dependency to `services/effect-coordinator` — `Capability`/payload retention is Runtime's own, exclusive responsibility (§5.5);
- change `admit_message`, `record_dispatched`, `record_denied`, `cancel_effects_for_actor`, `cancel_effect`, or any other existing Runtime or Effect Coordinator method's own signature or behavior;
- change the Timer Service, Capability Authority, Persistence Service, Supervisor, Scheduler, or Actor Host in any way;
- perform any opportunistic refactor, rename, or reformatting outside this milestone's own stated scope.

## 8. Validation Requirements

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace --all-targets --all-features
cargo test --workspace --all-targets --all-features
```

All four MUST pass cleanly, from a forced-clean state, before this milestone is considered complete. The baseline to compare against is 705 passed, 0 failed (this EWO's own base state, §2); the final count MUST be independently re-derived by direct summation, not asserted from a single aggregate figure, consistent with this engineering effort's own established validation discipline.

## 9. Reporting Requirements

An Engineering Report SHALL be authored recording: objective; implementation summary; every file modified; validation performed (all four gates, with actual, independently-derived test counts); test coverage mapped against §6; architectural conformance against §3; the disclosed Phase 1/Phase 2 scope split (§7) as an explicit, factual part of engineering history, not a defect to be minimized or concealed; deviations from this EWO, if any; and recommendations for a future Phase 2 (`ConstraintSet` extension) EWO. Its own document identifier SHALL be derived from repository evidence at authoring time (STD-001 §7), not assumed in advance.

## 10. Definition of Done

- All items in §3 are implemented and independently verifiable against source.
- The public API matches §5.3/§5.4/§5.5 exactly.
- Every test in §6 exists and passes.
- All four validation gates in §8 pass cleanly.
- No item listed in §7 (Explicit Exclusions) was implemented.
- `services/effect-coordinator/Cargo.toml` carries no new dependency.
- An Independent Implementation Review has been conducted and has not identified a BLOCKER or unresolved MAJOR finding.

## Critical Safety Rule (restated for the implementer)

ARCH-008 is Approved and frozen. Implementation MUST conform to it, never silently reinterpret it. If any behavior this EWO requires proves genuinely underspecified or contradictory once implementation begins, stop and report the exact gap rather than inventing semantics — the same discipline this engineering effort has already applied consistently across every prior EWO. In particular, do not add a capability-model dependency to `services/effect-coordinator` under any circumstance — this EWO's own engineering review specifically identified and corrected exactly that regression once already (§5.5); do not reintroduce it.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Denver Jacobs (AI-assisted) | Initial Draft, authored from ARCH-008 v0.4.3 §19.1–§19.4 (the Retry Architecture Completion amendment) and direct inspection of the published EWO-012/EWO-014 implementations' own timer-correlation and idempotency-map precedents. This engineering effort underwent a full review-correction-re-review-correction-re-review cycle before its first commit — disclosed here in full, in the order it occurred, rather than presented as correct on first draft: (1) an Independent Engineering Review found two MAJOR findings (no specified location for provider-operation storage, needed by the idempotency-lookup step of retry decisions; no specified integration with `process_due_timers`, the actual existing mechanism by which a fired timer becomes an action) and two MINOR findings (a proposed audit-helper signature inconsistent with every sibling helper; an `record_requested`-signature change whose 59-call-site blast radius was undisclosed) — concluding `EWO-015 REQUIRES CORRECTION`; (2) a correction pass resolved all four, adding `EffectRecord.operation`, a `retry_timer`/`by_retry_timer` correlation-map pair mirroring EWO-012's own `by_timer`/`attempt_timer` precedent, a corrected `process_due_timers` branch and `dispatch_retry` method, a corrected `actor`-keyed audit helper, and an additive `set_retry_intent` method rather than an `record_requested` signature change; (3) an Independent Engineering Re-review found that this correction had itself introduced a new regression — proposing storage of the originally-presented `Capability` and request `payload` inside `EffectRecord`, which would have made `services/effect-coordinator` the first replaceable-service crate ever to depend on `synapse-capability-authority`, breaking a universal, structural pattern confirmed across every other replaceable-service crate (`persistence`, `supervisor`, `timer`) — concluding `EWO-015 REQUIRES FURTHER CORRECTION`; (4) a regression correction relocated `Capability`/payload retention to a new, private, `EffectId`-keyed map on `Runtime` itself (which already, legitimately depends on `synapse-capability-authority` and already exclusively handles `Capability` values at first dispatch), introducing zero new crate dependencies; (5) a Focused Independent Engineering Re-review independently re-confirmed, by direct `Cargo.toml` and workspace-grep inspection, that the regression was fully eliminated with no new issue introduced, concluding `EWO-015 FOCUSED ENGINEERING RE-REVIEW COMPLETE — READY FOR FOUNDER APPROVAL`. The specification recorded in this document (§5) reflects only the final, fully-corrected design — no superseded design is presented as current. |
| 0.1.1 | 2026-07-28 | Denver Jacobs (Founder) | Governance disposition recorded — **no engineering-scope, requirement, or exclusion changed from 0.1.0**. Records the Founder's strategic approval of the completed Engineering Work Order review cycle (Independent Engineering Review → Correction → Independent Engineering Re-review → Regression Correction → Focused Independent Engineering Re-review, all summarized in 0.1.0's own changelog entry above): `status` transitions from `Draft` to **`Approved`**; the Approval Status table is completed (Approval Authority recorded against Denver Jacobs, Founder, exercising Class E implementation-decision authority per GOV-010 §4–§5 in the absence of an identified delegate, on the identical basis EWO-014's own 0.1.1 disposition already established). This disposition confirms EWO-015 remains faithful to ARCH-008 v0.4.3, preserves deterministic Runtime behaviour and clean Runtime layering (the regression correction, above, if anything strengthens this), and introduces no architectural compromise. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-28 |
| Independent Engineering Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `EWO-015 REQUIRES CORRECTION` (two MAJOR, two MINOR findings) → corrected → `EWO-015 REQUIRES FURTHER CORRECTION` (one MAJOR regression found in the correction itself) → corrected → `EWO-015 FOCUSED ENGINEERING RE-REVIEW COMPLETE — READY FOR FOUNDER APPROVAL` | 2026-07-28 |
| Approval Authority | Denver Jacobs, Founder, exercising Class E (Implementation) decision authority under GOV-010 §4–§5 in the absence of an identified delegate | **Approved** | 2026-07-28 |
