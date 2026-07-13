---
document_id: EWO-004
title: Runtime Integration Bootstrap — Host Execution Handle Binding
version: 0.2.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-13
last_updated: 2026-07-13
classification: Public
related_documents:
  standards:
    - STD-001
    - STD-002
    - STD-004
    - STD-011
  architecture:
    - ARCH-001
    - ARCH-002
    - ARCH-003 (architecture/ARCH-003-Runtime-Integration-Architecture.md)
  adrs:
    - ADR-0011
    - ADR-0012
    - ADR-0013
    - ADR-0014
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
    - ADR-0017 (Approved)
  predecessor: EWO-003 (work-orders/EWO-003-Message-Gateway.md) — prior EWO in sequence; the substantive predecessor authority for this milestone is ARCH-003, not EWO-003's own subject matter
  reported_by: ER-004 (engineering-reports/ER-004-Runtime-Integration-Bootstrap.md, not yet created)
---

# EWO-004 — Runtime Integration Bootstrap — Host Execution Handle Binding

Registered per STD-001 §46 (Engineering Work Orders). This is the first Engineering Work Order issued under the Runtime Integration phase ARCH-003 opens. Revised following Architecture Review Board review; disposition: **Approved with required revisions**. See Revision History.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-004 |
| Title | Runtime Integration Bootstrap — Host Execution Handle Binding |
| Version | 0.2.0 |
| Status | Draft |
| Author | Denver Jacobs |
| Created | 2026-07-13 |
| Last Updated | 2026-07-13 |
| Classification | Public |
| Predecessor milestone | EWO-003 — Message Gateway (numbering sequence only; see below) |
| Reported by | ER-004 (engineering-reports/ER-004-Runtime-Integration-Bootstrap.md, not yet created) |

---

## Engineering Authority

This implementation is governed by, in descending order:

1. ARCH-001
2. ARCH-002 (as amended at commit `2629241`, integrating ADR-0015 and ADR-0016)
3. ARCH-003 — Runtime Integration Architecture (architecture/ARCH-003-Runtime-Integration-Architecture.md), the specific authority for this milestone's scope, sequencing, and boundaries
4. ADR-0011
5. ADR-0012
6. ADR-0013
7. ADR-0014
8. ADR-0015 — Audit Emitter Failure Semantics (Approved)
9. ADR-0016 — Trusted Core Interaction Rule (Approved)
10. ADR-0017 — Bootstrap Capability Trust Root (Approved)
11. STD-001
12. STD-002
13. STD-004
14. STD-011

These documents are authoritative. This task implements them. It does not reinterpret or modify them.

---

## Objective

Implement the first Runtime Integration milestone ARCH-003 §18 identifies: **Execution Coordinator ↔ Host Adapter integration**, so that a real execution's `ExecutionContext.host_execution_handle` carries a genuine value obtained from Host Adapter's `allocate_execution_handle`, rather than the bare `HostExecutionHandle` marker value the current implementation causes the Execution Context to carry unconditionally — and so that every handle so allocated is released back to Host Adapter exactly once, regardless of whether the execution that used it ultimately succeeds or fails.

Nothing else. This is a narrow, mechanical wiring milestone — a "bootstrap" for the Runtime Integration phase in the same sense EWO-001 was a bootstrap for the Runtime itself: it establishes the composition-root pattern (Runtime obtains a value from one Trusted Core component and supplies it to another, per ADR-0016) that later Runtime Integration milestones will reuse, without itself attempting any of the larger integration work ARCH-003 §18 also names.

Completion means: Runtime acquires a Host Execution Handle from Host Adapter for each execution; Runtime ensures the Execution Context used during that execution contains that handle; and Runtime releases the handle back to Host Adapter after the execution concludes — on every path, successful or not — backed by tests that exercise the balance invariant directly. Runtime remains the sole composition root realizing this behaviour; no Trusted Core component gains a new peer dependency as a result; and the smallest interface evolution consistent with published architecture is used to make the handle reach the Execution Context, wherever that evolution is objectively required. If implementation determines that satisfying this behaviour requires a larger architectural change than the smallest such evolution, implementation stops and an Engineering Report is produced, per "Definition of Failure" below.

