---
document_id: EWO-005
title: "Runtime Integration: Truthful Actor Execution-State Tracking"
version: 0.1.0
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
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.3.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
  adrs:
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
    - ADR-0017 (Approved)
  predecessor: EWO-004 (work-orders/EWO-004-Runtime-Integration-Bootstrap.md) — prior Runtime Integration milestone; this EWO implements the first of ARCH-003 §18's two now-split deferred items
  reported_by: ER-005 (engineering-reports/ER-005-Truthful-Execution-State-Tracking.md, not yet created)
  base_state:
    runtime_head: 01047157b6783d816fed80361b40206a98ba6f2f
    docs_head: ac59655c64ebbb09fca67749f46de7c4d0e96dfa
---

# EWO-005 — Runtime Integration: Truthful Actor Execution-State Tracking

Registered per STD-001 §46 (Engineering Work Orders). This is the second Engineering Work Order issued under the Runtime Integration phase ARCH-003 opens, and the first implementation milestone arising directly from ARCH-002 v0.2.0's and ARCH-003 v0.3.0's `Executing`-truthfulness clarification. This document authorizes engineering work only. It does not itself constitute approval, implementation, or completion.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-005 |
| Title | Runtime Integration: Truthful Actor Execution-State Tracking |
| Version | 0.1.0 |
| Status | **Draft** — not yet approved, not yet implemented |
| Author | Denver Jacobs |
| Owner | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-13 |
| Last Updated | 2026-07-13 |
| Classification | Public |
| Applicable repository | `synapse-runtime` |
| Target branch | `main` |
| Runtime base HEAD | `01047157b6783d816fed80361b40206a98ba6f2f` |
| Documentation base HEAD | `ac59655c64ebbb09fca67749f46de7c4d0e96dfa` |
| Predecessor milestone | EWO-004 — Runtime Integration Bootstrap (Host Execution Handle Binding) |
| Reported by | ER-005 (engineering-reports/ER-005-Truthful-Execution-State-Tracking.md, not yet created) |

No implementation exists yet against this EWO. No approval act has occurred. This document authorizes a bounded scope of future engineering work; it does not report on work already done.

---

## Engineering Authority

This implementation is governed by, in descending order:

1. ARCH-001 — Constitutional Architecture
2. ARCH-002 — Runtime Architecture, **v0.2.0** (architecture/ARCH-002-Runtime-Architecture.md) — the authority for the Actor lifecycle state model (§15), the Execution Context model (§10), the `Executing`-truthfulness rule (§10, §12), and the ownership split between Lifecycle Guardian and Execution Coordinator (§6, §10, §15)
3. ARCH-003 — Runtime Integration Architecture, **v0.3.0** (architecture/ARCH-003-Runtime-Integration-Architecture.md) — the specific authority for this milestone's scope and boundaries, in particular §5, §12, and §18's now-split "Truthful execution-state tracking" item
4. ADR-0015 — Audit Emitter Failure Semantics (Approved)
5. ADR-0016 — Trusted Core Interaction Rule (Approved)
6. ADR-0017 — Bootstrap Capability Trust Root (Approved)
7. STD-001 — Documentation Standards (§46, Engineering Work Orders)

These documents are authoritative. This task implements them. It does not reinterpret or modify them.

---

## Purpose

This EWO exists to implement truthful synchronization between:

- the actor-instance `Executing` state Lifecycle Guardian tracks (ARCH-002 §6, §15); and
- the lifetime of a live Execution Context Execution Coordinator tracks (ARCH-002 §6, §10).

This is the **first implementation milestone** arising from ARCH-002 v0.2.0 and ARCH-003 v0.3.0 — the architecture clarification published immediately before this EWO was authored, which made explicit (without redesigning) that an actor instance is truthfully `Executing` if and only if it currently owns a live Execution Context (ARCH-002 §10), and that Lifecycle Guardian and Execution Coordinator each own their own state machine with neither holding authority over the other's (ARCH-002 §15).

---

## Problem Statement

Verified directly against `synapse-runtime` @ `01047157b6783d816fed80361b40206a98ba6f2f` (`runtime/src/lib.rs`, `core/execution-coordinator/src/internal.rs`, `core/lifecycle-guardian/src/internal.rs`), not assumed:

- `Runtime::execute_message` sequences Execution Coordinator's `construct_context` → `dispatch` → `complete` around a Host Adapter handle (EWO-004). Execution Coordinator's own private `active: HashMap<ActorInstanceId, ExecutionState>` genuinely tracks a live Execution Context for the duration of that sequence.
- **Lifecycle Guardian is never called anywhere in this flow.** Confirmed directly: zero occurrences of `self.core.lifecycle_guardian` in `Runtime::execute_message`.
- Consequently, for the entire duration a live Execution Context genuinely exists, Lifecycle Guardian's own tracked state for that instance remains whatever it was before — `Idle`, by default, for any instance that has not been separately, artificially marked otherwise — never `Executing`.
- This is a direct instance of the mismatch ARCH-002 v0.2.0 (§10) now names explicitly: an actor instance can genuinely own a live Execution Context while Lifecycle Guardian's own tracked state does not reflect it. Before this milestone, Lifecycle Guardian's state was never wrong in a way anyone could observe (nothing read it during execution), but it was never made *right* either — this EWO closes that gap by causing Lifecycle Guardian to be told.
- **This is not the same problem as suspend/restore unreachability.** ARCH-002 §12 and ARCH-003 §5/§12/§18 (v0.2.0/v0.3.0, both already corrected) establish that even after this milestone, no Runtime API call other than the one currently performing a given execution can observe that instance as `Executing`, because the current Runtime is fully synchronous — genuine external suspend/restore reachability against an actively-executing instance requires a real asynchronous or otherwise interruptible execution mechanism, which does not exist and is not introduced here. This EWO makes Lifecycle Guardian's *own* tracked state truthful; it does not, and is not attempting to, make that state observable from any call other than the one already in progress.

---

## Architectural Authority

