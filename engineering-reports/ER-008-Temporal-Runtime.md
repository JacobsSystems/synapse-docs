---
document_id: ER-008
title: "Temporal Runtime — Engineering Report"
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-15
last_updated: 2026-07-15
classification: Public
related_documents:
  reports_on: EWO-008 (work-orders/EWO-008-Temporal-Runtime.md)
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.4.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
    - ARCH-004 (v0.1.0 — architecture/ARCH-004-Local-Actor-Supervision-Architecture.md) — component-placement and identity-model precedent EWO-008 follows; not itself amended or implemented by this EWO
    - ARCH-005 (v0.1.0 — architecture/ARCH-005-Temporal-Runtime-Architecture.md) — the sole architectural authority EWO-008 implements
  adrs:
    - ADR-0015
    - ADR-0016
    - ADR-0017
  standards:
    - STD-001
  predecessor: ER-007 (engineering-reports/ER-007-Local-Actor-Supervision.md)
---

# ER-008 — Temporal Runtime — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. It records the implementation exactly as independently verified; it does not redesign the implementation or revisit the architecture. Nothing described here has been committed, staged, or pushed.

## 1. Title

ER-008 — Temporal Runtime — Engineering Report.

## 2. Status

**Draft.** Not yet reviewed for publication. Reports on **EWO-008 v0.1.0** (Draft — not yet formally approved as a governance act, per its own Document Control table), which implements **ARCH-005 v0.1.0** (Draft). This report's own informational status under STD-001 §47 does not depend on either governing document's approval act having occurred; it records engineering work exactly as independently verified, as of this writing.

## 3. Executive Summary

Temporal Runtime — the first realization of ARCH-005 — is implemented, tested, and independently reviewed. A new, isolated `synapse-timer` service crate is composed by `Runtime` alongside the existing `Scheduler` and `Supervisor` services, outside Trusted Core. Timer registrations are keyed exclusively by `ActorId`; a fired timer produces an ordinary `Message` submitted through the existing, unmodified admission pipeline; capability validation occurs only at fire time; firing is driven exclusively by caller-supplied monotonic `Instant` values; and six new truthfully-ordered audit events extend the existing `AuditEvent` shape without a new field. All 517 workspace tests pass; `cargo fmt`, `clippy` (zero warnings), and `build` are clean; zero `unsafe`; both pre-existing demonstrations run unchanged. No Trusted Core crate, `synapse-scheduler`, or `synapse-supervisor` was modified.

**Independent review verdict: APPROVED WITH MINOR OBSERVATIONS.** One observation — `Runtime::terminate_actor_instance` does not proactively cancel a directly-terminated actor's pending timers, unlike the Supervisor-mediated Stop path — is a disclosed, tested, non-corrupting deviation from ARCH-005 §23's Stop/Terminated text and from EWO-008's own Bounded Design Decision 4, arising from a genuine tension between two of EWO-008's own constraints (§16, below). No rework is required for EWO-008; the review recommends a future Engineering Modification Order.

## 4. Repository Verification

- `synapse-runtime`: path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `5ccc7f9083a71adc6ee704b2322a701935765679`, `HEAD == origin/main`, 0 ahead / 0 behind. Nothing staged. Modified (tracked): `Cargo.lock`, `Cargo.toml`, `common/src/lib.rs`, `core/actor-host/*`, `core/capability-authority/*`, `core/execution-coordinator/*`, `examples/README.md`, `runtime/Cargo.toml`, `runtime/README.md`, `runtime/src/lib.rs`, `services/scheduler/*` — these predate EWO-008 (EWO-006/EWO-007 work, left uncommitted before this milestone began, exactly as ER-007 already recorded). Untracked, attributable to EWO-008 specifically: `services/timer/`, `runtime/tests/timer.rs`. Untracked, predating EWO-008: `services/supervisor/`, `runtime/examples/`, `runtime/tests/{actor_supervision.rs, actor_to_actor_messaging.rs, bootstrap_grant.rs, worker_pool.rs}`. A raw `git diff HEAD` conflates all three uncommitted milestones; this report attributes changes to EWO-008 specifically by direct source inspection and file-modification-time analysis (§10), not diff-stat alone — the same method ER-007 used for EWO-007.
- `synapse-docs`: path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `e90404baa5140ce9004839bc51921c789777e003`, `HEAD == origin/main`, 0 ahead / 0 behind. `standards/STD-001-Documentation-Standards.md` shows a pre-existing, unrelated unstaged modification (0.3.1 → 0.4.0, registering RSS/ACR/AFR document families) — unrelated to this milestone. ARCH-005 (mtime 2026-07-14 20:13) and EWO-008 (mtime 2026-07-14 20:19) both predate the earliest Temporal Runtime source file (`services/timer/Cargo.toml`, mtime 2026-07-14 20:27) — confirming neither was edited during or after implementation. This report is the only file this task adds to `synapse-docs`.

