---
document_id: ER-017
title: "ConstraintSet-Based Retry Policy — Engineering Report"
version: 0.1.0
status: Published
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-28
last_updated: 2026-07-28
classification: Public
related_documents:
  reports_on: "EWO-016 (work-orders/EWO-016-ConstraintSet-Based-Retry-Policy.md, v0.1.2, Approved) — the governing Engineering Work Order this report records"
  architecture:
    - ARCH-008 (v0.4.3, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md) — §14, §19.4, §23.5, reused unmodified
    - ARCH-002 (Runtime Architecture) — §9 ("constructible only by Capability Authority"), reused unmodified
    - ARCH-001 (Constitutional Architecture) — the non-amplification rule this EWO's own narrowing enforcement implements
  adrs:
    - ADR-0017 (Approved — Bootstrap Capability Trust Root)
  standards:
    - STD-001
  predecessor: ER-016 (Retry Architecture Implementation — Engineering Report)
---

# ER-017 — ConstraintSet-Based Retry Policy — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or retroactively expand the scope of the EWO it reports against. It records the implementation exactly as independently verified.

**Numbering.** The highest existing Engineering Report on record is `ER-016-Retry-Architecture-Implementation.md`; no ER-017 exists yet. This document is therefore **ER-017**, derived directly from the repository's own contents (STD-001 §7), independently confirmed via `find engineering-reports -maxdepth 1 -type f` and a full-repository grep for "EWO-016" across `engineering-reports/` returning no prior report.

## 1. Executive Summary

EWO-016 (ConstraintSet-Based Retry Policy) completes the one input EWO-015's own Retry Decision mechanism deliberately left unconnected: `capability_limit: Option<u32>`, the parameter `EffectCoordinator::decide_retry` already accepted and already correctly combined with the requesting actor's own declared limit via `min(capability, actor)`, but which `Runtime::maybe_schedule_retry` supplied as a hardcoded `None` on every call. This milestone gives `ConstraintSet` its first concrete dimension — a capability-declared `RetryConstraint` (`max_total_attempts: NonZeroU32`) — together with real narrowing enforcement in Capability Authority's `issue` and `attenuate`, and a single-expression Runtime-side resolution from the already-retained `Capability` in `retry_dispatch_material`. The implementation is committed at Runtime commit `29ea55ced6348490f90bd7baeb08d3d4705f19ab`. An Independent Implementation Review (IIR-016) independently re-traced every required behavior to source — data-model immutability, narrowing atomicity, Runtime integration timing, mechanical-migration purity, Effect Coordinator/Timer/persistence non-interference — and concluded zero CRITICAL and zero MAJOR findings (one MINOR, no OBSERVATION), with the final outcome `IIR-016 COMPLETE — READY FOR ENGINEERING REPORT (ER-017)`. Validation independently re-run for this report passes completely: 770 tests, 0 failures, 0 warnings.

**Evidence basis, disclosed explicitly.** This report's implementation description, diff statistics, and validation results (§§3–8) are repository-verifiable — independently re-derived from the actual working tree and commit history during the authoring of this report, not restated from memory. The Independent Implementation Review's own findings and classification (§9) are recorded as supplied project evidence from that completed review; its full text is not itself a committed repository artifact, and is cited here as external session evidence, distinguished from the repository-verifiable facts surrounding it.

## 2. Authoritative Specification

`work-orders/EWO-016-ConstraintSet-Based-Retry-Policy.md`, version `0.1.2`, status **Approved**. Confirmed directly from its own frontmatter and Approval Status table at authoring time. EWO-016 was treated throughout implementation as a frozen implementation contract — every required type, method signature, and enforcement point (§§6–8) was realized as published. One correction is disclosed in §10 below: EWO-016's own earlier drafting had underestimated the migration scope (later corrected to 13 files during its own Independent Engineering Review, before implementation began), which the implementation itself independently re-confirmed rather than assumed.

## 3. Repository Baseline

