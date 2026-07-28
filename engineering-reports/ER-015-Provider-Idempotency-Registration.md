---
document_id: ER-015
title: "Provider Idempotency Registration — Engineering Report"
version: 0.1.1
status: Approved
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-28
last_updated: 2026-07-28
classification: Public
related_documents:
  reports_on: "EWO-014 (work-orders/EWO-014-Provider-Idempotency-Registration.md, v0.1.1) — the governing Engineering Work Order this report records"
  architecture:
    - ARCH-008 (v0.4.2, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md) — specifically §23.1–§23.5 and invariants 42–44
    - ARCH-002 (Runtime Architecture) — governs the capability model (§8, §9) reused unmodified
  adrs:
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  standards:
    - STD-001
  predecessor: ER-014 (Effect Cancellation on Actor Termination — Engineering Report)
---

# ER-015 — Provider Idempotency Registration — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. It records the implementation exactly as independently verified. Nothing described here has been staged, committed, or pushed.

**Numbering.** The highest existing Engineering Report on record is `ER-014-Effect-Cancellation-on-Actor-Termination.md`; no ER-015 exists yet. This document is therefore **ER-015**, derived directly from the repository's own contents (STD-001 §7).

## 1. Executive Summary

EWO-014 (Provider Idempotency Registration) completes the concrete representation ARCH-008 §23 explicitly deferred to implementation: a Provider-declared idempotency classification (`Idempotent`/`NonIdempotent`/`Unknown`), registered and owned by the Effect Coordinator, at `(Provider ActorId, operation)` granularity, Runtime-session-scoped. The implementation is a pure, additive bookkeeping primitive — it introduces no Runtime-level call site, no registration transport mechanism, and no consumer of the fact it records, each deliberately and disclosedly deferred to future, separately authorized work. Validation passes completely (705 tests, 0 failures, 0 warnings). The Independent Implementation Review found zero findings of any severity. The implementation is ready for Founder Approval and subsequent publication.

## 2. Repository Baseline

**`synapse-runtime`:** path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `3626a73288f31c1a97cdf4d1c8bca181d12c7d9b`, tracking `origin/main`, 0 ahead / 0 behind, confirmed via `git fetch origin` immediately before authoring this report. Working tree modified by the implementation this report records: `services/effect-coordinator/src/internal.rs`, `services/effect-coordinator/src/lib.rs` — the identical two files the Independent Implementation Review examined; no drift occurred between review and this report.

**`synapse-docs`:** path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `1c51f5e2c808384d3b201ddf53ac9c754af3dfad`, tracking `origin/main`, 0 ahead / 0 behind. Pre-existing, unrelated backlog left completely untouched. `work-orders/EWO-014-Provider-Idempotency-Registration.md` (v0.1.1, Approved, untracked) is the governing, unchanged EWO. This report (`engineering-reports/ER-015-Provider-Idempotency-Registration.md`) is the only file this task adds to `synapse-docs`.

**Nothing staged, committed, or pushed in either repository at any point during this task.**

## 3. Implementation Summary

**Files modified:** `services/effect-coordinator/src/lib.rs` and `services/effect-coordinator/src/internal.rs` only. No Runtime file (`runtime/src/lib.rs`) was touched. The complete diff is additive: 26 insertions in `internal.rs`, 205 insertions in `lib.rs`, 0 deletions in either file.

**`IdempotencyDeclaration`** — a new `pub enum` with exactly three variants (`Idempotent`, `NonIdempotent`, `Unknown`), `#[derive(Debug, Clone, Copy, PartialEq, Eq)]`, matching `AttemptOutcome`'s own established derive precedent.

**Effect Coordinator additions** — two new methods on the `EffectCoordinator` trait, `EffectCoordinatorImpl`, and `EffectCoordinatorHandle`:

```rust
fn register_idempotency(&mut self, provider: ActorId, operation: String, declaration: IdempotencyDeclaration);
fn idempotency_of(&self, provider: &ActorId, operation: &str) -> IdempotencyDeclaration;
```

backed by one new field on `EffectCoordinatorImpl`:

```rust
idempotency: HashMap<(ActorId, String), IdempotencyDeclaration>,
```

**Registration** is an unconditional `HashMap::insert`, infallible, never rejected — a later call for the identical `(provider, operation)` pair silently **replaces** the prior declaration.

**Lookup** (`idempotency_of`) returns the registered value, or `IdempotencyDeclaration::Unknown` via `.copied().unwrap_or(Unknown)` if no declaration was ever registered — the absent-registration and explicit-`Unknown` cases are structurally identical results, never distinguished by an `Option` at any call site.

