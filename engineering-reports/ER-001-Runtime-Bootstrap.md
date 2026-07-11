---
document_id: ER-001
title: Runtime Bootstrap — Engineering Report
version: 0.2.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-11
last_updated: 2026-07-11
classification: Public
related_documents:
  reports_on: EWO-001 (work-orders/EWO-001-Runtime-Bootstrap.md)
  architecture:
    - ARCH-001
    - ARCH-002
  standards:
    - STD-001
    - STD-004
---

# ER-001 — Runtime Bootstrap — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize anything.

**Task state: Resumed, completed, and committed locally.** EWO-001 halted before implementation on 2026-07-11 (Part A below), was resolved the same day by the Founder's Authorized Implementation Clarification, was then implemented and validated within its original scope (Part B below), and was subsequently committed as an immutable local Git commit in `synapse-runtime` (Part C below). The original halt is retained in full as historical evidence, per instruction not to erase the fact that engineering stopped for review. **This report, including Part C, does not itself constitute or create STD-001 §31 approval evidence.** That remains a separate, later, not-yet-performed act — see Part C for the exact boundary.

---

## Part A — Original Halt (2026-07-11, Historical Record)

*Unmodified from the version of this report filed at the time engineering stopped.*

### A.1 Objective (per EWO-001, as issued)

Implement the SynapseOS Runtime Bootstrap: lifecycle state machine, bootstrap sequence, Trusted Core initialization/shutdown, bootstrap orchestration, startup/shutdown audit emission, and bootstrap unit tests — nothing else.

### A.2 Implementation Summary

None. No file in `synapse-runtime` was created, modified, or deleted. Implementation was not attempted past the analysis stage, because that analysis surfaced a Definition-of-Failure condition EWO-001 itself required reporting rather than resolving.

### A.3 Validation Performed

The current state of `synapse-runtime` was read in full: the workspace manifest, all 13 crate manifests, every `lib.rs` and `internal.rs`, every crate `README.md`, the repository root `README.md`, `core/README.md`, and `services/README.md`. `git status` confirmed a clean tree with no remote, before and after. ARCH-002 §5, §6, §11, §15, §20, and §21 were re-read directly from source. A targeted search confirmed no `[[bin]]` target and no `fn main` existed anywhere in the workspace.

### A.4 Finding — No Composition Root Existed, and Its Absence Blocked EWO-001 as Originally Written

EWO-001 required constructing the seven trusted-core components and owning Runtime-level lifecycle state. No code, module, crate, or binary target existed to do this, and nothing in ARCH-002 or the repository designated where it should exist:

- Every trusted-core crate's `internal.rs` type was `pub(crate)` — invisible outside its own crate, by deliberate SRP-0 design. No crate could construct another crate's concrete implementation.
- ARCH-002 §11 step 1 assigned "Runtime initialization" to "Host Adapter, all trusted components" jointly, without assigning Host Adapter a construction role over its six siblings; Host Adapter's own trait and README gave no support for that reading.
- ARCH-002 §20's "Actor creation" row named the caller as "Actor Host client (bootstrap or supervisor)," confirming ARCH-002 anticipated an external bootstrap caller as a role, without assigning that role to any crate.
- `synapse-api` was explicitly scoped as the actor-authoring surface, not a Runtime component — ruled out by its own README.
- `synapse-common` was documented and previously reconciled (STD-004) as data-only — ruled out for orchestration logic.

This matched two conditions EWO-001's Definition of Failure listed verbatim: "a new Runtime abstraction appears necessary" and "an architectural decision is required." Engineering stopped rather than resolve this unilaterally.

### A.5 Recommendations (as originally filed)

Three non-prescriptive options were presented for the Founder's consideration: (1) a short ARCH-002 amendment naming a composition-root pattern; (2) a narrow, explicit exception to "no new crates" for exactly one orchestrator crate; (3) a visibility-only amendment exposing narrow public constructors plus a designated call site. See §7 (Resolution) below for which was chosen.

