---
document_id: ER-016
title: "Retry Architecture Implementation — Engineering Report"
version: 0.1.0
status: Approved
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-28
last_updated: 2026-07-28
classification: Public
related_documents:
  reports_on: "EWO-015 (work-orders/EWO-015-Retry-Architecture-Implementation.md, v0.1.1, Approved) — the governing Engineering Work Order this report records"
  architecture:
    - ARCH-008 (v0.4.3, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md) — specifically §19.1–§19.4 and invariant 45, added by the Retry Architecture Completion amendment
    - ARCH-002 (Runtime Architecture) — governs the capability model (§8, §9) reused unmodified
  adrs:
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
  standards:
    - STD-001
  predecessor: ER-015 (Provider Idempotency Registration — Engineering Report)
---

# ER-016 — Retry Architecture Implementation — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or retroactively expand the scope of the EWO it reports against. It records the implementation exactly as independently verified.

**Numbering.** The highest existing Engineering Report on record is `ER-015-Provider-Idempotency-Registration.md`; no ER-016 exists yet. This document is therefore **ER-016**, derived directly from the repository's own contents (STD-001 §7), independently confirmed via `find engineering-reports -maxdepth 1 -type f` and a full-repository grep for "EWO-015"/"Retry Architecture Implementation" returning no prior report.

## 1. Executive Summary

EWO-015 (Retry Architecture Implementation) gives ARCH-008 §19's Retry Architecture — completed normatively by the v0.4.3 Retry Architecture Completion amendment (§19.1–§19.4, invariant 45) — a real, tested Runtime mechanism: a deterministic Retry Decision (eligibility, requesting-actor authority, idempotency-class permission, retry-limit precedence), Timer-based retry scheduling and correlation, due-timer recognition, and retry re-dispatch through the existing, unmodified capability-admission pipeline. This was required because ARCH-008 §19 had, until this milestone, only ever existed as bookkeeping (`record_retry_scheduled`) with no decision mechanism ever wired to it — every retry-eligible outcome (`Failed`, `TimedOut`, `ProviderLost`) was left permanently `InProgress`, a disclosed, deliberate gap in every prior milestone back through EWO-012. The implementation is committed at Runtime commit `c5959bb5a2b26b816a17ab1bdc4e77c6183a8c22`. An Independent Implementation Review independently re-traced every required behavior to source — capability safety, determinism, timer correlation, dependency discipline, actor-termination safety — and concluded zero CRITICAL and zero MAJOR findings (two MINOR, two ADVISORY, none blocking), with the final outcome `EWO-015 IMPLEMENTATION REVIEW COMPLETE — IMPLEMENTATION ACCEPTED`. Validation independently re-run for this report passes completely: 744 tests, 0 failures, 0 warnings.

**Evidence basis, disclosed explicitly.** This report's implementation description, diff statistics, and validation results (§§3–8) are repository-verifiable — independently re-derived from the actual working tree and commit history during the authoring of this report, not restated from memory. The Independent Implementation Review's own findings and classifications (§9) are recorded as supplied project evidence from that completed review; its full text is not itself a committed repository artifact, and is cited here as external session evidence, distinguished from the repository-verifiable facts surrounding it.

## 2. Authoritative Specification

`work-orders/EWO-015-Retry-Architecture-Implementation.md`, version `0.1.1`, status **Approved**. Confirmed directly from its own frontmatter and Approval Status table at authoring time. EWO-015 was treated throughout implementation as a frozen implementation contract — every required type, method signature, and decision-logic step (§5.3–§5.9) was realized as published, with two exceptions independently identified and corrected during implementation itself (an internal-only accessor discovered unnecessary, and a denial-propagation defect in `process_due_timers`), both disclosed in §10 below rather than presented as originally correct.

## 3. Repository Baseline

**`synapse-runtime`:** path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `c5959bb5a2b26b816a17ab1bdc4e77c6183a8c22`, tracking `origin/main`, 0 ahead / 0 behind, confirmed via `git fetch origin` immediately before authoring this report. Starting commit for the implementation range: `4256b4434447fb9ab43d0d901a5baf8476c024e3` (the EWO-014/ER-015 baseline). Working tree clean — no modification occurred during the authoring of this report.

