---
document_id: ARCH-004
title: Local Actor Supervision Architecture
project: SynapseOS
specification: SynapseOS — local actor supervision, failure escalation, and restart ownership, realizing the Lifecycle Architecture ARCH-002 §15/§23 defers
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-14
last_updated: 2026-07-14
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
    - GOV-004 (Engineering Principles)
  standards:
    - STD-001 (Approved)
  architecture:
    - ARCH-000 (Draft — whole-system introduction)
    - ARCH-001 (Draft — constitutional foundation; §5.4 host-process vs Runtime-internal supervision boundary)
    - ARCH-002 (Draft — Runtime architecture; §7, §11, §13, §14, §15, §16, §18, §20, §23, §25 directly extended by this document)
    - ARCH-003 (Draft — Runtime integration status, current baseline evidence)
  rfcs: None
  adrs:
    - ADR-0015 (Draft; Approved disposition recorded separately per STD-001 §31 — Audit Emitter Failure Semantics)
    - ADR-0016 (Draft; Approved disposition recorded separately per STD-001 §31 — Trusted Core Interaction Rule)
    - ADR-0017 (Draft — Bootstrap Capability Trust Root)
  roadmap: None
  research: None
  operational: None
  source_artifacts:
    - synapse-runtime @ 5ccc7f9083a71adc6ee704b2322a701935765679 (working tree, including uncommitted SRP-001–007, EWO-004, EWO-005, EWO-006, and the actor-to-actor messaging admission pipeline)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-004 — Local Actor Supervision Architecture

*Filename pattern: `ARCH-004-Local-Actor-Supervision-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Document Control and Status

| Field | Value |
|---|---|
| Document ID | ARCH-004 |
| Title | Local Actor Supervision Architecture |
| Version | 0.1.0 |
| Status | **Draft** |
| Author | Denver Jacobs |
| Approval authority | Chief Architect (Class B, per GOV-010 §5), vacant; Founder (interim) |
| Created | 2026-07-14 |
| Classification | Public |

This document is Draft. It has not been reviewed or approved, and nothing in it should be read as operative or binding until it completes the same governance process ARCH-002 and ARCH-003 are themselves subject to (GOV-003, GOV-010). This document introduces no implementation and authorizes none; it establishes architecture only, to be realized by a future Engineering Work Order (STD-001 §46) once approved.

This document also serves as the authoritative source ARCH-002 §15 and §23 already anticipated and named: ARCH-002 §15 states "Deeper lifecycle *policy* — supervision strategy, restart backoff, detailed sub-states beyond architectural significance — is deferred to a later Lifecycle Architecture document," and ARCH-002 §23's Deferred Architecture table lists "Lifecycle Architecture: Supervision/restart policy, backoff strategy" against the contract "State model and legal-transition set (§15); revalidate-on-resume requirement." This document is that later Lifecycle Architecture document, scoped specifically to local (single-Runtime-process) actor supervision, failure escalation, and restart ownership. Backoff *algorithms*, restart-count *thresholds*, and other numeric policy remain out of scope here (§4) and remain deferred to future, separately authorized work.

## 2. Purpose

This document defines the authoritative architecture for **local actor supervision, failure escalation, and restart ownership** in the SynapseOS Runtime: which component observes actor execution failure, which component decides what happens next, which component performs the resulting action, how a restarted actor's identity, capabilities, and mailbox are treated, how repeated failure escalates, and what must be truthfully audited throughout. It resolves the "(supervision policy, deferred)" annotation on the `Failed` state already present in ARCH-002 §25's own Actor Lifecycle diagram, and the supervision/restart deferral already recorded in ARCH-002 §15 and §23.

This document does not select numeric policy (restart limits, backoff timing), does not define implementation APIs, and does not authorize implementation. It is architecture: what must be true, who owns what, and why — consistent with ARCH-002's own stated method (`ARCH-002 §1`: "precise enough that independent engineering teams could implement recognizably conformant runtimes without identical internal code").

## 3. Scope

**In scope:** the architectural placement of a new supervision responsibility within the existing Runtime component model; the boundary between supervision *policy* and the mechanisms it invokes; a failure taxonomy distinguishing genuine actor-execution failure from admission, authorization, backpressure, and infrastructure failure; the restart identity model (what a "restarted actor" is, in terms already fixed by ARCH-002 §7); the scheduling treatment of a `Failed` instance; mailbox and pending-message treatment across restart; a local, `ActorId`-keyed parent/child supervision model; an escalation model connecting restart exhaustion to parent supervision and, ultimately, to the Runtime itself; capability continuity across restart; and the audit obligations supervision introduces.

**Out of scope:** restart backoff algorithms, numeric restart-limit thresholds, and other quantitative policy (deferred to future policy-level work built on this architecture); message retry, redelivery, or dead-letter mechanisms; durable or persistent mailboxes; mailbox content transfer across restart (identified, not designed — §15); distributed or cross-process supervision, remote failure detection, clustering, or failover placement; workflow compensation and effect-system retries; timers and scheduled restart; persistence and event-sourcing/checkpoint recovery generally; any public Rust API, struct, trait, or function signature. See §4 for the complete non-goals statement and §22 for the future-compatibility boundary.

**Explicitly distinguished from host-process supervision.** ARCH-001 (`ARCH-001 §5.4`, Execution Semantics / host boundary) already states: "Host-level process supervision (restarting a crashed SynapseOS Runtime process) is the host platform's responsibility; the Runtime cannot supervise its own crash, for the same structural reason no Runtime mechanism can be scheduled by itself." This document is about a different, narrower concern: supervision of one **actor instance's** execution failure by the still-healthy Runtime process that instance runs inside. It does not touch, extend, or substitute for host-level process supervision.

## 4. Non-Goals

This document does not define, and takes no position on:

- restart backoff algorithms, timing, jitter, or numeric restart-limit thresholds and windows;
- message retry, redelivery, or acknowledgement protocols;
- dead-letter queues or dead-letter storage;
- durable or persistent mailboxes, or mailbox content transfer across restart (§15 identifies this as deferred work, not designed here);
- state snapshots, event sourcing, or checkpoint recovery;
- distributed supervision, remote failure detection, node monitoring, or clustering;
- workflow compensation or generalized effect-retry frameworks;
- timers or scheduled/delayed restart;
- any Rust struct, trait, enum, method signature, function name, or field layout;
- any new Trusted Core component (ARCH-002 §5–§6 is unchanged; see §10);
- any new lifecycle state beyond ARCH-002 §15's existing set;
- any new constitutional guarantee beyond ARCH-001's four (non-forgery, integrity, enforcement-at-invocation, revocation-state enforcement);
- host-level process supervision (§3).

## 5. Existing Architectural Context

This document amends no prior authority. It extends, and is bound by, the following without redefinition:

- **ARCH-000** established SynapseOS's whole-system introduction; this document inherits it without restatement.
- **ARCH-001** fixes the four constitutional guarantees and the host-process/Runtime-internal supervision boundary (`ARCH-001 §5.4`, §3). This document introduces no new guarantee and weakens none; every supervision action described here remains subject to all four.
- **ARCH-002** remains the sole authority for Trusted Core component responsibility, ownership, and prohibition (`ARCH-002 §6`), the Constitutional Execution Cycle (`ARCH-002 §11`), the actor and Runtime lifecycle state models (`ARCH-002 §15`), the minimum audit-event set (`ARCH-002 §18`), and the Runtime Interfaces table (`ARCH-002 §20`). This document amends none of it. It specifically completes the deferral `ARCH-002 §15` and `ARCH-002 §23` already recorded (§1 above), and it extends, without altering, the already-stated facts that:
  - "A restarted actor (a new instance under supervision) receives a new **instance identity**, distinct from its unchanging logical identity; capabilities generally bind to logical identity" (`ARCH-002 §7`);
  - "a new instance under supervision receives a new mailbox, with in-flight message handling deferred to Lifecycle Architecture" (`ARCH-002 §13`);
  - "When a logical actor is replaced by supervision, routing to its logical identity resolves to the new instance only through Lifecycle Guardian's restoration path (§9), never a transparent, unvalidated swap" (`ARCH-002 §14`);
  - the Actor lifecycle diagram already shows `Failed --(supervision policy, deferred)--> {Initializing | Terminated}` (`ARCH-002 §25`);
  - "Actor creation" is already an interface whose caller may be "Actor Host client (**bootstrap or supervisor**)" (`ARCH-002 §20`).
- **ARCH-003** records the current, verified implementation baseline and integration status of the seven Trusted Core components (`ARCH-003 §5`, §17). This document treats ARCH-003's baseline as evidence for §6 below, not as authority — where §6 states a current-implementation fact, it is independently verified against `synapse-runtime` (§6), not merely asserted from ARCH-003.
- **ADR-0015** (Audit Emitter Failure Semantics) governs every audit-emission failure behavior this document assumes for supervision-triggered audit obligations (§19): a mandatory audit emission that fails causes the *reporting* operation to fail, without rollback of already-committed component-level state.
- **ADR-0016** (Trusted Core Interaction Rule) is the specific authority this document depends on most directly. Its two rules — "the Runtime is the sole architectural entity accountable for establishing and coordinating Trusted Core interactions" (Rule 1) and "Trusted Core components must not independently establish or own direct peer interaction paths" (Rule 2) — are extended, not amended, by this document to a new participant (Supervisor, §10): Supervisor connects to every other component exactly as any other component would, through the Runtime, never directly.
- **ADR-0017** (Bootstrap Capability Trust Root) establishes that exactly one Bootstrap Capability is created, once, during Runtime bootstrap, and "is never exposed through any public Runtime interface — no Runtime operation, at any time after bootstrap, creates or re-creates it." §17 of this document depends on this directly: restart must never touch it.
- **Governance (GOV-003, GOV-010)** and **STD-001** govern this document's own review, approval, and evidentiary process; they are not architectural inputs to its content.

## 6. Current Failure Behavior (Verified Against Implementation)

Verified directly against `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree; includes uncommitted SRP-001–007, EWO-004/005/006, and this session's actor-to-actor messaging admission pipeline). This section states facts only; §7 states the problem they create.