Distinguishing the four separate concerns this EWO touches, per the corrected architecture:

| Concern | Owner | Authority |
|---|---|---|
| Actor lifecycle state, including `Executing` | Lifecycle Guardian | ARCH-002 §6 ("enforces legal lifecycle-state transitions"), §15 (legal transition graph) |
| Execution Context existence and progress | Execution Coordinator | ARCH-002 §6 ("constructs Execution Context; performs dispatch mechanics; enforces one-owner execution"), §10 |
| Connecting the two components | Runtime, exclusively | ADR-0016 Rule 1 (Runtime is the sole entity accountable for establishing and coordinating Trusted Core interaction) and Rule 2 (no Trusted Core component independently establishes or owns a direct peer interaction path) |
| External suspend/restore reachability against an actively-executing instance | Deferred, not this milestone | ARCH-002 §12 ("no Runtime-level call other than the one currently performing a given execution can observe that instance as genuinely `Executing` today"); ARCH-003 §5, §12, §18 (necessary-but-not-sufficient framing, corrected v0.3.0) |

STD-001 §46 governs this document's own form and authority: an EWO "MAY authorize implementation... MUST NOT redefine Architecture... If implementation... reveals an apparent architectural contradiction, the EWO MUST require engineering to stop and return the issue for architectural review rather than resolve it unilaterally." This EWO's own Stop Conditions (below) apply that rule concretely.

---

## Objective

Runtime must cause Lifecycle Guardian to reflect `Executing` for exactly the period during which Execution Coordinator owns a live Execution Context for the same actor instance, and must cause the actor to leave `Executing` immediately when that context completes or fails.

This objective does not include, and this EWO does not authorize, any work toward making `Executing` externally observable from a call other than the one currently performing the execution, or toward asynchronous, concurrent, or otherwise interruptible execution.

---

## Bounded Design Decision (resolved within this EWO)

The scope-determination review that authorized this EWO identified one implementation-equivalent decision requiring resolution before implementation: whether `Runtime`'s composition root should hold Lifecycle Guardian behind a trait object (`Box<dyn LifecycleGuardian>`, mirroring `TrustedCore.audit_emitter`) to enable a recording test double proving Runtime's exact call order, or whether the existing concrete `LifecycleGuardianHandle` field should be preserved and the required invariants proved by component-level tests plus Runtime-level success/failure end-state tests plus source-level sequencing verification.

**Decision: preserve the existing concrete `LifecycleGuardianHandle` field. Do not introduce `Box<dyn LifecycleGuardian>`.**

Rationale, addressing each consideration this EWO was required to weigh:

