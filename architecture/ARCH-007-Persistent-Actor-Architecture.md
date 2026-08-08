---
document_id: ARCH-007
title: Persistent Actor Architecture
project: SynapseOS
specification: SynapseOS — durable actor domain-state persistence and restoration, realizing the Storage Architecture ARCH-002 §6/§23 defers
version: 0.5.2
status: Approved
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-26
last_updated: 2026-08-05
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
    - GOV-004 (Engineering Principles)
    - GOV-016 (Draft — SynapseOS Engineering Lifecycle Standard; §5 ADR-governance criteria applied in determining this is an amendment, not a new ADR)
  standards:
    - STD-001 (v0.1.0 and v0.2.0 Approved via normal-governance disposition; current v0.4.0 content not separately approved — see STD-001's own Approval Status section)
  architecture:
    - ARCH-000 (Draft — whole-system introduction)
    - ARCH-001 (Draft — constitutional foundation; §11 Runtime/Infrastructure-layer scoping)
    - ARCH-002 (Draft — Runtime architecture; §6, §7, §9, §15, §18, §20, §21, §22, §23 directly realized by this document)
    - ARCH-004 (Draft — Local Actor Supervision Architecture; component-placement and authorization-boundary precedent, §9, §17)
    - ARCH-005 (Draft — Temporal Runtime Architecture; ActorId-keyed replaceable-service precedent, §11, §21)
    - ARCH-008 (Approved v0.5.0 — Effect Runtime Architecture; §11.2's "no new registry" discipline, cited by this amendment's §13.3; the corrected v0.5.1 resolution mechanism itself is Runtime-held, per `RuntimeCore`'s own existing `PersistenceHandle` precedent, not a direct extension of §11.2's own Actor-definition framing)
    - ARCH-011 (v0.1.3, Approved — Founder Architecture Approval recorded as FAA-011 — Durable Storage Mechanics; envelope/backend boundary this amendment's contract sits above)
  rfcs: None
  adrs:
    - ADR-0015 (Audit Emitter Failure Semantics)
    - ADR-0016 (Trusted Core Interaction Rule)
    - ADR-0017 (Bootstrap Capability Trust Root)
    - ADR-0018 (Draft — StorageBackend Serialization Boundary; the ownership boundary this amendment supplies the missing actor-facing half of)
  roadmap: None
  research: None
  operational: None
  consolidation:
    - DES-001 (v0.2.0, Draft — consolidation/DES-001-Persistent-Actor-Design-Exploration.md; the sole design-exploration authority this document codifies; approved by DAR-001 narrow re-review)
    - AFR-001 (v0.1.1, Draft — consolidation/AFR-001-Architecture-Freeze-Review.md)
  engineering:
    - EWO-024, ER-024, EWO-024A, ER-024 Re-Review, FIA-024 (Artifact-only, synapse-runtime repository — the StorageBackend implementation chain this amendment's contract sits above)
    - EWO-025 Architectural Blocker Report (Artifact-only — the direct discovery evidence for this amendment; correctly stopped rather than inventing false durability)
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
| Version | 0.5.2 |
| Status | **Approved** — Founder Architecture Approval (FAA-012) recorded 2026-08-06, with two non-blocking observations formally noted (Approval Status, below); this document is the authoritative SynapseOS Persistent Actor architecture |
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
- **ARCH-008 §11.2** (0.5.0) established, for Provider Actors, that "'Registration'... consists of exactly two ordinary, pre-existing Runtime operations — defining an actor and creating a live instance with behavior — performed by whatever embedding code composes the Runtime," and explicitly that "there is no provider-specific registration API, manifest, or discovery protocol." §13.3 of this document (0.5.0) applies the identical discipline to durable-state contract association: no new registry component, reusing the same, already-necessary act of defining an actor.
- **ARCH-011 v0.1.3** (Approved — Founder Architecture Approval recorded as `FAA-011`) established the `StorageBackend` seam beneath Persistence Service, and its own §9 states that a backend "owns exactly, and only, raw byte-level durability for one opaque blob... stores and returns bytes" — never `DomainState`. `ADR-0018` treated this as settled architecture; `ARCH-011`'s own approval now confirms that premise directly, though `ADR-0018` itself remains, separately, its own unapproved Draft — a distinct matter from `ARCH-011`'s own approval status. This document's §13.2–§13.9 (0.5.0) supply the missing actor-facing half of that same boundary: how an opaque `DomainState` becomes the encoded bytes `ARCH-011` §9 already requires Persistence Service to hand a backend, a question `ADR-0018` explicitly left unresolved (`ADR-0018` §10's own scope).
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
- MUST cause every persistence-related audit event this document requires (§19) to be truthfully emitted, in the order the underlying facts genuinely occurred;
- **MUST hold the composition-time-supplied `ActorId`-keyed Durable-State Contract association (0.5.1, §13.3), and MUST resolve and supply the associated contract, if any, to Persistence Service before requesting a checkpoint or restore decode** — resolvable from Runtime's own already-composed state, requiring no live actor instance and no call to Actor Host (§13.3).

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
- MUST report failure truthfully rather than silently substituting default, cached, or partial data for a genuine read or write outcome (§18);
- **MUST invoke the Durable-State Contract Runtime supplies with a given encode/decode request (0.5.1, §13.3, §13.4), and MUST NOT itself resolve, discover, or select a contract independently** — contract resolution is Runtime's own responsibility (§11), never Persistence Service's; Persistence Service's role remains limited to applying whatever contract it is given.

Persistence Service's storage technology, encoding format, and durability mechanism are explicitly not fixed by this document (§4) and remain a permitted implementation variation, on the same basis ARCH-002 §22 already permits variation in persistence technology generally.

## 13. Actor Responsibilities

An actor that opts into persistence MUST supply a structured representation of its own domain state, and MUST be able to reconstruct its own domain state from that representation, when instructed by Runtime to do so.

An actor:

- owns the meaning of its own domain state exclusively — no other component may inspect, infer, or reconstruct that meaning independently;
- MUST NOT produce, and Persistence Service MUST NOT accept, raw encoded bytes directly from an actor as the primary contract — the actor supplies structure; encoding remains Persistence Service's own responsibility (§10, §12);
- owns the evolution of its own domain-state representation over time (schema evolution), including determining whether a previously stored representation remains acceptable;
- gains no persistence-related authority over any other actor's state, and gains no authority over its own persisted state merely by being the actor that produced it (§20).

The exact interface, method names, type names, or language-level mechanism by which an actor supplies and reconstructs its own domain-state representation remains implementation-phase (§4, §24, consistent with `ARCH-011` §21's identical deferral of "concrete Rust types, trait signatures, and method names"). **What is no longer deferred, as of §13.1–§13.9 below (0.5.0):** the conceptual contract governing durability eligibility, encoding ownership, contract association, persistent type identity, representation versioning, determinism, and failure semantics — the load-bearing precondition every one of those implementation-phase details themselves depends on, discovered missing by `EWO-025`'s own architectural blocker report.

### 13.1 Durability Eligibility (0.5.0)

Unchanged, restated for direct proximity to what follows: persistence remains opt-in per actor (§6, §7) — "an actor that has never opted in remains exactly as it is today" (§7). This amendment adds nothing to that rule; it defines what "opting in" now requires in practice: an actor is eligible for durable checkpointing if and only if a **Durable-State Contract** (§13.2) has been associated with its `ActorId` in Runtime's own held, composition-time-supplied association (§13.3). An actor with no associated contract remains a fully valid, ordinary actor — its `DomainState`, if any, continues to exist only within its current live incarnation exactly as §7 already states — and a checkpoint requested for it MUST fail explicitly and truthfully (§13.8; §18), never silently, never by substituting a default or partial representation.

### 13.2 The Durable-State Contract (0.5.0)

A **Durable-State Contract** is the actor-owned pair of operations — *encode* (produce a durable representation from the actor's current `DomainState`) and *decode* (reconstruct a `DomainState` from a previously produced durable representation) — that makes one concrete domain-state type genuinely, repeatably durable. This is a conceptual contract, not a Rust trait signature (§4, §24; concrete syntax remains implementation-phase). It exists because `DomainState`'s own type-erased representation (`Rc<dyn Any>`, `common::DomainState`) carries no information Persistence Service could use to encode or decode it without the actor's own, type-specific cooperation (this document, §9: the actor alone knows what its own domain state means; `EWO-025`'s blocker report: `Any` provides no default byte representation and no way to reconstruct a concrete type without already knowing, statically, what that type is).

A Durable-State Contract:

- is authored by whoever defines the actor type — the same act, and the same authority, that already supplies the actor's own message-handling behavior (§13.3);
- MUST be deterministic in the sense §13.7 defines;
- MUST NOT depend on any process-local, non-durable value — a memory address, an `Rc` pointer, an in-process `TypeId`, or any other value meaningless outside the process and allocation that produced it (directly ruling out every False Solution `EWO-025`'s blocker report named and rejected);
- owns exactly, and only, the translation between one concrete domain-state type and its own durable representation — it owns no storage, no envelope, no integrity checksum, no capability decision, and no orchestration of when encoding or decoding occurs (§13.4).

### 13.3 Contract Association — Runtime-Held, Composition-Time Supply, Not a New Registry (0.5.1)

**Corrected in 0.5.1, replacing this section's own original v0.5.0 text.** The Independent Architecture Review of `ARCH-007` v0.5.0 (Finding M-1) found that v0.5.0's account of resolution — analogized to "Actor Host already resolves... construction" — was not reconciled against `§13.4`'s own separate claim that Runtime supplies the contract, and never named one concrete resolution owner. Re-investigating Actor Host's own actual, current interface (`core/actor-host/src/lib.rs`) directly, rather than relying on analogy, found the deeper reason no reconciliation was possible: **`ActorHost::define` stores no reusable construction or behavior data at all.** It "assigns a stable, unique logical identity to a newly defined actor" — nothing more; behavior is supplied fresh at `create_instance_with_behavior` time, "already constructed by the caller, not built from a stored factory" (`core/actor-host/src/lib.rs`, its own doc comment). Actor Host, a Trusted Core component (`ARCH-002` §5–§6), has no "definition" object to extend with a Durable-State Contract without giving it a new, Trusted-Core responsibility it does not currently have — directly prohibited by invariant 14 (§21) and by this amendment's own governing constraint not to alter an existing Trusted Core component's responsibility. **v0.5.0's own citation of Actor Host as the resolution precedent was therefore inaccurate, not merely under-specified**, disclosed here rather than concealed.

The corrected model instead reuses a mechanism already concretely present, outside Trusted Core, for exactly this kind of composition-time association: `runtime/src/lib.rs`'s own `RuntimeCore` already holds a `persistence: PersistenceHandle` field, itself constructible via the unprivileged, additive `PersistenceHandle::with_backend(...)` substitution point `ARCH-011` §8 and `EWO-024` already establish. **Runtime itself — already the thing embedding code composes at startup, already the sole cross-component orchestrator (ADR-0016 Rule 1), and already the holder of exactly this kind of composition-time-supplied replaceable-service configuration — holds the `ActorId`-keyed Durable-State Contract association**, supplied once, additively, by embedding code at Runtime composition time, on the identical unprivileged basis `PersistenceHandle::with_backend` already establishes. **No new registry, manifest, discovery protocol, or Trusted Core component is introduced**: this is one further, optional, composition-time-supplied association held by a struct (`RuntimeCore`) that already holds an equivalent one (`persistence: PersistenceHandle`) — not a new kind of component, and not a name change for what would otherwise be a new registry (`GOV-016` §5's own ADR/amendment-sufficiency discipline; this amendment's own governing Safety Rule against inventing a global type registry casually).

This resolves the specific problem `EWO-025`'s blocker report identified, and does so without any dependency on Actor Host or a live instance: Runtime already knows the target `ActorId` before it ever calls Persistence Service to checkpoint or restore (§11); resolving that `ActorId` against Runtime's own, already-held, composition-time-supplied association is an ordinary, internal lookup — no cross-component call, no new interaction path (ADR-0016 Rule 2, unaffected), and no requirement that any actor instance, live or otherwise, exist yet. Decoding is therefore resolvable strictly before Actor Host is ever invoked (§15, unaltered — Actor Host's own construction step remains exactly where it already is, last).

**Encode**, unlike decode, occurs while a live instance exists (at checkpoint time, §16) — Runtime obtains both the actor's current `DomainState` and its associated contract (resolved from its own held association, as above) together, at the point it directs a checkpoint, exactly as it already obtains authorization and orchestrates every other cross-component persistence sequence (§11; ADR-0016 Rule 1).

**Unknown `ActorId` versus known `ActorId` with no associated contract, distinguished.** An `ActorId` Runtime has never composed a contract association for is architecturally distinct from — and MUST NOT be conflated with — an `ActorId` Actor Host has never `define`d at all (the latter, pre-existing condition is already `UnknownTarget`, per Actor Host's own unmodified interface; this amendment does not alter it). This amendment's own new failure category (§13.8, §18) governs only the former: a genuinely defined, addressable actor with no associated Durable-State Contract.

### 13.4 Encoding Ownership, Clarified (0.5.0; resolution owner corrected 0.5.1)

Unaltered: Persistence Service remains the sole owner of the encode/decode *operation* — sequencing it, applying it, constructing and verifying the envelope, and enforcing versioning (§10, §12; `ARCH-011` §9, unchanged). This amendment distinguishes, without changing, four previously-conflated concerns:

| Concern | Owner | Unchanged from |
|---|---|---|
| Holding the composition-time-supplied `ActorId`-keyed Durable-State Contract association, and resolving it for a given `ActorId` before requesting encode/decode | **Runtime** (0.5.1, §13.3) | New in 0.5.1 — corrects 0.5.0's own inaccurate citation of Actor Host as this owner (Finding M-1) |
| Orchestrating *when* a checkpoint or restore occurs, and supplying Persistence Service with the actor's `DomainState` and its resolved Durable-State Contract together | **Runtime** | §11 (unaltered — Runtime already "MUST be the sole component that decides when a checkpoint or restoration operation begins") |
| Applying the supplied contract's encode/decode operations, constructing and verifying the envelope, enforcing versioning, and reporting failure truthfully | **Persistence Service** | §10, §12 (unaltered — Persistence Service still "MUST NOT itself perform... authorization decisions," and gains no new authority by invoking a caller-supplied contract; §12's own 0.5.1 addition now states explicitly that Persistence Service MUST NOT resolve a contract independently) |
| Supplying the type-specific *encode/decode logic itself* — the contract's own two operations (§13.2) | **Whoever authors the actor type** (embedding code, via the Runtime-held association above) | Unaltered in substance since 0.5.0; only *where* it is held and resolved from was corrected in 0.5.1 |
| Storing and returning the resulting opaque, encoded bytes | **`StorageBackend`** | `ARCH-011` §9, unaltered — a backend never receives `DomainState` and never sees a Durable-State Contract |

Persistence Service reaches no actor instance, and no Actor Host interface, directly to obtain a contract (ADR-0016 Rule 2, unaltered) — Runtime resolves the contract from its own already-held, composition-time-supplied association (§13.3) and supplies it to Persistence Service as an ordinary argument, exactly as it already supplies the `DomainState` itself, never through a new peer-interaction path.

### 13.5 Persistent Type Identity — the Durable-State Kind Identifier (0.5.0)

A stable, actor-definition-assigned string (or equivalent stable value; concrete type deferred, §4, §24) — the **Durable-State Kind Identifier** — is stored in the envelope `ARCH-011` §9 already establishes, alongside the outer format-version identifier that section already defines. This identifier is deliberately distinct from, and MUST NOT be conflated with:

- **`ActorId`** (§8) — the durable *storage key*, identifying *whose* state this is, unrelated to *which contract* decodes it;
- **the outer envelope version** (`ARCH-011` §9, §12) — Persistence-Service-owned, identifying *which encoding scheme* (mechanics) was used, unrelated to which actor-owned contract applies;
- **the inner, domain-meaningful schema version** (§13.6 below; this document's own §13, unaltered) — actor-owned, identifying whether a given decoded representation remains logically acceptable to the actor's *current* reconstruction logic, a question the Durable-State Kind Identifier does not answer.

At restore time, Persistence Service reads the stored Durable-State Kind Identifier and compares it against the identifier of the contract Runtime supplied (§13.3, §13.4) for that `ActorId`. A mismatch — the identifier is absent, unrecognized, or does not match the currently-associated contract — is a distinct, truthfully-reported failure (§13.8; §18), never silently ignored and never treated as an ordinary version mismatch (§13.6).

### 13.6 Representation Version (0.5.0)

Unaltered from §13's own existing text and `ARCH-011` §12: the actor's own reconstruction logic remains the sole authority on whether a given decoded representation remains acceptable ("Version mismatch between a stored representation and the current actor | The actor's own reconstruction logic is the sole authority on acceptability," §18). This amendment adds only that the Durable-State Contract MAY itself carry a representation-version value, compared by the contract's own decode operation — not by Persistence Service, which remains ignorant of what that version means (§10, §12, unaltered) — and that an unsupported version MUST be rejected explicitly by the contract's own decode operation, distinctly reported (§13.8), never silently accepted or silently defaulted. No general, automated migration mechanism is introduced or required (§13.9; consistent with `ARCH-011` §12's identical stance).

### 13.7 Encoding Determinism (0.5.0)

A Durable-State Contract's decode operation, applied to bytes its own encode operation produced from a given `DomainState`, MUST reconstruct state that is behaviorally equivalent for the actor's own logic — this is the determinism this document requires. **Byte-for-byte determinism across repeated encodings of equivalent state is explicitly not required** — no normative statement in this document, `ARCH-011`, or DES-001 depends on it, and requiring it would constrain concrete codec implementation choices this document has no evidentiary basis to make (§4, out of scope for an Engineering Work Order to decide, not an architecture document). What is required, without exception: decode is never permitted to silently produce a *different*, unrelated, or partially-populated domain state from what was genuinely encoded — a decode that cannot faithfully reconstruct MUST fail explicitly (§13.8), never substitute a plausible-looking approximation.

### 13.8 Failure Semantics — New Categories (0.5.0; restore coverage corrected 0.5.1)

Extends §18's existing Failure Architecture table (below) with the categories a Durable-State Contract's own existence makes possible: no associated contract exists for the `ActorId`; the supplied `DomainState`'s concrete type does not match what the associated contract expects; the contract's own encode operation fails; the contract's own decode operation fails; the Durable-State Kind Identifier stored in the envelope is unknown or does not match the currently-associated contract (§13.5); the stored representation's version is unsupported by the current contract (§13.6). **Each of these applies wherever it is architecturally possible for the underlying operation to occur — "no associated contract," in particular, applies to both a requested checkpoint and a requested restore** (corrected in 0.5.1; the Independent Architecture Review's Minor finding on this point found §18's own v0.5.0 table row, and invariant 15, worded as checkpoint-only, inconsistent with this section's own already-general framing). A restore requested for an `ActorId` whose durable record exists, but whose current Runtime composition holds no associated contract for it (for example, after redeployment with different embedding code), fails explicitly and truthfully on this identical basis — never silently treated as "state absent" (§18). Every one of these MUST be truthfully, distinctly reported (§19) and MUST NOT be collapsed into an existing, unrelated failure category — consistent with §18's own existing discipline for "missing" versus "corrupted."

### 13.9 What This Amendment Defers (0.5.0)

Consistent with §4 and §24: the concrete Rust type, trait, or function signature realizing a Durable-State Contract; the concrete representation format (bytes, a specific structured encoding, or otherwise) a contract's encode operation produces; any specific serialization library or format (Serde, bincode, or otherwise); the concrete syntax by which embedding code supplies a contract to Runtime's own held, `ActorId`-keyed association at composition time (0.5.1, §13.3); a general, automated schema-migration mechanism; and any concrete Durable-State Kind Identifier value scheme (string, integer, UUID, or otherwise) beyond the stability requirement §13.5 states. None of these is required to resolve the architectural question `EWO-025` discovered missing (§13.2–§13.5); each remains a genuinely separate, bounded, future Engineering Work Order decision, exactly as §24 already treats "concrete Rust types, trait signatures, and method names" for the broader persistence boundary.

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

**Cross-reference, disclosed for completeness (0.5.2).** This section's own ordering governs Temporal Runtime's own pending timer registrations specifically (`ARCH-005` §23). `ARCH-008` §21 separately, and without redesigning this pattern, extends the identical ordering to a second, independent category of dependent state — pending Effect Attempts for the same `ActorId` — cancelled at the same point in the sequence (strictly before the deletion attempt is made) for the same reason. This document's own "without exception" ordering claim governs the steps this document itself defines; it does not foreclose `ARCH-008` §21's own, separately-authorized extension to a category of state this document does not itself address (§9 already excludes Effect Coordinator state from this document's own persisted-state boundary).

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
| No Durable-State Contract associated with the `ActorId` (0.5.0 §13.1; restore coverage corrected 0.5.1 §13.8) | A requested checkpoint fails explicitly; a requested restore likewise fails explicitly rather than treating the durable record as absent; the actor remains valid for ordinary, non-durable execution |
| Supplied `DomainState`'s concrete type does not match the associated contract (0.5.0, §13.2) | The checkpoint fails explicitly; MUST NOT be reported as a successful checkpoint of the wrong data |
| Durable-State Contract encode operation fails (0.5.0, §13.2) | The checkpoint fails; no partial or substitute representation is stored |
| Durable-State Contract decode operation fails (0.5.0, §13.2) | Treated as a restoration failure (above); the specific reason (below) MUST remain distinguishable in the audit record |
| Durable-State Kind Identifier unknown or mismatched (0.5.0, §13.5) | A distinct restoration failure, MUST NOT be conflated with "corrupted" or "missing" |
| Representation version unsupported by the current contract (0.5.0, §13.6) | A distinct restoration failure, MUST NOT be silently accepted or defaulted |

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
15. A durable checkpoint or restore MUST NOT succeed for an `ActorId` with no associated Durable-State Contract (0.5.0 §13.1; corrected 0.5.1 to cover restore as well as checkpoint, §13.8).
16. No persisted representation may depend on a memory address, an in-process pointer, an in-process `TypeId`, or any other value meaningless outside the process and allocation that produced it (0.5.0, §13.2).
17. Runtime, not Actor Host and not any newly introduced component, holds the composition-time-supplied `ActorId`-keyed Durable-State Contract association; no new registry, manifest, discovery protocol, or Trusted Core component is introduced (corrected 0.5.1, §13.3 — replaces v0.5.0's own inaccurate citation of Actor Host).
18. `StorageBackend` never receives, and never returns, `DomainState` or a Durable-State Contract (0.5.0, §13.4; unaltered from `ARCH-011` §9).
19. The Durable-State Kind Identifier, the outer envelope version, and the actor-owned inner schema version remain three distinct values; none may be inferred from, or conflated with, either of the other two (0.5.0, §13.5, §13.6).
20. Durable-State Contract resolution for a given `ActorId` MUST be possible without constructing, or requiring the prior existence of, any live actor instance (0.5.1, §13.3).
21. Persistence Service MUST NOT resolve, discover, or select a Durable-State Contract on its own; it MUST use only the contract Runtime supplies with a given request (0.5.1, §12, §13.4).

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
- **The exact Rust-level interface, trait signature, or method names by which an actor supplies a Durable-State Contract, and by which embedding code supplies it to Runtime's own held association** (§13.2–§13.9, 0.5.0–0.5.1) — the *conceptual* contract is now fixed (durability eligibility, encoding ownership, Runtime-held composition-time association, persistent type identity, versioning, determinism, and failure semantics); the concrete language-level mechanism realizing it remains implementation-phase, on the same basis ARCH-004 fixed no Rust signatures and `ARCH-011` §21 defers "concrete Rust types, trait signatures, and method names."
- **Any specific serialization format or library** (Serde, bincode, or otherwise) a Durable-State Contract's encode operation might use internally (§13.9, 0.5.0) — not fixed at the architecture level; a future Engineering Work Order's own implementation choice.
- **The concrete Durable-State Kind Identifier value scheme** (string, integer, UUID, or otherwise) beyond the stability requirement §13.5 (0.5.0) states.
- **Version-envelope ownership** (outer format-version tag: Persistence Service versus actor-owned) — DES-001 identified this as an open question for ARCH-007; not resolved by this document.
- **Concurrency and checkpoint coordination for the same logical actor** (§18) — genuinely unresolved; a future architecture or implementation decision must either exclude this case structurally or introduce a coordination primitive this document does not define.
- ~~**Runtime-startup restoration policy** — whether previously persisted actors are automatically reattached on Runtime startup, or restoration is always explicit and on-demand.~~ **Resolved, not by this document but by `ARCH-011` §11** (Approved, v0.1.3, citing `IPR-003` §14 Q2 and `ARCH-008` §22's precedent): restoration is always explicit and on-demand, never automatic as a side effect of Runtime startup; no implementation of this architecture may cause a Runtime boot sequence to automatically reconstruct a live instance from a discovered stored representation without an explicit instruction to do so. Retained here, struck through rather than deleted, to preserve the record that this was once an open item this document itself left deferred.
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
- ADR-0018 — StorageBackend Serialization Boundary (0.5.0 — the ownership boundary this amendment supplies the missing actor-facing half of)
- ARCH-008 — Effect Runtime Architecture (0.5.0 — §11.2's "no new registry" discipline, cited by §13.3; 0.5.1 corrected the specific resolution mechanism to a Runtime-held association rather than an Actor-Host-definition analogy)
- ARCH-011 — Durable Storage Mechanics (0.5.0 — v0.1.1, Founder Approved; §9's envelope/backend boundary this amendment's contract sits above)
- DES-001 — Persistent Actor Design Exploration (v0.2.0), the sole design authority this document codifies
- AFR-001 — Architecture Baseline Certification

Source evidence (verified during design exploration and independently re-confirmed for this document, not restated from memory):

- `api/src/lib.rs` (`Actor` trait — currently no serialization capability, confirming §4/§13's scope boundary)
- `services/persistence/src/lib.rs`, `src/internal.rs` (existing `Persistence` trait stub — confirmed unimplemented, confirmed keyed by `ActorInstanceId`, confirmed non-authoritative per DES-001 §"Architectural Consequences")
- `core/actor-host/src/internal.rs` (`ActorHostImpl` field layout — confirms actor-defined behavior state exists and is currently opaque)
- `runtime/src/lib.rs` (`restore_actor_instance` — confirms the existing in-memory restore path does not yet perform capability revalidation, the precondition §15 requires closed before any durable restore implementation)
- `common/src/lib.rs` (0.5.0 — `DomainState(pub Rc<dyn Any>)`, independently re-confirmed to carry no serialization bound of any kind; direct evidence for §13.2's own stated necessity of an actor-supplied contract)
- Full-repository search for `serde`/`Serialize`/`Deserialize`/`Codec`/`TypeId` across every `.rs` and `Cargo.toml`/`Cargo.lock` file (0.5.0 — zero results, confirming no existing serialization infrastructure exists to build on; performed independently by `EWO-025`'s own blocker report and re-confirmed during this amendment's own preparation)
- `core/actor-host/src/lib.rs` (0.5.1 — the full, current `ActorHost` trait, re-read directly rather than assumed: `define` assigns only a stable identity from a name string and stores no reusable construction or behavior data; `create_instance_with_behavior`'s own doc comment confirms behavior is "supplied already constructed by the caller, not built from a stored factory." Direct evidence that v0.5.0's own citation of Actor Host as the contract-resolution owner was inaccurate — the specific finding this amendment corrects)
- `runtime/src/lib.rs` (0.5.1 — `RuntimeCore`'s own `persistence: PersistenceHandle` field, confirmed at three construction sites; direct evidence that Runtime already holds exactly this kind of composition-time-supplied replaceable-service configuration, the concrete precedent this amendment's corrected §13.3 relies on in place of the Actor Host analogy)

## 27. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-26 | Denver Jacobs | Initial Draft. Establishes the Persistent Actor Architecture: identity model (`ActorId`-keyed), persistent state model (domain state only), ownership model (actor/Persistence Service/Runtime/Capability Authority), Runtime/Persistence Service/Actor/Capability Authority responsibilities, restore architecture with mandatory capability revalidation, persistent state lifecycle, deletion and retention architecture, failure architecture, audit architecture, security architecture, fourteen architectural invariants, and conformance requirements. Completes the deferral recorded in ARCH-002 §6 and §23 ("Storage Architecture"). Codifies DES-001 v0.2.0 (DAR-001-approved) without introducing new design. |
| 0.2.0 | 2026-07-26 | Denver Jacobs (AI-assisted) | Targeted correction pass applying, and limited to, the findings of the independent Architecture Review of ARCH-007. No architectural decision was reopened, no ownership boundary altered, and no implementation detail introduced. (1) **Ownership Model table (§10)** — corrected all four internal cross-references, each previously pointing to the wrong section (Actor §11→§13; Persistence Service §10→§12; Capability Authority §12→§14; Runtime §9→§11). (2) **Required section structure** — added §22 (Replaceability) and §23 (Compatibility with Existing Architecture) as standalone sections, reusing existing, already-approved material from §5 and §12 without introducing new architectural content; Deferred Decisions, Conformance Requirements, References, Change History, and Approval Status renumbered §24–§28 accordingly. (3) **Infrastructure coupling prohibition** — added an explicit MUST NOT statement to §22 (Replaceability) and to §25's Prohibited list, carrying forward DES-001's own rejection of infrastructure-specific persistence coupling as a normative architectural prohibition. (4) **Deferred Decisions (§24)** — added "Version-envelope ownership" and "Failed or partial deletion representation" as explicitly deferred items, matching DES-001's own "Deferred to ARCH-007" list; neither is resolved. (5) **Internal cross-references** — performed a complete validation pass over every internal section citation in the document and corrected every remaining incorrect one found, beyond the Ownership table alone: §1 (Document Control and Status table's own Version field, unsynchronized with this correction's version bump); §§3–4 (Non-Goals/Scope: concurrency and event-sourcing citations); §5 (one ambiguous bare citation disambiguated as `ARCH-002 §9`, and three self-references to Persistence Service Responsibilities that had been left citing §9 from an earlier section count, corrected to §12); §7 (a self-reference reading "§7 below," inside §7 itself, corrected to "§9 below"); §9 (a citation to Runtime Responsibilities corrected to Actor Responsibilities, §13); §§11–14 (Runtime, Persistence Service, Actor, and Capability Authority Responsibilities: capability-revalidation, authorization-obtaining, encoding/decoding-ownership, authorization-ownership, and audit citations); §15 (restore-architecture diagram's audit citation); and §17 (deletion authorization and auditability citations). No citation to an external document (ARCH-001, ARCH-002, ARCH-004, ARCH-005, ADR-0015, ADR-0016, DES-001, STD-001) was altered. |
| 0.3.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Targeted correction pass resolving durable-restoration validation triggering, identified by the Independent Implementation Review of the Persistent Actors implementation (finding M-1) and completed across two correction passes. No architectural decision was reopened and no new component, capability, or interface was introduced. (1) **Ownership Model (§10)** — split the single "Triggering restoration validation" row into two: Lifecycle Guardian for the existing in-memory Suspended/Resume case (unchanged, ARCH-002 §9), and Runtime for durable restore, with an added explanatory paragraph. (2) **Restore Architecture (§15)** — corrected the restore diagram and its explanatory text: durable-restore capability revalidation is triggered by Runtime directly, not by Lifecycle Guardian, because no `ActorInstanceId` exists yet for the restored incarnation at the point revalidation must occur; Lifecycle Guardian's role begins once Actor Host creates and registers the new live instance. (3) **Compatibility with Existing Architecture (§23)** — corrected the ARCH-002 compatibility statement to split ARCH-002 §9's "joint act" into its Capability-Authority-performs half (extended unaltered to both cases) and its Lifecycle-Guardian-triggers half (extended only to the resume case ARCH-002 §9 already scopes it to). (4) **Capability Authority Responsibilities (§14)** — corrected the one surviving internal contradiction: the requirement previously stated capability-binding revalidation was "triggered by Lifecycle Guardian... for the durable case," contradicting the corrected §10/§15/§23; reworded to state the trigger source differs by restoration type (Lifecycle Guardian for resume, Runtime for durable restore) while Capability Authority performs and decides revalidation in both cases, unchanged. Root cause: ARCH-002 §9 scopes Lifecycle Guardian's triggering responsibility to an already-tracked `ActorInstanceId`'s `Suspended → Idle/Ready` transition; durable restoration is a case ARCH-002 does not address, and citing ARCH-002 §9 as unqualified authority for it was the specific defect corrected here. ARCH-002 itself is unamended and unaltered throughout. |
| 0.4.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Targeted correction, identified by the Persistent Actor and Temporal Runtime Integration Architecture Review, closing the previously undocumented relationship between durable-state deletion (§17) and Temporal Runtime's own pending timer registrations for the same `ActorId` (ARCH-005, which did not exist in its present form when this document's deletion architecture was first authored). Added one bullet to §17's governing list requiring Runtime to proactively cancel every pending Temporal Runtime timer registration for the same `ActorId` as part of durable-state deletion, cross-referencing the corresponding extension made to ARCH-005 §23 in the same correction effort. Added one new paragraph, "Deletion coordination ordering," fixing the required sequence (validate authority → cancel timers → audit cancellations → attempt deletion → audit outcome) and its failure semantics: cancellation is infallible under the existing mechanism and requires no invented failure mode; a subsequent deletion failure leaves durable state intact and timers already, harmlessly, cancelled; no compensation may fabricate restoration or automatic timer re-registration; the coordination is truthful and ordered, not transactionally atomic, and must not be described as such. Explicitly states this correction does not apply to checkpointing, ordinary restart, or restoration, and does not authorize durable timers, timer persistence, or automatic timer re-arming — all of which remain separate, unresolved future architecture concerns. No other architectural decision was reopened: §10, §14, §15, and §23 (the subject of the 0.3.0 correction) are unchanged; Persistence Service and Temporal Runtime remain, and are confirmed to remain, mutually unaware of one another; Runtime remains the sole coordinator of the extended sequence. |
| 0.5.0 | 2026-07-30 | Denver Jacobs (AI-assisted) | Architecture Amendment resolving the actor-facing encoding gap `EWO-025`'s own Architectural Blocker Report discovered: `DomainState` (`Rc<dyn Any>`) carries no serialization bound, no codec, type-registry, or reconstruction mechanism exists anywhere in the repository, and §13's own prior text left "the exact interface... by which an actor supplies and reconstructs its own domain-state representation... explicitly not fixed." Added §13.1–§13.9 (Durability Eligibility; the Durable-State Contract; Contract Association — reusing `ARCH-008` §11.2's definition-time mechanism rather than inventing a new registry; Encoding Ownership, Clarified — distinguishing orchestration, mechanical application, type-specific knowledge, and byte storage as four separate concerns with four separate, unchanged-or-newly-assigned owners; the Durable-State Kind Identifier, kept distinct from `ActorId`, the outer envelope version, and the inner schema version; Representation Version; Encoding Determinism; new Failure Semantics categories; and an explicit list of what remains deferred). Added six new failure-table rows to §18. Added five new invariants (15–19) to §21. Updated §24's Deferred Decisions to reflect that the *conceptual* contract is now fixed while concrete Rust signatures, serialization-library choice, and the concrete kind-identifier value scheme remain implementation-phase, consistent with `ARCH-011` §21's identical deferral pattern. **No prior architectural decision was reopened:** Persistence Service's ownership of the encode/decode *operation* (§10, §12), the `StorageBackend` byte-only boundary (`ARCH-011` §9, `ADR-0018`), the `ActorId`-keyed identity model (§8), capability revalidation on restore (§15), and every invariant 1–14 remain unaltered in meaning — this amendment adds exactly one missing layer beneath an unchanged ownership model. No SQLite, LMDB, concrete serialization format, or migration framework was designed, consistent with this amendment's own explicitly bounded scope. |
| 0.5.1 | 2026-07-30 | Denver Jacobs (AI-assisted) | Corrective amendment resolving all three findings of the Independent Architecture Review of v0.5.0 (Major Finding M-1; two Minor findings). **M-1 (Major, resolved):** v0.5.0's §13.3 cited Actor Host, by analogy, as the component resolving a Durable-State Contract during restore, while §13.4 separately assigned contract-supply to Runtime — an unreconciled inconsistency, and, on direct re-investigation of `core/actor-host/src/lib.rs`, an inaccurate one: `ActorHost::define` stores no reusable construction or behavior data, so there was never a "definition" for it to expose. Corrected by rewriting §13.3 in full: Runtime itself — evidenced by `RuntimeCore`'s own existing `persistence: PersistenceHandle` field, already an identical composition-time-supplied replaceable-service association — now holds the `ActorId`-keyed Durable-State Contract association, resolved internally without any call to Actor Host and without requiring a live instance. §13.4's ownership table gained a new row naming Runtime as this resolution owner explicitly. §11 gained an explicit MUST-bullet for this responsibility; §12 gained an explicit MUST NOT-bullet foreclosing independent resolution by Persistence Service. Invariant 17 (§21) rewritten to state the corrected owner; new invariants 20–21 added, closing the "resolvable without a live instance" and "Persistence Service never self-resolves" gaps the review's own invariant-by-invariant check identified as implied but unstated. **Minor Finding 1 (resolved):** §13.8, the §18 failure-table row, and invariant 15 were checkpoint-only in wording despite §13.8's own general framing; all three broadened to state the restore case explicitly. **Minor Finding 2 (resolved):** §11 and §12 each gained the explicit MUST-bullet described above, ending reliance on §13's own narrative prose to convey a normative responsibility. **No prior sound decision was reopened:** the Durable-State Contract's own conceptual shape (§13.2), the Kind Identifier (§13.5), Representation Version (§13.6), Determinism (§13.7), and the "no new registry" conclusion (§13.3, re-affirmed on stronger evidence) are unchanged in substance — only *where* resolution is held and *who* performs it were corrected. No Runtime code was modified; no unrelated documentation was touched. |
| 0.5.2 | 2026-08-05 | Claude (AI-assisted) | Targeted correction pass applying, and limited to, the four findings (F-1–F-4) of the Independent Architecture Re-Review of ARCH-007 v0.5.1. No architectural decision was reopened, no ownership boundary altered, no invariant added or removed, and no scope expanded. **F-1** (frontmatter) — `ARCH-008` citation corrected from "Draft" to "Approved v0.5.0," matching its own current frontmatter. **F-2** (frontmatter; §6 body) — `ARCH-011` citations corrected from "v0.1.1, Draft — Founder Architecture Approval Pending, FAA-011 not filed" to "v0.1.3, Approved — Founder Architecture Approval recorded as FAA-011," matching its own now-Approved, published state; the §6 sentence describing `ADR-0018`'s reliance on this premise was correspondingly updated to reflect that `ARCH-011`'s approval now confirms it, while noting `ADR-0018` itself remains a separate, still-unapproved Draft. **F-3** (§24) — the "Runtime-startup restoration policy" deferred item, struck through rather than deleted, now discloses that `ARCH-011` §11 (Approved, v0.1.3) has already resolved this question; the decision itself is not reopened or restated as new. **F-4** (§17) — added one cross-reference paragraph disclosing that `ARCH-008` §21 extends this section's own deletion-ordering pattern to a second, independent category of dependent state (pending Effect Attempts), without altering this document's own ordering, semantics, or scope. All four corrections are citation- and cross-reference-scoped; the Persistent Actor model, the Durable-State Contract, all 21 invariants, and every ownership boundary remain unchanged in substance. |

## 28. Approval Status

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted (v0.1.0); Amended (v0.5.0); Corrected (v0.5.1); Corrected (v0.5.2) | 2026-07-26; 2026-07-30; 2026-07-30; 2026-08-05 |
| Technical Review | TBD | v0.5.0 reviewed (Independent Architecture Review: 1 Major, 2 Minor findings, all addressed in v0.5.1). v0.5.1 reviewed (Independent Architecture Re-Review: 3 Major findings F-1–F-3, 1 Minor finding F-4, all addressed in v0.5.2). v0.5.2 reviewed (Final Independent Architecture Re-Review: `VERIFIED`, `NO ARCHITECTURAL REGRESSION DETECTED`, `READY FOR FOUNDER ARCHITECTURE APPROVAL`; one new Minor finding, F-5, recorded as a non-blocking observation, not resolved by further amendment). No review in this chain is itself an independently filed repository artifact — all are AI-conducted under Denver Jacobs' direction, disclosed as such consistent with this corpus's own practice. | 2026-07-30; 2026-08-05; 2026-08-06 |
| Approval Authority | Denver Jacobs, Founder, exercising the interim Class B default (GOV-003 §3.2) | **Approved, with non-blocking observations** — Founder Architecture Approval recorded as `FAA-012`. Denver Jacobs' own decision, recorded verbatim: "I, Denver Jacobs, acting as Founder of SynapseOS, have reviewed the complete ARCH-007 v0.5.2 architecture lifecycle, including the architecture document, amendment history, independent reviews, final re-review, repository evidence, and remaining observations. I adopt the recommendation of the completed review chain as my own Founder decision. I approve ARCH-007 v0.5.2 as the authoritative SynapseOS architecture for Persistent Actor Architecture, with the following non-blocking observations formally recorded, not requiring amendment before this approval takes effect: (1) §17 contains a parenthetical cross-reference that should cite ARCH-008 §22 rather than §9. The architectural statement remains correct; only the supporting citation requires correction. (2) §26 References still cites ARCH-011 as 'v0.1.1, Founder Approved' — stale on the version number (currently v0.1.3) — a defect outside this amendment's own assigned scope, noted for future correction. This decision authorizes repository filing, controlled commit, and publication through the established SynapseOS publication process. This decision does not itself commit, push, or publish repository content." | 2026-08-06 |

This document is now **Approved**, following Founder Architecture Approval (`FAA-012`) recorded directly above, on the identical "ordinary, mutable Approval Status convention" this repository's most-recently-approved documents use throughout (demonstrated: `ARCH-008`, `ARCH-010`, `ARCH-011`, each populated in place, without requiring a version-number change as a precondition). `ARCH-007` is hereby adopted as the authoritative Persistent Actor Architecture for SynapseOS: future implementation shall conform to it; future architectural modification shall occur only through the established governance process; no implementation may intentionally diverge from it without an approved ADR or a subsequent architecture amendment. The two non-blocking observations recorded above remain open, tracked matters — neither blocks this approval nor requires resolution before implementation may proceed.
