---
document_id: DES-001
title: "Persistent Actor Design Exploration"
version: 0.2.0
status: Draft
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
created: 2026-07-26
last_updated: 2026-07-26
classification: Public
document_family_note: >
  "DES" (Design Exploration) is not currently a registered controlled
  document family in STD-001 Appendix B. This document is placed in
  `consolidation/` — the narrowest existing, purpose-consistent
  location (STD-001 §10), on the same functional basis RSS/ACR/AFR
  already occupy it: an evidence-to-decision synthesis document that
  precedes, and directly informs, a later binding artifact (here,
  ARCH-007) without itself being architecture, governance, or an
  engineering authorization. This placement is a disclosed, narrow
  convenience, not a documentation-hierarchy redesign; formal
  registration of a "DES" family in STD-001, if ever wanted, is a
  separate, future, independently-authorized task, not performed
  here or implied by this placement.
related_documents:
  standards:
    - STD-001
  governance:
    - ADR-0016
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.1 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-004 (v0.1.0 — architecture/ARCH-004-Local-Actor-Supervision-Architecture.md)
    - ARCH-005 (v0.1.0 — architecture/ARCH-005-Temporal-Runtime-Architecture.md)
  consolidation:
    - AFR-001 (v0.1.1 — consolidation/AFR-001-Architecture-Freeze-Review.md)
  predecessor: The completed Persistent Actor Architecture Review (conversational record; not a filed repository document)
  reviewed_by: DAR-001 — Design Approval Review of DES-001 (conversational record; not a filed repository document); verdict on v0.1.0 — DESIGN CHANGES REQUIRED (four findings, all addressed in this v0.2.0)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# DES-001 — Persistent Actor Design Exploration

*Design exploration only. No architecture is authored here. No implementation is authorized here. This document is intended as direct input to a future ARCH-007.*

> **Status notice.** This document is **Draft**. No approval act has occurred. It has completed one design-approval review (DAR-001) at v0.1.0, which returned **DESIGN CHANGES REQUIRED** on four precisely identified findings. This v0.2.0 applies exactly those four corrections and no other design change. The core recommended direction is unchanged from v0.1.0.

---

## Executive Summary

