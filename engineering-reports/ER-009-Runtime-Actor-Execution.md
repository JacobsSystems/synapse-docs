---
document_id: ER-009
title: "Runtime Actor Execution — Engineering Report"
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-15
last_updated: 2026-07-15
classification: Public
related_documents:
  reports_on: EWO-009 (work-orders/EWO-009-Runtime-Actor-Execution.md)
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.4.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
    - ARCH-006 (v0.1.3 — architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md) — the sole architectural authority EWO-009 implements
  adrs:
    - ADR-0015
    - ADR-0016
    - ADR-0017
  standards:
    - STD-001
  predecessor: ER-008 (engineering-reports/ER-008-Temporal-Runtime.md) — the highest-numbered Engineering Report existing before this one, per STD-001's sequential-numbering rule (§7); **not** a milestone-chronology predecessor — see §2 and §4 for the historical-reconstruction distinction this report inherits from ARCH-006 and EWO-009
---

# ER-009 — Runtime Actor Execution — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. It records the implementation exactly as independently verified; it does not redesign the implementation, revisit the architecture, or reopen any governance discussion already resolved. Nothing described here has been committed, staged, or pushed.

## 1. Title

ER-009 — Runtime Actor Execution — Engineering Report.

## 2. Status

**Draft.** Not yet reviewed for publication. Reports on **EWO-009 v0.1.3** (Draft — not yet formally approved as a governance act, per its own Document Control table), which implements **ARCH-006 v0.1.3** (Draft). This report's own informational status under STD-001 §47 does not depend on either governing document's approval act having occurred; it records engineering work exactly as independently verified, as of this writing.

**This report is itself a historical reconstruction**, on the same basis ARCH-006 and EWO-009 already are — see §5, below, for the required explicit acknowledgement. The implementation this report records was completed, independently reviewed, and superseded by two later milestones (EWO-007, EWO-008 — each already reported by ER-007 and ER-008) *before* ARCH-006, EWO-009, or this report existed. This report's `predecessor` field names ER-008 only because STD-001's numbering is sequential by document-authoring order, not milestone-chronology order; the milestone this report documents chronologically precedes both ER-007's and ER-008's own subject matter.

## 3. Executive Summary

Runtime Actor Execution — the first realization of genuine, non-simulated `Actor::handle()` invocation in SynapseOS — is implemented, tested, and has now been independently reviewed a total of six times across this engineering effort (§8). Execution Coordinator's `dispatch` genuinely invokes actor-owned behavior and returns its emitted messages; those messages are treated as fresh, independently authorized admission requests — never already-sent facts — resolved through the identical, single, Runtime-owned admission pipeline (`admit_message`) external submission already uses; causation and authority are Runtime-owned and non-forgeable; and a bounded, disclosed bootstrap-grant mechanism gives a genuinely external, public-API-only caller its first capability. Three of seven Trusted Core components (Actor Host, Capability Authority, Execution Coordinator) gained narrow, disclosed additions; Scheduler — a replaceable service, not Trusted Core — received its own first genuine implementation. All 517 current workspace tests pass; `cargo fmt`, `clippy` (zero warnings), and `build` are clean; zero `unsafe`; both dedicated demonstrations (`worker_pool`, `actor_to_actor_messaging`) run to completion with correct output.

**Independent review verdict (Final Independent Implementation Review, Publication Candidate): APPROVED.** No deviation from EWO-009 was found. Three governance-accuracy defects were found and corrected across the review chain that preceded this report (§8) — none affected implementation correctness; all concerned the truthfulness of the *documentation* describing an implementation that was, throughout, already correct.

## 4. Repository Verification

- `synapse-runtime`: path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `5ccc7f9083a71adc6ee704b2322a701935765679`, `HEAD == origin/main`, 0 ahead / 0 behind. Nothing staged. Modified (tracked): `Cargo.lock`, `Cargo.toml`, `common/src/lib.rs`, `core/actor-host/*`, `core/capability-authority/*`, `core/execution-coordinator/*`, `examples/README.md`, `runtime/Cargo.toml`, `runtime/README.md`, `runtime/src/lib.rs`, `services/scheduler/*` — this milestone's own contribution, layered together with EWO-006's, EWO-007's, and EWO-008's own uncommitted work in the same flat diff against HEAD. Untracked, attributable to this milestone specifically: `runtime/examples/{worker_pool,actor_to_actor_messaging}.rs`, `runtime/tests/{bootstrap_grant,worker_pool,actor_to_actor_messaging}.rs`. Untracked, predating or postdating this milestone: `services/supervisor/`, `services/timer/`, `runtime/tests/{actor_supervision,timer}.rs`. A raw `git diff HEAD` conflates all four uncommitted milestones (EWO-006, this milestone, EWO-007, EWO-008); this report attributes changes to this milestone specifically by direct source inspection and cross-reference against EWO-009's own "Scope," "Trusted Core," and "Required Interface Evolution" sections — themselves independently re-verified against source three times across this engineering effort's review chain (§8) — not by diff-stat or file-modification-time alone, since `runtime/src/lib.rs` was subsequently touched again by both successor milestones.
- `synapse-docs`: path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `e90404baa5140ce9004839bc51921c789777e003`, `HEAD == origin/main`, 0 ahead / 0 behind. `standards/STD-001-Documentation-Standards.md` shows a pre-existing, unrelated unstaged modification, predating this entire engineering effort. ARCH-006 (v0.1.3) and EWO-009 (v0.1.3) are both present, both already carrying the corrections from every prior review in the chain (§8). This report is the only file this task adds to `synapse-docs`.

