---
document_id: ARCH-002
title: Runtime Architecture
project: SynapseOS
specification: SynapseOS — Runtime architecture realizing ARCH-001's constitutional concepts
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-11
last_updated: 2026-07-11
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
  standards:
    - STD-001 (Approved)
  architecture:
    - ARCH-000 (Draft — historical record, unmodified)
    - ARCH-001 (Draft — constitutional foundation this document realizes)
  rfcs: None
  adrs:
    - ADR-0011 (Draft, Act 1 effective)
    - ADR-0012 (Approved)
    - ADR-0013 (Draft)
  roadmap: None
  research: None
  operational: None
  source_artifacts: None
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-002 — Runtime Architecture

*Filename pattern: `ARCH-002-Runtime-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Purpose

This document defines how the SynapseOS Runtime realizes ARCH-001's constitutional architecture: how it creates, binds, executes, mediates, observes, suspends, resumes, and terminates constitutional execution while preserving explicit authority, actor isolation, immutable messages, capability integrity, and provider independence. It is precise enough that independent engineering teams could implement recognizably conformant runtimes without identical internal code. It selects no programming language, framework, library, database, transport, or deployment platform.

## 2. Scope

This document governs: Runtime identity and responsibility; the boundary between constitutional architecture, Runtime, host platform, and applications; the trusted Runtime core and its minimization; the Runtime component model; the concrete Runtime representation of Actor, Capability, Message, and Execution Context; the constitutional execution cycle; concurrency, mailbox, dispatch, lifecycle, failure, isolation, observation, and extension models; and conformance requirements for a Minimal Runtime Profile.

This document does not govern: ARCH-001's constitutional concepts, which it inherits without redefinition; scheduling algorithms, persistence technology, distributed transport mechanics, provider-adapter patterns, or resource-accounting algorithms, all deferred per §21; and governance, review, or approval process, governed by GOV-003, GOV-010, and STD-001.

## 3. Runtime Identity and Responsibility

**Problem.** Without a precise, single responsibility statement, the Runtime accumulates unrelated concerns over time — exactly the drift `.ai/ARCHITECTURAL-CONTEXT.md`'s Architectural Drift Test exists to catch.

**Selected architecture.** The SynapseOS Runtime is the system that makes constitutional execution actually happen, correctly, and enforces the constitutional laws at every point authority crosses a boundary. Its single overarching responsibility: realize Actor, Capability, Message, and Execution Semantics as running behavior, and guarantee — regardless of what applications or Runtime services do — that non-forgery, integrity, enforcement-at-invocation, and revocation-state enforcement continue to hold.

**What it owns:** actor lifecycle mechanics, mailbox mechanics, dispatch mechanics, capability validation and binding mechanics, execution-context construction, minimal audit-event emission, the bootstrap act.

**What it does not own:** application logic (contained within actors), business or domain policy (scheduling strategy, provider selection, budget allocation — all replaceable services), physical resource management (delegated to the host platform per §4).

**What belongs outside it:** applications, provider adapters, the host operating system, external infrastructure.

**What it must never become:** an application framework; an AI agent; a provider-specific orchestration layer; a global policy engine; a collection of unrelated services; a replacement for the host operating system.

## 4. Runtime Boundary

Extending ARCH-001 §7–§8 with Runtime-internal granularity:

| Layer | Contains |
|---|---|
| Constitutional | Actor, Capability, Message, Execution Semantics (ARCH-001, unchanged) |
| Runtime — trusted core | The mechanisms in §5 |
| Runtime — services | Scheduler policy, Actor Directory, Audit Pipeline, Persistence/Restoration mechanics, provider adapters, knowledge and storage mediators |
| Applications | Actors built on the Runtime's mailbox, message, and capability contracts |
| Host platform | Physical process/thread scheduling, memory allocation, device access, raw networking primitives, durable filesystem mechanisms |
| External infrastructure | Whatever the host platform itself runs on |

SynapseOS introduces, above the host platform: actor identity and isolation, capability-based authority, message-envelope integrity and immutability, and constitutional execution semantics. It introduces nothing at the physical-resource layer — no physical scheduler, no memory manager, no device driver, no raw network stack. Host assumptions are avoided throughout; "the host platform" is treated as an opaque provider of process/thread execution, memory, storage, and networking, regardless of which conventional operating system realizes it.

## 5. Trusted Runtime Core

**Problem.** ARCH-001 fixes four irreducible capability guarantees. Every Runtime mechanism must be classified against them, or the trusted core risks silent, ungoverned growth.

**Selected architecture.** Each trusted mechanism below is presented as an application of exactly one (or more) of ARCH-001's four guarantees — this keeps the trusted core conceptually minimal even where its component count is non-trivial.

| Mechanism | Guarantee(s) applied |
|---|---|
| Capability validation | Non-forgery, integrity |
| Capability-binding management | Integrity |
| Actor-identity assignment (not the queryable directory) | Integrity (identity underpins every capability target) |
| Actor-state isolation enforcement | Integrity (of actor state as a target of authority) |
| Message-envelope integrity | Integrity |
| Send-authority validation / mailbox admission | Enforcement at invocation |
| Execution-ownership enforcement | Enforcement at invocation |
| Dispatch mechanism (not dispatch *order*, which is Scheduler policy) | Enforcement at invocation |
| Lifecycle-transition validity enforcement (not lifecycle *policy*) | Enforcement at invocation |
| Suspension and restoration validation | Revocation-state enforcement |
| Minimal audit-event emission (not storage or processing) | Supports auditability of all of the above |
| Host Adapter (the sole seam to physical resources) | Integrity of everything crossing the Runtime/host boundary |

No mechanism outside this table is trusted core. In particular: scheduling *order*, actor directory *lookup*, audit *storage and processing*, and lifecycle *policy* (when, not whether, a transition is valid) are replaceable services, not trusted mechanisms — the mechanism/policy line from ARCH-001 §9 applied concretely.

## 6. Runtime Component Model

Merging overlapping candidates and rejecting unnecessary decomposition, per the required method:

**Trusted core (7 components):**

| Component | Responsibility | Prohibited from |
|---|---|---|
| **Capability Authority** | Validates, binds, attenuates, delegates, revokes capabilities; enforces non-amplification and the recursive minting ceiling; tracks authority lineage | Minting authority ambiently; treating identity as sufficient for authorization |
| **Actor Host** | Assigns actor identity (logical and instance); enforces state isolation; enforces execution ownership | Exposing an actor's private state to anything other than the actor's own message-handling logic |
| **Message Gateway** | Enforces envelope integrity and immutability; validates send authority before mailbox admission | Admitting a message on destination-identifier alone; treating a carried capability-transfer as send authority |
| **Execution Coordinator** | Constructs Execution Context; performs dispatch mechanics; enforces one-owner execution; performs capability revalidation at invocation where required | Selecting dispatch *order* (that is Scheduler policy consumed, not decided, here) |
| **Lifecycle Guardian** | Enforces legal lifecycle-state transitions; validates suspension and restoration, including capability revalidation on resume | Deciding *when* to suspend/restart (policy, deferred) |
| **Audit Emitter** | Emits minimal, unbypassable audit events at the moment trusted mechanisms act | Storing, indexing, retaining, or redacting events (all replaceable-service concerns) |
| **Host Adapter** | The sole, minimal, audited seam between the Runtime and physical host resources | Implementing policy about how host resources are used |

**Replaceable services (deferred in depth, contract-bound here):**

| Service | Responsibility | Bound by |
|---|---|---|
| **Scheduler** | Decides dispatch *order* among ready actors | The dispatch-selection interface (§18); MUST NOT starve a ready actor indefinitely |
| **Actor Directory** | Queryable actor lookup | MUST NOT return unrestricted, capability-free actor references |
| **Audit Pipeline** | Storage, indexing, retention, redaction of emitted events | Consumes Audit Emitter output only; failure here MUST NOT affect the triggering execution |
| **Persistence/Restoration Service** | Performs the mechanical snapshot/replay I/O that Lifecycle Guardian validates | MUST NOT reinstate authority Lifecycle Guardian has not revalidated |

Audit Emitter is deliberately not folded into any single other trusted component: every other trusted component calls into it, making it a genuine cross-cutting utility rather than an artificially separated concern.

## 7. Actor Runtime Representation

An actor has a **logical identity**, assigned once at definition, stable across suspension, resumption, and restart. A restarted actor (a new instance under supervision) receives a new **instance identity**, distinct from its unchanging logical identity; capabilities generally bind to logical identity, with instance-level scoping used for isolation and execution-ownership bookkeeping. Actor state is private, held per-instance, enforced by Actor Host. The message-handling contract is actor-defined and Runtime-agnostic — ARCH-002 requires that one exists and is triggered per Execution Semantics, not how it is written. Each actor instance has exactly one mailbox (§11). Capability bindings are held by Capability Authority, associated with logical identity. Local versus remote representation is a location concern, not an identity concern: logical identity is location-transparent by contract; the concrete distributed-routing mechanism is deferred (§21).

**Knowledge of an actor identifier grants no ability to communicate with it.** An identifier alone — however obtained, including from the Actor Directory — is insufficient; every send additionally requires a valid capability naming that actor as a target, validated by Message Gateway before admission. No global ambient actor registry exists; the Actor Directory is a replaceable lookup service, bound to never return unrestricted references.

## 8. Message Runtime Representation

The Runtime message envelope contains: a unique, immutable message identity; message type (load-bearing — capability-based type restriction depends on it); sender identity (for causation and audit, never for authority); destination (validated against a presented capability, never trusted alone); payload (opaque to the Runtime); causation identity; correlation identity; delivery constraints (explicit, never an ambient default); durability classification (explicit per message, never ambient); deadline; replay-protection information; a distinct capability-transfer section, for capabilities an actor deliberately delegates via this message; information classification; and trace metadata.

Message *content* (actor-defined payload), the Runtime *envelope* (structural fields above), *send authority* (the capability Message Gateway checks before admission), *deliberately transferred capabilities* (the transfer section — an application-level choice, distinct from send authority), and *operational trace information* (Runtime-managed) are four distinct concerns and MUST NOT be conflated. The Runtime MUST NOT treat a destination identifier as send authority. The Runtime MUST validate authority before mailbox admission, before any recipient-controlled processing occurs.

## 9. Capability Runtime Representation

**Local representation:** an unforgeable, in-process structured object, constructible only by Capability Authority. **Remote representation:** required at any boundary where actors are not co-located within one trusted Runtime instance; MUST be cryptographically protected and independently verifiable; the concrete cryptographic scheme is deferred (§21). Both representations conform to one abstract model — target, permitted operations, constraints, provenance, revocation handle — never two competing definitions of "capability."

Capability Authority is the sole binding creator and sole source of truth for binding lookup; bindings are queried fresh, never cached as independent truth elsewhere, avoiding staleness after revocation. Capabilities are immutable once issued; attenuation, delegation, and transfer each produce a new, distinct object, never a mutation. Revocation state (lease expiry, generation invalidation, and — where warranted — indirection, per the Architecture Review Board's capability-model findings) is Capability Authority's responsibility; the concrete revocation-technique implementation is deferred, but the contract — Capability Authority can always answer "is this currently valid" — is not. Every capability's lineage traces to the single bootstrap root through a chain that never widens; issuance requires the issuer's own constraint set to bound whatever it mints, recursively. Restoration validation is a joint act: Lifecycle Guardian triggers it on resume; Capability Authority performs it, and a binding that fails revalidation is not reinstated.

Deferred to later architecture: the concrete cryptographic scheme for remote capabilities, the concrete revocation-list or indirection implementation, and cross-instance federation of trust roots. Not deferred: the Runtime flow by which capability validation participates in every execution (§10).

## 10. Execution Context

Execution Context is the Runtime-layer data structure realizing ARCH-001's Execution Semantics for one specific, in-progress execution. Its minimum contents: the owning actor-instance binding; the triggering message's identity (causation); correlation metadata; the capability bindings active for this specific execution (not necessarily the actor's entire binding set); a deadline; cancellation state; an execution budget; a trace identity; an opaque host-execution handle (its contents are entirely host/implementation-specific and MUST remain opaque above this boundary); and, where applicable, suspension metadata. Information classification SHOULD be present where the deployment requires audit or redaction tiering.

Execution Context MUST NOT become an authority source (it composes references, it does not itself grant), a global or shared object, an application session, a container for arbitrary services, or the owner of any constitutional concept. One Execution Context is bounded to exactly one message-processing cycle — no departure from this boundary is justified against ARCH-001, and none is proposed here.

## 11. Constitutional Execution Cycle

| # | Step | Responsible | Trust | Key invariant | Failure → resulting state |
|---|---|---|---|---|---|
| 1 | Runtime initialization | Host Adapter, all trusted components | Trusted | Exactly one, disclosed, one-time bootstrap act | Init failure → Runtime does not start |
| 2 | Actor definition | Actor Host | Trusted | Logical identity stable, unique | Rejected → actor never exists |
| 3 | Actor creation | Actor Host | Trusted | Isolation established before any execution | Creation failure → remains Defined |
| 4 | Capability issuance/delegation | Capability Authority | Trusted | Non-amplification, ceiling, lineage recorded | Denied → no capability created |
| 5 | Capability binding to initiating actor | Capability Authority | Trusted | Binding referenced, never copied | Rejected → actor holds no binding |
| 6 | Message creation | Actor (content) + Message Gateway (envelope structure) | Mixed | Message immutable once constructed | Malformed → rejected at construction |
| 7 | Send-authority validation | Message Gateway | Trusted | No admission without validated authority | Rejected → discarded, audited |
| 8 | Envelope validation | Message Gateway | Trusted | Envelope integrity intact | Rejected → discarded, audited |
| 9 | Mailbox admission | Message Gateway | Trusted (mechanism); overflow policy replaceable | Bounded capacity; no silent loss | Overflow → policy-defined handling, always audited |
| 10 | Dispatch selection | Scheduler (policy) → Execution Coordinator (mechanism) | Mixed | Selection re-validated as current before use | Stale selection → re-selection requested |
| 11 | Execution Context construction | Execution Coordinator | Trusted | Context reflects current bindings, not cached | Actor gone → abort, no execution |
| 12 | Capability revalidation at invocation | Capability Authority | Trusted | No revoked/expired capability silently reinstated | Fails → binding dropped or execution fails |
| 13 | Actor message processing | Actor | Untrusted | Bounded by held capabilities only | Failure → handled per step 19 |
| 14 | Actor-state transition | Actor Host (isolation) wrapping actor logic | Mixed | Isolation preserved | Corruption detected → actor instance fails |
| 15 | Outbound message creation | Actor, recursing into steps 6–9 | Mixed | Same as steps 6–9 | Same as steps 6–9 |
| 16 | Capability transfer/attenuation | Capability Authority | Trusted | Transfer no broader than delegator holds | Rejected → transfer omitted |
| 17 | Audit-event emission | Audit Emitter | Trusted (emission only) | Unbypassable; occurs regardless of outcome | Consumer failure MUST NOT block execution |
| 18 | Execution completion | Execution Coordinator | Trusted | Exactly one completion per execution | N/A — success path |
| 19 | Actor → idle/suspended/failed/terminated | Lifecycle Guardian | Trusted | Only legal transitions accepted | Illegal transition → forced safe failure state |
| 20 | Runtime shutdown | Host Adapter, trusted components | Trusted | No silent loss of admitted/committed work | Forced shutdown → best-effort suspension, audited |

## 12. Concurrency Model

**Problem.** Whether one actor may process more than one message at a time determines whether the actor model's core promise — private state reasoned about without internal concurrency hazards — actually holds.

**Trade-off analysis.** Allowing concurrent processing within one actor would require the actor's own logic to internally synchronize state, reintroducing exactly the shared-mutable-state hazard class the model exists to avoid, and would blur the "one execution, one message-processing cycle" boundary into ambiguous overlapping cycles.

**Selected architecture.** An actor instance processes at most one message at a time; the next message on that instance does not begin until the current execution completes. This is independently justified for SynapseOS by the consequence above, not by convention. Reentrancy is excluded by direct construction: while an actor is Executing, it is not eligible for a new dispatch. Ordering within one actor's mailbox matches admission order, a mechanism-level guarantee; ordering *across* different senders relative to each other is not guaranteed deterministic absent a specific transport choice (deferred). Concurrency *across* actors is unrestricted — this is the system's source of parallelism. The Scheduler MUST NOT starve a ready actor indefinitely; the specific fairness algorithm is deferred. Every execution is bounded: it carries a deadline and budget via Execution Context, and unbounded (run-forever) execution cycles are disallowed by construction. Cancellation is cooperative — the architectural contract requires actor logic to observe cancellation state; forceful preemption of arbitrary in-progress logic is a host/implementation capability, not constitutionally guaranteed. Mailbox capacity MUST be finite and enforced; the specific bound is a deployment parameter.

## 13. Mailbox Model

Each actor instance has **exactly one mailbox** — not multiple typed mailboxes, not several internal queues. This is independently justified: multiple queues would introduce an additional, unnecessary ordering/fairness question inside a single actor, working against the sequential-processing decision in §12 rather than with it. The mailbox does not outlive its actor instance; a new instance under supervision receives a new mailbox, with in-flight message handling deferred to Lifecycle Architecture. Admission is gated by Message Gateway (§11, steps 7–9). Ordering preserves admission order (mechanism). Capacity is bounded and finite (mechanism-level MUST); the specific number is a deployment parameter (policy). Priority-based reordering is not defined at this layer and is Scheduler-adjacent policy, deferred. Overflow MUST produce a defined, non-silent response — reject-with-audit-event is the mandatory mechanism-level contract; the choice among reject-newest, reject-oldest, or backpressure is policy, deferred. A message whose deadline lapses before dispatch MUST be treated as expired, never delivered. Deduplication MAY use the replay-protection fields; the mechanism and window are deferred. Every rejected, expired, or otherwise undeliverable message MUST generate an audit event — dead-letter handling is never silent. On shutdown, mailboxes drain per deferred policy, but already-admitted messages MUST NOT be silently lost without an audit trail.

This defines the Runtime contract the Scheduler must later satisfy; it does not itself design the Scheduler.

## 14. Dispatch and Routing

Routing resolves a capability-bound destination to a live actor instance; a bare identifier never suffices (§7). Logical-identity-to-instance resolution is a trusted mechanism (Actor Host), since a misresolution is a correctness risk, not merely an availability one. A stale destination (terminated actor, superseded instance) MUST fail safely — rejection and dead-letter audit, never silent misdelivery or silent no-op. When a logical actor is replaced by supervision, routing to its logical identity resolves to the new instance only through Lifecycle Guardian's restoration path (§9), never a transparent, unvalidated swap. Local (same-Runtime-instance) routing MUST NOT require a network hop — independently justified by the unnecessary latency and complexity this would impose on co-located actors. Distributed routing extends this contract (location-transparent logical identity) without redefining it; the concrete mechanism is deferred (§21). The Actor Directory (§6) MUST NOT return unrestricted actor references. Routing remains provider- and transport-independent throughout.

## 15. Runtime Lifecycle

Candidate states were not accepted automatically; only states with distinct architectural meaning were retained.

**Actor:** Defined → Initializing (merging "created" and "initializing" — no genuine architectural distinction was found between them) → Idle ⇄ Ready → Executing → {Idle | Suspended | Failed | Stopping} → Terminated. Suspended returns to Idle/Ready only after successful revalidation (§9); failed revalidation routes to Terminated. Failed may route to Stopping/Terminated, or to a fresh Initializing instance under supervision — policy, deferred. Invalid transitions include: Terminated → anything (terminal); Executing → Executing directly (must pass through completion, per §12's single-message rule); Defined → Executing directly (must pass through Initializing).

**Mailbox:** Active → Draining (no new admissions, existing messages still processed) → Closed.

**Execution Context:** Constructed → Active → {Completed | Suspended | Failed}.

**Capability binding:** Bound → {re-attenuated into a new binding | Revoked | Expired}.

**Message:** Created → {Admitted → Dispatched → Consumed} or {Rejected} or {Admitted → Expired}.

**Runtime:** Initializing → Running → Stopping → {Stopped | Failed}. A distinct Failed state (rather than folding into Stopped) is retained because operators and audit consumers must be able to distinguish a clean shutdown from a trusted-core integrity halt (§16).

Deeper lifecycle *policy* — supervision strategy, restart backoff, detailed sub-states beyond architectural significance — is deferred to a later Lifecycle Architecture document; this section defines only what the first working Runtime requires.

## 16. Failure Model

| Failure | Response tier |
|---|---|
| Malformed message | Reject request only |
| Invalid, revoked, or expired capability | Reject request only; security-relevant audit event |
| Unauthorized send / unknown destination | Reject request only; audited |
| Mailbox overflow | Reject request only (mailbox-owning actor unaffected) |
| Actor initialization failure | Actor instance fails to become Ready; isolated, audited |
| Actor execution failure (application logic error) | Fail current execution only; containment is mandatory |
| Actor state corruption | Fail the actor instance (not merely the execution); high-priority audit |
| Deadline expiry / cancellation | Fail current execution only; not inherently the actor's fault |
| Runtime component failure (a trusted-core mechanism itself fails) | Threatens Runtime integrity — fail-stop for the affected scope |
| Host-platform failure | Outside Runtime control; Runtime's own contract is to leave recoverable, non-corrupted state where possible |
| Audit consumer failure | MUST NOT block or fail the triggering execution — isolated to the replaceable service |
| Restoration failure | Failed bindings are not reinstated; actor fails if minimum required bindings cannot be revalidated |

An actor's failure MUST NOT corrupt trusted Runtime state, under any of the above. Where a trusted-core mechanism itself is what has failed, fail-stop — halting rather than continuing in a possibly-compromised state — is the mandated response, the same principle generalized from individual-actor containment to the trusted core's own integrity.

## 17. Isolation Model

Private actor state, message-only interaction, absence of shared mutable application state, explicit capability-mediated authority, single execution ownership, and failure containment are all required, per §3–§14 above. What the Runtime architecturally requires is that isolation holds; *how* it is delivered — separate host processes per actor, language-level enforcement within a shared process, or a hybrid — is an implementation choice, deliberately not assumed here in either direction. Actor Host is accountable for the guarantee holding regardless of which mechanism a given implementation chooses; it need not itself implement the isolation mechanism, but it must verify or enforce that the chosen mechanism actually delivers it.

## 18. Observation and Audit

Minimum audit events: Runtime start and shutdown; actor creation and termination; capability issuance, delegation, attenuation, and revocation; failed capability validation; message rejection and mailbox admission; execution start, completion, and failure; suspension and restoration; externally observable provider or tool invocation. Security audit, operational tracing, debugging, metrics, billing, and provenance are distinct consumers of overlapping but non-identical event streams, with different retention and fidelity needs — not one undifferentiated log. Full-fidelity logging of every internal operation is not required: security-relevant events are always logged; routine, already-authorized message processing is served by trace and correlation metadata rather than duplicate audit-grade records. Raw capability material MUST NOT appear in any log; identifiers or hashes are logged instead.

## 19. Extension and Replaceability Model

Scheduler, persistence, distributed transport, provider adapters, knowledge services, storage, monitoring, policy engines, and resource accounting are all Runtime services — built from the same Actor, Capability, and Message primitives as any other actor, or, for Scheduler specifically, interacting with the trusted core solely through the dispatch-selection interface (§18). No extension may bypass capability enforcement, message integrity, actor isolation, execution ownership, or lifecycle controls; an extension that required bypassing any of these would not be a bounded extension but a new trusted-core component, subject to the same rigor as §5–§6. There is no generic, unrestricted plugin interface: each extension category receives exactly the scoped interface its role requires, never a broader hook into the trusted core.

## 20. Runtime Interfaces

| Interface | Caller | Receiver | Purpose | Failure behavior |
|---|---|---|---|---|
| Actor creation | Actor Host client (bootstrap or supervisor) | Actor Host | Instantiate a new actor instance | Creation failure leaves actor Defined only |
| Actor termination | Lifecycle Guardian | Actor Host | Tear down an instance | Must not leak isolated state |
| Message submission | Actor (sender) | Message Gateway | Request delivery | Rejected without admission on any validation failure |
| Mailbox admission | Message Gateway | Actor's mailbox | Enqueue a validated message | Overflow handled per §13 |
| Dispatch selection | Scheduler | Execution Coordinator | Propose next (actor, message) pair | Stale proposals are re-validated, not trusted |
| Execution start | Execution Coordinator | Actor | Begin one message-processing cycle | Aborts cleanly if preconditions no longer hold |
| Execution completion | Actor (implicitly, by returning) | Execution Coordinator | End the cycle | Exactly one completion per execution |
| Capability validation | Any trusted component | Capability Authority | Check current validity | Invalid → rejection, audited |
| Capability binding | Capability Authority | Actor Host (actor's active set) | Associate a capability with an actor | Rejected if target actor does not exist |
| Capability delegation | Actor (via Capability Authority) | Capability Authority | Attenuate and transfer | Rejected if it would amplify authority |
| Capability revocation | Any authorized revoker | Capability Authority | Invalidate a capability or generation | Immediate for indirection-backed capabilities; bounded latency otherwise |
| Suspension | Lifecycle Guardian | Actor Host / Persistence Service | Pause an execution or actor | Capability validity is not extended by suspension |
| Restoration | Lifecycle Guardian | Capability Authority, Actor Host | Resume, with revalidation | Failed revalidation withholds the binding |
| Audit-event emission | Any trusted component | Audit Emitter | Record that an event occurred | Never blocks on downstream consumption |
| Host interaction | Any trusted component | Host Adapter | Cross into physical resources | Failure is reported, never silently absorbed |

## 21. Minimal Runtime Profile

Sufficient for the first executable milestone: Runtime initialization; one or more local actors; immutable typed messages; explicit capability issuance; capability-bound message sending; bounded mailbox admission; one-message actor execution; private actor-state transition; execution completion; minimal audit-event emission; clean actor termination; clean Runtime shutdown.

Explicitly deferred, with no contradiction to the core architecture: distributed execution (contract: location-transparent logical identity, §7); durable persistence (contract: Persistence/Restoration Service boundary, §6, §9); external providers (contract: capability-scoped provider adapters as ordinary actors); knowledge architecture (contract: mediator-actor pattern); advanced scheduling (contract: dispatch-selection interface, §18, plus the no-starvation requirement); migration; high availability; multi-host capability representation (contract: one abstract capability model, two representations, §9); production-grade security hardening.

## 22. Conformance Requirements

**Mandatory:** the four trusted-core guarantees (§5) hold at all times; single-message processing per actor instance (§12); bounded mailboxes (§13); immutable messages and capabilities; singular execution ownership; authority only through presented capability, never ambient; audit emission for every security-relevant event.

**Permitted variation:** implementation language; transport technology; persistence technology; scheduling algorithm, provided it satisfies the no-starvation contract; isolation mechanism (process-level, language-level, or hybrid).

**Optional extensions:** distributed execution; advanced scheduling; knowledge and storage services; provider adapters — none required for a minimally conformant Runtime.

**Prohibited:** ambient authority; direct shared mutable actor state; mutable messages; mutable capability authority; implicit capability inheritance; actor communication without validated send authority; provider credentials exposed directly to ordinary actors; execution without a single owning actor; Runtime extensions bypassing trusted enforcement; spontaneous authority creation.

## 23. Deferred Architecture

| Area | Deferred | Contract ARCH-002 establishes |
|---|---|---|
| Scheduler Architecture | Dispatch-order algorithm, fairness implementation, priority | Dispatch-selection interface; no-starvation requirement; admission-order preservation |
| Lifecycle Architecture | Supervision/restart policy, backoff strategy | State model and legal-transition set (§15); revalidate-on-resume requirement |
| Knowledge Architecture | Store design, indexing, query semantics | Actor-mediated, capability-gated access pattern |
| Storage Architecture | Persistence technology, snapshot format, durability guarantees | Persistence/Restoration Service interface boundary; no silent reinstatement of revoked authority |
| Provider Architecture | Adapter patterns, retry and circuit-breaking policy | Provider adapters as ordinary, capability-scoped actors, never ambient |
| Distributed Runtime | Cross-host routing, remote capability cryptography, partition handling | Location-transparent logical identity; one abstract capability model, two representations |
| Resource Accounting | Budget and quota algorithms | Execution Context budget field; capability-level constraint enforcement |
| Security Architecture | Threat modeling, cryptographic selection, hardening | The four trusted-core guarantees; structural non-forgery and integrity requirements |
| Boot Architecture | Concrete bootstrap sequence mechanics, host-startup integration | Bootstrap as a one-time, disclosed, Runtime-privileged act (§11, step 1) |

No unresolved Runtime contradiction is hidden in this table; each deferred area has a stated contract it must satisfy when addressed.

## 24. Consequences and Risks

**Positive consequences:** a precise, implementation-agnostic blueprint enabling independently built, conformant runtimes; isolation and audit properties inherited directly, and provably, from the constitutional model; a trusted core small enough to reason about as a whole, with every element mapped to one of four named guarantees.

**Costs and risks, named explicitly:** Capability Authority and Message Gateway are natural contention points — every message send and every capability operation passes through them, a real centralization-bottleneck risk requiring careful attention (sharding or replication of internal state while preserving unified trust properties) at implementation time, not resolved here. The Actor Directory, if implemented naively under high actor churn, risks becoming its own bottleneck. Per-message capability validation is a genuine, non-zero cost whose acceptability depends on implementation efficiency. Bounded mailboxes mitigate but do not eliminate the real behavioral consequences of overflow and backpressure policy choices deferred to later work. Tiered audit logging mitigates but does not eliminate event-volume concerns at high throughput. Seven trusted-core components is a non-trivial size; each has been justified against one of four guarantees here, but future review should periodically re-examine whether further reduction is possible, consistent with `.ai/ARCHITECTURAL-CONTEXT.md`'s ongoing simplicity discipline. The boundary between dispatch *mechanism* and dispatch *policy* — and the equivalent boundaries elsewhere — is easy to blur in actual implementation; this is a standing implementation-discipline risk that specification alone cannot fully prevent. The level of indirection this architecture requires (logical versus instance identity, capability versus binding, envelope versus content) carries genuine conceptual overhead for engineers learning the system, accepted here for the guarantees it purchases. Distributed mechanics are deliberately not designed now, consistent with a local-only Minimal Runtime Profile — an intentional scope boundary, not an oversight.

## 25. Diagrams

**Runtime boundary and layers**

```
Constitutional Layer:  Actor | Capability | Message | Execution Semantics
        |
        v realized by
