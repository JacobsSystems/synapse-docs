---
document_id: ARCH-005
title: Temporal Runtime Architecture
project: SynapseOS
specification: SynapseOS — timer ownership, delayed execution, time observation, and timer lifecycle within the local Runtime process, realizing the temporal architecture the completed "Temporal Runtime, Timers, Delayed Execution, and Time Ownership" review recommended
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
    - ARCH-001 (Draft — constitutional foundation; §6, "time observation" named among foundational Runtime mechanisms outside the Actor model)
    - ARCH-002 (Draft — Runtime architecture; §5, §6, §8, §13, §15, §16, §18, §19, §23 directly extended by this document)
    - ARCH-003 (Draft — Runtime integration status, current baseline evidence)
    - ARCH-004 (Draft — Local Actor Supervision Architecture; component-placement and identity-model precedent this document follows directly, §7, §9)
  rfcs: None
  adrs:
    - ADR-0015 (Draft; Approved disposition recorded separately per STD-001 §31 — Audit Emitter Failure Semantics)
    - ADR-0016 (Draft; Approved disposition recorded separately per STD-001 §31 — Trusted Core Interaction Rule)
    - ADR-0017 (Draft — Bootstrap Capability Trust Root)
  roadmap: None
  research: None
  operational: None
  reports:
    - ER-007 (Local Actor Supervision — Engineering Report; current verified implementation baseline this document's evidence is checked against)
  source_artifacts:
    - synapse-runtime @ 5ccc7f9083a71adc6ee704b2322a701935765679 (working tree, including uncommitted local actor supervision implementation)
  review: "SynapseOS Architecture Review — Temporal Runtime, Timers, Delayed Execution, and Time Ownership" (this session; analytical basis for this document, independently re-verified against source per §7, not cited as a substitute for that verification)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-005 — Temporal Runtime Architecture

*Filename pattern: `ARCH-005-Temporal-Runtime-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Document Control and Status

| Field | Value |
|---|---|
| Document ID | ARCH-005 |
| Title | Temporal Runtime Architecture |
| Version | 0.1.0 |
| Status | **Draft** |
| Author | Denver Jacobs |
| Approval authority | Chief Architect (Class B, per GOV-010 §5), vacant; Founder (interim) |
| Created | 2026-07-14 |
| Classification | Public |

This document is Draft. It has not been reviewed or approved, and nothing in it should be read as operative or binding until it completes the same governance process ARCH-002, ARCH-003, and ARCH-004 are themselves subject to (GOV-003, GOV-010). This document introduces no implementation and authorizes none; it establishes architecture only, to be realized by a future Engineering Work Order (STD-001 §46) once approved.

This document is derived directly from, and requires no departure from, the completed "SynapseOS Architecture Review — Temporal Runtime, Timers, Delayed Execution, and Time Ownership" conducted earlier in this engineering effort. Every architectural claim about the current runtime that review made has been independently re-verified against `synapse-runtime` source before being restated here (§7); nothing in this document is merely restated from that review without such verification.

## 2. Purpose

This document defines the authoritative architecture for **time within the local SynapseOS Runtime process**: which component observes the passage of time, which component owns a registered future point of interest ("a timer"), how a timer's eventual firing becomes ordinary Runtime work, how a timer's identity survives (or does not survive) actor restart and stop, and what must be truthfully audited throughout. It resolves the open architectural question ARCH-004 §4 and §21 explicitly named and declined to resolve ("timers or scheduled/delayed restart" — out of scope there; "Timer-driven restart backoff would be Supervisor-internal state" — future-compatible, not designed there), by establishing the general-purpose temporal mechanism any future policy (including a future Supervisor restart-backoff policy) would be built upon — without itself defining that policy.

This document does not select numeric policy (specific delays, timeouts, backoff values), does not define implementation APIs or Rust types, and does not authorize implementation. It is architecture: what must be true, who owns what, and why — consistent with ARCH-002's own stated method (`ARCH-002 §1`: "precise enough that independent engineering teams could implement recognizably conformant runtimes without identical internal code").

## 3. Scope

**In scope:** the architectural placement of a new temporal-observation responsibility within the existing Runtime component model; the identity model governing what a timer registration belongs to; the lifecycle of a timer registration from creation to its terminal state; the mechanism by which a fired timer becomes deliverable work (never a new execution path); the interaction of timer delivery with the existing single admission pipeline, capability model, mailbox model, and audit model; the interaction (or, more precisely, the deliberate absence of direct interaction) between timer registrations and local actor supervision; the failure taxonomy for timer-related failures; and the clock model (monotonic versus wall-clock) this architecture requires.

**Out of scope:** cron-style calendar scheduling; distributed clocks or distributed timer coordination; durable or persistent timers surviving a Runtime-process restart; a workflow engine or generalized effect-scheduling system; message retry, redelivery, or dead-letter mechanisms of any kind; backoff algorithms or numeric threshold policy of any kind (including any future Supervisor restart-backoff policy this architecture may eventually support — that policy remains ARCH-004 §4/§16's own deferred concern); deadline propagation across a causal chain of messages; time-zone handling; real-time execution guarantees; cluster or multi-host scheduling; any public Rust API, struct, trait, or function signature. See §5 for the complete non-goals statement and §22 for the future-compatibility boundary.

**Explicitly distinguished from the pre-existing, unimplemented `Message.deadline`/`ExecutionContext.deadline` fields.** `common/src/lib.rs` already declares an opaque `deadline: Option<u64>` field on both `Message` and `ExecutionContext` (ARCH-002 §8, §10), and ARCH-002 §13 already states "a message whose deadline lapses before dispatch MUST be treated as expired, never delivered" — a requirement independently verified (§7) to have **no implementation anywhere in the current workspace**: the field is copied through unread and unchecked. This document does not close that gap, does not redesign those fields, and does not require closing it. It is named here only so that a future milestone that does address deadline enforcement can be verified against this document's own clock and audit model (§20, §21) for consistency, rather than inventing a second, incompatible temporal model.

## 4. Non-Goals

This document does not define, and takes no position on:

- cron scheduling, calendar scheduling, or time-zone handling;
- distributed clocks or any cross-host clock synchronization mechanism;
- persistent or durable timers surviving a Runtime-process restart (as distinct from an actor restart, §11, §12);
- a workflow engine or generalized effect-scheduling system;
- message retry, redelivery, or acknowledgement protocols;
- dead-letter queues or dead-letter storage;
- backoff algorithms, jitter, or any numeric timing threshold (including any future Supervisor restart-backoff policy — ARCH-004 §4, §16, unaffected and unresolved by this document);
- deadline propagation across a causal chain of messages, or any change to the existing, unimplemented `Message.deadline`/`ExecutionContext.deadline` fields (§3);
- real-time execution guarantees of any kind;
- cluster coordination or multi-host scheduling;
- any Rust struct, trait, enum, method signature, function name, or field layout;
- any new Trusted Core component (ARCH-002 §5–§6 is unchanged; see §9);
- any new lifecycle state on the existing Actor, Message, Capability-binding, or Runtime state machines beyond ARCH-002 §15's existing set;
- any new constitutional guarantee beyond ARCH-001's four (non-forgery, integrity, enforcement-at-invocation, revocation-state enforcement).

## 5. Existing Architectural Context

This document amends no prior authority. It extends, and is bound by, the following without redefinition:

- **ARCH-000** established SynapseOS's whole-system introduction; this document inherits it without restatement.
- **ARCH-001** fixes the four constitutional guarantees and, directly relevant here, its §6 Constitutional Laws already name **"time observation"** as one of "a fixed, named set of foundational Runtime mechanisms — capability enforcement, audit emission, scheduling, time observation, transport, and bootstrap — [that] are necessarily outside the Actor model, because each is either structurally prior to actor execution or required to remain unbypassable by it." This document is the first to give that already-named mechanism an architectural home (§9). It introduces no new constitutional guarantee and weakens none.
- **ARCH-002** remains the sole authority for Trusted Core component responsibility, ownership, and prohibition (`ARCH-002 §5`, §6), the actor and Runtime lifecycle state models (`ARCH-002 §15`), the mailbox model (`ARCH-002 §13`), the minimum audit-event set (`ARCH-002 §18`), and the Extension and Replaceability Model (`ARCH-002 §19`). This document amends none of it. It specifically extends, without altering:
  - the Trusted Runtime Core mechanism table (`ARCH-002 §5`), which independent re-verification (§7) confirms contains no clock, timer, or time-observation mechanism among its twelve entries — the same absence that already places "scheduling *order*" outside the trusted core applies, by identical reasoning, to time observation (§9);
  - "Scheduler, persistence, distributed transport, provider adapters, knowledge services, storage, monitoring, policy engines, and resource accounting are all Runtime services" (`ARCH-002 §19`) — the category this document's new component joins;
  - the already-declared, currently unimplemented `Message`/`ExecutionContext` `deadline` fields (`ARCH-002 §8`, §10) and the mailbox deadline-expiry requirement (`ARCH-002 §13`), neither altered nor closed by this document (§3);
  - the already-declared `Message: ... {Admitted → Expired}` and `Capability binding: ... {Revoked | Expired}` lifecycle terminal states (`ARCH-002 §15`), neither of which this document's own timer lifecycle (§12) reuses or redefines — a timer registration's own lifecycle is a distinct state machine, owned by the new component this document introduces, not a reuse of either existing one.
- **ARCH-003** records the current, verified implementation baseline and integration status of the seven Trusted Core components (`ARCH-003 §5`, §17). This document treats ARCH-003's baseline as evidence for §7 below, not as authority.
- **ARCH-004** established the Supervisor precedent this document follows directly: a new, narrow, Runtime-composed replaceable service, positioned parallel to Scheduler, reached only through Runtime (ADR-0016 Rule 2 extended to a new participant), owning policy and state but no direct component access. ARCH-004 §4 explicitly excluded "timers or scheduled/delayed restart" from its own scope, and ARCH-004 §21 named "Timers" as a compatible-but-undesigned future milestone ("Timer-driven restart backoff would be Supervisor-internal state; introduces no change to Lifecycle Guardian's or Scheduler's own contract"). This document is that future milestone's general mechanism — it establishes the Temporal Runtime component ARCH-004's own future restart-backoff policy, should one ever be authorized, would be built upon, without itself defining that policy (§4). This document does not redesign ARCH-004; §11 and §18 below extend its `ActorId`/`ActorInstanceId` identity model and its Supervisor isolation precedent respectively, without altering either.
- **ADR-0015** (Audit Emitter Failure Semantics) governs every audit-emission failure behavior this document assumes for temporal audit obligations (§21): a mandatory audit emission that fails causes the *reporting* operation to fail, without rollback of already-committed component-level state.
- **ADR-0016** (Trusted Core Interaction Rule) is the specific authority this document depends on most directly. Its two rules — "the Runtime is the sole architectural entity accountable for establishing and coordinating Trusted Core interactions" (Rule 1) and "Trusted Core components must not independently establish or own direct peer interaction paths" (Rule 2) — are extended, not amended, by this document to a new participant (Temporal Runtime, §9): it connects to every other component exactly as any other Runtime-composed service would, through the Runtime, never directly, and never to Supervisor either (§18).
- **ADR-0017** (Bootstrap Capability Trust Root) establishes that exactly one Bootstrap Capability is created, once, during Runtime bootstrap, and is never exposed through any public Runtime interface. §14 of this document depends on this directly: a fired timer's resulting message must earn admission through the ordinary capability-validation path and must never be granted, or fall back to, bootstrap-level authority.
- **ER-007** (Local Actor Supervision — Engineering Report) records the currently completed, independently reviewed implementation state this document's own evidence (§7) is checked against — `synapse-runtime` @ the same commit, with the same accumulated uncommitted milestones (capability messaging, bounded mailboxes, bootstrap grants, local actor supervision) present.
- **Governance (GOV-003, GOV-010)** and **STD-001** govern this document's own review, approval, and evidentiary process; they are not architectural inputs to its content.

## 6. Architectural Principles

This document is governed by, and introduces no exception to, the following principles already established by the documents in §5:

- **Mechanism/policy separation** (ARCH-001 §9, applied concretely by ARCH-002 §5): a component that only proposes, and whose proposal is always re-validated by an existing trusted mechanism before it has any effect, does not itself require trust-boundary protection. This is the test §9 applies to place Temporal Runtime outside the Trusted Core.
- **Sole-composer discipline** (ADR-0016 Rule 1): the Runtime alone connects any two components; no component establishes or owns a direct peer-interaction path (ADR-0016 Rule 2). This document extends that discipline to Temporal Runtime exactly as ARCH-004 extended it to Supervisor.
- **Single admission pipeline** (established by the approved actor-to-actor messaging architecture review and unmodified since): every message, regardless of origin, passes through one identical Runtime-owned admission pipeline before becoming eligible for scheduling. This document treats that invariant as constitutional to the Runtime's own truthfulness guarantees and introduces no second path (§13).
- **Truthful, non-anticipatory audit** (ARCH-004 §18's own discipline, itself derived from ADR-0015): no audit record may claim an outcome before it has genuinely occurred.
- **Identity stability across restart** (ARCH-002 §7, extended by ARCH-004 §12): a logical `ActorId` is stable across suspension, resumption, and restart; an `ActorInstanceId` is not. This document's timer-identity decision (§11) is a direct application of this already-established principle, not a new one.

## 7. Runtime Evidence Verified

Before authoring, the completed review's conclusions were independently re-verified directly against `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree). No contradiction was found between the review and the current implementation; every item below was re-confirmed by direct source inspection in this task, not assumed from the review's own text.