**`synapse-runtime`:** path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `29ea55ced6348490f90bd7baeb08d3d4705f19ab`, tracking `origin/main`, 0 ahead / 0 behind, confirmed via `git fetch origin` immediately before authoring this report. Starting commit for the implementation range: `c5959bb5a2b26b816a17ab1bdc4e77c6183a8c22` (the EWO-015/ER-016 baseline). Working tree clean — no modification occurred during the authoring of this report.

**`synapse-docs`:** path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `1038e8ac4a1d84c009fcc8165f70e4c41a28bddc` at the start of this task, tracking `origin/main`, 0 ahead / 0 behind. Pre-existing, unrelated backlog left completely untouched: `standards/STD-001-Documentation-Standards.md` (modified, pre-existing drift), `.ai/`, `consolidation/ACR-001-Architecture-Consolidation-Review.md`, `consolidation/RSS-001-Research-Synthesis-Review.md`, `governance/GOV-002-Vision-and-Mission.md`, `maintenance/EMO-001-ActorHost-Single-Live-Instance.md`, `research/RES-001` through `RES-006`, `work-orders/EWO-003-Message-Gateway.md` (all untracked). `work-orders/EWO-016-ConstraintSet-Based-Retry-Policy.md` (v0.1.2, Approved) is the governing, unchanged EWO. This report (`engineering-reports/ER-017-ConstraintSet-Based-Retry-Policy.md`) is the only file this task adds to `synapse-docs`.

## 4. Implementation Scope

Thirteen files were modified — confirmed via `git diff --name-only c5959bb..29ea55c`, exactly matching EWO-016 §12's two-tier authorized list:

**Semantic implementation (4 files):**

```text
common/src/lib.rs
core/capability-authority/src/internal.rs
core/capability-authority/src/lib.rs
runtime/src/lib.rs
```

**Mechanical construction migration (9 files):**

```text
core/message-gateway/src/lib.rs
runtime/examples/actor_to_actor_messaging.rs
runtime/examples/worker_pool.rs
runtime/tests/actor_supervision.rs
runtime/tests/actor_to_actor_messaging.rs
runtime/tests/bootstrap_grant.rs
runtime/tests/bootstrap.rs
runtime/tests/timer.rs
runtime/tests/worker_pool.rs
```

`git diff --numstat` for this range, independently re-derived: `common/src/lib.rs` +81/−2; `core/capability-authority/src/internal.rs` +75/−36; `core/capability-authority/src/lib.rs` +319/−41; `runtime/src/lib.rs` +421/−40; the 9 mechanical files together +37/−37. **933 insertions, 156 deletions** across all thirteen files combined. No `Cargo.toml` anywhere in the workspace was modified — independently confirmed via `git diff --name-only` filtered for manifest files, returning nothing. No Documentation file was modified by the implementation itself; this report is the first Documentation-repository change associated with EWO-016 since its own publication.

## 5. Implemented Design

**Constraint data model** (`common/src/lib.rs`) — `ConstraintSet` gains one private, optional field: `retry: Option<RetryConstraint>`. `RetryConstraint { max_total_attempts: std::num::NonZeroU32 }`, deriving `Debug`/`Clone`/`Copy`/`PartialEq`/`Eq`, deliberately carrying no `Default` impl (no zero-equivalent default retry constraint exists — the "no constraint declared" state is `ConstraintSet.retry: None` exclusively). `ConstraintSet::with_retry(RetryConstraint)` is the sole constructor; `ConstraintSet::retry() -> Option<&RetryConstraint>` the sole accessor. `ConstraintSet::default()` continues to yield `retry: None`, identical in every observable respect to the prior zero-field unit value.

