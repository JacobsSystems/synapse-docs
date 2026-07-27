---
document_id: ER-011
title: "Effect Runtime Foundation — Engineering Report"
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-27
last_updated: 2026-07-27
classification: Public
related_documents:
  reports_on: "No numbered Engineering Work Order exists for this milestone — see 'Numbering and Traceability Disclosure' below. The governing task was informally titled 'EWO-001 — Effect Runtime Foundation' within this engineering effort, but no file by that name, or any other number, was ever created; STD-001 §47 states an ER 'SHOULD' (not MUST) identify a governing EWO, and this report proceeds on that explicit basis."
  architecture:
    - ARCH-008 (v0.3.0, Approved for implementation planning — architecture/ARCH-008-Effect-Runtime-Architecture.md)
    - ARCH-002 (Runtime Architecture; Trusted Core and Replaceable-services model, unmodified)
    - ARCH-007 (Persistent Actor Architecture; execution-state exclusion precedent ARCH-008 §22 extends)
  adrs:
    - ADR-0015 (Audit Emitter Failure Semantics)
    - ADR-0016 (Trusted Core Interaction Rule)
    - ADR-0017 (Bootstrap Capability Trust Root)
  standards:
    - STD-001
  predecessor: None — the first Engineering Report for any Effect Runtime milestone
---

# ER-011 — Effect Runtime Foundation — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. It records the implementation exactly as independently verified. Nothing described here has been staged, committed, or pushed.

**Numbering and Traceability Disclosure.** The highest existing Engineering Report on record is `ER-010-Local-Actor-Supervision-Hardening.md`; no ER-011 exists yet. The next identifier is **ER-011**, derived directly from the repository's own contents (STD-001 §7: each document family numbered independently, sequentially) — not from any assumption about "EWO-001," which is not a governing document number here at all. The governing task for this milestone's implementation was informally labeled, within this engineering effort, "EWO-001 — Effect Runtime Foundation" — but this label was never realized as a numbered Engineering Work Order file: no `work-orders/EWO-0NN-Effect-Runtime-Foundation.md` (or any other number) exists anywhere in this repository, confirmed by direct search. **This is disclosed explicitly, not concealed.** The identifier "EWO-001" is, separately, already permanently assigned to an entirely unrelated, pre-existing milestone (`work-orders/EWO-001-Runtime-Bootstrap.md`, `engineering-reports/ER-001-Runtime-Bootstrap.md`), so this report is titled and numbered independently of that label to avoid any collision with, or misattribution against, that already-published historical record. This report instead cites ARCH-008 v0.3.0 directly as its governing specification (§2, §5) and the independent implementation review conducted within this same engineering effort as its governing verification (§7), consistent with the precedent this corpus already established for an analogous gap (`maintenance/EMR-002-Persistent-Actor-Durable-Deletion-Timer-Cleanup.md` §5, disclosing that `delete_actor_state` itself was introduced with no governing EWO of its own).

## 1. Repository Verification

**`synapse-runtime`:** path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `3b56551952b6dfe70ed557b6c0a5b74097734b3f`, tracking `origin/main`, 0 ahead / 0 behind, confirmed via `git fetch origin` immediately before authoring this report. Working tree modified by the implementation this report records: `Cargo.lock`, `Cargo.toml`, `runtime/Cargo.toml`, `runtime/src/lib.rs` (modified); `services/effect-coordinator/` (new, untracked — `Cargo.toml`, `README.md`, `src/lib.rs`, `src/internal.rs`). No other file in `synapse-runtime` was touched at any point.

**`synapse-docs`:** path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `946b139b474123d05dbaa07492f0da40c1aadda2`, tracking `origin/main`, 0 ahead / 0 behind, confirmed via `git fetch origin` immediately before authoring this report. Pre-existing, unrelated backlog left untouched throughout this and every preceding task this engineering effort: `standards/STD-001-Documentation-Standards.md` (modified), `.ai/`, `consolidation/ACR-001...`, `consolidation/RSS-001...`, `governance/GOV-002...`, `maintenance/EMO-001...`, `research/RES-001`–`RES-006`, `work-orders/EWO-003-Message-Gateway.md` (all untracked). `architecture/ARCH-008-Effect-Runtime-Architecture.md` (untracked, v0.3.0 — the governing specification for this report) is likewise unchanged since its own approval review. This report (`engineering-reports/ER-011-Effect-Runtime-Foundation.md`) is the only file this task adds to `synapse-docs`.

