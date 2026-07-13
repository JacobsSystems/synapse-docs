---
document_id: ARCH-003
title: Runtime Integration Architecture
project: SynapseOS
specification: SynapseOS — how the published Trusted Core components integrate into one coherent runtime, realizing ARCH-002
version: 0.3.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-13
last_updated: 2026-07-13
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
  standards:
    - STD-001 (Approved)
  architecture:
    - ARCH-000 (Draft — whole-system introduction)
    - ARCH-001 (Draft — constitutional foundation)
    - ARCH-002 (Draft — Runtime architecture this document integrates)
  rfcs: None
  adrs:
    - ADR-0011 (Draft, Act 1 effective)
    - ADR-0015 (Draft; Approved disposition recorded separately per STD-001 §31)
    - ADR-0016 (Draft; Approved disposition recorded separately per STD-001 §31)
    - ADR-0017 (Draft; Approved disposition recorded separately per STD-001 §31)
  roadmap: None
  research: None
  operational: None
  source_artifacts:
    - synapse-runtime @ 01047157b6783d816fed80361b40206a98ba6f2f (SRP-001 through SRP-007; EWO-004 Runtime Integration Bootstrap — Host Execution Handle Binding)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-003 — Runtime Integration Architecture

*Filename pattern: `ARCH-003-Runtime-Integration-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Document Control and Status

| Field | Value |
|---|---|
| Document ID | ARCH-003 |
| Title | Runtime Integration Architecture |
| Version | 0.3.0 |
| Status | **Draft** |
| Author | Denver Jacobs |
| Approval authority | Chief Architect (Class B, per GOV-010 §5), vacant; Founder (interim) |
| Created | 2026-07-13 |
| Classification | Public |

This document is Draft. It has not been reviewed or approved, and nothing in it should be read as operative or binding until it completes the same governance process ARCH-002 itself is subject to (GOV-003, GOV-010). Marking a section "already implemented" below is a statement about the current `synapse-runtime` source tree, verified directly against it — it is not a statement about this document's own approval status, and it grants this document no authority ARCH-002 has not already established.

## 2. Purpose

ARCH-003 defines how the seven Trusted Core components ARCH-002 §6 already specifies — Capability Authority, Actor Host, Message Gateway, Execution Coordinator, Lifecycle Guardian, Audit Emitter, and Host Adapter — integrate into one coherent, operating Runtime. All seven are individually implemented and published (SRP-001 through SRP-007). None of ARCH-002's component responsibility, ownership, or prohibition statements are redefined here. This document's own and only job is to record, precisely and against the actual committed implementation, which cross-component sequences already exist, which do not yet exist, and where the boundary between them currently sits — so that future integration work has a stated, honest baseline to build from rather than an assumed one.

## 3. Scope

**In scope:** Runtime construction and startup sequencing; actor-instance creation flow; message submission and routing flow; message execution flow; the Host Adapter / Execution Coordinator boundary; lifecycle-transition ownership and sequencing; capability-enforcement boundaries as currently wired; audit-event emission as currently wired; failure propagation and atomicity as currently implemented; Runtime shutdown; a concise set of integration invariants; and an explicit inventory of deferred integration work.

**Out of scope:** component responsibility or ownership (ARCH-002's domain, unchanged here); any new Trusted Core component; any new lifecycle state, audit event, or public API; capability semantics; networking, clustering, or distributed execution; persistence architecture; SDK design; AI orchestration; host-specific process or thread models. See §19 for the complete Non-Goals statement.

## 4. Relationship to Existing Authority

- **ARCH-000** established SynapseOS's whole-system introduction and shared architectural context; ARCH-003 inherits it without restatement.
- **ARCH-001** fixes the four constitutional guarantees (non-forgery, integrity, enforcement-at-invocation, revocation-state enforcement) that every interaction described below must continue to satisfy; ARCH-003 introduces no new guarantee and weakens none.
- **ARCH-002** remains the sole authority for Trusted Core component responsibility, ownership, and prohibition (§5–§6), the Constitutional Execution Cycle (§11), the lifecycle state model (§15), the audit-event minimum (§18), and the Runtime Interfaces table (§20). ARCH-003 does not amend, reinterpret, or supersede any of it. Where this document describes a sequence, that sequence is ARCH-002's own §11/§20 content, checked against what the committed implementation actually does with it — never a new architectural decision.
- **ADR-0016** (Trusted Core Interaction Rule) is the specific authority this document depends on most directly: the Runtime is the sole entity accountable for establishing and coordinating interaction among Trusted Core components, and no Trusted Core component independently establishes or owns a direct peer interaction path. Every sequence in §7–§12 below is Runtime-mediated for exactly this reason — it is the only architecturally permitted place such sequencing can live.
- **ADR-0015** (Audit Emitter Failure Semantics) governs every audit-emission failure behavior described in §14–§15 below: where an operation carries a mandatory audit obligation, failure of that emission fails the operation's own report to its caller, with no rollback of already-committed component-level state.
- **ADR-0017** (Bootstrap Capability Trust Root) governs the capability-issuance boundary referenced in §13.
- **Governance (GOV-003, GOV-010)** and **STD-001** govern this document's own review, approval, and evidentiary process; they are not architectural inputs to its content.
- **Existing implementation** (`synapse-runtime` @ `01047157b6783d816fed80361b40206a98ba6f2f`) is treated as evidence, not as authority: where implementation and ARCH-002 appear to diverge, ARCH-002 governs, and the divergence is recorded as a defect or as deferred work, never silently reconciled by describing the implementation as if it were the architecture.

Component responsibilities, ownership, and prohibitions remain exactly as ARCH-002 §6 states them. ARCH-003 adds no new responsibility to any component and removes none.

## 5. Current Implementation Baseline

Verified directly against `synapse-runtime` @ `01047157b6783d816fed80361b40206a98ba6f2f` (`runtime/src/lib.rs` and each Trusted Core crate's `src/internal.rs`), not assumed:

- All seven Trusted Core components are implemented and published: Runtime Bootstrap (SRP-001), Actor Host (SRP-002), Message Gateway (SRP-003), Capability Authority (SRP-004), Execution Coordinator (SRP-005), Lifecycle Guardian (SRP-006), Host Adapter (SRP-007).
- **Execution Coordinator ↔ Host Adapter integration is implemented (EWO-004; ER-004).** `Runtime::execute_message` obtains a Host Execution Handle from Host Adapter's `allocate_execution_handle` before Execution Coordinator constructs the Execution Context for that execution, supplies the handle so the returned context genuinely carries it, and releases the same handle back to Host Adapter's `release_execution_handle` after the execution concludes, on every path — successful completion, or rejection during construction, dispatch, or completion. Execution Coordinator still never calls Host Adapter directly, and Host Adapter still never calls Execution Coordinator directly — Runtime alone connects them (ADR-0016), exactly as this document's own Integration Invariants (§17, invariants 6–7) already required of any such connection.
- Several cross-component runtime flows remain incomplete. Specifically:
  - **Lifecycle Guardian and Execution Coordinator are not integrated with each other.** Execution Coordinator tracks per-instance dispatch progress (`Constructed`/`Dispatched`) entirely in its own private state; this state is never communicated to Lifecycle Guardian, which tracks its own, separate per-instance `ActorState` and has no path to Execution Coordinator's state (ADR-0016 forbids either from reaching the other directly).
  - **Successful suspend and restore paths are currently unreachable through the Runtime's own public API.** Lifecycle Guardian's `suspend` is legal only from `Executing` (ARCH-002 §15's only edge into `Suspended`). Nothing in the currently wired system ever marks an instance `Executing` in Lifecycle Guardian's own tracked state — that would require Execution Coordinator to report dispatch start to Lifecycle Guardian, which does not happen. This reporting is necessary for truthful execution-state tracking, but is not sufficient to make suspend/restore reachable against an actively executing instance; see §12. Consequently `Runtime::suspend_actor_instance` observes `IllegalTransition` for any genuinely live instance reached through the rest of the Runtime's own public API, and `Runtime::restore_actor_instance` (legal only from `Suspended`) is transitively unreachable for the same reason. Both methods are correctly implemented and independently exercisable (each crate's own tests, using a test-only state seam scoped to that crate, prove the success path exists), but neither succeeds via any sequence the Runtime's committed, integrated API actually offers today.
  - **End-to-end actor execution is not demonstrated.** No actor-defined message-handling logic exists anywhere in the workspace (ARCH-002 §7 requires the contract exist and be triggered; it does not require it be written by this milestone). `Execution Coordinator::dispatch` and `::complete` enforce the legal construct → dispatch → complete sequence and nothing more; no actor logic is actually invoked.
  - **Capability revalidation at invocation is not performed.** ARCH-002 §6 lists it among Execution Coordinator's responsibilities ("performs capability revalidation at invocation where required"); `construct_context` populates `ExecutionContext.active_capabilities` with an empty set, because its own signature supplies no path to Capability Authority's current bindings for the instance (ADR-0016).
  - **Restoration performs no capability revalidation.** ARCH-002 §9 assigns restoration validation as "a joint act: Lifecycle Guardian triggers it on resume; Capability Authority performs it." Lifecycle Guardian has no path to Capability Authority (ADR-0016); `restore` performs only the state transition (`Suspended` → `Idle`) it can honestly own.
  - **Mailbox capacity is currently unbounded, and overflow is unhandled.** ARCH-002 §13 states bounded, finite mailbox capacity is a mechanism-level MUST, with a mandatory, audited, non-silent response to overflow; §22 lists "bounded mailboxes" among its Mandatory conformance requirements. Actor Host's current mailbox storage (`mailboxes: HashMap<ActorInstanceId, Vec<Message>>`) is unbounded by construction — `enqueue` always succeeds if the target instance exists, regardless of how many messages are already queued — and no overflow-audit event exists because overflow can never occur. This is disclosed in Actor Host's own module documentation as out of scope for the current milestone; it is recorded here because it is a mandatory ARCH-002 conformance item, not merely a convenience gap.

None of the above is treated below as a defect requiring correction by this document — ARCH-003 records integration status, it does not authorize or perform integration work. Each item reappears in §18 as identified deferred work.

## 6. Integration Principles

- **Ownership remains with the responsible Trusted Core component.** No sequence described below transfers, absorbs, or duplicates any component's own responsibility as ARCH-002 §6 assigns it.
- **Runtime composes and sequences; it does not decide.** Every multi-component flow below is Runtime-mediated (ADR-0016) and Runtime's own role in it is limited to calling each component in order and propagating the result — never evaluating validity, resolving identity, or storing state itself.
- **Validation occurs before side effects.** Every flow below performs every rejection-capable check before any state-mutating call; a rejection at any step prevents every subsequent step, mutating or not.
- **Failures propagate explicitly.** No flow below swallows an error from a called component; every `Err` a component returns is either propagated unchanged or is the direct cause of a documented, narrower Runtime-level error.
- **Audit emission occurs only where ARCH-002 §18 already authorizes it.** No integration described here introduces an audit obligation ARCH-002 does not already name.
- **No integration may bypass capability or lifecycle authority.** Where a flow below does not currently perform a check ARCH-002 assigns to Capability Authority or Lifecycle Guardian, that omission is recorded as a gap (§5, §18), never worked around by an alternate, unauthorized check elsewhere.
- **Opaque types are not assigned semantics they do not carry.** `HostExecutionHandle` is treated throughout this document exactly as ARCH-002 §10 defines it — opaque — and never as unforgeable, identity-bearing, or host-specific, because the type itself carries none of that (§11).

## 7. Runtime Construction and Startup Sequence

**Already implemented** (`Runtime::bootstrap`, `TrustedCore::construct`):

1. `TrustedCore::construct()` constructs all seven Trusted Core components. Construction order — Host Adapter, Audit Emitter, Capability Authority, Actor Host, Message Gateway, Execution Coordinator, Lifecycle Guardian — is a recorded implementation decision (Host Adapter first as the seam to host resources; Audit Emitter next as the cross-cutting utility every other component may call into; the remaining five in ARCH-002 §6's own table order), not an architectural requirement; no other component ordering is prohibited by ARCH-002.
2. Verification of successful initialization (ARCH-002 §11 step 1's "Init failure → Runtime does not start" contract): currently unconditional, because construction is infallible today — no component performs I/O or any operation capable of failing during construction. The check remains a real, structural step in the code so that if construction later becomes fallible, failure routes to `Failed` rather than silently proceeding; this is a stated readiness for future work, not evidence such a path exists today.
3. Runtime Started audit-event emission (`runtime.started`), while Runtime state is still `Initializing`, via Audit Emitter — the only audit event ARCH-002 §18 requires for startup.
4. Transition to `Running` (ARCH-002 §15: `Initializing → Running`), validated against the Runtime-level legal transition set before being applied.

**Failure behavior during startup:** if step 3's audit emission fails, `bootstrap()` returns `Err` and the Runtime is never constructed into a usable value — a caller cannot obtain a `Runtime` in any state, `Initializing` included, from a failed bootstrap call (ADR-0015). No case in the current implementation causes step 1 to fail, since construction is infallible; this is a property of the current milestone, not a permanent architectural guarantee.

**Architecturally required, not yet applicable:** a fallible construction path for any of the seven components — none currently has one, so this remains theoretical until a future component gains fallible construction.

## 8. Actor-Instance Creation Flow

**Request entry point:** `Runtime::create_actor_instance(actor: &ActorId)`.

**Already implemented:**

1. Actor Host owns actor-instance existence outright: `Runtime` delegates the entire creation decision to `ActorHostHandle::create_instance(actor)` and decides nothing itself.
2. **Single-live-instance invariant** is enforced by Actor Host, not Runtime: a second `create_instance` for an `ActorId` that already has a live instance is rejected with `IllegalTransition`; an `ActorId` that was never defined is rejected with `UnknownTarget`.
3. On success, Runtime constructs and causes emission of the `actor.created` audit event (ARCH-002 §18) — the only audit obligation this operation carries.
4. Failure propagation: any error Actor Host returns is returned to the caller unchanged, with no audit emission attempted (audit is for the successful case only, mirroring ARCH-002 §18 naming "actor creation," not rejected attempts).
5. Per ADR-0015: if step 3's audit emission itself fails, this operation reports `Err` even though the instance already exists in Actor Host's own records — no rollback occurs or is required (this failure mode does not un-create the instance).

**Explicitly not part of this flow, and not invented here:** Capability Authority issuance or binding, Lifecycle Guardian registration or transition, and Execution Coordinator involvement are each separate, independently invoked Runtime operations (`issue_capability`, `validate_actor_transition`, `execute_message` respectively). None of them is called from `create_actor_instance`, and none is implied by it. Lifecycle Guardian's own internal model treats an instance absent from its tracked state as implicitly `Idle` (§12) — this is a documented internal assumption within Lifecycle Guardian, not a step actor creation performs.

## 9. Message Submission and Routing Flow

**Request entry point:** `Runtime::submit_message(message: Message, presented: &Capability)`.

**Already implemented**, in order:

1. Message Gateway: `validate_envelope(&message)` — structural/envelope integrity.
2. Message Gateway: `validate_send_authority(&message, presented)` — structural match between the presented capability and the message's declared type/destination.
3. Capability Authority: `validate(presented)` — current-validity check (not revoked, not expired) against Capability Authority's own binding records.
4. Actor Host: `live_instance(&message.destination)` — existence verification; resolves the destination `ActorId` to its current live `ActorInstanceId`, or fails if none exists.
5. Message Gateway: `admit(message)` — re-runs its own envelope-integrity check (`validate_envelope`) as a final gate; its signature carries no `Capability` parameter, so it cannot and does not re-run send-authority validation, and it performs no mailbox-capacity or overflow check of any kind (see below).
6. Actor Host: `enqueue(&instance, message)` — commits the message into that instance's mailbox, unconditionally if the instance exists; the mailbox itself is unbounded (see below).
7. Audit Emitter: `message.admitted` (ARCH-002 §18).

A failure at any of steps 1–6 causes immediate rejection: no subsequent step runs, `message.rejected` is emitted (ARCH-002 §18), and the triggering error is returned — unless that rejection-audit emission itself fails, in which case the audit failure is what the caller observes (ADR-0015).

Per ADR-0016, Runtime is the sole entity accountable for connecting Message Gateway, Capability Authority, and Actor Host here; none of the three has any dependency, direct or otherwise, on another. This flow decides nothing about validity, resolution, or storage itself — it sequences each component's own decision and causes the resulting mandatory audit event.

**Current integration status:** the envelope, send-authority, capability-validity, existence, and audit-emission portions of this flow are fully implemented and integrated (SRP-003, SRP-004). **Not implemented:** bounded mailbox capacity and audited overflow handling, which ARCH-002 §13 and §22 name as a mechanism-level MUST / Mandatory conformance requirement. Actor Host's mailbox is currently unbounded (`Vec<Message>` with no capacity check), so `enqueue` never rejects for capacity reasons and no overflow-audit event can ever be emitted. This is deferred work (§18), not a completed part of this flow.

## 10. Execution Flow

**Request entry point:** `Runtime::execute_message(message: Message)`. Precondition: `message` has already been legitimately admitted via §9 — this flow performs none of Message Gateway's envelope/structural checks and none of Capability Authority's current-validity check itself; those are §9's completed responsibility and are not repeated here.

**Already implemented**, in order:

1. Actor Host: `live_instance(&message.destination)` — the same existence check §9 already performs before admission, re-applied here (ARCH-002 §11 step 11: "Actor gone → abort, no execution").
2. Host Adapter: `allocate_execution_handle()` — acquires a Host Execution Handle for this execution (EWO-004). On failure (currently unreachable, since allocation is unconditional), no handle exists to release.
3. Execution Coordinator: `construct_context(&instance, &message, handle)` — builds an `ExecutionContext`, embedding the handle Runtime just obtained (EWO-004); rejects with `IllegalTransition` if `instance` already has an unfinished construction in progress (ARCH-002 §12's single-execution-ownership rule, enforced by direct construction).
4. Execution Coordinator: `dispatch(context)` — advances the tracked execution from `Constructed` to `Dispatched`; rejects if the context was never legitimately constructed or was already dispatched.
5. Execution Coordinator: `complete(context)` — removes the tracked execution entirely, freeing the instance for a subsequent execution; rejects if the context was never dispatched.
6. Host Adapter: `release_execution_handle(handle)` — releases the handle acquired in step 2 back to Host Adapter (EWO-004). If this fails, `execute_message` reports that failure instead of success, even though the execution itself completed.
7. Audit Emitter: `execution.completed` (ARCH-002 §18).

A failure at steps 1, 3, 4, or 5 releases the handle acquired in step 2 (if any was acquired) before emitting `execution.failed` (ARCH-002 §18) and returning the triggering error — or, if that release itself fails, the release's own error is returned instead, subject to the same "a later mandatory step's failure overrides the original error" rule already established for audit emission (ADR-0015; EWO-004, "Handle Lifecycle Invariant").

**Already implemented but not integrated with a wider flow:** Capability Authority and Lifecycle Guardian are constructed as part of `TrustedCore` but are **not called anywhere in this flow**. `ExecutionContext.active_capabilities` is populated empty (Execution Coordinator has no path to Capability Authority's bindings for the instance); no call informs Lifecycle Guardian that the instance has entered `Executing`. `ExecutionContext.host_execution_handle` (EWO-004) now carries a genuine Host-Adapter-issued value, supplied by Runtime — Execution Coordinator itself still has no path to Host Adapter (§11).

**Architecturally required for the integration phase, not yet performed:** capability revalidation at invocation (ARCH-002 §6's own stated Execution Coordinator responsibility); communicating dispatch start/end to Lifecycle Guardian, so that `Executing` becomes a state Lifecycle Guardian can actually observe, which is a necessary but not sufficient precondition for suspend/restore reachability against an actively executing instance; see §12.

**Not required by the Minimal Runtime Profile at this milestone** (ARCH-002 §21): actual actor-defined message-handling logic being invoked; real host-level execution.

No claim is made anywhere in this section that the current runtime performs actor logic. It performs exactly the mechanism ARCH-002 assigns Execution Coordinator — enforcing the legal construct/dispatch/complete sequence — and nothing beyond it.

## 11. Host Execution-Handle Boundary

- `HostExecutionHandle` (`synapse-common`) is currently opaque and zero-field: `#[derive(Debug, Clone, Default)] pub struct HostExecutionHandle;`. It carries no thread id, process handle, or any other host-specific data, because it has no field to hold one.
- Host Adapter's current implementation (SRP-007) tracks allocation/release **balance** internally (a single counter of currently-outstanding handles); it does not, and structurally cannot, verify a specific handle's identity or provenance, because the type it operates on carries none.
- Execution Coordinator's `construct_context` still never calls Host Adapter directly, and Host Adapter still never calls Execution Coordinator directly (ADR-0016 unchanged) — but `construct_context` now accepts a `HostExecutionHandle` parameter (EWO-004) and embeds exactly the value it is given. Runtime obtains that value from Host Adapter's `allocate_execution_handle` before calling `construct_context`, and releases it back to Host Adapter via `release_execution_handle` after the execution concludes, on every path (§10).
- This connection is now implemented (EWO-004): `construct_context`'s signature gained exactly one additional parameter to receive the handle — the minimum interface evolution EWO-004 itself authorized, per this document's own identification of the boundary above. No further signature change was made, and none is implied by this update.
- No claim of unforgeability or distinct handle identity exists anywhere in current authority. ARCH-002 §10 requires `HostExecutionHandle` be opaque — a materially weaker property than the unforgeability ARCH-001 §5.2 requires of `Capability` — and the current implementation fully satisfies that weaker requirement. This is treated here as a known, disclosed constraint of the shared type's own definition, not automatically as an architectural defect requiring resolution by this document.