**Runtime-session scope** required no removal logic of any kind: `EffectCoordinatorImpl`'s `idempotency` field is constructed empty via `#[derive(Default)]`, and `EffectCoordinatorHandle::new()` is itself constructed fresh inside `Runtime::bootstrap()` (confirmed directly at `runtime/src/lib.rs:1111`) — a Runtime process restart discards every registered declaration as a direct structural consequence of nothing persisting it, exactly as ARCH-008 §22/§23.2 require.

## 4. Architectural Traceability

| ARCH-008 requirement | Citation | Implementation evidence |
|---|---|---|
| Idempotency has exactly three values | §23 | `IdempotencyDeclaration` — exactly `Idempotent`/`NonIdempotent`/`Unknown` |
| Declared at operation granularity, per `(Provider ActorId, operation)` pair | §23.1; invariant 42 | `idempotency: HashMap<(ActorId, String), IdempotencyDeclaration>` |
| Registration is not an Effect request; no lifecycle transition; no `EffectId`/`EffectAttemptId` minted | §23.1 | `register_idempotency` touches no `effects`/`attempts` map |
| Effect Coordinator sole owner, bookkeeping only | §23.2 | Field lives exclusively on `EffectCoordinatorImpl`; no retry decision made by either new method |
| Runtime-session-scoped; never actor domain state; never survives a process restart | §22 (general rule); §23.2; invariant 43 | No checkpoint/restore interaction anywhere in the crate (independently confirmed by direct grep, both during implementation and during review); `EffectCoordinatorHandle::new()` constructed fresh at `Runtime::bootstrap()` |
| No declaration ≡ explicit `Unknown` | §23.3 | `idempotency_of`'s own `unwrap_or(Unknown)` |
| Provider Actor restart/stop/replacement does not invalidate within the same session | §23.3 | No key or clearing logic references `ActorInstanceId`, `by_actor`, or any Provider-lifecycle event |
| Re-registration replaces, always legal, never an error | §23.3 | Unconditional `HashMap::insert`, infallible signature (no `Result`) |
| No new capability, operation string, or authorization step | §23.5; invariant 44 | No capability-related file touched; no new `effect.<domain>.<operation>` string introduced |

## 5. Engineering Traceability

| EWO-014 requirement | Citation | Implementation evidence |
|---|---|---|
| `IdempotencyDeclaration` type | §5.3 | Matches specified shape exactly |
| `register_idempotency`/`idempotency_of` signatures | §5.4 | Matches exactly, character-for-character |
| `HashMap<(ActorId, String), IdempotencyDeclaration>` internal representation | §5.4 | Matches exactly |
| No removal logic; session-scoping structural | §5.5 | No removal code written, per design |
| Infallibility | §5.6 | Neither method returns `Result` |
| No new non-determinism | §5.7 | Pure, synchronous `HashMap` operations only |
| Testing requirements | §6 | 10 new unit tests, one per named scenario (§6, below) |
| Explicit Exclusions (§7) | §7 | None implemented — verified in §7 of this report |

## 6. Validation