---

## Existing Baseline (verified against the source tree, not assumed)

Verified directly against `synapse-runtime` @ `ee0b691383c11009538809542e4f1cd185020dc9`, the same commit ARCH-003 §5 verified against:

- `runtime/src/lib.rs`'s `Runtime::execute_message` calls, in order: `actor_host.live_instance(&message.destination)`, `execution_coordinator.construct_context(&instance, &message)`, `execution_coordinator.dispatch(context)`, `execution_coordinator.complete(&context)`, then `audit_emitter.emit(execution_completed_event(&message))`. A failure at any of the first three calls routes to `fail_execution`, which emits `execution.failed` and returns the triggering error.
- `TrustedCore` already constructs and holds a `HostAdapterHandle` (SRP-007). `Runtime` never calls any method on it — confirmed directly: no occurrence of `self.core.host_adapter` exists anywhere in `runtime/src/lib.rs` outside `TrustedCore::construct` and the test-only `running_with_emitter` helper.
- `synapse_execution_coordinator::internal::ExecutionCoordinatorImpl::construct_context(&mut self, instance: &ActorInstanceId, message: &Message) -> Result<ExecutionContext, RuntimeError>` populates the returned `ExecutionContext`'s `host_execution_handle` field with the bare `HostExecutionHandle` value directly — confirmed directly in `core/execution-coordinator/src/internal.rs`. Its own module documentation states this is because "nothing in `construct_context`'s signature supplies" a real handle, and that "what... 'delegate host execution through Host Adapter' will mean once those components exist is therefore not invented here."
- `HostAdapterHandle::allocate_execution_handle(&mut self) -> Result<HostExecutionHandle, RuntimeError>` and `::release_execution_handle(&mut self, handle: HostExecutionHandle) -> Result<(), RuntimeError>` are both fully implemented and already published (SRP-007): allocation is unconditional and always increments a private outstanding-count; release requires `outstanding > 0`, decrementing on success and returning `Err(RuntimeError::IllegalTransition)` otherwise.
- `HostExecutionHandle` (`synapse-common`) is `#[derive(Debug, Clone, Default)] pub struct HostExecutionHandle;` — zero fields, carries no identity. This is unchanged by this EWO and is not proposed to change.
- `synapse-execution-coordinator`'s `Cargo.toml` already depends on exactly `synapse-common` — no other dependency. `HostExecutionHandle` is already imported and used in `core/execution-coordinator/src/internal.rs` (via `ExecutionContext`'s own field), so no new crate dependency of any kind is required by this EWO — confirmed directly, regardless of which interface evolution implementation ultimately selects.
- ARCH-003 §11 (Host Execution-Handle Boundary) states this connection "requires a future, separately authorized decision about `Execution Coordinator`'s own trait signature," and identifies but does not prescribe it. ARCH-003 §18 lists "Execution Coordinator ↔ Host Adapter integration" as deferred work. ARCH-003 §20 states a future work order authorizing any §18 item "must itself be evaluated against ARCH-002 and this document before implementation begins, per the existing EWO process (STD-001 §46)." This EWO is that authorization, for that one item, and no other — it authorizes the behaviour ARCH-003 §11 identified, not the specific signature decision it declined to make.

This EWO's job is to give Runtime the narrow, additional composition-root behaviour ARCH-003 identified but deliberately did not design, and to give Execution Coordinator's own public interface the minimum evolution that behaviour objectively requires — not to redesign either component, not to prescribe the exact form of that evolution, not to touch any other deferred item ARCH-003 §18 lists, and not to make any of them reachable through this milestone's own change.

---

## Scope

Implement only:

- The minimum public interface evolution to Execution Coordinator's own trait and implementation, objectively required so that the Execution Context used during a given execution contains the Host Execution Handle Runtime acquired for that execution, rather than a default/marker value. See "Construction Surface" below for the precise, bounded authorization.
- The minimum Runtime composition-root change (`runtime/src/lib.rs`, `Runtime::execute_message` and its failure path) sufficient to: obtain a handle from Host Adapter before the Execution Context for that execution is constructed; ensure that Execution Context contains it; and release the same handle back to Host Adapter after the execution concludes, on every path — success or failure — per "Handle Lifecycle Invariant" below.
- Unit tests, within `synapse-execution-coordinator`, proving the Execution Context Execution Coordinator produces contains exactly the Host Execution Handle supplied to it (not a fabricated or default one).
- Integration tests, within `runtime/tests/`, proving the allocate/release balance invariant holds on both the successful-execution path and each execution-failure path (Execution Context construction rejection, dispatch rejection, completion rejection).
- Documentation updates to `core/execution-coordinator/README.md` and `runtime/README.md` reflecting the real behaviour now present, and the continued honest disclosure that `HostExecutionHandle` itself remains zero-field and identity-free (unchanged by this EWO).

---

## Out of Scope

Do NOT implement:

- Actor scheduling of any kind.
- Actor execution — no actor-defined message-handling logic is invoked by this EWO; `dispatch` and `complete` retain exactly their current, non-invoking mechanism-enforcement behaviour beyond whatever minimum evolution this EWO authorizes. This EWO changes only how the Host Execution Handle reaches the Execution Context, never what happens to that context afterward.
- Capability enforcement during execution — `ExecutionContext.active_capabilities` remains untouched, populated empty, exactly as it is today. Capability revalidation at invocation (ARCH-003 §13, §18) is explicitly excluded from this milestone.
- Lifecycle transitions beyond existing bootstrap behaviour — this EWO does not call `LifecycleGuardian` in any way, does not mark any instance `Executing`, and does not change suspend/restore reachability (ARCH-003 §12). Execution Coordinator ↔ Lifecycle Guardian integration (ARCH-003 §18) is a distinct, later milestone, not this one.
- Distributed Runtime behaviour of any kind.
- Persistence, snapshotting, or restoration.
- Plugins or any extension mechanism.
- Supervision or restart policy.
- Mailbox redesign of any kind.
- Bounded mailbox capacity implementation (ARCH-003 §5, §9, §18 — a separate, already-identified, later deferred item).
- Mailbox overflow handling of any kind.
- Any change to `HostExecutionHandle`'s own definition (still zero-field; its identity limitation, per ARCH-003 §11 and §18, remains explicitly out of scope for this or any milestone until a future requirement and a separate architectural decision authorize changing the shared type itself).
- Any change to `dispatch`'s or `complete`'s own signature or behaviour beyond what the minimum interface evolution ("Construction Surface") objectively requires — this milestone's own behavioural requirement concerns only how the Host Execution Handle reaches the Execution Context, not dispatch or completion mechanics, which otherwise remain unchanged.
- Any change to Host Adapter's own trait, internal counter logic, or error semantics (SRP-007's implementation is correct and untouched; see "Construction Surface").
- A new audit event of any kind. ARCH-002 §18 does not name Host Adapter allocation or release among the minimum audit events, and ARCH-003 §14 confirms none should be invented. This EWO does not add one.
- A direct dependency from `synapse-execution-coordinator` to `synapse-host-adapter`, or from `synapse-host-adapter` to `synapse-execution-coordinator`, in either direction. Standing prohibition under ADR-0016 and ARCH-002 §3/§6, reaffirmed by ARCH-003 §6/§17.
- Any construction-surface or public-API change to Host Adapter. Its existing `HostAdapterHandle` and trait are already sufficient (see "Construction Surface").
- Any later Runtime Integration milestone, including but not limited to those ARCH-003 §18 also lists: a complete actor execution flow; capability revalidation during restoration; successful suspend/restore reachability; end-to-end audit sequencing; deterministic cleanup after partial execution failure; a first runnable actor; an end-to-end Runtime demonstration; SDK or host-specific implementation work.

---

## Trusted Core