**Narrowing comparison** (`core/capability-authority/src/internal.rs`) — a private `retry_narrows(parent, child) -> bool` function, mirroring the existing `matches_canonical` pattern: an absent parent constraint permits any child value; a present parent constraint requires an equal-or-smaller child value and rejects an absent child value outright (per EWO-016 §7's complete-resulting-set semantics — omission means "no limit at all," not "inherit the parent's value").

**Issuance enforcement** — `issue` now applies `retry_narrows` as a second, independent non-amplification check alongside the existing `operations` subset check, both evaluated before anything is minted. No bootstrap-specific exemption was required for the retry dimension: the Bootstrap Capability's own retry constraint is unconstrained by construction (`ConstraintSet::default()`), which `retry_narrows` already permits universally — unlike `operations`, whose empty-by-construction Bootstrap value required an explicit exemption.

**Attenuation enforcement** — `attenuate`'s previously-unused `narrower: ConstraintSet` parameter is now genuinely consulted: rejected via `retry_narrows` before minting on a would-be widening; on success, the minted capability's `constraints` field is set to `narrower` directly — the child's complete resulting constraint set, never a merge with the parent's own value.

**Shared error semantics** — both enforcement points reuse the existing `RuntimeError::ExceedsIssuingCeiling` variant, already used for the `operations`-ceiling violation; no new error variant was introduced. Neither `issue` nor `attenuate` mutates the canonical parent/issuer record on a rejected attempt — the narrowing check runs strictly before the corresponding `self.issued.insert` call in both methods.

**Runtime integration** (`runtime/src/lib.rs`) — `Runtime::maybe_schedule_retry`'s previously hardcoded `let capability_limit = None;` is replaced by a single expression: the Effect's retained `Capability` is looked up in the existing `retry_dispatch_material` map (populated once, at original `request_effect` time, before any attempt or failure occurs — confirmed by direct trace, not assumed), its `constraints().retry()` is read, and `.max_total_attempts().get()` converts the resulting `NonZeroU32` to the `Option<u32>` `decide_retry` already accepts. An absent dispatch-material entry and an absent declared constraint both resolve to `None` — neither is a new failure mode. No second, duplicate storage of the resolved value was introduced anywhere.

**Unmodified downstream consumer** — `EffectCoordinator::decide_retry`'s signature and internal logic (the `min(capability_limit, actor_declared_limit)` composition, established by EWO-015) are untouched by this milestone; this EWO supplies real data to an already-correct, already-tested consumer.

## 6. Dependency and Ownership Discipline

Confirmed directly from source, not merely asserted: `services/effect-coordinator` was not touched at all by this commit — its crate dependencies (`synapse-common`, `synapse-timer` only) remain unaffected, and `decide_retry`'s public signature is byte-identical to its EWO-015 state. `services/timer` and `services/persistence` were likewise not touched. `Runtime::dispatch_retry`'s call to `admit_message` — the fresh, unconditional capability-validation path — is byte-identical to its pre-implementation state; independently confirmed by a direct diff search finding zero lines referencing either function name anywhere in the commit. Capability Authority remains the sole enforcement point for narrowing, on both dimensions (`operations`, `retry`) it now recognizes. No new capability operation string or authorization gate was introduced. No `Cargo.toml` in the workspace was modified.

## 7. Testing

**5 new `synapse-common` unit tests** (3 → 8): default-`ConstraintSet` carries no retry constraint; an explicitly declared constraint is retained and read back exactly; the minimum representable value (`1`) is accepted; larger values are retained; zero is confirmed inexpressible through the public API (`NonZeroU32::new(0) == None`).

**15 new `synapse-capability-authority` unit tests** (53 → 68): issuance with a retry constraint succeeds and retains the exact declared value; direct root issuance invents no numeric ceiling regardless of declared magnitude; unconstrained issuance is unaffected; all five rows of the attenuation narrowing table (unconstrained→unconstrained, unconstrained→constrained, equal, smaller, larger-rejected, unconstrained-child-rejected); operation-narrowing and retry-narrowing enforced together and independently; an operation-narrow/retry-widen combination fails atomically with the parent's own record left unmutated; a failed attenuation leaves the parent valid and unchanged; the attenuated child stores the complete supplied constraint set, not a merge with the parent's; an omitted parent constraint in the child is confirmed not to silently inherit.

**6 new `synapse-runtime` library tests** (293 → 299): a capability-only retry limit of `1` governs with no actor limit declared; a capability limit of `3` and an actor limit of `1` resolve to their minimum; a capability limit of `3` permits no more than three total attempts — proven via the identical `record_dispatched`/`record_failed`/`record_retry_scheduled` attempt-count-cycling pattern EWO-015's own `no_limit_from_either_source_permits_unlimited_retry` test already establishes, since a `FailingActor` instance cannot itself be redispatched to more than once through `step` (a pre-existing test-infrastructure constraint, not something this milestone changes or works around); two Effects, each holding a distinct capability with a distinct declared limit, resolve completely independently; `decide_retry`'s determinism is confirmed for a genuinely non-`None` capability limit for the first time; a retry scheduled under a constraint-bearing capability is still denied by fresh validation once that capability is revoked.

**Mechanical migration regression:** every pre-existing test across the 13 changed files continues to pass, in substance, unmodified — only the `ConstraintSet` construction syntax present in 9 of those files (37 sites) required the mechanical `ConstraintSet` → `ConstraintSet::default()` update; no test's own assertions, names, or intent were altered.

**Final total: 770 passed, 0 failed** (744 baseline + 26 net new: 5 + 15 + 6).

## 8. Verification

Independently re-run for this report, from the actual working tree, immediately before authoring:

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean, exit 0 |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace --all-targets --all-features` | Clean |
| `cargo test --workspace --all-targets --all-features` | **770 passed, 0 failed, 0 ignored** |
| Focused: `synapse-common` | **8 passed, 0 failed** |
| Focused: `synapse-capability-authority` | **68 passed, 0 failed** |
| Focused: `synapse-runtime --lib` | **299 passed, 0 failed** |
| `git diff --check c5959bb..29ea55c` | Clean, no whitespace errors |

Test total independently re-derived by direct summation across all 26 test binaries in this run, matching the Independent Implementation Review's own re-derivation exactly. The focused `synapse-common` total (**8**, not 53) is stated correctly here — see §9 below for the earlier misreporting this corrects.

## 9. Independent Implementation Review

An Independent Implementation Review (IIR-016) was conducted against Runtime commit `29ea55ced6348490f90bd7baeb08d3d4705f19ab`, independently tracing every required behavior to source rather than accepting the implementation's own report — including direct inspection of `retry_narrows`, `issue`, `attenuate`, `maybe_schedule_retry`, `dispatch_retry`, and `admit_message`, none of which were assumed correct without being read, plus a fresh, independent re-derivation of every workspace and focused test total.

**Findings: zero CRITICAL, zero MAJOR, one MINOR, zero OBSERVATION — none blocking.**

1. **MINOR** — the implementation report circulated ahead of independent review stated `cargo test -p synapse-common → 53 passed (48 lib + 5 new)`. Independent re-derivation found the true result to be **8 passed** (3 pre-existing — confirmed via `git show c5959bb:common/src/lib.rs | grep -c '#\[test\]'` — plus 5 new); the `48` figure belonged to a different crate's output, misattributed while reading the full workspace test log. This was a reporting-accuracy error only: the Runtime implementation itself was, and remains, correct; the workspace-wide aggregate (770 passed, 0 failed) was independently re-confirmed accurate throughout; only the one focused per-crate figure required correction. That correction is carried into this report's own §7–§8 above, and is not repeated here as a defect requiring rework.

**Resolution:** the Independent Implementation Review's own concluding statement: `IIR-016 COMPLETE — READY FOR ENGINEERING REPORT (ER-017)`.

## 10. Deviations and Refinements

**Migration scope corrected before implementation began, not during it.** EWO-016's own initial drafting (v0.1.0) claimed "31+ call sites... continue to compile... without modification" and authorized only 4 files. Its own Independent Engineering Review (IER-016, conducted before Founder Approval) independently disproved this by direct `grep` evidence (zero `ConstraintSet::default()` uses anywhere in the workspace; 131 bare-literal construction occurrences across 13 files) and corrected the draft to v0.1.1 before approval — expanding the authorized file list to the accurate two-tier, 13-file scope this implementation then executed against unchanged. This is disclosed here as a correction to the *specification*, made before implementation, not a deviation the implementation itself introduced.

**No implementation-time deviation from the approved EWO.** Every required type, signature, and enforcement point was realized exactly as EWO-016 v0.1.2 specifies; no additional design decision was required beyond what §§6–8 already resolved.

## 11. Deferred Work

Recorded here exactly as explicitly deferred by EWO-016 or its accepted review — none of the following is implemented, specified, or begun by this report:

- **Retry backoff, exponential backoff, jitter, absolute deadlines, total retry duration, retry budgets, circuit breakers** — explicitly out of scope (EWO-016 §5, §18).
- **Provider-specific, operation-specific, or failure-class-specific retry constraints** — explicitly out of scope.
- **Policy-source audit enrichment** (recording, in retry audit events, whether the actor or the capability supplied the effective limit) — a genuine, low-risk future enhancement, not required for this milestone's correctness.
- **Durable persistence of capability or retry-policy data, and restart/recovery behaviour of any kind** — explicitly out of scope; no `services/persistence` interaction exists or was added.
- **A generalized, multi-dimensional constraint-validation framework** extending beyond the one retry-constraint dimension this milestone requires — the other five dimensions `ConstraintSet`'s own doc comment names (time, usage count generally, budget, rate, scope, delegation depth) remain their own, separately authorized future milestones.
- **SynapseOS Control Centre policy editing or visualization** — explicitly out of scope.

## 12. Final Assessment

EWO-016 (ConstraintSet-Based Retry Policy) was faithfully implemented against ARCH-001's non-amplification rule and ARCH-002 §9, exactly as its own frozen specification requires. Independent verification, re-run for this report, passes completely (770 tests, 0 failures, fmt/clippy/build all clean, diff-check clean). Dependency and ownership discipline was preserved: Effect Coordinator, Timer, and Persistence remain completely untouched; fresh capability validation before retry dispatch is preserved unconditionally, confirmed by direct inspection of the unmodified `admit_message` chain and a dedicated revocation test against a constraint-bearing capability. The Independent Implementation Review accepted the implementation with zero CRITICAL and zero MAJOR findings — the single MINOR finding was a reporting-accuracy correction to a focused test total, not a defect in the Runtime implementation itself, and is fully reflected correctly in this report.

## 13. Implementation Evidence

- Ending commit: `29ea55ced6348490f90bd7baeb08d3d4705f19ab`
- Subject: `runtime: enforce capability-declared retry constraints (EWO-016)`
- Starting commit: `c5959bb5a2b26b816a17ab1bdc4e77c6183a8c22`
- Files changed: 13 total (4 semantic, 9 mechanical — see §4); 933 insertions, 156 deletions

## 14. Closure Status

```text
ENGINEERING REPORT COMPLETE

READY FOR FOUNDER ACCEPTANCE

EWO-016 NOT YET CLOSED
```

This report's own Approval Status (below) records `Pending` for Approval Authority: Founder Acceptance of the implementation this report records — the action that would close EWO-016 — is a distinct, subsequent governance step this report's own authoring and publication does not itself constitute or perform.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Claude (AI-assisted) | Initial and final report. Records the ConstraintSet-Based Retry Policy implementation exactly as completed and independently verified: `RetryConstraint`/`ConstraintSet` data model, narrowing enforcement in `issue`/`attenuate`, `retry_narrows`, Runtime-side `capability_limit` resolution from retained dispatch material, the 13-file (4 semantic + 9 mechanical) change scope — independently re-verified against source at authoring time (770 tests, 0 failures; diff-check clean). Records the Independent Implementation Review's zero-CRITICAL, zero-MAJOR, one-MINOR outcome truthfully and proportionately, including the corrected `synapse-common` focused test total (8, not the 53 an earlier implementation report misstated), distinguishing repository-verifiable evidence from externally supplied review evidence throughout. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Claude (AI-assisted) | Drafted | 2026-07-28 |
| Reviewer | Independent Implementation Review (this engineering effort) | Accepted — zero Critical, zero Major, one Minor finding (reporting correction only), no rework required | 2026-07-28 |
| Approval Authority | Denver Jacobs | Pending | |