**No operational supervision subsystem exists.** An exhaustive search of the runtime workspace for "supervision" and "restart" (case-insensitive, all `.rs` files) returns exactly: one doc comment in `core/lifecycle-guardian/src/internal.rs:40` describing a *declared but unused* transition ("a fresh `Initializing` instance under supervision (policy, deferred..."); two mirrored comments in `runtime/src/lib.rs` (lines 1534 and 5546 at time of writing) restating the same deferral; and the `ActorId`/`ActorInstanceId` restart-identity doc comments in `common/src/lib.rs:6-13`. No supervision component, restart counter, escalation path, parent/child field, or supervision-specific audit event exists anywhere in source.

**Failure trace.** `Actor::handle()` returning `Err` is surfaced by `ExecutionCoordinator::dispatch` inside `Runtime::execute_message_capturing` (`runtime/src/lib.rs:1357`). The dispatch-rejection branch calls `Runtime::fail_active_execution` (`runtime/src/lib.rs:1552`), which:

1. calls `ExecutionCoordinator::fail` to clear its own stale `Dispatched` bookkeeping entry (never claiming a successful completion occurred);
2. calls `LifecycleGuardian::fail_execution`, transitioning the instance to `Failed` (`core/lifecycle-guardian/src/internal.rs:194-203`; legal only from `Executing`, per the transition graph at `internal.rs:47-69`);
3. delegates to `Runtime::release_and_fail_execution` (`runtime/src/lib.rs:1484`), which releases the Host Adapter execution handle and emits the mandatory `execution.failed` audit event (`runtime/src/lib.rs:190-197`; ADR-0015 governs this emission's own failure behavior).

**What remains after failure, verified:**

| State holder | Post-failure condition | Evidence |
|---|---|---|
| Actor Host | Instance still exists; mailbox and bound behavior untouched; still occupies the actor's one-live-instance slot | `core/actor-host/src/internal.rs:154-201`; nothing in the failure path above calls `terminate_instance` |
| Lifecycle Guardian | Instance truthfully `Failed` | `core/lifecycle-guardian/src/internal.rs:194-203` |
| Execution Coordinator | Clean — no stale entry | `ExecutionCoordinator::fail`'s own contract, `core/execution-coordinator/src/lib.rs:74-90` |
| Host Adapter | Balanced — handle released | `runtime/src/lib.rs:1484-1493` |
| Scheduler | Frequently still ready | See below |
| Capability Authority | Untouched — no revocation or rebinding occurs | No call to `revoke`/`bind` anywhere in the failure path |

**The Scheduler/Lifecycle Guardian gap, verified.** `Scheduler::remove` (`services/scheduler/src/lib.rs:48`) is called by Runtime in exactly one place: `Runtime::terminate_actor_instance` (`runtime/src/lib.rs:747`). It is never called from any `fail_*` path. Separately, `Runtime::schedule_next_message` (`runtime/src/lib.rs:1021-1034`) re-marks an instance Scheduler-ready **before** the message about to be executed is even attempted, whenever that instance's mailbox still holds further work (`dequeued.mailbox_has_more`). Consequence, proven by an existing test (`repeated_steps_after_handler_errors_do_not_panic_and_eventually_reach_idle`, `runtime/src/lib.rs:5685-5718`): with two messages queued for a permanently failing actor, the *second* message is dequeued and *also* fails — the Scheduler proposes the instance again, `execute_message_capturing` immediately rejects at its `validate_transition(_, Executing)` guard (Failed has no edge to Executing), another `execution.failed` is audited, and the message is silently, permanently discarded. This repeats once per queued message until the mailbox is exhausted, with **no retry, no dead-letter, and no halt** — confirmed exactly as the completed architecture review (supplied with this task) concluded, and independently reproduced here against source.

**The declared, unused restart edge.** `is_legal_transition` (`core/lifecycle-guardian/src/internal.rs:47-69`) includes `(Failed, Initializing)` as a legal transition, with the doc comment at line 40 naming it "a fresh `Initializing` instance under supervision (policy, deferred..." A grep of the entire `runtime` crate confirms this edge is exercised only inside Lifecycle Guardian's own unit tests (`core/lifecycle-guardian/src/lib.rs`, via `set_state_for_testing`) — never by any Runtime orchestration code.

**Identity contract, verified.** `common/src/lib.rs:6-13`: `ActorId` "Persists across suspension, resumption, and restart"; `ActorInstanceId` "Changes across restarts even when the owning actor's logical identity does not." This matches `ARCH-002 §7` exactly (§5 above) and is not a new fact this document introduces.

**Capability-binding key, verified.** `CapabilityAuthorityImpl.bindings: HashMap<ActorId, HashSet<CapabilityId>>` (`core/capability-authority/src/internal.rs:127`) — bindings key on logical `ActorId`, never on `ActorInstanceId`.

**Mailbox key, verified.** `ActorHostImpl.mailboxes: HashMap<ActorInstanceId, Vec<Message>>` (`core/actor-host/src/internal.rs:89`) — mailboxes key on per-instantiation `ActorInstanceId`; `terminate_instance` (`internal.rs:183-193`) unconditionally drops the mailbox "without leaking its isolated state, including its mailbox and any retained messages." No mailbox-transfer operation exists anywhere in Actor Host's public or internal surface.

**Absence of parent/child relationships, verified.** `ActorId`, `ActorInstanceId`, `ExecutionContext`, and `Message` (`common/src/lib.rs`) carry no parent, owner, or supervisor field. No such concept exists in any crate.

**Audit events, verified.** Exactly eleven event constructors exist in `runtime/src/lib.rs` (lines 91-222): `runtime.started`, `runtime.shutdown`, `actor.created`, `actor.terminated`, `message.rejected`, `message.admitted`, `capability.issued`, `execution.completed`, `execution.failed`, `actor.suspended`, `actor.restored`. None represents a supervision decision, a restart, or an escalation.

**Interaction with the current actor-to-actor messaging admission pipeline.** This session's own addition (`runtime/src/lib.rs`: `admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`) admits messages an actor's own `Actor::handle()` returns, resolving authority only from that actor's currently bound, currently valid capabilities (`resolve_emitted_message_authority`, `runtime/src/lib.rs:863`), and setting causation to the truthful, Runtime-established triggering-message id, never an actor's self-declared claim (`process_emitted_messages`, `runtime/src/lib.rs:934`). This is directly relevant here: it is the only existing channel by which an actor's own output becomes Runtime-admitted work, and it depends on the actor having *already succeeded* (`Actor::handle()` returning `Ok`) — a failed handler returns `Err`, carrying no messages, and this pipeline is never reached for it (proven by `handler_failure_never_reaches_emitted_message_processing`, `runtime/src/lib.rs`). §11 relies on this directly.

## 7. Problem Statement

Given §6's verified facts, three concrete problems exist:

1. **A failed actor instance is not contained.** It remains fully resident in Actor Host, frequently remains Scheduler-eligible, and silently discards its remaining queued work one message at a time, each producing a truthful but uncoordinated `execution.failed` audit event — with no observer that recognizes "this instance has failed and should not keep being offered work."
2. **A declared restart mechanism has no owner.** The `Failed → Initializing` transition is legal at the Lifecycle Guardian layer and named in ARCH-002's own diagram, but no component decides *when* to use it, no component *drives* it, and no component tracks *how many times* it has already been used for a given actor.
3. **No escalation, hierarchy, or supervision-specific audit trail exists**, so a repeatedly failing actor has no path to a decision beyond "keep silently discarding its own mailbox forever."

This document resolves all three by introducing exactly one new architectural participant (§10) and assigning existing components no new responsibility beyond what they already, individually, own (§11).

## 8. Architectural Principles

The following principles govern every decision in this document and are each a direct application of existing authority (§5):

- **Ownership remains with the responsible component.** No decision in this document transfers, absorbs, or duplicates any existing component's ARCH-002 §6 responsibility.
- **Runtime composes; it does not gain a second decision-maker for the same fact.** Per ADR-0016 Rule 1/Rule 2, every cross-component sequence introduced by this document is Runtime-mediated; no new direct peer-interaction path is created.
- **Policy and mechanism remain separated**, exactly as ARCH-002 §5's mechanism/policy line already requires of Scheduler ("dispatch order... policy" vs. "dispatch mechanism... trusted") — applied here to restart: *whether/when* to restart is policy (owned by the new component, §10); *that a restart transition is legal* and *how identity/capability/mailbox behave* under it are mechanism, already fixed by existing components and by ARCH-002 §7/§13/§14/§15.
- **Only a genuine actor-execution fault is treated as restart-eligible** (§12). Restarting cannot repair an authorization gap, a provisioning mistake, a backpressure condition, or a Runtime-integrity failure — these require different responses, never a uniform restart reflex.
- **Truthfulness over convenience** (extending ADR-0015's own precedent): no audit record may claim a restart completed before the replacement instance is genuinely live, and no lifecycle state may be held or represented as other than what it currently, genuinely is.
- **Additive, not retroactive.** Supervision is opt-in per actor; no existing actor, test, or demonstration in the current workspace is required to gain a supervisor for this architecture to be valid (§16).

## 9. Supervision Component Placement

### 9.1 Selected architecture

A new, narrow, Runtime-composed service component is introduced: **Supervisor**. It is positioned architecturally parallel to Scheduler (`ARCH-002 §6`'s "Replaceable services" table) — a service the Runtime composes and calls through a defined interface, not a Trusted Core component, and not a new constitutional concept. Supervisor owns supervision *policy* and supervision-specific *state* only (§11); it directly controls no other component (§11.2).

### 9.2 Why not elsewhere — reasoning preserved

| Candidate | Verdict | Reasoning |
|---|---|---|
| **Runtime itself** | Rejected as sole home | Every existing Runtime method is policy-free composition (§8; ADR-0016 Rule 1 assigns Runtime *interaction accountability*, never decision authority over what a component decides). Embedding restart-limit or escalation policy directly into `Runtime`'s own methods would be the first internal policy decision this crate has made, breaking a pattern held consistently across every existing Runtime method (`submit_message`, `execute_message_capturing`, and every method §6 of ARCH-002 catalogues). |
| **Execution Coordinator** | Rejected | Its ARCH-002 §6 responsibility is "Constructs Execution Context; performs dispatch mechanics; enforces one-owner execution" — one execution's mechanics, not what happens *after* a failed one. It has no path to Actor Host (cannot create a replacement instance) or Capability Authority (cannot confirm capability continuity), per ADR-0016 Rule 2. |
| **Lifecycle Guardian** | Rejected as sole home; retains its own role unchanged | Its ARCH-002 §6 responsibility explicitly excludes policy: "Prohibited from: Deciding *when* to suspend/restart (policy, deferred)." It correctly continues to own *whether* `Failed → Initializing` is legal (§6, `is_legal_transition`) and continues to have no path to Actor Host or Capability Authority (ADR-0016 Rule 2) — it cannot itself create a replacement instance or confirm capability continuity, and must not be made to. |
| **Actor Host** | Rejected | Its ARCH-002 §6 responsibility is identity, isolation, and mailbox mechanics, exercised unconditionally on caller instruction (`define`, `create_instance`, `terminate_instance` take no policy input today). It is correctly the *executor* of instance replacement (§14), never the *decider*. |
| **Scheduler** | Rejected | Explicitly, by its own contract, unaware of actor lifecycle: `services/scheduler/src/lib.rs:14-23` states it "holds no dependency on `synapse-capability-authority`, `synapse-actor-host`, or `synapse-lifecycle-guardian`" and "never reaches into their internal state." Giving it supervision awareness would violate this contract directly (§13 formalizes the corresponding invariant). |
| **Capability Authority** | Rejected | Owns capability lifecycle only; has no concept of instances, execution, or failure. Its `ActorId`-keyed binding map is *load-bearing* for restart (§17) but that is a property Supervisor depends on, not a responsibility Capability Authority gains. |
| **Message Gateway** | Rejected | Stateless envelope/structural-authority validation (`core/message-gateway/src/lib.rs:1-47`); has no actor-lifecycle awareness by design and must not gain any. |
| **Ordinary actors** ("supervisor actors") | Rejected for this architecture | See §13. |
| **A completely new Runtime component** | **Selected** | The only option consistent with ARCH-002 §6's own component-table method: one narrow responsibility per component, reached only through Runtime-mediated coordination (ADR-0016). |

## 10. Responsibility Boundaries

### 10.1 Supervisor owns

- supervision registration/adoption of a logical actor (`ActorId`) — opt-in, per §16;
- parent/child supervision relationships, keyed by `ActorId` (§16);
- per-actor failure history (that failures occurred and how many; not message content);
- restart accounting (how many restarts have been attempted for a given `ActorId`);
- the restart/replace/escalate/stop/ignore decision for an observed actor-execution failure (§12);
- escalation routing through supervision ancestry (§18);
- its own supervision-specific policy state.

### 10.2 Supervisor does not own, and MUST NOT directly perform

Supervisor MUST NOT directly:

- terminate an actor instance;
- create a replacement instance;
- mutate Lifecycle Guardian's tracked state;
- enqueue, dequeue, or otherwise manipulate a mailbox;
- mark an actor Scheduler-ready or otherwise touch Scheduler's tracked state;
- bind, issue, or revoke a capability;
- dispatch actor execution;
- admit a message (through `submit_message`, `admit_message`, or any other admission path);
- emit an audit event through any channel other than the one every other component already uses (Runtime → Audit Emitter);
- communicate with an actor through any channel other than the existing message/mailbox/capability pipeline (it does not gain a second, privileged one — §13).

Every one of these remains exactly where ARCH-002 §6 already assigns it: Lifecycle Guardian (state legality), Actor Host (instances, mailboxes, behavior), Capability Authority (capability lifecycle), Execution Coordinator (dispatch mechanics), Scheduler (ready-order), Message Gateway (admission), Audit Emitter (emission). Runtime alone reaches all of them (ADR-0016 Rule 1); Supervisor reaches none of them directly (ADR-0016 Rule 2, extended to this new participant) — Supervisor returns or records a decision, and **Runtime** executes it through the existing component boundaries.

### 10.3 Component responsibility table, extended

| Component | Responsibility (unchanged from ARCH-002 §6) | New responsibility from this document |
|---|---|---|
| Capability Authority | Validates, binds, attenuates, delegates, revokes capabilities | None. Its `ActorId`-keyed binding already makes restart capability-continuous (§17). |
| Actor Host | Actor identity, instance/mailbox/behavior ownership | None. Executes `terminate_instance`/`create_instance_with_behavior` exactly as any caller already can, on Runtime's instruction. |
| Message Gateway | Envelope, structural send-authority, admission | None. |
| Execution Coordinator | Execution-context construction, dispatch, completion, cleanup | None. Its existing `fail` cleanup (§6) is unaffected and unduplicated. |
| Lifecycle Guardian | Legal transition enforcement | None new — the already-declared `(Failed, Initializing)` edge is finally exercised, by Runtime, on Supervisor's authority. |
| Scheduler | Ready-order among ready actors | None — remains lifecycle-unaware (§13). |
| Audit Emitter | Unbypassable emission | None new beyond the new event categories §19 identifies (an implementation-phase, not architectural, addition). |
| **Supervisor (new)** | Supervision policy, failure history, restart accounting, parent/child registry, escalation decision | — |
| Runtime | Sole cross-component composer (ADR-0016 Rule 1) | Additionally sequences Supervisor into the existing failure path (§11, §18) and executes its decisions through the existing component boundaries — no new decision authority of its own. |

## 11. Failure Taxonomy

Not every failure is an ordinary actor-restart trigger. Restarting cannot repair an authorization gap, a provisioning mistake, a backpressure condition, or a Runtime-integrity fault — only a fault in the actor's *own executing logic*.

### 11.1 Actor execution failure

An error returned from `Actor::handle()` (`runtime/src/lib.rs:1357`, `ExecutionCoordinator::dispatch` rejection branch). **This is the sole ordinary local-supervision trigger.** It reflects the actor's own logic, not its surrounding infrastructure, and it is the only failure type this document routes to Supervisor at all.

### 11.2 Lifecycle sequencing rejection

An illegal lifecycle transition (e.g., a second concurrent execution attempt, or a transition attempted from `Terminated`) — `IllegalTransition`, read-only rejection, no state mutation (`core/lifecycle-guardian/src/internal.rs:109-119`). This reflects a caller/Runtime sequencing mistake, never an actor fault. **Not restart-eligible;** must not reach Supervisor as an actor-failure signal.

### 11.3 Admission failure

Invalid envelope; structural send-authority mismatch; capability denial, revocation, or expiry; ambiguous authority (this session's own `RuntimeError::AmbiguousAuthority`, `common/src/lib.rs`); unavailable destination; mailbox overflow (`ARCH-002 §16`'s Failure Model table: each of these is "Reject request only," never an actor-instance fault). **Not restart-eligible.** Restarting the actor changes nothing about a missing capability binding, a full downstream mailbox, or a destination that does not exist.

### 11.4 Scheduler or Runtime infrastructure failure

A future fallible Scheduler operation (`Scheduler::mark_ready`/`select_next` are `Result`-typed today for exactly this future-proofing reason, though currently infallible under the FIFO implementation — `services/scheduler/src/lib.rs`), or a mandatory audit emission itself failing (ADR-0015). Per `ARCH-002 §16`: "Runtime component failure (a trusted-core mechanism itself fails) — Threatens Runtime integrity — fail-stop for the affected scope." **Must escalate above ordinary per-actor supervision entirely** (§18) — this is a Runtime/operator-level concern, on the same basis `RuntimeState::Failed` already exists as a state distinct from ordinary actor-level `Failed`, "so operators and audit consumers can distinguish a clean shutdown from a trusted-core integrity halt" (`ARCH-002 §15`).

### 11.5 Summary

| Failure class | §11 reference | Restart-eligible? | Why |
|---|---|---|---|
| Actor execution failure | 11.1 | **Yes** | Actor's own logic faulted; a fresh incarnation may succeed where the failed one did not. |
| Lifecycle sequencing rejection | 11.2 | No | Caller/Runtime sequencing mistake; no actor state change occurred to restart from. |
| Admission failure (envelope, authority, capability, overflow, destination) | 11.3 | No | Reflects configuration, provisioning, or backpressure — a restart does not change authorization or capacity. |
| Scheduler/Runtime infrastructure failure | 11.4 | No — escalates above the actor level | Threatens Runtime integrity; per-actor restart cannot address a trusted-core fault. |

## 12. Restart Identity Semantics

Stated normatively, directly from already-fixed authority (§5):

```text
Restarted logical actor:
  same ActorId
  new ActorInstanceId
```

- `ActorId` is the stable logical identity: "Persists across suspension, resumption, and restart" (`common/src/lib.rs:6-7`; `ARCH-002 §7`).
- `ActorInstanceId` identifies one concrete incarnation: "Changes across restarts even when the owning actor's logical identity does not" (`common/src/lib.rs:11-13`; `ARCH-002 §7`).
- Restart is **replacement of an incarnation**, never reuse of the failed one. The failed `ActorInstanceId` is terminated (Actor Host, §14); a new `ActorInstanceId` is created for the same `ActorId`.
- Supervision relationships (§16) attach to `ActorId`, never to `ActorInstanceId` — a supervised logical actor remains supervised across any number of restarts.
- Capability bindings attach to `ActorId` (`core/capability-authority/src/internal.rs:127`) and therefore remain applicable to the replacement incarnation automatically, without a new ambient grant (§17).
- Execution state and Lifecycle Guardian state do **not** carry over as instance state: the replacement instance begins its lifecycle exactly as any freshly created instance does — absent from Lifecycle Guardian's tracked map, therefore `Idle` by that crate's own stated default (`core/lifecycle-guardian/src/internal.rs:88-96`, "an instance absent from the tracked map is treated as `Idle`... it has already, structurally, passed through the Defined → Initializing → Idle prefix").
- This document redefines no identity semantics; it states what `ARCH-002 §7` already fixes and builds restart mechanics consistently with it.

## 13. Failed-Instance Scheduling Semantics

**Requirement.** Once an actor instance enters `Failed`, it MUST no longer remain dispatch-eligible. Its remaining queued messages MUST NOT continue to be silently drained into repeated, individually truthful but collectively uncoordinated `execution.failed` events (§6, §7).

**Architectural requirement, not a mechanism.** This document does not prescribe *how* dispatch-eligibility is withdrawn (no API is proposed). It requires that:

- withdrawing dispatch-eligibility for a `Failed` instance is coordinated by **Runtime**, at the same point Runtime already learns of the failure (immediately after the existing `fail_active_execution`/`LifecycleGuardian::fail_execution` sequence, §6) — never by Scheduler discovering the fact on its own;
- **Scheduler remains lifecycle-unaware.** It continues to hold no dependency on Lifecycle Guardian, Actor Host, or Capability Authority (`services/scheduler/src/lib.rs:14-23`, unchanged) — this document does not ask Scheduler to learn what `Failed` means; Runtime instead ensures a `Failed` instance is no longer marked ready, using whatever mechanism Scheduler's own existing interface already offers for exactly this purpose (`Scheduler::remove`, already used by `terminate_actor_instance`, `runtime/src/lib.rs:747`) or an architecturally equivalent one;
- **Lifecycle Guardian does not directly manipulate Scheduler.** ADR-0016 Rule 2 is preserved without exception: the two components remain mutually unaware; only Runtime connects them, exactly as it already connects every other Trusted-Core-to-Trusted-Core sequence.

```text
Actor::handle() returns Err
        |
        v
Execution cleanup (ExecutionCoordinator::fail, existing)
        |
        v
Lifecycle Guardian: Executing -> Failed (existing)
        |
        v
execution.failed audited (existing)
        |
        v
Runtime withdraws this instance's Scheduler dispatch-eligibility     <-- NEW requirement
   (Scheduler itself remains lifecycle-unaware; Runtime alone acts,
    through Scheduler's own existing ready-set interface)
        |
        v
Runtime informs Supervisor (§18)
```

## 14. Mailbox and Pending-Message Semantics Across Restart

**Current architectural truth, stated plainly:**

- Mailboxes are instance-owned, keyed by `ActorInstanceId` (`core/actor-host/src/internal.rs:89`).
- Restart produces a new instance, with a new `ActorInstanceId` (§12) and therefore a new, empty mailbox (`ActorHostImpl::create_instance_internal`, `core/actor-host/src/internal.rs:131-152`, always inserts a fresh empty `Vec`).
- `terminate_instance` unconditionally discards the failed instance's mailbox "without leaking its isolated state, including its mailbox and any retained messages" (`core/actor-host/src/internal.rs:183-193`).
- No operation anywhere in Actor Host's public or internal surface transfers one instance's mailbox contents to another instance.
- **Pending messages are therefore not preserved by current restart mechanics.** This document does not, and must not, imply otherwise.

**Classification: intentionally deferred, not rejected.** `ARCH-002 §13` already states "a new instance under supervision receives a new mailbox, with in-flight message handling deferred to Lifecycle Architecture" — this document is that Lifecycle Architecture, and it explicitly defers mailbox-content preservation rather than resolving it, because resolving it requires capability Actor Host does not currently expose (no way to read, drain, or transfer a mailbox's contents from outside Actor Host — the closest existing operation, `mailbox_message_ids`, is `#[cfg(test)]`-only).

Preserving queued work across restart in a future milestone may require, in some combination:

- an explicit mailbox drain-and-transfer operation, newly added to Actor Host's own interface;
- durable mailbox ownership (a persistence concern, out of scope here — §4);
- dead-letter handling for messages that cannot be transferred;
- a redelivery policy;
- replay or persistence semantics for messages admitted before a restart.

None of these is designed here. For the initial local supervision model, this architecture MUST NOT be read as implying that mailbox contents survive restart — an actor's own pending, unprocessed messages at the moment of failure are lost under the mechanics this document authorizes, unless and until a future, separately authorized milestone adds explicit transfer.

## 15. Parent/Child Supervision Model

A logical supervision hierarchy, keyed by `ActorId`:

- **Supervision relationships represent logical actors, not individual incarnations.** A child relationship attaches to a child's `ActorId`; when that child is restarted (§12), the replacement incarnation remains under the identical supervision relationship without any re-registration act, because the relationship was never keyed to the incarnation that failed.
- **Supervision is initially optional and additive.** An actor with no registered supervisor may exist — nothing in this architecture requires every existing actor, test, or demonstration in the current workspace to acquire one (§8, §16 non-retroactivity).
- **Assignment, reassignment, and removal are explicit, Runtime-mediated operations.** No actor may unilaterally assign or change its own supervisor — this would create exactly the second privileged channel §13 (of this document) and ADR-0016 Rule 2 prohibit. Supervision-relationship changes are acts Runtime performs on explicit instruction (an embedder's, or a higher-level supervisor's own decision relayed through Runtime), never something an actor's own `Actor::handle()` output can trigger through the ordinary message-admission pipeline (§6's closing note).
- **Hierarchy depth is not artificially limited to one level.** A supervisor may itself be a supervised logical actor with its own parent, enabling escalation (§18) to propagate arbitrarily far up a tree.
- **Cycles are architecturally prohibited.** A logical actor must never appear as its own ancestor in the supervision hierarchy — this is a structural invariant Supervisor's own registration/adoption operation must enforce, on the same "must not corrupt trusted state" basis ARCH-002 §16 already requires generally ("An actor's failure MUST NOT corrupt trusted Runtime state").
- **A logical actor has at most one active supervising parent** under this architecture. Multiple simultaneous supervisors for one logical actor are not introduced here; a future architecture may explicitly replace this constraint, but this document does not anticipate or design for it.

```text
Escalation hierarchy (logical, by ActorId — no direct component calls implied)

        Supervisor-of-root (ActorId: coordinator)
              |
     -------------------------
     |                       |
Supervised child A      Supervised child B
(ActorId: worker-a)     (ActorId: worker-b)

If worker-a's restart budget under child-A's own supervision is
exhausted, escalation travels upward along this ActorId-keyed edge —
never as a direct call from worker-a's Lifecycle Guardian entry to
"coordinator"; Runtime mediates every step, exactly as every other
cross-component sequence in this Runtime already is (ADR-0016).
```

## 16. Escalation Model

Distinguishing the possible responses to failure, conceptually, without algorithms:

- **Retrying the same message** — not addressed by this document; §11 already establishes only the actor's own execution fault (not a per-message outcome) is restart-eligible, and message-level retry is explicitly out of scope (§4).
- **Restarting an actor** — Supervisor's decision (§10.1); Runtime executes it via Actor Host's existing `terminate_instance`/`create_instance_with_behavior` operations (§14), same `ActorId`, new `ActorInstanceId` (§12).
- **Replacing an actor instance** — mechanically identical to restart (§12); the distinction (if any) is purely in Supervisor's own bookkeeping of *why* (first failure vs. Nth), not in what Runtime/Actor Host actually do.
- **Stopping the logical actor** — Runtime terminates the instance (Actor Host) and does not create a replacement; the `ActorId` remains defined but has no live instance, exactly as any other explicitly stopped actor today (`terminate_actor_instance`, `runtime/src/lib.rs:735`).
- **Escalating failure to its supervisor** — when a supervised actor's own restart accounting (owned by Supervisor, §10.1) is exhausted, and that actor itself has a parent in the hierarchy (§15), the failure is reported one level up, through Runtime, exactly as the original failure was first reported to this level's own Supervisor relationship (§18 diagram, §13 of the original review).
- **Escalating beyond the supervision root** — a logical actor with no parent (the root of its own tree, or an actor with no registered supervisor at all) whose own restart accounting is exhausted has nowhere further local to escalate to; this becomes an operator/embedder-visible condition (surfaced via the audit trail, §19), not a silent stop.
- **Failing or degrading the Runtime itself** — reserved exclusively for §11.4's infrastructure-failure class; ordinary actor supervision, however deep its hierarchy, MUST NOT conceal or substitute for a genuine Runtime-integrity failure. `RuntimeState::Failed` remains the distinct, correct representation of that condition (`ARCH-002 §15`), never reachable through ordinary per-actor restart exhaustion.

**Ownership summary:**

| Concern | Owner |
|---|---|
| Restart accounting (has this `ActorId` failed before, how many times) | Supervisor |
| Lifecycle-transition legality (is `Failed → Initializing` legal now) | Lifecycle Guardian (unchanged) |
| Replacement mechanics (terminate old, create new instance) | Actor Host (unchanged), invoked by Runtime |
| Cross-component orchestration of the whole sequence | Runtime (unchanged authority, ADR-0016 Rule 1) |
| Deciding when a restart budget is exhausted and escalation is warranted | Supervisor |
| Deciding what happens when escalation reaches the root | Operator/embedder, via the audit trail (§19) — not silently absorbed anywhere in the Runtime |

No numeric threshold, time window, backoff algorithm, or default strategy value is defined by this document (§4).

## 17. Capability Semantics Across Restart

- Capability bindings are keyed by `ActorId` (`core/capability-authority/src/internal.rs:127`), not by `ActorInstanceId`. A replacement instance sharing its predecessor's `ActorId` therefore **ordinarily inherits the logical actor's currently valid bound capabilities automatically, without a new ambient grant** — this is a direct, existing consequence of how Capability Authority already stores bindings, not a new mechanism this document introduces.
- **Revoked or expired capabilities do not become valid because of restart.** `CapabilityAuthority::validate` (`core/capability-authority/src/lib.rs`) is checked identically regardless of which incarnation of `ActorId` presents or holds a capability; restart performs no revalidation shortcut and bypasses no check.
- **Restart does not bypass Capability Authority.** Every capability the replacement instance uses is validated exactly as any other capability use is (`ARCH-002 §9`, §11 step 12).
- **Supervisor does not grant authority.** It has no path to Capability Authority (§10.2) and holds no capability-issuing power of its own.
- **Runtime does not invent fallback authority.** If a restarted actor's previously bound capabilities have since been revoked or narrowed, the replacement instance operates with exactly what Capability Authority's own registry currently states — nothing is silently reinstated or substituted.
- **Capability narrowing after repeated failure** (e.g., "restart with fewer permissions after N failures") may be considered by a later policy milestone, but if so, it is performed through Capability Authority's own existing operations (`bind`/`revoke`, exercised by Runtime on Supervisor's decision — exactly as this session's own `Runtime::bind_capability` already demonstrates as the ordinary, ADR-0016-consistent mechanism), never through Supervisor holding independent authority state of its own.
- **Bootstrap authority MUST NOT become a supervision fallback.** ADR-0017 fixes that exactly one Bootstrap Capability is created, once, at Runtime bootstrap, and "is never exposed through any public Runtime interface — no Runtime operation, at any time after bootstrap, creates or re-creates it." Restart is unconditionally excluded from any exemption to this rule; no supervision-triggered code path may retrieve, reconstruct, or substitute the Bootstrap Capability as a source of replacement-instance authority. There is no separate "Bootstrap Authority" component to appeal to in the current architecture (§9.2's table; ADR-0017 assigns the Bootstrap Capability's ownership to Capability Authority alone) — this document assumes none exists, consistent with the current implementation.

## 18. Audit Requirements

Supervision decisions and outcomes require truthful audit representation, extending — never contradicting — the minimum audit-event set `ARCH-002 §18` already establishes ("Runtime start and shutdown; actor creation and termination; capability issuance, delegation, attenuation, and revocation; failed capability validation; message rejection and mailbox admission; execution start, completion, and failure; suspension and restoration; externally observable provider or tool invocation").

At minimum, audit visibility is required for concepts equivalent to:

- actor failure observed (by Supervisor, distinct from the existing `execution.failed` event which records the execution fact, not the supervision observation of it);
- a supervision decision made (restart / replace / escalate / stop / ignore);
- restart initiated;
- the failed incarnation's termination (reusing the existing `actor.terminated` event's own truthful basis, `runtime/src/lib.rs:129-136`, applied to a supervision-initiated termination);
- the replacement incarnation's creation (reusing the existing `actor.created` event's own truthful basis, `runtime/src/lib.rs:114-121`);
- restart completed;
- restart refused or exhausted;
- failure escalated (and to what, per §15's hierarchy);
- an actor stopped under supervision;
- a supervision relationship established, changed, or removed, where security or operational traceability requires it (§15).

This document does not define exact event-type identifiers or a new `AuditEvent` field layout — STD-001 does not require normative event identifiers at the architecture level, and `AuditEvent`'s own shape (`common/src/lib.rs`: `event_type`, `actor`, `capability`, `message`) is an implementation concern for the eventual Engineering Work Order, not fixed here.

**Ordering requirement — truth over convenience:**

```text
1. Execution failure and its existing cleanup are recorded
   (execution.failed — already implemented, unchanged by this document)
        |
        v
2. Supervisor observes the completed failure state
   (only after step 1 has genuinely finished — the same "validation
    occurs before side effects" discipline ARCH-003 §6 already applies
    to every existing Runtime flow)
        |
        v
3. A supervision decision is recorded
        |
        v
4. Runtime performs the authorized action
   (through the existing component boundaries — §10.2)
        |
        v
5. The result is recorded
```

**No audit record may claim a restart completed before the replacement incarnation is genuinely live** — the same truthfulness discipline `ARCH-002 §12` already requires of `Executing` state generally ("Lifecycle state MUST represent the Runtime's actual, current condition... MUST NOT be held, extended, or otherwise made to outlive the genuine fact it represents"), applied here to "restart completed."

## 19. Interaction With Existing Components

```text
Runtime
├── Trusted Core components
│     Capability Authority, Actor Host, Message Gateway,
│     Execution Coordinator, Lifecycle Guardian, Audit Emitter,
│     Host Adapter
│     (ARCH-002 §6 — unchanged by this document)
├── Scheduler
│     (ARCH-002 §6 replaceable service — unchanged; remains
│      lifecycle-unaware, §13)
└── Supervisor  (NEW — this document)
      owns: restart policy, per-ActorId failure history,
            parent/child registry (§15), escalation decisions
      reached only via Runtime; reaches nothing else directly
      (mirrors Scheduler's own "no dependency on Actor Host /
       Capability Authority / Lifecycle Guardian" contract,
       services/scheduler/src/lib.rs:14-23)
```

- **Lifecycle Guardian:** no new method gains a Supervisor caller directly. Runtime continues to be the only caller of `validate_transition`/`fail_execution`/`begin_execution`/`complete_execution`, now additionally informing Supervisor of the outcome and, on Supervisor's authorized decision, finally driving the already-legal `Failed → Initializing` edge — never Supervisor invoking Lifecycle Guardian itself.
- **Scheduler:** unaffected in its own responsibility; gains no new caller and no new awareness. §13's withdrawal-of-dispatch-eligibility requirement is satisfied entirely through Runtime calling Scheduler's own existing interface, exactly as `terminate_actor_instance` already does.
- **Execution Coordinator:** unaffected; its existing `fail` cleanup already runs, unmodified, before Supervisor is ever informed (§18 ordering).
- **Actor Host:** gains no new *automatic* behavior. Supervisor (via Runtime) invokes exactly the same `terminate_instance`/`create_instance_with_behavior` any existing caller already can. Mailbox transfer (§14) is explicitly not one of these — it does not exist as an operation to invoke.
- **Capability Authority:** gains no new responsibility. Restart's capability continuity is a free consequence of the existing `ActorId`-keyed binding map (§17), not a new Capability Authority behavior.
- **Message Gateway:** no interaction at all — it has no actor-lifecycle awareness by design and gains none here.
- **Audit Emitter:** gains new *event categories* to emit (§18) but no new emission mechanism — Runtime continues to be the sole caller, exactly as ADR-0016/§18's ordering already requires.

No responsibility is duplicated: every capability Supervisor's decisions depend on (state-transition legality, instance creation/destruction, capability continuity, dispatch-eligibility) already has exactly one owner; Supervisor's own contribution is strictly the decision of *when* to invoke them, never a parallel implementation of *what* they do.

## 20. Lifecycle and Failure-Flow Diagrams

**Actor failure flow (extends `ARCH-002 §25`'s own Actor Lifecycle diagram)**

```text
Actor::handle() returns error
        |
        v
Execution cleanup (ExecutionCoordinator::fail — existing)
        |
        v
Lifecycle Guardian: Executing -> Failed (existing)
        |
        v
Failure audit: execution.failed (existing)
        |
        v
Runtime withdraws Scheduler dispatch-eligibility for this instance (§13, NEW)
        |
        v
Runtime informs Supervisor (NEW)
        |
        v
Supervisor decision: restart | replace | escalate | stop | ignore (NEW, §12/§16)
        |
        v
Runtime executes the decision through existing component owners:
  restart/replace -> Actor Host: terminate_instance(old), create_instance_with_behavior(new)
                     (same ActorId, new ActorInstanceId -- §12)
  escalate        -> Runtime informs the parent-ActorId's own Supervisor relationship (§15/§18)
  stop            -> Actor Host: terminate_instance(old), no replacement
  ignore          -> no further action beyond what already occurred above
        |
        v
Result recorded (audit, §19)
```

**Restart identity**

```text
Logical ActorId A
    |
    +-- Instance A::instance#1 --(handler Err)--> Failed --(supervision decision: restart)-->
    |                                                                                        |
    +-- Instance A::instance#2 (new ActorInstanceId, same ActorId A) <-----------------------+
          created via Actor Host, capability bindings for A already apply (§17),
          new empty mailbox (§14), fresh Lifecycle Guardian entry (absent -> Idle, §12)
```

**Failed-scheduling boundary**

```text
                     Scheduler ready set
                  (per-instance boolean, FIFO,
                   lifecycle-unaware -- unchanged)
                            |
        instance genuinely  |  instance now Failed at
        has more mail       |  the Lifecycle Guardian layer
                            |
                            v
     Runtime is the only party that reconciles the two:
     it withdraws this instance's ready-set membership at
     the moment it learns of the failure (§13) -- Scheduler
     never queries Lifecycle Guardian, and Lifecycle Guardian
     never calls Scheduler, directly or otherwise (ADR-0016).
```

## 21. Future Compatibility

| Future milestone | Compatible? | Constraint this document imposes to keep it so |
|---|---|---|
| Timers | Yes | Timer-driven restart backoff would be Supervisor-internal state; introduces no change to Lifecycle Guardian's or Scheduler's own contract. |
| Persistence | Partial — pre-existing gap, not introduced here | If Runtime-level (not just actor-level) restart is ever required, Supervisor's own failure-history and hierarchy state must be representable as plain, serializable data keyed by `ActorId` — never by an ephemeral `ActorInstanceId`, in-memory handle, or process/thread identity — so it remains meaningful across a Runtime restart, not merely an actor restart. |
| Durable mailboxes | Directly anticipated, not designed | §14 defers exactly the operation a durable-mailbox milestone would need to add (explicit transfer). This document must not be read as having foreclosed it by assuming mailbox loss is acceptable forever — only that it is acceptable for this milestone. |
| Workflow engine | Likely realized as supervised actors | Requires the parent/child model (§15) to nest arbitrarily, which this document does not artificially limit. |
| Effect system | Compatible by precedent | This session's own "actor emissions are admission requests, not sent facts" discipline (§6, closing note) is the model a future effect system should follow; a restart action is analogously something Runtime performs on Supervisor's authorization, never something Supervisor does by reaching into Actor Host itself (§10.2). |
| Remote messaging / distributed runtimes / clustering | Highest-risk area; kept open, not solved | Keying every supervision relationship by `ActorId` (already stable, already location-transparent per `ARCH-002 §7`) rather than by instance, process, or host identity is the one design choice here that keeps a future distributed supervisor viable — a remote node is then just "where this `ActorId` currently lives," not a new identity scheme. Supervision decisions (§10.1) remain conceptually separable from the mechanics that execute them (§10.2, always via Runtime) specifically so that a future distributed implementation could replace *how* a decision is executed without rewriting *how* it is made. |

**Forward constraints, stated explicitly (binding on any future extension of this document):**

- Supervisor state MUST remain conceptually representable as serializable logical data keyed by `ActorId`.
- Supervision MUST NOT depend on direct in-process calls between components (ADR-0016 Rule 2 extends to Supervisor without exception).
- Parent/child identity MUST NOT be tied to memory addresses, ephemeral host handles, or `HostExecutionHandle` values (which `ARCH-002 §10` already requires to remain opaque).
- Supervision decisions MUST remain separable from the mechanics that execute them.
- A future distributed-supervision design MUST be able to replace local execution of a decision without rewriting the policy model this document establishes.
- Mailbox preservation MUST remain an explicit future feature (§14), never an accidental assumption baked into this milestone's restart mechanics.

## 22. Risks and Deferred Decisions

- **Mailbox-content loss on restart (§14)** is a real, disclosed limitation of the current mechanism, not merely a documentation gap — any embedder relying on this architecture before a durable-mailbox milestone lands must expect queued work to be lost when a supervised actor restarts.
- **New audit-event categories are required (§18) but not named here** — the next implementation milestone must define them; until then, this architecture is not fully auditable in the sense ARCH-002 §18 requires for its own existing minimum set.
- **`AuditEvent`'s current shape may be insufficient** (`common/src/lib.rs`: `event_type`, `actor`, `capability`, `message` — no restart-count or escalation-target field) to carry everything §18 requires; this is flagged as an implementation-phase question, not resolved here.
- **Cycle prevention (§15) is stated as a requirement, not designed** — the concrete mechanism by which Supervisor's registration/adoption operation detects and rejects a cycle is deferred to implementation.
- **Single-parent constraint (§15)** is adopted for this milestone only; whether SynapseOS ever needs multiple simultaneous supervisors per logical actor is an open question this document does not resolve.
- **Numeric restart policy (§4, §16)** — thresholds, backoff, and windows — is entirely undecided by this document and is the most likely source of a future ADR or policy-level RFC.
- **Scheduler-eligibility withdrawal mechanism (§13)** is stated as a requirement only; whether it reuses `Scheduler::remove` verbatim or an architecturally equivalent new interface is an implementation decision this document does not make.

## 23. Normative Architecture Decisions

- Runtime MUST remain the sole cross-component composer (ADR-0016 Rule 1, extended to Supervisor without exception).
- Supervisor MUST NOT directly access Actor Host, Lifecycle Guardian, Scheduler, Capability Authority, Execution Coordinator, Message Gateway, or Audit Emitter (§10.2).
- Supervisor MUST own supervision policy and supervision-specific history (§10.1); no other component may duplicate this ownership.
- Lifecycle Guardian MUST remain the sole owner of lifecycle-transition legality, including the `Failed → Initializing` edge (§9.2, §13).
- Actor Host MUST remain the sole owner of actor-instance creation, termination, behavior, and mailbox state (§9.2, §14).
- Restart MUST retain `ActorId` and create a new `ActorInstanceId` (§12).
- A `Failed` incarnation MUST become non-dispatchable (§13).
- Scheduler MUST remain lifecycle-unaware (§13, §19).
- Capability continuity across restart MUST remain governed exclusively by Capability Authority (§17).
- Bootstrap authority (the Bootstrap Capability, ADR-0017) MUST NOT be used as fallback restart authority under any circumstance (§17).
- Supervision relationships MUST attach to logical actor identity (`ActorId`), never to `ActorInstanceId` (§12, §15).
- Supervision-trigger classification MUST distinguish genuine actor-execution failure from admission, authorization, backpressure, lifecycle-sequencing, and Runtime-infrastructure failure (§11); only actor-execution failure is ordinarily restart-eligible.
- Initial supervision MUST be additive and MAY remain optional per actor (§8, §15).
- This architecture MUST NOT claim preservation of pending mailbox messages across restart (§14).
- Mailbox preservation MUST remain explicitly deferred, not silently assumed (§14, §21).
- Supervision decisions and outcomes MUST be truthfully auditable, in the ordering §18 establishes.
- Ordinary actor messaging MUST NOT become a privileged supervision-control channel (§13 of the completed review; restated at §10.2, §15).

## 24. References

Internal:

- GOV-003 — Governance Model
- GOV-010 — Decision Framework
- GOV-004 — Engineering Principles
- STD-001 — Documentation Standards
- ARCH-000 — Introduction
- ARCH-001 — Constitutional Architecture (§3, §5.4)
- ARCH-002 — Runtime Architecture (§5, §6, §7, §9, §11, §12, §13, §14, §15, §16, §18, §20, §23, §25)
- ARCH-003 — Runtime Integration Architecture (§5, §6, §17)
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root

Source evidence (verified directly, §6):

- `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree):
  - `common/src/lib.rs` (identity contracts, `RuntimeError`)
  - `core/actor-host/src/internal.rs` (instance/mailbox mechanics)
  - `core/capability-authority/src/internal.rs`, `src/lib.rs` (binding key, `bound_capabilities`)
  - `core/lifecycle-guardian/src/internal.rs` (`is_legal_transition`, `Failed` handling)
  - `core/execution-coordinator/src/lib.rs` (`fail` cleanup contract)
  - `core/host-adapter/src/internal.rs` (execution-handle accounting)
  - `services/scheduler/src/lib.rs` (lifecycle-unaware contract)
  - `runtime/src/lib.rs` (`execute_message_capturing`, `fail_execution`, `release_and_fail_execution`, `fail_active_execution`, `schedule_next_message`, `terminate_actor_instance`, audit-event constructors, `admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`, `bind_capability`, and test `repeated_steps_after_handler_errors_do_not_panic_and_eventually_reach_idle`)
- Completed architecture review — "SynapseOS Architecture Review — Local Actor Supervision, Failure Escalation, and Restart Ownership" (this conversation session; analytical basis for this document, independently re-verified against source per §6, not cited as a substitute for that verification).

## 25. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-14 | Denver Jacobs | Initial Draft. Establishes the Local Actor Supervision Architecture: Supervisor component placement, responsibility boundaries, failure taxonomy, restart identity semantics, failed-instance scheduling semantics, mailbox semantics across restart, parent/child supervision model, escalation model, capability semantics across restart, audit requirements, and future-compatibility constraints. Completes the deferral recorded in ARCH-002 §15 and §23 ("Lifecycle Architecture: Supervision/restart policy, backoff strategy"). |

## 26. Approval Status

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-14 |
| Technical Review | TBD | Pending | |
| Approval Authority | Chief Architect (vacant); Founder (interim) | Pending | |