| Claim | Verification | Result |
|---|---|---|
| Scheduling ownership | `services/scheduler/src/lib.rs`: `Scheduler` trait (`mark_ready`, `remove`, `select_next`), FIFO ready set, no dependency on any other component's crate | Confirmed — Scheduler decides dispatch order only, unaware of lifecycle, capability, or time |
| Readiness ownership | Same file: `mark_ready`/`select_next`, a per-instance boolean | Confirmed |
| Execution ownership | `core/execution-coordinator/src/internal.rs`: `construct_context`, `dispatch` | Confirmed unchanged; no clock interaction |
| Absence of timers | Exhaustive re-grep (`timer|clock|Instant|SystemTime|std::time`) across every `.rs` file in the workspace | Confirmed — zero matches outside comments disclaiming their absence (e.g. `runtime/src/lib.rs:1252`: "No thread, timer, sleep... returns control to the caller") |
| Absence of delayed execution | `Scheduler::select_next` only ever returns already-ready work; `Runtime::step()`/`run_until_idle` are synchronous, bounded, caller-driven | Confirmed — no mechanism anywhere holds work back until a future moment |
| `deadline` field semantics | `common/src/lib.rs:88,144`; `core/execution-coordinator/src/internal.rs:118` | Confirmed opaque `Option<u64>`, copied through, never read or compared anywhere |
| Capability expiry semantics | `core/capability-authority/src/internal.rs:204-212`: `mark_expired_for_testing`, `#[cfg(test)]` only | Confirmed — no production lease/clock-triggered expiry mechanism exists |
| Audit model | `common/src/lib.rs:157-162`: `AuditEvent { event_type, actor, capability, message }` | Confirmed unchanged since ER-007; no timestamp field |
| Runtime composition | `runtime/src/lib.rs:585-590`: `struct Runtime { state, core, scheduler, supervisor }` | Confirmed — exactly the two replaceable services (Scheduler, Supervisor) alongside the Trusted Core, no third as of this commit |
| Scheduler composition | Same | Confirmed present, unchanged |
| Supervisor composition | Same | Confirmed present, unchanged, per ER-007 |
| Admission pipeline | `runtime/src/lib.rs:895-917`, `admit_message`: envelope → send-authority → capability validity → `live_instance` → Message Gateway `admit` → Actor Host `enqueue` → Scheduler `mark_ready` | Confirmed — one pipeline, no deadline check present anywhere in it |
| `ActorId` semantics | `common/src/lib.rs:6-9` | Confirmed: stable logical identity, persists across restart |
| `ActorInstanceId` semantics | `common/src/lib.rs:11-15` | Confirmed: per-incarnation identity, changes across restart |