Nothing has changed in either repository since the approved Final Independent Implementation Review (Publication Candidate) — confirmed by identical `git status --short` output and, for `synapse-runtime`, an identical `git status --short | md5sum` fingerprint (`04e39ef24f5b8fbcbb61eba8cbccc5f2`) to every checkpoint recorded across that review and this one.

**Numbering.** The highest existing Engineering Report is ER-008 (`engineering-reports/ER-008-Temporal-Runtime.md`); no ER-009 exists yet. The next identifier is therefore **ER-009**, derived from the repository's own contents (STD-001 §7: each family numbered independently, sequentially, starting at 001) — not assumed. EWO-009's own metadata (`related_documents.reported_by`) independently names the same identifier and filename in advance: `ER-009 (engineering-reports/ER-009-Runtime-Actor-Execution.md, not yet created)`.

**Filename.** `engineering-reports/ER-009-Runtime-Actor-Execution.md`, per STD-001 §8 (`TYPE-NNN-Short-Descriptive-Title.md`) and §10 (ER documents belong under `engineering-reports/`), mirroring the short title EWO-009's own filename uses and the filename EWO-009's own metadata already predicted.

## 5. Historical Reconstruction Acknowledgement

Per EWO-009's own "Required Completion Report Contents" item 7: **this report, EWO-009, and ARCH-006 are all historical reconstructions.** The implementation this report documents predates all three governing documents; it was built, ran, and was first independently reviewed before any of ARCH-006, EWO-009, or this report existed. This report does not authorize, and did not require authorizing, any implementation work — it records what was already, verifiably, complete. Every date, test-count figure, and source citation below reflects the implementation's state as it already stood before this report was authored.

## 6. Standards Verification

STD-001 §47 requires an ER to record, at minimum: objective, implementation summary, validation performed, deviations from the authorizing EWO if any, architectural conformance, and recommendations. This report satisfies all six (§9–§10, §13–§14, §19, §20 below) and follows the structure and evidentiary style established by ER-007/ER-008 — metadata fields, frontmatter ordering, the `related_documents.reports_on` convention, the "informational only" opening statement, and the closing disposition/revision-history sections are all carried forward unchanged. No new report format is invented. This report additionally satisfies EWO-009's own "Required Completion Report Contents" (seven items, §"Required Completion Report Contents" of that document), which is stricter than STD-001's own minimum and is the controlling checklist for this specific report.

## 7. Sources Read

ARCH-006 v0.1.3, EWO-009 v0.1.3, the Runtime Actor Execution Architecture Review, the Architecture Reconstruction Review ("Capability-Authorized Actor-to-Actor Messaging Runtime"), the Publication Recovery Review, both Independent Implementation Reviews of ARCH-006/EWO-009 (Scheduler correction; `dequeue`/`create_instance_with_behavior`/`fail` correction), the Runtime Actor Execution Implementation Inventory, the Governance Coverage Reconciliation, the Historical Provenance Audit, the Historical Provenance Corrections, the Final Independent Implementation Review (Publication Candidate — verdict APPROVED), ARCH-001, ARCH-002 v0.2.0, ARCH-003 v0.4.0, ADR-0015, ADR-0016, ADR-0017, and STD-001 (§7, §8, §10, §11, §46, §47) for precedent and governing authority. `runtime/src/lib.rs`, `core/actor-host/src/{lib,internal}.rs`, `core/capability-authority/src/{lib,internal}.rs`, `core/execution-coordinator/src/{lib,internal}.rs`, `services/scheduler/src/{lib,internal}.rs`, `common/src/lib.rs`, `core/actor-host/Cargo.toml`, `core/execution-coordinator/Cargo.toml`, `runtime/examples/{worker_pool,actor_to_actor_messaging}.rs`, and `runtime/tests/{bootstrap_grant,worker_pool,actor_to_actor_messaging}.rs` were re-confirmed directly against current source for this report. The Final Independent Implementation Review is treated as the authoritative verification source, per this task's own governing instruction; its conclusions are not reinterpreted here.

## 8. Independent Review Summary