**Nothing staged, committed, or pushed in either repository at any point during this task.**

## 2. Purpose

**Why this milestone existed.** ARCH-008 (Effect Runtime Architecture, v0.3.0) was approved for implementation planning following its own three-stage correction and final approval review, all conducted within this same engineering effort. The milestone informally titled "EWO-001 — Effect Runtime Foundation" existed to begin realizing that approved architecture in code — specifically, to establish the constitutional foundation (identity, lifecycle, and the Effect Coordinator's own ownership boundary) the architecture requires before any retry, timer, or Provider-specific work could be meaningfully built on top of it.

**Relationship to ARCH-008.** The implementation this report records traces directly to ARCH-008 §9 (Ownership Model), §10 (Effect Coordinator), §13 (Effect Request and Result Flow), §15 (Effect Identity), and §16.1–§16.3 (the two-level lifecycle). No architectural decision was invented, reinterpreted, or extended by this implementation; every ownership boundary and lifecycle transition below is cited against the specific ARCH-008 clause it realizes (§5).

**Implementation objectives**, as bounded by the governing task: establish Effect Runtime infrastructure, the Effect Coordinator, `EffectId`, `EffectAttemptId`, the Effect lifecycle model, the Attempt lifecycle model, Runtime integration, and basic audit infrastructure — explicitly excluding retry execution, retry policies, timer scheduling, Provider-specific logic, external effects, compensation, distributed execution, advanced metrics, and optimization work, all reserved for later milestones.

## 3. Scope

**Implemented:**

- A new, narrow, non-Trusted-Core replaceable service, `synapse-effect-coordinator`, owning Effect and Effect Attempt identity and the complete two-level lifecycle state machine (ARCH-008 §10, §16.1–§16.3).
- `EffectId` and `EffectAttemptId` — opaque, Runtime-scoped identifiers (ARCH-008 §15).
- The Effect-level lifecycle (`Requested`, `Denied`, `InProgress`, `RetryScheduled`, `Accepted`) and the Attempt-level lifecycle (`Dispatched`, `Executing`, and five terminal outcomes: `Completed`, `Failed`, `Cancelled`, `TimedOut`, `ProviderLost`), with full illegal-transition rejection.
- Runtime integration: a new `effect_coordinator` field on `Runtime`, wired into `bootstrap_with_config`; four new public methods (`request_effect`, `complete_effect`, `fail_effect`, `cancel_effect`) reusing the existing, unmodified admission pipeline and Capability Authority.
- Basic audit infrastructure: six new `effect.*` audit events (`requested`, `denied`, `dispatched`, `completed`, `failed`, `cancelled`), using the existing, unmodified `AuditEvent` shape.

**Explicitly excluded**, matching the governing task's own stated bounds and ARCH-008's own deferrals (§33): retry execution and retry policy (§19); Timer Service integration for `record_timed_out`; Supervisor integration for `record_provider_lost`; any concrete Provider Actor implementation; compensation; distributed execution; Runtime Control API or Control Centre surfaces; any concrete concurrency mechanism for the forward-progress constraint (§13). Each of these is implemented in the architecture's own governing text as a deliberately deferred concern, not an oversight of this milestone (§8 below).

## 4. Implementation Summary

**Files created:**

- `services/effect-coordinator/Cargo.toml` — depends on `synapse-common` only.
- `services/effect-coordinator/src/lib.rs` — `EffectId`, `EffectAttemptId`, `AttemptOutcome`, `AttemptStatus`, `EffectStatus`, the `EffectCoordinator` trait, `EffectCoordinatorHandle`, 43 unit tests.
- `services/effect-coordinator/src/internal.rs` — `EffectCoordinatorImpl`, the concrete state machine.
- `services/effect-coordinator/README.md`.

**Files modified:**

- `Cargo.toml` (workspace) — added `services/effect-coordinator` as a member and `synapse-effect-coordinator` as a workspace dependency.
- `runtime/Cargo.toml` — added the `synapse-effect-coordinator` dependency.
- `runtime/src/lib.rs` — added the `effect_coordinator` field (wired into `bootstrap_with_config` and both test-only direct `Runtime` constructors); six new audit-event helper functions; `request_effect`, `complete_effect`, `fail_effect`, `cancel_effect`, and the shared private `effect_requester` helper; 13 new tests.
- `Cargo.lock` — updated automatically by the build; no manual edit.

**Major implementation decisions.** `Completed` and `Cancelled` are the only two attempt outcomes ARCH-008 §19 never treats as retry-eligible, so `record_completed`/`record_cancelled` each immediately, automatically accept the outcome as the owning Effect's own accepted logical terminal outcome; `Failed`, `TimedOut`, and `ProviderLost` — the three outcomes §19 does name as retry-eligible — each leave the Effect `InProgress`, awaiting a future retry-decision mechanism this milestone does not build. `request_effect` constructs an ordinary `Message` and dispatches it through the existing, private `admit_message` method — the identical function `submit_message` and `process_emitted_messages` already call — introducing no second admission or authorization path. Every admission-pipeline failure (capability denial, an unreachable provider, or any other rejection) is uniformly recorded as the Effect-level `Denied` outcome, since ARCH-008 §16.1 names no second Effect-level terminal state for a try that never produces an Attempt; this is a disclosed engineering reading of the architecture, not an invented state (§5, §9).

**Architectural compliance** is addressed in full in §5.

## 5. Architectural Compliance

| ARCH-008 requirement | Implementation | Evidence |
|---|---|---|
| Runtime remains sole orchestrator (§9) | `request_effect` mediates every Effect request through `admit_message`, the single existing admission function; the Effect Coordinator initiates no cross-component sequence of its own | `runtime/src/lib.rs`, `request_effect`; confirmed no second call site to `admit_message`-equivalent logic exists |
| Effect Coordinator is bookkeeping-only, non-Trusted-Core (§10) | `synapse-effect-coordinator` depends on `synapse-common` only — confirmed by its own `Cargo.toml` and by a direct search of its source for any reference to Capability Authority, Actor Host, Message Gateway, Execution Coordinator, Lifecycle Guardian, Supervisor, or Temporal Runtime, which returned nothing | `services/effect-coordinator/Cargo.toml`; grep of `services/effect-coordinator/src/` |
| Provider Actors remain ordinary actors (§11) | Every test "provider" is created via the existing `define_actor`/`create_actor_instance` path with no privileged treatment; no provider registry was introduced | `runtime/src/lib.rs` test module |
| Capability Authority remains the sole authorizer; fresh, never-cached validation (§9, §14) | Unchanged — `core/capability-authority` was not modified; `request_effect` presents its capability to the identical `admit_message` validation path every other message origin already uses | `git diff --stat` shows no `core/*` file touched |
| `EffectId` stable, `EffectAttemptId` minted only once authorization succeeds, `Denied` creates no attempt (§15, §16.1, §31 invariant 40) | `record_requested` mints one `EffectId` per logical request; `record_dispatched` — called only after `admit_message` succeeds — is the sole place an `EffectAttemptId` is minted; `record_denied` never touches the attempts map | `services/effect-coordinator/src/internal.rs`; independently traced during implementation review |
| `RetryScheduled` is Effect-level only, never reopens an attempt (§16.1, §31 invariant 41) | `record_retry_scheduled` clears the Effect's own current-attempt reference; the next `record_dispatched` call always mints a genuinely new `EffectAttemptId` | `services/effect-coordinator/src/internal.rs`, `record_retry_scheduled`; tests `scheduling_a_retry_after_a_failed_attempt_transitions_the_effect_and_clears_the_attempt`, `a_new_attempt_after_retry_scheduled_receives_a_fresh_id_never_reusing_the_old_one` |
| At most one terminal outcome per attempt; at most one accepted outcome per Effect (§16.2, §16.3, §31 invariants 9–10) | `record_terminal` rejects any attempt already terminal; `accept` rejects any Effect not genuinely `InProgress` with a terminal current attempt | Tests `completing_an_already_completed_attempt_is_rejected`, `an_effect_never_reaches_more_than_one_accepted_logical_terminal_outcome` |
| In-flight Effects are Runtime execution state, never actor domain state (§22) | `EffectCoordinatorHandle` is a `Runtime` field entirely disjoint from `PersistenceHandle` and from `Actor::export_domain_state()`; no code path connects them | Structural — confirmed by inspection; **not independently exercised by a dedicated test in this milestone** (§9, deferred observation) |
| No new Trusted Core component (§9, §31 invariant 26) | `synapse-effect-coordinator` is positioned as a replaceable service, parallel to `synapse-timer`/`synapse-supervisor`/`synapse-persistence`, never added to `TrustedCore` | `runtime/src/lib.rs` — `effect_coordinator` is a top-level `Runtime` field, not a `TrustedCore` field |

No architectural deviation was found in this milestone's own implementation.

## 6. Validation

**Baseline** (before this milestone's implementation began, from the immediately preceding, unrelated task in this engineering effort): 569 tests passing workspace-wide.

**Final validation**, independently re-run twice within this engineering effort (once at implementation completion, once again during the separate independent implementation review):

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean, exit 0 |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | **625 passed, 0 failed, 0 ignored** |

**Test totals:** 569 (baseline) + 43 (new, `synapse-effect-coordinator`) + 13 (new, `runtime/src/lib.rs`) = **625**, confirmed by direct summation of every `test result:` line `cargo test --workspace` reports, independently re-derived twice, not asserted from a single aggregate figure.

## 7. Independent Review

An Independent Implementation Review was conducted within this same engineering effort, immediately following implementation, treating the implementation as the sole subject of review and re-deriving every finding directly from source rather than from this report's own prior description of the work. The review independently re-executed all four validation gates (§6) from a forced-clean state and confirmed the identical result (625 passed, 0 failed). It traced, line by line, the Effect Coordinator's own state-machine logic (`record_dispatched`, `record_denied`, `record_terminal`, `record_retry_scheduled`, `accept`) and the Runtime-level integration (`request_effect`, `complete_effect`, `fail_effect`, `cancel_effect`, `effect_requester`) against ARCH-008 directly, not against this implementation's own documentation.

**Findings:** no BLOCKER findings; no MAJOR findings. Two MINOR findings and two OBSERVATIONS were recorded: (1) every admission-pipeline failure is mapped uniformly to the Effect-level `Denied` outcome, conflating ARCH-008 §18's distinct "Authorization denial" and "Admission failure" categories — disclosed here (§4, §5) and in the review as a defensible, non-silent reading, not a defect requiring correction; (2) no test explicitly demonstrates Effect Coordinator/Persistence Service non-interaction, unlike this corpus's own established precedent for equivalent non-interactions (e.g. `checkpointing_does_not_cancel_pending_timers`) — noted as a genuine, if minor, test-coverage gap (§5, §8). Neither finding required rework.

**Resolution:** both MINOR findings were accepted as disclosed, non-blocking observations rather than corrected in this milestone, consistent with the review's own explicit approval — reopening either would require either inventing a new Effect-level failure taxonomy ARCH-008 itself does not yet define, or adding scope (a persistence-interaction test) the reviewing task itself classified as a recommended future addition, not a required correction.

**Approval outcome:** `EWO-001 EFFECT RUNTIME FOUNDATION APPROVED FOR ENGINEERING REPORT` — the review's own exact, required concluding statement.

## 8. Deferred Work

The following are explicitly, deliberately deferred, matching both the governing task's own stated exclusions and ARCH-008's own deferrals (§33):

- **Retry execution and retry policy** (ARCH-008 §19) — the Effect Coordinator's own `record_retry_scheduled` and `accept` methods exist and are unit-tested, but no Runtime-level retry-decision mechanism calls them; deciding *whether* a failed attempt should be retried is explicitly out of this milestone's scope.
- **Timer Service integration** — `record_timed_out` exists and is unit-tested in isolation, but nothing in this milestone wires a real Temporal Runtime timeout to call it.
- **Supervisor integration for `ProviderLost`** — `record_provider_lost` exists and is unit-tested in isolation, but nothing in this milestone informs the Effect Coordinator when a real Supervisor restart/stop decision affects an in-flight attempt.
- **Provider Actor implementations of any kind** — no HTTP client, filesystem API, SQL driver, or any other concrete provider was designed or built; test "providers" are ordinary, behavior-free actors.
- **Compensation, distributed execution, Runtime Control API, Control Centre surfaces, advanced metrics, and optimization work** — none touched by this milestone, exactly as excluded.
- **A dedicated Effect Coordinator/Persistence Service non-interaction test** (§5, §7) — the separation is structurally real but not yet independently demonstrated by a test.

Each item above is reserved for its own future, separately authorized milestone; none represents an oversight of this one.

## 9. Engineering Assessment

**Architectural fidelity:** high. Every ownership boundary ARCH-008 §9/§10 establishes is preserved not merely by discipline but structurally — the Effect Coordinator crate's own dependency graph makes it impossible for it to authorize a capability, perform I/O, or supervise a Provider Actor, since it has no dependency path to any component that could do so (§5). No second admission or authorization path was introduced anywhere in the diff (§5, §7).

**Implementation quality:** the two-level lifecycle (Effect-level and Attempt-level, ARCH-008 §16.1–§16.3) is implemented completely, including the two corrections a prior architecture-review cycle specifically required (`RetryScheduled` as Effect-level-only; `Denied` as Effect-level-only, never minting an Attempt ID) — both independently re-verified against source during the review (§7), not merely asserted from this milestone's own documentation.

**Testing:** 56 new tests (43 Effect Coordinator, 13 Runtime integration), each independently confirmed during review to verify a genuine architectural property (illegal-transition rejection, identity uniqueness, denial-never-attempts, retry-never-reuses-an-identity, at-most-one-accepted-outcome, truthful audit ordering and attribution) rather than merely exercising code paths for coverage's own sake.

**Remaining risk:** low, and entirely confined to the two disclosed MINOR findings (§7) — neither touches Runtime ownership, capability authorization, or lifecycle correctness. The larger, structurally significant remaining risk is inherent to the milestone's own deliberate incompleteness (§8): retry, timeout, and Provider-loss reactions exist only as isolated, unit-tested state transitions with no live trigger yet, which is the intended, disclosed shape of a "foundation" milestone, not a defect in what was built.

## 10. Conclusion

EWO-001 (informally so titled; no numbered Engineering Work Order file exists, §"Numbering and Traceability Disclosure" above) successfully implemented the approved architectural foundation ARCH-008 v0.3.0 requires for the Effect Runtime: `EffectId`/`EffectAttemptId` identity, the complete two-level lifecycle state machine, the Effect Coordinator as a narrow, non-Trusted-Core replaceable service, Runtime integration reusing the existing admission pipeline and Capability Authority without modification, and basic audit infrastructure. The milestone is complete within its own explicitly bounded scope; validation succeeds completely (625 passed, 0 failed, zero warnings); the Independent Implementation Review found no BLOCKER or MAJOR defect and approved the implementation without requiring rework. The implementation is ready for publication, subject to whatever governance disposition Denver Jacobs applies to this and every other Draft document in this corpus.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-27 | Claude (AI-assisted) | Initial report. Records the Effect Runtime Foundation implementation: the `synapse-effect-coordinator` crate, `EffectId`/`EffectAttemptId`, the two-level Effect/Attempt lifecycle, Runtime integration (`request_effect`/`complete_effect`/`fail_effect`/`cancel_effect`), and basic audit infrastructure — independently re-verified against source and against a full workspace validation run (625 tests, 0 failures) both at implementation completion and during a separate Independent Implementation Review. Discloses the absence of any numbered governing Engineering Work Order and the resulting choice of ER-011 (not "ER-001") to avoid collision with the pre-existing, unrelated `ER-001-Runtime-Bootstrap.md`. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Claude (AI-assisted) | Drafted | 2026-07-27 |
| Reviewer | Independent Implementation Review (this engineering effort) | Approved, no rework required | 2026-07-27 |
| Approval Authority | Denver Jacobs | Pending | |