## 12. Lifecycle Integration

Lifecycle states used below are exactly ARCH-002 §15's set; none is introduced or renamed here: `Defined → Initializing → Idle ⇄ Ready → Executing → {Idle | Suspended | Failed | Stopping} → Terminated`.

- **Creation:** owned by Actor Host (§8). Lifecycle Guardian tracks no state for an instance until Actor Host has created it, and is never called during creation. Lifecycle Guardian's own internal model treats any instance absent from its tracked map as implicitly `Idle` — reasoned, not observed: by the time an instance exists at all, it has structurally already passed through `Defined → Initializing → Idle`, and Lifecycle Guardian has no path to Actor Host (ADR-0016) to observe it mid-transition. This is Lifecycle Guardian's own documented assumption, consistent with but not verified against Actor Host's actual internal state.
- **Readiness / execution-related transitions:** ARCH-002 §15 names `Idle ⇄ Ready → Executing` and `Executing → Idle`. No component currently drives any of these transitions in Lifecycle Guardian's tracked state — Execution Coordinator's own dispatch tracking is separate, private state never communicated to Lifecycle Guardian (§10). `Runtime::validate_actor_transition` can check whether a proposed transition would be legal from Lifecycle Guardian's current (defaulted or explicitly tracked) view, but nothing in the integrated system currently drives that view to `Executing`.
- **Suspension:** owned by Lifecycle Guardian (`Runtime::suspend_actor_instance`, verifying instance existence via Actor Host first, then delegating). Legal only from `Executing`. **Currently unreachable via the Runtime's own integrated public API**, because nothing marks an instance `Executing` in Lifecycle Guardian's tracked state (see above). The suspend operation itself is correctly implemented and independently exercisable using Lifecycle Guardian's own test-only state seam, scoped to that crate alone.
- **Restoration:** owned by Lifecycle Guardian (`Runtime::restore_actor_instance`, same existence-verification pattern). Legal only from `Suspended`. Transitively unreachable for the same reason suspension is. Performs the state transition (`Suspended → Idle`) only — no capability-binding revalidation is performed, because Lifecycle Guardian has no path to Capability Authority (ADR-0016), and ARCH-002 §9 assigns that revalidation jointly to both.
- **Termination:** owned by Actor Host (`Runtime::terminate_actor_instance`), independent of Lifecycle Guardian's tracked `ActorState`. Clears Actor Host's own liveness record; a subsequent operation against the same instance id fails with `UnknownTarget`.