This milestone has been independently reviewed six times across this engineering effort, in this order:

1. **Publication Recovery Review** — determined the truthful publication strategy for the stacked, uncommitted milestone history.
2. **Architecture Reconstruction Review** ("Capability-Authorized Actor-to-Actor Messaging Runtime") — first reconstruction pass.
3. **Runtime Actor Execution Architecture Review** — deeper reconstruction pass, the direct analytical basis for ARCH-006 and EWO-009.
4. **Independent Implementation Review of ARCH-006/EWO-009 (first pass)** — verdict CHANGES REQUIRED; found Scheduler's own trait redesign mischaracterized as unaffected by this milestone. Corrected (ARCH-006/EWO-009 v0.1.1).
5. **Independent Implementation Review of ARCH-006/EWO-009 (second pass, fully independent re-review)** — verdict CHANGES REQUIRED; found three further undisclosed items (`ActorHost::create_instance_with_behavior`, `ActorHost::dequeue`, `ExecutionCoordinator::fail`). A subsequent forensic Implementation Inventory and Governance Coverage Reconciliation additionally surfaced `Runtime::step()`/`run_until_idle()` as undisclosed. Corrected (ARCH-006/EWO-009 v0.1.2). A dedicated Historical Provenance Audit then found two further, narrower defects (a stale "pre-existing cleanup path" claim; an understated `dispatch` signature-evolution description) plus one already-known item (`Runtime::create_actor_instance_with_behavior` mischaracterized as pre-existing) still uncorrected from the prior pass. All three were corrected via the Historical Provenance Corrections task (ARCH-006/EWO-009 v0.1.3).
6. **Final Independent Implementation Review (Publication Candidate)** — re-verified architecture, engineering record, governance coverage, historical provenance, and implementation behaviour against v0.1.3 of both documents; re-ran full validation (§13); found no remaining discrepancy. **Verdict: APPROVED.**

No further implementation change was made or required at any point in this chain — every correction was to the *governance record*, never to `synapse-runtime` itself. This report treats reviews 5 and 6 as its own direct verification basis, consistent with EWO-009's own "Independent Implementation Review Expectations" section, which requires a dedicated ER to independently re-verify rather than merely restate.

## 9. Objectives

Implement the first realization of ARCH-006 — Runtime Actor Execution Architecture: genuine, non-simulated `Actor::handle()` invocation; treatment of an actor's own emitted messages as fresh, independently authorized admission requests; the single, shared, Runtime-owned admission pipeline every message origin converges through; deterministic, non-arbitrary capability resolution for those requests; Runtime-owned, non-forgeable causation; and a bounded, one-time bootstrap-grant mechanism giving a genuinely external caller its first capability — exactly as EWO-009 authorizes, and no further.

## 10. Scope

**Architectural scope.** Bounded entirely by ARCH-006 v0.1.3 — the sole architectural authority for this milestone. No architecture document was amended, reinterpreted, or extended by this work.