The governing question is not "how do we persist actors" but **"what does it mean for a SynapseOS actor to exist over time?"** SynapseOS has already, independently, answered a version of this question three times — for actor identity (`ActorId` survives, `ActorInstanceId` does not, ARCH-002 §7), for supervision state (Supervisor's hierarchy and restart accounting are `ActorId`-keyed, ARCH-004 §12/§15), and for temporal registrations (Timer entries are `ActorId`-keyed, ARCH-005 §11). A comparative study of seven industry systems (Orleans, Erlang/OTP, Akka Persistence, Service Fabric Reliable Actors, Dapr Actors, Temporal, Ray) finds the identical pattern — a persistent logical identity separated from an ephemeral execution incarnation — independently converged upon in **every single system studied**, without exception. Persistent Actors is not a new architectural idea for SynapseOS; it is the natural extension of an already-proven, three-times-repeated internal pattern to a fourth category of `ActorId`-keyed data: an actor's own domain state.

The recommended design is: whole-snapshot persistence (not event sourcing) as the first milestone, owned by a new, narrow, `ActorId`-keyed extension to the existing (currently unimplemented) Persistence/Restoration Service, with actors opting in via a typed, framework-encoded state contract (not raw actor-supplied bytes), Runtime as sole orchestrator, and Capability Authority holding exclusive, mandatory revalidation authority on every restore — closing a currently-disclosed implementation gap first. Event sourcing is explicitly named as a compatible, not-precluded future extension for workflow and long-running-agent use cases, not designed here.

**Changes in this revision (v0.2.0), applying DAR-001's four findings exactly:** the illustrative pseudo-API in the serialization-contract discussion has been removed and replaced with purely conceptual language; the Ownership model has been corrected so the actor is never described as producing encoded bytes; a new "Persisted-State Deletion and Retention" section has been added; a new "Capability Authorization for Persistence Operations" section has been added. No other content changed. The core persistence direction — `ActorId` keying, whole-snapshot-first, actor-supplied structured state, Runtime-orchestrated, Capability-Authority-validated — is unchanged and not reopened.

---

## Current SynapseOS Baseline

*(Independently re-verified against `synapse-docs`/`synapse-runtime` current state; not restated from memory alone.)*

- **Constitutional layering.** ARCH-001 §11: all persistence mechanics belong to the Runtime/Infrastructure layer, never the constitutional layer; ARCH-007 may not redefine Actor, Capability, Message, or Execution Semantics.
- **Already-named architectural slot.** ARCH-002 §6 already names a **Persistence/Restoration Service** as a replaceable, non-Trusted-Core service ("performs the mechanical snapshot/replay I/O that Lifecycle Guardian validates... MUST NOT reinstate authority Lifecycle Guardian has not revalidated"), and §23 already names the deferred document — **Storage Architecture** — with a fixed contract: the Persistence/Restoration Service interface boundary, and "no silent reinstatement of revoked authority." ARCH-007 fills an already-scoped slot; it does not invent one.
- **Identity split, already established.** `ActorId` (`common/src/lib.rs`) is stable across suspension, resumption, and restart; `ActorInstanceId` changes on every restart and is never reused (ARCH-002 §7; ARCH-004 §12). Capability bindings, Supervisor's hierarchy, and Timer's registrations are **all already `ActorId`-keyed**, specifically because each must survive the one event (restart) persistence exists to survive.
- **The `Persistence` trait exists but is an unimplemented stub**, keyed by `ActorInstanceId` (`services/persistence/src/lib.rs`) — the wrong key for anything meant to survive a restart, and not composed into `Runtime` at all.
- **The `Actor` trait has no serialization capability** (`api/src/lib.rs`: exactly one method, `handle`).
- **Capability revalidation on restore is architecturally required (ARCH-002 §9: "a joint act — Lifecycle Guardian triggers it on resume; Capability Authority performs it") but not implemented** (`Runtime::restore_actor_instance`, `runtime/src/lib.rs`, calls only Lifecycle Guardian, never Capability Authority — its own doc comment discloses this directly).
- **ADR-0016 Rule 1/Rule 2:** "The Runtime is the sole architectural entity accountable for establishing and coordinating Trusted Core interactions... Trusted Core components must not independently establish or own direct peer interaction paths." Any new persistence-triggered cross-component sequence must be Runtime-mediated, never direct — the exact rule EWO-007 already applied when introducing Supervisor.
- **Precedent for how a new replaceable service is introduced and for how authorization is scoped.** ARCH-004 (Supervisor) established the template this document reuses directly, including its explicit authorization-boundary statement: "Supervisor does not grant authority. It has no path to Capability Authority (§10.2) and holds no capability-issuing power of its own." (ARCH-004 §17.)

---

## Comparative Analysis

*(Unchanged from v0.1.0 — not reopened by DAR-001. Retained here in full for continuity of evidence.)*

For each system: persistence / ownership / state / recovery / durability / failure / lifecycle / identity / serialization models, then strengths, weaknesses, hidden assumptions, scalability, operational complexity, and alignment with SynapseOS.

### Microsoft Orleans (virtual actors / grains)

| Dimension | Model |
|---|---|
| Persistence | Explicit `WriteStateAsync`/`ReadStateAsync` on an injected `IPersistentState<T>`; pluggable storage providers |
| Ownership | Grain decides *when*; storage provider owns *how*; framework owns serialization of a typed state object |
| State | Single (or multiple named) typed POCO state object(s) per grain; default is whole-object overwrite, with ETag-based optimistic concurrency |
| Recovery | "Virtual actor" activation: state loads automatically on first call after (re)activation; a grain is always logically addressable even before activation |
| Durability | Entirely delegated to the chosen storage provider |
| Failure | Write failure throws to grain code; no built-in retry/WAL |
| Identity | Grain ID (persistent) vs. activation (ephemeral) |
| Serialization | Framework serializes a **typed state object the grain supplies**, not raw bytes |

**Strengths:** mature at extreme scale; clean storage-technology abstraction; the always-addressable virtual-actor illusion is elegant. **Weaknesses:** whole-object overwrite is inefficient for large, frequently-mutated state; silent reactivation can obscure *when* restore actually happens. **Hidden assumption:** state object is small enough to hold fully in memory. **Alignment:** grain-ID/activation split is essentially identical to `ActorId`/`ActorInstanceId`; "framework serializes a supplied typed object" directly informs the serialization-contract recommendation below.

### Erlang/OTP

| Dimension | Model |
|---|---|
| Persistence | **None, by design.** Durable state, if wanted, is externalized to Mnesia/ETS/an external DB, entirely by application code |
| Ownership | Entirely the application's |
| State | Arbitrary process term; framework never touches it |
| Recovery | Supervisor restarts the process with a fresh `init/1`; if "recovery" is wanted, `init/1` itself re-reads external storage |
| Durability | Whatever Mnesia/ETS/DB provides |
| Failure | "Let it crash" — no defensive handling of corrupted state; correctness comes from idempotent, from-scratch re-derivation |
| Identity | Pid (ephemeral) vs. registered name (stable) — a close analog to `ActorInstanceId`/`ActorId` |

**Strengths:** decades of proven reliability; forces idempotent recovery design, eliminating an entire bug class (stale-state corruption). **Weaknesses:** zero framework guidance — every team reinvents persistence; poor fit for state that is expensive to recompute. **Hidden assumption:** state can always be cheaply rebuilt externally. **Alignment:** the *default* "restart with fresh state, no automatic persistence" behavior directly validates Supervisor's own existing default (ARCH-004 §14); the complete absence of specification conflicts with SynapseOS's own precisely-bounded-component culture — **rejected as a model to copy**, partially validated as a philosophical default.

### Akka Persistence

| Dimension | Model |
|---|---|
| Persistence | **Event sourcing.** `persist(event)(handler)` appends immutable domain events to a journal; state is `fold(events, initial)` |
| Ownership | Actor decides what events to persist and how to fold them; journal plugin owns durable append-only storage |
| State | Not stored directly — derived by replaying events; snapshots optionally accelerate replay |
| Recovery | Automatic deterministic replay of all events since the last snapshot before the actor accepts new commands |
| Durability | Very strong — the full history is preserved, not just current state |
| Failure | `onPersistFailure` typically stops the actor (cannot safely continue with unpersisted state); `onPersistRejected` for validation-level rejection |
| Identity | `PersistenceId` (stable) vs. `ActorRef` path (can vary) |
| Serialization | Each **event type** requires a registered serializer (Protobuf/JSON/custom) |

**Strengths:** best-in-class auditability — the entire causal history is always reconstructable, directly matching SynapseOS's own audit-truthfulness culture (ADR-0015); replay determinism is *identical* to the determinism requirement this task states explicitly. **Weaknesses:** event-schema evolution is a well-known, hard operational problem; unbounded journal growth requires active snapshot/compaction management; meaningfully higher conceptual overhead. **Hidden assumption:** every state change can be modeled as a discrete, replayable event with no non-deterministic logic inside the fold function. **Alignment:** the single strongest philosophical match on the audit dimension of any system studied; too heavy as a **first** milestone given ARCH-002 §21's own Minimal-Runtime-Profile-first philosophy — **recommended as a named future extension, not the initial model.**

### Service Fabric Reliable Actors

| Dimension | Model |
|---|---|
| Persistence | Reliable State Manager: state changes synchronously quorum-replicated across a primary+secondary replica set before acknowledgment |
| Ownership | Framework owns replication/durability entirely; actor calls typed dictionary-like APIs |
| State | Key-value bag of serializable objects |
| Recovery | Already-replicated locally on failover — very fast |
| Durability | Quorum-based; optional backup for full cluster-loss durability |
| Identity | ActorId is the sole addressing/persistence key |
| Serialization | Framework `DataContract` serializer (pluggable) applied to typed objects |

**Strengths:** very low recovery latency (no network round-trip needed, data already local). **Weaknesses:** tightly coupled to a specific, heavyweight cluster/replication infrastructure — the least portable of all seven systems; declining investment/adoption. **Hidden assumption:** a Service-Fabric-managed cluster is always available. **Alignment:** infrastructure lock-in directly conflicts with ARCH-002 §22's explicit "Permitted variation: persistence technology" — **rejected.**

### Dapr Actors

| Dimension | Model |
|---|---|
| Persistence | Pluggable "state store" component (Redis, Cosmos, DynamoDB, ...) reached through a sidecar process via HTTP/gRPC |
| Ownership | Application decides what/when; sidecar + state-store component own mechanics — a genuinely replaceable, pluggable boundary |
| State | Key-value, arbitrary serializable values |
| Recovery | Virtual-actor-style activation-triggered load, explicitly modeled on Orleans |
| Identity | ActorId, same split pattern |
| Serialization | Typically JSON via SDK, pluggable |

**Strengths:** infrastructure/language-agnostic; the sidecar/pluggable-state-store pattern is, of all seven systems, the **closest philosophical match** to SynapseOS's own "replaceable Runtime service reached through a defined boundary" principle (ARCH-002 §6, §19). **Weaknesses:** network hop adds latency to every state operation; consistency guarantees vary by backend with no enforced floor. **Hidden assumption:** the sidecar is always co-located and healthy. **Alignment:** strongest single influence on the *ownership/boundary* shape of the recommendation below.

### Temporal (workflow durable execution)

| Dimension | Model |
|---|---|
| Persistence | Event-sourced **workflow history**, durably stored server-side; workflow code is deterministically re-executed against recorded history on every worker pickup |
| Ownership | Server owns durable history; SDK/worker owns deterministic replay; workflow author must write **strictly deterministic** code (all side effects routed through specific SDK primitives) |
| State | Implicit — local variables reconstructed by replay, never explicitly serialized |
| Recovery | Full deterministic replay (accelerated by sticky-execution caching) |
| Durability | Extremely strong; designed for processes lasting days to years |
| Failure | Activities have explicit, configurable retry; "Continue-As-New" bounds unbounded history growth |
| Identity | WorkflowId (persistent, reusable) vs. RunId (per-execution) |
| Serialization | Only activity/signal inputs-outputs are serialized; workflow-local state itself is never serialized |

**Strengths:** unmatched, *enforced* determinism discipline; purpose-built for exactly the long-running orchestration use cases this task names explicitly (Workflow Runtime, scheduled jobs, long-running AI actors, autonomous agents). **Weaknesses:** replay-determinism is a real, unforgiving programming-model constraint; "non-determinism errors" from code changes are a well-documented operational hazard; heavier than needed for small, simple, frequently-mutated actor state. **Hidden assumption:** all side effects can be fully isolated behind SDK-controlled primitives. **Alignment:** exceptionally strong for the *future* capabilities named in the Future Compatibility section below — **explicitly named as a compatible second-milestone extension, not the first.**

### Ray (distributed compute actors)

| Dimension | Model |
|---|---|
| Persistence | Optional, purely application-supplied checkpoint/restore hooks; no framework storage abstraction |
| Ownership | Entirely the actor author's |
| State | Arbitrary, unconstrained object state |
| Recovery | Fresh restart; app-supplied restore logic runs if present, otherwise default/empty state |
| Failure | `max_restarts` bounds retry attempts |
| Identity | Weakest formalization of the seven — named/detached actors provide loose persistent addressability |

**Strengths:** maximum flexibility, minimal framework overhead; familiar to ML-checkpoint-literate teams. **Weaknesses:** no architectural guidance whatsoever. **Hidden assumption:** authors will correctly implement their own checkpoint logic, with no framework safety net. **Alignment:** `max_restarts` is directly, independently precedented by Supervisor's own `RESTART_ALLOWANCE` constant (EWO-007) — a validating coincidence, not new information; the "no framework guidance" posture is **rejected** for the same reason as Erlang/OTP.

### Cross-System Synthesis

**The single most important finding of this comparative study:** all seven systems — designed independently, for different purposes, over three decades — converge on the identical structural answer to "what does it mean to exist over time": **a persistent logical identity, separated from an ephemeral execution incarnation.** SynapseOS's `ActorId`/`ActorInstanceId` split (ARCH-002 §7, already published before this exploration began) is not a novel design choice requiring justification from scratch — it is independently validated by every comparable system studied.

---

## Design Alternatives

### 1. What is Actor State?

| Category | Definition | Persist? | Why |
|---|---|---|---|
| Domain state | The actor's own accumulated, application-meaningful data (currently opaque inside `Box<dyn Actor>`) | **Yes, when opted in** | This is the entire point of the feature |
| Framework/identity state | `ActorId`, capability bindings | **Already persistent by design** | Pre-existing, unaffected — `ActorId`-keyed storage, Capability Authority |
| Execution state | `ExecutionContext`, host execution handle | **Never** | ARCH-002 §10 already requires `HostExecutionHandle` to remain opaque and scoped to one execution cycle |
| Capability state | Bindings, revocation state | **Never persisted by the new mechanism** — already owned, already durable-in-the-sense-that-matters, by Capability Authority | Duplicating it inside an actor snapshot would create a second, conflicting source of truth — directly prohibited by the ARCH-002 §9 "no silent reinstatement" rule |
| Transient/mechanical state | Mailbox contents, Scheduler ready-set membership, in-flight dispatch bookkeeping | **Never** | Consistent with the already-established, disclosed non-preservation of mailbox contents across restart (ARCH-004 §14) |
| Runtime-owned audit facts | Audit event history | **Not part of actor state at all** — a separate, already-existing concern | No change |

**Rule:** the *only* thing a Persistent Actor mechanism should ever durably capture is domain state — everything else is either already durable via an existing, better-suited owner, or is architecturally required to remain ephemeral.

### 2. What Should Persistence Mean?

| Strategy | Determinism | Auditability | Correctness | Performance | Complexity | Recovery time | Maintainability | Verdict |
|---|---|---|---|---|---|---|---|---|
| Full snapshot | High | Low | High | Good for small/medium state | Low | Fast (O(1) load) | High | **Recommended for the first milestone** |
| Incremental snapshot (deltas) | Medium | Medium | Medium | Better write throughput | Medium | Medium | Medium | Rejected for milestone 1 |
| Event sourcing | High (if disciplined) | **Highest** | High, but schema-evolution-sensitive | Write-cheap, read-cost grows with history | High | Slower without snapshots | Lower without active management | **Named as a future, compatible extension** |
| Command replay (Temporal-style) | **Highest, enforced** | High | High, if determinism enforced | Heavy | Highest | Slowest without caching | High if disciplined | **Named as a future, compatible extension** |
| Write-ahead log | High | Medium | High | Write-cheap | Medium | Requires replay-to-checkpoint | Medium | Rejected |
| Operation log (audit-derived) | High | High | Medium | Medium | Medium | Medium | Low | Rejected — conflates Audit Emitter and Persistence Service |
| Hybrid (snapshot + event log) | High | High | High | Best once mature | Highest | Fast + precise | High once built | The natural **second-milestone evolution**, not the starting point |

**Recommendation:** whole-state snapshot for the first milestone. **Rejected for milestone 1, not permanently:** incremental snapshots, write-ahead logging, operation-log-as-persistence. **Explicitly named future extensions, not designed here:** event sourcing and command-replay/durable-execution.

### 3. Ownership

*(Corrected per DAR-001 Finding 2. The actor never produces encoded bytes; Persistence Service owns all encoding and storage mechanics. See "Correction Notes" at the end of this document for the exact defect and fix.)*

Applying SynapseOS's own already-established, twice-precedented principle (Supervisor, Temporal Runtime: policy/decision with a new narrow service; mechanics with existing owners; Runtime as sole composer):

| Responsibility | Owner | Basis |
|---|---|---|
| Domain-state meaning — supplying a structured representation of the actor's own state, and reconstructing the actor's own state from that representation | **The actor itself**, via a new opt-in contract | Only the actor knows its own domain state's shape (§4 below) |
| Encoding the actor-supplied structured representation into stored form, and decoding stored data back into that representation | **Persistence Service** | Keeps storage-format evolution independent of any single actor's own domain schema; the actor never touches bytes or a storage format directly |
| Storage mechanics (where bytes live, how they're written/read), retrieval, durability | **Persistence Service** | Already named for exactly this in ARCH-002 §6 |
| Restoration triggering | **Lifecycle Guardian** | Already assigned in ARCH-002 §9 |
| Restoration authority validation | **Capability Authority**, exclusively | Already assigned in ARCH-002 §9; currently unimplemented — must close first (§11 below) |
| Migration (schema/version evolution of the actor's own domain representation) | **The actor itself** (via versioned decoding, not a framework migration engine) | Consistent with "actor owns its own domain state's meaning"; a generic framework migration engine is unjustified scope for milestone 1 |
| Checkpointing decision (when to persist) | **Runtime**, on external instruction (an embedder or a policy layer), never ambient | Mirrors `register_supervision`'s own explicit, non-automatic registration pattern exactly |
| Versioning of the stored snapshot's own outer envelope (as distinct from the actor's domain schema) | **Persistence Service**, as an opaque envelope wrapping the actor-supplied representation | Keeps format evolution independent of any single actor's own domain schema |
| Cross-component orchestration of the whole persist/restore sequence | **Runtime**, sole composer | ADR-0016 Rule 1, without exception |
| Authorization of protected persistence operations | **Capability Authority**, exclusively | See §11 (new) |

**The boundary, stated without ambiguity:**

```text
Actor owns domain-state meaning.
Persistence Service owns encoding and storage mechanics.
Runtime owns orchestration.
Capability Authority owns authorization.
```

### 4. Serialization Contract (the Critical finding from the Architecture Review)

*(Corrected per DAR-001 Finding 1. The illustrative pseudo-signature has been removed. See "Correction Notes.")*

| Option | Coupling | Encapsulation | Evolution | Testing | Language independence | Distributed-future compatibility | Verdict |
|---|---|---|---|---|---|---|---|
| Actor supplies raw encoded bytes directly (the current stub's implicit model) | Low framework coupling, but **zero structure** | Good, but pushes all correctness burden onto every actor author individually | Unmanaged — every actor invents its own versioning | Poor — nothing to assert against generically | N/A (opaque bytes) | Poor — opaque bytes can't be introspected, migrated, or validated centrally | **Rejected** — this is precisely the underspecified shape the Architecture Review flagged as Critical |
| Framework-mandated serialization baked directly into the core actor contract | High coupling — forces one specific serialization convention onto every actor, persistent or not | Weak — exposes internal representation directly | Painful without an explicit versioning layer | Good — generic round-trip tests possible | Poor — ties the actor programming model to one specific mechanism | Weak — a future non-native actor host would need to reproduce the identical mechanism | Rejected as the sole mechanism — too tightly coupled for an architecture document to mandate, and would burden actors that never opt into persistence |
| **Actor supplies a structured representation of its own domain state; Persistence Service performs all encoding** — the actor never produces bytes, never touches a storage format, and never sees the encoded form; the exact interface, method names, type names, generic parameters, ownership semantics, and error model are deliberately left to future architecture and authorized implementation work | Low-to-moderate — actor supplies structure, not encoding | **Strong** — actor's internal representation stays private; only the exported structured representation is contractual | Actor author controls their own domain schema's evolution | Good — the exported representation is directly testable in isolation from encoding | Good — the *concept* (structured value in, structured value out) is encoding-agnostic | **Strong** — mirrors Orleans/Dapr's "actor supplies typed state, framework serializes" model exactly, the two systems with the best distributed-scale track record among those studied | **Recommended** |
| Immutable state records (event-sourcing-adjacent: actor never exposes mutable state at all, only ever emits immutable deltas) | Low | Strong | Excellent (additive event types) | Excellent | Good | Excellent | Correct model for the **named future event-sourcing extension** (§2 above), not milestone 1 |

**Recommendation:** an actor that opts into persistence supplies a structured representation of its own domain state and can reconstruct its own domain state from that representation. This document establishes *responsibility and information flow* — who supplies what, in which direction, and who is forbidden from inventing structure the actor did not supply — never an interface. The exact interface, method names, type names, Rust trait shape, generic parameters, ownership semantics, and error model are deliberately not fixed here; they are architecture and implementation questions, listed explicitly under "Deferred to ARCH-007" below.

### 5. Identity

**Recommendation: `ActorId`.**

Supporting evidence, independently converging from four directions: Supervisor's hierarchy and restart accounting (ARCH-004 §12, §15); Timer registrations (ARCH-005 §11, §21); Capability Authority's own bindings (`core/capability-authority/src/internal.rs:127`); and the comparative study's own single strongest finding — all seven external systems studied independently arrive at the identical persistent-identity/ephemeral-incarnation split.

`ActorInstanceId`-keying (the current stub's own signature) is rejected outright: an `ActorInstanceId` that changes on every restart cannot, by construction, key data meant to survive a restart.

### 6. Restore Process (conceptual sequence, no code)

1. Runtime is instructed (explicit, non-ambient — mirroring `register_supervision`) that `ActorId` X should be restored from persisted state, or restoration is triggered as part of an already-authorized recovery flow (a future-scope question — see "Deferred to ARCH-007").
2. Runtime asks Persistence Service for the stored representation for `ActorId` X. Persistence Service performs mechanical I/O and decoding only — it does not decide whether the restoration is authorized.
3. Runtime asks the actor's own registered reconstruction logic to turn the decoded, structured representation back into a live `Box<dyn Actor>`.
4. **Before** the resulting instance is made live, Runtime asks Lifecycle Guardian to trigger restoration validation; Lifecycle Guardian in turn — through Runtime's own mediation, never directly (ADR-0016 Rule 2) — causes Capability Authority to revalidate every capability binding associated with `ActorId` X. This step currently does not exist for the analogous suspend/restore case and must be implemented before any restore path, durable or in-memory, can be trusted.
5. Only if revalidation succeeds does Actor Host create the live instance (new `ActorInstanceId`, per the identity model above).
6. Audit events are emitted in strict order: restoration requested → representation read (success/failure) → decode (success/failure) → capability revalidation (success/failure, per binding) → instance created → restoration completed. No event may claim completion before every preceding step has genuinely, truthfully succeeded.
7. Supervision interaction: a restored actor is, if registered, still supervised under its pre-existing hierarchy — no re-registration act required, mirroring ARCH-004 §15's identical treatment of ordinary supervised restart.

### 7. Failure Model

| Failure | Required architectural behaviour |
|---|---|
| Failed snapshot (write) | Per ADR-0015's already-established pattern: the reporting operation fails; no partial write may be treated as authoritative; no actor-visible state changes as a result of a failed persist |
| Failed restore (read) | The restoration attempt fails cleanly; the `ActorId` is left with **no live instance** — an honest degradation, never a silent fallback to default/empty state and never an ambient retry loop |
| Corrupted snapshot | Treated identically to a failed restore — decode failure is a restoration failure, not a special case |
| Missing snapshot | Must be architecturally distinguished from corruption in the audit record — "never persisted" is not an error; "persisted then lost" is |
| Concurrent persistence | Given SynapseOS's own single-message-per-instance, synchronous execution discipline (ARCH-002 §12), concurrent persistence of the *same* `ActorId` is not reachable through the existing execution model without a new concurrency primitive this exploration does not introduce — a question for ARCH-007, not resolved here |
| Version mismatch | The actor's own registered reconstruction logic is the sole authority on whether a given stored version is acceptable; Persistence Service and Runtime never infer compatibility |
| Schema evolution | Owned entirely by the actor author's own reconstruction logic (§4/§3) — no framework migration engine is proposed for milestone 1 |
| Rollback | Not designed here — a distinct capability, closer to the event-sourcing extension's own natural territory |
| Storage outage | Deliberately outside architecture scope (ARCH-002 §22: "Permitted variation: persistence technology") — the *contract* (persist/restore may fail; failure must never be silently absorbed) is architectural; the outage-handling policy is not |

### 8. Audit

Required event categories (concrete identifiers deferred to the implementing EWO, per ARCH-004 §22/ARCH-005 §20's own established precedent): persistence registered; snapshot requested; snapshot written (success/failure); restoration requested; representation read (success/failure, distinguishing missing-vs-corrupted); capability revalidation outcome (per binding); restoration completed; restoration refused; plus the authorization-specific events introduced in §11 (new). Ordering guarantee: identical to the restore-sequence ordering in §6. Replay guarantees: none are introduced by the recommended whole-snapshot model. Determinism requirement: the actor's own encode/decode of its structured representation must be a pure function of that representation — no non-deterministic logic permitted inside either direction, mirroring the identical requirement Temporal enforces on workflow code and Akka enforces on event-fold functions.

### 9. Future Compatibility

| Future capability | Composes cleanly? | Basis |
|---|---|---|
| Workflow Runtime | Yes, via the named future event-sourcing/command-replay extension, not the milestone-1 whole-snapshot model directly | Temporal/Akka precedent |
| Distributed Runtime / Clustering | Yes, contingent on preserving `ActorId`-based, location-transparent keying | ARCH-002 §7; Orleans/Dapr precedent |
| Long-running AI actors / durable autonomous agents | Yes, but **only after** the serialization-contract decision (§4) is resolved by ARCH-007 | An AI agent's accumulated context is precisely the kind of domain state the recommended structured-representation contract is designed to carry |
| Scheduled jobs | Yes, already anticipated by ARCH-005 §21 | No new work required beyond the general mechanism |
| Future Intelligence Core | Cannot be evaluated concretely (no architecture yet defines it); the recommended design imposes no known obstruction | Consistent with ARCH-004/005's own "compatible, not solved" treatment of equally undefined future concerns |

### 10. Persisted-State Deletion and Retention

*(New section, added per DAR-001 Finding 3. This design question was required by the antecedent Architecture Review and was absent from v0.1.0.)*

**Distinct concepts, not to be conflated:**

- **Stopping an actor instance** — ordinary, explicit termination of one `ActorInstanceId` (already-existing behavior, unaffected).
- **Actor failure** — a genuine `Actor::handle()` fault, routed to Supervisor (ARCH-004 §11).
- **Actor restart** — Supervisor-driven replacement of a failed incarnation with a fresh one under the same `ActorId` (ARCH-004 §12).
- **Logical actor retirement** — an `ActorId` no longer expected to run again, but whose durable record and identity have not been formally removed.
- **Permanent logical actor deletion** — an explicit, distinct act that removes the `ActorId`'s own durable record.
- **Deleting durable state vs. retaining it** — two independently controllable outcomes; deletion of state and deletion of the logical actor's own identity are related but not automatically identical acts.
- **Administrative retention or archival policy** — a governance/operational concern layered on top of the mechanism, not designed here.

**Required statements, settled now:**

- Stopping or failing an `ActorInstanceId` **must not** automatically delete the durable state associated with the stable `ActorId`. This follows directly from the already-approved identity model (§5): state is `ActorId`-keyed *specifically* so that it survives instance loss; if instance termination deleted it, the entire keying rationale would be defeated.
- Ordinary restart semantics **depend on** durable state surviving execution-incarnation loss — this is a restatement of the already-approved recovery model (§6), not a new decision.
- Permanent logical actor deletion is **distinct** from instance termination — different trigger, different authorization weight (§11), different audit weight.
- Durable-state deletion must be **explicit, auditable, policy-governed, and associated with the stable `ActorId`** — never inferred from an absence of activity.
- Accidental state deletion **must not** occur as a side effect of supervision, restart, deactivation, Runtime shutdown, or instance replacement.
- The persistence design must support a future retention policy **without coupling the actor to storage infrastructure** — consistent with the already-rejected infrastructure-specific models (Service Fabric, §"Comparative Analysis").

**Design disposition, settled now:**

> Persistent state survives instance termination and is deleted only through an explicit logical-actor state-removal operation governed by architecture, authorization, policy, and audit.

**Explicit questions for ARCH-007** (architecture-level; not answered here):

- Whether permanent logical actor deletion immediately purges state, creates a tombstone, enters a retention period, or transitions to an archived state.
- Which runtime service owns retention-policy enforcement.
- How failed deletion and partial deletion are represented.
- Whether deletion is reversible during a configured retention period.
- How related metadata, snapshots, version envelopes, and audit references are treated upon deletion.
- Whether deletion of actor state is permitted while any live actor incarnation exists.

### 11. Capability Authorization for Persistence Operations

*(New section, added per DAR-001 Finding 4. This design question was identified as missing despite direct, available precedent in ARCH-004 §17.)*

**Operations covered:** checkpoint/persist; restore; inspect persistence metadata (where applicable); delete durable actor state; apply retention or archival actions (where applicable).

**Required statements, settled now — preserving the existing SynapseOS authorization model without exception, directly applying ARCH-004 §17's own precedent ("Supervisor does not grant authority... has no path to Capability Authority... holds no capability-issuing power of its own"):**

- Persistence Service does not grant authority.
- Persistence Service does not define security policy.
- Runtime does not invent authority.
- Lifecycle Guardian does not grant persistence rights.
- Supervisor does not grant persistence rights.
- An actor's own possession of an `ActorId` does not itself constitute authority to persist, restore, inspect, or delete that actor's state.
- **Capability Authority remains the sole source of authorization truth** for every protected persistence operation.
- Runtime must obtain or verify the required authorization before directing any protected persistence operation.
- Restore must include capability revalidation before the restored actor becomes live (already settled in §6; restated here for completeness of the authorization model).
- **State deletion requires stronger, separately auditable authorization than ordinary checkpointing** — the two must never share a single, undifferentiated authorization check.
- Authorization decisions and persistence outcomes are **distinct audit events** — an authorization grant is not itself proof an operation occurred, and an operation's success is not itself proof it was authorized.
- A storage operation succeeding does not prove that the operation was authorized.
- An authorization decision succeeding does not prove that the storage operation completed.

**Design disposition, settled now:**

> All protected persistence operations are capability-authorized through the existing Capability Authority model, while Persistence Service remains a policy-neutral storage mechanism.

**Explicit questions for ARCH-007** (architecture-level; not answered here):

- Which persistence operations require distinct capabilities.
- Whether an actor may request its own checkpoint.
- Who may initiate restore.
- Who may permanently delete durable state.
- Whether administrative retention actions require separate authority from ordinary persistence operations.
- How authority is revalidated after restart, at the mechanism level (the *requirement* that it must be revalidated is already settled, §6).
- What audit evidence is required for denied, failed, and successful operations of each kind.

---

## Decision Matrix

*(Unchanged from v0.1.0 — consolidated: persistence strategies × the full criteria set, including future-compatibility and architectural consistency.)*

| Strategy | Simplicity | Correctness | Scalability | Determinism | Auditability | Recovery | Op. complexity | Future compat. | Arch. consistency | **Overall for milestone 1** |
|---|---|---|---|---|---|---|---|---|---|---|
| Full snapshot | High | High | Medium | High | Low-Med | Fast | Low | Medium | High | **Selected** |
| Incremental snapshot | Medium | Medium | Medium-High | Medium | Medium | Medium | Medium | Medium | Medium | Rejected (premature) |
| Event sourcing | Low | High | High | High | **Highest** | Slow w/o snapshots | High | **High** | High | Deferred (future) |
| Command replay | Low | High | High | **Highest** | High | Slowest w/o caching | Highest | **High** for workflow use cases | High | Deferred (future) |
| Write-ahead log | Medium | High | Medium | High | Medium | Medium | Medium | Low | Medium | Rejected |
| Operation log (audit-derived) | Medium | Medium | Medium | High | High | Medium | Medium | Low | **Low** (conflates two owners) | Rejected |
| No framework model (Erlang/Ray style) | Highest | Low (unspecified) | N/A | N/A | N/A | N/A | Low (framework) / High (app) | Low | **Lowest** | Rejected |

---

## Recommended SynapseOS Design

**Guiding principle:** an actor's *existence* (`ActorId`, capability bindings) is already durable by construction; Persistent Actors extends that same durability, on the same key, to a fourth category of data — the actor's own domain state — and to nothing else.

- **Ownership model:** actor supplies domain-state meaning; Persistence Service supplies encoding and storage mechanics; Lifecycle Guardian triggers restoration; Capability Authority exclusively authorizes and validates; Runtime alone orchestrates the sequence (ADR-0016 Rule 1).
- **Persistence model:** whole-state snapshot for the first milestone. Event sourcing and command-replay/durable-execution are named, explicitly, as compatible future extensions.
- **Serialization model:** an actor that opts in supplies a structured representation of its own domain state; it never produces encoded bytes and never touches a storage format; Persistence Service performs all encoding and decoding. Exact interface left to ARCH-007/implementation.
- **Identity model:** `ActorId`-keyed, without exception.
- **Recovery model:** the seven-step sequence in §6, with mandatory capability revalidation as a hard precondition, requiring the currently-disclosed implementation gap in `Runtime::restore_actor_instance` to close first.
- **Deletion and retention model:** durable state survives instance termination, failure, and restart; permanent deletion is a distinct, explicit, separately-authorized, auditable, policy-governed act, never an implicit side effect (§10).
- **Authorization model:** Capability Authority is the sole source of authorization truth for every protected persistence operation; no other component — Persistence Service, Runtime, Lifecycle Guardian, or Supervisor — grants authority (§11).
- **Failure model:** ADR-0015's uniform policy extended to persistence I/O; missing state architecturally distinguished from corrupted state; no ambient retry, no silent fallback to default state.
- **Audit model:** new named event categories (concrete identifiers deferred to the implementing EWO); strict, truthful ordering; mandatory distinction between "restored" and "fresh," and between authorization decisions and persistence outcomes.
- **Rejected alternatives:** `ActorInstanceId`-keying; raw-bytes serialization; a framework-mandated serialization mechanism baked into the core `Actor` trait itself; event sourcing and command replay as the *first* milestone; any model requiring a specific storage/cluster infrastructure.
- **Trade-offs accepted:** whole-snapshot persistence sacrifices full historical auditability in exchange for milestone-1 simplicity and correctness, consistent with ARCH-002 §21's Minimal Runtime Profile philosophy; event sourcing remains explicitly available as a later, additive extension.
- **Trade-offs rejected:** infrastructure-specific replication (Service Fabric) — rejected outright, since it directly contradicts an already-published conformance requirement (ARCH-002 §22).

---

## Architectural Consequences

- ARCH-002's own "Storage Architecture" deferral (§23) is fulfilled, not amended.
- `services/persistence`'s existing trait stub must be treated as **non-authoritative** — its `ActorInstanceId` keying and its implicit "actor produces bytes" shape should not be carried forward into ARCH-007.
- ARCH-007 will need to state, normatively, that `Runtime::restore_actor_instance`'s missing capability-revalidation step is a **precondition**, not a parallel-track concern.
- No constitutional concept (ARCH-001) requires amendment. No existing Trusted Core component's responsibility changes. No new direct peer-interaction path is introduced (ADR-0016 preserved without exception).

## Questions for ARCH-007

### Already decided by DES-001

- Persistence is keyed by `ActorId`, never `ActorInstanceId`.
- `ActorInstanceId` represents an ephemeral execution incarnation only.
- Whole-state snapshot persistence is the recommended first milestone; event sourcing and command replay are compatible future extensions, not designed now.
- Actors supply a structured representation of their own domain state; the actor never performs storage encoding or produces bytes directly.
- Persistence Service owns encoding, decoding, storage, retrieval, and storage-boundary error/corruption detection.
- Runtime remains the sole cross-component orchestrator of every persist/restore sequence.
- Capability Authority remains the sole source of authorization truth for all protected persistence operations; Persistence Service, Runtime, Lifecycle Guardian, and Supervisor do not grant authority.
- Capability revalidation is mandatory before a restored instance becomes live.
- Durable state survives instance termination, failure, and ordinary restart.
- Permanent logical-actor deletion is a distinct, explicit, auditable, policy-governed operation, never an implicit side effect of instance-level events.
- Infrastructure-specific persistence coupling is rejected.

### Deferred to ARCH-007

- Exact opt-in contract shape (interface/method/type names, generics, ownership semantics, error model).
- Concurrency and checkpoint coordination for the same `ActorId`.
- Runtime-startup restoration policy (automatic reattachment vs. explicit/on-demand).
- Version-envelope ownership (outer format-version tag: Persistence Service vs. actor-owned).
- Deletion lifecycle semantics: immediate purge vs. tombstone vs. retention period vs. archived state.
- Which service owns retention-policy enforcement.
- Failed/partial deletion representation.
- Reversibility of deletion during a configured retention period.
- Treatment of related metadata, snapshots, version envelopes, and audit references upon deletion.
- Whether deletion is permitted while a live incarnation exists.
- Capability granularity — which specific operations (checkpoint, restore, inspect, archive, delete) require distinct capabilities.
- Whether an actor may request its own checkpoint.
- Who may initiate restore.
- Who may permanently delete durable state.
- Whether administrative retention actions require separate authority from ordinary persistence operations.
- The mechanism by which authority is revalidated after restart.
- Required audit evidence for denied, failed, and successful operations of each kind.

## Open Risks

*(Unchanged from v0.1.0.)*

- The capability-revalidation gap is a precondition this document cannot itself close — if unaddressed before ARCH-007's implementing EWO, durable persistence would materially increase the real-world consequence of a known, already-disclosed gap.
- Whole-snapshot persistence, if actor domain state grows large (a realistic risk for the explicitly-named "long-running AI actors" use case), degrades toward the same performance weakness Orleans is known for.
- `services/actor-directory` remains an equally unimplemented stub; any future "discover previously-persisted actors after a Runtime restart" capability depends on it.
- No system studied in the comparative analysis was designed with SynapseOS's specific capability-security model in mind; the capability-authorization and revalidation requirements (§6, §11) have no direct precedent in any of the seven systems studied.

## Appendix — Supporting Evidence

Primary sources cited above, all independently re-verified: ARCH-001 §11; ARCH-002 §6, §7, §9, §15, §18, §20, §21, §22, §23; ARCH-004 §11, §12, §14, §15, §17, §18, §21, §22; ARCH-005 §11, §20, §21; ADR-0015; ADR-0016 (Rule 1/Rule 2, quoted verbatim); AFR-001 (Gate 2); `services/persistence/{Cargo.toml,README.md,src/lib.rs,src/internal.rs}`; `services/actor-directory/README.md`; `api/src/lib.rs` (`Actor` trait); `core/actor-host/src/internal.rs` (`ActorHostImpl` field layout, `behaviors` map); `runtime/src/lib.rs` (`restore_actor_instance`, `suspend_actor_instance`); the completed Persistent Actor Architecture Review (conversational record). Comparative-system claims (Orleans, Erlang/OTP, Akka Persistence, Service Fabric Reliable Actors, Dapr Actors, Temporal, Ray) drawn from established, stable, publicly documented architectural models of each system, not from repository evidence — disclosed explicitly, not treated as repository fact.

## Correction Notes (v0.1.0 → v0.2.0)

Applied exactly the four findings of DAR-001 (verdict: DESIGN CHANGES REQUIRED). No other content changed; the core recommended direction is unchanged and was not reopened.

1. **Pseudo-API removal.** §4 (Serialization Contract) previously illustrated the recommended option with a concrete pseudo-signature naming specific methods and a specific type. This violated the document's own stated scope discipline. The signature has been removed and replaced with purely conceptual language describing responsibility and information flow only.
2. **Ownership correction.** §3 (Ownership) previously stated the actor owns "turning domain state into bytes," contradicting §4's actual position that the actor supplies a structured representation while Persistence Service performs encoding. §3's table and its closing statement have been corrected to match §4 exactly, and the boundary is now stated in one unambiguous block in §3.
3. **Deletion and retention.** A new §10, "Persisted-State Deletion and Retention," has been added, distinguishing instance termination from permanent logical-actor deletion, settling that durable state survives instance-level events, and listing the deletion-specific questions deferred to ARCH-007.
4. **Capability authorization.** A new §11, "Capability Authorization for Persistence Operations," has been added, applying ARCH-004 §17's own precedent directly, settling that Capability Authority is the sole authorization source for every protected persistence operation, and listing the authorization-specific questions deferred to ARCH-007.

The "Questions for ARCH-007" section has been restructured into "Already decided by DES-001" and "Deferred to ARCH-007," with every item from both new sections correctly placed and no item appearing in both lists.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-26 | Denver Jacobs (AI-assisted) | Initial design exploration (delivered as conversational content, not filed as a repository document at this version). Reviewed by DAR-001; verdict: DESIGN CHANGES REQUIRED (four findings). |
| 0.2.0 | 2026-07-26 | Denver Jacobs (AI-assisted) | First filed revision. Applies exactly DAR-001's four findings: pseudo-API removed (§4); Ownership corrected (§3); Persisted-State Deletion and Retention added (§10, new); Capability Authorization for Persistence Operations added (§11, new); "Questions for ARCH-007" restructured into decided/deferred. No other content changed; core recommended direction not reopened. |
