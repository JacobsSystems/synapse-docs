---
document_id: ER-007
title: "Local Actor Supervision — Engineering Report"
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-14
last_updated: 2026-07-14
classification: Public
related_documents:
  reports_on: EWO-007 (work-orders/EWO-007-Local-Actor-Supervision.md)
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.4.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
    - ARCH-004 (v0.1.0 — architecture/ARCH-004-Local-Actor-Supervision-Architecture.md) — the sole architectural authority EWO-007 implements
  adrs:
    - ADR-0015
    - ADR-0016
    - ADR-0017
  standards:
    - STD-001
  predecessor: ER-006 (engineering-reports/ER-006-Bounded-Actor-Mailboxes.md)
---

# ER-007 — Local Actor Supervision — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. It records the implementation exactly as independently verified; it does not redesign the implementation or revisit the architecture. Nothing described here has been committed, staged, or pushed.

## 1. Repository Verification

- `synapse-runtime`: path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `5ccc7f9083a71adc6ee704b2322a701935765679`, `HEAD == origin/main`, 0 ahead / 0 behind. Nothing staged. Modified (tracked): `Cargo.lock`, `Cargo.toml`, `runtime/Cargo.toml`, `runtime/README.md`, `runtime/src/lib.rs`, plus files predating this milestone (`common/src/lib.rs`, `core/actor-host/*`, `core/capability-authority/*`, `core/execution-coordinator/*`, `examples/README.md`, `services/scheduler/*`) — HEAD predates several already-completed, independently reviewed prior milestones (capability messaging, bounded mailboxes, bootstrap grants), all left uncommitted; a raw `git diff HEAD` therefore conflates that earlier work with this milestone's own changes, exactly as the independent implementation review itself noted. This report attributes changes to EWO-007 specifically by direct source inspection and file-modification-time analysis, not by diff-stat alone (§2, §5). Untracked (new): `services/supervisor/`, `runtime/tests/actor_supervision.rs` (plus `runtime/examples/`, `runtime/tests/{actor_to_actor_messaging.rs,bootstrap_grant.rs,worker_pool.rs}`, all predating this milestone and unrelated to it).
- `synapse-docs`: path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `e90404baa5140ce9004839bc51921c789777e003`, `HEAD == origin/main`, 0 ahead / 0 behind. `standards/STD-001-Documentation-Standards.md` shows an unstaged modification with mtime 2026-07-13 20:25 — predating this milestone's own implementation window (2026-07-14 17:13–19:38) and unrelated to it. ARCH-004 (mtime 2026-07-14 16:20) and EWO-007 (mtime 2026-07-14 16:43) both predate the earliest supervision source file (`services/supervisor/`, 2026-07-14 17:13) — confirming neither was edited during or after implementation. This report (`ER-007-Local-Actor-Supervision.md`) is the only file this task adds to `synapse-docs`.

**Numbering.** The highest existing Engineering Report is ER-006 (`engineering-reports/ER-006-Bounded-Actor-Mailboxes.md`); no ER-007 exists yet. The next identifier is therefore **ER-007**, derived from the repository's own contents (STD-001 §7: each family numbered independently, sequentially, starting at 001) — not assumed. EWO-007's own metadata (`related_documents.reported_by`) independently names the same identifier and filename in advance: `ER-007 (engineering-reports/ER-007-Local-Actor-Supervision.md, not yet created)`.

**Filename.** `engineering-reports/ER-007-Local-Actor-Supervision.md`, per STD-001 §8 (`TYPE-NNN-Short-Descriptive-Title.md`) and §10 (ER documents belong under `engineering-reports/`), mirroring the title EWO-007 itself uses and the filename EWO-007's own metadata already predicted.

## 2. Standards Verification