**Numbering.** The highest existing Engineering Report is ER-007 (`engineering-reports/ER-007-Local-Actor-Supervision.md`); no ER-008 exists yet. The next identifier is therefore **ER-008**, derived from the repository's own contents (STD-001 §7: each family numbered independently, sequentially, starting at 001) — not assumed. EWO-008's own metadata (`related_documents.reported_by`) independently names the same identifier and filename in advance: `ER-008 (engineering-reports/ER-008-Temporal-Runtime.md, not yet created)`.

**Filename.** `engineering-reports/ER-008-Temporal-Runtime.md`, per STD-001 §8 (`TYPE-NNN-Short-Descriptive-Title.md`) and §10 (ER documents belong under `engineering-reports/`), mirroring the short title EWO-008's own filename uses and the filename EWO-008's own metadata already predicted.

## 5. Standards Verification

STD-001 §47 requires an ER to record, at minimum: objective, implementation summary, validation performed, deviations from the authorizing EWO if any, architectural conformance, and recommendations. This report satisfies all six (§7–§8, §11–§12, §16, §17 below) and follows the structure and evidentiary style established by ER-007 — metadata fields, frontmatter ordering, the `related_documents.reports_on` convention, the "informational only" opening statement, and the closing disposition/revision-history sections are all carried forward unchanged. No new report format is invented.

## 6. Sources Read

ARCH-005 v0.1.0, EWO-008 v0.1.0, the complete independent EWO-008 implementation review (conducted immediately prior to this report, within this same engineering effort), ARCH-004 v0.1.0, ARCH-003 v0.4.0, ARCH-002 v0.2.0, ER-007, ADR-0015, ADR-0016, ADR-0017, and STD-001 (§7, §8, §10, §11, §47) for precedent and governing authority. `services/timer/src/{lib.rs,internal.rs}`, `services/timer/README.md`, `services/timer/Cargo.toml`, `runtime/src/lib.rs`, `runtime/README.md`, `runtime/Cargo.toml`, `runtime/tests/timer.rs`, and workspace `Cargo.toml` were re-confirmed directly against current source for this report. The independent review is treated as the authoritative verification source, per this task's own governing instruction; this report does not summarize a separate implementation report (none exists as a distinct artifact for this milestone) and cross-checks conclusions against source directly where restated below.

## 7. Objectives

Implement the first realization of ARCH-005 — Temporal Runtime Architecture: a new `synapse-timer` service component; Runtime-mediated firing-check orchestration reusing the existing, unmodified admission pipeline; `ActorId`-keyed timer identity with restart preservation and Stop-triggered removal; the full seven-state timer lifecycle (`Registered`/`Waiting` permissibly merged, ARCH-005 §12); fire-time-only capability validation; ordinary mailbox, causation, and audit treatment; and a monotonic-clock requirement with a deterministic test seam — exactly as EWO-008 authorizes, and no further.

## 8. Scope

**Architectural scope.** Bounded entirely by ARCH-005 v0.1.0 — the sole architectural authority for this milestone. No architecture document was amended, reinterpreted, or extended by this work.

**Implementation scope.** One new service crate (`synapse-timer`, `services/timer`), depending on `synapse-common` only; Runtime composition of it alongside the existing `Scheduler` and `Supervisor`; three new public `Runtime` methods (`register_timer`, `cancel_timer`, `process_due_timers`); Stop-path cancellation wiring inside `execute_stop_decision`; six new audit-event constructors on the existing, unmodified `AuditEvent` shape.

## 9. Explicit Exclusions

Confirmed, at implementation's end, to remain exactly as EWO-008 excludes: no persistence or durable timers; no workflow engine or effect-scheduling system; no message retry, redelivery, or acknowledgement protocol; no Supervisor restart-backoff or delayed/scheduled restart; no backoff, timing, or jitter mechanism; no cron, calendar, or recurring/repeating timers; no time-zone handling; no distributed clocks, remote timers, or clustering; no deadline propagation or change to `Message.deadline`/`ExecutionContext.deadline`; no mailbox persistence, dead-letter queue, event sourcing, or checkpoint/snapshot recovery; no `Actor`/`ActorHost`/`ExecutionCoordinator`/`LifecycleGuardian`/`MessageGateway`/`CapabilityAuthority`/`Scheduler`/`Supervisor`/`synapse-common` API redesign; no new `RuntimeError` variant (the existing `UnknownTarget` and `IllegalTransition` are reused); no scheduling DSL or generic timing-policy framework; no architecture, ADR, or STD-001 change. None of these appears anywhere in the diff.

## 10. Repository State

