---
document_id: ER-004
title: Runtime Integration Bootstrap — Host Execution Handle Binding — Engineering Report
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-13
last_updated: 2026-07-13
classification: Public
related_documents:
  reports_on: EWO-004 (work-orders/EWO-004-Runtime-Integration-Bootstrap.md)
  architecture:
    - ARCH-001
    - ARCH-002
    - ARCH-003 (architecture/ARCH-003-Runtime-Integration-Architecture.md)
  adrs:
    - ADR-0015
    - ADR-0016
    - ADR-0017
  standards:
    - STD-001
    - STD-002
    - STD-004
    - STD-011
---

# ER-004 — Runtime Integration Bootstrap — Host Execution Handle Binding — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize anything.

## 1. Repository Verification

Verified before implementation began:

- `synapse-runtime`: path `~/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `ee0b691383c11009538809542e4f1cd185020dc9`, `HEAD == origin/main`, 0 ahead / 0 behind, tracked working tree clean, nothing staged.
- `synapse-docs`: path `~/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `7824d7705ebde3cd507a6f159d3334f38a86b727`, `HEAD == origin/main`, tracked working tree clean. The three protected untracked files (`.ai/ARCHITECTURAL-CONTEXT.md`, `maintenance/EMO-001-ActorHost-Single-Live-Instance.md`, `work-orders/EWO-003-Message-Gateway.md`) confirmed present and untouched throughout.
- EWO-004 (v0.2.0), ARCH-003, and ARCH-002 read in full before any change. The affected source (`core/execution-coordinator/{src/lib.rs,src/internal.rs}`, `runtime/src/lib.rs`, `core/host-adapter/{src/lib.rs,src/internal.rs}`, `runtime/tests/bootstrap.rs`, both READMEs) was re-read directly against the source tree rather than assumed to match EWO-004's own "Existing Baseline" section — it matched exactly, with no drift since ARCH-003 and EWO-004 were authored.

## 2. Implementation Summary