**Implementation scope.** `Execution Coordinator::dispatch` gaining a `message: &Message` parameter, an `actor: Option<&mut dyn Actor>` parameter, and a return type change from `Result<(), RuntimeError>` to `Result<Vec<Message>, RuntimeError>`; `ExecutionCoordinator::fail`; `ActorHost::behavior_mut`, `ActorHost::create_instance_with_behavior`, `ActorHost::dequeue` (plus the new `behaviors` stored field and `Runtime::create_actor_instance_with_behavior`, its public wrapper); `CapabilityAuthority::bound_capabilities`; `Runtime::admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`, `execute_message_capturing`; `Runtime::step()`, `run_until_idle()`, and their supporting internals (`schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, `execute_next_scheduled_message_with_outcome`, `bind_capability`); `BootstrapGrantName`, `BootstrapGrant`, `RuntimeBootstrapConfig`, `BootstrapGrantSet`, `Runtime::bootstrap_with_config`; `RuntimeError::AmbiguousAuthority`; Scheduler's own first genuine implementation (trait redesign to `mark_ready`/`remove`/`select_next()`, `SchedulerImpl`'s FIFO storage); supporting engineering types `ActorExecutionOutcome`, `RuntimeStepOutcome`, `RuntimeRunOutcome`, `EmittedMessageOutcome`, `DequeuedMessage`; and the `synapse-api` dependency addition to Actor Host and Execution Coordinator.

## 11. Explicit Exclusions

Confirmed, as of this report's own authoring, against each item EWO-009's own "Explicit Exclusions" names:

| Excluded (by this milestone) | Status as of this report |
|---|---|
| Supervision | Remained excluded by this milestone; **subsequently delivered** by ARCH-004/EWO-007 (ER-007) — resolved, not a gap |
| Timers | Remained excluded by this milestone; **subsequently delivered** by ARCH-005/EWO-008 (ER-008) — resolved, not a gap |
| Retries | Confirmed still excluded — no redelivery mechanism exists anywhere in the current workspace |
| Restart strategies | Confirmed still excluded at this milestone's own scope; restart itself is EWO-007's concern, unaffected here |
| Persistence | Confirmed still excluded — no durability contract exists |
| Durable mailboxes | Confirmed still excluded — mailbox contents remain in-memory, lost on termination |
| Workflow runtime | Confirmed still excluded — no cross-message orchestration concept exists |
| Effect runtime | Confirmed still excluded — no generalized effect-scheduling abstraction exists |
| Networking | Confirmed still excluded — entirely in-process |
| Distributed runtime | Confirmed still excluded — no location or transport concept exists |
| Clustering | Confirmed still excluded — not addressed |
| Service discovery | Confirmed still excluded beyond the existing, unmodified Actor Directory contract |
| Remote execution | Confirmed still excluded — dispatch remains entirely local |

## 12. Architecture Compliance

Verified directly against ARCH-006 v0.1.3, independently of any implementation narrative:

| ARCH-006 requirement | Compliance |
|---|---|
| Runtime remains sole orchestrator (§9.3, ADR-0016 Rule 1) | Confirmed — `execute_message_capturing`, `process_emitted_messages`, `resolve_emitted_message_authority`, `admit_message` are all private-or-Runtime-scoped; no other crate references any of the four |
| Genuine `Actor::handle()` invocation (§8) | Confirmed — `dispatch` genuinely invokes `actor.handle(&context, message)` when `actor` is `Some`; proven by both passing demonstrations |
| Emitted messages treated as fresh admission requests, never already-sent facts (§10) | Confirmed — `process_emitted_messages`'s own sequential, independent loop |
| Single shared admission pipeline (§11) | Confirmed — `admit_message` called identically by `submit_message` and `process_emitted_messages` |
| Runtime-owned, non-forgeable causation (§10, §13) | Confirmed — `message.causation` unconditionally overwritten with the Runtime-known `triggering_message` |
| Deterministic authority resolution — exactly one match, zero or multiple both rejected (§8, §13) | Confirmed — `EnforcementDenied` (zero match, existing variant) and `AmbiguousAuthority` (multiple match, the sole new `RuntimeError` variant) |
| Bootstrap grants minted once, before `Running`, Bootstrap Capability never exposed (§12) | Confirmed — `bootstrap_with_config` mints declared grants through the existing `issue_capability` path during the one-time bootstrap act |
| Trusted Core boundary unchanged at seven components (§9.1) | Confirmed — three components (Actor Host, Capability Authority, Execution Coordinator) narrowly extended; four (Lifecycle Guardian, Message Gateway, Audit Emitter, Host Adapter) received zero source change |
| Scheduler's own first genuine implementation, architecturally unaffected (§9.1, §9.2) | Confirmed — trait redesigned to `mark_ready`/`remove`/`select_next()`; role (ready-order policy only) and isolation (`synapse-common`-only dependency) unchanged |
| `step()`/`run_until_idle()` the sole caller-facing entry points into genuine dispatch (§10) | Confirmed for the new execution-driving surface specifically; the pre-existing `execute_message` path (unchanged public signature) also reaches dispatch via `execute_message_capturing`, exactly as ARCH-006 §10's own diagram already shows both paths |
| Ordering guarantees 1–7 (§10) | Confirmed by direct statement-order inspection of `execute_message_capturing` |
| No new Trusted Core component, lifecycle state, or constitutional guarantee (§5) | Confirmed |

No architectural deviation was found.

## 13. Implementation Summary

**Genuine dispatch.** `Execution Coordinator::dispatch`'s signature evolved from `dispatch(&mut self, context: ExecutionContext) -> Result<(), RuntimeError>` to `dispatch(&mut self, context: ExecutionContext, message: &Message, actor: Option<&mut dyn Actor>) -> Result<Vec<Message>, RuntimeError>` — two new parameters and a changed return type, confirmed by direct diff against the committed predecessor state. `actor` is genuinely invoked when present; a mechanical, behaviour-free instance still dispatches, invoking nothing. The changed return type is the mechanism by which an actor's own emitted messages are captured and routed onward. `ExecutionCoordinator::fail` was added as the truthful counterpart to `complete`, for a dispatch that genuinely failed — closing the exact cleanup gap EWO-005/ER-005 explicitly disclosed and left open.

**Runtime orchestration.** `Runtime` remains the sole cross-component composer for every operation this milestone introduces. `execute_message_capturing` — a new private method, extracted from what was previously inline logic in the public `execute_message` — obtains behaviour via `Actor Host::behavior_mut`, passes it into `dispatch`, and, once dispatch has genuinely, truthfully completed (success via `complete`, failure via `fail`), routes any emitted messages into `process_emitted_messages`.

**Shared admission pipeline.** `Runtime::admit_message` is the single, private helper both `submit_message` (external origin) and `process_emitted_messages` (actor-emitted origin) call identically — one function, two call sites, no divergence.

**Capability resolution.** `Runtime::resolve_emitted_message_authority` resolves send authority purely from the emitting actor's own currently bound, currently valid capability set (`CapabilityAuthority::bound_capabilities`, a new public accessor), requiring exactly one structural match: zero is rejected `EnforcementDenied` (existing variant, reused); more than one is rejected `AmbiguousAuthority` (the sole new `RuntimeError` variant this milestone introduces).

**Causation.** Unconditionally overwritten by Runtime with the truthful, Runtime-established triggering-message id — an emitting actor's own self-declared causation claim is never trusted.

**Scheduling.** `Scheduler`'s public trait was redesigned from a single, stateless `select_next(ready: &[ActorInstanceId])` method to three stateful operations (`mark_ready`, `remove`, `select_next()`), backed by a genuine FIFO implementation (`VecDeque`-backed order, `HashSet`-backed membership) — the crate's own first genuine implementation, previously an empty, zero-field placeholder with no trait implementation.

**Execution outcomes.** `ActorExecutionOutcome`, `RuntimeStepOutcome`, `RuntimeRunOutcome`, and `EmittedMessageOutcome` are new, non-architectural supporting engineering types recording, respectively: a single dispatch's genuine result; the result of one `step()` call; the result of a full `run_until_idle()` call; and a single emitted message's own independent admission result.

**Bootstrap grants.** `BootstrapGrantName`, `BootstrapGrant`, `RuntimeBootstrapConfig`, and `BootstrapGrantSet` are private-field, constructor-validated types; `Runtime::bootstrap_with_config` mints declared grants through the existing `issue_capability` path, bounded by `MAX_BOOTSTRAP_GRANTS = 16` (a fixed, documented, non-configurable constant, deliberately modeled on `core/actor-host`'s own `MAILBOX_CAPACITY` precedent). `bootstrap()` delegates to `bootstrap_with_config` with an empty configuration, preserving every existing caller's own behaviour unchanged.

**Trusted Core evolution.** Three of seven Trusted Core components gained a narrow, disclosed addition: Actor Host (`behavior_mut`, `create_instance_with_behavior`, `dequeue`; new `behaviors: HashMap<ActorInstanceId, Box<dyn Actor>>` stored field; new `synapse-api` dependency), Capability Authority (`bound_capabilities`, no new field or dependency), Execution Coordinator (`dispatch`'s widened signature, `fail`; no new stored field; new `synapse-api` dependency). The remaining four Trusted Core crates (Lifecycle Guardian, Message Gateway, Audit Emitter, Host Adapter) required, and received, zero source change. No new Trusted Core component was introduced; the count remains exactly seven.

**Caller-facing execution surface.** `Runtime::step()` (one bounded unit of work) and `Runtime::run_until_idle()` (repeated `step()` calls until idle or a caller-supplied bound) are the sole new entry points driving genuine dispatch, built from `schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, and `execute_next_scheduled_message_with_outcome`; `bind_capability` is a further new public method, a thin delegation to `CapabilityAuthority::bind`; `Runtime::create_actor_instance_with_behavior` is the public wrapper reaching `ActorHost::create_instance_with_behavior`.

**Testing additions.** Extensive unit-test growth within `core/actor-host` (+17), `core/capability-authority` (+5), `core/execution-coordinator` (+15), `services/scheduler` (+18), and `runtime/src/lib.rs` (+97, this milestone's own isolated contribution per EWO-009's own reconciliation); three new public-API-only integration test files (`bootstrap_grant.rs` +8, `worker_pool.rs` +29, `actor_to_actor_messaging.rs` +2); two new demonstrations (`worker_pool.rs`, `actor_to_actor_messaging.rs`).

## 14. Engineering Decisions

Recorded in full, with their evidentiary basis, in EWO-009's own "Bounded Design Decisions" (nine decisions) — not reproduced here. Summarized: admission-pipeline consolidation into one shared helper was structurally required by ARCH-002 §8 and ADR-0016 Rule 1, not a genuinely open choice; capability-resolution ambiguity was refused rather than arbitrated, evidenced by `AmbiguousAuthority`'s narrow, single-purpose introduction; the bootstrap-grant bound was fixed and modeled explicitly on `MAILBOX_CAPACITY`'s own precedent; Actor Host's new query/attachment surface (`behavior_mut`, `create_instance_with_behavior`, `dequeue`) and Capability Authority's new accessor (`bound_capabilities`) were each kept to the narrowest possible surface; Execution Coordinator's `fail` closes a gap EWO-005/ER-005 itself disclosed and left open; Runtime's caller-facing execution-driving surface (`step`/`run_until_idle`) was the structurally required consequence of genuine dispatch needing some way to actually be driven; Scheduler's trait redesign was the structurally required consequence of genuine execution needing something to actually schedule; and the `synapse-api` dependency addition introduces no new external (outside-workspace) dependency.

## 15. Testing Summary

**Baseline validation** (this milestone's own predecessor state, ER-006's own final figures, independently reconciled by EWO-009 and re-confirmed by this report's own review chain): 232 tests passing.

**This milestone's own delta** (independently reconciled against ER-007's own stated starting baseline, 423 tests):

| Crate/target | Before | After | Delta |
|---|---|---|---|
| `synapse-actor-host` | 31 | 48 | **+17** |
| `synapse-capability-authority` | 48 | 53 | **+5** |
| `synapse-execution-coordinator` | 15 | 30 | **+15** |
| `synapse-scheduler` | 1 | 19 | **+18** |
| `synapse-runtime` unit tests (`src/lib.rs`) | 54 | 151 | **+97** |
| `runtime/tests/actor_to_actor_messaging.rs` (new) | 0 | 2 | **+2** |
| `runtime/tests/bootstrap_grant.rs` (new) | 0 | 8 | **+8** |
| `runtime/tests/worker_pool.rs` (new) | 0 | 29 | **+29** |
| `runtime/tests/bootstrap.rs` | 16 | 16 | 0 |
| All other crates | unchanged | unchanged | 0 |
| **Workspace total** | **232** | **423** | **+191** |

Per-crate deltas sum exactly to the workspace-level delta (17+5+15+18+97+2+8+29 = 191). This is EWO-009's own reconciliation, cross-checked against ER-006's and ER-007's own separately-authored figures; independent re-derivation of the isolated `synapse-runtime` unit-test delta (54→151) from current source alone is not possible, since EWO-007 and EWO-008 both added their own tests to the same file afterward — an already-disclosed limitation (EWO-009 "Risks"), not a defect in this report.

**Current, full-workspace validation** (independently re-run for this report, reflecting this milestone plus EWO-007's and EWO-008's own later additions):

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | **517 passed, 0 failed, 0 ignored** |
| `cargo tree --workspace` | Clean; no cycle |
| `cargo run --example worker_pool` | Runs; deterministic round-robin dispatch, 9 messages executed, 3 workers × 3 completions each |
| `cargo run --example actor_to_actor_messaging` | Runs; capability-authorized actor-to-actor messaging completes successfully |
| `grep -r unsafe` (workspace, excluding `target/`) | Zero matches |

Per-crate figures directly re-confirmed against this milestone's own claimed post-implementation counts: `synapse-actor-host` 48 ✓, `synapse-capability-authority` 53 ✓, `synapse-execution-coordinator` 30 ✓, `synapse-scheduler` 19 ✓, `runtime/tests/bootstrap.rs` 16 ✓, `runtime/tests/actor_to_actor_messaging.rs` 2 ✓, `runtime/tests/bootstrap_grant.rs` 8 ✓, `runtime/tests/worker_pool.rs` 29 ✓ — exact match on every independently checkable figure.

## 16. Deviations from EWO-009

**None found.** No implementation deviation from EWO-009 v0.1.3 was identified at any point across the six-review chain (§8). Every correction made across that chain was to the governance documents' own historical and engineering-provenance claims — never to `synapse-runtime` itself, which was confirmed, repeatedly and independently, to already conform.

## 17. Governance Summary

- **Implementation Inventory** — completed. Exhaustively enumerated every implementation item this milestone introduced, without architectural judgement.
- **Governance Coverage Reconciliation** — completed. Classified every inventoried item (Category A/B/C/D) and approved the additions and one trim (Scheduler, in ARCH-006) subsequently applied.
- **Historical Provenance Audit** — completed. Swept every historical-claim statement in ARCH-006 and EWO-009 against direct source evidence; produced a full Provenance Matrix; found three remaining discrepancies.
- **Historical Provenance Corrections** — applied. All three discrepancies corrected (ARCH-006/EWO-009 v0.1.3); no residual contradiction found on re-sweep.
- **Final Independent Implementation Review (Publication Candidate)** — approved publication readiness. Re-verified architecture, engineering record, governance coverage, historical provenance, and implementation behaviour; re-ran full validation; verdict **APPROVED**.

This report does not reproduce the content of any of the above; each is cited as a completed, authoritative prior act.

## 18. Deferred Scope

Recorded exactly as EWO-009's own "Explicit Exclusions" and this report's own §11 already establish — no new future work is introduced here:

- Persistence and durable mailboxes.
- A workflow engine or generalized effect-scheduling system.
- Networking, distributed execution, or clustering.
- Message retry, redelivery, or acknowledgement protocols.
- Service discovery beyond the existing, unmodified Actor Directory contract.
- Remote execution.

(Supervision and timers, also originally excluded by this milestone, have since been delivered by ARCH-004/EWO-007 and ARCH-005/EWO-008 respectively — see §11.)

## 19. Risks

| # | Risk | Classification |
|---|---|---|
| 1 | This milestone's own "Implementation Sequence" ordering cannot be independently verified against a real-time record — no commit-level history exists | Major (historical-completeness limitation, already disclosed in EWO-009's own "Risks"; not an engineering-correctness risk) |
| 2 | The isolated `synapse-runtime` unit-test delta (54→151) is not independently re-derivable from current source alone, since later milestones share the same file | Minor (already disclosed; does not affect the workspace-level total, independently confirmed at 517) |
| 3 | Governing documents (ARCH-006, EWO-009) remain uncommitted in the documentation repository | Observation (process, not engineering) |