**Before implementation** (this milestone's own baseline, independently re-verified): `synapse-runtime` at the same HEAD as today, with the accumulated, already-reviewed uncommitted work of prior milestones (capability messaging, bounded mailboxes, bootstrap grants, local actor supervision — ER-007) present and untouched. Baseline workspace validation clean, 470 tests passing (ER-007 §9). `synapse-docs` at the same HEAD as today, with ARCH-005 and EWO-008 already present as the governing, already-authored, unmodified authorities for this task.

**After implementation:** `synapse-runtime`'s working tree contains this milestone's own changes (§14, "Files Changed") layered on top of that same pre-existing accumulated state — no prior work was altered, discarded, or overwritten. File-modification-time analysis confirms every Trusted Core, `synapse-scheduler`, and `synapse-supervisor` file's last edit (06:47–17:13 on 2026-07-14) predates the earliest Temporal Runtime source file (20:27 the same day). `synapse-docs` is unchanged except for this report.

**Repository integrity.** Both repositories remain on `main`, at the same HEAD as before this task, 0 ahead / 0 behind `origin/main`. Nothing staged in either repository. No destructive Git operation (`reset`, `clean`, `checkout -f`, force-push, branch deletion) was performed or is required. Protected pre-existing untracked files in `synapse-docs` (`.ai/`, `consolidation/`, `maintenance/`, `research/RES-001` through `RES-006`, `work-orders/EWO-003`, `work-orders/EWO-006`) remain present and untouched.

## 11. Implementation Summary

**Temporal Runtime service introduction.** `synapse-timer` is a new, minimal service crate depending on exactly `synapse-common` — no dependency on `synapse-api` or any other component crate (unlike `synapse-supervisor`, which requires `synapse-api` solely to construct a `Box<dyn Actor>` value; Temporal Runtime constructs no such value). It exposes a `Timer` trait (`register`, `actor_of`, `state_of`, `cancel`, `cancel_all_for_actor`, `mark_expired`, `query_due`, `mark_completed`, `mark_discarded`) implemented by `TimerImpl`, held behind the public `TimerHandle`. Internal state is a `HashMap<TimerId, Node>`, each `Node` carrying its owning `ActorId`, `fire_at: Instant`, message-construction data, and lifecycle state — no actor instance, mailbox, capability, or reference to any other component's state.

**Timer lifecycle.** `TimerState` implements six externally observable states — `Pending` (merging ARCH-005's `Registered` and `Waiting`, a merge ARCH-005 §12 explicitly permits), `Fired`, `Cancelled`, `Expired`, `Discarded`, `Completed` — with exactly ARCH-005 §12's legal transitions enforced by `internal.rs`'s own match arms; illegal transitions are rejected with `RuntimeError::IllegalTransition`/`UnknownTarget` without mutating existing state.

**Timer identity.** Registrations are keyed exclusively by `ActorId`; `synapse-timer` has no `ActorInstanceId` field anywhere. Restart preserves registrations as a free, code-free consequence of this keying — `execute_restart_decision` never touches Temporal Runtime. Stop proactively cancels registrations for the terminated `ActorId`, but only along the Supervisor-mediated `execute_stop_decision` path (§16).

**Delayed execution and admission reuse.** `Runtime::process_due_timers(now: Instant)` queries Temporal Runtime for due registrations (each transitioned to `Fired` inside `query_due`, sorted by `fire_at`, never registration or storage order), and for each: emits `timer.fired`; constructs an ordinary `Message` with `sender == destination == entry.actor` (a timer wakes its own owning actor) and Runtime-set causation (`MessageId("timer:{id}")`, distinguishable from ordinary message-to-message causation, never self-asserted by Temporal Runtime); resolves send authority via the existing `resolve_emitted_message_authority` against the actor's currently bound capabilities; and submits it through the existing, unmodified `admit_message` pipeline — the identical sequence `submit_message` and `process_emitted_messages` already use. On success: `message.admitted`, `mark_completed`, `timer.completed`. On admission failure: `message.rejected`, `mark_discarded`, `timer.discarded` — never routed to Supervisor.

**Capability interaction.** `resolve_emitted_message_authority` is invoked only inside `process_due_timers`, never inside `register_timer` — capability validation is fire-time-only, never cached, never performed by Temporal Runtime itself (which holds no capability-related dependency or state of any kind).

**Clock.** `synapse-timer`'s production code contains zero calls to `Instant::now()` or `SystemTime`; every operation is driven by a caller-supplied `std::time::Instant`. Firing order is determined by an explicit sort on `fire_at` inside `query_due`, never `HashMap` iteration order. Tests exercise the identical production code path with synthetic `Instant` offsets — no separate mock clock, no `#[cfg(test)]` branching in the firing logic itself.

**Runtime composition.** `Runtime` gained one new field, `timer: TimerHandle`, held alongside the existing `scheduler`/`supervisor` fields, outside `TrustedCore`, constructed in `bootstrap_with_config`. Three new public methods (`register_timer`, `cancel_timer`, `process_due_timers`) are thin, direct delegations, mirroring `register_supervision`'s own "narrowest possible public surface" precedent. `step()` and `run_until_idle()` are byte-for-byte unchanged and never invoke `process_due_timers` automatically — it is reachable only through its own explicit public call.

**Audit implementation.** Six new constructors on the unchanged `AuditEvent` shape (`event_type`, `actor`, `capability`, `message`): `timer.registered`, `timer.cancelled`, `timer.expired`, `timer.fired`, `timer.completed`, `timer.discarded`. Delivery outcome reuses the existing `message.admitted`/`message.rejected` events, exactly as ARCH-005 §20 requires ("not a new, parallel pair"). Ordering is truthful throughout: `timer.fired` is emitted before the resulting message is constructed or admitted; `timer.completed`/`timer.discarded` are emitted only after admission genuinely resolves.

**Stop / Restart semantics.** Restart requires, and received, no timer-specific code. Stop cancellation is wired into `execute_stop_decision` (the Supervisor-mediated Stop path, reachable with the owning `ActorId` already in scope) but not into the general-purpose public `Runtime::terminate_actor_instance`, which receives only an `ActorInstanceId` and has no path to the owning `ActorId` (§16).

**Testing additions.** 29 new unit tests in `synapse-timer`; 13 new unit tests in `runtime/src/lib.rs`; 5 new public-API-only integration tests in `runtime/tests/timer.rs`. No pre-existing test's assertions were modified.

## 12. Architecture Compliance

Verified directly against ARCH-005 v0.1.0, independently of any implementation narrative:

| ARCH-005 requirement | Compliance |
|---|---|
| Runtime remains sole composer | Confirmed — `synapse-timer` has zero dependency on any other component crate; every call to it originates from `Runtime`'s own methods |
| Temporal Runtime remains outside Trusted Core | Confirmed — held alongside `scheduler`/`supervisor`, outside `TrustedCore`; no clock/timer mechanism added to the Trusted Runtime Core table |
| Trusted Core untouched | Confirmed — zero `synapse-timer` reference in any of the seven Trusted Core crates; file-modification-time analysis places every Trusted Core file's last edit before this milestone's earliest file |
| Replaceable service model | Confirmed — `synapse-timer` positioned architecturally parallel to `Scheduler`/`Supervisor`, per ARCH-005 §9.1 |
| `ActorId`-keyed timer identity, never `ActorInstanceId` | Confirmed — `Node.actor: ActorId`; no `ActorInstanceId` field anywhere in the crate |
| Restart preserves registrations, no restart-specific code | Confirmed — `execute_restart_decision` never touches `self.timer`; proven end-to-end by `restart_preserves_a_pending_timer_which_later_fires_against_the_replacement_instance` |
| Stop proactively removes registrations | **Partial** — confirmed for the Supervisor-mediated Stop path only; not wired into `terminate_actor_instance` (§16) |
| Shared, unmodified admission pipeline; no alternate execution/dispatch/mailbox/admission path | Confirmed — `process_due_timers` reuses `resolve_emitted_message_authority` + `admit_message` byte-identically; proven by `overflow_remains_an_ordinary_admission_failure_for_a_fired_timers_message` |
| Fire-time-only capability validation, no cache, no duplicated logic | Confirmed — proven by `a_capability_revoked_after_registration_but_before_firing_causes_discard_not_completion` |
| Scheduler isolation (time-unaware) | Confirmed — zero "timer" references in `services/scheduler`; file predates this milestone entirely |
| Supervisor isolation (no Temporal Runtime interaction) | Confirmed — zero cross-references in either direction; `a_discarded_timer_never_reaches_supervision` proves discard never reaches supervision |
| Monotonic clock only; deterministic seam genuinely exercises production behavior | Confirmed — zero `SystemTime` anywhere; tests use the identical production code path with synthetic `Instant` values |
| Truthful audit ordering, existing `AuditEvent` shape, no new field | Confirmed — proven by `audit_ordering_for_a_successfully_delivered_timer_is_truthful` and `audit_ordering_for_a_discarded_timer_is_truthful` |
| Constitutional runtime invariants (sole composer, single admission pipeline, no reverse coupling) | Confirmed — `grep -rln synapse_timer` across the whole workspace returns exactly four files, all `runtime`'s own composition or `synapse-timer` itself |

No architectural deviation was found beyond the one disclosed exception recorded in §16.

## 13. Engineering Decisions

- Temporal Runtime was implemented as a new Runtime service crate (`synapse-timer`), parallel to `Scheduler` and `Supervisor`, outside Trusted Core, depending on `synapse-common` only — the direct, mechanical consequence of EWO-008's own Bounded Design Decision 1, not a genuinely open choice.
- Runtime performs all cross-component coordination; Temporal Runtime itself coordinates nothing and cannot reach any other component (structurally, by dependency graph).
- Timer registrations are owned exclusively by logical `ActorId` — never `ActorInstanceId` — making restart continuity a free consequence of identity, requiring no coordination protocol with Supervisor.
- Firing is driven exclusively by a caller-supplied monotonic `std::time::Instant`; production and test code share the identical code path, differing only in the supplied value (EWO-008 Bounded Design Decision 5).
- The deterministic testing seam is not a parallel mock: it is the production firing logic itself, exercised with synthetic `Instant` values — the strongest available realization of ARCH-005 §19's "single, substitutable notion of now."
- A fired timer's resulting message is admitted only through the existing, unmodified `admit_message` pipeline — no new admission logic of any kind exists in `synapse-timer` or in the new `Runtime` methods.
- Mailbox treatment for a timer-generated message is ordinary in every respect (bounded capacity, `Overflow` rejection, FIFO order) — no exemption, no timer-specific mailbox.
- Capability validation occurs only at fire time, through the existing `resolve_emitted_message_authority`, with no capability cache of any kind inside Temporal Runtime and no duplicated validation logic.
- No coupling was introduced between Temporal Runtime and Scheduler in either direction — Scheduler remains exactly as time-unaware as it already was lifecycle-unaware.
- No coupling was introduced between Temporal Runtime and Supervisor in either direction — the two replaceable services remain mutually unaware, exactly as ARCH-005 §18 requires.
- **Accepted implementation decision — internal state merge.** ARCH-005's seven-state lifecycle is represented internally with `Registered` and `Waiting` merged into a single `Pending` state, while every externally observable ARCH-005 semantic — that no state may claim firing before it has genuinely occurred, and that all legal/illegal transitions are preserved exactly — remains intact. This representation was independently reviewed and accepted: ARCH-005 §12 itself explicitly permits the merge ("this document does not require them to be distinguishable, only that neither claims firing has occurred before it truthfully has"), and the review found no externally observable ARCH-005 lifecycle state has effectively disappeared.

## 14. Testing Summary

**Baseline validation** (before this milestone's own implementation began, per ER-007's own final figures): `cargo fmt --all -- --check` clean; `cargo clippy --workspace --all-targets --all-features -- -D warnings` clean; `cargo build --workspace` clean; `cargo test --workspace` — 470 tests passing, 0 failed.

**Final validation** (independently re-run for this report):

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | **517 passed, 0 failed, 0 ignored** |
| `cargo tree --workspace` | 16 workspace members (+1 over the ER-007 baseline of 15); no cycle |
| `cargo tree -p synapse-timer` | Exactly `synapse-common` — no other dependency |
| `cargo run --example worker_pool` | Runs; output unchanged |
| `cargo run --example actor_to_actor_messaging` | Runs; output unchanged |
| `grep -r unsafe` (workspace, excluding `target/`) | Zero matches |

**Test totals, independently reconciled per crate/target:**

| Crate/target | Tests | Result |
|---|---|---|
| `synapse-actor-directory` | 1 | Pass |
| `synapse-actor-host` | 48 | Pass |
| `synapse-api` | 1 | Pass |
| `synapse-audit-emitter` | 3 | Pass |
| `synapse-audit-pipeline` | 1 | Pass |
| `synapse-capability-authority` | 53 | Pass |
| `synapse-common` | 3 | Pass |
| `synapse-execution-coordinator` | 30 | Pass |
| `synapse-host-adapter` | 11 | Pass |
| `synapse-lifecycle-guardian` | 32 | Pass |
| `synapse-message-gateway` | 14 | Pass |
| `synapse-persistence` | 1 | Pass |
| `synapse-runtime` unit tests (`src/lib.rs`) | 177 (164 pre-existing + 13 new) | Pass |
| `synapse-runtime` `src/main.rs` unit tests | 0 | Pass |
| `runtime/tests/actor_supervision.rs` | 5 | Pass |
| `runtime/tests/actor_to_actor_messaging.rs` | 2 | Pass |
| `runtime/tests/bootstrap.rs` | 16 | Pass |
| `runtime/tests/bootstrap_grant.rs` | 8 | Pass |
| `runtime/tests/timer.rs` (new) | 5 | Pass |
| `runtime/tests/worker_pool.rs` | 29 | Pass |
| `synapse-scheduler` | 19 | Pass |
| `synapse-supervisor` | 29 | Pass |
| `synapse-timer` (new) | 29 | Pass |
| Doc-tests (all crates) | 0 | Pass (none exist) |
| **Total** | **517** | **All pass, 0 failures** |

**Arithmetic reconciliation:** ER-007's own final total (470) + 29 (`synapse-timer`, new crate) + 13 (new `runtime/src/lib.rs` unit tests) + 5 (`runtime/tests/timer.rs`, new integration file) = **517**, matching the independently re-run total exactly. The 13-new-Runtime-unit-test figure is independently corroborated by the crate's own before/after count: 164 (ER-007's baseline) → 177 (this milestone) = 13. No numerical inconsistency was found requiring correction for this milestone.

**New unit tests (`synapse-timer`, 29):** registration, distinct-id assignment, duplicate-registration independence, unregistered-id reporting, cancellation (success, unregistered rejection, already-fired rejection), `cancel_all_for_actor` (actor-scoped, empty case), expiry (success, never-due-again, already-fired rejection), illegal-transition rejection without state mutation, `Fired → {Completed | Discarded}`, terminal-state re-cancellation rejection, due-detection (future, exact-now, twice-only-once, payload fidelity), ascending `fire_at` ordering regardless of registration order, deterministic firing under synthetic instants, wall-clock irrelevance to firing order, and structural state-isolation.

**New Runtime tests (13):** genuine end-to-end delivery through the admission pipeline; immediate expiry for an unreachable target; discard (not completion) when the target becomes unreachable after registration; discard when a bound capability is revoked between registration and firing (fire-time-only proof); ordinary `Overflow` admission failure for a fired timer's message; restart continuity delivering to the replacement instance; Stop-decision cancellation of every pending registration; public-API cancellation preventing delivery; independent firing of duplicate registrations for one actor; cross-actor isolation; truthful audit ordering for both the completed and discarded outcomes; and confirmation that a discarded timer never reaches supervision.

**New integration tests (5, `runtime/tests/timer.rs`, public API only):** genuine delivery to a real actor handler; capability-authorized delivery continuing to work after restart, through a genuine replacement instance; a Stopped actor's pending timer never delivering; `fire_at`-determined firing order under out-of-order registration; public-API cancellation preventing delivery.

**Regression preservation:** all pre-existing tests across every crate pass unmodified in behavior; both pre-existing demonstrations (`worker_pool`, `actor_to_actor_messaging`) run to completion with unchanged output; no existing test's own assertions were altered.

## 15. Independent Review Summary

An independent implementation review was conducted, re-validating every finding directly from source, tests, and runtime behavior rather than from any implementation report (none exists as a separate artifact for this milestone — the review is itself the primary verification act, per this task's own governing instruction). It independently repeated: `cargo fmt`, `cargo clippy`, `cargo build`, `cargo test --workspace`, `cargo tree` (workspace and `-p synapse-timer`), and both existing demonstrations. It independently verified: architecture compliance against every ARCH-005 normative decision (§23); every constitutional runtime invariant (sole composer, single admission pipeline, Trusted Core untouched, `ActorId` identity, replaceable-service placement, Scheduler/Supervisor mutual isolation, Runtime as sole cross-component coordinator) via dependency-graph inspection, content grep, and file-modification-time analysis rather than design intent; Trusted Core non-modification (mtime clustering plus a content grep for "timer" across every Trusted Core crate, finding zero matches); the `synapse-timer` dependency graph (exactly `synapse-common`, confirmed via `cargo tree -p synapse-timer`); the seven-state timer lifecycle and its legal-transition set (direct reading of `internal.rs`'s match arms); the admission pipeline's reuse (direct reading of `process_due_timers`/`resolve_emitted_message_authority`/`admit_message`, plus the overflow regression test); and audit ordering (direct reading of all six new event constructors plus both dedicated ordering tests).

**Final verdict recorded by the independent review: APPROVED WITH MINOR OBSERVATIONS.**

## 16. Approved Minor Observations

The following observations were accepted by the independent review. Each requires no corrective work for EWO-008:

1. **`Runtime::terminate_actor_instance` does not proactively remove timer registrations** because Runtime currently lacks a reverse `ActorInstanceId → ActorId` lookup, and EWO-008 simultaneously prohibited the Trusted Core change (a new Actor Host public method) that would expose one, while also bounding Runtime's own composition to exactly one new field. No corrective work required for EWO-008; future Engineering Modification Order candidate, once a reverse-identity-lookup mechanism is separately architecturally authorized. See §17 for the full engineering assessment of this observation.
2. **Equal `fire_at` timestamps currently have no deterministic tie-break ordering beyond `HashMap` iteration order.** ARCH-005 does not require tie-break determinism, and no test relies on cross-run tie-break behavior. No corrective work required for EWO-008; future hardening only, should deterministic same-instant ordering ever become a genuine requirement.
3. **Temporal Runtime currently performs actor self-wake only** (a fired timer's resulting message is always addressed to its own owning `ActorId`). This is a reasonable, EWO-permitted default, not itself an ARCH-005 requirement. No corrective work required for EWO-008; a future milestone extending delivery to a distinct target actor would require its own architectural authorization, not merely new implementation.
4. **Governing documents (ARCH-005, EWO-008) remain uncommitted in the documentation repository.** Process observation only — content was independently verified unchanged during implementation via file-modification-time evidence (§4, §10). No corrective work required for EWO-008.
5. **No dedicated Temporal Runtime example was added under `runtime/examples/`.** Not required by EWO-008 ("if a new Temporal Runtime demonstration is added, it must be deterministic... minimal, and limited to this EWO's own approved scope"); `runtime/tests/timer.rs`'s five public-API-only integration tests provide equivalent demonstration coverage. No corrective work required for EWO-008.

## 17. Engineering Assessment of the Major Observation

Observation 1 (§16) — `Runtime::terminate_actor_instance` not cancelling pending timers — was examined separately, in depth, by the independent review, which found:

- It is a **genuine architectural tension**, not an implementation error: satisfying EWO-008's own Bounded Design Decision 4 literally (wiring `terminate_actor_instance`) is not achievable without violating one of two other explicit constraints in the same document — the "exactly one new `Runtime` field" limit, or the prohibition on modifying any Trusted Core crate (a reverse lookup would require a new Actor Host public method).
- It **arises from competing approved constraints** within EWO-008 itself, not from a decision made outside the EWO's own text.
- It was **disclosed by the implementation** in three independent locations: `execute_stop_decision`'s own doc comment, `runtime/README.md`'s Temporal Runtime section, and a dedicated regression test (`a_timer_whose_actor_becomes_unreachable_before_firing_is_discarded_not_completed`).
- It was **independently verified** — both the gap itself (by call-site enumeration: `cancel_all_for_actor` is invoked exactly once, inside `execute_stop_decision`, never inside `terminate_actor_instance`) and its disclosed fallback behavior (by direct execution of the dedicated test).
- It **degrades safely to a truthful `Discarded` state**: a timer registered against an actor terminated only through `terminate_actor_instance` remains `Pending` and simply fails ordinary admission (`Discarded`, never `Completed`) whenever it next fires — exactly the outcome any other message addressed to a no-longer-live actor already receives.
- It **does not corrupt runtime state**: no panic, no silent loss, no orphaned state beyond a `Pending` registration that resolves itself truthfully at its own next fire attempt.
- It **does not violate constitutional runtime invariants**: the admission pipeline, capability validation, audit truthfulness, and Trusted Core boundaries all remain fully intact for this residual case.

The review recommends handling this through a future Engineering Modification Order rather than reopening EWO-008 — an EMO is the correct instrument under STD-001 §48 for a narrowly scoped conformance restoration against already-approved architecture and standards, once a reverse `ActorInstanceId → ActorId` lookup mechanism (or an architecturally equivalent alternative) is separately authorized.

## 18. Deferred Scope

The following remain out of scope for this milestone, exactly as ARCH-005 and EWO-008 already establish. This section records the scope boundary; it proposes no implementation for any of them:

- Durable timers and persistence of timer state across a Runtime-process restart.
- A workflow engine or generalized effect-scheduling system (Effect Runtime).
- A distributed runtime; remote timers; clustering.
- Message retry, redelivery, or acknowledgement protocols of any kind.
- Recurring or repeating timers; cron scheduling; calendar scheduling.
- Time-zone handling of any kind.
- Deadline propagation, or any change to the existing, unimplemented `Message.deadline`/`ExecutionContext.deadline` fields.
- Event sourcing; state snapshots; checkpoint recovery.
- Mailbox persistence or content transfer across restart.
- Dead-letter queues.
- Supervisor restart-backoff policy (ARCH-004 §4, §21 — unaffected, unresolved by this milestone).

## 19. Risks

| # | Risk | Classification |
|---|---|---|
| 1 | `terminate_actor_instance` does not cancel pending timers for a directly-terminated (non-supervised) actor | Major — disclosed, tested, non-corrupting (§17) |
| 2 | `query_due` tie-break for identical `fire_at` values is not deterministic across runs | Minor |
| 3 | Timer delivery is self-wake only | Observation |
| 4 | Governing documents uncommitted in `synapse-docs` | Observation (process, not engineering) |
| 5 | No dedicated Temporal Runtime example added | Observation |

No Critical findings. No scope deviations beyond Risk 1. No speculative or unauthorized future infrastructure was found anywhere in the diff.

## 20. Engineering Assessment

**Architecture compliance:** full, independently confirmed (§12), with one disclosed, narrowly-scoped, non-corrupting exception (§16–§17). **Engineering quality:** high — every change traces to a specific ARCH-005/EWO-008 clause, cited directly in source doc comments; no incidental refactor or unrelated change was found. **Code quality:** `synapse-timer`'s public surface is minimal (one trait, one handle type, four small data types); Runtime gained exactly one new field and three new public methods, each a thin delegation. **Testing quality:** high — new tests prove genuine behavior (fire-time-only capability validation, restart continuity through a real replacement instance, truthful audit ordering, ordinary overflow treatment), not coverage for its own sake. **Dependency quality:** optimal — `synapse-timer` depends on exactly `synapse-common`, confirmed by `cargo tree`, with zero reverse coupling from any other crate. **Runtime integrity:** preserved — all 517 tests pass, both existing demonstrations run unchanged, zero `unsafe`. **Trusted Core integrity:** preserved — zero modification, independently confirmed by file-modification-time analysis and content grep. **Maintainability:** the one disclosed limitation (§16–§17) is documented at three independent points a future maintainer would actually encounter, reducing the risk of it being mistaken for an oversight. **Future compatibility:** strong — persistence, workflow, distributed, and AI-orchestration milestones remain structurally unblocked; no accidental implementation decision constrains any of them.

**Overall:** a faithful, narrowly-scoped, well-tested realization of ARCH-005's first milestone, independently verified against source, tests, and runtime behavior, with one disclosed and safely-degrading limitation recommended for future EMO-level resolution.

## 21. Final Verdict

**Engineering Status: APPROVED WITH MINOR OBSERVATIONS**

Implementation accepted. No rework required. Observations recorded (§16–§17). Ready for publication.

## References

- ARCH-001 — Constitutional Architecture
- ARCH-002 — Runtime Architecture (v0.2.0)
- ARCH-003 — Runtime Integration Architecture (v0.4.0)
- ARCH-004 — Local Actor Supervision Architecture (v0.1.0)
- ARCH-005 — Temporal Runtime Architecture (v0.1.0)
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- STD-001 — Documentation Standards (§7, §8, §10, §11, §46, §47, §48)
- EWO-008 — Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution (work-orders/EWO-008-Temporal-Runtime.md)
- ER-007 — Local Actor Supervision — Engineering Report (engineering-reports/ER-007-Local-Actor-Supervision.md)
- `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree): `services/timer/src/{lib.rs,internal.rs}`, `services/timer/Cargo.toml`, `services/timer/README.md`, `runtime/src/lib.rs`, `runtime/README.md`, `runtime/Cargo.toml`, `runtime/tests/timer.rs`, root `Cargo.toml`
- Independent EWO-008 implementation review (this session; the authoritative verification source for this report, per its own governing instruction)

## Files Changed

| File | Change |
|---|---|
| `services/timer/Cargo.toml` (new) | New crate manifest; depends on `synapse-common` only |
| `services/timer/src/lib.rs` (new) | `Timer` trait, `TimerId`, `TimerState`, `DueTimer`, `TimerHandle`; 29 unit tests |
| `services/timer/src/internal.rs` (new) | `TimerImpl`: registration, cancellation, expiry, due-detection, lifecycle-transition enforcement |
| `services/timer/README.md` (new) | Responsibility, isolation, clock model, dependency rationale |
| `Cargo.toml` | Added `services/timer` workspace member and `synapse-timer` dependency alias |
| `runtime/Cargo.toml` | Added `synapse-timer` dependency |
| `runtime/src/lib.rs` | New `timer: TimerHandle` field; `register_timer`/`cancel_timer`/`process_due_timers` public methods; Stop-path cancellation wiring in `execute_stop_decision`; six new audit-event constructors; 13 new unit tests |
| `runtime/README.md` | New section describing the Temporal Runtime implementation and its one disclosed limitation |
| `runtime/tests/timer.rs` (new) | 5 public-API-only integration tests |
| `Cargo.lock` | One new package entry: `synapse-timer`; no new external dependency |

No Trusted Core crate (`core/capability-authority`, `core/actor-host`, `core/message-gateway`, `core/execution-coordinator`, `core/lifecycle-guardian`, `core/audit-emitter`, `core/host-adapter`) and no `services/scheduler` or `services/supervisor` file was modified for this milestone — confirmed by file-modification-time analysis and a content grep finding no "timer" reference in any of them.

## Final Git Status

`synapse-runtime` (`git status --short`, this milestone's own files):

```
?? runtime/tests/timer.rs
?? services/timer/
```

(`runtime/src/lib.rs`, `runtime/Cargo.toml`, `runtime/README.md`, `Cargo.toml`, and `Cargo.lock` show as modified relative to HEAD, but HEAD predates several already-reviewed prior milestones layered underneath this one — unaffected by this task, exactly as ER-007 already recorded for its own predecessor milestones.) Nothing staged. HEAD remains `5ccc7f9083a71adc6ee704b2322a701935765679`; `origin/main` unchanged.

`synapse-docs` (`git status --short`, this task's own files):

```
?? engineering-reports/ER-008-Temporal-Runtime.md
```

Nothing staged. HEAD remains `e90404baa5140ce9004839bc51921c789777e003`. ARCH-005 and EWO-008 were not modified by this report — confirmed by file-modification time, both predating this task. This report is the only file this task adds to `synapse-docs`.

## Confirmations

- **Runtime repository:** untouched by this report-authoring task — no source, test, or manifest file was modified while authoring ER-008. All runtime changes described above were already complete, tested, and independently reviewed before this report began.
- **Documentation repository:** changed only by the addition of this new Engineering Report. No other file in `synapse-docs` was created, modified, staged, or removed.
- **No commits, stages, or pushes occurred** in either repository during this task.

## Final Disposition

This report records completed, independently reviewed engineering work. It does not itself authorize, approve, publish, commit, or push anything, consistent with STD-001 §47's informational-only status for Engineering Reports. No further review or implementation was performed as part of authoring it.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-15 | Claude (AI-assisted) | Initial report following EWO-008 v0.1.0 implementation and its independent implementation review (verdict: APPROVED WITH MINOR OBSERVATIONS). Records the Temporal Runtime implementation, architecture compliance, testing (517 total tests, arithmetically reconciled against the ER-007 baseline of 470 + 29 + 13 + 5), the five approved minor observations, and the separate engineering assessment of the major observation (`terminate_actor_instance` timer-cancellation gap), recommended for future EMO-level resolution. |