STD-001 §47 requires an ER to record, at minimum: objective; implementation summary; validation performed; deviations from the authorizing EWO, if any; architectural conformance; recommendations. This report satisfies all six (§4, §6–§8, §9, §11 below) and follows the structure and evidentiary style established by the immediately preceding ER-006 — the closest precedent, both chronologically and in kind (both close a milestone the independent review already approved, both correct a figure from the underlying implementation report). Metadata fields, frontmatter ordering, the `related_documents.reports_on` convention, the "informational only" opening statement, and the `Final Disposition`/`Revision History` closing sections are all carried forward unchanged from ER-006. No new report format is invented.

## 3. Sources Read

ARCH-004 v0.1.0, EWO-007 v0.1.0, the complete independent EWO-007 implementation review (conducted immediately prior to this report, within this same engineering effort), ARCH-002 v0.2.0, ARCH-003 v0.4.0, ADR-0015, ADR-0016, ADR-0017, STD-001 (§7, §8, §10, §11, §47), and ER-001, ER-002, ER-004, ER-005, ER-006 for precedent. `services/supervisor/src/{lib.rs,internal.rs}`, `services/supervisor/README.md`, `services/supervisor/Cargo.toml`, `runtime/src/lib.rs`, `runtime/README.md`, `runtime/Cargo.toml`, workspace `Cargo.toml`, and `runtime/tests/actor_supervision.rs` were re-read directly against the current source tree for this report — the independent review is treated as the authoritative verification source; this report cross-checks its conclusions against source directly rather than restating the original implementation report.

## 4. Engineering Summary

**Purpose.** Implement the first realization of ARCH-004 — Local Actor Supervision Architecture — closing the "Lifecycle Architecture: Supervision/restart policy" item ARCH-002 §15/§23 had deferred since Runtime's first implementation.

**Objectives.** Introduce a new, Runtime-composed Supervisor service; route only genuine `Actor::handle()` failures to it; implement Restart (same `ActorId`, new `ActorInstanceId`), Stop, Escalate, and Ignore; eliminate the pre-existing defect permitting a `Failed` instance's remaining queued mail to be repeatedly dequeued into further failures; preserve capability continuity through the existing `ActorId`-keyed binding structure; and do all of this without modifying any Trusted Core crate.

**Architectural scope.** Bounded entirely by ARCH-004 v0.1.0 — the sole architectural authority for this milestone. No architecture document was amended, reinterpreted, or extended by this work.

**Implementation scope.** One new service crate (`synapse-supervisor`); Runtime composition of it alongside the existing Scheduler; a single new routing call site inside `execute_message_capturing`'s dispatch-failure branch; unconditional Scheduler-eligibility withdrawal for a failed instance; six new audit-event constructors on the existing `AuditEvent` shape; one new public `Runtime` method (`register_supervision`).

**Explicit exclusions.** No reassignment or removal of an existing supervision relationship; no cascading action against a parent's own live instance on repeated child escalation; no mailbox transfer, retry, or dead-letter mechanism; no backoff, timing, or jitter policy; no distributed or remote supervision; no persistence — all confirmed, at implementation's end, to remain exactly as deferred (§16).

**Engineering outcome.** Implementation complete, tested, and independently reviewed. Verdict: **APPROVED WITH MINOR OBSERVATIONS** (§14). No corrective work required before this milestone is considered complete.

## 5. Repository State

