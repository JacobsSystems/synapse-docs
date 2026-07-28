---
document_id: EWO-014
title: "Provider Idempotency Registration"
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
    - ARCH-008 (v0.4.2, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md) — the sole architectural authority this EWO implements; specifically §23.1–§23.5 and invariants 42–44, added by the Idempotency Metadata Amendment; not amended by this EWO
    - ARCH-002 (Runtime Architecture) — governs the capability model (§8, §9) this EWO reuses unmodified
  adrs:
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  predecessor: "EWO-013 / ER-014 (Effect Cancellation on Actor Termination) — its `by_actor` reverse-index pattern (Effect Coordinator field + trait query, populated at the point a fact is established, never evicted) is the direct, reusable design precedent this EWO follows for idempotency-declaration storage"
  reported_by: "The next sequentially available Engineering Report identifier per STD-001 §7 at the time this EWO is implemented — not assumed in advance"
  base_state:
    runtime_head: 3626a73288f31c1a97cdf4d1c8bca181d12c7d9b
    docs_head: 7a12b806a3b143476209bf746548728c724ea76b
---

# EWO-014 — Provider Idempotency Registration

Registered per STD-001 §46 (Engineering Work Orders). This authorizes implementation work only. It does not itself constitute approval, and does not amend ARCH-008 or any other architecture, standards, or governance document.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-014 |
| Title | Provider Idempotency Registration |
| Version | 0.1.1 |
| Status | **Approved** |
| Author | Denver Jacobs |
| Created | 2026-07-28 |
| Predecessor milestone | EWO-013 / ER-014 — Effect Cancellation on Actor Termination |
| Governing architecture | ARCH-008 v0.4.2 (Approved), §23.1–§23.5, invariants 42–44 |

---

## 1. Purpose

ARCH-008 §23 reserved provider-declared idempotency metadata (`Idempotent`/`NonIdempotent`/`Unknown`) from the document's own initial approval, but left "concrete metadata types, storage, and enforcement mechanics... deferred to implementation." The v0.4.2 Idempotency Metadata Amendment (§23.1–§23.5) completed the *normative* model this deferral required — registration granularity (per `(Provider Actor ActorId, operation)` pair, never per Effect request), ownership (Effect Coordinator, bookkeeping only), session-scoping (never persisted, never actor domain state), and default/failure semantics (no declaration ≡ `Unknown`; re-registration replaces without error) — while explicitly leaving the *concrete representation* to implementation. This EWO closes that remaining gap: it gives the Effect Coordinator a real, concrete, tested facility for recording and querying an idempotency declaration, exactly as §23.1–§23.5 already specify, introducing nothing those sections do not already authorize.

This EWO does not implement Retry Architecture (ARCH-008 §19) itself. It builds the bookkeeping primitive a future Retry Decision will consume; it does not build the consumer.

## 2. Repository Baseline

- `synapse-runtime` @ `3626a73288f31c1a97cdf4d1c8bca181d12c7d9b` (published; Effect Runtime Foundation through Effect Cancellation on Actor Termination implementation).
- `synapse-docs` @ `7a12b806a3b143476209bf746548728c724ea76b` (published; ARCH-008 v0.4.2 Approved, ER-014).

An implementer MUST re-verify both repositories are still at these exact commits (or a documented, reconciled later state) before beginning implementation, per this engineering effort's own established repository-verification discipline.

## 3. Architectural Scope — Requirements Extracted from ARCH-008, with Evidence

