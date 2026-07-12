---
document_id: ER-002
title: Actor Host — Engineering Report
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-12
last_updated: 2026-07-12
classification: Public
related_documents:
  reports_on: EWO-002 (work-orders/EWO-002-Actor-Host.md)
  architecture:
    - ARCH-001
    - ARCH-002
  adrs:
    - ADR-0015
    - ADR-0016
  standards:
    - STD-001
    - STD-002
    - STD-004
    - STD-011
---

# ER-002 — Actor Host — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize anything.

## 1. Executive Summary

SRP-002 (Actor Host) is implemented per EWO-002 v0.3.0, itself integrating the now-approved ADR-0015 (Audit Emitter Failure Semantics) and ADR-0016 (Trusted Core Interaction Model). Actor Host's `define`, `create_instance`, and `terminate_instance` are real, tested behaviour. The mandatory actor-creation and actor-termination audit events reach Audit Emitter through a minimal, Runtime-mediated interaction — Actor Host has no dependency on Audit Emitter, direct or otherwise. Audit-emission failure fails the triggering operation, with no Runtime-wide effect and no rollback. All validation gates pass: `cargo fmt`, `cargo clippy -D warnings`, `cargo build`, `cargo test` (all 27 workspace tests plus EWO-001's original 5, unmodified in outcome). Zero external dependencies, zero `unsafe`, zero new `#[allow(...)]` suppressions.

## 2. Scope Implemented

- Real `ActorHostImpl` behaviour for `define`, `create_instance`, `terminate_instance` (ARCH-002 §7, §11 steps 2–3, §15, §17, §20).
- Define-before-create, duplicate-instance uniqueness, unknown/duplicate-termination rejection — all via `RuntimeError::UnknownTarget`, an existing variant, no new error type introduced.
- Private, per-instance existence/liveness bookkeeping, exposed through no public accessor.
- Runtime-mediated audit coordination: `Runtime::create_actor_instance` and `Runtime::terminate_actor_instance` invoke Actor Host, then construct and cause the required `AuditEvent` to reach Audit Emitter via the Runtime's own already-existing dependency on it.
- Audit failure semantics: a failed mandatory audit emission fails the triggering operation; Runtime state and other operations are unaffected.
- The one minimal construction-surface change EWO-002 anticipated and authorized: `ActorHostHandle` now implements `ActorHost` directly (delegating to the private `ActorHostImpl`), making its real behaviour invocable by the Runtime.

## 3. Files Changed

| File | Change |
|---|---|
| `core/actor-host/src/internal.rs` | Real `ActorHostImpl` state (sequence counters, `defined: HashSet<ActorId>`, `instances: HashMap<ActorInstanceId, ActorId>`) and `impl ActorHost for ActorHostImpl` |
| `core/actor-host/src/lib.rs` | `impl ActorHost for ActorHostHandle` (delegating); 7 new behavioural unit tests added to the existing 2 |
| `runtime/src/lib.rs` | `Runtime::define_actor`, `create_actor_instance`, `terminate_actor_instance`; `actor_created_event`/`actor_terminated_event` helpers; 8 new unit tests (including a test-only selective-failure `AuditEmitter` double, constructed via existing private-field struct literals — no DI mechanism introduced) |
| `runtime/tests/bootstrap.rs` | 2 new integration tests exercising Actor Host through the Runtime's public API only |

No file outside `core/actor-host/` and `runtime/` was touched. `common/src/lib.rs` is unmodified — no new type, field, or function was added to it.

## 4. Public API Changes

- `synapse_actor_host::ActorHostHandle` now implements the `ActorHost` trait (previously implemented none). No new public type, factory, builder, or generic abstraction was introduced — the existing trait and the existing handle type are the entire surface.
- `synapse_runtime::Runtime` gained three public methods: `define_actor(&mut self, &str) -> Result<ActorId, RuntimeError>`, `create_actor_instance(&mut self, &ActorId) -> Result<ActorInstanceId, RuntimeError>`, `terminate_actor_instance(&mut self, &ActorInstanceId) -> Result<(), RuntimeError>`. `Runtime::bootstrap`, `Runtime::shutdown`, and `Runtime::state` are unchanged in signature and behaviour.

## 5. Architecture Compliance