Runtime Layer
  +-- Trusted Core: Capability Authority, Actor Host, Message Gateway,
  |                 Execution Coordinator, Lifecycle Guardian,
  |                 Audit Emitter, Host Adapter
  +-- Services (replaceable): Scheduler, Actor Directory, Audit Pipeline,
                               Persistence/Restoration, provider adapters,
                               knowledge/storage mediators
        |
        v hosted by
Infrastructure / Host Platform:  process & thread scheduling, memory,
                                  devices, raw network, durable storage
```

**Runtime component architecture**

```
        Scheduler (policy) --selects--> Execution Coordinator
                                              |
Actor --submits--> Message Gateway --admits--> Mailbox --dispatch--> Execution Coordinator
                       |                                                   |
                 Capability Authority <---------- validates/binds ---------+
                       |
                 Actor Host <---------- isolation, identity ----------+
                       |
                 Lifecycle Guardian <---- suspend/resume/terminate ---+
                       |
                 Audit Emitter <---- every trusted component --------+
                       |
                 Host Adapter <---- sole seam to physical resources
```

**Minimal constitutional execution cycle**

```
Init -> Define Actor -> Create Actor -> Issue Capability -> Bind Capability
  -> Create Message -> Validate Send Authority -> Validate Envelope
  -> Admit to Mailbox -> Select for Dispatch -> Construct Execution Context
  -> (Revalidate if resumed) -> Actor Processes Message -> State Transition
  -> Outbound Messages (recurse) -> Emit Audit Event -> Complete Execution
  -> Actor -> {Idle | Suspended | Failed | Terminated} -> ... -> Shutdown