**`synapse-docs`:** path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `3ca11f2398f14af8ed75dc2dead689962b4682a0` at the start of this task, tracking `origin/main`, 0 ahead / 0 behind. Pre-existing, unrelated backlog left completely untouched: `standards/STD-001-Documentation-Standards.md` (modified, pre-existing drift), `.ai/`, `consolidation/ACR-001-Architecture-Consolidation-Review.md`, `consolidation/RSS-001-Research-Synthesis-Review.md`, `governance/GOV-002-Vision-and-Mission.md`, `maintenance/EMO-001-ActorHost-Single-Live-Instance.md`, `research/RES-001` through `RES-006`, `work-orders/EWO-003-Message-Gateway.md` (all untracked). `work-orders/EWO-015-Retry-Architecture-Implementation.md` (v0.1.1, Approved) is the governing, unchanged EWO. This report (`engineering-reports/ER-016-Retry-Architecture-Implementation.md`) is the only file this task adds to `synapse-docs`.

## 4. Implementation Scope

Exactly three Runtime files were modified — confirmed via `git diff --name-only 4256b44..c5959bb`:

```text
services/effect-coordinator/src/lib.rs
services/effect-coordinator/src/internal.rs
runtime/src/lib.rs
```

`git diff --numstat` for this range, independently re-derived: `runtime/src/lib.rs` +777/−33; `services/effect-coordinator/src/internal.rs` +113/−1; `services/effect-coordinator/src/lib.rs` +795/−0; **1,685 insertions, 34 deletions** across all three files combined. No `Cargo.toml` anywhere in the workspace was modified — independently confirmed via `git diff --name-only` filtered for manifest files, returning nothing. No Documentation file was modified by the implementation itself; this report is the first Documentation-repository change associated with EWO-015 since its own publication.

## 5. Implemented Design

**Retry intent** (`RetryIntent`) — a `Copy` struct (`requested: bool`, `accept_non_idempotent_risk: bool`, `max_attempts: Option<u32>`), `Default` = not requested, carried once on `EffectRequest` and set on the owning `EffectRecord` via an additive `set_retry_intent` call — never re-solicited per attempt, never folded into `record_requested`'s own signature.

**Deterministic retry decision** (`decide_retry`) — a pure function of Effect Coordinator tracked state and its own parameters (outcome, idempotency classification, capability limit): retry-eligibility gate (`Failed`/`TimedOut`/`ProviderLost` only) → retry-intent gate → retry-limit gate → idempotency-class gate, returning exactly `Retry` or `Accept`. No wall-clock read, no thread-local state, no randomness anywhere in the function body.

**Attempt counting** — `EffectRecord.attempt_count: u32`, incremented once per freshly dispatched attempt inside the existing `record_dispatched`, the exact point EWO-015 specifies for retry-limit enforcement.

**Constitutional retry ceiling** — a capability-declared limit, where present, narrows via `min(capability, actor)`, never widens; absent a capability-declared value (Phase 1 of this milestone; `ConstraintSet` remains empty), the requesting actor's own preference alone governs, exactly as ARCH-008 §19.4 requires.

**Idempotency classification** — reused unmodified from EWO-014's own `idempotency_of(provider, operation)`; `Idempotent` permits retry (subject to intent); `NonIdempotent`/`Unknown` require the requesting actor's own explicit `accept_non_idempotent_risk`.