Independently re-run at implementation completion, again during the Independent Implementation Review, and a final time immediately before authoring this report, from the actual working tree:

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean, exit 0 |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace --all-targets --all-features` | Clean |
| `cargo test --workspace --all-targets --all-features` | **705 passed, 0 failed, 0 ignored** |

**Test totals:** 695 (EWO-013/ER-014 baseline) + 10 new = **705**, confirmed by direct summation of every `test result:` line across all 26 test binaries, independently re-derived at three separate points in this milestone's history. All 10 new tests are in `synapse-effect-coordinator` (61 → 71); `synapse-runtime` itself remains unchanged at exactly 286, directly confirming zero Runtime behavioral change. No existing test was removed, ignored, renamed, or had an existing assertion weakened — the complete diff across both modified files is purely additive (231 insertions, 0 deletions).

New tests, one per EWO-014 §6 requirement: default `Unknown` for an unregistered pair; registration and lookup for each of the three variants; operation-granularity isolation (one operation's registration does not affect another on the same Provider); Provider-granularity isolation (one Provider's registration does not affect the identical operation string on another Provider); replacement semantics (a later registration replaces, never merges or errors); idempotent re-registration (registering the same value twice is legal); Runtime-session isolation (a second, independent `EffectCoordinatorHandle` does not inherit a first handle's registrations); fabricated-`ActorId` robustness (no panic, no special-cased error).

## 7. Explicit Non-Implementation

The following remain intentionally, disclosedly absent, exactly as EWO-014 §7 requires:

- **Retry execution, retry scheduling, retry policy, backoff algorithms** — a distinct, future milestone; this implementation builds the bookkeeping primitive only, never a consumer of `idempotency_of`.
- **Registration transport / Runtime wiring** — `register_idempotency` is not called from anywhere in `runtime/src/lib.rs`, or from any Provider Actor callback, admission-pipeline call site, or public Runtime method. ARCH-008 §23.1 itself explicitly defers the concrete registration mechanism ("whether Runtime-mediated, established at Provider definition time, or otherwise") to implementation; this milestone provides the Effect-Coordinator-level facility such a future mechanism will call, and discloses — rather than silently invents — that no such call site exists after this implementation.
- **Persistent storage, checkpoint integration** — no interaction with the Persistence Service; session-scoping achieved structurally, not by any explicit exclusion logic.
- **Distributed Runtime** — untouched.
- **Capability changes** — no new capability, operation string, or authorization step.
- **Message redesign** — `common/src/lib.rs` untouched.
- **Effect lifecycle changes** — no new state, no new terminal outcome.
- **Architectural amendments** — ARCH-008 itself untouched by this task.

## 8. Independent Review Outcome

An Independent Implementation Review was conducted, re-deriving every conclusion directly from source rather than from this milestone's own implementation report, and independently re-executing all four validation gates from the working tree (705 passed, 0 failed, matching §6 above exactly).

**Findings: none — no Critical, Major, Minor, or Editorial finding of any kind.** The review independently confirmed: `ActorId` genuinely derives `Hash, Eq, Clone` (making the tuple-key `HashMap` valid, compiling code, not merely asserted); `EffectCoordinatorHandle::new()` is genuinely constructed fresh inside `Runtime::bootstrap()` at `runtime/src/lib.rs:1111`; no checkpoint/restore reference exists anywhere in the `synapse-effect-coordinator` crate; all 10 new tests genuinely exercise the real public API rather than a mock or shortcut; the diff is genuinely, completely additive.

**Resolution:** the Independent Implementation Review's own concluding statement: `EWO-014 IMPLEMENTATION REVIEW COMPLETE — READY FOR ENGINEERING REPORT`.

## 9. Conclusion

EWO-014 (Provider Idempotency Registration) successfully completed the concrete representation ARCH-008 §23 explicitly deferred: a Provider-declared idempotency classification, correctly scoped, correctly owned, correctly defaulted, and correctly session-scoped, with zero Runtime-level wiring — a deliberate, disclosed limitation matching ARCH-008 §23.1's own explicit deferral of the registration mechanism, not an oversight. The implementation faithfully realizes both ARCH-008 v0.4.2 and EWO-014 v0.1.1; preserves Runtime architecture completely (zero Runtime files touched); preserves deterministic behavior (no clock read, no I/O, no randomness); and passed independent review with zero findings of any severity. The implementation is ready for Founder Approval and subsequent publication.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Claude (AI-assisted) | Initial report. Records the Provider Idempotency Registration implementation exactly as completed and independently verified: `IdempotencyDeclaration`, `register_idempotency`/`idempotency_of`, the `idempotency` map, and structural session-scoping — independently re-verified against source across implementation, review, and this report's own re-execution of all four validation gates (705 tests, 0 failures). Records the Independent Implementation Review's zero-finding outcome truthfully. |
| 0.1.1 | 2026-07-28 | Denver Jacobs (Founder) | Governance disposition recorded — **no engineering content changed from 0.1.0**. Records the Founder's decision on this report: `status` transitions from `Draft` to **`Approved`**; the Approval Status table is completed (Approval Authority recorded against Denver Jacobs, Founder). Prior to recording this disposition, the report's own key factual claims were independently re-verified directly against the working tree — test total (705), diff totals (26/0 insertions/deletions in `internal.rs`, 205/0 in `lib.rs`), and the `EffectCoordinatorHandle::new()` citation at `runtime/src/lib.rs:1111` — all confirmed accurate. Disclosed on the identical basis as the EWO-014 approval: recording a formal Approval Authority disposition on an Engineering Report's own Approval Status table is more formal than this repository's established practice (neither ER-013 nor ER-014 ever received this treatment, both remaining "Approval Authority: Pending" indefinitely) — not a correction of those prior reports, recorded here because explicitly requested for this milestone. This disposition governs the Engineering Report only; it does not itself publish the underlying implementation, which remains a separate, subsequent act. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Claude (AI-assisted) | Drafted | 2026-07-28 |
| Reviewer | Independent Implementation Review (this engineering effort) | Approved, zero findings, no rework required | 2026-07-28 |
| Approval Authority | Denver Jacobs | **Approved** | 2026-07-28 |