```

**Message and capability flow**

```
Sender Actor
   | holds Capability C (target=Recipient, ops={send:TypeX})
   | constructs Message M (type=TypeX, dest=Recipient, envelope)
   v
Message Gateway --validate(C authorizes M)--> Capability Authority
   | valid
   v
Mailbox(Recipient) --admit(M)-->
   | dispatch
   v
Execution Context(actor=Recipient, message=M, bindings=Recipient's active set)
   v
Recipient Actor processes M --may produce--> Outbound Message M'
   (M' capability-transfer section populated only via attenuation, never widened)
```

**Actor lifecycle**

```
Defined -> Initializing -> Idle <-> Ready -> Executing -> Idle
                                                 |
                                                 +-> Suspended --(revalidate)--> Idle/Ready
                                                 |                 |
                                                 |                 +--(fail)--> Terminated
                                                 +-> Failed --(supervision policy, deferred)--> {Initializing | Terminated}
                                                 +-> Stopping -> Terminated
```

**Trusted core versus replaceable Runtime services**

```
TRUSTED CORE (minimal, unbypassable)          REPLACEABLE SERVICES (policy, extensible)
  Capability Authority                          Scheduler
  Actor Host                                    Actor Directory
  Message Gateway                               Audit Pipeline
  Execution Coordinator                         Persistence/Restoration
  Lifecycle Guardian                            Provider adapters
  Audit Emitter                                 Knowledge/storage mediators
  Host Adapter                                  Monitoring, policy engines
  ---------------------------------------------------------------------
  Failure here => fail-stop                     Failure here => contained,
  (Runtime-integrity threatening)                  never threatens the core
```