| Requirement | ARCH-008 citation |
|---|---|
| Idempotency metadata has exactly three values: `Idempotent`, `NonIdempotent`, `Unknown` | §23 |
| Declared at operation granularity — per `(Provider Actor ActorId, operation)` pair — never per Effect request or Effect Attempt | §23.1; invariant 42 |
| Registration is not itself an Effect request; undergoes no Effect lifecycle transition; mints no Effect ID or Effect Attempt ID | §23.1 |
| The concrete registration mechanism (how a Provider's declaration reaches the Effect Coordinator) is deferred to implementation | §23.1; §33 |
| Effect Coordinator is the sole owner; bookkeeping only, never itself deciding whether a retry occurs | §23.2 |
| Runtime-session-scoped; MUST NOT be included in actor checkpoints; MUST NOT be actor domain state; MUST NOT survive a Runtime process restart | §22 (general rule); §23.2; invariant 43 |
| No declaration for an operation is treated identically to an explicit `Unknown` declaration | §23.3 |
| A Provider Actor restart, stop, or replacement does not, by itself, invalidate or require re-declaration within the same Runtime session | §23.3 |
| Re-registration for an already-declared operation replaces the prior declaration; always legal, never an error | §23.3 |
| No new capability, capability operation string, or capability-authorization step is introduced; the operation is already gated by the existing `effect.<domain>.<operation>` capability structure | §23.5; invariant 44 |
| No new Effect lifecycle state, terminal outcome, or `AuditEvent` field-layout change | §23 (closing sentence); §31 invariant 34 area (Effect Classification precedent for reserved-extension, non-enforcement) |

No behavior beyond this table is inferred. Where ARCH-008 is silent on the registration mechanism specifically (§23.1's own explicit deferral), this EWO does not invent one — see §7 (Explicit Exclusions).

## 4. Existing Implementation Analysis — Components to Be Reused

- **Effect Coordinator (`synapse-effect-coordinator`)** — the direct, reusable design precedent is the `by_actor: HashMap<ActorId, Vec<EffectAttemptId>>` reverse index (EWO-013): a new field on `EffectCoordinatorImpl`, exposed via one new trait method, populated at the exact point the underlying fact is established, never evicted, on the same basis `effects`/`attempts`/`by_message`/`by_timer`/`attempt_timer`/`by_actor` are never evicted. This EWO adds a structurally identical map for idempotency declarations.
- **`EffectCoordinatorHandle`** (`services/effect-coordinator/src/lib.rs`) — the public delegating wrapper every trait method already passes through (`self.0.<method>(...)`); this EWO's two new methods follow the identical delegation pattern every existing method already uses.
- **Session-scoping, already structurally free** — `EffectCoordinatorHandle` is constructed fresh at `Runtime::bootstrap()` (confirmed: no checkpoint/restore interaction exists anywhere in `services/effect-coordinator`, and ARCH-008 §22 already, generally forbids one). §23.2/§23.3's "MUST NOT survive a Runtime process restart" requirement is therefore satisfied automatically by *not* adding any persistence hook — there is no removal logic to write; a fresh process genuinely has no prior map to remove from.
- **`ActorId`** — already an existing dependency of `synapse-effect-coordinator` via `synapse-common`; no new crate dependency is required.

No reusable pattern from EWO-001/EWO-002/EWO-012/EWO-013 is bypassed; the new state mirrors an already-established shape exactly.

## 5. Implementation Design

### 5.1 Implementation Objectives

1. Give the Effect Coordinator a concrete `IdempotencyDeclaration` type realizing ARCH-008 §23's own reserved three values.
2. Give the Effect Coordinator a way to register a declaration for a `(Provider ActorId, operation)` pair, always legally, replacing any prior declaration for that same pair.
3. Give the Effect Coordinator a way to query the declaration for a `(Provider ActorId, operation)` pair, returning the registered value or `Unknown` if none exists — the default and the query are the same operation; no separate "has a declaration been made" check is introduced.
4. Introduce no new capability, no new Effect lifecycle state, no new terminal outcome, and no wiring into Retry Decision, Runtime dispatch, or any public Runtime method (§7).

### 5.2 Components Affected

- `services/effect-coordinator/src/lib.rs` — new public `IdempotencyDeclaration` enum (`Idempotent`, `NonIdempotent`, `Unknown`); two new trait methods on `EffectCoordinator`; two new delegating methods on `EffectCoordinatorHandle`.
- `services/effect-coordinator/src/internal.rs` — one new field on `EffectCoordinatorImpl`; two new method implementations.
- No change to `runtime/src/lib.rs`, `Cargo.toml` (no new dependency), the Timer Service, Capability Authority, Persistence Service, Supervisor, or Scheduler.

### 5.3 The `IdempotencyDeclaration` Type

```text
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum IdempotencyDeclaration {
    Idempotent,
    NonIdempotent,
    Unknown,
}
```

Exactly the three values ARCH-008 §23 already reserves by name. No additional variant, no payload on any variant — a classification only, per §23's own framing throughout.

### 5.4 Effect Coordinator Changes

Add to the `EffectCoordinator` trait, `EffectCoordinatorImpl`, and `EffectCoordinatorHandle`:

```text
fn register_idempotency(&mut self, provider: ActorId, operation: String, declaration: IdempotencyDeclaration);
fn idempotency_of(&self, provider: &ActorId, operation: &str) -> IdempotencyDeclaration;
```

Internal state: one new field on `EffectCoordinatorImpl`, mirroring `by_actor`'s own shape precedent:

```text
idempotency: HashMap<(ActorId, String), IdempotencyDeclaration>,
```

`register_idempotency` unconditionally inserts (or overwrites) the entry for the given `(provider, operation)` key — never rejects, never returns a `Result`, on the direct basis §23.3 states registration "is an ordinary, always-legal registration, never a rejected or erroneous act." `idempotency_of` returns `self.idempotency.get(&(provider.clone(), operation.to_string())).copied().unwrap_or(IdempotencyDeclaration::Unknown)` — the absent-key case and the explicit-`Unknown` case are structurally identical results, directly realizing §23.3's "no declaration is treated identically to an explicit `Unknown` declaration" without a separate `Option`-unwrapping step at any call site.

No existing method changes behavior. No existing field is touched.

### 5.5 Session-Scoping and Removal

No removal logic is implemented, and none is required. `idempotency` is constructed empty by `EffectCoordinatorImpl::new()` (via `#[derive(Default)]`, the same mechanism every other field already uses) and exists only for the lifetime of one `EffectCoordinatorHandle`, itself constructed fresh once per `Runtime::bootstrap()` call. A Runtime process restart therefore discards every registered declaration as a direct, structural consequence of nothing persisting it — exactly as §23.2/§23.3 require, and exactly as `by_message`/`by_timer`/`by_actor` already, identically behave. §23.3's "a Provider Actor restart, stop, or replacement does not, by itself, invalidate... within the same Runtime session" requirement is satisfied by the same absence of action: nothing in this EWO's own design keys or clears entries by `ActorInstanceId`, `by_actor`, or any Provider-lifecycle event — the `idempotency` map is untouched by Stop, Restart, or durable deletion of any actor, Provider or otherwise.

### 5.6 Error Handling

`register_idempotency` is infallible (no `Result`, no panic path — a `HashMap` insert/overwrite cannot fail). `idempotency_of` is infallible and total (`unwrap_or(Unknown)` guarantees a value for every input, including a fabricated/unknown `ActorId` or operation string that was never registered).

### 5.7 Determinism

No new non-determinism. Both new methods are pure, synchronous `HashMap` operations — no clock read, no I/O, no randomness.

## 6. Testing Strategy

Every behavior in §3/§5 maps to at least one test. No test relies on real elapsed time.

**Unit tests (Effect Coordinator, `services/effect-coordinator`):**
- `idempotency_of` returns `Unknown` for a `(provider, operation)` pair with no registration (missing-registration default, §23.3).
- Registering `Idempotent` for a `(provider, operation)` pair and querying it returns `Idempotent`; likewise for `NonIdempotent` and an explicit `Unknown` registration.
- Registering a declaration for one operation does not affect the query result for a different operation on the same Provider (operation-granularity, §23.1).
- Registering a declaration for one Provider does not affect the query result for the same operation string on a different Provider (Provider-granularity, §23.1).
- Registering `NonIdempotent` and then registering `Idempotent` for the identical `(provider, operation)` pair: the second registration replaces the first; the query returns `Idempotent`, never an error and never a merged/ambiguous result (replacement semantics, §23.3).
- Registering the identical declaration twice in a row for the same pair is legal and idempotent in its own right (no error, same query result).
- A freshly constructed `EffectCoordinatorHandle` (mirroring a fresh Runtime session) reports `Unknown` for every `(provider, operation)` pair, including ones that were registered against a *previous*, now-discarded `EffectCoordinatorHandle` instance — the direct unit-level proof of session-scoping (§23.2, §23.3): construct one handle, register a declaration, construct a second, independent handle, and confirm the second reports `Unknown`.
- Registration and query for a fabricated/never-defined `ActorId` behave identically to any other `ActorId` — no panic, no special-cased error (§5.6).

**Integration tests (Runtime, `runtime/src/lib.rs`):**
- None required by this EWO's own scope. This milestone introduces no Runtime-level call site, and Runtime does not yet read or write idempotency declarations at all (§7). Existing Runtime tests are expected to pass entirely unmodified, since no Runtime file is touched by this EWO.

**Regression tests:**
- Full existing workspace test suite (695 tests, per the current published baseline) continues to pass unmodified — this EWO adds a new field and two new methods to one crate; it changes no existing method's signature or behavior anywhere.

## 7. Explicit Exclusions

This EWO MUST NOT:

- implement Retry Architecture (ARCH-008 §19), retry decisions, retry scheduling, retry execution, or any consumer of `idempotency_of` — a distinct, future milestone; this EWO builds the bookkeeping primitive only;
- implement backoff algorithms or any numeric retry policy;
- wire `register_idempotency` into any Runtime-level public method, Provider Actor callback, or admission-pipeline call site — the concrete registration *mechanism* (how a Provider's own declaration actually reaches the Effect Coordinator at runtime) remains explicitly deferred per ARCH-008 §23.1's own text ("the concrete registration mechanism... is deferred to implementation"); this EWO provides the Effect-Coordinator-level facility such a future mechanism will call, and discloses, rather than silently invents, that no such call site exists yet after this EWO completes;
- implement Provider business/operation failure signalling, persistent storage, checkpoint integration, or Distributed Runtime — each its own, separate, future milestone per the Effect Runtime Roadmap and ARCH-002 §23's Deferred Architecture table;
- add any new capability, capability operation string, or capability-authorization step (§23.5; invariant 44);
- add any new Effect lifecycle state, terminal outcome, or `AuditEvent` field-layout change;
- change `request_effect`, `cancel_effects_for_actor`, `cancel_effect`, `process_due_timers`, or any other existing Runtime method in any way;
- change the Timer Service, Capability Authority, Persistence Service, Supervisor, Scheduler, or Actor Host in any way;
- perform any opportunistic refactor, rename, or reformatting outside this milestone's own stated scope.

## 8. Validation Requirements

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace --all-targets --all-features
cargo test --workspace --all-targets --all-features
```

All four MUST pass cleanly, from a forced-clean state, before this milestone is considered complete. The baseline to compare against is 695 passed, 0 failed (this EWO's own base state, §2); the final count MUST be independently re-derived by direct summation, not asserted from a single aggregate figure, consistent with this engineering effort's own established validation discipline.

## 9. Reporting Requirements

An Engineering Report SHALL be authored recording: objective; implementation summary; every file modified; validation performed (all four gates, with actual, independently-derived test counts); test coverage mapped against §6; architectural conformance against §3; the disclosed absence of any registration call site (§7) as an explicit, factual part of engineering history, not a defect to be minimized or concealed; deviations from this EWO, if any; and recommendations. Its own document identifier SHALL be derived from repository evidence at authoring time (STD-001 §7), not assumed in advance.

## 10. Definition of Done

- All items in §3 are implemented and independently verifiable against source.
- The public API matches §5.3/§5.4 exactly — `IdempotencyDeclaration` with exactly three variants; `register_idempotency`/`idempotency_of` with exactly the signatures given.
- Every test in §6 exists and passes.
- All four validation gates in §8 pass cleanly.
- No item listed in §7 (Explicit Exclusions) was implemented.
- An Independent Implementation Review has been conducted and has not identified a BLOCKER or unresolved MAJOR finding.

## Critical Safety Rule (restated for the implementer)

ARCH-008 is Approved and frozen. Implementation MUST conform to it, never silently reinterpret it. If any behavior this EWO requires proves genuinely underspecified or contradictory once implementation begins, stop and report the exact gap rather than inventing semantics — the same discipline this engineering effort has already applied consistently across every prior EWO. In particular, do not invent a registration call site to make this milestone feel "more complete" — ARCH-008 §23.1 explicitly, deliberately defers that mechanism, and this EWO's own scope deliberately stops at the Effect-Coordinator-level primitive.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Denver Jacobs (AI-assisted) | Initial Draft. Authored from ARCH-008 v0.4.2 §23.1–§23.5 (the Idempotency Metadata Amendment) and direct inspection of the published EWO-013 implementation's own `by_actor` reverse-index precedent. Specifies: a new `IdempotencyDeclaration` enum (`Idempotent`/`NonIdempotent`/`Unknown`); a new Effect-Coordinator-owned `idempotency: HashMap<(ActorId, String), IdempotencyDeclaration>` map; `register_idempotency`/`idempotency_of` trait methods, both infallible; session-scoping and restart-discard satisfied structurally (no removal logic required, mirroring every existing Effect Coordinator map); no new capability; no Runtime-level wiring of any kind, explicitly and disclosedly deferred (ARCH-008 §23.1's own stated deferral of the concrete registration mechanism). |
| 0.1.1 | 2026-07-28 | Denver Jacobs (Founder) | Governance disposition recorded — **no engineering-scope, requirement, or exclusion changed from 0.1.0**. Records the Founder's decision on the completed Engineering Work Order review cycle (Independent Engineering Review, concluding `EWO-014 ENGINEERING REVIEW COMPLETE — READY FOR PUBLICATION`, one disclosed Minor citation-precision finding, no Major or Critical finding, no engineering defect): `status` transitions from `Draft` to **`Approved`**; the Approval Status table is completed (Approval Authority recorded against Denver Jacobs, Founder, exercising Class E implementation-decision authority per GOV-010 §4–§5 — Class E, "local design and engineering choices within approved architecture," ordinarily delegable, but exercised here directly by the Founder in the absence of an identified delegate, consistent with the interim-authority pattern GOV-003 §3.2 already establishes for Class B). This disposition is disclosed as more formal than the Approval Status treatment any prior EWO in this repository has received (EWO-012 and EWO-013 both remained "Approval Authority: TBD/Pending" indefinitely, with implementation instead governed by their own review cycles, Independent Implementation Review, and Engineering Report) — not a correction of those prior EWOs, and not a new mandatory requirement for future ones; recorded here because it was explicitly requested for this milestone. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-28 |
| Technical Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | Completed — Independent Engineering Review concluded `EWO-014 ENGINEERING REVIEW COMPLETE — READY FOR PUBLICATION` (one disclosed Minor finding, non-blocking) | 2026-07-28 |
| Approval Authority | Denver Jacobs, Founder, exercising Class E (Implementation) decision authority under GOV-010 §4–§5 in the absence of an identified delegate | **Approved** | 2026-07-28 |