1. **Existing Runtime composition style.** `TrustedCore` holds six of its seven components as concrete, non-trait-object handles (`HostAdapterHandle`, `CapabilityAuthorityHandle`, `ActorHostHandle`, `MessageGatewayHandle`, `ExecutionCoordinatorHandle`, `LifecycleGuardianHandle`); only `audit_emitter: Box<dyn AuditEmitter>` is a trait object. This is not an accident of convenience — EWO-001's own Authorized Implementation Clarification states each of the seven components was given "either an intentionally opaque public handle (six crates, where the underlying trait's behavior remains out of scope) or the crate's public trait abstraction directly via `Box<dyn Trait>` (Audit Emitter only, since emission is explicitly in scope)." Lifecycle Guardian is one of the six.
2. **Whether `Box<dyn LifecycleGuardian>` would change production design merely to improve testing.** Yes, it would: no production requirement — no replaceable-service boundary, no runtime-selectable implementation, no ARCH-002 §6 provision — calls for Lifecycle Guardian to vary by implementation the way Audit Emitter's emission channel legitimately does. Introducing indirection here would exist solely to make a test double installable, which is exactly the case the task's own preferred decision principle names.
3. **Existing precedent from Audit Emitter.** Audit Emitter is a trait object because ARCH-002 explicitly treats emission-channel pluggability as in-scope replaceable behavior, and `RecordingAuditEmitter`/`FailingAuditEmitter` already exist as legitimate, precedented test doubles for it. Lifecycle Guardian carries no analogous architectural rationale for pluggability — ARCH-002 §6 lists it as Trusted Core, not a replaceable service (§6's replaceable-services table). The precedent argues for the *opposite* choice here: trait-object status tracks a genuine architectural property, not merely "this crate has a trait."
4. **Existing precedent from EWO-004.** EWO-004 faced a structurally identical unprovability problem: `HostExecutionHandle` is a zero-field type, so no runtime assertion can confirm "the caller-supplied handle, not a fabricated one, was embedded." EWO-004 resolved this by explicit disclosure — proving what the type's own definition permits (the call accepts the parameter and succeeds) and relying on direct construction-path inspection (`ExecutionCoordinatorImpl`'s own implementation: `host_execution_handle: handle`) as the remaining proof, stated plainly in both the crate's tests and its doc comments, rather than introducing new indirection to manufacture a stronger runtime-checkable guarantee. This EWO applies the identical, already-accepted resolution pattern.
5. **Whether the transient `Executing` period can be proved without artificial concurrency.** No — and this is not a gap this EWO can close. ARCH-002 §12 states plainly that "no Runtime-level call other than the one currently performing a given execution can observe that instance as genuinely `Executing` today," because the Runtime is fully synchronous. Any test — recording double or not — that ran *during* `execute_message` to observe the transient state would itself have to run on the same call stack, inside Execution Coordinator's or Lifecycle Guardian's own code, which a recording *Lifecycle Guardian* double achieves (it observes call order, not mid-flight external state) but which does not require, and is not strengthened by, `Box<dyn LifecycleGuardian>` specifically — a recording double could equally be installed via a `#[cfg(test)]` seam within `LifecycleGuardianHandle` itself if ever needed, without changing its production type.
6. **Whether a narrow recording test double is proportionate to this milestone.** No. EWO-004 itself, facing an analogous problem one level down (proving Host Adapter's allocate/release balance), did not introduce a recording double or new indirection — it proved the balance invariant indirectly, by attempting one further release after a successful call and observing it fail (`execute_message_leaves_host_adapter_balanced_after_success`). This EWO follows that same established, proportionate pattern: component-level tests prove each new Lifecycle Guardian method's legality behavior in isolation; Runtime-level tests prove observable end states (the actor is not falsely `Executing` before or after `execute_message` returns, in every success and failure case); and the exact call ordering Runtime enforces is verified by source-level inspection, recorded in ER-005, exactly as EWO-004's own handle-fidelity claim was.

This resolution is final for this EWO's scope. It does not preclude a future, separately-authorized decision to generalize `TrustedCore`'s construction pattern, should a genuine production need for Lifecycle Guardian pluggability arise later — no such need exists today, and none is invented here.

---

## Scope

Implement only:

- The minimum Lifecycle Guardian trait evolution — new methods only, described under "Required Interface Evolution" below — and the corresponding `LifecycleGuardianImpl` implementation.
- The minimum Runtime composition-root change (`runtime/src/lib.rs`, `Runtime::execute_message`) sequencing the new Lifecycle Guardian calls around the existing, unchanged `construct_context` → `dispatch` → `complete` sequence, per "Runtime Sequencing" below.
- Truthful successful exit (`Executing → Idle` on genuine completion).
- Truthful failure exit (`Executing → Failed`), to the extent required by "Failure Semantics" below, even where currently unreachable through ordinary public use.
- Unit tests within `synapse-lifecycle-guardian` proving the new methods' legality behavior.
- Integration tests within `runtime/tests/` (or the existing in-crate `#[cfg(test)]` module, matching current convention) proving Runtime-level end-state truthfulness on every success and failure path this EWO defines.
- Documentation updates to `core/lifecycle-guardian/README.md` and `runtime/README.md`, strictly limited to describing the newly implemented interface and behavior.

---

## Explicit Exclusions

Do NOT implement, and do not let any of the following creep into this milestone's scope:

- External suspend/restore reachability against an actively-executing instance (ARCH-002 §12; ARCH-003 §5, §12, §18 — remains deferred, unconditionally, by this EWO).
- Asynchronous execution.
- Concurrent execution.
- Interruptible execution.
- Real actor-defined handler invocation (`dispatch` continues to enforce mechanism only; no handler exists anywhere in the workspace).
- Capability revalidation at invocation (`ExecutionContext.active_capabilities` remains empty).
- Bounded mailbox capacity or overflow handling (ARCH-002 §13, §22 — a separate, independent, already-identified deferred item; see the EWO-005 scope-determination review, Candidate B).
- SDK work.
- Actor-contract design (ARCH-002 §7's message-handling contract remains unwritten; not this milestone's concern).
- Any change to `HostExecutionHandle`'s definition or Execution Coordinator's Host Adapter boundary (EWO-004's own scope, unchanged here).
- Any new audit event type.
- Any new Trusted Core component.
- Any direct dependency, in either direction, between `synapse-execution-coordinator` and `synapse-lifecycle-guardian` (ADR-0016 — Runtime remains the sole connector).
- A generic lifecycle-state-mutation API of any kind on the `LifecycleGuardian` trait.
- Unrelated refactoring of `synapse-lifecycle-guardian`, `synapse-execution-coordinator`, or `runtime/src/lib.rs` beyond what this EWO's own scope requires.
- Modification of any shared lifecycle enum (`ActorState` in `synapse-common`) unless implementation proves this is unavoidable — see Stop Conditions.
- Resolution of the pre-existing `Suspended → Idle/Ready` wording divergence noted elsewhere in the corpus. Out of scope; unrelated to this milestone.

---

## Trusted Core

Execution Coordinator and Lifecycle Guardian remain two of exactly seven Trusted Core components (ARCH-002 §6). No new Trusted Core component is introduced. No responsibility is transferred: Execution Coordinator continues to own only Execution Context construction, dispatch mechanics, and one-owner enforcement; Lifecycle Guardian continues to own only lifecycle-transition legality and enforcement for the state it tracks. Runtime's new role — calling Lifecycle Guardian's new methods at the correct points in `execute_message` — is composition-root sequencing, not a transfer of either component's own behavioural authority (ARCH-003 §6, §17 invariant 7).

---

## Architecture Constraints

Do not:

- introduce new Runtime concepts;
- invent lifecycle states, audit events, or capability checks;
- modify constitutional concepts;
- reinterpret ARCH-001, ARCH-002, or ARCH-003;
- redesign the `LifecycleGuardian` trait beyond the minimum interface evolution this EWO authorizes;
- redesign the `ExecutionCoordinator` trait or `ExecutionCoordinatorImpl` in any way, unless "Failure Semantics" below proves a minimal, disclosed exception unavoidable, in which case: STOP and report per Stop Conditions rather than implementing it unilaterally;
- introduce new crates;
- introduce external dependencies;
- use unsafe Rust;
- give `synapse-lifecycle-guardian` a dependency on `synapse-execution-coordinator`, or the reverse, in either direction;
- introduce `Box<dyn LifecycleGuardian>` or any other change to `TrustedCore`'s existing composition pattern (resolved above, under "Bounded Design Decision").

If implementation requires any of the above: STOP. Produce an Engineering Report explaining the issue. Do not invent a solution.

---

## Repository Constraints

Do not modify: governance documents; architecture documents; standards; work orders other than this one (including, but not limited to, `EWO-003-Message-Gateway.md`, which remains an untracked, protected file this EWO does not touch); repository structure beyond `core/lifecycle-guardian/`, the minimum `runtime/` change this EWO authorizes, and their manifests/READMEs. Do not modify `synapse-execution-coordinator`, `synapse-host-adapter`, `synapse-actor-host`, `synapse-message-gateway`, `synapse-capability-authority`, `synapse-audit-emitter`, or `common/` unless "Failure Semantics" proves a change unavoidable — in which case, STOP per Stop Conditions rather than proceeding.

---

## Required Interface Evolution

ARCH-002 §6 assigns Lifecycle Guardian sole responsibility for enforcing legal lifecycle-state transitions; ARCH-002 §15 fixes the legal transition graph. Consequently, the only component authorized to mutate an actor instance's tracked `Executing` state is Lifecycle Guardian — this is an entailment of already-published architecture, not a decision this EWO makes (mirrors EWO-004's own "Construction Surface" framing).

This EWO authorizes exactly the minimum public interface evolution objectively required: two fixed-purpose, non-generic Lifecycle Guardian operations, plus a third if "Failure Semantics" below proves it necessary for truthful failure handling.

**1. Enter actor execution.**
- Valid only from an architecture-authorized pre-execution state — `Idle` or `Ready` (ARCH-002 §15's only two edges into `Executing`).
- Transitions the actor instance to `Executing`.
- Uses Lifecycle Guardian's own existing transition-legality machinery (the same `is_legal_transition` graph `validate_transition`, `suspend`, and `restore` already use) — no new, parallel legality logic.
- Does not accept an arbitrary destination state; the destination is fixed to `Executing`, exactly as `suspend`'s destination is fixed to `Suspended` and `restore`'s is fixed to `Idle`.

**2. Complete actor execution successfully.**
- Valid only from `Executing` (ARCH-002 §15's `Executing → Idle` edge).
- Transitions the actor instance to `Idle`.
- Does not expose a generic state-mutation API; the destination is fixed to `Idle`.

**3. Fail actor execution (required — see rationale below).**
- Valid only from `Executing` (ARCH-002 §15's `Executing → Failed` edge).
- Transitions the actor instance to `Failed`.
- Required because "Failure Semantics" below identifies a genuine, if currently unreachable through ordinary public use, path (dispatch or completion rejection after a live Execution Context has already been entered) where the alternative — leaving the actor falsely `Executing` with no live context — would violate this EWO's own Objective and ARCH-002 §10's truthfulness rule. This mirrors `construct_context`'s own precedent: a defensively correct, fully implemented path that is currently exercised only via directly-seeded internal test state, disclosed as such rather than silently omitted.

**Naming.** This EWO recommends, but does not mandate, the names `begin_execution`, `complete_execution`, and `fail_execution` for these three operations, for consistency with the existing `suspend`/`restore` naming style. Implementation may select different names if doing so produces clearer code; the required *behavior* above is what this EWO fixes, not the identifiers. (Note: `Runtime` already has an unrelated private method also named `fail_execution`, operating on a different type with a different signature and purpose — this is not a naming conflict in Rust, since the two live on different types, but implementation should choose names that do not read as confusingly identical in review.)

**Do not create a generic "set state" method.** No method accepting an arbitrary `ActorState` destination parameter is authorized. Each of the three operations above has exactly one fixed destination, matching every existing mutating method on this trait.

No change to `synapse-common` is required: `ActorState::Executing`, `::Idle`, and `::Failed` already exist; `RuntimeError::IllegalTransition` already covers every rejection case these three methods can produce.

---

## Runtime Sequencing

`Runtime::execute_message`'s required sequence, in order:

1. Actor Host: verify the actor instance is live (`live_instance`) — existing, unchanged.
2. Lifecycle Guardian: validate that entry into `Executing` is legal (`validate_transition(instance, Executing)`) — **existing, read-only method**, called here for the first time in this flow.
3. Host Adapter: allocate a Host Execution Handle (`allocate_execution_handle`) — existing, unchanged (EWO-004).
4. Execution Coordinator: construct the Execution Context (`construct_context`) — existing, unchanged. The Execution Context becomes live at this point (ARCH-002 §10).
5. Lifecycle Guardian: enter `Executing` (new) — the actor instance becomes truthfully `Executing` at this point.
6. Execution Coordinator: dispatch (`dispatch`) — existing, unchanged.
7. Execution Coordinator: complete (`complete`) — existing, unchanged. The Execution Context ceases to be live at this point.
8. Lifecycle Guardian: exit `Executing` to `Idle` (new).
9. Host Adapter: release the handle (`release_execution_handle`) — existing, unchanged (EWO-004).
10. Audit Emitter: emit `execution.completed` — existing, unchanged.
11. Return success.

**Why Lifecycle Guardian's entry mutation (step 5) occurs after successful context construction (step 4), not before:**

- Before construction, the actor instance does not yet own a live Execution Context — marking it `Executing` at that point would be false, precisely the failure mode ARCH-002 §10/§12 forbid ("MUST NOT be held, extended, or otherwise made to outlive the genuine fact it represents").
- If construction fails (step 4 rejects), the actor must never enter `Executing` at all. Sequencing entry after construction makes this automatic: step 5 is simply never reached.
- Sequencing entry after construction, rather than before, means a construction failure requires **no Lifecycle Guardian rollback of any kind** — no mutation was ever attempted, so there is nothing to undo. This is the specific design property that makes "Context-construction failure," below, simple rather than requiring compensating logic.

**Why read-only prevalidation (step 2) occurs before allocation and construction, rather than being folded into step 5's own mutation:**

- ARCH-003 §6's own Integration Principle states: "Validation occurs before side effects... a rejection at any step prevents every subsequent step, mutating or not." Step 2 uses Lifecycle Guardian's own existing, already-published `validate_transition` — a read-only check — to confirm the eventual mutation (step 5) will be legal, *before* any handle is allocated or any Execution Coordinator state is mutated.
- This closes a latent risk that would otherwise exist if entry legality were checked for the first time at step 5: Execution Coordinator's own reentrancy check (inside `construct_context`) and Lifecycle Guardian's own transition-legality check are independent, differently-scoped checks over independently-mutated state. Validating legality *before* either component's state is touched means a rejection at step 2 leaves both components' state completely untouched — no orphaned Execution Coordinator entry, no attempted-and-failed Lifecycle Guardian mutation. See "State-Consistency Invariant" below for why this specific ordering is what makes the invariant hold given the current, non-concurrent implementation.

---

## Failure Semantics

### Live-instance verification failure (step 1 rejects)

No allocation. No context. No lifecycle mutation. Return the existing error path (`fail_execution`, unchanged).

### Lifecycle prevalidation failure (step 2 rejects)

No allocation. No context. No lifecycle mutation. Return the existing error path. (Currently unreachable in practice through genuine public-API use, since every instance Lifecycle Guardian has not separately tracked defaults to `Idle`, and `Idle → Executing` is always legal — this gate exists for correctness and for when a future milestone makes a non-`Idle`, non-`Ready` state reachable at this point, not because it is expected to reject today.)

### Handle-allocation failure (step 3 rejects)

No context. No lifecycle mutation. Return the existing error path — unchanged from EWO-004 (currently unreachable, since allocation is unconditional).

### Context-construction failure (step 4 rejects)

Release any allocated handle through Runtime's existing `release_and_fail_execution` cleanup (EWO-004, unchanged). Do not enter `Executing` — step 5 is simply never reached, so no Lifecycle Guardian cleanup is required (see "Runtime Sequencing," above). Emit the existing `execution.failed` audit behaviour, unchanged. Do not invent a cancellation mechanism on Execution Coordinator; none is required, because Lifecycle Guardian's state was never mutated in the first place.

### Lifecycle entry failure after successful prevalidation (step 5 fails despite step 2 having already validated the identical transition)

This is an **invariant violation** under the current synchronous, non-interleaved implementation: `Runtime` is single-threaded, and no call can interleave between step 2 and step 5 within one `execute_message` invocation (the same reasoning EWO-004 already established for why Host Adapter release "cannot fail in ordinary operation"). This path must not silently be ignored. **If implementation determines this path can genuinely occur, implementation MUST stop and report it as a Stop Condition (below) rather than inventing a workaround, retry, or suppression.**

### Dispatch failure (step 6 rejects)

Verified directly against `core/execution-coordinator/src/internal.rs`: `dispatch` rejecting after a successful `construct_context` for the same call is not reachable through any genuine, sequential `execute_message` invocation in the current single-threaded implementation (`dispatch` only rejects when `active.get(instance)` is not `Some(Constructed)`, which cannot occur immediately after `construct_context` just inserted exactly that value, with nothing able to interleave). It is currently reachable only via directly pre-seeding `ExecutionCoordinatorImpl`'s internal state through same-crate test access, exactly as the existing `execute_message_leaves_host_adapter_balanced_when_execution_context_construction_fails` test already does for construction.

Because `ExecutionCoordinatorImpl` provides no method to remove a `Constructed`-or-`Dispatched` entry other than a successful `complete` call, **Execution Coordinator today has no mechanism to cleanly cease context liveness after a dispatch failure.** This EWO does not authorize adding one (that would touch Execution Coordinator, out of this EWO's scope per "Repository Constraints"). This EWO therefore requires:

- the actor instance's Lifecycle Guardian state to leave `Executing` and enter `Failed` (the new `fail_execution`-equivalent method, "Required Interface Evolution" item 3) — Lifecycle Guardian's own state is kept truthful regardless of Execution Coordinator's internal bookkeeping;
- existing handle cleanup (`release_and_fail_execution`) and existing `execution.failed` audit behaviour, unchanged;
- no actor may remain falsely `Executing` after this path, even though this specific path is not reachable through ordinary public use today.

This EWO does **not** require Execution Coordinator's own internal entry to be cleaned up, since doing so is out of scope and the path itself is currently unreachable through genuine use — this limitation must be disclosed explicitly in ER-005, exactly as ARCH-003 already discloses comparable gaps rather than silently assuming them away. If future work later makes this path genuinely reachable (e.g., once real actor logic can fail), closing Execution Coordinator's own cleanup gap is separate, later work, not this EWO's.

### Completion failure (step 7 rejects)

Same reasoning and requirement as dispatch failure, immediately above: currently reachable only via test-seeded internal state; requires the actor to leave `Executing` and enter `Failed` via the same new method; no fabricated success transition to `Idle` is permitted where completion did not genuinely succeed; existing handle cleanup and `execution.failed` audit behaviour apply unchanged.

### Lifecycle successful-exit failure (step 8 fails despite the Execution Context having just genuinely completed)

Treat this as a state-divergence invariant violation, on the same basis as "Lifecycle entry failure" above: nothing in the current, single-threaded implementation can cause Lifecycle Guardian's tracked state to have changed between step 5's successful entry and step 8's exit attempt, other than the sequence this EWO itself defines. **If implementation determines the existing transition graph cannot make this impossible, implementation MUST stop and report it, per Stop Conditions.**

### Host release failure (step 9)

Lifecycle exit (step 8) occurs **before** handle release (step 9), preserving EWO-004's own established sequencing and error-priority rule unchanged: Lifecycle Guardian's state must be made truthful as soon as the genuine underlying fact (the Execution Context's completion) is known, not deferred behind a resource-release step that is architecturally unrelated to lifecycle truthfulness. If release then fails, `execute_message` reports that failure per EWO-004's existing "a later mandatory step's failure overrides an otherwise-successful outcome" rule — this EWO does not redesign Host Adapter or alter that rule in any way. The actor's Lifecycle Guardian state remains correctly `Idle` (or `Failed`, on the failure paths above) regardless of whether the subsequent release succeeds, since the underlying fact it represents (the Execution Context's genuine end) has already, truthfully, occurred.

---

## State-Consistency Invariant

**Named invariant, required by this EWO:** for every genuine sequential Runtime execution path supported by the current public API, Lifecycle Guardian reports `Executing` if and only if Execution Coordinator owns the live Execution Context for that actor instance, except only during the immediately adjacent Runtime statements that establish or clear the corresponding state (Exception A, below). This is a statement about the sequence of ordinary Rust statements within one synchronous function call, not a claim of atomic or transactional execution: no transaction, rollback log, or locking mechanism exists or is introduced by this EWO. The invariant holds because `Runtime::execute_message` takes `&mut self` and nothing in the current workspace spawns a thread, task, or any other concurrent execution path (unchanged, confirmed by EWO-004's own equivalent finding for Host Adapter) — no other call can observe or mutate either component's state between any two of this EWO's sequenced statements.

A second, separately disclosed and materially different exception (Exception B, below) applies only to the dispatch/completion-rejection paths "Failure Semantics" already discloses as currently unreachable through genuine public use. The two exceptions must not be conflated with one another, and neither excuses divergence beyond what it specifically describes.

**Exception A — normal statement-ordering window.** The unavoidable, immediately closed sequencing gap between (a) successful context construction (step 4) and Lifecycle Guardian entry (step 5), and (b) successful context completion (step 7) and Lifecycle Guardian exit (step 8). This exception is permitted only between these adjacent ordered Runtime statements and must not remain after `execute_message` returns.

**Exception B — disclosed, currently unreachable rejection-path limitation.** On the dispatch- and completion-rejection paths described under "Failure Semantics" — reachable today only through internal test-state manipulation, never through genuine sequential public Runtime execution — Execution Coordinator's own `active` map retains its stale `Constructed`/`Dispatched` entry after Lifecycle Guardian truthfully transitions the actor to `Failed`, because no current public Execution Coordinator method exists to mark that entry failed or remove it outside a successful `complete` call. This is a pre-existing Execution Coordinator limitation, not something EWO-005 introduces or is authorized to correct (see "Repository Constraints," "Failure Semantics"). The stale entry does not represent a truthfully live execution — Lifecycle Guardian's `Failed` state is what is truthful on this path — and this exception does not extend to, or excuse, any normal or genuinely reachable Runtime behaviour. If either rejection path is ever found to be reachable through genuine public Runtime execution, implementation MUST stop under Stop Condition 5 rather than relying on this exception to justify shipping the divergence.

Consequences this invariant requires and this EWO's design (above) already satisfies:

- **No actor is marked `Executing` before context construction succeeds.** Step 5 (entry) is sequenced immediately after, never before, step 4 (`construct_context`) succeeds.
- **No normal successful Runtime call returns with the actor still `Executing`.** Step 8 (exit) is sequenced immediately after step 7 (`complete`) succeeds, before Host Adapter release or audit emission.
- **No normal, reachable failed Runtime call returns with the actor falsely `Executing`.** Every failure path defined under "Failure Semantics" either never reaches step 5 (live-instance, prevalidation, allocation, construction failures) or explicitly transitions the actor out of `Executing` into `Failed` before returning (dispatch, completion failures) — subject only to Exception B's disclosed, currently-unreachable bookkeeping limitation on the Execution-Coordinator side, which never applies to Lifecycle Guardian's own reported state.
- **Successful completion does not transition the actor to `Idle` while the context remains live.** Step 8 is reachable only after step 7 has already, genuinely, removed the context from Execution Coordinator's tracked state.
- **Exception A is immediate and temporary.** It exists only between two adjacent statements within one synchronous call and never persists past `execute_message`'s return.
- **Exception B is explicitly separate, pre-existing, and not normal conformance.** It applies only to the two disclosed, currently-unreachable rejection paths, is confined to Execution Coordinator's own internal bookkeeping, and is never treated as license for Lifecycle Guardian itself to report anything other than the truthful `Failed` state on those paths.
- **Genuine reachability of Exception B is a stop condition, not an accepted permanent design point.** See Stop Condition 5.

---

## Audit Semantics

- No new audit event type is authorized by this EWO.
- The existing `execution.completed` and `execution.failed` events (ARCH-002 §18) remain exactly as EWO-004 left them.
- This EWO does not add an `execution.started`-equivalent event. ARCH-003 §18 already states explicitly that no such event currently exists and none is proposed here.
- Lifecycle truthfulness (the new entry/exit mutations) must be established *before* the final audit emission in every path — Runtime Sequencing already places steps 5 and 8 ahead of step 10, and every failure path's lifecycle mutation (where one applies) ahead of its own `execution.failed` emission.
- Audit emission failure does not roll back truthful lifecycle state. No existing ADR authority (ADR-0015 included) requires or authorizes rollback of already-committed component-level state on audit failure — ADR-0015 and ARCH-003 §15 already establish this generally ("no rollback of already-committed component-level state... this operation reports `Err`... regardless"); this EWO applies the same, already-established rule to the two new Lifecycle Guardian mutations, without exception.

---

## Definition of Done

The task is complete only if all of the following are true:

- The actor instance enters `Executing` (Lifecycle Guardian's own tracked state) only after Execution Context construction succeeds.
- The actor instance's Lifecycle-Guardian-tracked state is `Executing` only while a live Execution Context genuinely exists for it.
- The actor instance leaves `Executing` on successful completion, to `Idle`.
- The actor instance leaves `Executing` on every failure path this EWO defines, to `Failed` where one is reachable, or is never entered at all where construction never succeeded.
- No successful or failed `execute_message` call returns with the actor instance falsely `Executing`.
- No external suspend/restore reachability is claimed, implemented, or implied anywhere in the implementation or its documentation.
- Runtime remains the sole Trusted Core composition root (ADR-0016 unaffected).
- No new direct dependency exists between `synapse-lifecycle-guardian` and `synapse-execution-coordinator`, in either direction.
- No new audit event type exists anywhere in the implementation.
- `synapse-lifecycle-guardian`'s dependency set remains exactly `synapse-common`. No new dependency of any kind, in any crate.
- EWO-001's, EWO-002's, EWO-003's (where applicable), and EWO-004's existing tests continue to pass unmodified in behaviour.
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

All must pass. Additionally verify: workspace member count unchanged; zero `unsafe`; no dependency cycle; `cargo tree -p synapse-lifecycle-guardian` shows exactly `synapse-common`; no crate other than the Runtime composition root depends on `synapse-lifecycle-guardian`'s public interface for execution-sequencing purposes; `Runtime::execute_message`'s own public signature is byte-for-byte unchanged.

---

## Data and State Constraints

No new state is required beyond what already exists:

- `LifecycleGuardianImpl` requires no new field — its existing `states: HashMap<ActorInstanceId, ActorState>` already tracks exactly what the new methods mutate.
- `ExecutionCoordinatorImpl` requires no change of any kind (see "Failure Semantics" — dispatch/completion-failure cleanup on the Execution Coordinator side is explicitly out of scope, disclosed as a limitation, not implemented here).
- `Runtime` requires no new field — no handle, token, or correlation value needs to be held across the new calls; each new Lifecycle Guardian call takes only the `ActorInstanceId` already in scope within `execute_message`.
- Do not add: a pending-transition registry, a generation counter, or any bookkeeping beyond the two (or three) new fixed-purpose method calls this EWO authorizes.

---

## Definition of Failure / Stop Conditions

Stop immediately, and produce an Engineering Report rather than resolving the issue unilaterally, if any of the following occurs:

1. Truthful tracking requires a direct dependency, in either direction, between `synapse-execution-coordinator` and `synapse-lifecycle-guardian`.
2. A new `ActorState` or Execution-Context-state variant is required.
3. `synapse-common`'s shared lifecycle enums must change in any way.
4. A generic state-mutation API (accepting an arbitrary destination state) proves necessary on the `LifecycleGuardian` trait.
5. Execution Coordinator cannot be left in a state consistent with this EWO's requirements after a dispatch or completion failure without itself being modified beyond what "Failure Semantics" above already discloses and accepts as an out-of-scope limitation.
6. Runtime cannot prevent the state-divergence class of defect described under "State-Consistency Invariant" without introducing transactional machinery (locking, rollback log, or equivalent).
7. Lifecycle entry (step 5) or exit (step 8) is found to be capable of failing after the identical transition was already validated or just performed, within the current synchronous, non-interleaved implementation.
8. Proving any required behaviour is found to require artificial concurrency (threads, async tasks, sleeps, polling) purely for test purposes.
9. External suspend/restore reachability against an actively-executing instance becomes necessary to satisfy any acceptance criterion.
10. Scope is found to require capability-authority work, mailbox work, actor-handler/contract design, SDK work, or Host Adapter redesign.
11. Any ADR guarantee or ARCH-002/ARCH-003 architectural boundary is found to require change to complete this milestone.

Do not resolve these issues yourself. Report them.

---

## Implementation Sequence

1. Baseline verification: re-confirm the current `core/lifecycle-guardian/`, `core/execution-coordinator/`, and `runtime/` state matches this EWO's "Problem Statement" section exactly.
2. Interface evolution: implement the two (or three, per "Required Interface Evolution" item 3) new Lifecycle Guardian methods, using the existing `is_legal_transition` graph, matching `suspend`/`restore`'s established fixed-destination style.
3. Component-local tests: unit tests within `synapse-lifecycle-guardian` proving each new method's legality behavior in isolation, per "Test Requirements" below.
4. Runtime integration: apply the minimum `runtime/src/lib.rs` change to `execute_message` implementing "Runtime Sequencing" and "Failure Semantics" above.
5. Integration tests: within `runtime`'s existing test module (or `runtime/tests/`, matching current convention), prove every end-state this EWO requires, per "Test Requirements" below.
6. Regression check: confirm all pre-existing tests (EWO-001 through EWO-004) still pass unmodified in outcome.
7. Documentation updates: `core/lifecycle-guardian/README.md` and `runtime/README.md`, reflecting the real behaviour now present, including explicit disclosure of the dispatch/completion-failure Execution-Coordinator-side limitation named under "Failure Semantics."
8. Complete quality validation, per Mandatory Validation above.
9. ER-005 preparation after implementation and validation are complete — not before, and not as part of this EWO.

Do not include work from later milestones.

---

## Acceptance Criteria

Every criterion is objective and testable:

- The actor instance's Lifecycle-Guardian-tracked state enters `Executing` only after Execution Context construction succeeds (never before).
- The actor instance's Lifecycle-Guardian-tracked state is `Executing` only while a live Execution Context exists for that instance.
- The actor instance leaves `Executing` on successful completion.
- The actor instance leaves `Executing` on every implemented failure path following successful entry.
- No successful or failed Runtime call returns with the actor instance falsely `Executing`.
- No external suspend/restore reachability is claimed anywhere in code, tests, or documentation produced under this EWO.
- Runtime remains the sole Trusted Core composition root — no new peer-to-peer path between any two Trusted Core components.
- No new Trusted Core component is introduced.
- No new audit event type is added.
- All pre-existing tests (EWO-001 through EWO-004) pass unmodified in outcome.
- All new tests pass.
- `cargo fmt --all -- --check` and `cargo clippy --workspace --all-targets --all-features -- -D warnings` both pass with zero warnings.
- `git status` after implementation shows changes confined to `core/lifecycle-guardian/`, the minimum touched parts of `runtime/`, and their manifests/READMEs — no unrelated file touched.
- Architecture and ADR documents remain byte-for-byte untouched.

---

## Required Tests

### Lifecycle Guardian unit tests (`synapse-lifecycle-guardian`)

At minimum:

- Legal entry from `Idle`.
- Legal entry from `Ready`.
- Illegal entry from every other state (`Defined`, `Initializing`, `Executing`, `Suspended`, `Failed`, `Stopping`, `Terminated`).
- Successful `Executing → Idle` exit.
- Successful `Executing → Failed` exit (per "Required Interface Evolution" item 3).
- Illegal successful-exit attempt from any non-`Executing` state.
- Illegal failure-exit attempt from any non-`Executing` state.
- All pre-existing `suspend`/`restore`/`validate_transition` tests continue to pass unmodified.

### Runtime tests (`runtime`)

At minimum:

- A successful `execute_message` call returns with the actor instance no longer `Executing` (Idle).
- A successful `execute_message` call permits a subsequent execution for the same instance, exactly as the existing `execute_message_fails_for_a_second_concurrent_execution_of_the_same_instance` test already establishes for Execution Coordinator's own state — extended to confirm Lifecycle Guardian's tracked state is likewise available for re-entry.
- Execution Context construction failure (via the existing pre-seeding technique already used by `execute_message_leaves_host_adapter_balanced_when_execution_context_construction_fails`) never marks the actor `Executing`.
- Dispatch failure (via pre-seeded internal Execution Coordinator state) does not leave the actor `Executing`.
- Completion failure (via pre-seeded internal Execution Coordinator state) does not leave the actor `Executing`.
- Handle release still occurs on both the success and failure paths this EWO defines, matching EWO-004's already-established balance-invariant test pattern.
- Existing `execution.completed`/`execution.failed` audit events still occur exactly where they did before this EWO.
- No test claims, exercises, or asserts external mid-flight suspension of an actively-executing instance.
- No test introduces sleeps, threads, async tasks, polling, or any other artificial execution window.

**Illegal lifecycle source state — not a Runtime-level test.** Lifecycle entry rejection from illegal source states is proved at the Lifecycle Guardian component-test boundary only (see "Lifecycle Guardian unit tests," above — "Illegal entry from every other state"), not at the Runtime level. `LifecycleGuardianImpl::set_state_for_testing` is `#[cfg(test)] pub(crate)` inside Lifecycle Guardian's own private `internal` module, and `LifecycleGuardianHandle`'s wrapped field is private — neither is nameable or reachable from `runtime`'s own test code, exactly as `runtime/src/lib.rs`'s own existing test-module comment already discloses for the identical seam. This is not merely a missing test: the current Runtime public API also provides no other way to place a live execution target into a non-`Idle`/non-`Ready` source state before this milestone's own new methods run, since any untracked instance defaults to `Idle` and every reachable public sequence leaves the instance `Idle` or `Ready` by the time `execute_message` is called again. EWO-005 does not add a new test-only API, expose Lifecycle Guardian's private internals, or convert Runtime's Lifecycle Guardian ownership (see "Bounded Design Decision") solely to manufacture this Runtime-level scenario. This mirrors, and does not contradict, "Failure Semantics"'s own disclosure that lifecycle prevalidation failure is currently unreachable in practice.

### Transient-state proof

Resolved under "Bounded Design Decision" above: the transient period during which the actor instance is genuinely `Executing` (Runtime Sequencing steps 5–7) is proved by the combination of (a) component-level tests proving each new Lifecycle Guardian method's legality behavior in isolation, (b) Runtime-level tests proving observable end states before and after `execute_message` returns in every success and failure case defined above, and (c) source-level, code-order verification recorded in ER-005 — not by a runtime-observable mid-flight assertion, which the current, correctly-scoped synchronous architecture does not and should not make possible (ARCH-002 §12).

**This EWO prohibits changing Runtime's production composition (including but not limited to introducing `Box<dyn LifecycleGuardian>`) solely to make this synchronous transient state externally observable.**

---

## Engineering Decision Log

Record:

- implementation decisions;
- repository decisions;
- deferred decisions;
- architectural decisions (expected: None);
- constitutional decisions (expected: None);
- future work enabled (expected: a documented, working example of Runtime-mediated Execution-Coordinator/Lifecycle-Guardian truthful-state sequencing that a future, separately-authorized milestone addressing genuine suspend/restore reachability — which requires an asynchronous or otherwise interruptible execution mechanism this EWO does not provide — could build on);
- future work deferred (expected: external suspend/restore reachability against an actively-executing instance; capability revalidation at invocation; bounded mailboxes and overflow handling; real actor-handler execution; Execution Coordinator's own dispatch/completion-failure cleanup gap, disclosed but not closed by this EWO).

---

## Completion Report

ER-005 must provide, after implementation:

1. Files modified.
2. Files created (expected: none beyond test files within existing crates).
3. Lifecycle Guardian interface evolution implemented: which method names, signatures, and legality checks were selected, and confirmation they match "Required Interface Evolution" above.
4. Runtime sequencing implemented, confirmed against "Runtime Sequencing" above, statement by statement.
5. Invariants enforced: the State-Consistency Invariant, confirmed by source-level inspection (not merely asserted).
6. Tests added, mapped against "Required Tests" above.
7. Validation results (Mandatory Validation, in full).
8. Dependency changes (expected: none).
9. Trusted Core changes (expected: none — same seven components, same boundaries).
10. Architecture changes (expected: none).
11. Explicit disclosure of the dispatch/completion-failure Execution-Coordinator-side cleanup limitation named under "Failure Semantics," confirming it remains a documented, disclosed gap rather than a silently-assumed one.
12. Engineering Decision Log.
13. Any Stop Condition encountered, and its resolution status.
14. An explicit statement of which ARCH-003 §18 items remain deferred after this milestone (expected: "Genuine suspend/restore reachability against an actively-executing instance" — the second, still-deferred split item), and confirmation that ARCH-003 itself should be updated (per its own §20 conformance requirement) to reflect this one item's new implemented-and-integrated status.

Stop after this milestone. Do not begin the next Runtime Integration engineering milestone.

---

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-13 | Denver Jacobs | Initial EWO, registered per STD-001 §46. Authorizes the first implementation milestone arising from ARCH-002 v0.2.0 and ARCH-003 v0.3.0's `Executing`-truthfulness clarification, per the approved EWO-005 scope-determination review. Derived exclusively from ARCH-001, ARCH-002 (v0.2.0), ARCH-003 (v0.3.0), ADR-0015 through ADR-0017, STD-001, and the verified current state of `core/lifecycle-guardian/`, `core/execution-coordinator/`, and `runtime/` at commit `01047157b6783d816fed80361b40206a98ba6f2f`. Resolves the one bounded design decision the scope-determination review identified (Lifecycle Guardian composition/testability) in favor of preserving the existing concrete `LifecycleGuardianHandle` field, per "Bounded Design Decision" above. |

## Disposition

Draft. Not yet reviewed. Not yet approved. Not yet implemented.