### A.6 Files Touched at Time of Halt

None. `work-orders/EWO-001-Runtime-Bootstrap.md` and this report were the only files created, both in `synapse-docs`. `synapse-runtime` was confirmed untouched.

---

## Part B — Resolution and Completion (2026-07-11)

### B.1 Resolution

The Founder issued an Authorized Implementation Clarification (recorded in EWO-001 §"Authorized Implementation Clarification"), selecting a variant of recommendation options (2) and (3) from A.5: exactly one new workspace member, an executable composition-root crate (package `synapse-runtime`, directory `runtime/`), authorized to construct the seven trusted-core components and own Runtime-level lifecycle state, explicitly not an eighth trusted-core component, replaceable service, or new constitutional concept. No ARCH amendment was required or made. EWO-001's original objective, scope, and constraints were left unchanged.

### B.2 Implementation Summary

**New crate:** `runtime/` (package `synapse-runtime`), with a library (`src/lib.rs`) and a thin binary (`src/main.rs`).

- `TrustedCore`: a private-field struct holding all seven trusted-core components' opaque construction handles. `TrustedCore::construct()` instantiates them in a documented order (Host Adapter, then Audit Emitter, then the remaining five in ARCH-002 §6's table order).
- `Runtime`: owns `RuntimeState` (added to `synapse-common`) and the constructed `TrustedCore`. `Runtime::bootstrap()` runs EWO-001's steps 1–4 (construct, verify, emit Runtime Started, enter Running) in that literal order; `Runtime::shutdown(self)` runs steps 5–7 (accept shutdown, emit Runtime Shutdown, terminate), consuming `self` by value so a terminated Runtime cannot be reused.
- `validate_transition`: enforces ARCH-002 §15's exact legal transition set (`Initializing -> Running -> Stopping -> {Stopped | Failed}`); any other transition returns `RuntimeError::IllegalTransition`.

**Trusted-core crates (six):** `capability-authority`, `actor-host`, `message-gateway`, `execution-coordinator`, `lifecycle-guardian`, `host-adapter` each gained a `pub struct <Name>Handle`, wrapping the existing `pub(crate)` impl type, with `pub fn new()` and `Default`. The handle implements no trait — the underlying behavior (capability validation, actor hosting, message admission, execution-coordination internals, lifecycle-transition-enforcement internals, host operations) remains explicitly out of scope and unimplemented.

**Audit Emitter (one, treated differently):** because "Startup audit emission" and "Shutdown audit emission" are explicitly in EWO-001's Scope, this crate received a real, minimal `impl AuditEmitter for AuditEmitterImpl` — an in-process `Vec<AuditEvent>` buffer that `emit` pushes to. This is emission only, exactly per the trait's own documented contract; no storage, indexing, retention, or redaction was added (that remains the separate, out-of-scope Audit Pipeline service). A public `synapse_audit_emitter::new() -> Box<dyn AuditEmitter>` factory was added.

**`synapse-common`:** one addition — the `RuntimeState` enum (`Initializing`, `Running`, `Stopping`, `Stopped`, `Failed`), taken directly from ARCH-002 §15's text, not invented. No function or method was added to `synapse-common`; it remains data-only.

### B.3 Runtime Behaviour Implemented

Exactly EWO-001's seven bootstrap/shutdown steps, nothing more: construct the seven trusted-core components; verify initialization (structurally checked, currently always succeeds since construction cannot yet fail); emit Runtime Started; enter Running; accept shutdown; emit Runtime Shutdown; cleanly terminate. No actor execution, capability issuance/validation/revocation, message routing, mailbox, scheduling, persistence, actor-directory, networking, distributed-Runtime, external-provider, plugin, SDK, or configuration-loading behavior was implemented, per EWO-001's Out of Scope list.