Execution Coordinator and Host Adapter remain two of exactly seven Trusted Core components (ARCH-002 §6). No new Trusted Core component is introduced. No responsibility is transferred between components: Host Adapter continues to own host interaction exclusively through its own two-method interface; Execution Coordinator continues to own only execution-context coordination and one-owner enforcement. Runtime's new role — obtaining a handle from Host Adapter and ensuring the Execution Context used during execution contains it — is composition-root sequencing, not a transfer of either component's own behavioural authority, exactly as ARCH-003 §6 ("Runtime composes and sequences; it does not decide") and §17 (invariant 7) already require.

---

## Architecture Constraints

Do not:

- introduce new Runtime concepts;
- invent lifecycle states, audit events, or capability checks;
- modify constitutional concepts;
- reinterpret ARCH-001, ARCH-002, or ARCH-003;
- redesign the `ExecutionCoordinator` trait beyond the minimum interface evolution this EWO authorizes (see "Construction Surface");
- redesign the `HostAdapter` trait or `HostAdapterImpl` in any way;
- introduce new crates;
- introduce external dependencies;
- use unsafe Rust;
- give `synapse-execution-coordinator` a dependency on `synapse-host-adapter`, or `synapse-host-adapter` a dependency on `synapse-execution-coordinator`, in either direction. ADR-0016 and ARCH-002 §3/§6 make the Runtime the sole entity accountable for establishing and coordinating Trusted Core interaction; either crate independently reaching the other would itself be exercising that accountability, which belongs to Runtime alone.
- attempt to close `HostExecutionHandle`'s identity limitation (ARCH-003 §11, §18) — this would require changing a shared type outside this EWO's scope and is explicitly deferred pending a future, separate architectural decision.

If implementation requires any of the above: STOP. Produce an Engineering Report explaining the issue. Do not invent a solution.

---

## Repository Constraints

Do not modify: governance documents; architecture documents; standards; work orders other than this one; repository structure beyond `core/execution-coordinator/` (the only Trusted Core component ARCH-002 §6 authorizes to construct or evolve the Execution Context), the minimum `runtime/` change this EWO authorizes, and their manifests/READMEs. Do not modify `synapse-host-adapter`, `synapse-actor-host`, `synapse-message-gateway`, `synapse-capability-authority`, `synapse-lifecycle-guardian`, `synapse-audit-emitter`, or `common/` in any way — none of them requires any change to realize this milestone.

---

## Construction Surface

The Execution Context ARCH-002 §10 defines already includes a `host_execution_handle` field; ARCH-002 §6 assigns Execution Coordinator sole responsibility for constructing that Execution Context. Consequently, whatever interface evolution is required to make a real, Host-Adapter-issued handle reach that field must occur within Execution Coordinator's own public interface — no other Trusted Core component is authorized to construct or mutate an Execution Context. This is an entailment of already-published architecture, not a decision this EWO makes.

This EWO authorizes only **the minimum public interface evolution objectively required to satisfy the architectural behaviour**: the Execution Context used during a given execution must contain the Host Execution Handle Runtime acquired for that execution.

This EWO does not prescribe:

- which existing method's signature changes, or whether a new method is introduced instead;
- the name, position, type (by value or by reference), or count of any new parameter;
- whether the handle is supplied at construction time or attached to the Execution Context afterward — provided the change is realized entirely within Execution Coordinator's own interface, no Trusted Core ownership boundary is crossed, and one execution's Execution Context construction remains governed by exactly the invariants ARCH-002 §12 already requires (single-owner, not reentrant).

Implementation determines the smallest such evolution. If no interface evolution smaller than a broader redesign of Execution Coordinator's trait, or one touching more than one Trusted Core component's public interface, can satisfy this behaviour, that is a stop condition (Definition of Failure, below), not something to resolve by expanding scope.

No change to `HostAdapterHandle`, the `HostAdapter` trait, or `HostAdapterImpl` is required or authorized — Host Adapter's existing `allocate_execution_handle`/`release_execution_handle` interface, already published under SRP-007, is already sufficient for Runtime to call directly; Runtime already holds a `HostAdapterHandle` inside `TrustedCore`.