No Critical findings. No scope deviations. No engineering-correctness risk was identified at any point across the six-review chain.

## 20. Final Engineering Assessment

**Implementation complete:** confirmed — every capability EWO-009 records is present, genuinely functioning, and demonstrated by two passing, deterministic public-API-only examples. **Independently verified:** confirmed — six independent reviews, the last two (Historical Provenance Audit, Final Independent Implementation Review) conducted after every prior correction, finding no remaining implementation defect. **Governance reconciled:** confirmed — Implementation Inventory, Governance Coverage Reconciliation, Historical Provenance Audit, and Historical Provenance Corrections form a complete, closed chain, each building on and re-verifying the last rather than assuming it. **Historically reconstructed:** confirmed and explicitly acknowledged (§5) — this report, ARCH-006, and EWO-009 all postdate the implementation they record, and none authorizes, implies, or performs any new implementation work. **Publication approved:** confirmed — the Final Independent Implementation Review's own verdict, APPROVED, is adopted here without reinterpretation.

**Overall:** a faithful, exhaustively cross-checked, historically-reconstructed record of an already-complete, already-correct implementation milestone — the missing governance chain is now closed, not because the implementation required correction, but because its documentation, across three iterative rounds, was brought into full, self-consistent, source-verified agreement with what the implementation has been, truthfully, all along.