**Operation retention** — `EffectRecord.operation: Option<String>`, set once via `set_operation` at original request time, stable across every retry of the same Effect (§15's own text: the Effect ID "retains... the operation reference originally presented").

**Provider attempt-specificity** — unchanged: `AttemptRecord.provider` remains per-attempt, since a retry may target a different Provider Actor destination if replaced (§15).

**Runtime-owned dispatch material** — a private `Runtime` field, `retry_dispatch_material: HashMap<EffectId, (Capability, Vec<u8>)>`, populated once in `request_effect` (before `payload` is moved into the outgoing `Message`), never evicted, consulted only by `dispatch_retry`.

**Timer-based retry scheduling and correlation** — an immediate (`Instant::now()`) Timer Service registration on `RetryDecision::Retry`, correlated bidirectionally via new `retry_timer`/`by_retry_timer` `HashMap`s on `EffectCoordinatorImpl`, mirroring the EWO-012 `by_timer`/`attempt_timer` pattern exactly, at Effect level rather than Attempt level (since `RetryScheduled` has no current attempt).

**Due-timer integration** — one new branch in the existing `process_due_timers` loop, alongside the pre-existing timeout branch: a due retry timer resolves to an `EffectId`, and — only if the Effect is still genuinely `RetryScheduled` (a cancelled Effect is a harmless, silently discarded race) — invokes `dispatch_retry`.

**Retry dispatch reconstruction** — `dispatch_retry` rebuilds the retry `Message` from the retained `(Capability, payload)` and the Effect Coordinator's own retained `operation`, with destination derived from `capability.target()` directly (never a separately tracked "last known provider"), and presents it through the identical, unmodified `admit_message` pipeline.

**Fresh capability admission** — every retry dispatch undergoes the same, complete validation chain the original dispatch does: envelope validation, send-authority (target/operation-membership) validation, and Capability Authority's own domain/forgery, integrity, revocation, and expiry checks — all performed fresh, unconditionally, on every call, confirmed by direct inspection of `admit_message`'s own body, byte-identical before and after this implementation.

**Retry audit events** — `effect.retry_scheduled` and `effect.retry_dispatched`, both actor-keyed, matching every existing sibling audit helper's shape and `AuditEvent`'s own actual fields exactly.

**Actor termination behaviour** — a `RetryScheduled` Effect's pending retry timer is registered under the requesting actor's own `ActorId`, identical to the existing timeout-timer convention; the pre-existing, unmodified `execute_stop_decision`'s own `cancel_all_for_actor(actor)` call (confirmed byte-identical to the pre-implementation commit) therefore already cancels it, with no new mechanism required.

## 6. Dependency and Ownership Discipline

Confirmed directly from Cargo manifests and source, not merely asserted: `services/effect-coordinator/Cargo.toml` depends only on `synapse-common` and `synapse-timer` — unchanged by this implementation, and still the only two dependencies of that crate. `Capability` and payload retention live exclusively on `Runtime` (`runtime/Cargo.toml`, which already, legitimately depended on `synapse-capability-authority` before this milestone). Operation remains Effect-Coordinator-owned (`EffectRecord.operation`). Provider remains attempt-specific (`AttemptRecord.provider`, untouched). Timer Service remains the sole scheduling mechanism (`self.timer.register`/`query_due`/`cancel_all_for_actor`, all pre-existing and unmodified). Runtime remains the sole cross-component orchestrator — the Effect Coordinator decides retry outcomes but never dispatches, authorizes, or touches a capability object anywhere in its own crate.

## 7. Testing

**32 new Effect Coordinator unit tests** (`services/effect-coordinator`, 71 → 103): operation storage and stability across retries; `dispatch_target_of` combining per-attempt provider with per-Effect operation, including after a provider change; retry-intent storage and its silent no-op on an unrequested Effect; retry eligibility (`Completed`/`Cancelled` never eligible; `Failed`/`TimedOut`/`ProviderLost` all eligible); retry authority (no intent, no retry, unconditionally); the full idempotency-class permission matrix (`Idempotent`, `NonIdempotent` with and without risk acceptance, `Unknown` treated identically to `NonIdempotent`); retry-limit ownership (actor-only, capability-only, both — narrower always wins in either direction, unlimited when neither is present); decision determinism (repeated calls, identical inputs, identical results) and cross-instance session-scoping (a fresh coordinator inherits nothing from a prior one); a multi-retry chain proving the limit is evaluated against the correctly accumulated attempt count; and bidirectional retry-timer correlation, including replacement of a stale reverse mapping.

**7 new Runtime integration tests** (`runtime`, 286 → 293): the full retry-and-redispatch cycle (schedule → fire → fresh attempt, same provider); non-idempotent retry denial and permission at the Runtime level; a Runtime-level retry-limit-reached scenario; actor termination cancelling a pending retry via the pre-existing `execute_stop_decision`; two independent Effects retrying without cross-contamination; retry-scheduled/retry-dispatched audit-event content and actor attribution; and a revoked capability causing a scheduled retry to be denied, proving fresh validation is never bypassed.

**Five pre-existing tests updated, each individually justified:** four assertions (`failing_an_effect_attempt_...`, the `ProviderLost`, and two `TimedOut` cases) changed their expected `EffectStatus` from `InProgress` to `Accepted(outcome)` — the correct, spec-mandated consequence of a default (`requested: false`) `RetryIntent` now flowing through a real decision mechanism that immediately accepts, rather than leaving the Effect permanently unresolved as every prior milestone disclosedly did. One test (`retry_scheduling_remains_unimplemented_by_this_milestone`) was fully replaced by `retry_scheduling_is_now_implemented_and_re_dispatches_after_the_retry_timer_fires`, since its own name and assertion existed specifically to document the absence this milestone closes — disclosed as an intentional, named supersession, never a silent deletion. No test was weakened, and no existing assertion was removed without a replacement proving the corrected behavior.

**Final total: 744 passed, 0 failed** (705 baseline + 39 net new/replaced).

## 8. Verification

Independently re-run for this report, from the actual working tree, immediately before authoring:

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean, exit 0 |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace --all-targets --all-features` | Clean |
| `cargo test --workspace --all-targets --all-features` | **744 passed, 0 failed, 0 ignored** |
| Focused: `synapse-effect-coordinator` | 103 passed, 0 failed |
| Focused: `synapse-runtime` | 293 passed, 0 failed |
| `git diff --check 4256b44..c5959bb` | Clean, no whitespace errors |

Test total independently re-derived by direct summation across all test binaries in this run, matching the Independent Implementation Review's own re-derivation exactly.

## 9. Independent Implementation Review

An Independent Implementation Review was conducted against Runtime commit `c5959bb5a2b26b816a17ab1bdc4e77c6183a8c22`, independently tracing every required behavior to source rather than accepting the implementation's own report — including direct inspection of `admit_message`, `execute_stop_decision`, `cancel_all_for_actor`, `query_due`, and `Capability::validate`, none of which were assumed correct without being read.

**Findings: zero CRITICAL, zero MAJOR, two MINOR, two ADVISORY — none blocking.**

1. **MINOR** — `EffectRecord.operation` is typed `Option<String>`, not the `String` EWO-015 §5.4 illustrates. No behavioral impact: this avoids a fabricated empty-string default at `record_requested` time and is strictly more correct than the illustrated type; every documented `operation_of`/`dispatch_target_of` behavior is preserved exactly.
2. **MINOR** — EWO-015 §6 names a specific integration-test category ("an identical failure-outcome sequence fed into two fresh Runtime instances produces bit-for-bit identical retry decisions") that was not implemented as its own dedicated test. The underlying property (`decide_retry`'s purity) is independently confirmed directly from source and is very likely to hold; it is simply not proven by a test bearing that exact name. Recommended, not required, as follow-up.
3. **ADVISORY** — `RetryIntent.max_attempts` counts total attempts including the original dispatch, not retries-only (`max_attempts: Some(1)` means zero retries, not one retry). This is a faithful, literal, correct implementation of EWO-015's own decision-logic pseudocode, not a deviation — worth making explicit in a future Phase 2 EWO's own field documentation.
4. **ADVISORY** — An initial implementation draft added a `retry_timer_for_effect` reverse accessor not specified by EWO-015; independent tracing of `Timer::cancel_all_for_actor` and `query_due` confirmed it was unnecessary (retry timers, registered under the requesting actor's own `ActorId`, are already cancelled by the pre-existing, actor-keyed bulk-cancellation mechanism). Its removal was independently confirmed correct — conformance to the published specification, not a missing feature.

**Resolution:** the Independent Implementation Review's own concluding statement: `EWO-015 IMPLEMENTATION REVIEW COMPLETE — IMPLEMENTATION ACCEPTED`.

## 10. Deviations and Refinements

**`Option<String>` for `operation`, in place of the EWO-015 §5.4-illustrated `String`.** `EffectRecord.operation` is set once, after `record_requested`, via the additive `set_operation` call — there is no meaningful value to initialize a plain `String` field to at construction time that would not itself be a fabricated placeholder (an empty string indistinguishable from a genuinely-empty-but-set operation). `Option<String>`, defaulting to `None` and populated only once genuinely known, preserves truthful state throughout and does not alter any required observable behavior described anywhere in EWO-015 or ARCH-008 — `operation_of` and `dispatch_target_of` both behave exactly as specified in every test scenario.

**Two implementation-time discoveries, both fully resolved before commit, both disclosed in EWO-015's own Revision History (v0.1.0) and not repeated in full here:** an initial correction introduced, and a subsequent re-review caught, a crate-dependency regression (`Capability`/payload retention briefly proposed inside the Effect Coordinator); it was corrected before any commit occurred, and `services/effect-coordinator/Cargo.toml` was never actually modified at any point in the repository's own committed history. A denial-propagation defect in `process_due_timers` (a denied retry incorrectly `?`-propagated as a `process_due_timers`-level failure) was found and fixed during implementation itself, before the accepted commit — confirmed resolved by the passing `a_retry_whose_capability_was_revoked_before_the_timer_fires_is_denied` test.

## 11. Deferred Work

Recorded here exactly as explicitly deferred by EWO-015 or its accepted review — none of the following is implemented, specified, or begun by this report:

- **Capability-declared retry-limit ceiling via `ConstraintSet`** — Phase 2, explicitly named and deferred by EWO-015 §7; `ConstraintSet` (`common/src/lib.rs`) remains a genuinely empty unit struct, untouched.
- **Explicit documentation of the `max_attempts` total-attempts-vs-retries-only convention** — an ADVISORY recommendation for whichever future EWO extends retry-limit representation, not a defect requiring correction now.
- **A dedicated two-independent-Runtime-instances replay-determinism test** — a MINOR, recommended (not required) test-coverage addition; `decide_retry`'s underlying purity is independently confirmed from source.
- **A broader Effect cleanup/eviction lifecycle for `retry_dispatch_material` and every other never-evicted Effect Coordinator correlation map** — accepted, disclosed, Runtime-session-scoped retention, consistent with every other correlation map in this crate; not itself an EWO-015 requirement, and not addressed here.

## 12. Final Assessment

EWO-015 (Retry Architecture Implementation) was faithfully implemented against ARCH-008 v0.4.3 §19.1–§19.4 and invariant 45, exactly as its own frozen specification requires. Independent verification, re-run for this report, passes completely (744 tests, 0 failures, fmt/clippy/build all clean, diff-check clean). Dependency and ownership discipline was preserved and, per the Independent Implementation Review's own direct source tracing, is now more robustly confirmed than before this milestone began. Fresh capability validation is preserved unconditionally on every retry dispatch — confirmed by direct inspection of the unmodified `admit_message` chain and a dedicated revocation test. The Independent Implementation Review accepted the implementation with zero CRITICAL and zero MAJOR findings. EWO-015 is closed.

## 13. Implementation Evidence

- Ending commit: `c5959bb5a2b26b816a17ab1bdc4e77c6183a8c22`
- Subject: `runtime: implement EWO-015 retry architecture`
- Starting commit: `4256b4434447fb9ab43d0d901a5baf8476c024e3`
- Files changed: `services/effect-coordinator/src/lib.rs`, `services/effect-coordinator/src/internal.rs`, `runtime/src/lib.rs` (1,685 insertions, 34 deletions total)

## 14. Closure Status

```text
EWO-015 IMPLEMENTATION ACCEPTED

ENGINEERING REPORT COMPLETE

EWO-015 CLOSED
```

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Claude (AI-assisted) | Initial and final report. Records the Retry Architecture Implementation exactly as completed and independently verified: `RetryIntent`/`RetryDecision`, `decide_retry`'s deterministic four-gate logic, operation/retry-intent retention, Runtime-owned dispatch-material retention, Timer-based scheduling and bidirectional correlation, due-timer recognition and re-dispatch, fresh capability admission, retry audit events, and actor-termination safety via the pre-existing `cancel_all_for_actor` mechanism — independently re-verified against source at authoring time (744 tests, 0 failures; diff-check clean). Records the Independent Implementation Review's zero-CRITICAL, zero-MAJOR, two-MINOR, two-ADVISORY outcome truthfully and proportionately, distinguishing repository-verifiable evidence from externally supplied review evidence throughout. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Claude (AI-assisted) | Drafted | 2026-07-28 |
| Reviewer | Independent Implementation Review (this engineering effort) | Accepted — zero Critical, zero Major, two Minor, two Advisory findings, no rework required | 2026-07-28 |
| Approval Authority | Denver Jacobs | **Approved** | 2026-07-28 |