The minimum change to the Runtime composition root (`runtime/src/lib.rs`) necessary to sequence acquisition, use, and release of the handle around the existing execution flow, per "Handle Lifecycle Invariant" below, is authorized. No additional public method, type, or abstraction is authorized on `Runtime` itself; `execute_message`'s own existing public signature does not change.

---

## Handle Lifecycle Invariant

This is the one substantive design rule this EWO fixes, stated precisely so implementation does not need to invent it:

- **Every handle Host Adapter allocates on behalf of one `execute_message` call must be released back to Host Adapter exactly once, before that call returns, regardless of outcome.** Host Adapter's own counter-balance model (SRP-007) has no other way to stay correct: `HostExecutionHandle` carries no identity, so an unreleased allocation is a permanent, silent leak of Host Adapter's internal outstanding-count, not merely an inconvenience.
- **Allocation happens once, immediately before the Execution Context for that execution is constructed**, after the existing instance-existence check (Actor Host's `live_instance`) has already succeeded — that check is unchanged and remains first. If allocation itself fails — currently impossible, since `allocate_execution_handle` is unconditional, but the signature returns `Result` — this is treated exactly as an existing `fail_execution` case: no Execution Context is ever constructed, and (since nothing was successfully allocated) no release is attempted.
- **If Execution Context construction, dispatch, or completion fails**, the handle already allocated for this call must still be released before the failure is reported to the caller — otherwise a rejected or failed execution would leak a handle every time, which is a materially worse outcome than the transient, disclosed alternative of allocating a handle that is then immediately released without ever backing a completed execution. This is an accepted, disclosed characteristic of this design, not a violation of "validation before side effects" (ARCH-003 §6): Execution Coordinator's own internal "already executing" check, performed as part of Execution Context construction, cannot be queried separately without a distinct, unauthorized new read-only interface, so the handle must exist before that check can run.
- **If completion succeeds**, the handle must be released before `execution.completed` is emitted.
- **Error priority on the failure path mirrors this codebase's existing, established pattern** (`Runtime::reject_message`/`Runtime::fail_execution`, both already structured as "attempt the mandatory later step; if it fails, its own error is what the caller observes, not the original triggering error"): if release fails after an original Execution Context construction, dispatch, or completion failure, the release's own error is what `execute_message` ultimately returns — not the original triggering error. This is not an invented asymmetry; it is the same precedence rule this codebase already applies to audit emission, applied here to the one additional mandatory step this EWO introduces.
- **On the success path, if release fails after completion has already succeeded**, `execute_message` reports failure (the release's own error) even though the execution itself completed — the same "a later mandatory step's failure overrides an otherwise-successful outcome" rule ADR-0015 already establishes for audit emission, applied here to release instead.
- **In ordinary, correct operation, release cannot fail.** `Runtime` is single-threaded and fully synchronous: one `execute_message` call runs allocate → ... → release to completion before any other `Runtime` method can execute (no async runtime, no spawned threads exist anywhere in this workspace). Host Adapter's shared outstanding-counter therefore never has more than one call's allocation pending release at a time. A release failure occurring in practice would indicate a genuine internal-consistency defect, not a normal error case — implementation must not add any workaround, retry, or suppression for it; it must simply propagate, per the rule above.

---

## Definition of Done

The task is complete only if all of the following are true:

- The Execution Context used during a given execution contains the Host Execution Handle Runtime acquired for that execution — never the bare `HostExecutionHandle::default()`/marker construction — realized via the minimum interface evolution "Construction Surface" authorizes.
- `Runtime::execute_message` obtains a handle from Host Adapter before the Execution Context for that execution is constructed, ensures that Execution Context contains it, and releases the same handle back to Host Adapter exactly once before returning, on every path (successful completion, and rejection during Execution Context construction, dispatch, or completion).
- The allocate/release balance holds after any single `execute_message` call, whatever its outcome: Host Adapter's outstanding count is the same immediately before and immediately after the call.
- Release failure, on the rare/theoretical path it could occur, is surfaced as `execute_message`'s own returned error, per "Handle Lifecycle Invariant."
- No actor-defined message-handling logic is invoked. No capability revalidation is performed. No Lifecycle Guardian call is made. No mailbox behaviour changes.
- No new audit event exists anywhere in the implementation.
- `synapse-execution-coordinator`'s dependency set remains exactly `synapse-common`. `synapse-host-adapter`'s dependency set remains exactly `synapse-common`. No new dependency of any kind, in any crate.
- EWO-001's, EWO-002's, and EWO-003's (SRP-003 through SRP-007) existing tests continue to pass unmodified in behaviour.
- All tests pass. No warnings. No unsafe.
- Trusted Core boundary unchanged (still exactly seven components). Architecture unchanged.

---

## Mandatory Validation

Execute:

```text
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace
cargo test --workspace
cargo tree --workspace
```

All must pass. Additionally verify: workspace member count unchanged; zero `unsafe`; no dependency cycle; `cargo tree -p synapse-execution-coordinator` and `cargo tree -p synapse-host-adapter` each show exactly `synapse-common`; no crate other than the Runtime composition root depends on either; `Runtime::execute_message`'s own public signature is byte-for-byte unchanged.

---

## Data and State Constraints

No new state is required beyond what already exists:

- Execution Coordinator's own internal implementation requires no new field — the supplied handle is used only to populate the Execution Context it produces; it is not additionally stored inside Execution Coordinator's own tracked state.
- `HostAdapterImpl` requires no change of any kind — its existing single `outstanding: u64` counter is already sufficient.
- `Runtime` requires no new field — the handle obtained from Host Adapter is a local value within `execute_message`'s own call, held only for the duration of that call, never stored on `Runtime` itself.
- Do not add: a handle registry, a pending-handle map, a generation counter, or any other bookkeeping beyond the single local value "Handle Lifecycle Invariant" describes. Host Adapter's own internal counter is the only state that tracks allocation/release balance, exactly as SRP-007 already established.

---

## Failure and Error Behaviour

Per ARCH-002 §16 (Failure Model), ARCH-003 §15 (Failure Propagation and Atomicity), and "Handle Lifecycle Invariant" above:

- Actor Host's existing `live_instance` rejection is unchanged: `fail_execution` runs immediately, no handle is ever allocated.
- Host Adapter allocation failure (currently unreachable, since allocation is unconditional) is treated exactly as any other pre-construction failure: `fail_execution` runs, no release is attempted, since nothing was successfully allocated.
- Execution Context construction, dispatch, or completion rejection: the already-allocated handle is released first; if release itself fails, the release's error is what `fail_execution` ultimately reports (per "Handle Lifecycle Invariant"'s error-priority rule); if release succeeds, the original triggering error is reported, exactly as today.
- Successful completion: the handle is released; if release fails, `execute_message` reports that failure instead of success; if release succeeds, `execution.completed` is emitted and `Ok(())` is returned, exactly as today.
- No rollback, transaction model, or compensating action beyond the single release call is required or authorized. This mirrors ARCH-003 §15's existing "no operation performs rollback of already-committed component-level state" — applied here, there is nothing to roll back beyond releasing the one resource this milestone itself introduces.

---

## Definition of Failure

Stop immediately if:

- a constitutional contradiction is discovered;
- ARCH-002 or ARCH-003 cannot be implemented as written;
- Trusted Core expansion becomes necessary;
- a new Runtime abstraction beyond the local, call-scoped handle value appears necessary;
- an architectural decision is required beyond what "Handle Lifecycle Invariant" and "Construction Surface" above already bound;
- a direct dependency between `synapse-execution-coordinator` and `synapse-host-adapter`, in either direction, appears unavoidable;
- `HostExecutionHandle`'s own zero-field definition proves insufficient to complete this milestone (it should not — this milestone only relocates where an existing value comes from, it does not require the value to carry new data) — if it does, this is a distinct, separate architectural question, not something to resolve here;
- no interface evolution smaller than a broader redesign of Execution Coordinator's trait, or one touching more than one Trusted Core component's public interface, can satisfy the required behaviour;
- `release_execution_handle`'s failure semantics prove insufficient for the error-priority rule this EWO specifies;
- satisfying this milestone would require work from a later Runtime Integration milestone (Execution Coordinator ↔ Lifecycle Guardian integration, capability revalidation, bounded mailboxes, actor execution, or any other item ARCH-003 §18 lists beyond this one).

Do not resolve these issues yourself. Report them.

---

## Implementation Sequence

1. Baseline verification: re-confirm the current `core/execution-coordinator/`, `core/host-adapter/`, and `runtime/` state matches this EWO's "Existing Baseline" section exactly.
2. Interface evolution: implement the minimum interface evolution to Execution Coordinator's own trait and implementation necessary to satisfy the required behaviour — the Execution Context used during a given execution contains the Host Execution Handle Runtime acquired for that execution — per "Construction Surface." Do not prescribe or default to a specific API beyond what is objectively required.
3. Component-local tests: unit tests, within `synapse-execution-coordinator`, proving the Execution Context produced contains exactly the supplied handle.
4. Runtime integration: apply the minimum `runtime/src/lib.rs` change to `execute_message` (and its failure path) sequencing allocation, use, dispatch, completion, and release, per "Handle Lifecycle Invariant."
5. Integration tests: within `runtime/tests/`, prove the allocate/release balance invariant on the successful path and on each of the three failure paths.
6. Regression check: confirm all of EWO-001's, EWO-002's, and EWO-003's (SRP-003 through SRP-007) existing tests still pass unmodified in outcome.
7. Documentation updates: `core/execution-coordinator/README.md` and `runtime/README.md`, reflecting the real behaviour now present.
8. Complete quality validation, per Mandatory Validation above.
9. ER-004 preparation after implementation and validation are complete — not before, and not as part of this EWO.

Do not include work from later milestones.

---

## Acceptance Criteria

Every criterion is objective and testable:

- The Execution Context used during a given execution contains the Host Execution Handle Runtime acquired for that execution (not independently verifiable by equality, since `HostExecutionHandle` has no fields to compare — verified instead by direct construction-path inspection: the Execution Context is built from the handle Runtime supplied, not from a separately constructed default).
- A successful `execute_message` call leaves Host Adapter's outstanding-allocation count unchanged from what it was immediately before the call (net zero: one allocation, one release).
- An `execute_message` call that fails during Execution Context construction (instance already executing) leaves Host Adapter's outstanding-allocation count unchanged from before the call.
- An `execute_message` call that fails at dispatch or completion (if reachable given existing Execution Coordinator invariants) leaves Host Adapter's outstanding-allocation count unchanged from before the call.
- `cargo tree -p synapse-execution-coordinator` and `cargo tree -p synapse-host-adapter` each show exactly `synapse-common`.
- Workspace member count unchanged.
- `cargo test --workspace` shows all of EWO-001's, EWO-002's, and EWO-003's (SRP-003 through SRP-007) existing tests still passing, unmodified in outcome.
- No occurrence of `HostExecutionHandle::default()`, bare `HostExecutionHandle`/`HostExecutionHandle {}` construction, or any equivalent, remains anywhere in the code path that produces the Execution Context used during execution.
- No new audit event exists anywhere in `runtime/src/lib.rs`.
- Every item in Out of Scope has zero corresponding code.
- All Mandatory Validation gates pass with zero warnings.
- `git status` after implementation shows changes confined to `core/execution-coordinator/`, the minimum touched parts of `runtime/`, and their manifests/READMEs — no unrelated file touched.

---

## Required Tests

- Unit tests (within `synapse-execution-coordinator`): the Execution Context produced for a given execution contains the Host Execution Handle supplied to Execution Coordinator, not a default/fabricated one; existing construction/dispatch/completion-sequence tests continue to pass with whatever minimum interface evolution is applied.
- Integration tests (within `runtime/tests/`): allocate/release balance after a successful `execute_message`; allocate/release balance after an `execute_message` that fails at each of Execution Context construction, dispatch, and completion (to whatever extent each is independently reachable given existing Execution Coordinator invariants — where one is not independently reachable through the public API, that limitation is stated in ER-004, not silently assumed away); regression tests confirming EWO-001, EWO-002, and EWO-003 (SRP-003 through SRP-007) behaviour is unchanged.
- No test is required for, or should attempt to exercise, any Out-of-Scope behaviour, including actual actor execution, capability revalidation, or any Lifecycle Guardian interaction.

---

## Engineering Decision Log

Record:

- implementation decisions;
- repository decisions;
- deferred decisions;
- architectural decisions (expected: None);
- constitutional decisions (expected: None);
- future work enabled (expected: a documented, working example of the Runtime-mediated, ADR-0016-compliant pattern later Runtime Integration milestones — Execution Coordinator ↔ Lifecycle Guardian integration in particular — can reuse);
- future work deferred (expected: everything ARCH-003 §18 lists beyond this one item, unchanged in status by this EWO).

---

## Completion Report

ER-004 must provide, after implementation:

1. Files modified.
2. Files created (expected: none).
3. Execution Coordinator / Host Adapter integration behaviour implemented, including which interface evolution was selected and why it was the smallest available.
4. Invariants enforced (handle-value fidelity, allocate/release balance on every path, error-priority ordering on release failure).
5. Tests added.
6. Validation results.
7. Dependency changes (expected: none).
8. Trusted Core changes (expected: none — same seven components, same boundaries).
9. Architecture changes (expected: none).
10. Construction-surface and Runtime-integration changes made, exactly as authorized by "Construction Surface" and "Handle Lifecycle Invariant."
11. Engineering Decision Log.
12. Any issues requiring architectural review, including whether any Definition-of-Failure condition was encountered.
13. An explicit statement of which ARCH-003 §5/§18 items remain deferred after this milestone, and confirmation that ARCH-003 itself should be updated (per its own §20 conformance requirement) to reflect this one item's new implemented-and-integrated status.

Stop after this milestone. Do not begin the next Runtime Integration engineering milestone.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-13 | Denver Jacobs | Initial EWO, registered per STD-001 §46. Derived exclusively from ARCH-001, ARCH-002, ARCH-003 (§5, §6, §10, §11, §17, §18, §20), ADR-0011 through ADR-0017, STD-001, STD-002, STD-004, STD-011, and the verified current state of `core/execution-coordinator/`, `core/host-adapter/`, and `runtime/`. Scoped to the single ARCH-003 §18 item — Execution Coordinator ↔ Host Adapter integration — that survives every exclusion named for this milestone. |
| 0.2.0 | 2026-07-13 | Denver Jacobs | Revised following Architecture Review Board review (disposition: Approved with required revisions). Removed normative language prescribing a specific `construct_context` signature change as the implementation mechanism. Rewrote "Construction Surface" to authorize only the minimum public interface evolution objectively required, deriving from already-published ARCH-002 §6/§10 that the evolution must occur within Execution Coordinator's own interface, without prescribing which method, parameter, or form. Reworded Scope, Out of Scope, Definition of Done, Acceptance Criteria, Required Tests, and Implementation Sequence to describe required behaviour (the Execution Context used during execution contains the Host Execution Handle acquired for it) rather than a predetermined API. Left already-published, pre-existing interfaces (`allocate_execution_handle`, `release_execution_handle`, `dispatch`, `complete`, `execute_message`) named throughout, since these are fixed facts this EWO cites, not decisions it makes. Objective's engineering intent, architecture references, Runtime composition-root model, ownership boundaries, Handle Lifecycle Invariant, failure semantics, testing philosophy, stop conditions, quality gates, prohibited shortcuts, and completion criteria are unchanged in substance. |

## Disposition

Approved with required revisions (Architecture Review Board, 2026-07-13). This revision (0.2.0) addresses that disposition.