## 21. Final Verdict

**APPROVED FOR PUBLICATION**

## References

- ARCH-001 — Constitutional Architecture
- ARCH-002 — Runtime Architecture (v0.2.0)
- ARCH-003 — Runtime Integration Architecture (v0.4.0)
- ARCH-006 — Runtime Actor Execution Architecture (v0.1.3) — the sole architectural authority this report's milestone implements
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- STD-001 — Documentation Standards (§7, §8, §10, §11, §46, §47)
- EWO-009 — Runtime Integration: Genuine Actor Execution, Capability-Authorized Actor-to-Actor Messaging, and Bootstrap Grants (work-orders/EWO-009-Runtime-Actor-Execution.md)
- EWO-006 — Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection (milestone predecessor)
- ER-006 — Bounded Actor Mailboxes — Engineering Report
- EWO-007 — Runtime Integration: Local Actor Supervision, Failure Escalation, and Restart Ownership (milestone successor)
- ER-007 — Local Actor Supervision — Engineering Report
- EWO-008 — Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution (milestone successor)
- ER-008 — Temporal Runtime — Engineering Report (highest-numbered prior Engineering Report; report-sequence predecessor only, per §2)
- "SynapseOS — Publication Recovery Architecture Review" (this engineering effort)
- "SynapseOS Architecture Review — Capability-Authorized Actor-to-Actor Messaging Runtime" (this engineering effort)
- "SynapseOS Architecture Review — Runtime Actor Execution Architecture" (this engineering effort)
- `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree): `runtime/src/lib.rs`, `core/actor-host/src/{lib,internal}.rs`, `core/actor-host/Cargo.toml`, `core/capability-authority/src/{lib,internal}.rs`, `core/execution-coordinator/src/{lib,internal}.rs`, `core/execution-coordinator/Cargo.toml`, `services/scheduler/src/{lib,internal}.rs`, `common/src/lib.rs`, `runtime/examples/{worker_pool,actor_to_actor_messaging}.rs`, `runtime/tests/{bootstrap_grant,worker_pool,actor_to_actor_messaging}.rs`
- Final Independent Implementation Review (Publication Candidate; this session; the authoritative verification source for this report, per its own governing instruction)

## Files Changed

Attributed to this milestone specifically, per EWO-009's own "Scope"/"Trusted Core"/"Required Interface Evolution" sections, independently re-verified three times across this engineering effort's review chain:

| File | Change |
|---|---|
| `runtime/src/lib.rs` | `admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`, `execute_message_capturing` (new private methods); `step`, `run_until_idle`, `schedule_next`, `schedule_next_message`, `execute_next_scheduled_message`, `execute_next_scheduled_message_with_outcome`, `bind_capability`, `create_actor_instance_with_behavior`, `bootstrap_with_config` (new public methods); `BootstrapGrantName`, `BootstrapGrant`, `RuntimeBootstrapConfig`, `BootstrapGrantSet`, `ActorExecutionOutcome`, `RuntimeStepOutcome`, `RuntimeRunOutcome`, `EmittedMessageOutcome` (new types); `MAX_BOOTSTRAP_GRANTS` constant; extensive new unit tests |
| `core/actor-host/src/lib.rs`, `core/actor-host/src/internal.rs` | `behavior_mut`, `create_instance_with_behavior`, `dequeue` added to the `ActorHost` trait and implementation; `DequeuedMessage` type; new unit tests |
| `core/actor-host/Cargo.toml` | New dependency: `synapse-api` |
| `core/capability-authority/src/lib.rs`, `core/capability-authority/src/internal.rs` | `bound_capabilities` added to the `CapabilityAuthority` trait and implementation; new unit tests |
| `core/execution-coordinator/src/lib.rs`, `core/execution-coordinator/src/internal.rs` | `dispatch`'s signature widened (`message`, `actor` parameters; `Result<Vec<Message>, RuntimeError>` return); `fail` added to the trait and implementation; new unit tests |
| `core/execution-coordinator/Cargo.toml` | New dependency: `synapse-api` |
| `services/scheduler/src/lib.rs`, `services/scheduler/src/internal.rs` | `Scheduler` trait redesigned (`mark_ready`, `remove`, `select_next()`); `SchedulerImpl` gained genuine FIFO implementation; extensive new unit tests |
| `common/src/lib.rs` | `RuntimeError::AmbiguousAuthority` — the sole new variant |
| `runtime/examples/worker_pool.rs`, `runtime/examples/actor_to_actor_messaging.rs` (new) | Public-API-only demonstrations |
| `runtime/tests/bootstrap_grant.rs`, `runtime/tests/worker_pool.rs`, `runtime/tests/actor_to_actor_messaging.rs` (new) | Public-API-only integration test suites |
| `Cargo.lock` | Reflects the `synapse-api` dependency additions above; no new external dependency |

No Lifecycle Guardian, Message Gateway, Audit Emitter, or Host Adapter (the four untouched Trusted Core crates) file was modified for this milestone.

## Final Git Status

`synapse-runtime` (`git status --short`, this milestone's own untracked files):

```
?? runtime/examples/
?? runtime/tests/bootstrap_grant.rs
?? runtime/tests/worker_pool.rs
?? runtime/tests/actor_to_actor_messaging.rs
```

(`runtime/src/lib.rs`, the Trusted Core/Scheduler crate files, `common/src/lib.rs`, `Cargo.toml`, and `Cargo.lock` show as modified relative to HEAD, but HEAD predates this and three other already-reviewed milestones layered together — unaffected by this task.) Nothing staged. HEAD remains `5ccc7f9083a71adc6ee704b2322a701935765679`; `origin/main` unchanged. Fingerprint (`git status --short | md5sum`): `04e39ef24f5b8fbcbb61eba8cbccc5f2` — identical to every checkpoint recorded across this engineering effort.

`synapse-docs` (`git status --short`, this task's own files):

```
?? engineering-reports/ER-009-Runtime-Actor-Execution.md
```

Nothing staged. HEAD remains `e90404baa5140ce9004839bc51921c789777e003`. ARCH-006 and EWO-009 were not modified by this report. This report is the only file this task adds to `synapse-docs`.

## Confirmations

- **Runtime repository:** untouched by this report-authoring task — no source, test, or manifest file was modified while authoring ER-009. All runtime changes described above were already complete, tested, and independently reviewed (six times) before this report began.
- **Documentation repository:** changed only by the addition of this new Engineering Report. No other file in `synapse-docs` was created, modified, staged, or removed.
- **No commits, stages, or pushes occurred** in either repository during this task.

## Final Disposition

This report records completed, independently reviewed, historically reconstructed engineering work. It does not itself authorize, approve, publish, commit, or push anything, consistent with STD-001 §47's informational-only status for Engineering Reports. No further review or implementation was performed as part of authoring it.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-15 | Claude (AI-assisted) | Initial report following EWO-009 v0.1.3's completion of the full review-and-correction chain (six independent reviews; three correction rounds spanning Scheduler disclosure, `dequeue`/`create_instance_with_behavior`/`fail`/`step`/`run_until_idle` disclosure, and three historical-provenance corrections). Records the Runtime Actor Execution implementation, architecture compliance, testing (517 total workspace tests; this milestone's own +191 delta independently reconciled per-crate against ER-006's and ER-007's own figures), zero deviations from EWO-009, and the Final Independent Implementation Review's verdict of APPROVED. Explicitly acknowledges its own historical-reconstruction status, per EWO-009's own "Required Completion Report Contents" item 7. |