### B.4 Lifecycle States Implemented

The five ARCH-002 §15 Runtime-level states: `Initializing`, `Running`, `Stopping`, `Stopped`, `Failed`. No state was invented beyond these five. Legal transitions: `Initializing→Running`, `Initializing→Failed`, `Running→Stopping`, `Stopping→Stopped`, `Stopping→Failed`. All other transitions are rejected with `RuntimeError::IllegalTransition`.

### B.5 Tests Added

- Six trusted-core crates: one `handle_is_constructible` unit test each (in addition to each crate's pre-existing `trait_is_object_safe` test).
- `audit-emitter`: `new_returns_a_working_emitter` (lib.rs) and `emit_records_the_event` (internal.rs).
- `runtime` crate unit tests (`src/lib.rs`): `bootstrap_reaches_running_state`, `shutdown_from_running_succeeds`, `legal_transitions_are_accepted`, `illegal_transitions_are_rejected` (exhaustive over all 20 non-legal state pairs), `terminal_states_have_no_outgoing_legal_transition`.
- `runtime` crate integration tests (`tests/bootstrap.rs`, public-API only): `runtime_boots_and_reaches_running`, `runtime_shuts_down_cleanly_after_bootstrap`, `full_bootstrap_and_shutdown_cycle_succeeds`.

### B.6 Validation Results

All commands run from the `synapse-runtime` workspace root, all passing:

| Command | Result |
|---|---|
| `cargo fmt --all -- --check` | Pass — no diff |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Pass — zero warnings |
| `cargo build --workspace` | Pass |
| `cargo test --workspace` | Pass — all unit, integration, and doc-tests green |
| `cargo tree --workspace` | Pass — confirms dependency shape below |

Additional verification performed:

- **Exactly one new crate:** `cargo metadata` lists 14 workspace members (13 before this EWO, plus `synapse-runtime`). Confirmed by direct enumeration.
- **No external dependencies introduced:** `Cargo.lock` contains no `[[package]]` entry with a `source` field — every package resolves to an in-workspace path. Confirmed by direct grep, before and after.
- **No `unsafe` code:** `grep -rn "unsafe"` across all `.rs` files (excluding `target/`) returns nothing.
- **No trusted-core responsibility moved into the composition root:** the composition root contains only construction calls, the Runtime-level state machine, and audit-emission calls into Audit Emitter's own trait method — no capability, actor, message, execution, or lifecycle-transition logic. Verified by direct review of `runtime/src/lib.rs`.
- **No architecture or standards document modified:** confirmed via `git status` in `synapse-docs` — `architecture/`, `standards/`, and `governance/` show no changes from this task beyond the EWO-001/ER-001 documents themselves.
- **`synapse-common` remains data-only:** confirmed by scanning for any function defined outside a `#[cfg(test)]` module in `common/src/lib.rs` — none exists; only the new `RuntimeState` enum was added.

### B.7 Dependency Changes

One, entirely internal to the workspace: the new `runtime` crate depends on `synapse-common` and all seven trusted-core crates (`synapse-capability-authority`, `synapse-actor-host`, `synapse-message-gateway`, `synapse-execution-coordinator`, `synapse-lifecycle-guardian`, `synapse-audit-emitter`, `synapse-host-adapter`), added to `[workspace.dependencies]` in the root `Cargo.toml` for consistency with the existing `synapse-common` entry. No replaceable-service crate is depended on — none is required by ARCH-002 §21 for bootstrap or shutdown. No external (non-workspace) dependency was introduced anywhere. No existing crate's dependency list was changed. No dependency cycle exists — confirmed via `cargo tree --workspace`; no trusted-core or replaceable-service crate depends on the composition root.

### B.8 Trusted Core Changes

No trusted-core crate's boundary, responsibility, or trait was changed. Each of the seven retains exactly its original `pub trait`, unimplemented. Six gained one new `pub` construction-handle type each (no behavior, no field exposure). One (Audit Emitter) gained a real implementation of its own, already-defined, single-method trait — not a new responsibility, but the one already assigned to it (ARCH-002 §6: "Emits a minimal, unbypassable record") finally realized, within the bounds EWO-001 explicitly placed in scope.

### B.9 Architecture Changes

None. ARCH-001 and ARCH-002 were not modified. The Authorized Implementation Clarification confirmed no ARCH amendment was required; the composition-root crate is explicitly not a new architectural concept, per its own README and per EWO-001's clarification.

### B.10 Engineering Decision Log

- **Implementation decisions:**
  - Trusted-core construction order: Host Adapter, then Audit Emitter, then Capability Authority, Actor Host, Message Gateway, Execution Coordinator, Lifecycle Guardian (ARCH-002 §6 table order for the last five). Recorded as an implementation choice; ARCH-002 does not itself enumerate a sub-order among the seven for step 1.
  - Six trusted-core crates: opaque public handle (no trait implementation), since their trait behavior is out of scope. Audit Emitter: real trait implementation via `Box<dyn AuditEmitter>`, since emission is explicitly in scope — the two different forms EWO-001's "Preferred form" language offered were each applied to the crates they honestly fit, rather than picking one form for all seven.
  - Audit event type strings (`"runtime.started"`, `"runtime.shutdown"`) are an implementation-level naming choice; ARCH-002 §18 mandates the event categories, not exact string values.
  - `Runtime::shutdown` takes `self` by value, using Rust's ownership system to make "cannot shut down twice" a compile-time property rather than only a runtime check.
  - Composition-root crate directory placed at top level (`runtime/`), matching the existing `common/`/`api/` convention for single, non-categorized crates.
- **Repository decisions:** Added `runtime` to the workspace `members` list and seven new entries to `[workspace.dependencies]` in the root `Cargo.toml`, following the existing pattern already used for `synapse-common`. Updated the repository root `README.md`'s structure listing and Status section for factual accuracy (a new crate and directory now exist).
- **Deferred decisions:** Making `TrustedCore::construct()` fallible (it is currently infallible because nothing inside it can yet fail); a real Audit Pipeline consumer for emitted events; any behavior for the six out-of-scope trusted-core traits.
- **Architectural decisions:** None made in this implementation pass — the one identified in Part A was resolved by the Founder directly, not by engineering.
- **Constitutional decisions:** None — none were implicated.
- **Future work enabled:** Actor execution, capability issuance/validation/revocation, message routing, and the other out-of-scope items from EWO-001 can now be built on top of a working, tested bootstrap/shutdown cycle and a real audit-emission path.
- **Future work deferred:** Everything EWO-001 places out of scope remains out of scope, unchanged by this completion.

### B.11 Issues Requiring Architectural Review

None outstanding. The one issue identified in Part A was resolved by the Founder's Authorized Implementation Clarification before implementation proceeded.

### B.12 Files Modified

`synapse-runtime` repository: `Cargo.toml`, `Cargo.lock`, `README.md`, `common/src/lib.rs` (4 files), and `lib.rs`/`internal.rs` in each of `core/actor-host`, `core/audit-emitter`, `core/capability-authority`, `core/execution-coordinator`, `core/host-adapter`, `core/lifecycle-guardian`, `core/message-gateway` (7 crates × 2 files = 14 files) — 18 files total, matching `git diff --stat`.

`synapse-docs` repository: `work-orders/EWO-001-Runtime-Bootstrap.md` (added the Authorized Implementation Clarification).

### B.13 Files Created

`synapse-runtime` repository: `runtime/Cargo.toml`, `runtime/README.md`, `runtime/src/lib.rs`, `runtime/src/main.rs`, `runtime/tests/bootstrap.rs`.

`synapse-docs` repository: this report (already existed from Part A; substantively rewritten to record completion).

---

## Part C — Commit Record (2026-07-11)

The implementation Part B describes, which existed only as uncommitted working-tree changes when Part B was written, was subsequently established as one immutable local Git commit in `synapse-runtime`, as a distinct, later repository-reconciliation act.

### C.1 Runtime Implementation Artifact Identity

```text
Repository: /home/sudonimm/Development/SynapseOS/synapse-runtime
Branch: main
Commit: 00aa309bac3eaccd3e53d311f27b8ea0587dab1c
Parent: 25c0ea0e0941666db06b47fc92d76424c31583e4
Subject: feat(runtime): implement Runtime bootstrap
```

Committed file count: 23 (18 modified, 5 newly created under `runtime/`). Diff stat: `23 files changed, 662 insertions(+), 14 deletions(-)`. Every committed path was independently verified, before staging, against the exact path list EWO-001's scope implies — zero unexplained paths.

### C.2 Post-Commit Validation (re-run against the committed state, not merely the prior working tree)

| Check | Result |
|---|---|
| `cargo fmt --all -- --check` | Pass |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Pass — zero warnings |
| `cargo build --workspace` | Pass |
| `cargo test --workspace` | Pass — 30 tests passed, 0 failed, 0 ignored |
| `cargo tree --workspace` | Pass — dependency shape unchanged from Part B |
| Workspace member count | 14 (`cargo metadata`) |
| External dependencies | Zero — no `source` field in any `Cargo.lock` package entry |
| `unsafe` occurrences | Zero |
| Replaceable-service dependency from `synapse-runtime` | Zero — depends only on `synapse-common` and the seven trusted-core crates |
| Reverse dependency onto `synapse-runtime` | None found |
| Post-commit working tree | Clean |

### C.3 Remote and Push Status

`synapse-runtime` has no remote configured. The commit above exists **locally only** — it has not been pushed, and cannot be, until a remote is configured (a decision not made by this report). This is a factual repository-state limitation, not a defect in the implementation.

### C.4 Boundary: Engineering Completion Versus Formal Approval Evidence

The following are distinct, and this report deliberately does not collapse them:

- **Engineering work completed** — true, per Part B and C.1–C.2.
- **Engineering report produced** — true, this document.
- **Architectural review performed** — the composition-root gap (Part A) was reviewed and resolved by the Founder via the Authorized Implementation Clarification; no further architectural review issue was identified (§B.11).
- **Formal approval evidence recorded** — **not true.** No STD-001 §31 content-non-mutating evidence commit exists for EWO-001, ER-001, or the Runtime implementation commit identified in C.1. No approver identity, disposition, or authority citation has been recorded for any of them. Whether such evidence should bind to the Runtime commit, its tree, or another artifact set, who holds approval authority for it, and whether a local-only (unpushed) commit is sufficient for that purpose, are unresolved questions this report does not decide.

`synapse-docs` was not modified in the production of this Part C beyond this report and EWO-001 itself; no ADR, ARCH, GOV, or STD document was touched.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-11 | Claude (AI-assisted) | Initial report: EWO-001 halted before implementation per its own Definition of Failure. |
| 0.2.0 | 2026-07-11 | Claude (AI-assisted) | Restructured as Part A (original halt, retained as historical record) / Part B (resolution and completion). Records the Founder's Authorized Implementation Clarification, full implementation summary, tests, and validation results. Task state changed from halted to resumed and completed. |
| 0.2.0 | 2026-07-11 | Claude (AI-assisted) | Repository reconciliation addition (no version bump — Draft-phase addition, consistent with EWO-001's treatment in the same reconciliation task): added Part C, recording the Runtime implementation's actual committed-artifact identity (commit `00aa309...`, parent `25c0ea0e...`), re-run post-commit validation results, remote/push status (none configured, local-only), and an explicit statement that this report does not itself constitute STD-001 §31 approval evidence. Parts A and B are otherwise unchanged. |
