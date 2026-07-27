---
document_id: ARCH-007
title: Persistent Actor Architecture
project: SynapseOS
specification: SynapseOS — durable actor domain-state persistence and restoration, realizing the Storage Architecture ARCH-002 §6/§23 defers
version: 0.4.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-26
last_updated: 2026-07-27
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
    - GOV-004 (Engineering Principles)
  standards:
    - STD-001 (v0.1.0 and v0.2.0 Approved via normal-governance disposition; current v0.4.0 content not separately approved — see STD-001's own Approval Status section)
  architecture:
    - ARCH-000 (Draft — whole-system introduction)
    - ARCH-001 (Draft — constitutional foundation; §11 Runtime/Infrastructure-layer scoping)
    - ARCH-002 (Draft — Runtime architecture; §6, §7, §9, §15, §18, §20, §21, §22, §23 directly realized by this document)
    - ARCH-004 (Draft — Local Actor Supervision Architecture; component-placement and authorization-boundary precedent, §9, §17)
    - ARCH-005 (Draft — Temporal Runtime Architecture; ActorId-keyed replaceable-service precedent, §11, §21)
  rfcs: None
  adrs:
    - ADR-0015 (Audit Emitter Failure Semantics)
    - ADR-0016 (Trusted Core Interaction Rule)
    - ADR-0017 (Bootstrap Capability Trust Root)
  roadmap: None
  research: None
  operational: None
  consolidation:
    - DES-001 (v0.2.0, Draft — consolidation/DES-001-Persistent-Actor-Design-Exploration.md; the sole design-exploration authority this document codifies; approved by DAR-001 narrow re-review)
    - AFR-001 (v0.1.1, Draft — consolidation/AFR-001-Architecture-Freeze-Review.md)
  source_artifacts: None — this document defines architecture only; no implementation exists yet
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-007 — Persistent Actor Architecture

*Filename pattern: `ARCH-007-Persistent-Actor-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Document Control and Status

| Field | Value |
|---|---|
| Document ID | ARCH-007 |
| Title | Persistent Actor Architecture |
| Version | 0.4.0 |
| Status | **Draft** |
| Author | Denver Jacobs |
| Approval authority | Chief Architect (Class B, per GOV-010 §5), vacant; Founder (interim) |
| Created | 2026-07-26 |
| Classification | Public |

This document is Draft. It has not been reviewed or approved, and nothing in it should be read as operative or binding until it completes the same governance process ARCH-002 through ARCH-006 are themselves subject to (GOV-003, GOV-010). This document introduces no implementation and authorizes none; it establishes architecture only, to be realized by a future Engineering Work Order (STD-001 §46) once approved.

This document is the authoritative source ARCH-002 §6 and §23 already anticipated and named: ARCH-002 §23's Deferred Architecture table lists "Storage Architecture: Persistence technology, snapshot format, durability guarantees" against the contract "Persistence/Restoration Service interface boundary; no silent reinstatement of revoked authority." This document is that Storage Architecture document, scoped specifically to a durable actor's own domain state and its restoration. Persistence technology, snapshot encoding format, and durability guarantees remain out of scope here (§3) and remain deferred to future, separately authorized work.

This document codifies, and introduces no design beyond, DES-001 v0.2.0 (Persistent Actor Design Exploration), which completed a design-approval review (DAR-001) confirming it as a sufficient, internally consistent, evidence-based foundation. Every architectural decision recorded below traces to a specific DES-001 recommendation; none is newly invented here.

## 2. Purpose

This document defines the authoritative architecture for **durable persistence of an actor's own domain state** in the SynapseOS Runtime: what may be persisted, who supplies it, who encodes and stores it, who authorizes each protected operation, how restoration is validated before an actor becomes live again, what happens when persistence or restoration fails, and what must be truthfully audited throughout. It resolves the "Storage Architecture" deferral ARCH-002 §6 and §23 already recorded, and establishes the architectural boundary for actor-domain-state deletion and retention that no prior document has defined.

This document does not select a persistence technology, snapshot encoding, or storage engine; does not define implementation APIs; and does not authorize implementation. It is architecture: what must be true, who owns what, and why — consistent with ARCH-002's own stated method (`ARCH-002 §1`: "precise enough that independent engineering teams could implement recognizably conformant runtimes without identical internal code").

## 3. Scope

**In scope:** the architectural placement of a new Persistence/Restoration responsibility within the existing Runtime component model, realizing the boundary ARCH-002 §6 already named; the distinction between an actor's durable domain state and every other category of state in the system; the identity model governing what persisted state belongs to; the ownership boundary between an actor's own domain-state meaning and the mechanical encoding/storage of it; the restoration sequence and its mandatory capability-revalidation precondition; the architectural treatment of persisted-state deletion and retention, as distinct from instance-level termination; the failure taxonomy for persistence- and restoration-related failures; the audit obligations this capability introduces; and the capability-authorization boundary governing every protected persistence operation.

**Out of scope:** persistence technology, storage engine, or database selection of any kind; snapshot encoding format; the exact opt-in interface an actor uses to supply its domain state; numeric retention-policy thresholds, schedules, or archival mechanisms; event sourcing, command replay, or any durable-execution model (identified as compatible future extensions, not designed here — §24); concurrent-persistence coordination for the same logical actor (identified, not resolved — §18); distributed or cross-process persistence, replication, or clustering; workflow orchestration; any public Rust API, struct, trait, enum, method signature, or storage schema. See §24 for the complete deferred-decision statement.

**Explicitly distinguished from in-memory suspend/resume.** ARCH-002 §15 already defines a `Suspended` actor state, restored "only after successful revalidation" (ARCH-002 §9). This document extends the identical revalidation discipline to durable persistence, but does not redefine, narrow, or duplicate the existing in-memory Suspended/Restore mechanism — it establishes the durable case as a second, architecturally consistent application of the same already-published revalidation requirement, not a competing one.

## 4. Non-Goals

This document does not define, and takes no position on:

- persistence technology, storage engine, database, or filesystem choice of any kind;
- snapshot encoding format, serialization mechanism, or wire representation;
- the exact interface, method names, type names, generic parameters, ownership semantics, or error model an actor uses to supply or reconstruct its own domain state (deferred to implementation, per §24);
- numeric retention periods, archival schedules, or purge timing of any kind;
- event sourcing, command replay, or durable-execution/workflow models (compatible future extensions, §24 — not designed here);
- concurrent persistence of the same logical actor's state (identified as an open question, §18 — not resolved here);
- distributed, cross-process, or clustered persistence, replication, or failover;
- Runtime-process-level restart or reattachment policy at startup (identified, §24 — not resolved here);
- any Rust struct, trait, enum, method signature, function name, or field layout;
- any new Trusted Core component (ARCH-002 §5–§6 is unchanged; see §12);
- any new actor lifecycle state beyond ARCH-002 §15's existing set;
- any new constitutional guarantee beyond ARCH-001's four (non-forgery, integrity, enforcement-at-invocation, revocation-state enforcement).

## 5. Architectural Context

This document amends no prior authority. It extends, and is bound by, the following without redefinition:

- **ARCH-000** established SynapseOS's whole-system introduction; this document inherits it without restatement.
- **ARCH-001** fixes the four constitutional guarantees and confines persistence mechanics to the Runtime/Infrastructure layer (`ARCH-001 §11`: "all subsequent ARCH documents... refine Runtime-layer and Infrastructure-layer design — the mechanics of scheduling, isolation, persistence, transport, and subsystem behavior"). This document introduces no new guarantee and weakens none.
- **ARCH-002** remains the sole authority for Trusted Core component responsibility, ownership, and prohibition (`ARCH-002 §6`), the actor and Runtime lifecycle state model (`ARCH-002 §15`), the minimum audit-event set (`ARCH-002 §18`), the Runtime Interfaces table (`ARCH-002 §20`), and the Extension and Replaceability Model (`ARCH-002 §19`). This document amends none of it. It specifically completes the deferral `ARCH-002 §6` and `ARCH-002 §23` already recorded (§1 above), and it extends, without altering, the already-stated facts that:
  - "`ActorId` persists across suspension, resumption, and restart; `ActorInstanceId` changes across restarts even when the owning actor's logical identity does not" (`ARCH-002 §7`);
  - a **Persistence/Restoration Service** already exists as a named, replaceable, non-Trusted-Core service whose responsibility is "the mechanical snapshot/replay I/O that Lifecycle Guardian validates," bound by "MUST NOT reinstate authority Lifecycle Guardian has not revalidated" (`ARCH-002 §6`);
  - "Restoration validation is a joint act: Lifecycle Guardian triggers it on resume; Capability Authority performs it, and a binding that fails revalidation is not reinstated" (`ARCH-002 §9`).
- **ARCH-004** established the precedent this document follows directly for introducing a new replaceable service: narrow responsibility, Runtime-mediated reach only, and an explicit authorization-boundary statement — "Supervisor does not grant authority. It has no path to Capability Authority and holds no capability-issuing power of its own" (`ARCH-004 §17`, §12). This document applies the identical pattern to Persistence Service.
- **ARCH-005** established the precedent for `ActorId`-keyed replaceable-service state that must survive an actor restart, and explicitly anticipated this document: "a registration is already a discrete, `ActorId`-keyed data record — the same shape a persistence layer would need to snapshot, on the same Persistence/Restoration Service interface boundary ARCH-002 §6/§23 already establishes" (`ARCH-005 §21`).
- **ADR-0016** (Trusted Core Interaction Rule) is the specific authority this document depends on most directly. Its two rules — "the Runtime is the sole architectural entity accountable for establishing and coordinating Trusted Core interactions" (Rule 1) and "Trusted Core components must not independently establish or own direct peer interaction paths" (Rule 2) — are extended, not amended, by this document to a new participant (Persistence Service, §12): Persistence Service connects to every other component exactly as any other replaceable service already does, through the Runtime, never directly.
- **ADR-0015** (Audit Emitter Failure Semantics) governs every audit-emission failure behavior this document assumes: a mandatory audit emission that fails causes the *reporting* operation to fail, without rollback of already-committed component-level state.
- **DES-001** (Persistent Actor Design Exploration, v0.2.0, DAR-001-approved) is the sole design authority for every decision this document codifies. Where this document states a normative architectural requirement, it traces directly to a DES-001 recommendation; this document does not reopen, narrow, or extend any DES-001 decision beyond what expressing it normatively as architecture requires.
- **Governance (GOV-003, GOV-010)** and **STD-001** govern this document's own review, approval, and evidentiary process; they are not architectural inputs to its content.

## 6. Design Principles

The following principles govern every decision in this document and are each a direct application of existing authority or a direct codification of DES-001:

- **An actor's existence is already durable by construction.** `ActorId` and its capability bindings already survive instance loss (ARCH-002 §7, §9). This document extends that same durability, on the same key, to exactly one further category of data — the actor's own domain state — and to nothing else (DES-001 §1).
- **Ownership remains with the responsible component.** No decision in this document transfers, absorbs, or duplicates any existing component's ARCH-002 §6 responsibility.
- **Runtime composes; it does not gain a second decision-maker for the same fact.** Per ADR-0016 Rule 1/Rule 2, every cross-component sequence this document introduces is Runtime-mediated; no new direct peer-interaction path is created.
- **Meaning and mechanics remain separated.** The actor alone knows what its own domain state means; Persistence Service alone knows how to store and retrieve bytes. Neither may perform the other's role (DES-001 §3).
- **Authorization is never inferred from mechanism or possession.** No component that performs a persistence-related mechanism thereby gains authority over it, and an actor's own possession of an `ActorId` does not itself constitute authority over that actor's own persisted state (DES-001 §11).
- **Truthfulness over convenience**, extending ADR-0015's own precedent: no audit record may claim a persistence or restoration operation completed before it genuinely, verifiably did, and no state may be represented as authorized because it was merely possible to perform.
- **Additive, not retroactive.** Persistence is opt-in per actor; no existing actor, test, or demonstration in the current workspace is required to gain persistence for this architecture to be valid.

## 7. Persistent Actor Model

A **Persistent Actor** is a logical actor (`ActorId`) that has opted into durable retention of its own domain state, such that state genuinely survives the loss of any single execution incarnation (`ActorInstanceId`), up to and including a restart of that incarnation.

Persistence is a property of the logical actor, not of any one incarnation. An actor that has never opted in remains exactly as it is today: its state, if any, exists only within its current live incarnation and is lost on termination, restart, or Runtime shutdown, exactly as ARCH-004 §14 already discloses for ordinary supervised restart. Opting into persistence changes nothing about an actor's execution semantics, message handling, or capability model; it adds exactly one new capability — the ability for the actor's own domain state to be captured, stored, and later used to reconstruct a live incarnation.

A Persistent Actor's identity, capability bindings, and supervision relationships are unaffected by this document — they are already durable by the mechanisms ARCH-002, ARCH-004, and ARCH-005 already establish. This document adds durability for exactly one additional category of data: the actor's own domain state (§9 below is intentionally the "Persistent State Model," not a further "Persistent Actor" redefinition — no constitutional or Runtime-lifecycle concept of "actor" is altered).

## 8. Identity Model

Persisted domain state MUST be keyed by `ActorId`, and MUST NOT be keyed by `ActorInstanceId`.

This is not a new decision; it is the direct, required consequence of ARCH-002 §7's own already-published identity model, independently reconfirmed by DES-001 §5 from four converging directions: Supervisor's own hierarchy and restart accounting (ARCH-004 §12, §15); Temporal Runtime's own registrations (ARCH-005 §11, §21); Capability Authority's own bindings (already `ActorId`-keyed); and a comparative study of external actor-persistence systems that converged, without exception, on the identical distinction between a persistent logical identity and an ephemeral execution incarnation.

`ActorInstanceId` remains exactly what ARCH-002 §7 and ARCH-004 §12 already establish: an ephemeral identifier for one execution incarnation, replaced — never reused — on every restart. No mechanism this document establishes may cause an `ActorInstanceId` to be treated as durable, reused, or restored in place; restoring a Persistent Actor's domain state always produces a new incarnation under the same, unchanged `ActorId`, exactly as an ordinary supervised restart already does (ARCH-004 §12).

## 9. Persistent State Model

Persisted domain state MUST contain only an actor's own domain state — the accumulated, application-meaningful data the actor's own logic produces and consumes. Persisted domain state MUST NOT contain, and no mechanism this document establishes may cause it to contain:

- **Execution state** — any data scoped to one in-progress execution cycle (ARCH-002 §10), including any host-execution-handle value, which ARCH-002 §10 already requires to remain opaque and non-persisted;
- **Capability state** — capability bindings or revocation state, which remain exclusively Capability Authority's own tracked state (ARCH-002 §9); persisting a second, independent copy would create a competing source of truth, directly prohibited by ARCH-002 §6's own "MUST NOT reinstate authority Lifecycle Guardian has not revalidated" rule;
- **Transient Runtime-mechanical state** — mailbox contents, Scheduler ready-set membership, or any in-flight dispatch bookkeeping, consistent with the already-established, disclosed non-preservation of mailbox contents across restart (ARCH-004 §14);
- **Audit history** — the record that an event occurred is Audit Emitter's own, separate, already-established responsibility (ARCH-002 §18) and is never part of an actor's own persisted domain state.

This boundary is exhaustive: the only thing a conformant Persistence/Restoration Service implementation may ever durably capture on an actor's behalf is that actor's own domain state, as the actor itself supplies it (§13).

## 10. Ownership Model

Every responsibility this architecture introduces has exactly one owner. No responsibility below may be duplicated, shared, or silently assumed by a component other than the one named.

| Responsibility | Owner |
|---|---|
| The meaning of an actor's own domain state; supplying a structured representation of it; reconstructing it from that representation | **Actor** (§13) |
| Encoding a supplied domain-state representation into stored form; decoding stored data back into that representation; storage, retrieval, and durability mechanics | **Persistence Service** (§12) |
| Triggering restoration validation — in-memory Suspended/Resume | **Lifecycle Guardian**, exactly as ARCH-002 §9 establishes, for an already-tracked `ActorInstanceId` |
| Triggering restoration validation — durable restore | **Runtime**, as sole orchestrator (ADR-0016 Rule 1) — Lifecycle Guardian has no applicable pre-instance role (§15) |
| Authorizing every protected persistence operation; revalidating capability bindings on restore | **Capability Authority** (§14) |
| Deciding when a checkpoint or restoration occurs; orchestrating the full sequence across every component involved | **Runtime** (§11), sole composer (ADR-0016 Rule 1) |

This table is the single authoritative statement of ownership for this architecture. Every other section is a more detailed statement of one row.

**Why "triggering restoration validation" has two owners.** ARCH-002 §9 establishes exactly one triggering responsibility for Lifecycle Guardian, scoped to resuming an already-tracked `ActorInstanceId` from `Suspended` back to `Idle`/`Ready` (ARCH-002 §6, §9, §15, §20). Durable restoration (§15 below) is a case ARCH-002 never addressed and this document does not retroactively assign to it: the actor's prior instance may no longer exist, and the restored incarnation's own `ActorInstanceId` does not exist yet either, at the point revalidation must begin. Lifecycle Guardian's ARCH-002-defined responsibility has no applicable role before such an instance is created — there is no tracked instance, and no legal transition, for it to act on. For durable restore, Runtime — already the sole cross-component orchestrator (§11; ADR-0016 Rule 1) — triggers Capability Authority's revalidation directly instead. Capability Authority remains, in both cases, the sole performer of the revalidation itself (ARCH-002 §9; §14 below); this split changes who *triggers* the step, never who *performs* it or who *authorizes* anything. Once Actor Host has created and registered the new live instance, Lifecycle Guardian governs its lifecycle exactly as it already governs any other instance (ARCH-002 §6, §15) — this document neither removes nor diminishes that ongoing responsibility, only the pre-instance trigger role Lifecycle Guardian was never actually positioned to perform for this case.

## 11. Runtime Responsibilities

Runtime MUST remain the sole cross-component composer of every persistence-related sequence this document introduces, on the identical basis ADR-0016 Rule 1 already establishes for every other Trusted-Core-adjacent sequence in the system.

Runtime:

- MUST be the sole component that decides when a checkpoint or restoration operation begins, on explicit instruction (an embedder or a policy layer), never ambiently or as an automatic side effect of an unrelated operation;
- MUST obtain or verify the required authorization from Capability Authority before directing any protected persistence operation (§14, §20);
- MUST sequence restoration so that capability revalidation (§14) completes, successfully, before any restored actor instance is made live;
- MUST NOT itself perform encoding, decoding, or storage mechanics — those remain Persistence Service's own responsibility (§12);
- MUST NOT itself perform authorization decisions — those remain Capability Authority's own responsibility (§14);
- MUST cause every persistence-related audit event this document requires (§19) to be truthfully emitted, in the order the underlying facts genuinely occurred.

Runtime gains no new decision authority of its own by virtue of this document; it gains exactly one new orchestration responsibility, on the same basis it already orchestrates supervision (ARCH-004) and temporal delivery (ARCH-005).

## 12. Persistence Service Responsibilities

**Persistence Service** is a new, narrow, Runtime-composed replaceable service, positioned architecturally parallel to Scheduler, Supervisor, and Temporal Runtime (ARCH-002 §6's "Replaceable services" table; ARCH-004 §9.1's identical precedent). It is not a Trusted Core component and introduces no new constitutional concept.

Persistence Service:

- MUST perform the mechanical encoding of an actor-supplied domain-state representation into stored form, and the mechanical decoding of stored data back into that representation;
- MUST perform storage, retrieval, and durability mechanics for exactly the data supplied to it — never inventing, inferring, or supplementing content the actor did not supply;
- MUST be reachable only through Runtime, and MUST NOT establish or use any direct interaction path to any other component (ADR-0016 Rule 2, extended to this new participant exactly as ARCH-004 §10.2 and ARCH-005 already extend it to Supervisor and Temporal Runtime);
- MUST NOT decide whether a persistence or restoration operation is authorized — that determination belongs exclusively to Capability Authority (§14);
- MUST NOT reinstate authority Capability Authority has not revalidated (ARCH-002 §6, restated here as the load-bearing constraint this entire document exists to uphold);
- MUST be policy-neutral: it defines no security policy, grants no authority, and makes no decision about who may invoke it (§20);
- MUST report failure truthfully rather than silently substituting default, cached, or partial data for a genuine read or write outcome (§18).

Persistence Service's storage technology, encoding format, and durability mechanism are explicitly not fixed by this document (§4) and remain a permitted implementation variation, on the same basis ARCH-002 §22 already permits variation in persistence technology generally.

## 13. Actor Responsibilities

An actor that opts into persistence MUST supply a structured representation of its own domain state, and MUST be able to reconstruct its own domain state from that representation, when instructed by Runtime to do so.

An actor:

- owns the meaning of its own domain state exclusively — no other component may inspect, infer, or reconstruct that meaning independently;
- MUST NOT produce, and Persistence Service MUST NOT accept, raw encoded bytes directly from an actor as the primary contract — the actor supplies structure; encoding remains Persistence Service's own responsibility (§10, §12);
- owns the evolution of its own domain-state representation over time (schema evolution), including determining whether a previously stored representation remains acceptable;
- gains no persistence-related authority over any other actor's state, and gains no authority over its own persisted state merely by being the actor that produced it (§20).

The exact interface, method names, type names, or language-level mechanism by which an actor supplies and reconstructs its own domain-state representation is explicitly not fixed by this document (§4, §24).

## 14. Capability Authority Responsibilities

Capability Authority MUST remain the sole source of authorization truth for every protected persistence operation this document defines (checkpoint, restore, inspection of persistence metadata where applicable, deletion, and retention/archival actions where applicable).

Capability Authority:

- MUST perform every capability-binding revalidation required before an actor instance is restored to live execution, regardless of trigger source (§10): for in-memory Suspended/Resume, Lifecycle Guardian triggers revalidation exactly as ARCH-002 §9 already establishes; for durable restoration, Runtime triggers revalidation directly, exactly as §10 and §15 of this document establish. Capability Authority remains, in both cases, the sole component that performs and decides capability revalidation, and no restored actor instance may become live before required revalidation has genuinely completed;
- MUST be the exclusive authority Runtime consults before directing any protected persistence operation;
- MUST NOT be bypassed, duplicated, or second-guessed by Persistence Service, Runtime, Lifecycle Guardian, or Supervisor — none of which may grant persistence-related authority of their own (§20);
- MUST apply a materially stronger, separately auditable authorization standard to durable-state deletion than to ordinary checkpointing (§17, §20) — the two MUST NOT share a single, undifferentiated authorization check.

Capability Authority gains no new component dependency by virtue of this document; it is reached exactly as it already is today, through Runtime, on the same ADR-0016 Rule 1 basis every other cross-component sequence in this system already follows.

## 15. Restore Architecture

Restoration of a Persistent Actor's domain state MUST follow this ordering, without exception:

```text
Runtime is instructed (or an already-authorized recovery flow
triggers) that a logical actor's persisted state should be restored
        |
        v
Runtime obtains the stored representation from Persistence Service
   (mechanics only; Persistence Service does not decide authorization)
        |
        v
Runtime causes the actor's own domain state to be reconstructed
   from the decoded representation
        |
        v
BEFORE any resulting instance is made live:
Runtime identifies the actor's currently bound capabilities and
directly requests Capability Authority to revalidate every one of
them (no ActorInstanceId yet exists for Lifecycle Guardian to act
on — see the explanatory paragraph below and §10)
   (Runtime-mediated throughout; no direct component-to-component
    call exists at any step — ADR-0016)
        |
        v
A binding that fails revalidation is not reinstated (ARCH-002 §9);
this does not, by itself, abort the restore (see below)
        |
        v
Only once required revalidation has genuinely completed: Actor Host
creates a live instance under the same, unchanged ActorId and a new
ActorInstanceId (§8)
        |
        v
Lifecycle Guardian now governs the newly created instance, exactly
as it already governs any other instance (ARCH-002 §6, §15)
        |
        v
The result is truthfully audited (§19)
```

A restored actor MUST be treated, for supervision purposes, exactly as an ordinary restarted actor already is (ARCH-004 §15): if registered, it remains under its existing supervision relationship without any re-registration act, because that relationship is already `ActorId`-keyed and therefore unaffected by whether the current incarnation was freshly created or restored.

Capability revalidation on restoration is a **mandatory precondition**, not an optional hardening step. No implementation of this architecture may make a restored instance live without it. This closes, for the durable case, a gap independently identified against the existing in-memory Suspended/Restore mechanism (ARCH-002 §9's own revalidation requirement, not yet realized in that narrower case) — this document does not resolve that pre-existing implementation gap, but requires that no durable-restore implementation may be built without it.

Durable restoration's revalidation is **Runtime-triggered**, not Lifecycle-Guardian-triggered (§10) — a direct consequence of no `ActorInstanceId` existing yet at the point revalidation must occur, not an amendment to ARCH-002. The existing in-memory Suspended/Resume mechanism is unchanged by this document: there, Lifecycle Guardian continues to trigger restoration validation exactly as ARCH-002 §9 already establishes, because a tracked `ActorInstanceId` already exists in that case. In both the durable and in-memory cases, Capability Authority alone performs the actual revalidation (ARCH-002 §9; §14) — only the trigger differs, and only because of whether an already-tracked instance is present at the point revalidation must begin. Once Actor Host has created and registered the new live instance, Lifecycle Guardian governs its lifecycle from that point forward exactly as it already governs any other instance (ARCH-002 §6, §15).

## 16. Persistent State Lifecycle

The following events are architecturally distinct and MUST NOT be conflated:

| Event | Architectural effect on durable domain state |
|---|---|
| Actor creation | No durable state exists yet unless explicitly checkpointed |
| Checkpoint (persist) | Durable state is created or updated for the logical actor |
| Instance termination (ordinary stop) | Durable state is unaffected — it belongs to the `ActorId`, not the terminated `ActorInstanceId` |
| Actor failure | Durable state is unaffected |
| Supervisor-driven restart | Durable state is unaffected by the restart itself; whether it is *used* to restore the replacement instance is a separate, explicit act (§15), never automatic |
| Restore | Durable state is read and used to reconstruct a live instance, subject to mandatory capability revalidation (§15) |
| Runtime shutdown | Durable state is unaffected; no new restoration may begin once Runtime is no longer accepting new work, on the same basis already established for supervision (see ARCH-004's own shutdown-aware precedent) |
| Logical actor retirement | The `ActorId` is no longer expected to run again; its durable state and identity remain intact until an explicit deletion act occurs (§17) |
| Permanent logical actor deletion | The `ActorId`'s own durable record is explicitly, distinctly removed (§17) — never implied by any of the events above |

Instance-level events (termination, failure, ordinary restart) MUST NOT cause durable domain-state loss. This is a direct, required consequence of the identity model (§8): state is `ActorId`-keyed specifically so that it survives instance-level loss; permitting instance-level events to delete it would defeat the identity model this entire architecture is built on.

## 17. Deletion and Retention Architecture

Permanent deletion of a Persistent Actor's durable domain state is an act architecturally distinct from every instance-level event in §16, and MUST be governed as follows:

- Durable-state deletion MUST be explicit — it MUST NOT occur as an inferred consequence of inactivity, instance termination, actor failure, supervised restart, or Runtime shutdown.
- Durable-state deletion MUST be associated with the stable `ActorId` it removes, never with any transient incarnation.
- Durable-state deletion MUST be separately, and more strongly, authorized than ordinary checkpointing (§14, §20) — the two operations MUST NOT share a single authorization check.
- Durable-state deletion MUST be truthfully auditable, distinguishing at minimum a deletion request, its authorization outcome, and its completion or failure outcome (§19).
- The architecture MUST support a future retention policy — a period during which deletion is reversible, or during which state is archived rather than immediately purged — without requiring the actor itself to be coupled to any specific storage infrastructure (ARCH-002 §22's already-established "permitted variation" principle, extended here).
- Durable-state deletion MUST cause Runtime to proactively cancel every pending Temporal Runtime timer registration associated with the same `ActorId` (ARCH-005 §23), ordered strictly before the deletion outcome is reported — Persistence Service performs no timer cleanup of any kind, and Temporal Runtime performs no deletion of any kind; Runtime alone coordinates both, exactly as it already coordinates every other cross-component sequence this document defines (§11; ADR-0016 Rule 1).

**Deletion coordination ordering.** Runtime MUST perform durable-state deletion in this order, without exception:

```text
Validate deletion authority
        |
        v
Cancel pending timers for ActorId
        |
        v
Audit timer cancellations
        |
        v
Attempt durable-state deletion
        |
        v
Audit deletion completion or failure
```

Cancellation is Runtime-owned preparatory cleanup, using Temporal Runtime's own existing cancellation operation, which returns the cancelled timer identities directly rather than a fallible persistence-style result — this document does not invent a cancellation failure mode the existing mechanism does not expose. Durable deletion is the definitive, externally meaningful act and MUST be attempted only after cancellation and its audit are complete, so that deletion is never reported complete while pending timers remain unintentionally associated with the deleted `ActorId`. If durable-state deletion then fails, the durable state remains, the already-cancelled timers remain cancelled, and Runtime MUST report the deletion failure truthfully; a subsequent retry MAY safely repeat timer cleanup, since cancelling an already-empty registration set is harmless. No implementation of this architecture may fabricate restoration or automatic timer re-registration as a response to a failed deletion — this coordination is not transactionally atomic, and MUST NOT be described as such; it is a truthful, ordered sequence of independently auditable steps. Timer re-arming upon a Persistent Actor's own restoration remains a separate, unresolved future architecture concern, unaffected by this paragraph.

**Normative disposition:** durable domain state survives every instance-level event in §16 and is removed only through an explicit, separately authorized, auditable, policy-governed logical-actor state-removal operation. No implementation of this architecture may cause durable state to be deleted as a side effect of any other operation.

The concrete retention model — immediate purge, tombstoning, a bounded retention period, or transition to an archived state; which service enforces retention policy; the treatment of related metadata upon deletion; and whether deletion is permitted while a live incarnation of the same `ActorId` exists — is explicitly deferred to future architecture and implementation work (§24). This document fixes the boundary (deletion is explicit, distinct, and auditable); it does not fix the mechanism.

## 18. Failure Architecture

| Failure | Required architectural behaviour |
|---|---|
| Failed checkpoint (write) | The reporting operation fails (ADR-0015's already-established rule, extended here); no partial write may be treated as authoritative; no actor-visible state change results from a failed persist |
| Failed restore (read) | The restoration attempt fails cleanly; the logical actor is left with no live instance — an honest degradation, never a silent fallback to default or empty state, and never an ambient retry |
| Corrupted stored representation | Treated identically to a failed restore — a decode failure is a restoration failure, not a distinct case requiring different architectural treatment |
| Missing stored representation | MUST be architecturally distinguishable, in the audit record, from a corrupted one — "never persisted" is not an error condition; "persisted, then lost" is, and the two MUST NOT be reported identically |
| Concurrent persistence of the same logical actor | Not resolved by this document; identified as requiring either an explicit architectural exclusion or a new coordination primitive before implementation may proceed (§24) |
| Version mismatch between a stored representation and the current actor | The actor's own reconstruction logic is the sole authority on acceptability; Persistence Service and Runtime MUST NOT infer compatibility on the actor's behalf |
| Schema evolution of an actor's own domain-state representation | Owned entirely by the actor itself (§13); this document does not require or preclude a general migration mechanism |
| Storage-technology outage | Deliberately outside architectural scope (ARCH-002 §22); the *contract* — persistence and restoration operations may fail, and failure MUST NOT be silently absorbed — is architectural; outage-handling policy is not |

Every failure in this table MUST be truthfully reported through the audit mechanism (§19). No failure condition may be represented as success, and no partial or ambiguous outcome may be represented as fully determinate.

## 19. Audit Architecture

The following facts MUST be truthfully, distinctly auditable, extending — never contradicting — the minimum audit-event set ARCH-002 §18 already establishes:

- a checkpoint (persist) was requested, and its outcome (success or failure);
- a restoration was requested, and its outcome, including whether the stored representation was missing versus corrupted (§18);
- capability revalidation's own outcome, per binding, during restoration (§15);
- an actor instance was created as the result of a successful restoration, distinguished from ordinary creation;
- a durable-state deletion was requested, its authorization outcome, and its completion or failure outcome (§17);
- every authorization decision this document requires (§14, §20), recorded **separately** from the operational outcome it governs.

**Ordering requirement, extending ARCH-004 §18's own established discipline:** no audit record may claim a checkpoint, restoration, or deletion completed before the underlying operation has genuinely, verifiably completed. Authorization-decision events and operation-outcome events MUST remain distinct: a recorded authorization grant is not itself evidence that the authorized operation occurred, and a recorded operation success is not itself evidence that it was properly authorized.

This document does not fix concrete audit-event identifiers or a new `AuditEvent` field layout — consistent with ARCH-004 §18 and ARCH-005 §20's own established precedent, this is an implementation-phase concern for the eventual Engineering Work Order, not fixed at the architecture level.

## 20. Security Architecture

Every protected persistence operation this document defines — checkpoint, restore, inspection of persistence metadata where applicable, deletion, and retention/archival actions where applicable — MUST be capability-authorized through the existing Capability Authority model. No second, parallel, or informal authorization system may be introduced by this document or by any implementation of it.

The following MUST hold without exception:

- Persistence Service does not grant authority and defines no security policy of its own.
- Runtime does not invent authority; it obtains or verifies authorization from Capability Authority before acting.
- Lifecycle Guardian does not grant persistence-related rights.
- Supervisor does not grant persistence-related rights.
- An actor's own possession of an `ActorId` does not itself constitute authority over that actor's persisted state.
- Restoration MUST include capability revalidation, performed by Capability Authority, before the restored instance becomes live (§15) — this is a security requirement, not merely a lifecycle-ordering convenience.
- Durable-state deletion MUST require authorization materially stronger, and separately auditable from, ordinary checkpointing (§17).
- A storage operation succeeding is never, by itself, evidence that it was authorized; an authorization decision succeeding is never, by itself, evidence that the storage operation completed. The two MUST remain independently verifiable (§19).

This section applies ARCH-004 §17's own established precedent — "\[the specialised service\] does not grant authority... has no path to Capability Authority... holds no capability-issuing power of its own" — to Persistence Service without exception, and extends the identical discipline to every other component this document touches.

## 21. Architectural Invariants

Any implementation of this architecture MUST preserve the following invariants:

1. Persisted domain state is keyed by `ActorId`; it is never keyed by `ActorInstanceId`.
2. Persisted domain state contains only an actor's own domain state — never execution state, capability state, transient Runtime-mechanical state, or audit history.
3. An actor owns the meaning of its own domain state exclusively; no other component may infer or reconstruct that meaning independently.
4. Persistence Service owns encoding, decoding, and storage mechanics exclusively; it never determines meaning and never determines authorization.
5. Runtime is the sole orchestrator of every persistence-related cross-component sequence; no direct peer-interaction path exists between any two components this document touches.
6. Capability Authority is the sole source of authorization truth for every protected persistence operation; no other component grants authority.
7. Restoration MUST NOT make an instance live until capability revalidation has genuinely succeeded.
8. Restart always produces a new `ActorInstanceId` under the same `ActorId`; a restored incarnation is never a reused one.
9. Instance-level events (termination, failure, ordinary restart, Runtime shutdown) MUST NOT delete durable domain state.
10. Durable-state deletion is explicit, distinct from every instance-level event, separately authorized, and auditable.
11. No audit record may claim a persistence, restoration, or deletion operation completed before it genuinely did.
12. Authorization-decision audit records and operation-outcome audit records remain distinct; neither may be inferred from the other.
13. No component this document touches gains capability-issuing authority it did not already, independently, hold.
14. This document introduces no new Trusted Core component and alters no existing Trusted Core component's own ARCH-002 §6 responsibility.

## 22. Replaceability

Persistence Service is a replaceable Runtime service, positioned architecturally parallel to Scheduler, Supervisor, and Temporal Runtime (§12; ARCH-002 §6's "Replaceable services" table; ARCH-004 §9.1's identical precedent). It is not a Trusted Core component and introduces no new constitutional concept (§21, invariant 14).

Persistence Service's storage technology, encoding format, and durability mechanism are explicitly not fixed by this document (§4) and remain a permitted implementation variation, on the same basis ARCH-002 §22 already permits variation in persistence technology generally. The complete statement of permitted variation and prohibited substitution is given in §25 (Conformance Requirements).

**Persistence Service MUST NOT be architected to require a specific storage, replication, or clustering infrastructure as a precondition for conformance.** An implementation locked to one particular replication or clustering topology — rather than treating persistence technology as one substitutable implementation choice among several — would not be a conformant realization of this architecture. This carries forward, as an explicit architectural prohibition, DES-001's own rejection of infrastructure-specific persistence coupling.

## 23. Compatibility with Existing Architecture

This document amends no prior authority and introduces no conflict with it (§5). Compatibility is confirmed as follows:

- **ARCH-001** — no new constitutional guarantee is introduced and none is weakened; persistence mechanics remain confined to the Runtime/Infrastructure layer (§5; ARCH-001 §11).
- **ARCH-002** — this document completes, without amending, the "Storage Architecture" deferral already recorded in ARCH-002 §6 and §23; the Persistence/Restoration Service and the `ActorId`/`ActorInstanceId` identity split are extended, not altered (§5; ARCH-002 §6, §7). ARCH-002 §9's "joint act" — Capability Authority performs revalidation, Lifecycle Guardian triggers it on resume — is extended only in its Capability-Authority-performs half, which applies unaltered to both the in-memory and durable cases; its Lifecycle-Guardian-triggers half applies, as ARCH-002 §9 itself already scopes it, only to resuming an already-tracked `ActorInstanceId`. Durable restoration (§15) is a case ARCH-002 does not address — no such tracked instance exists yet — so this document introduces Runtime-triggered revalidation for that case specifically (§10, §15), rather than extending Lifecycle Guardian's trigger role to a case it was never scoped to cover. This scopes ARCH-002 §9 accurately; it does not alter, contradict, or narrow it.
- **ARCH-004** — the component-placement and authorization-boundary precedent Supervisor established ("does not grant authority... has no path to Capability Authority... holds no capability-issuing power of its own," ARCH-004 §17) is applied to Persistence Service without exception (§5, §20).
- **ARCH-005** — the `ActorId`-keyed replaceable-service-state precedent Temporal Runtime established, and its own explicit anticipation of this document's own interface boundary (ARCH-005 §21), are extended without alteration (§5).
- **ADR-0016** — Rule 1 and Rule 2 are extended, not amended, to a new participant (Persistence Service): it connects to every other component exactly as any other replaceable service already does, through the Runtime, never directly (§5, §11, §12).
- **ADR-0015** — the audit-emission failure rule is extended, not altered, to persistence-related mandatory audit obligations (§5, §18).
- **DES-001** — every normative statement in this document traces to a specific DES-001 v0.2.0 (DAR-001-approved) recommendation; no DES-001 decision is reopened, narrowed, or extended beyond what expressing it normatively as architecture requires (§5).

No existing Trusted Core component's ARCH-002 §6 responsibility is altered. No new Trusted Core component is introduced. Runtime remains the sole cross-component orchestrator (ADR-0016 Rule 1). Capability Authority remains the sole source of authorization truth (§14, §20). `ActorId` remains the durable identity; `ActorInstanceId` remains ephemeral (§8).

## 24. Deferred Decisions

The following are explicitly, deliberately deferred to future architecture or authorized implementation work, and are not resolved by this document:

- **Persistence technology, storage engine, and encoding format** (§4, §12) — a future implementation concern, permitted to vary (ARCH-002 §22).
- **The exact opt-in interface an actor uses to supply and reconstruct its own domain-state representation** (§13) — deliberately not fixed at the architecture level, on the same basis ARCH-004 fixed no Rust signatures.
- **Version-envelope ownership** (outer format-version tag: Persistence Service versus actor-owned) — DES-001 identified this as an open question for ARCH-007; not resolved by this document.
- **Concurrency and checkpoint coordination for the same logical actor** (§18) — genuinely unresolved; a future architecture or implementation decision must either exclude this case structurally or introduce a coordination primitive this document does not define.
- **Runtime-startup restoration policy** — whether previously persisted actors are automatically reattached on Runtime startup, or restoration is always explicit and on-demand. Not resolved here.
- **The concrete retention and deletion mechanism** (§17) — immediate purge versus tombstoning versus a bounded retention period versus an archived state; which service enforces retention policy; treatment of related metadata upon deletion; whether deletion is permitted while a live incarnation exists.
- **Failed or partial deletion representation** — how a durable-state deletion that fails, or completes only partially, is represented as distinct from a successful one. DES-001 identified this as an open question for ARCH-007; not resolved by this document.
- **Capability granularity for persistence operations** (§20) — which specific operations (checkpoint, restore, inspect, archive, delete) require distinct capabilities; whether an actor may request its own checkpoint; who may initiate restore or permanent deletion; whether administrative retention actions require separate authority from ordinary persistence operations.
- **Concrete audit-event identifiers and any `AuditEvent` shape change** (§19) — an implementation-phase concern.
- **Event sourcing and command-replay/durable-execution models** — named as compatible future extensions for workflow, scheduled-job, and long-running-agent use cases (DES-001 §2, §9), not designed by this document, and not precluded by it.
- **Distributed, cross-process, or clustered persistence** — out of scope entirely (§3); this document's `ActorId`-keying discipline is the specific design choice that keeps a future distributed implementation viable without redesign, on the same basis ARCH-004 §21 and ARCH-005 §21 already established for supervision and temporal delivery respectively.

None of these requires reopening any decision this document does record; each is a genuinely separate, bounded question left to the process — future architecture, ADR, or Engineering Work Order — that STD-001 already provides for it.

## 25. Conformance Requirements

**Mandatory for any implementation claiming conformance to this architecture:**

- Every architectural invariant in §21 holds at all times.
- Persisted domain state is `ActorId`-keyed without exception.
- No component other than the actor itself determines the meaning of that actor's own domain state.
- No component other than Persistence Service performs encoding, decoding, or storage mechanics.
- No component other than Capability Authority authorizes a protected persistence operation.
- No component other than Runtime orchestrates a cross-component persistence sequence.
- Restoration never completes without genuine, prior capability revalidation.
- Instance-level events never delete durable domain state.
- Durable-state deletion is always explicit, distinctly authorized, and auditable.

**Permitted variation:** persistence technology; storage engine; snapshot encoding format; the exact actor-facing interface for supplying and reconstructing domain state; retention-policy mechanism and timing; concrete audit-event identifiers.

**Prohibited:** any mechanism by which durable state is deleted as a side effect of an instance-level event; any mechanism by which an actor's own domain state is inferred or reconstructed by a component other than the actor itself; any restoration path that omits capability revalidation; any authorization shortcut that treats checkpoint and deletion as equivalently authorized operations; any direct peer-interaction path between Persistence Service and any other component; any persistence-related capability-issuing power held by Persistence Service, Runtime, Lifecycle Guardian, or Supervisor; any architectural dependency on a specific storage, replication, or clustering infrastructure as a precondition for conformance (§22).

## 26. References

Internal:

- GOV-003 — Governance Model
- GOV-010 — Decision Framework
- GOV-004 — Engineering Principles
- STD-001 — Documentation Standards
- ARCH-000 — Introduction
- ARCH-001 — Constitutional Architecture (§11)
- ARCH-002 — Runtime Architecture (§6, §7, §9, §15, §18, §19, §20, §21, §22, §23)
- ARCH-004 — Local Actor Supervision Architecture (§9, §10, §12, §14, §15, §17, §18)
- ARCH-005 — Temporal Runtime Architecture (§11, §21)
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- DES-001 — Persistent Actor Design Exploration (v0.2.0), the sole design authority this document codifies
- AFR-001 — Architecture Baseline Certification

Source evidence (verified during design exploration and independently re-confirmed for this document, not restated from memory):

- `api/src/lib.rs` (`Actor` trait — currently no serialization capability, confirming §4/§13's scope boundary)
- `services/persistence/src/lib.rs`, `src/internal.rs` (existing `Persistence` trait stub — confirmed unimplemented, confirmed keyed by `ActorInstanceId`, confirmed non-authoritative per DES-001 §"Architectural Consequences")
- `core/actor-host/src/internal.rs` (`ActorHostImpl` field layout — confirms actor-defined behavior state exists and is currently opaque)
- `runtime/src/lib.rs` (`restore_actor_instance` — confirms the existing in-memory restore path does not yet perform capability revalidation, the precondition §15 requires closed before any durable restore implementation)

## 27. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-26 | Denver Jacobs | Initial Draft. Establishes the Persistent Actor Architecture: identity model (`ActorId`-keyed), persistent state model (domain state only), ownership model (actor/Persistence Service/Runtime/Capability Authority), Runtime/Persistence Service/Actor/Capability Authority responsibilities, restore architecture with mandatory capability revalidation, persistent state lifecycle, deletion and retention architecture, failure architecture, audit architecture, security architecture, fourteen architectural invariants, and conformance requirements. Completes the deferral recorded in ARCH-002 §6 and §23 ("Storage Architecture"). Codifies DES-001 v0.2.0 (DAR-001-approved) without introducing new design. |
| 0.2.0 | 2026-07-26 | Denver Jacobs (AI-assisted) | Targeted correction pass applying, and limited to, the findings of the independent Architecture Review of ARCH-007. No architectural decision was reopened, no ownership boundary altered, and no implementation detail introduced. (1) **Ownership Model table (§10)** — corrected all four internal cross-references, each previously pointing to the wrong section (Actor §11→§13; Persistence Service §10→§12; Capability Authority §12→§14; Runtime §9→§11). (2) **Required section structure** — added §22 (Replaceability) and §23 (Compatibility with Existing Architecture) as standalone sections, reusing existing, already-approved material from §5 and §12 without introducing new architectural content; Deferred Decisions, Conformance Requirements, References, Change History, and Approval Status renumbered §24–§28 accordingly. (3) **Infrastructure coupling prohibition** — added an explicit MUST NOT statement to §22 (Replaceability) and to §25's Prohibited list, carrying forward DES-001's own rejection of infrastructure-specific persistence coupling as a normative architectural prohibition. (4) **Deferred Decisions (§24)** — added "Version-envelope ownership" and "Failed or partial deletion representation" as explicitly deferred items, matching DES-001's own "Deferred to ARCH-007" list; neither is resolved. (5) **Internal cross-references** — performed a complete validation pass over every internal section citation in the document and corrected every remaining incorrect one found, beyond the Ownership table alone: §1 (Document Control and Status table's own Version field, unsynchronized with this correction's version bump); §§3–4 (Non-Goals/Scope: concurrency and event-sourcing citations); §5 (one ambiguous bare citation disambiguated as `ARCH-002 §9`, and three self-references to Persistence Service Responsibilities that had been left citing §9 from an earlier section count, corrected to §12); §7 (a self-reference reading "§7 below," inside §7 itself, corrected to "§9 below"); §9 (a citation to Runtime Responsibilities corrected to Actor Responsibilities, §13); §§11–14 (Runtime, Persistence Service, Actor, and Capability Authority Responsibilities: capability-revalidation, authorization-obtaining, encoding/decoding-ownership, authorization-ownership, and audit citations); §15 (restore-architecture diagram's audit citation); and §17 (deletion authorization and auditability citations). No citation to an external document (ARCH-001, ARCH-002, ARCH-004, ARCH-005, ADR-0015, ADR-0016, DES-001, STD-001) was altered. |
| 0.3.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Targeted correction pass resolving durable-restoration validation triggering, identified by the Independent Implementation Review of the Persistent Actors implementation (finding M-1) and completed across two correction passes. No architectural decision was reopened and no new component, capability, or interface was introduced. (1) **Ownership Model (§10)** — split the single "Triggering restoration validation" row into two: Lifecycle Guardian for the existing in-memory Suspended/Resume case (unchanged, ARCH-002 §9), and Runtime for durable restore, with an added explanatory paragraph. (2) **Restore Architecture (§15)** — corrected the restore diagram and its explanatory text: durable-restore capability revalidation is triggered by Runtime directly, not by Lifecycle Guardian, because no `ActorInstanceId` exists yet for the restored incarnation at the point revalidation must occur; Lifecycle Guardian's role begins once Actor Host creates and registers the new live instance. (3) **Compatibility with Existing Architecture (§23)** — corrected the ARCH-002 compatibility statement to split ARCH-002 §9's "joint act" into its Capability-Authority-performs half (extended unaltered to both cases) and its Lifecycle-Guardian-triggers half (extended only to the resume case ARCH-002 §9 already scopes it to). (4) **Capability Authority Responsibilities (§14)** — corrected the one surviving internal contradiction: the requirement previously stated capability-binding revalidation was "triggered by Lifecycle Guardian... for the durable case," contradicting the corrected §10/§15/§23; reworded to state the trigger source differs by restoration type (Lifecycle Guardian for resume, Runtime for durable restore) while Capability Authority performs and decides revalidation in both cases, unchanged. Root cause: ARCH-002 §9 scopes Lifecycle Guardian's triggering responsibility to an already-tracked `ActorInstanceId`'s `Suspended → Idle/Ready` transition; durable restoration is a case ARCH-002 does not address, and citing ARCH-002 §9 as unqualified authority for it was the specific defect corrected here. ARCH-002 itself is unamended and unaltered throughout. |
| 0.4.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Targeted correction, identified by the Persistent Actor and Temporal Runtime Integration Architecture Review, closing the previously undocumented relationship between durable-state deletion (§17) and Temporal Runtime's own pending timer registrations for the same `ActorId` (ARCH-005, which did not exist in its present form when this document's deletion architecture was first authored). Added one bullet to §17's governing list requiring Runtime to proactively cancel every pending Temporal Runtime timer registration for the same `ActorId` as part of durable-state deletion, cross-referencing the corresponding extension made to ARCH-005 §23 in the same correction effort. Added one new paragraph, "Deletion coordination ordering," fixing the required sequence (validate authority → cancel timers → audit cancellations → attempt deletion → audit outcome) and its failure semantics: cancellation is infallible under the existing mechanism and requires no invented failure mode; a subsequent deletion failure leaves durable state intact and timers already, harmlessly, cancelled; no compensation may fabricate restoration or automatic timer re-registration; the coordination is truthful and ordered, not transactionally atomic, and must not be described as such. Explicitly states this correction does not apply to checkpointing, ordinary restart, or restoration, and does not authorize durable timers, timer persistence, or automatic timer re-arming — all of which remain separate, unresolved future architecture concerns. No other architectural decision was reopened: §10, §14, §15, and §23 (the subject of the 0.3.0 correction) are unchanged; Persistence Service and Temporal Runtime remain, and are confirmed to remain, mutually unaware of one another; Runtime remains the sole coordinator of the extended sequence. |

## 28. Approval Status

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-26 |
| Technical Review | TBD | Pending | |
| Approval Authority | Chief Architect (vacant); Founder (interim) | Pending | |