**Honest reachability limitation, corrected and stated precisely (ARCH-002 v0.2.0):** as of the current implementation, no sequence of calls through `Runtime`'s own public API can produce a successful `suspend_actor_instance` or `restore_actor_instance` call against a genuinely live instance. Earlier drafts of this document attributed this solely to "Execution Coordinator and Lifecycle Guardian not yet being integrated" — correct as far as it went, but incomplete in a way that matters for scoping future work: ARCH-002 §10/§12/§15 (v0.2.0) now make explicit that an actor instance is truthfully `Executing` only for exactly as long as it owns a live Execution Context, and that within a synchronous Runtime — which this one genuinely is; no concurrent, asynchronous, or otherwise interruptible execution mechanism exists anywhere in the current implementation — no Runtime API call other than the one currently performing a given execution can ever observe that instance as `Executing`. Integrating Execution Coordinator's dispatch tracking with Lifecycle Guardian's own tracked state (the item §18 lists) would make Lifecycle Guardian's state *truthful* — a real, valuable, independently deliverable improvement — but it would **not**, by itself, make `suspend_actor_instance`/`restore_actor_instance` reachable against an actively-executing instance from a separate call, because no such call can ever arrive while that instance is genuinely `Executing` in a synchronous Runtime. Genuine suspend/restore reachability against an actively-executing instance requires a real asynchronous or otherwise interruptible execution mechanism first — a prerequisite this integration phase does not provide and is not scoped to provide. See §18's now-split entries for the corrected consequence.