Implemented exactly the one Runtime Integration milestone EWO-004 authorizes: Runtime now acquires a Host Execution Handle from Host Adapter for each execution, ensures the Execution Context used during that execution contains it, and releases the same handle back to Host Adapter after the execution concludes on every path — successful or not. Runtime remains the sole composition root realizing this behaviour (ADR-0016): Execution Coordinator and Host Adapter still never reach into each other, directly or otherwise. No actor execution, capability enforcement during execution, Lifecycle Guardian interaction, mailbox change, or any other later Runtime Integration item was implemented. All validation gates pass: `cargo fmt --all -- --check`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`, `cargo build --workspace`, `cargo test --workspace` (all 208 workspace tests pass, 0 failures), `cargo tree --workspace`. Zero new dependencies, zero `unsafe`, zero new Trusted Core component, zero new lifecycle state or audit event.

## 3. Files Changed

| File | Change |
|---|---|
| `core/execution-coordinator/src/lib.rs` | `ExecutionCoordinator::construct_context` trait signature gained a `handle: HostExecutionHandle` parameter; `ExecutionCoordinatorHandle`'s delegating implementation updated to match; 1 new unit test added (14 → 15) |
| `core/execution-coordinator/src/internal.rs` | `ExecutionCoordinatorImpl::construct_context` accepts and embeds the supplied handle (`host_execution_handle: handle`) instead of constructing `HostExecutionHandle` itself; module- and method-level doc comments updated for accuracy |
| `core/execution-coordinator/README.md` | Status section updated to describe the real Host Execution Handle binding now present |
| `runtime/src/lib.rs` | `Runtime::execute_message` rewritten to acquire, supply, and release a Host Execution Handle around the existing construct/dispatch/complete sequence; new private helper `Runtime::release_and_fail_execution`; 2 new unit tests added (46 → 48) |
| `runtime/README.md` | Status section updated to describe the first Runtime Integration milestone now implemented |

No file outside these five was touched. `common/src/lib.rs` is unmodified — `HostExecutionHandle`'s own definition is unchanged. `core/host-adapter/` is unmodified. `runtime/tests/bootstrap.rs` is unmodified (see §8 for why the balance-invariant tests could not be placed there).

## 4. Interface Evolution Selected

`ExecutionCoordinator::construct_context`'s signature gained one additional parameter, `handle: HostExecutionHandle`, passed by value:

```rust
fn construct_context(
    &mut self,
    instance: &ActorInstanceId,
    message: &Message,
    handle: HostExecutionHandle,
) -> Result<ExecutionContext, RuntimeError>;
```

The implementation embeds this parameter directly into the returned context's `host_execution_handle` field, replacing the bare `HostExecutionHandle` marker construction the prior implementation used.

### Rationale for selecting this as the minimum evolution

EWO-004's "Construction Surface" left open which method's signature changes, whether a new method is introduced instead, and whether the handle is supplied at construction time or attached afterward — provided the change stays entirely within Execution Coordinator's own interface. Three options were considered:

1. **Add a parameter to the existing `construct_context` (selected).** One additional parameter on an already-existing method. No new public method, no new public type, no window in which an `ExecutionContext` exists without its real handle already embedded — the context is fully and atomically formed in one call, exactly as it already was before this change.
2. **Introduce a new method (e.g., a distinct `attach_host_execution_handle` called after `construct_context`).** Rejected: this would require two calls where one suffices, would introduce a transient state (a constructed-but-handle-less context) that ARCH-002 §10 never describes, and would be a strictly larger interface surface than option 1 for no additional benefit — violating "the smallest interface evolution."
3. **Have Execution Coordinator obtain the handle itself, directly from Host Adapter.** Rejected outright: this would give `synapse-execution-coordinator` a direct dependency on `synapse-host-adapter`, an explicit ADR-0016 and EWO-004 Architecture Constraints violation. Not seriously considered as a candidate; recorded here only to show it was recognized and excluded, not overlooked.

Option 1 is the smallest change satisfying the required behaviour: it touches exactly one method, adds exactly one parameter, introduces no new type, and preserves construction as a single atomic act. `ExecutionCoordinatorHandle`'s trait delegation required a purely mechanical, one-line update to match — no behavioural change beyond passing the parameter through, exactly as every other delegated method already does.

## 5. Behaviour Implemented

`Runtime::execute_message` now sequences, in order:

1. Actor Host: `live_instance` (unchanged).
2. Host Adapter: `allocate_execution_handle()` — new. On failure (currently unreachable, since allocation is unconditional), routes directly to `fail_execution`; no handle exists to release.
3. Execution Coordinator: `construct_context(&instance, &message, handle.clone())` — the cloned handle is supplied; the original is retained for release regardless of outcome. On failure, routes to the new `release_and_fail_execution` helper.
4. Execution Coordinator: `dispatch(context)` — unchanged logic; on failure, routes to `release_and_fail_execution`.
5. Execution Coordinator: `complete(&context)` — unchanged logic; on failure, routes to `release_and_fail_execution`.
6. Host Adapter: `release_execution_handle(handle)` — new. On failure, `execute_message` reports that failure instead of success, via `fail_execution`.
7. Audit Emitter: `execution.completed` (unchanged).

`Runtime::release_and_fail_execution(message, handle, reason)` — new private helper: attempts to release `handle`; if release itself fails, its own error is returned (via `fail_execution(message, release_reason)`), overriding `reason`; if release succeeds, the original `reason` is reported (via `fail_execution(message, reason)`), exactly as before this milestone.

`Runtime::execute_message`'s own public signature (`fn execute_message(&mut self, message: Message) -> Result<(), RuntimeError>`) is byte-for-byte unchanged — confirmed by inspection and by every pre-existing caller (unit and integration tests) compiling and passing unmodified.

## 6. Invariants Enforced

- **Handle-value fidelity.** The Execution Context used during a given execution contains the exact `HostExecutionHandle` value Runtime supplied to `construct_context`, never a separately constructed default — a direct, reviewable consequence of `ExecutionCoordinatorImpl::construct_context`'s own implementation (`host_execution_handle: handle`), confirmed by code inspection (§9 addresses why this cannot additionally be confirmed by a runtime equality assertion).
- **Allocate/release balance on every path.** Every handle Host Adapter allocates on behalf of one `execute_message` call is released back to it exactly once before that call returns — on the successful-completion path, and on rejection during Execution Context construction, dispatch, or completion. Verified directly by two new unit tests (§8).
- **Error-priority ordering on release failure.** If release fails after an original construction/dispatch/completion failure, the release's own error is what `execute_message` ultimately returns, not the original triggering error — mirroring the same precedence `fail_execution` and `reject_message` already apply to audit-emission failure. If release fails after `complete` has already succeeded, `execute_message` reports that failure instead of success, mirroring ADR-0015's existing "a later mandatory step's failure overrides an otherwise-successful outcome" rule. Both branches are implemented and code-reviewable; neither is independently unit-testable, because Host Adapter's `release_execution_handle` cannot be made to fail under any sequence of calls reachable from outside its own crate given this workspace's single-threaded, fully synchronous execution model (no async runtime, no spawned threads exist anywhere) — exactly as EWO-004's own "Handle Lifecycle Invariant" anticipated ("a release failure occurring in practice would indicate a genuine internal-consistency defect, not a normal error case").

## 7. Tests Added

`core/execution-coordinator/src/lib.rs` (14 → 15 tests):

- `construct_context_accepts_and_embeds_a_supplied_host_execution_handle` — proves the call accepts the new parameter and succeeds, producing a context with a populated `host_execution_handle` field. Its own doc comment records precisely why runtime equality against the supplied value cannot additionally be asserted: `HostExecutionHandle` is a zero-field type with no `PartialEq` implementation, so every value of the type is indistinguishable from every other by construction — the same limitation EWO-004's own Acceptance Criteria already disclosed.
- All 13 pre-existing tests updated mechanically to supply a `HostExecutionHandle` third argument at each `construct_context` call site; no test's own assertions or intent changed.

`runtime/src/lib.rs` (46 → 48 tests):

- `execute_message_leaves_host_adapter_balanced_after_success` — after a successful `execute_message`, a direct probe release (`runtime.core.host_adapter.release_execution_handle(HostExecutionHandle)`) is asserted to fail with `IllegalTransition`, proving no handle remains outstanding. A leaked handle would instead let this probe succeed, failing the assertion and correctly catching the defect.
- `execute_message_leaves_host_adapter_balanced_when_execution_context_construction_fails` — pre-seeds Execution Coordinator's own tracked state (via same-crate access to `runtime.core.execution_coordinator`, mirroring the established pattern other tests in this file already use for `runtime.core.capability_authority`) so the target instance already has a construction in progress, forcing `execute_message`'s own `construct_context` call to fail; the same balance probe then confirms the handle `execute_message` allocated before that failure was still released, not leaked.

`runtime/tests/bootstrap.rs`: unmodified. See §8 for why.

## 8. Validation Results

All Mandatory Validation gates pass:

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | All tests pass, 0 failures (per-crate breakdown below) |
| `cargo tree --workspace` | Clean, no cycle |

| Crate | Tests | Result |
|---|---|---|
| `synapse-execution-coordinator` | 15 (14 pre-existing + 1 new) | All pass |
| `synapse-runtime` unit tests | 48 (46 pre-existing + 2 new) | All pass |
| `synapse-runtime` integration tests (`tests/bootstrap.rs`) | 16 (unchanged) | All pass |
| Every other workspace crate (`synapse-actor-directory`, `synapse-actor-host`, `synapse-api`, `synapse-audit-emitter`, `synapse-audit-pipeline`, `synapse-capability-authority`, `synapse-common`, `synapse-host-adapter`, `synapse-lifecycle-guardian`, `synapse-message-gateway`, `synapse-persistence`, `synapse-scheduler`) | unchanged | All pass |

**On the placement of the balance-invariant tests.** EWO-004's own Required Tests section asked for integration tests, within `runtime/tests/`, proving the allocate/release balance invariant — with an explicit anticipation that "where one is not independently reachable through the public API, that limitation is stated in ER-004, not silently assumed away." Host Adapter's own outstanding-allocation counter is `pub(crate)` to `synapse-host-adapter` alone, with no public accessor anywhere (SRP-007, unchanged by this milestone). `runtime/tests/bootstrap.rs` is an external integration-test crate that sees only `synapse_runtime::Runtime`'s public API; it has no way to obtain a reference to `runtime.core.host_adapter` at all (a private field), and no successful or failed call through `Runtime`'s own public methods differs observably based on the counter's exact internal value — a leaked handle would never cause any externally visible symptom, since `allocate_execution_handle` is unconditional and the counter only ever grows on a leak, never causing an unexpected rejection. The balance invariant is therefore provable only via same-crate, private-field access to `runtime.core.host_adapter` — exactly the same class of access `issue_capability_without_audit` and `running_with_emitter` already use for `runtime.core.capability_authority`, and exactly the same class of limitation this workspace already documented for Actor Host's mailbox contents ("Runtime has, and must have, no accessor to re-verify mailbox contents directly"). The two new tests were accordingly placed in `runtime/src/lib.rs`'s own test module, not `runtime/tests/bootstrap.rs`. `runtime/tests/bootstrap.rs`'s existing `execute_message_succeeds_for_a_live_instance` test continues to pass unmodified and now exercises the new internal Host Adapter wiring as part of its own regression coverage, since `execute_message`'s public behaviour is otherwise unchanged.

**On dispatch- and completion-failure path reachability.** `execute_message` always runs Execution Context construction, dispatch, and completion synchronously to conclusion within one call, using the context its own construction step just returned. Because dispatch and completion can only fail when the passed context does not match Execution Coordinator's own tracked state, and `execute_message` never passes any context but the one it just freshly, successfully constructed, there is no sequence of calls through `execute_message`'s own public API that can make its internal `dispatch` or `complete` calls fail independently of the construction-failure case already tested. The `release_and_fail_execution` code path both failure points share with the construction-failure case is the same code, already exercised by `execute_message_leaves_host_adapter_balanced_when_execution_context_construction_fails`; no additional, independently-reachable test scenario exists for these two specific failure points, consistent with EWO-004's own acknowledgement that this might be the case.

## 9. Dependency Verification

- `synapse-execution-coordinator`'s `Cargo.toml`: unchanged, `synapse-common` only. `cargo tree -p synapse-execution-coordinator` confirms exactly one dependency edge, to `synapse-common`.
- `synapse-host-adapter`'s `Cargo.toml`: unchanged, `synapse-common` only. `cargo tree -p synapse-host-adapter` confirms exactly one dependency edge, to `synapse-common`.
- `cargo tree --workspace -i synapse-execution-coordinator` and `-i synapse-host-adapter`: each shows exactly `synapse-runtime` as the sole dependent — no crate other than the Runtime composition root depends on either.
- No new crate dependency, in any direction, was added anywhere in the workspace. Workspace member count unchanged at 14 (`cargo metadata`).
- Zero `unsafe` anywhere in `core/` or `runtime/` (confirmed by direct search).
- No dependency cycle (confirmed by `cargo tree --workspace` completing without a cycle diagnostic).

## 10. Trusted Core Verification

- Still exactly seven Trusted Core components; no new component introduced.
- No responsibility transferred between components: Host Adapter's own interface (`allocate_execution_handle`, `release_execution_handle`) is untouched — SRP-007's implementation, error semantics, and internal counter logic are byte-for-byte unchanged. Execution Coordinator's own responsibility (execution-context coordination, one-owner enforcement) is unchanged beyond the single new input parameter; its `active: HashMap<ActorInstanceId, ExecutionState>` tracking logic is unchanged.
- No direct dependency exists, or was introduced, between `synapse-execution-coordinator` and `synapse-host-adapter` in either direction (§9). The only entity holding both is `Runtime`, exactly as ADR-0016 requires.
- No new peer interaction path was established by either component independently; every new call in this milestone originates from `Runtime::execute_message` alone.

## 11. Architecture Verification

- ARCH-002 §6, §10: Execution Coordinator remains the sole component constructing the Execution Context; the `host_execution_handle` field ARCH-002 §10 already names is now populated with a genuine value rather than a bare marker, with no other field of `ExecutionContext` touched.
- ARCH-002 §12: single-owner, non-reentrant execution is unchanged — `ExecutionCoordinatorImpl`'s own `active` map logic, which enforces this, was not modified.
- ARCH-003 §11 (Host Execution-Handle Boundary): the boundary ARCH-003 identified but declined to design is now closed by exactly the "future, separately authorized decision about `Execution Coordinator`'s own trait signature" ARCH-003 anticipated. `HostExecutionHandle`'s own zero-field definition is unchanged — no claim of identity or unforgeability is made anywhere in this implementation, consistent with ARCH-003 §11's own position that this is a known, disclosed constraint, not a defect this milestone was asked to resolve.
- ARCH-003 §17 (Integration Invariants): invariant 5 ("Execution Coordinator owns execution-context coordination... and nothing beyond it") and invariant 6 ("Host Adapter owns host interaction within its own interface... and is not reached by any other component directly") both continue to hold, unchanged, after this milestone.
- No architecture document was modified. No ADR was modified. No governance document was modified.

## 12. Deferred ARCH-003 Items Remaining

Of ARCH-003 §5's disclosed integration gaps and §18's deferred-work inventory, the following remain deferred, unchanged in status by this milestone:

- Execution Coordinator ↔ Lifecycle Guardian integration (dispatch start/end are still not communicated to Lifecycle Guardian's own tracked state).
- The identity limitation of `HostExecutionHandle` itself (still zero-field; a specific handle's provenance still cannot be verified) — unaffected by this milestone, which relocates where the value comes from without changing what the value is.
- A complete actor execution flow (no actor-defined message-handling logic is invoked by `dispatch`).
- Capability revalidation during restoration and at invocation (`ExecutionContext.active_capabilities` remains empty).
- Successful suspend/restore reachability (still blocked on the first item above).
- Bounded mailbox capacity and audited overflow handling.
- End-to-end audit sequencing, deterministic cleanup after partial execution failure, a first runnable actor, an end-to-end Runtime demonstration, and SDK/host-specific implementation work.

**What has newly become implemented-and-integrated:** Execution Coordinator ↔ Host Adapter integration — the one ARCH-003 §18 item this milestone targeted. ARCH-003 §20 requires this document's own recorded outcome to trigger an explicit update to ARCH-003 §5 (Current Implementation Baseline) and §18 (Deferred Integration Work) reflecting this. Per EWO-004's own Phase 4 instruction, that ARCH-003 update is intentionally **not** made as part of this report or this commit — it is deferred to a separate, subsequent conformance-update task, so that ARCH-003's own revision history records the update as a distinct, reviewable act rather than folding it silently into an implementation commit.

## 13. Engineering Decision Log

- **Implementation decisions:** `construct_context` gained a by-value `HostExecutionHandle` parameter (§4); the caller (`Runtime`) clones the handle before supplying it to `construct_context`, retaining the original for guaranteed release regardless of outcome; a new private `Runtime::release_and_fail_execution` helper centralizes the release-then-report-failure sequence used on every non-success path.
- **Repository decisions:** the two balance-invariant tests were placed in `runtime/src/lib.rs` rather than `runtime/tests/bootstrap.rs`, for the reasons given in §8.
- **Deferred decisions:** none beyond what EWO-004 itself already deferred (§12).
- **Architectural decisions:** None. No ARCH-002 or ARCH-003 content was reinterpreted; the one interface evolution made was explicitly anticipated and authorized by EWO-004's own "Construction Surface."
- **Constitutional decisions:** None.
- **Future work enabled:** a documented, working, Runtime-mediated, ADR-0016-compliant pattern (acquire from one component, supply to another, release regardless of outcome, with a centralized release-then-report-failure helper) that a future Execution Coordinator ↔ Lifecycle Guardian integration milestone can reuse directly.
- **Future work deferred:** everything listed in §12, unchanged in status by this milestone.

## 14. Final Implementation Assessment

EWO-004 v0.2.0 is complete against its own Definition of Done. No Definition-of-Failure condition was encountered: no constitutional contradiction, no case where ARCH-002 or ARCH-003 could not be implemented as written, no Trusted Core expansion, no new Runtime abstraction beyond the single local, call-scoped handle value, no direct dependency between Execution Coordinator and Host Adapter, and no interface evolution larger than the single-parameter addition selected. All Mandatory Validation gates pass with zero warnings, zero `unsafe`, zero new dependency of any kind. Changes are confined to exactly the five files EWO-004's own Repository Constraints authorize. The repository is left unstaged and uncommitted at the point this report was prepared, consistent with EWO-004's own Implementation Sequence (ER-004 preparation only after implementation and validation are complete, and not itself part of the commit this report will accompany). ARCH-003 itself is intentionally left unrevised, per §12 above, pending a separate, subsequent conformance-update task.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-13 | Claude (AI-assisted) | Initial report following EWO-004 v0.2.0 implementation. |