**Before implementation** (this milestone's own baseline, re-verified by the independent review): `synapse-runtime` at the same HEAD as today, with the accumulated, already-approved uncommitted work of prior milestones (capability messaging, bounded mailboxes, bootstrap grants) present and untouched; workspace baseline validation clean (`fmt`, `clippy`, `build`) with 423 tests passing (§9). `synapse-docs` at the same HEAD as today, with ARCH-004 and EWO-007 already present as the governing, already-authored, unmodified authorities for this task.

**After implementation:** `synapse-runtime` working tree contains this milestone's changes (§8, "Files Changed") layered on top of that same pre-existing accumulated state — no prior work was altered, discarded, or overwritten (confirmed by file-modification-time analysis placing every Trusted Core and Scheduler file's last edit before this milestone's own earliest file, §7). `synapse-docs` is unchanged except for this report.

**Repository integrity.** Both repositories remain on `main`, at the same HEAD as before this task, 0 ahead / 0 behind `origin/main`. Nothing staged in either repository. No destructive Git operation (`reset`, `clean`, `checkout -f`, force-push, branch deletion) was performed or is required. Protected pre-existing untracked files in `synapse-docs` (`.ai/`, `consolidation/`, `maintenance/`, `research/RES-001` through `RES-006`, `work-orders/EWO-003`, `work-orders/EWO-006`) remain present and untouched.

## 6. Implementation Summary

**Supervisor service introduction.** `synapse-supervisor` is a new, minimal service crate depending on exactly `synapse-common` and `synapse-api` (the latter solely because a supervised actor's fresh initial behavior value is a `Box<dyn Actor>` — the same reason `synapse-actor-host` and `synapse-execution-coordinator` already depend on `synapse-api`). It holds a logical, `ActorId`-keyed node map (parent, restart count, optional behavior factory) and nothing else — no actor instance, mailbox, lifecycle state, or capability, and no call path to any other component.

**Runtime composition.** `Runtime` gained one new field, `supervisor: SupervisorHandle`, composed alongside the existing `scheduler` field, and one new public method, `register_supervision`, delegating directly to `Supervisor::register`.

**Failure routing.** A single new call, `route_actor_execution_failure`, is reached from exactly one place: the dispatch-`Err` arm of `execute_message_capturing`, and only once the existing cleanup sequence (`fail_active_execution`) has genuinely, truthfully completed — distinguished from an overriding cleanup failure by comparing the cleanup result against the original error value.

**Restart, Stop, Ignore, Escalation.** Restart terminates the failed instance and creates a replacement under the same `ActorId` with a fresh behavior value from Supervisor's stored factory. Stop terminates with no replacement. Ignore is Runtime's own interpretation of "not registered" — Supervisor is never even consulted for an unregistered `ActorId`. Escalation records one incremented failure against the supervising parent's own restart accounting and emits a truthful audit event; it takes no action of any kind against the parent's own live instance.

**Dispatch eligibility fix.** `route_actor_execution_failure` unconditionally withdraws the failed instance from Scheduler's ready set as its first action, regardless of registration — eliminating the pre-existing defect in which a `Failed` instance's remaining queued mail could be dequeued into further failures.

**Capability continuity.** Achieved with no new code: Capability Authority's existing bindings are keyed by `ActorId`, not `ActorInstanceId`, so a replacement instance inherits its predecessor's bindings automatically, and any revocation remains in force.

**Mailbox semantics.** Unchanged: `terminate_instance` discards the failed instance's mailbox exactly as before this milestone; nothing transfers pending messages to a replacement.

**Audit implementation.** Six new constructors on the unchanged `AuditEvent` shape (`event_type`, `actor`, `capability`, `message`); each supervision fact is carried by a distinct `event_type` string value, never a new field. Ordering is truthful throughout: no event claims a restart completed before the replacement instance genuinely exists.

**Testing additions.** 29 new unit tests in `synapse-supervisor`; 13 new unit tests in `runtime/src/lib.rs` (§9 corrects this figure); 5 new public-API-only integration tests in `runtime/tests/actor_supervision.rs`; one pre-existing regression test's assertions were strengthened, not weakened, to require the dispatch-eligibility defect's actual elimination.

## 7. Architecture Compliance

Verified directly against ARCH-004 v0.1.0, independently of the implementation report:

| ARCH-004 requirement | Compliance |
|---|---|
| Runtime remains sole composer | Confirmed — Supervisor is reached only through `Runtime`; it has no dependency on any other component's crate |
| Trusted Core untouched | Confirmed — no `core/*` crate gained a `synapse-supervisor` dependency or supervision-integration code; file-modification-time analysis places every Trusted Core file's last edit before this milestone's earliest file |
| Supervisor isolation | Confirmed — `synapse-supervisor` depends on exactly `synapse-common` and `synapse-api`; no reference to Scheduler, Lifecycle Guardian, Actor Host, Capability Authority, Execution Coordinator, or Message Gateway exists outside its own doc comments disclaiming such calls |
| `ActorId` identity model | Confirmed — Supervisor's node map, and every decision it renders, is keyed by `ActorId`, never `ActorInstanceId` |
| `ActorInstance` replacement | Confirmed — restart mints a new `ActorInstanceId` under the same `ActorId` via Actor Host's own existing sequence counter |
| Capability continuity | Confirmed — a direct, structural consequence of the existing `ActorId`-keyed binding map; proved end-to-end by an integration test using a genuine second actor |
| Failure routing | Confirmed — only the dispatch-`Err` arm (a genuine `Actor::handle()` failure) reaches supervision; every other failure class (admission, authorization, mailbox overflow, illegal lifecycle transition, audit-infrastructure failure) is structurally and behaviorally excluded |
| Audit ordering | Confirmed — truthful throughout; no completion claimed before it genuinely occurs |
| Mailbox semantics | Confirmed — no preservation, no transfer, no retry |
| Dispatch eligibility | Confirmed — unconditional withdrawal on failure, regardless of registration; the pre-existing defect is eliminated |
| Parent hierarchy | Confirmed — single-parent, `ActorId`-keyed, cycle-rejected at any ancestry depth, optional, never ambient |
| Escalation | Confirmed — bookkeeping against the parent's own accounting only; no action against the parent's own instance |
| Explicit exclusions | Confirmed — none of the excluded mechanisms (reassignment/removal, cascading escalation, backoff/timing, mailbox transfer) exist anywhere in the diff |

No architectural deviation was found. The one implementation deviation from EWO-007's literal text (§8) does not touch ARCH-004 itself.

## 8. Engineering Decisions

- Supervisor was implemented as a new Runtime service crate, parallel to Scheduler, outside Trusted Core.
- Runtime performs all cross-component coordination; Supervisor itself coordinates nothing.
- Scheduler remains lifecycle unaware; Lifecycle Guardian remains scheduler unaware — neither gained any new awareness of the other or of supervision.
- Failure routing is limited to `Actor::handle()` errors, reached from exactly one call site.
- A failed actor instance becomes non-dispatchable unconditionally, independent of whether it is supervised.
- The failed instance's mailbox is intentionally not preserved across restart.
- Capability continuity is achieved entirely through the existing `ActorId`-keyed binding structure — no new mechanism was introduced to provide it.
- Supervisor stores logical supervision state only: registration, hierarchy, restart accounting, and the decision rule — no actor instance, mailbox, lifecycle, or capability state.

**Engineering correction approved during independent review:** Supervisor depends upon `synapse-api`, in addition to `synapse-common`, because the approved fresh-behavior factory design requires producing a `Box<dyn Actor>` value, and `Actor` is defined in `synapse-api`. This narrows EWO-007's literal "depends on `synapse-common` only" phrasing but was found, on independent review, to be a necessary and precedented correction — matching the identical dependency combination `synapse-actor-host` and `synapse-execution-coordinator` already carry for the same reason — rather than architectural drift. It introduces no new coupling to Trusted Core or Scheduler and does not alter Supervisor's isolation from them.

## 9. Testing Summary

**Baseline validation** (before this milestone's own implementation began, per the independent review): `cargo fmt --all -- --check` clean; `cargo clippy --workspace --all-targets --all-features -- -D warnings` clean; `cargo build --workspace` clean; `cargo test --workspace` — 423 tests passing, 0 failed.

**Final validation** (independently re-run for this report):

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | 470 passed, 0 failed, 0 ignored |
| `cargo tree --workspace` | `synapse-supervisor` depends on exactly `synapse-common` + `synapse-api`; consumed only by `synapse-runtime` |
| `cargo run --example worker_pool` | Runs; output consistent with unchanged behavior |
| `cargo run --example actor_to_actor_messaging` | Runs; output consistent with unchanged behavior |

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
| `synapse-runtime` unit tests (`src/lib.rs`) | 164 (151 pre-existing + 13 new) | Pass |
| `synapse-runtime` `src/main.rs` unit tests | 0 | Pass |
| `runtime/tests/actor_supervision.rs` (new) | 5 | Pass |
| `runtime/tests/actor_to_actor_messaging.rs` | 2 | Pass |
| `runtime/tests/bootstrap.rs` | 16 | Pass |
| `runtime/tests/bootstrap_grant.rs` | 8 | Pass |
| `runtime/tests/worker_pool.rs` | 29 | Pass |
| `synapse-scheduler` | 19 | Pass |
| `synapse-supervisor` (new) | 29 | Pass |
| Doc-tests (all crates) | 0 | Pass (none exist) |
| **Total** | **470** | **All pass, 0 failures** |

**Correction to the implementation report's own figure:** the original implementation report stated 14 new Runtime unit tests. Direct, independent recount of the marked "Local actor supervision (ARCH-004; EWO-007)" section of `runtime/src/lib.rs` (line 6049 onward) finds exactly **13** new `#[test]` functions, not 14. This is corroborated arithmetically: 423 (baseline) + 29 (Supervisor) + 13 (Runtime) + 5 (integration) = **470**, matching the independently re-run total exactly — 423 + 14 + 29 + 5 would total 471, one more than the true figure. This report's own figures are the corrected ones, on the same basis ER-006 corrected the EWO-006 implementation report's test-count figures.

**New unit tests (Supervisor, 29):** registration (success, duplicate rejection, not-yet-registered parent, already-registered parent), self-supervision and cycle rejection (direct and deep-ancestry), parent/child lookup, Restart/Stop/Escalate/Ignore determinism, escalation accounting, fresh-behavior invocation, and no-mutation-on-rejection invariants.

**New Runtime tests (13):** restart identity and instance replacement; capability-binding inheritance and continued revocation; no ambient/fallback capability creation; mailbox non-preservation; Stop after allowance exhaustion with no parent; Escalate resolving the parent by `ActorId` without touching its instance; Ignore producing no supervision audit event; a queued message never being re-drained for an unregistered actor's failure; truthful restart-decision audit ordering; and three dedicated exclusion tests (admission failures, illegal lifecycle transition, mandatory audit-infrastructure failure) proving each never reaches supervision.

**New integration tests (5, `runtime/tests/actor_supervision.rs`, public API only):** restart with same `ActorId`/new `ActorInstanceId`; capability-authorized actor-to-actor messaging continuing to work after restart, through a genuine second actor; the failed incarnation's pending mailbox not preserved; an unsupervised actor's failure behaving exactly as before this milestone; a stopped supervised actor rejecting further messages with no live instance.

**Dispatch-defect regression:** `repeated_steps_after_handler_errors_do_not_panic_and_eventually_reach_idle` was rewritten — its prior assertions documented the defect (a second queued message was dequeued and also failed); its current assertions require `RuntimeStepOutcome::Idle` on the second step, proving the defect is eliminated. This is a strengthening of an existing regression test, not a new test and not a weakening.

**Capability continuity tests:** `replacement_instance_inherits_capability_bindings_automatically`, `a_revoked_capability_remains_revoked_after_replacement`, `restart_never_creates_ambient_or_fallback_capability_authority` (Runtime unit tests), and `capability_authorized_actor_to_actor_messaging_continues_to_work_after_restart` (integration test, the strongest proof — exercised end to end through a genuine second actor, not merely asserted internally).

**Audit ordering tests:** `restart_decision_audit_ordering_is_truthful`.

## 10. Independent Review Summary

An independent implementation review was conducted, re-validating every finding directly from source rather than accepting the original implementation report. It repeated: `cargo fmt`, `cargo clippy`, `cargo build`, `cargo test --workspace`, `cargo tree`, and both existing demonstrations. It independently verified architecture compliance (no forbidden call edge exists, confirmed by dependency-graph and call-site inspection rather than design intent); Trusted Core non-modification (file-modification-time analysis plus a content grep for "supervis" across every Trusted Core crate, finding only one pre-existing, pre-dating doc comment); the Supervisor dependency graph (exactly `synapse-common` + `synapse-api`, one new package in `Cargo.lock`); the dispatch-eligibility defect's elimination (direct reading of `route_actor_execution_failure` and the rewritten regression test, plus its own independent test run); restart mechanics (same `ActorId`, new `ActorInstanceId`, read directly from `execute_restart_decision`); capability continuity (structural proof plus the end-to-end integration test); and audit ordering (direct reading of all six new event constructors and the ordering test).

**Final verdict recorded by the independent review: APPROVED WITH MINOR OBSERVATIONS.**

## 11. Approved Minor Observations

The following observations were accepted by the independent review. Each requires no corrective work for EWO-007 — future hardening only:

- **Supervisor's dependency upon `synapse-api`.** A disclosed, necessary, precedented deviation from EWO-007's literal dependency text (§8). No corrective work required for EWO-007; future hardening only, if EWO-007's own text is ever revised to reflect this precedent explicitly.
- **The ambiguous-authority exclusion relies on structural proof (a single call site) rather than a dedicated behavioral test.** The exclusion is real and independently confirmed by code structure. No corrective work required for EWO-007; future hardening only, should a later change to `execute_message_capturing` introduce a second call site.
- **Restart-failure edge-case coverage could be expanded in a future hardening milestone** — specifically, the case where `terminate_instance` succeeds but the subsequent `create_instance_with_behavior` fails, leaving the actor with no live instance and no `restart_completed` event (an honest, disclosed degradation, not a silently swallowed error). No corrective work required for EWO-007; future hardening only.
- **The escalation audit event does not identify the supervising parent it reaches.** This is a disclosed limitation named explicitly in both ARCH-004 §22 and EWO-007's "Bounded Design Decision 5" — pre-authorized, not discovered as a gap during implementation. No corrective work required for EWO-007; future hardening only, should a future milestone extend `AuditEvent`'s own shape.

## 12. Deferred Scope

The following remain out of scope for this milestone, exactly as ARCH-004 and EWO-007 already establish. This section records the scope boundary; it proposes no implementation for any of them:

- Backoff, timers, retries, and retry policies of any kind.
- Mailbox transfer across restart; durable mailboxes.
- Persistence of any kind.
- A workflow engine or effect system.
- A distributed runtime; remote supervision; clustering; node monitoring.
- General reassignment or removal of an existing supervision relationship.
- Cascading supervision policies (any automatic action against a parent's own instance triggered by repeated child escalation).
- State snapshots; event sourcing.
- Dead-letter queues.

## 13. Final Engineering Assessment

**Engineering quality:** high — every change traces to a specific ARCH-004/EWO-007 clause, cited directly in source doc comments; no incidental refactor or unrelated change was found. **Architecture compliance:** full, independently confirmed (§7). **Code quality:** Supervisor's public surface is minimal (one enum, one trait, one handle type); Runtime gained exactly one new public method. **Testing quality:** high — new tests prove genuine behavior (restart identity, capability continuity through a real second actor, mailbox loss, decision determinism), not coverage for its own sake. **Regression quality:** the one modified pre-existing test was strengthened to require the targeted defect's actual elimination; no test was weakened to accommodate the implementation. **Runtime integrity:** preserved — all 470 tests pass, both existing demonstrations run unchanged. **Trusted Core integrity:** preserved — zero modification, independently confirmed by file-modification-time analysis and content inspection. **Scope discipline:** exact — no reassignment/removal, no cascading escalation, no backoff, no new example, nothing beyond what EWO-007 authorized. **Maintainability:** the disclosed, narrow `synapse-api` dependency and the escalation audit's parent-identity limitation are both clearly documented at their point of relevance (crate README, doc comments), reducing the risk of a future maintainer mistaking either for an oversight.

**Overall:** a faithful, narrowly-scoped, well-tested realization of ARCH-004's first milestone, independently verified against source rather than accepted on report alone.

## 14. Final Verdict

**Engineering Status: APPROVED WITH MINOR OBSERVATIONS**

Implementation accepted. No rework required. Observations recorded (§11). Ready for publication.

## 15. Files Changed

| File | Change |
|---|---|
| `services/supervisor/Cargo.toml` (new) | New crate manifest; depends on `synapse-common`, `synapse-api` |
| `services/supervisor/src/lib.rs` (new) | Public trait `Supervisor`, `SupervisionDecision` enum, `SupervisorHandle`; 29 unit tests |
| `services/supervisor/src/internal.rs` (new) | `SupervisorImpl`: registration, ancestry/cycle detection, restart accounting, decision rule (`RESTART_ALLOWANCE = 3`) |
| `services/supervisor/README.md` (new) | Responsibility, isolation, decision rule, dependency rationale |
| `Cargo.toml` | Added `services/supervisor` workspace member and `synapse-supervisor` dependency alias |
| `runtime/Cargo.toml` | Added `synapse-supervisor` dependency |
| `runtime/src/lib.rs` | New `supervisor` field; `register_supervision` public method; `route_actor_execution_failure`, `execute_restart_decision`, `execute_stop_decision` private methods; six new audit-event constructors; one pre-existing regression test strengthened; 13 new unit tests |
| `runtime/README.md` | One new paragraph describing the supervision implementation (existing content otherwise unchanged) |
| `runtime/tests/actor_supervision.rs` (new) | 5 public-API-only integration tests |
| `Cargo.lock` | One new package entry: `synapse-supervisor`; no new external dependency |

No Trusted Core crate (`core/capability-authority`, `core/actor-host`, `core/message-gateway`, `core/execution-coordinator`, `core/lifecycle-guardian`, `core/audit-emitter`, `core/host-adapter`) and no `services/scheduler` file was modified for this milestone — confirmed by file-modification-time analysis and a content grep finding no supervision-integration code in any of them (§7).

## 16. Final Git Status

`synapse-runtime` (`git status --short`, this milestone's own files):

```
 M Cargo.lock
 M Cargo.toml
 M runtime/Cargo.toml
 M runtime/README.md
 M runtime/src/lib.rs
?? runtime/tests/actor_supervision.rs
?? services/supervisor/
```

(Additional modified/untracked entries reported by `git status` belong to prior, already-approved milestones left uncommitted before this task began — unaffected by this task.) Nothing staged. HEAD remains `5ccc7f9083a71adc6ee704b2322a701935765679`; `origin/main` unchanged.

`synapse-docs` (`git status --short`, this task's own files):

```
?? engineering-reports/ER-007-Local-Actor-Supervision.md
```

Nothing staged. HEAD remains `e90404baa5140ce9004839bc51921c789777e003`. ARCH-004 and EWO-007 were not modified by this report — confirmed by file-modification time, both predating this task. This report is the only file this task adds to `synapse-docs`.

## 17. Confirmations

- **Runtime repository:** untouched by this report-authoring task — no source, test, or manifest file was modified while authoring ER-007. All runtime changes described above were already complete, tested, and independently reviewed before this report began.
- **Documentation repository:** changed only by the addition of this new Engineering Report. No other file in `synapse-docs` was created, modified, staged, or removed.
- **No commits, stages, or pushes occurred** in either repository during this task.

## Final Disposition

This report records completed, independently reviewed engineering work. It does not itself authorize, approve, publish, commit, or push anything, consistent with STD-001 §47's informational-only status for Engineering Reports. No further review or implementation was performed as part of authoring it.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-14 | Claude (AI-assisted) | Initial report following EWO-007 v0.1.0 implementation and its independent implementation review (verdict: APPROVED WITH MINOR OBSERVATIONS). Corrects the original implementation report's Runtime unit-test-count figure (14 → 13 new tests), independently re-derived and arithmetically reconciled against the full workspace total (423 baseline + 29 + 13 + 5 = 470). |