## 13. Capability Enforcement

Existing capability checks, mapped to their actual current boundary:

| Check | Owner | Where it currently runs |
|---|---|---|
| Structural send-authority match (message type/destination vs. presented capability) | Message Gateway | `submit_message`, step 2 (§9) |
| Current validity (not revoked, not expired) | Capability Authority | `submit_message`, step 3 (§9) |
| Non-amplification / issuing ceiling at mint time | Capability Authority | `issue_capability` (Runtime operation, independent of creation/submission/execution) |
| Issuer-forgery rejection | Capability Authority | `issue_capability`, and transitively `submit_message` (a capability never issued by this Runtime's own Capability Authority instance is rejected as `NonForgeryViolation`) |
| Revalidation at invocation | Capability Authority | **Not currently performed.** ARCH-002 §6 assigns this to Execution Coordinator; `construct_context` supplies an empty `active_capabilities` set (§10) |
| Revalidation on restoration | Capability Authority (triggered by Lifecycle Guardian) | **Not currently performed** (§12) |

No check beyond this table exists, and none is added here merely because it might appear desirable — every row above is either directly observed in the committed implementation or explicitly recorded as a documented gap. The two "not currently performed" rows are the integration gap this document identifies for future work (§18); ARCH-003 does not resolve them.

## 14. Audit Integration

Every currently emitted audit event, its emitter, its trigger, and its integration status:

| Event | Emitter | Triggering operation | Emitted on | Implemented | Further integration deferred |
|---|---|---|---|---|---|
| `runtime.started` | Audit Emitter | `Runtime::bootstrap` | Success of construction, before entering `Running` | Yes | No |
| `runtime.shutdown` | Audit Emitter | `Runtime::shutdown` | Entering `Stopping`, before `Stopped` | Yes | No |
| `actor.created` | Audit Emitter | `Runtime::create_actor_instance` | Success only | Yes | No |
| `actor.terminated` | Audit Emitter | `Runtime::terminate_actor_instance` | Success only | Yes | No |
| `message.rejected` | Audit Emitter | `Runtime::submit_message` (via `reject_message`) | Any rejection at steps 1–6 of §9 | Yes | No |
| `message.admitted` | Audit Emitter | `Runtime::submit_message` | Success only | Yes | No |
| `capability.issued` | Audit Emitter | `Runtime::issue_capability` | Success only | Yes | No |
| `execution.completed` | Audit Emitter | `Runtime::execute_message` | Success only | Yes | No |
| `execution.failed` | Audit Emitter | `Runtime::execute_message` (via `fail_execution`) | Any rejection at steps 1, 3, 4, or 5 of §10 (or a Host Adapter release failure at step 6, EWO-004) | Yes | No |
| `actor.suspended` | Audit Emitter | `Runtime::suspend_actor_instance` | Success only | Yes, but currently unreachable in an integrated flow (§12) | Yes — see §12/§18: truthful execution-state tracking plus a genuinely asynchronous execution mechanism, not integration alone |
| `actor.restored` | Audit Emitter | `Runtime::restore_actor_instance` | Success only | Yes, but currently unreachable in an integrated flow (§12) | Yes — same dependency |

No event beyond this table exists in the current implementation, and none is created here. Every event listed is one ARCH-002 §18 already names as a minimum required event; ARCH-003 introduces no new event and no new emitter.

## 15. Failure Propagation and Atomicity

- **Validation-before-mutation ordering:** every flow in §7–§10 performs every rejection-capable check before any state-mutating call in the same flow (§6). No flow described in this document mutates any component's state before the checks that could reject the operation have all passed.
- **Component error propagation:** every error a called component returns is either returned to the caller unchanged (e.g., Actor Host's `UnknownTarget`/`IllegalTransition` from creation, existence, or single-live-instance checks) or is the direct, undisguised cause of a narrower Runtime-level rejection (e.g., `reject_message`, `fail_execution`). No flow substitutes a different error for the one a component actually returned.
- **Prevention of downstream side effects following rejection:** confirmed directly in §9 and §10 — a rejection at any numbered step in either flow prevents every later step in that same flow from running, with one disclosed exception (EWO-004): a rejection during Execution Context construction, dispatch, or completion (§10 steps 3–5) still causes the Host Execution Handle acquired at step 2 to be released at that point, out of its normal step-6 order, before `execution.failed` is emitted. This is deliberate compensating cleanup for an already-acquired resource, not continued forward progress of the rejected operation — no other component's state is touched, and the operation is still reported as failed.
- **Audit behavior on rejected operations:** `message.rejected` and `execution.failed` are each emitted exactly once per rejection, per ARCH-002 §18; a rejection during actor creation, termination, or capability issuance carries no separate rejection-audit obligation, since ARCH-002 §18 names only the successful form of each of those three operations.
- **Rollback limitations, stated plainly:** no operation described in this document performs rollback of already-committed component-level state when a later step fails. Concretely: if `create_actor_instance`'s own mandatory audit emission fails, the instance already created in Actor Host's own records is not un-created (ADR-0015, EWO-002's "Audit Failure Semantics"); if `submit_message`'s final admission-audit emission fails, the message already enqueued by Actor Host's `enqueue` is not removed. This is an existing, deliberate consequence of ADR-0015, not something this document introduces or extends.
- **Recovery behavior left undefined or deferred:** what a caller should do after observing an audit-emission-caused failure with otherwise-committed component state (retry, discard, reconcile) is not defined by ARCH-002 or by this document — it remains deferred, consistent with ARCH-002 §23's Deferred Architecture table not yet addressing recovery policy. No transaction, compensating-action, or exactly-once semantics are invented here to fill that gap.

## 16. Runtime Shutdown

**Already implemented** (`Runtime::shutdown`):

1. Transition to `Stopping` (validated against the Runtime-level legal transition set).
2. Emit `runtime.shutdown` (ARCH-002 §18) — the only audit obligation shutdown carries.
3. Transition to `Stopped`.

`shutdown` consumes `Runtime` by value: a terminated Runtime cannot be reused, enforced at compile time. If step 2's audit emission fails, `shutdown` returns `Err` and the Runtime instance is already consumed — this is the same audit-failure-fails-the-operation pattern used throughout (ADR-0015), applied here with no further state for the caller to observe or recover, since ownership was already given up.

**Integration obligation as execution becomes real, not yet applicable:** ARCH-002 §11 step 20 states shutdown must produce "no silent loss of admitted/committed work" under a forced shutdown, with "best-effort suspension, audited." No in-flight execution or admitted-but-undispatched message currently exists to lose, because no actor logic runs and no scheduler drives dispatch order yet (ARCH-002 §21 Minimal Runtime Profile); this obligation becomes concretely testable only once those exist, and is recorded here as a future integration concern rather than as a defect in the current, correctly-scoped shutdown implementation.

## 17. Integration Invariants

The following hold given the current implementation and ARCH-002's own authority; none is a new architectural rule — each restates an existing ARCH-002 §6/§9/§15/§20 assignment in integration terms:

1. Actor Host is authoritative for live actor-instance existence; every other component and every Runtime flow that needs to know whether an instance exists asks Actor Host, never maintains its own copy.
2. Capability Authority is authoritative for capability validity; no component caches a validity determination independently of it.
3. Lifecycle Guardian is authoritative for lifecycle-transition legality for the state it tracks; no component performs its own parallel legality check.
4. Message Gateway is authoritative for message admission and routing within its assigned scope (envelope integrity, structural send-authority, mailbox admission); it does not decide capability validity or destination liveness itself.
5. Execution Coordinator owns execution-context coordination (construction, dispatch, completion, one-owner enforcement) and nothing beyond it.
6. Host Adapter owns host interaction within its own interface (`allocate_execution_handle`, `release_execution_handle`) and is not reached by any other component directly.
7. Runtime sequences components without duplicating their authority: every multi-component flow in this document is Runtime-mediated (ADR-0016), and at no point does Runtime itself decide validity, resolve identity, or hold authoritative state a Trusted Core component already owns.

## 18. Deferred Integration Work

The following is identified as future work, without prescribing implementation detail this document has no authority to fix:

- **Truthful execution-state tracking** (corrected scope, ARCH-002 v0.2.0): Execution Coordinator's dispatch tracking integrated with Lifecycle Guardian's own tracked state, so that an instance is marked `Executing` for exactly the genuine duration of its live Execution Context (ARCH-002 §10/§12/§15's coupling rule) — narrow, deliverable within the current synchronous Runtime, and independent of the item below. This alone does not make suspend/restore reachable against an actively-executing instance; see the next item.
- **Genuine suspend/restore reachability against an actively-executing instance** (distinct from the item above, per ARCH-002 v0.2.0's clarification): unreachable until the Runtime gains a real asynchronous, concurrent, or otherwise interruptible execution mechanism — within a synchronous Runtime, no call other than the one currently performing an execution can ever observe that instance as `Executing` (ARCH-002 §12), regardless of how faithfully Lifecycle Guardian's state otherwise tracks it. This is a materially larger prerequisite than the item above and is not unlocked by it.
- The identity limitation of `HostExecutionHandle` (§11), if a future requirement demands distinguishing specific handles from one another — this would require a change to the shared type itself, outside Host Adapter's or Execution Coordinator's own scope, and outside this document's authority to resolve.
- A complete actor execution flow, including actual actor-defined message-handling logic being invoked during dispatch.
- Capability revalidation during restoration, per ARCH-002 §9's "joint act" assignment, once Lifecycle Guardian and Capability Authority have an authorized interaction path.
- Capability revalidation at invocation, per ARCH-002 §6's Execution Coordinator responsibility, once Execution Coordinator and Capability Authority have an authorized interaction path.
- Bounded mailbox capacity and audited overflow handling (ARCH-002 §13, §22 — currently unbounded and unenforced; see §5, §9).
- End-to-end audit sequencing once execution becomes real — for example, confirming ordering guarantees (or the deliberate absence of them) across `message.admitted`, a future `execution.started` observation point (no such event currently exists or is proposed here), and `execution.completed`/`execution.failed`.
- Deterministic cleanup after partial execution failure, once real actor logic and real host interaction exist to fail partway through.
- A first runnable actor — the first concrete demonstration of one complete, real message-handling cycle.
- An end-to-end Runtime demonstration exercising construction through shutdown with genuine actor execution in between.
- SDK and host-specific implementation work — entirely out of this document's scope (§19), listed here only as work this integration phase's completion would eventually unblock.

## 19. Non-Goals

ARCH-003 does not define, and this document takes no position on:

- new Trusted Core components beyond the seven ARCH-002 §6 already names;
- new lifecycle states beyond ARCH-002 §15's existing set;
- new capability architecture or semantics beyond ARCH-001 §5.2 and ARCH-002 §9;
- networking, distributed execution, or cross-host routing (ARCH-002 §21, §23 — deferred);
- clustering or high availability;
- persistence or storage architecture (ARCH-002 §23 — deferred);
- SDK design;
- AI orchestration;
- plugin frameworks (ARCH-002 §19 already establishes there is no generic, unrestricted plugin interface);
- host-specific process or thread models (ARCH-002 §17 leaves the isolation mechanism an implementation choice, deliberately not fixed here or there);
- public API signatures not already present in the committed implementation — where an integration gap requires a future signature change (e.g., §11's Execution Coordinator/Host Adapter boundary), this document identifies the boundary and defers the design to future, separately authorized work.

## 20. Conformance and Future Work-Order Requirements

Future integration work claiming conformance to this document must demonstrate, at minimum:

- conformance to ARCH-002's component responsibility, ownership, and prohibition table (§6) — no work item may cause a component to exercise authority ARCH-002 assigns to another;
- conformance to ADR-0016 — no new direct peer-to-peer path between two Trusted Core components; every new cross-component sequence remains Runtime-mediated;
- conformance to ADR-0015 — any new mandatory audit obligation introduced by future work fails its own operation on emission failure, exactly as every existing obligation in §14 already does;
- an explicit update to this document's §5 (Current Implementation Baseline) and §18 (Deferred Integration Work) reflecting what has newly become implemented-and-integrated versus what remains deferred — this document is expected to be revised as integration work completes, not left to silently drift out of date;
- no claim, in any future work order or engineering report, that an interaction is "integrated" unless it is reachable through the Runtime's own public API in the same sense §5 and §12 use that term here — an operation correctly implemented but reachable only through a test-only, single-crate seam does not meet this bar, exactly as suspend/restore do not meet it today.

A future work order authorizing any of §18's items must itself be evaluated against ARCH-002 and this document before implementation begins, per the existing EWO process (STD-001 §46) — this document authorizes no implementation and is not itself a work order.

## 21. Diagrams

Diagrams below reflect exactly what §7–§16 establish. Any interaction not shown, or explicitly marked deferred, is not implemented in the current integrated system.

**Runtime startup sequence (fully implemented)**

```text
Runtime::bootstrap()
  |
  v
TrustedCore::construct()
  -- Host Adapter, Audit Emitter, Capability Authority, Actor Host,
     Message Gateway, Execution Coordinator, Lifecycle Guardian --
  |
  v
Verify initialization (unconditional today; construction is infallible)
  |
  v
Audit Emitter: emit "runtime.started"   (state: Initializing)
  |  failure -> bootstrap() returns Err; no Runtime value exists
  v
Runtime state: Initializing -> Running
  |
  v
Runtime ready (state() == Running)
```

**Message submission and execution sequence (implemented flows shown separately; not chained automatically)**

```text
submit_message(message, presented)                execute_message(message)
  |                                                   (precondition: message
  v                                                    already admitted above)
Message Gateway: validate_envelope                   |
  | reject -> message.rejected, return Err            v
  v                                                 Actor Host: live_instance(dest)
Message Gateway: validate_send_authority              | reject -> execution.failed, return Err
  | reject -> message.rejected, return Err            v
  v                                                 Host Adapter: allocate_execution_handle()
Capability Authority: validate(presented)              | fail -> execution.failed, return Err  (EWO-004)
  | reject -> message.rejected, return Err            v
  v                                                 Execution Coordinator: construct_context(handle)
Actor Host: live_instance(destination)                 | reject -> release handle, execution.failed  (EWO-004)
  | reject -> message.rejected, return Err            v
  v                                                 Execution Coordinator: dispatch(context)
Message Gateway: admit(message)                         | reject -> release handle, execution.failed  (EWO-004)
  | reject -> message.rejected, return Err            v
  v                                                 Execution Coordinator: complete(context)
Actor Host: enqueue(instance, message)                   | reject -> release handle, execution.failed  (EWO-004)
  |                                                     v
  v                                                 Host Adapter: release_execution_handle(handle)  (EWO-004)
Audit Emitter: emit "message.admitted"                   | fail -> execution.failed, return Err
                                                        v
                                                      Audit Emitter: emit "execution.completed"

  [ DEFERRED — not called by either flow above: ]
  [   Lifecycle Guardian (never invoked here)    ]
  [   actor-defined message handling (does not exist yet) ]
```

**Lifecycle ownership map**

```text
Defined --(Actor Host: define)--> Initializing --(implied; not separately
                                                    tracked by any component)
   --> Idle <-> Ready --(NEVER DRIVEN by any integrated call)--> Executing
                                                                     |
                          Lifecycle Guardian: suspend()  <-- requires Executing
                          [ UNREACHABLE today: nothing marks Executing ]
                                                                     |
                                                                     v
                                                                Suspended
                                                                     |
                          Lifecycle Guardian: restore()  <-- requires Suspended
                          [ UNREACHABLE today, transitively ]
                                                                     |
                                                                     v
                                                                   Idle

Terminated <--(Actor Host: terminate_instance, independent of the above)-- any
```

**Trusted Core authority boundaries in the current Runtime**

```text
Runtime (sequences only; owns Runtime-level state and TrustedCore)
  |
  +-- calls --> Actor Host            (existence, liveness, mailbox storage)
  +-- calls --> Message Gateway       (envelope, structural send-authority, admission)
  +-- calls --> Capability Authority  (validity, issuance)
  +-- calls --> Execution Coordinator (context construction, dispatch, completion)
  +-- calls --> Lifecycle Guardian    (transition legality, suspend, restore)
  +-- calls --> Audit Emitter         (every event in §14)
  +-- calls --> Host Adapter          (allocate_execution_handle, release_execution_handle — EWO-004)

  No line above this box exists between any two Trusted Core components
  directly (ADR-0016) -- every connection shown is Runtime-mediated.
```

## 22. References

- ARCH-000 — Introduction
- ARCH-001 — Constitutional Architecture
- ARCH-002 — Runtime Architecture
- ADR-0011 — Bootstrap Approval Authority
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- GOV-003 — Governance Model
- GOV-010 — Decision Framework
- STD-001 — Documentation Standards
- `synapse-runtime` @ `01047157b6783d816fed80361b40206a98ba6f2f`: `runtime/src/lib.rs`, `core/actor-host/src/internal.rs`, `core/message-gateway/src/internal.rs`, `core/capability-authority/src/internal.rs`, `core/execution-coordinator/src/internal.rs`, `core/lifecycle-guardian/src/internal.rs`, `core/host-adapter/src/internal.rs`
- EWO-004 — Runtime Integration Bootstrap — Host Execution Handle Binding (work-orders/EWO-004-Runtime-Integration-Bootstrap.md)
- ER-004 — Runtime Integration Bootstrap — Host Execution Handle Binding — Engineering Report (engineering-reports/ER-004-Runtime-Integration-Bootstrap.md)

## 23. Change History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-13 | Denver Jacobs | Initial Draft. Defines Trusted Core runtime integration status against the committed `synapse-runtime` implementation through SRP-007 (Host Adapter). Records current integration baseline, integration principles, per-flow sequencing, host-execution-handle boundary, lifecycle-integration reachability limitations, capability-enforcement boundary, audit integration, failure propagation, integration invariants, and deferred integration work. Does not redesign or supersede ARCH-002. |
| 0.2.0 | 2026-07-13 | Denver Jacobs | Conformance update per §20, recording EWO-004/ER-004's completion: Execution Coordinator ↔ Host Adapter integration is now implemented and integrated. Updated §5 (moved this item from the incomplete-flows list to a newly-implemented statement), §10 (execution flow now shows Host Adapter allocation/release around Execution Coordinator's construct/dispatch/complete sequence), §11 (the previously-identified boundary is now closed, by exactly the minimum interface evolution ARCH-003 itself anticipated), §18 (removed the now-completed deferred-work item), §21 (both diagrams updated to show the real Host Adapter calls and remove it from the deferred callouts), and §22 (evidentiary commit hash updated to the post-EWO-004 commit; EWO-004/ER-004 added as references). No architectural principle, ownership boundary, interaction model, Runtime sequencing rule, Trusted Core boundary, or design rationale was changed — every edit restates already-published architecture (ARCH-002 §6/§10, ADR-0016) as now-realized implementation status, per this document's own §20 conformance requirement. |
| 0.3.0 | 2026-07-13 | Denver Jacobs | Corrective update following ARCH-002 v0.2.0's semantic clarification of `Executing`'s ownership and truthfulness (an EWO-005 planning-review finding, resolved as an ARCH clarification, not a new ADR). This document's own §12 previously attributed unreachable suspend/restore solely to "Execution Coordinator and Lifecycle Guardian not yet being integrated" — accurate as far as it went, but incomplete: even after that integration, no Runtime API call other than the one currently performing an execution can observe an instance as `Executing` in a synchronous Runtime (ARCH-002 §12 v0.2.0), so integration alone does not deliver reachability against an actively-executing instance. Rewrote §12's "Honest reachability limitation" paragraph accordingly. Split §18's single "Execution Coordinator ↔ Lifecycle Guardian integration" item into two distinct items — "Truthful execution-state tracking" (narrow, deliverable now) and "Genuine suspend/restore reachability against an actively-executing instance" (requires a real asynchronous or otherwise interruptible execution mechanism, not provided by this integration phase) — and removed the now-redundant, now-imprecise "Successful suspend/restore reachability... (currently blocked on the first item above)" bullet that conflated the two. No architectural principle, ownership boundary, Runtime sequencing rule, or Trusted Core boundary was changed by this revision; it applies ARCH-002 v0.2.0's own clarification to this document's already-established integration-status sections, per §20's standing conformance requirement. |

## 24. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs | Drafted | 2026-07-13 |
| Technical Review | TBD | Pending | |
| Approval Authority | Chief Architect (vacant); Founder (interim) | Pending | |