No contradiction between the completed review and the current implementation was found. This document proceeds on that basis.

## 8. Temporal Runtime

**What it is.** Temporal Runtime is the name this document gives to the architectural responsibility of time observation (ARCH-001 §6) within the local Runtime process: recording that a future point of interest has been registered, and truthfully proposing to the Runtime the moment that point has arrived. It is not a scheduler of actor work (that remains Scheduler's own, unchanged responsibility, §9.2) and not a policy engine (it carries no backoff, retry, or numeric-threshold logic of any kind, §4).

**What it owns.** Timer registrations, keyed by `ActorId` (§11); each registration's own lifecycle state (§12); and the single, narrow act of observing monotonic time (§20) to determine when a registration's condition is satisfied.

**What it must never become.** A second execution or admission path (§13); a component with direct knowledge of Supervisor, Scheduler, Lifecycle Guardian, Actor Host, Execution Coordinator, Capability Authority, or Message Gateway (§10); or a source of authority — a fired timer confers no capability and bypasses no validation (§14).

## 9. Component Placement

### 9.1 Selected architecture

A new, narrow, Runtime-composed service component is introduced: **Temporal Runtime**, implemented by a new crate, `synapse-timer`. It is positioned architecturally parallel to Scheduler and Supervisor (ARCH-002 §6's "Replaceable services" table; ARCH-004 §9) — a service the Runtime composes and calls through a defined interface, not a Trusted Core component, and not a new constitutional concept.

**Why time observation is foundational, Runtime-owned, replaceable, and non-authoritative:**

- **Foundational** — ARCH-001 §6 already names "time observation" among the fixed set of mechanisms "necessarily outside the Actor model," structurally prior to, or required to remain unbypassable by, actor execution — the same standing already accorded to scheduling and audit emission.
- **Runtime-owned** — no actor, and no existing Trusted Core component, currently has (or should gain) any awareness of time; only the Runtime composes services with cross-cutting, non-actor responsibility (ARCH-002 §19).
- **Replaceable** — Temporal Runtime's proposals are always re-validated by the existing admission pipeline before they have any effect (§13, §14), on the identical "stale proposals are re-validated, not trusted" basis ARCH-002 §11 already establishes for Scheduler's own proposals. A component whose output is never trusted as-is does not require trust-boundary protection (§6).
- **Non-authoritative** — Temporal Runtime mints no capability, enforces no authority, and validates nothing itself (§14); it only proposes that a moment has arrived.

### 9.2 Why not elsewhere — reasoning preserved

| Candidate | Verdict | Reasoning |
|---|---|---|
| **Trusted Core (as an eighth component, or folded into an existing one)** | Rejected | The Trusted Runtime Core mechanism table (`ARCH-002 §5`) maps exactly twelve mechanisms to ARCH-001's four guarantees; independent re-verification (§7) confirms none concerns time observation. Time observation decides *when* to propose, never *whether* something is authorized — the same distinction that already places "scheduling *order*" outside the trusted core (`ARCH-002 §5`: "no mechanism outside this table is trusted core"). |
| **Scheduler** | Rejected | Scheduler's own contract (`services/scheduler/src/lib.rs:14-23`, independently re-verified §7) holds no dependency on any other component and decides dispatch order among *already-ready* work only. Folding time observation into it would make Scheduler responsible for deciding *whether* something has become ready due to elapsed time, not merely *which* ready thing runs next — a materially different, lifecycle-and-time-aware responsibility Scheduler's own established isolation (ARCH-002 §5, §13; ARCH-004 §9.2) already forbids. |
| **Lifecycle Guardian** | Rejected | Its ARCH-002 §6 responsibility is legal-transition enforcement, explicitly excluding policy ("Deciding *when* to suspend/restart (policy, deferred)"). Time observation is exactly this kind of "when," not a transition-legality question. |
| **Execution Coordinator** | Rejected | Its responsibility is one execution's mechanics (context construction, dispatch, completion) — it has no path to compose a new, independent, cross-cutting service, and giving it clock awareness would couple execution mechanics to a concern (time) it has never needed. |
| **Actor Host** | Rejected | Owns identity, isolation, and mailbox mechanics only, exercised unconditionally on caller instruction. It has no basis to also decide when a future condition is satisfied. |
| **Capability Authority** | Rejected | Owns capability lifecycle only. Time is named in `ConstraintSet`'s own doc comment as one *constraint category* a capability's constraints may reference, but `ConstraintSet`'s internal representation remains entirely deferred (`common/src/lib.rs:35-40`) and no such mechanism exists; Capability Authority remains unchanged by this document (§14). |
| **Ordinary actors ("timer actors")** | Rejected for this architecture | Would require every actor needing delayed work to independently reimplement time observation, or to depend on an ambient, un-auditable external clock — precisely the "no generic, unrestricted plugin interface" ARCH-002 §19 already forbids for extensions requiring bypass of a bounded interface. |
| **A completely new Runtime component** | **Selected** | The only option consistent with ARCH-002 §6's own component-table method: one narrow responsibility per component, reached only through Runtime-mediated coordination (ADR-0016), on the same precedent ARCH-004 already established for Supervisor. |

## 10. Responsibility Boundaries

### 10.1 Temporal Runtime owns

- timer registrations, keyed by `ActorId` (§11) — never `ActorInstanceId`;
- each registration's own lifecycle state (§12);
- the single act of observing monotonic time (§20) and truthfully proposing, to the Runtime, that a registration's condition is now satisfied.

### 10.2 Temporal Runtime does not own, and MUST NOT directly perform

Temporal Runtime MUST NOT directly:

- construct, admit, or deliver a `Message`;
- enqueue, dequeue, or otherwise manipulate a mailbox;
- mark an actor Scheduler-ready or otherwise touch Scheduler's tracked state;
- dispatch actor execution, or invoke `Actor::handle` in any way;
- mutate Lifecycle Guardian's tracked state;
- bind, issue, validate, or revoke a capability;
- read or query Supervisor's own tracked state, or be queried by it;
- emit an audit event through any channel other than the one every other Runtime-composed service already uses (Runtime → Audit Emitter);
- observe wall-clock time as the basis for any firing or ordering decision (§20).

Every one of these remains exactly where ARCH-002 §6 and ARCH-004 §10 already assign it: Message Gateway and Actor Host (admission, mailbox), Scheduler (ready-order), Lifecycle Guardian (state legality), Capability Authority (capability lifecycle), Execution Coordinator (dispatch mechanics), Supervisor (failure policy), Audit Emitter (emission). Runtime alone reaches all of them (ADR-0016 Rule 1); Temporal Runtime reaches none of them directly (ADR-0016 Rule 2, extended to this new participant) — Temporal Runtime proposes that a moment has arrived, and **Runtime** constructs the resulting message and submits it through the existing, unmodified admission pipeline (§13).

### 10.3 Component responsibility table, extended

| Component | Responsibility (unchanged from ARCH-002 §6 / ARCH-004 §10) | New responsibility from this document |
|---|---|---|
| Capability Authority | Validates, binds, attenuates, delegates, revokes capabilities | None. A fired timer's resulting message is validated exactly as any other, at fire time only (§14). |
| Actor Host | Actor identity, instance/mailbox/behavior ownership | None. |
| Message Gateway | Envelope, structural send-authority, admission | None. A timer-generated message is architecturally identical to any other once submitted (§13). |
| Execution Coordinator | Execution-context construction, dispatch, completion, cleanup | None. |
| Lifecycle Guardian | Legal transition enforcement | None. |
| Scheduler | Ready-order among ready actors | None — remains time-unaware, exactly as it remains lifecycle-unaware (ARCH-004 §13). |
| Supervisor | Supervision policy, failure history, restart accounting | None. No interaction with Temporal Runtime exists or is required (§18). |
| Audit Emitter | Unbypassable emission | None new beyond the new event categories §21 identifies (an implementation-phase, not architectural, addition — the same pattern ARCH-004 §19 already used). |
| **Temporal Runtime (new)** | Timer registration, lifecycle, monotonic time observation, firing proposals | — |
| Runtime | Sole cross-component composer (ADR-0016 Rule 1) | Additionally composes Temporal Runtime, constructs the message a fired timer implies, and submits it through the existing admission pipeline (§13) — no new decision authority of its own. |

## 11. Timer Identity

**Selected architecture: a timer registration belongs to `ActorId`, never to `ActorInstanceId`.**

This is a direct application of the identity model ARCH-002 §7 already fixes and ARCH-004 §12 already applies to supervision, extended here without alteration:

- **Restart.** ARCH-004 §12 establishes that a restarted actor retains its `ActorId` and receives a new `ActorInstanceId`. A timer keyed to `ActorInstanceId` would be silently orphaned by every restart — a truthfulness violation by omission, since nothing would signal that the registration had become meaningless. A timer keyed to `ActorId` is unaffected: it continues to refer to "whichever instance is currently live for this logical actor," the same property capability bindings already have (`core/capability-authority/src/internal.rs`, `ActorId`-keyed).
- **Replacement.** Because capability continuity across restart is already a free consequence of `ActorId`-keyed binding storage (ARCH-004 §17), timer continuity across restart is architected to be an equally free consequence of `ActorId`-keyed registration storage — no coordination between Temporal Runtime and Supervisor is required (§18).
- **Stop / Termination.** A logical actor that has Stopped has no live instance for its `ActorId`. A timer registration for that `ActorId` is not thereby rendered meaningless in the same silent way an instance-keyed one would be after restart — its eventual firing simply fails at ordinary admission (`live_instance` resolution failure, `RuntimeError::UnknownTarget`), the same outcome any other message directed at a stopped actor already receives (§16). This document nonetheless requires Runtime to remove the registration proactively at Stop (§12), rather than leaving it to fail silently at every future fire attempt — see §12 for why this is resolved normatively rather than left to implementation.
- **Supervision escalation.** Escalation (ARCH-004 §16) never touches a live instance — it is bookkeeping against the parent's own restart accounting only. A timer registration keyed to the escalating child's `ActorId` is entirely unaffected by an escalation targeting its parent.
- **Logical actor identity.** `ActorId`-keying means a timer registration's meaning is exactly "this logical actor, whichever instance currently realizes it" — never "this specific incarnation." This is the same locational and temporal stability `ActorId` already provides for capability bindings and supervision relationships, applied here for the third time in this architecture's evolution, not a new identity concept.

**Normative statement:** a restarted logical actor retains its timer registrations because it retains its `ActorId`; nothing about restart requires, or triggers, any timer-specific action (§12).

## 12. Timer Lifecycle

A timer registration's lifecycle is a distinct state machine, owned entirely by Temporal Runtime, and is not a reuse of the existing Message, Capability-binding, or Actor lifecycle machines (ARCH-002 §15) — it governs the registration record itself, not any of those other constitutional or Runtime concepts.

**States:**

- **Registered** — the initial state, entered the instant Runtime accepts a registration request on Temporal Runtime's behalf. Carries the target `ActorId` and whatever condition determines firing.
- **Waiting** — the ordinary steady state between registration and firing (or cancellation, or expiry). A registration may be considered to enter this state immediately upon Registered, or Registered and Waiting may be treated as the same observable state — this document does not require them to be distinguishable, only that neither claims firing has occurred before it truthfully has.
- **Fired** — entered the instant Temporal Runtime truthfully observes that the registration's condition is satisfied. This is the point at which Runtime is informed and constructs the resulting `Message` (§13). Reaching `Fired` does not itself imply the resulting message was successfully delivered — delivery is a separate, subsequent admission outcome (§16), on the same "the observation and the consequence are truthfully distinct facts" discipline ARCH-004 §18/§19 already applies to `supervision.failure_observed` versus the decision that follows it.
- **Cancelled** — a terminal state, entered only through an explicit, Runtime-mediated cancellation request. A registration in `Cancelled` never fires, regardless of whether its condition would otherwise have been satisfied.
- **Expired** — a terminal state, entered when a registration's own condition can no longer be satisfied without ever having fired (for example: its target `ActorId` denotes a logical actor already Terminated at the time the registration was made, or some other bounded, implementation-defined validity-window rule places it here rather than in `Waiting`). This is a distinct terminal state from `Fired`-then-discarded (§16): `Expired` means the registration itself never reached its firing condition; a `Fired` registration whose resulting message subsequently fails admission is a separate outcome (`Discarded`, below), because the registration itself did do its job truthfully.
- **Discarded** — a terminal state, entered when a `Fired` registration's resulting message fails ordinary admission (capability denial, unknown destination, mailbox overflow, or any other admission-time rejection, §16). This is architecturally distinguished from `Expired` precisely because the timer itself behaved correctly; the failure occurred one layer downstream, in admission, not in time observation.
- **Completed** — a terminal state, entered when a `Fired` registration's resulting message is successfully admitted. This is the only state in which the temporal responsibility can be said to have been fully and successfully discharged.

**Legal transitions:**

```text
Registered -> Waiting -> Fired -> {Discarded | Completed}
Registered -> Cancelled
Waiting    -> Cancelled
Registered -> Expired
Waiting    -> Expired
```

**Illegal transitions include:** any transition out of `Cancelled`, `Expired`, `Discarded`, or `Completed` (all four are terminal); `Fired` directly to `Cancelled` (a firing that has already truthfully occurred cannot be retroactively un-fired — cancellation only ever prevents a firing that has not yet happened); `Registered` or `Waiting` directly to `Discarded` or `Completed` (both require having genuinely passed through `Fired` first — no admission outcome may be claimed for a registration that never fired, the identical truthful-ordering discipline ARCH-004 §18 already requires for supervision's own restart-completion event).

**Ownership:** Temporal Runtime owns the state itself and the transition `Registered/Waiting → Fired`. Runtime owns the transitions `Fired → {Discarded | Completed}`, since those depend on the admission pipeline's own outcome — a fact only Runtime, as the pipeline's sole composer, truthfully knows at the moment it occurs. `Cancelled` is entered only on an explicit request Runtime mediates on a caller's behalf. `Expired` is entered by Temporal Runtime itself, applying whatever bounded validity rule governs a registration that can no longer be satisfied.

## 13. Delayed Execution

**Normative requirement: timers never execute actors, and never dispatch. A fired timer produces an ordinary `Message`, submitted by Runtime through the single, existing, unmodified admission pipeline (§7) — never a bypass, never a second path.**

This is the single most load-bearing requirement in this document, and is treated as a **constitutional Runtime invariant**, not merely a design preference: the approved actor-to-actor messaging architecture already established that "every message, regardless of origin, must pass through one identical Runtime-owned admission pipeline before becoming eligible for scheduling," and ARCH-002 §19 already forbids any extension that would require bypassing execution ownership, message integrity, or admission — "an extension that required bypassing any of these would not be a bounded extension but a new trusted-core component, subject to the same rigor as §5–§6." A Timer service that directly executed, dispatched, enqueued, or manipulated Scheduler or Lifecycle Guardian state would be exactly such an unbounded extension, and is prohibited by this document without exception.

**Explicitly prohibited, without exception:**

- direct execution of an actor's `Actor::handle` by Temporal Runtime or on Temporal Runtime's authority;
- direct dispatch through Execution Coordinator bypassing the ordinary admission-then-scheduling sequence;
- direct mailbox insertion bypassing Message Gateway's admission and Actor Host's `enqueue`;
- direct manipulation of Scheduler's ready set by Temporal Runtime;
- direct manipulation of Lifecycle Guardian's tracked state by Temporal Runtime;
- any parallel or alternative admission path, however narrow, that a fired timer's resulting message could take instead of `admit_message`.

## 14. Admission Interaction

The exact architectural flow a fired timer follows:

```text
Temporal Runtime: condition satisfied (Waiting -> Fired)
        |
        v
Runtime: timer.fired audited (§21); Message constructed
        |
        v
Runtime: submits the Message through the existing,
         single admission pipeline (§7):
           Message Gateway  -> validate_envelope, validate_send_authority
           Capability Authority -> validate(presented)      (§ this section)
           Actor Host       -> live_instance resolution, enqueue (mailbox)
           Scheduler        -> mark_ready
        |
        v
Ordinary Scheduler selection, ordinary dispatch, ordinary execution
```

**Every timer-generated message is architecturally identical to every other message once submitted to Runtime.** No special execution path exists past the point of submission — the message's origin (external caller, actor emission, or timer firing) has no bearing on how it is validated, admitted, queued, or dispatched from that point forward.

**Capability validation** occurs **only at fire time**, as part of this ordinary admission sequence — never at registration time, never cached, and never independently revalidated by Temporal Runtime itself. This preserves Capability Authority's own established rule that "bindings are queried fresh, never cached as independent truth elsewhere, avoiding staleness after revocation" (ARCH-002 §9): a capability cached at registration time could be revoked long before firing, and caching its validity would silently resurrect exactly the staleness ARCH-002 already forbids. Temporal Runtime holds no capability state of any kind, and Capability Authority is unchanged by this document (§10.3).

## 15. Mailbox Interaction

A fired timer's resulting message, once submitted, is subject to every existing mailbox rule without exception:

- **Bounded mailboxes / overflow** — the same finite mailbox capacity and `Overflow` rejection apply; no exemption exists for a timer-originated message.
- **Ordering** — the message takes its place in FIFO admission order at the moment it genuinely enters the pipeline (when `Fired`), never at registration time.
- **Causation** — see §16.
- **Audit** — the existing `message.admitted`/`message.rejected` events fire exactly as for any other message (§21).
- **Admission** — the same single pipeline (§14); no timer-specific mailbox, queue, or buffer exists anywhere in this architecture.

No bypass of any kind is permitted. Overflow remains an ordinary admission failure (§16), not a Temporal Runtime failure.

## 16. Causation

**Runtime owns causation for a timer-generated message. Temporal Runtime never self-asserts causation.**

This mirrors the existing, established rule that an actor's own self-declared sender or causation claim carries no authority (ARCH-001 §5.3; the existing `process_emitted_messages` implementation, which overwrites causation with the Runtime-known triggering message rather than trusting the emitting actor's own claim). Runtime — which alone knows, truthfully, which timer registration produced a given message — establishes:

```text
Timer registration (identity)
        |
        v
Timer fired (Runtime-observed fact, audited: timer.fired)
        |
        v
Resulting Message: causation set by Runtime to identify
the firing registration, never left to Temporal Runtime's
own unverified assertion
```

This preserves truthful causation exactly as actor-emitted messages already require: the component that produces a message's *reason for existing* is never trusted to assert that reason itself; the Runtime, as sole composer, establishes it.

## 17. Failure Semantics

| Failure | Ownership | Classification |
|---|---|---|
| Registration failure (malformed target, invalid condition) | Temporal Runtime | Rejected at registration; no lifecycle state is entered |
| Firing failure (Temporal Runtime itself malfunctions) | Temporal Runtime / Runtime-infrastructure | Not an admission or supervision failure — a Runtime-infrastructure concern, on the same "trusted-core mechanism itself fails → fail-stop for the affected scope" basis ARCH-002 §16 already applies generally |
| Capability failure (denied, revoked, expired by fire time) | Admission (Capability Authority, via the ordinary pipeline) | **Admission failure** — `Discarded` (§12), never routed to Supervisor |
| Mailbox overflow | Admission (Actor Host, via the ordinary pipeline) | **Admission failure** — `Discarded`, never routed to Supervisor |
| Unknown destination (actor never defined, or Terminated) | Admission (`live_instance` resolution) | **Admission failure** — `Discarded`, never routed to Supervisor |
| Actor stopped (Stop decision already executed) | Admission | **Admission failure** — `Discarded`, identical in kind to submitting an ordinary message to a stopped actor |
| Actor restarted (between registration and fire) | None — not a failure at all | Resolved structurally by `ActorId`-keying (§11); the replacement instance receives the message, admitted fresh |
| Timer cancelled | Temporal Runtime | Not a failure — a legitimate, explicit withdrawal (§12), audited (§21) |
| Clock failure (backwards movement) | Temporal Runtime, eliminated by construction | Not a runtime failure to route anywhere — addressed structurally by requiring a monotonic clock (§20), which cannot move backwards by definition |
| Runtime shutdown | Runtime | Governs whether pending registrations are retained or discarded across a Runtime-process lifecycle event; not resolved further here (§4 — durable/persistent timers across a Runtime-process restart are out of scope) |

**Normative statement: timer delivery failures are admission failures. They are not supervision failures.** A fired timer's resulting message that fails admission is discarded through the same, existing, unmodified admission-failure handling every other message already receives — never routed to Supervisor. This is a direct extension of EWO-007's own established invariant ("admission failures never reach supervision") to a new message origin, without modification: Supervisor's whole model (ARCH-004) concerns genuine `Actor::handle()` execution failure, reached from exactly one call site; an admission-time rejection, whatever produced the message that was rejected, never reaches that call site and never reaches Supervisor.

## 18. Supervision Interaction

**Normative statement: Temporal Runtime has no direct relationship with Supervisor. No peer interaction exists, and none is required.**

- Restart requires no timer rebinding — a timer registration was never keyed to the replaced instance in the first place (§11).
- Replacement requires no timer migration — there is nothing to migrate; the registration already refers to the logical actor, not the incarnation.
- Timer continuity across restart and replacement arises **solely** because registrations belong to `ActorId` (§11) — this is a property of the shared identity convention both components independently honor, not a coordination protocol between them.
- Neither component depends on the other's crate, calls the other's interface, or is aware the other exists — the identical isolation ADR-0016 Rule 2 already requires of Trusted Core components, extended by precedent (ARCH-004 §10.2, this document's own §10.2) to hold between the two replaceable services introduced by these two architecture documents.

This is the strongest form of preserving the truthful-runtime architecture available: correctness follows from a shared naming convention, not from any interaction that could itself fail, race, or drift out of sync.

## 19. Clock Architecture

**Normative statement: Temporal Runtime shall observe monotonic time. Monotonic observation MUST determine firing. Wall-clock time MUST NOT determine execution ordering.**

This architecture distinguishes:

- **Monotonic clock** — the sole basis for determining whether a registration's condition is satisfied and for ordering firings relative to one another. A monotonic clock cannot move backwards by definition, which eliminates the "clock moved backwards" failure mode (§17) by construction rather than by exception-handling — the same "eliminate a defect by construction" discipline EWO-007 already applied to the mailbox-drain defect.
- **Wall clock** — relevant only if some future, separately authorized milestone gives real meaning to the existing, currently inert `Message.deadline`/`ExecutionContext.deadline` fields (§3), which are external, human-meaningful timestamps by nature. This document does not require or design that; it only requires that Temporal Runtime's own firing logic never conflate the two.
- **Logical runtime time** — not architecturally required by anything this document's evidence (§7) supports. SynapseOS's current execution model is single-threaded and fully synchronous; a logical or vector clock would only become relevant under a future distributed-runtime milestone (ARCH-002 §21, §23, already deferred), and this document does not anticipate one now.
- **Test / simulated clock** — this architecture requires that *some* seam exist allowing deterministic testing of "time has passed" without introducing real wall-clock waits into test execution, consistent with the existing test suite's wholly synchronous, instantaneous character (no existing test anywhere sleeps or waits on a real clock). This document does not specify the seam's shape — that is an implementation-API question, out of scope here (§4).

**The architecture requires an abstract notion of time** — Temporal Runtime's "is it time yet" decision must be sourced from a single, substitutable notion of "now," on the same "narrowest public surface" discipline Scheduler and Supervisor already establish, never an ambient, direct call to the host's real clock scattered through Temporal Runtime's own internals. This is a requirement that a seam exist, not a definition of what that seam looks like.

## 20. Audit Architecture

The following timer-related facts require truthful audit, each realized as a distinct `event_type` string value on the **existing, unmodified** `AuditEvent` shape (`event_type`, `actor`, `capability`, `message` — `common/src/lib.rs`, independently re-verified unchanged, §7) — the same "no new field, new string values only" discipline ARCH-004 §18/§19 already established for supervision, for the identical reason (extending via new string values avoids repeating the disclosed "which parent" / "which registration" limitation a new field would otherwise require solving here):

| Concept | When truthfully emitted |
|---|---|
| Timer registered | Immediately upon successful registration |
| Timer cancelled | Immediately upon an explicit cancellation request, before the registration's state changes |
| Timer fired | The instant Temporal Runtime truthfully observes the condition satisfied — before the resulting message is constructed or admitted |
| Timer expired | The instant a registration enters `Expired` without ever having fired |
| Timer discarded | The instant a `Fired` registration's resulting message fails admission |
| Timer delivery | Realized by the existing `message.admitted`/`message.rejected` events firing for the resulting message — not a new, parallel pair, to avoid a truthfulness fork between "the timer fired" and "the message was/wasn't admitted" |
| Timer completion | The instant a `Fired` registration's resulting message is successfully admitted (`Completed`, §12) |

**Ordering discipline:** no event may claim a later state before it has genuinely been reached — `timer.fired` must never be emitted after the resulting message has already been admitted; no event may claim delivery succeeded before admission genuinely returns success. This is the identical "no audit record may claim completion before it genuinely occurs" rule ARCH-004 §18 already established for supervision, applied here without modification. This document redesigns no part of the existing audit architecture (ARCH-002 §18); it extends it exactly as ARCH-004 already did, by new event-type values on the unchanged shape.

## 21. Future Compatibility

| Future milestone | Compatible? | Constraint this document imposes to keep it so |
|---|---|---|
| Persistence / durable timers | Yes | A registration is already a discrete, `ActorId`-keyed data record (§11) — the same shape a persistence layer would need to snapshot, on the same Persistence/Restoration Service interface boundary ARCH-002 §6/§23 already establishes for everything else. |
| Workflow engine | Yes | Temporal Runtime produces an ordinary message through the ordinary pipeline (§13); a workflow engine realized as an ordinary, capability-scoped actor (ARCH-002 §19) can consume timer-fired messages exactly like any other, without any change to this document's own model. |
| Effect Runtime | Yes | Nothing in this architecture is specific to any actor's purpose; an effect-scheduling actor is an ordinary consumer of ordinary timer-fired messages. |
| Distributed Runtime / remote timers / cluster Runtime | Yes, structurally; not solved operationally | `ActorId`-keying is already location-transparent by contract (`ARCH-002 §7`); a distributed Temporal Runtime implementation is a location/transport concern, deferred exactly as Scheduler's and Actor Directory's own distributed variants already are (`ARCH-002 §21`, §23) — no identity-model change would be required. |
| AI orchestration | Yes | An orchestration actor is an ordinary consumer of ordinary timer-fired messages; this document imposes no constraint specific to any actor's application purpose. |

**Forward constraints, stated explicitly (binding on any future extension of this document):**

- Timer registration state MUST remain conceptually representable as serializable logical data keyed by `ActorId`.
- Temporal Runtime MUST NOT depend on direct in-process calls to or from Supervisor, Scheduler, Lifecycle Guardian, Actor Host, Execution Coordinator, Capability Authority, or Message Gateway (ADR-0016 Rule 2 extends to Temporal Runtime without exception).
- A fired timer's resulting message MUST continue to be submitted through whatever the single admission pipeline has become, never a path parallel to it.
- Capability validation for a timer-generated message MUST continue to occur at fire time only, never cached or performed by Temporal Runtime itself.
- Any future distributed Temporal Runtime design MUST be able to replace *where* time is observed without rewriting the identity model (`ActorId`-keying) or the admission-reuse model this document establishes.

## 22. Non-Goals (Restated as the Future-Compatibility Boundary)

Consistent with §4, this document explicitly excludes, and no section above should be read as partially designing: cron or calendar scheduling; distributed clocks; persistent or durable timers surviving a Runtime-process restart; workflow-engine implementation; effect-scheduling implementation; dead-letter queues; retries; backoff algorithms or thresholds; deadline propagation; time-zone handling; real-time execution guarantees; cluster scheduling. Each of these remains a candidate for a future, separately authorized architecture or engineering work order, built on — not requiring redesign of — the mechanism this document establishes (§21).

## 23. Normative Architecture Decisions

- Runtime MUST remain the sole cross-component composer (ADR-0016 Rule 1, extended to Temporal Runtime without exception).
- Temporal Runtime MUST remain outside the Trusted Core (§9.1) — no clock, timer, or time-observation mechanism is added to the Trusted Runtime Core mechanism table (`ARCH-002 §5`).
- Timer registrations MUST belong to `ActorId`, never to `ActorInstanceId` (§11).
- Restart MUST preserve timer registrations, as a free consequence of `ActorId`-keying — no restart-specific timer action exists or is required (§11).
- Stop / Terminated MUST cause Runtime to remove the corresponding timer registrations proactively (§12) — this is a normative decision, not left to implementation discretion, precisely because leaving it undecided would allow an indefinitely growing set of registrations whose eventual firing can only ever fail, which this document resolves now rather than deferring.
- Temporal Runtime MUST NOT execute actors, dispatch, enqueue mailboxes, or otherwise bypass the single admission pipeline under any circumstance (§13).
- Every timer-generated message MUST be submitted through the existing, unmodified admission pipeline and MUST be architecturally indistinguishable, from that point forward, from any other message (§13, §14).
- Temporal Runtime MUST NOT directly manipulate Scheduler, Lifecycle Guardian, Actor Host, Execution Coordinator, Capability Authority, or Message Gateway (§10.2).
- Capability validation for a timer-generated message MUST occur only at fire time, never at registration time, never cached, and never independently revalidated by Temporal Runtime (§14).
- Timer delivery failures MUST remain ordinary admission failures and MUST NOT be routed to Supervisor (§17).
- Timer registrations, firings, cancellations, expirations, and discards MUST be truthfully auditable, in the ordering §20 establishes, using the existing, unmodified `AuditEvent` shape.
- Monotonic time MUST determine firing and firing order; wall-clock time MUST NOT determine execution ordering (§19).
- Temporal Runtime MUST have no direct relationship with Supervisor; timer continuity across restart MUST arise solely from shared `ActorId`-keying, never coordination (§18).
- The single admission pipeline MUST remain a constitutional Runtime invariant, unmodified and unbypassed by this or any future temporal mechanism (§13).
- This architecture MUST NOT be read as authorizing any numeric timing policy, backoff, or restart-policy decision — those remain separately deferred (§4).

## 24. Risks and Deferred Decisions

- **The bounded rule governing when a registration enters `Expired` versus remaining `Waiting` indefinitely (§12)** is stated as a requirement for some rule to exist, not designed here — the concrete condition is deferred to implementation, on the same basis ARCH-004 §22 already deferred its own cycle-detection mechanism's concrete form.
- **Whether pending timer registrations survive a whole-Runtime-process shutdown (§17)** is explicitly out of scope (§4, durable/persistent timers) and is flagged, not resolved, here.
- **New audit-event categories are required (§20) but not named as concrete Rust identifiers here** — the next implementation milestone must define them, exactly as ARCH-004 §22 flagged the same open item for supervision before EWO-007 resolved it.
- **The existing, unimplemented `Message.deadline`/`ExecutionContext.deadline` fields (§3) remain unaddressed** — this document deliberately does not close that pre-existing gap, and a future milestone addressing it must be checked for consistency against this document's clock and audit model rather than assumed automatically compatible.
- **Whether Temporal Runtime requires any dependency beyond `synapse-common`** is an implementation question this document does not resolve — by the reasoning in §9–§14, no dependency on any other component's crate is architecturally required, but this is not itself a normative decision recorded in §23, since it is not yet verified against a concrete implementation the way ARCH-004's own `synapse-api` dependency was (ER-007).
- **Numeric timing policy of every kind (§4, §22)** remains entirely undecided by this document and is the most likely source of a future ADR, RFC, or Engineering Work Order — including any eventual Supervisor restart-backoff policy this mechanism could someday support (ARCH-004 §21).

## 25. References

Internal:

- GOV-003 — Governance Model
- GOV-010 — Decision Framework
- GOV-004 — Engineering Principles
- STD-001 — Documentation Standards
- ARCH-000 — Introduction
- ARCH-001 — Constitutional Architecture (§6, "time observation")
- ARCH-002 — Runtime Architecture (§5, §6, §7, §8, §9, §10, §13, §15, §16, §18, §19, §23)
- ARCH-003 — Runtime Integration Architecture (§5, §17)
- ARCH-004 — Local Actor Supervision Architecture (§4, §9, §10, §12, §16, §17, §18, §19, §21, §22)
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- ER-007 — Local Actor Supervision — Engineering Report

Source evidence (verified directly, §7):

- `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree):
  - `common/src/lib.rs` (`ActorId`, `ActorInstanceId`, `Message`, `ExecutionContext`, `AuditEvent`, `ConstraintSet`, `RuntimeError`)
  - `core/execution-coordinator/src/internal.rs` (`construct_context`, deadline pass-through)
  - `core/capability-authority/src/internal.rs` (`mark_expired_for_testing`, test-only expiry)
  - `services/scheduler/src/lib.rs` (`Scheduler` trait, FIFO ready set, lifecycle-unaware contract)
  - `services/supervisor/src/lib.rs`, `src/internal.rs` (component-placement and identity-model precedent)
  - `runtime/src/lib.rs` (`Runtime` struct composition; `admit_message`; `run_until_idle`; `step`)
- Completed architecture review — "SynapseOS Architecture Review — Temporal Runtime, Timers, Delayed Execution, and Time Ownership" (this session; analytical basis for this document, independently re-verified against source per §7, not cited as a substitute for that verification).

## 26. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-14 | Denver Jacobs | Initial Draft. Establishes the Temporal Runtime Architecture: Temporal Runtime component placement, responsibility boundaries, timer identity model, timer lifecycle, delayed-execution and admission-interaction requirements, capability interaction, mailbox interaction, causation, supervision interaction, failure semantics, clock architecture, audit architecture, and future-compatibility constraints. Resolves the open question ARCH-004 §4/§21 named and declined to resolve ("timers or scheduled/delayed restart") by establishing the general temporal mechanism a future restart-backoff policy could be built upon, without defining that policy. |

## 27. Approval Status

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-14 |