- ARCH-002 §7 (Actor Runtime Representation): logical vs. instance identity distinction preserved; capability bindings not touched (remain Capability Authority's, never referenced here).
- ARCH-002 §11 steps 2–3: define-before-create enforced and tested; step 3's "Creation failure → remains Defined" honoured (`create_instance` never partially constructs an instance).
- ARCH-002 §17: isolation delivered by ordinary language-level enforcement (private struct fields, no public mutation path) — the only one of the three sanctioned mechanisms this single-process workspace can realize.
- ARCH-002 §6 / Trusted Core boundary: still exactly seven components; no responsibility moved between them; `synapse-actor-host`'s dependency graph is unchanged — `synapse-common` only (`cargo tree -p synapse-actor-host` confirmed).
- ARCH-002 §18: exactly the two mandatory events implemented (`actor.created`, `actor.terminated`) — no additional event category invented; `actor` definition itself carries no audit obligation and none was added.
- Per ADR-0015, mandatory audit emission failure does not imply state rollback. Actor Host state mutations remain committed even when the triggering operation returns an error because audit emission failed.

## 6. ADR-0015 Compliance (Audit Emitter Failure Semantics)

`create_actor_instance` and `terminate_actor_instance` each call Actor Host first, then Audit Emitter, propagating either failure via `?`. A failed required audit emission therefore fails the operation as a whole — verified directly by `create_actor_instance_fails_if_required_audit_emission_fails` and `terminate_actor_instance_fails_if_required_audit_emission_fails`. `audit_emission_failure_does_not_affect_runtime_state` confirms `RuntimeState` is untouched by an audit failure. `audit_emission_failure_on_one_operation_does_not_affect_another` confirms one operation's audit failure does not affect an independent one. No rollback, transaction, persistence, or mutation-ordering logic was introduced anywhere — per EWO-002, Actor Host's own internal record of a "phantom" instance created just before a failing audit call is left exactly as it lands; this is explicitly sanctioned implementation freedom, not a defect.

## 7. ADR-0016 Compliance (Trusted Core Interaction Model)

`synapse-actor-host`'s `Cargo.toml` is unchanged — it depends on `synapse-common` only; no dependency on `synapse-audit-emitter` or any other Trusted Core crate was added or is reachable (confirmed via `cargo tree -p synapse-actor-host`). Actor Host's own code never references `AuditEvent` or `AuditEmitter`. All coordination — invoking Actor Host, then constructing and emitting the audit event from Actor Host's own return values — is implemented exclusively in `runtime/src/lib.rs`, the Runtime composition root, which already depended on both crates before this milestone (established under EWO-001). No behavioural responsibility was absorbed, duplicated, or redefined: `Runtime` decides nothing about whether an actor definition/creation/termination is valid — that remains entirely `ActorHostImpl`'s own logic.

## 8. Testing Summary

| Suite | Tests | Result |
|---|---|---|
| `synapse-actor-host` unit tests | 9 (2 pre-existing + 7 new) | All pass |
| `synapse-runtime` unit tests | 13 (5 pre-existing + 8 new) | All pass |
| `synapse-runtime` integration tests (`tests/bootstrap.rs`) | 5 (3 pre-existing + 2 new) | All pass |
| Every other workspace crate | unchanged | All pass |
| **Total** | **32 tests across the workspace** (5 crates report 0 doc-tests each, as before) | **0 failures** |

Coverage against EWO-002's Required Tests: define-before-create ✓; duplicate-name definition still yields distinct identities ✓; duplicate creation on one actor yields distinct instance identities ✓; terminate existing / terminate unknown / double-terminate ✓; unique logical and instance identities ✓; private state exposed through no public accessor ✓ (verified by code review — no accessor exists); audit reaches Audit Emitter — demonstrated indirectly via successful operation outcome in the external integration test, since the production `AuditEmitterImpl` cannot itself fail and no public read-back API exists (Audit Pipeline, the consumption side, is out of scope) ✓; audit failure fails operation — demonstrated directly via an internal, same-crate unit test using a deliberately-failing `AuditEmitter` test double ✓; Pipeline failure remains independent — no Pipeline code exists or was touched, so this is vacuously true and was not separately tested ✓; deterministic behaviour — every test is fully deterministic (in-process monotonic counters, no timing or randomness anywhere) ✓.

## 9. Explicitly Excluded Future Work

Message Gateway, Capability Authority, Execution Coordinator, Lifecycle Guardian behaviour (including the full `ActorState` transition set), Host Adapter expansion, scheduling, execution coordination, message routing/delivery, persistence, networking, distributed execution, replaceable services, `synapse-api`'s `Actor` trait, security enforcement beyond this milestone's own invariants, plugins, SDK, any public API layer beyond what this milestone required, and any later SRP milestone. None of these has any corresponding code anywhere in this change.

## 10. Known Deviations

None. One pre-existing detail worth recording, not a deviation: `TrustedCore`'s `#[allow(dead_code)]` (added under EWO-001) remains necessary and untouched — `host_adapter`, `capability_authority`, `message_gateway`, `execution_coordinator`, and `lifecycle_guardian` are still genuinely unread by any code, exactly as EWO-001 documented; only `actor_host` and `audit_emitter` are now used. Narrowing or removing this attribute would be a change to fields entirely outside SRP-002's scope and was not made.

## 11. Final Engineering Assessment

SRP-002 is complete against EWO-002's Definition of Done. All mandatory validation gates pass with zero warnings, zero `unsafe`, zero external dependencies, zero new dependency of any kind (`synapse-actor-host` unchanged at `synapse-common` only), and no scope beyond what EWO-002 authorized. The two architectural ambiguities that previously blocked this milestone (audit coupling mechanism; audit failure semantics) are fully resolved in code, matching ADR-0016 and ADR-0015 exactly. No architectural ambiguity was encountered during implementation that EWO-002 did not already resolve. The repository is left unstaged, uncommitted, and ready for independent engineering review.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-12 | Claude (AI-assisted) | Initial report following SRP-002 implementation. |
