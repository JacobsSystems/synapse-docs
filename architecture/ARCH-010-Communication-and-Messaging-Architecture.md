---
document_id: ARCH-010
title: Communication & Messaging Architecture
project: SynapseOS
specification: SynapseOS — the architecture governing discrete, capability-authorized communication (point-to-point messaging, request/response, notifications, publish/subscribe, narrow event distribution) as a family of Effect Providers realized entirely under the existing Effect Runtime Architecture, extending it without redesign
version: 1.0.0
status: Approved
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available; GOV-003 §3.4, §3.5) — Architecture Review Board Session 002 (ARB-002): zero Critical, zero Major, six Minor findings; recommendation APPROVE WITH REQUIRED AMENDMENTS; all six resolved by the v0.1.1 correction pass
approval_authority: Denver Jacobs, Founder, exercising the interim Class B (architectural) approval default under GOV-003 §3.2 (Chief Architect role vacant)
created: 2026-07-29
last_updated: 2026-07-29
classification: Public
related_documents:
  governance:
    - GOV-003 (Governance Model) — roles and authority this document draws on without redefinition
    - GOV-013 (Engineering Lifecycle, Approved via evidence commit `3ac7cfb`) — governs this document's own Architecture Authoring stage (§6.8); this document's Draft status and unpopulated Approval Status table are the expected state at this stage, not an omission
  standards:
    - STD-001 (Documentation Standards) — filename (§8), identifier (§7), and Required Core Structure (§15) this document follows
    - STD-031 (Engineering Work Order Lifecycle Standard, v0.2.1, Approved) — governs the Engineering Work Order lifecycle any future Communication Provider implementation will pass through; not itself extended or restated here
  architecture:
    - ARCH-001 (Constitutional Architecture) — the four constitutional guarantees this document inherits without redefinition
    - ARCH-002 (Runtime Architecture) — Trusted Core boundary, `Message` shape, and the single admission pipeline this document reuses entirely unmodified
    - ARCH-008 (Effect Runtime Architecture, v0.5.0, Approved) — the primary architecture this document extends; Communication & Messaging is realized entirely as a family of Effect Providers under ARCH-008's own, unmodified Provider Architecture, Effect lifecycle, capability model, audit model, failure model, and retry architecture
  consolidation:
    - ACR-002 (Communication Architecture Consolidation Review) — the primary evidentiary and decision basis for this document's own scope, boundaries, and normative content; cited throughout as the immediate authority this document formalizes
  review:
    - ARB-002 (Architecture Review Board Session 002) — independent review of ARCH-010 v0.1.0; zero Critical, zero Major, six Minor findings; recommendation APPROVE WITH REQUIRED AMENDMENTS; all six findings resolved by the v0.1.1 correction pass this Founder Approval (FA-002) relies on
  research:
    - ARR-002 (Communication Systems Research) — the twenty-system comparative evidence base underlying ACR-002's own findings
    - RSR-002 (Communication Systems Research Synthesis Review) — the independent synthesis of ARR-002 that ACR-002 itself consolidated
  reports:
    - PAR-001 (Provider Architecture Validation Review) — the empirical basis for this document's own confidence that the Provider Architecture requires no modification to host a Communication Provider; also the source of the Runtime execution-model dependency this document explicitly defers (§21)
  governance_charters:
    - ACT-003 (Act 3 Authorization and Charter) — the predecessor phase whose Provider Architecture validation (PAR-001) this document builds directly on; this document does not author or presuppose any "Act 4" charter of its own
  engineering:
    - EWO-017 through EWO-021 (the five reference Effect Providers — HTTP, Filesystem, Process, SQLite, Timer & Scheduler — whose empirical validation, consolidated by PAR-001, is the direct precedent this document's own Provider-responsibility sections extend to the communication domain)
  rfcs: None
  adrs: None
  roadmap: None
  operational: None
  source_artifacts: None
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-010 — Communication & Messaging Architecture

*Filename pattern: `ARCH-010-Communication-and-Messaging-Architecture.md` (per STD-001 §8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

> **Status notice.** This document is **Approved** (v1.0.0, following Founder Approval on 2026-07-29, recorded as FA-002). It is the authoritative Communication & Messaging Architecture for SynapseOS. Drafting, staging, committing, or pushing this document did not itself constitute its approval — the recorded Founder Approval act (§26) did. See §26 (Approval Status).

## 1. Purpose

This document defines the SynapseOS architecture for **discrete communication**: point-to-point messaging, request/response, notifications, publish, subscribe, and narrow event distribution, realized entirely as an application of the existing Effect Runtime Architecture (ARCH-008 v0.5.0). It defines how SynapseOS itself models communication — what concepts are architecturally load-bearing, who owns what, and what a future Communication Provider must and must not do. It does **not** define how any specific product (RabbitMQ, Kafka, MQTT, NATS, Redis Streams, Amazon SQS/SNS, Azure Service Bus, Google Pub/Sub, or any other system) operates internally; those remain implementation details entirely owned by whichever concrete Communication Provider a future Engineering Work Order builds, exactly as ARCH-008 §11 already establishes generally for every provider domain.

This document formalizes decisions already reached through the evidence chain that precedes it — ARR-002 (Communication Systems Research) → RSR-002 (its own independent synthesis) → ACR-002 (Architecture Consolidation Review) — and does not reopen or re-derive them from first principles. Per this document's own Evidence Hierarchy (governing authorship order: ARCH-008 → ACR-002 → RSR-002 → ARR-002 → PAR-001), every normative statement below traces to a specific, cited source, and every unresolved question found in that evidence is disclosed explicitly here rather than silently decided.

## 2. Scope

**In scope — the discrete-delivery family** (ACR-002 §2's own architectural classification): point-to-point messaging, request/response (realized as the existing Effect request/reply pattern, not a new mechanism), notifications, publish, subscribe, and narrow event distribution (a publish is one ordinary Effect; fan-out to multiple independent subscribers is delegated to the external system a Communication Provider connects to, not tracked by SynapseOS itself) — per ACR-002 §3.

**Explicitly out of scope — acknowledged only for boundary and future-compatibility purposes, not architected here:** streaming execution, continuous event sources, replay architecture, workflow execution, and durable execution history (aligned with §4's own identical enumeration). See §15 (for Replay specifically) and §21 (Future Compatibility, comprehensively) for exactly what is deferred and why. This document does not define execution semantics for any of these, consistent with ACR-002 §3's own explicit scoping recommendation and with the task authorizing this document.

**Out of scope, permanently, not merely deferred:** selection or endorsement of any specific messaging technology; wire protocols; APIs, Rust types, or method signatures; numeric policy of any kind (timeout durations, retry counts, throughput targets); any new Trusted Core component, admission path, or capability mechanism.

## 3. Goals

1. Establish Communication as an ordinary application of the existing Provider Architecture (ARCH-008 §11) — no new Trusted-Core or Runtime-privileged concept.
2. Define the small set of technology-neutral concepts the evidence chain found genuinely load-bearing (Topic, Delivery Guarantee, Ordering, Communication Provider), sufficient to guide a future concrete Provider without prescribing its implementation.
3. Preserve ARCH-008 (and, through it, ARCH-001/ARCH-002) entirely unmodified — zero redesign, zero reopened invariant.
4. Provide sufficient normative (MUST-level) guidance that a future Engineering Work Order could implement a first concrete Communication Provider on the identical basis EWO-017 through EWO-021 already, repeatedly, successfully demonstrated for five other domains.
5. Disclose, rather than silently omit, every question the evidence chain found genuinely unresolved.

## 4. Non-Goals

This document does not:

- select, endorse, or recommend RabbitMQ, ActiveMQ, IBM MQ, Kafka, Pulsar, Redpanda, NATS, ZeroMQ, MQTT, SQS, SNS, Azure Service Bus, Google Pub/Sub, Redis Streams, EventStoreDB, WebSockets, SSE, or any other technology researched by ARR-002, or any technology not researched by it;
- define streaming execution, continuous event sources, replay architecture, workflow execution, or durable execution history (§21);
- define wire protocols, APIs, Rust structs, traits, method signatures, or field layouts;
- introduce any new Trusted Core component, a second admission pipeline, a second capability-authorization mechanism, or any Runtime-privileged construct (ARCH-002 §5, ARCH-008 §13, unmodified);
- amend, reinterpret, or reopen any invariant, ownership boundary, or guarantee stated in ARCH-001, ARCH-002, or ARCH-008;
- fix numeric policy of any kind — timeout durations, retry ceilings, throughput or latency targets, message-size limits;
- design Runtime Control API endpoints or Control Centre user interfaces (ARCH-008 §29's own identical, already-standing deferral, unaffected by this document);
- resolve the Runtime execution-model dependency ACR-002 §6/§9 and PAR-001 identified for continuous event sources — that dependency's resolution belongs to a separate, future architecture effort this document depends on but does not perform (§21).

## 5. Architectural Principles

The following principles govern this document and are, in every case, either a direct restatement of an existing ARCH-008 principle applied to a new domain, or a principle ACR-002 §10 found strongly evidence-supported:

- **Communication is capability-based** (ACR-002 §10; ARCH-008 §14, unmodified) — every Communication operation is authorized exactly as any other Effect operation, with no domain-specific exception.
- **Delivery guarantees are explicit, never implicit** (ACR-002 §10/§11 invariant candidate 1; ARR-002 §5 (item 2)'s own near-universal cross-system finding) — a Communication Effect's delivery guarantee is a declared property, never an ambient default.
- **Ordering, where offered, is always explicitly scoped; global ordering is never implied** (ACR-002 §10/§11 invariant candidate 3; ARR-002 §5 (item 4)/RSR-002 §4's own universal finding across every system researched).
- **Messages are immutable; transport never redefines message semantics** (ACR-002 §10; already an existing property of ARCH-002's own `Message` type — this principle states explicitly, for Communication, what is already true everywhere else in this architecture).
- **Transport does not define semantics** (ACR-002 §10) — the identical Effect abstraction that already sits, unmodified, above HTTP, filesystem, operating-system process, SQL, and timer transports (EWO-017–021) sits above Communication with no special case.
- **Discrete delivery and continuous observation are architecturally distinct concerns** (ACR-002 §2's own central finding, and its own proposed additional principle, §10) — this document addresses the former only, deliberately, and does not attempt to unify the two into one mechanism.
- **No redesign of existing architecture** (ARCH-008 §8, restated here without modification) — every mechanism this document reuses (the admission pipeline, the capability model, the Effect lifecycle, the audit model, the retry architecture) is reused exactly as ARCH-008 v0.5.0 already defines it; this document introduces no new Trusted Core concept and reopens no existing invariant.

## 6. Communication Model

A **Communication Effect** is an ordinary Effect (ARCH-008 §7, §15–§16), dispatched by a requesting actor to a **Communication Provider** — an ordinary, capability-scoped `synapse_api::Actor` implementation (ARCH-008 §11) — through the existing, unmodified admission pipeline (ARCH-002/ARCH-008 §13). For the discrete-delivery family this document scopes (§2), a Communication Effect Attempt completes synchronously, within one `handle()` call, returning a typed result exactly as every one of the five existing reference Providers already does (EWO-017–021, PAR-001).

No new Runtime mechanism, no new admission path, no new Trusted Core component, and no new Effect-level or Attempt-level lifecycle state is introduced by this document (§15). Communication is not a parallel architecture alongside ARCH-008's Provider Architecture — it is a **named domain within it**, on the identical basis HTTP, filesystem, process execution, SQL, and timer scheduling already are.

```text
Requesting Actor (Producer)
    │
    │  Communication Effect request (capability-authorized)
    ▼
Runtime — Effect Coordinator (bookkeeping only, unmodified)
    │
    ▼
Communication Provider (ordinary Provider Actor)
    │
    │  translates to/from the external system's own protocol
    ▼
External Communication System (out of this document's own scope)
    │
    ▼
Typed Result
    │
    ▼
Requesting Actor (Consumer)
```

This diagram states nothing beyond what ARCH-008 §11/§13 already establish generically; it is included for clarity, not as a new architectural claim.

## 7. Communication Concepts

Only concepts the evidence chain found genuinely supported (ACR-002 §5) are defined here. Each is stated as either a reuse of an existing ARCH-008/ARCH-002 concept, or, where genuinely new, disclosed as such.

- **Message** — reuses ARCH-002's own `Message` type entirely unmodified. Communication introduces no new envelope or header/payload concept; the existing header-field/`payload` separation already satisfies it (ACR-002 §5).
- **Communication Provider** — an ordinary Provider Actor (ARCH-008 §11), specialized to a communication domain, subject to every rule this document and ARCH-008 already establish for any Provider — no new actor category (§12).
- **Producer** — the requesting actor initiating a Communication Effect. Not a new role: this is the existing "requesting actor" (ARCH-008 §9's Ownership Model), named here for domain clarity only (ACR-002 §5).
- **Consumer** — the requesting actor that receives a Communication Effect's own typed result, for the discrete-delivery family this document scopes. This document does not define "consumer" as an ongoing relationship, a durable subscription, or a tracked position — that meaning is explicitly deferred (§21), consistent with ACR-002 §5's own finding that "Consumer" is only fully load-bearing once continuous observation is in scope.
- **Topic** — a named, capability-authorized destination a Communication Provider exposes, to which Communication Effects may be addressed for point-to-point or fan-out delivery. This is the one concept ACR-002 §5/§11 found genuinely new — no existing ARCH-008 concept models a dynamically-named, potentially-many-subscriber addressable channel distinct from a capability's own fixed Provider-target binding. Defined normatively in §11.
- **Delivery Guarantee** — a declared, explicit, per-request-or-per-capability property (at-most-once, at-least-once, or exactly-once-within-Provider-boundary). Defined normatively in §9.
- **Ordering** — a declared, explicitly-scoped property; never implied as a global default. Defined normatively in §10.
- **Correlation** — reuses ARCH-008 §23.4's existing mechanism entirely unmodified: `Message.correlation` is already Runtime-populated with the requesting Effect's own Effect ID for every Effect-request message, independently re-verified against the current ARCH-008 text during this document's own preparation. Communication introduces no new correlation concept or field.

**Concepts deliberately not introduced**, per ACR-002 §5's own classification, because they are either already satisfied by an existing concept, are Provider-internal implementation detail, or are contingent on the deferred continuous-observation family: Envelope (derived from `Message`), Queue (implementation-specific — a Provider's own internal concern), Stream, Subscription, Replay, and Offset/Cursor (all contingent on the deferred family, §21). Dead Letter is discussed in §17 but not defined as a new architectural concept by this document.

## 8. Message Model

A Communication Effect's request and result content is ordinary `Message.payload` data, encoded via that Provider's own private, hand-written wire format (ARCH-008 §13.1) — the identical, non-shared-serialization pattern every existing reference Provider already establishes (EWO-017–021).

**Normative:**

- A Communication message MUST be treated as immutable once constructed; a Communication Provider MUST NOT mutate a message's own content after admission (§5's own immutability principle, restated normatively here).
- Correlation MUST use the existing `Message.correlation` field (ARCH-008 §23.4). A Communication Provider MUST NOT introduce a parallel, Provider-specific correlation mechanism where the existing field is sufficient.
- A Communication Provider's own typed request/response shapes are that Provider's own choice (ARCH-008 §13.1: "a Provider requiring a materially different result shape is free to choose its own wire representation") — this document does not prescribe one.

## 9. Delivery Guarantees

**Normative:**

- A Communication Effect's delivery guarantee MUST be a declared, explicit property of the request or the presented capability. It MUST NOT be an implicit, unstated, system-wide default (ACR-002 §10/§11 invariant candidate 1; traced through RSR-002 §9 to ARR-002 §5.2's cross-system finding, itself independently re-confirmed against ARR-002's own text during this document's preparation).
- The declared delivery-guarantee vocabulary is: **at-most-once**, **at-least-once**, or **exactly-once-within-Provider-boundary**.
- A Communication Provider claiming exactly-once-within-Provider-boundary MUST scope that claim explicitly to its own transactional boundary, or state it as at-least-once-plus-idempotency. A Provider MUST NOT claim an unscoped, absolute exactly-once guarantee (ACR-002 §10/§11 invariant candidate 2; ARR-002 §5 (item 3)'s own finding that every one of twenty systems researched either scopes this term narrowly or reduces it to at-least-once-plus-idempotency, with zero counterexample).
- Idempotency declaration for a Communication operation reuses ARCH-008 §23 entirely unmodified — a Communication Provider declares `Idempotent`/`NonIdempotent`/`Unknown` for its own operation exactly as any other Provider domain already does.
- The concrete mechanism by which a delivery-guarantee declaration is expressed (a request field, a capability constraint, or another means) is an implementation decision for a future Engineering Work Order, on the identical basis ARCH-008 §33 already defers comparable mechanism questions generally — not decided here.

## 10. Ordering Model

**Normative:**

- Ordering MUST NOT be assumed, implied, or claimed as a default property of any Communication Effect.
- Where a Communication Provider offers ordering, that ordering MUST be explicitly scoped to a named unit smaller than the whole system (for example, a specific Topic, or a specific correlation group) — global, system-wide ordering MUST NOT be implied or claimed by any Communication Provider (ACR-002 §10/§11 invariant candidate 3; RSR-002 §4's own finding, independently confirmed against every one of the twenty systems ARR-002 researched, that no system offering ordering at all offers it unscoped).
- The absence of a stated ordering scope for a given Communication Provider or operation MUST be read as "no ordering is guaranteed" — it MUST NOT be inferred, by a requesting actor or by any future architecture, as "ordering is assumed."

## 11. Topic Model

**Descriptive.** A Topic is the one genuinely new concept this evidence chain identified (ACR-002 §5, §12 Mandatory Candidate ADR) — no existing ARCH-008 concept models a named, potentially-dynamically-created, potentially-many-subscriber addressable destination distinct from a capability's own fixed Provider-target binding (ARCH-008 §14).

**Normative:**

- A Topic MUST be addressed by name, through a Communication Provider, exactly as any other capability-gated Provider operation is addressed — a Topic is never itself a Trusted Core concept, a Runtime-privileged construct, or an admission-pipeline concept distinct from the existing `Message`/capability model.
- A Communication Provider MAY expose fixed Topics (defined at Provider configuration time) or dynamically-named Topics (named at request time); this document does not decide which any given Provider must support.
- Fan-out delivery to multiple independent subscribers of one Topic is, for the scope this document defines (§2), the responsibility of the external system a Communication Provider connects to. SynapseOS itself does not track independent per-subscriber consumption state under this document — doing so would require the continuous-observation family this document explicitly defers (§21).

**Explicitly left open — Mandatory Candidate ADR (§23):** whether authorizing access to a *dynamically-named* Topic requires a new capability dimension beyond ARCH-008 §14's existing target-binding model, or is satisfied by binding a capability to the Provider itself, with the Provider enforcing per-Topic scope internally — the identical pattern the SQLite Provider's own per-connection-handle ownership already establishes (EWO-020, `SqlError::UnknownHandle`-equivalent per-handle enforcement). ACR-002 §4/§7/§12 found this question directly blocking even the narrow, in-scope event-distribution case, and this document does not resolve it.

## 12. Provider Responsibilities

Directly extending ARCH-008 §11's own list to the communication domain, on the identical basis PAR-001 already confirmed for five other domains:

**Communication Providers MUST:**

- be ordinary `synapse_api::Actor` implementations, exactly as ARCH-008 §11 already requires of any Provider Actor — no new actor category is introduced by this document;
- own their own external connection(s), credentials, and wire protocol exclusively (ARCH-008 §27, unmodified) — never shared with, or visible to, the requesting actor, the Effect Coordinator, or Runtime;
- translate a SynapseOS Communication Effect request into the specific external system's own protocol, and that system's own response back into a typed SynapseOS result, entirely within their own implementation;
- enforce their own provider-specific mechanisms (queue semantics, exchange routing, topic partitioning, subscription modes, or any other product-specific behavior) entirely internally — such mechanisms MUST NOT be exposed as a SynapseOS architectural concept (§4's own permanent non-goal);
- declare their own Provider Classification (ARCH-008 §11.3); **Stateful** is the expected common case for a Communication Provider holding an open connection or subscription handle, on the identical, empirically-confirmed basis the SQLite Provider already establishes (EWO-020, PAR-001);
- reuse the existing provider error model (`EFFECT_PROVIDER_RESULT_FAILED`, ARCH-008 §18.1) for any ordinary, retry-eligible business or operation failure — never `Err` from `handle()` for such a case, which remains reserved for a genuine instance-level fault;
- remain subject to the Mandatory Provider Isolation Rule without exception (§19).

**Communication Providers MUST NOT:**

- expose a product-specific concept (an "exchange," a "partition," a specific quality-of-service level) as a SynapseOS architectural concept — such concepts remain internal to that Provider's own implementation, never surfaced through this architecture;
- assume that a capability authorizing one Topic or operation grants access to any other Topic or operation (no amplification, ARCH-008 §14, unmodified);
- invoke another Effect Provider directly, under any circumstance (§19).

## 13. Runtime Responsibilities

Directly restating ARCH-008 §9.1 for the communication domain — this document introduces **zero** new Runtime responsibility.

**Runtime MUST**, for a Communication Effect exactly as for any other Effect:

- mediate every Communication Effect request and outcome through the existing, unmodified admission pipeline (ARCH-002 §13, ARCH-008 §13);
- obtain or verify the required capability authorization from Capability Authority before a Communication Effect is dispatched to a Communication Provider;
- coordinate timeout and cancellation for a Communication Effect Attempt through the existing, unmodified Effect Coordinator (ARCH-008 §20–§21), with no Communication-specific exception;
- cause every mandatory Communication-related audit event (§16) to be truthfully emitted, in the order the underlying facts genuinely occurred.

**Runtime MUST NOT:**

- perform any communication-specific external operation itself;
- decide Communication-specific authorization — that remains Capability Authority's own, entirely unmodified responsibility;
- gain any new decision authority, orchestration responsibility, or communication-domain-aware logic of any kind by virtue of this document.

This document's own §12/§13 split is a direct, deliberate restatement of the boundary ARCH-008 §9/§9.1/§11 already draws — stated here explicitly so a future Communication Provider implementer, and a future Independent Architecture Review, has a domain-specific checkpoint without any ambiguity about which side of the line a given behavior belongs on.

## 14. Capability Integration

Reuses ARCH-008 §14 entirely, without modification.

**Normative:**

- Communication capability operation strings follow the existing `effect.<domain>.<operation>` structure (ARCH-008 §14) — for example, `effect.messaging.publish`, `effect.messaging.subscribe` — with `<domain>` naming the specific Communication Provider's own domain, exactly as `effect.sql.*` and `effect.timer.*` already do for their own domains (EWO-020, EWO-021).
- Each `effect.<domain>.<operation>` string remains a distinct, separately-grantable capability. Possessing one operation MUST NOT imply possessing another — the identical operation-specific least-privilege rule ARCH-008 §14 already establishes generically, with no Communication-specific exception.
- Capability validation for a Communication Effect occurs fresh, at the moment of dispatch, never cached — identical to every existing Effect domain (ARCH-008 §14).
- Per-Topic authorization granularity beyond fixed Provider-target binding is not decided by this section — see §11's own disclosed open question.

## 15. Effect Integration

**Normative:**

- A Communication request MUST be modeled as an ordinary Effect, using the existing, entirely unmodified Effect lifecycle (ARCH-008 §15–§16): `Requested` → `Denied` or a dispatched Effect Attempt; `RetryScheduled` as the Effect-level transitional state; `Dispatched` → `Executing` → `{Completed | Failed | Cancelled | TimedOut | ProviderLost}` at the Attempt level.
- A Communication Effect Attempt reaches `Completed` once its Provider genuinely, synchronously produces a typed result within one `handle()` call — identical in kind to every one of the five existing reference Providers (EWO-017–021).
- This document introduces **no new Effect-level state, no new Attempt-level state, and no new terminal outcome**.
- Idempotency declaration (ARCH-008 §23) and Retry Architecture (ARCH-008 §19–§19.4) apply to Communication Effects entirely unmodified — a Communication Provider declares its own operation's idempotency classification exactly as any other Provider domain, and retry-decision authority remains exclusively the Effect Coordinator's own, informed by the requesting actor's retry intent and the Provider's own idempotency declaration, with no Communication-specific exception.
- Timeout (ARCH-008 §20) and Cancellation (ARCH-008 §21) architectures apply to Communication Effects entirely unmodified, with no Communication-specific exception — the identical late-signal-discard discipline (a stale completion arriving after a terminal outcome is already recorded is discarded and truthfully audited) applies to Communication exactly as it already does to the five existing domains.
- **Replay** (re-observing an already-`Completed` Effect Attempt's own historical result) is explicitly **not** modeled by the existing Effect Attempt lifecycle, and this document does not extend that lifecycle to model it. The Effect Attempt model's own rule that a given attempt reaches no more than one terminal outcome, never reopened (ARCH-008 §16.2), remains completely unchanged and unextended by this document. Replay is explicitly deferred (§21).

## 16. Audit Integration

Reuses ARCH-008 §17 entirely, without modification.

**Normative:**

- Communication introduces new `event_type` string values only where a future Engineering Work Order finds them useful (for example, illustrative — not fixed here — `effect.communication.published`), never a new field on the existing `AuditEvent` shape and never a parallel, Communication-specific audit mechanism.
- Every mandatory Effect-level audit event ARCH-008 §17 already requires (`effect.requested`, `effect.dispatched`, `effect.completed`, `effect.failed`, `effect.cancelled`, `effect.timeout`, `effect.denied`) applies to Communication Effects identically to every other Effect domain, with the identical ordering discipline (no event may claim a later state before it has genuinely occurred).

**Disclosed, inherited gap, not introduced or closed by this document:** ARCH-008 §17.1 already discloses that the current `AuditEvent` structure carries no timestamp field and no explicit correlation-identifier field of its own. Communication inherits this gap unmodified. ACR-002 §4 found this gap is likely to be felt more acutely in a correlation-heavy domain than in the five domains already validated — this document records that observation but does not close the gap, consistent with ARCH-008 §17.1's own statement that closing it remains an implementation-phase concern for a future, separately authorized Engineering Work Order.

## 17. Failure Semantics

Reuses ARCH-008 §18/§18.1 entirely, without modification.

**Normative:**

- A Communication Provider's own business/operation failure (for example, a broker rejecting a publish, a connection refused, a malformed topic name) MUST be reported via the existing provider error model (`EFFECT_PROVIDER_RESULT_FAILED`, ARCH-008 §18.1) — never via `Err` from `handle()`, which remains reserved for a genuine instance-level fault, exactly as every existing reference Provider already demonstrates.
- This document does not introduce a communication-specific failure lifecycle, a new terminal Effect-Attempt outcome, or a parallel failure taxonomy. The existing failure-category table (ARCH-008 §18: provider business/operation failure, provider actor execution failure, admission failure, authorization denial, timeout, cancellation, provider actor lost, audit-infrastructure failure, Runtime-infrastructure failure) governs Communication failures without addition.

**Explicitly left open — Recommended Candidate ADR (§23):** dead-letter representation — a "set aside for inspection, never silently dropped" concept for a Communication Effect that exhausts its own bounded retry — is identified by ACR-002 §5/§12 as a genuinely evidence-supported future need (recurring near-universally across the systems ARR-002 researched, ARR-002 §7). Whether this becomes a new terminal Effect-Attempt outcome, an audit-only convention layered on the existing model, or a Provider-level pattern requiring no architectural change at all, is **not decided by this document**.

## 18. Security Model

Reuses ARCH-008 §26–§27 entirely, without modification.

**Normative:**

- Communication Providers own their own credentials exclusively; Runtime, the Effect Coordinator, and requesting actors never receive them, directly or indirectly (ARCH-008 §27, unmodified).
- Capability operation strings (§14) remain the sole authorization boundary for Communication — classification or metadata of any kind is never itself an authorization mechanism (ARCH-008 §14/§24, unmodified).
- No ambient network, filesystem, or messaging-system access is granted merely because code executes inside the Runtime process; every such access requires a presented, valid, capability-authorized Effect request (ARCH-008 §27, restated without modification for this domain).
- Actor and capability isolation, for Communication exactly as for any other domain, is explicitly **not** process-, operating-system-, or host-level sandboxing (ARCH-008 §27, restated without modification) — a Communication Provider's own implementation code is trusted to comply with its own architecturally granted scope, identical to every other Provider domain; this document introduces no new enforcement mechanism beyond what ARCH-008 §27 already, explicitly declines to provide.

**Descriptive, not decided here:** tenancy and per-destination isolation (recurring, in some form, across nearly every system ARR-002 researched — ARR-002's own Security Model dimension) are not modeled as a new SynapseOS architectural concept by this document. Per §11's own Topic Model, such isolation remains a Communication Provider's own internal enforcement concern unless and until a future ADR (§23) determines a new capability dimension is warranted.

## 19. Provider Isolation

Reuses ARCH-008 §12 entirely, without modification, unconditionally.

**Normative:**

- **Communication Providers MUST NOT invoke other Effect Providers directly, without exception** — including another Communication Provider, and including where both are believed to be trusted, colocated, or maintained by the same party.
- A Communication Provider requiring another Effect, of any domain including another communication operation, MUST submit a new Effect request through Runtime exactly as any ordinary requesting actor would, undergoing fresh capability validation exactly as any other dispatch.
- The Mandatory Provider Isolation Rule's own rationale (ARCH-008 §12: preventing a hidden execution path that bypasses capability authorization, audit emission, Effect identity, timeout enforcement, cancellation, retry controls, and Runtime Control API observability) applies to Communication with no special exception of any kind.

## 20. Compatibility

This document is fully consistent with, and extends without modification: **ARCH-001** (all four constitutional guarantees, inherited without redefinition); **ARCH-002** (Trusted Core boundary, `Message` shape, single admission pipeline, all reused unmodified); **ARCH-008 v0.5.0** (the primary architecture this document extends — every mechanism named in §9–§19 above is reused, never redesigned); and the demonstrated Provider Extension Rules (ARCH-008 §11.4) and reference-Provider precedent (EWO-017 through EWO-021, empirically validated by PAR-001). A future Communication Provider is expected to follow the identical crate structure (`internal.rs`/`wire.rs`/`lib.rs`), private wire-format discipline, and required-test-category precedent every one of the five existing reference Providers already establishes, without modification to that pattern.

## 21. Future Compatibility

The following are explicitly addressed for future compatibility by this document — named here, with the evidence-chain reasoning for each, rather than left silent (ARCH-008 §4/§33's own repeatedly successful disclosed-deferral discipline, applied here). Four of the five are genuine, unresolved deferrals; one (Distributed / cluster-spanning communication) is already confirmed compatible and requires no further resolution — distinguished in its own row below rather than left to be inferred from an undifferentiated list:

| Future capability | Deferred because | Evidence basis |
|---|---|---|
| **Streaming / continuous, non-destructive (offset-based) consumption** | Blocked on a genuine Runtime execution-model dependency: a Provider cannot currently initiate its own future re-invocation from within `handle()` — `Runtime::register_timer` is reachable only by code holding `&mut Runtime` directly, never from inside an Actor's own `handle()`. This is not a communication-domain-specific gap — ACR-002 §9 found it is, more precisely, a general execution-model question shared with any continuously-producing Provider regardless of domain (filesystem watchers, OS notifications, database change feeds, and the already-built Timer Provider, EWO-021, all share this identical property). This document treats resolving that dependency as a **precondition belonging to a separate, future architecture effort**, not a Communication-specific ADR. | PAR-001 §5/§7 (original disclosure); RSR-002 §8 (confirmed as the most consequential open compatibility question); ACR-002 §6/§9 (reframed as a general Runtime question, not a Communication-domain one) |
| **Replay** | Structurally meaningless without non-destructive consumption already being in scope; additionally, would require extending the Effect Attempt model's own "exactly one terminal outcome, never reopened" rule (ARCH-008 §16.2) in a way this document does not attempt, since the underlying prerequisite (streaming) is itself deferred. | ACR-002 §5/§8 |
| **Workflow / durable execution signalling** (Temporal-/Cadence-style multi-step, crash-resilient program control flow) | Found structurally distinct enough from ordinary message delivery — it concerns durable program control-flow, not message transport — that ACR-002 §3/§12 recommends it be evaluated as its **own, separate future architecture candidate**, not folded into Communication & Messaging Architecture at all. This document does not treat it as an ARCH-010 extension. | ARR-002 §3 (Temporal.io/Cadence profiles); RSR-002 §10; ACR-002 §3/§12 |
| **Distributed / cluster-spanning communication** (already confirmed compatible — not an unresolved deferral) | Compatible in principle — `ActorId`-keying, used throughout this document exactly as ARCH-008 already uses it, is already location-transparent by contract (ARCH-002 §7), the identical property already keeping Temporal Runtime and Persistence Service distributable-in-principle without redesign. No redesign of this document is anticipated for this future capability. | ARCH-008 §29 (identical existing entry for the Provider Architecture generally) |
| **Dead-letter representation as a formal architectural concept** | Additive, not blocking — can follow the identical pattern EWO-017's own `EFFECT_PROVIDER_RESULT_FAILED` addition already demonstrated works well when deferred until a real implementation need makes the concrete shape clear. | ACR-002 §5/§12 (Recommended-tier) |

No redesign of this document is anticipated for any of the five items above once each is separately, explicitly resolved — consistent with the identical confidence ARCH-008 §29 already expresses for its own comparable future-capability list.

## 22. Architectural Invariants

Numbered independently, beginning at 1, as this document's own first invariant set — distinct from, and additive to, ARCH-008's own 51 invariants, which remain entirely unmodified and unrenumbered by this document.

1. A Communication Effect MUST be dispatched through the existing, unmodified admission pipeline; no second admission path for Communication is introduced (§6, §13; ARCH-008 §13).
2. A Communication Provider MUST be an ordinary `synapse_api::Actor`; no new actor category or Runtime-privileged construct is introduced (§12; ARCH-008 §11).
3. A Communication capability operation string MUST follow the existing `effect.<domain>.<operation>` structure and MUST NOT imply possession of any other operation (§14; ARCH-008 §14).
4. A Communication Effect's delivery guarantee MUST be a declared, explicit property; it MUST NOT be an implicit or ambient default (§9).
5. A claimed exactly-once delivery guarantee MUST be scoped to the claiming Provider's own transactional boundary, or stated as at-least-once-plus-idempotency; it MUST NOT be claimed as an unscoped, absolute guarantee (§9).
6. Ordering MUST NOT be assumed as a default property of any Communication Effect; where offered, it MUST be explicitly scoped, and global ordering MUST NOT be implied (§10).
7. A Communication Provider business/operation failure MUST be reported via the existing provider error model (`EFFECT_PROVIDER_RESULT_FAILED`); it MUST NOT be reported via `Err` from `handle()` (§17; ARCH-008 §18.1).
8. A Communication Provider MUST NOT invoke another Effect Provider directly, under any circumstance (§19; ARCH-008 §12).
9. A Communication Provider MUST NOT expose a product-specific mechanism (an exchange, a partition, a quality-of-service level, or any other vendor-specific concept) as a SynapseOS architectural concept (§4, §12).
10. This document's own reuse of the Effect lifecycle, capability model, audit model, failure model, and retry architecture MUST NOT be read as introducing any new state, field, or mechanism to any of them; every such reuse is, and MUST remain, entirely unmodified (§13, §14, §15, §16, §17).

## 23. Rationale

This document's own shape follows directly from ACR-002's central findings, restated here as rationale rather than re-argued:

- **Why the scope is narrowed to discrete delivery.** ACR-002 §2 found the six originally-named communication models (queue-, stream-, broker-, brokerless-, actor-native-, and event-log-oriented) reduce to two architecturally distinct families — discrete-delivery and continuous-observation — with only the first directly supported by existing, empirically-validated precedent (PAR-001). Attempting to architect both in one document risked either weakening the well-proven discrete model or producing an ill-fitting continuous one (ACR-002 §10's own additional principle).
- **Why so little is genuinely new.** Of the concepts examined (§7), only Topic lacks a direct existing analog. Message, Producer, Consumer, Delivery Guarantee, Ordering, and Correlation are each either an existing ARCH-008/ARCH-002 concept applied to a new domain, or a domain-specific name for an existing role — this is the direct, evidenced consequence of PAR-001's own finding that ARCH-008's Provider Architecture generalizes cleanly across genuinely distinct domains (HTTP, filesystem, process execution, relational data, time) without requiring new mechanism each time.
- **Why continuous event sources are deferred to a separate, non-Communication-specific effort.** ACR-002 §9's own central finding — that the property distinguishing a Kafka consumer from a discrete publish is *continuous, externally-triggered re-invocation*, an execution-model property shared by the already-built Timer Provider (EWO-021) and by entirely non-messaging examples (filesystem watchers, USB detection) — means the correct architectural boundary is drawn by *execution shape*, not by *domain*. Bundling this dependency's resolution into ARCH-010 would misplace a general Runtime question under a domain-specific heading.
- **Why workflow/durable execution is named but not designed.** Temporal/Cadence's own event-sourced control-flow model (ARR-002 §3) is structurally closer to "durable program execution" than to "message transport" — RSR-002 §10 and ACR-002 §3/§12 both independently reached this conclusion, and this document adopts it without re-litigation.

## 24. References

Internal:

- ARCH-001 — Constitutional Architecture
- ARCH-002 — Runtime Architecture (§5, §7, §13)
- ARCH-008 — Effect Runtime Architecture, v0.5.0, Approved (§8, §9, §9.1, §11–§11.4, §12, §13–§13.2, §14, §15–§16, §17–§17.1, §18–§18.1, §19–§19.4, §20, §21, §23, §23.4, §24, §26, §27, §29, §33) — the primary authority this document extends
- GOV-003 — Governance Model
- GOV-013 — Engineering Lifecycle (§6.8 Architecture Authoring; §6.9 Architecture Review)
- STD-001 — Documentation Standards (§7, §8, §10, §15)
- STD-031 — Engineering Work Order Lifecycle Standard, v0.2.1, Approved
- ACT-003 — Act 3 Authorization and Charter
- PAR-001 — Provider Architecture Validation Review — empirical basis throughout (§1, §6, §12, §21, §23)
- ARR-002 — Communication Systems Research
- RSR-002 — Communication Systems Research Synthesis Review
- ACR-002 — Communication Architecture Consolidation Review — the immediate, primary decision basis for this document, cited throughout
- EWO-017 (Reference Effect Provider Framework), EWO-018 (Filesystem Provider), EWO-019 (Process Execution Provider), EWO-020 (SQLite Provider), EWO-021 (Timer & Scheduler Provider) — the five reference implementations whose demonstrated pattern this document's own Provider Responsibilities section (§12) extends to the communication domain

Traceability note: every normative section above (§8–§19, §22) cites, at minimum, ACR-002 as its immediate consolidating authority and, wherever an existing mechanism is reused, the specific ARCH-008 section it reuses — consistent with this document's own governing Evidence Hierarchy (ARCH-008 → ACR-002 → RSR-002 → ARR-002 → PAR-001) and with the full ARR-002 → RSR-002 → ACR-002 chain this document formalizes.

## 25. Change History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-29 | Denver Jacobs (AI-assisted) | Initial Draft. Formalizes the discrete-communication-family architectural direction consolidated by ACR-002 (itself built on RSR-002's synthesis of ARR-002's twenty-system research), as an extension of ARCH-008 v0.5.0's own Provider Architecture, Effect lifecycle, capability model, audit model, failure model, and retry architecture — none of which is modified. Introduces one genuinely new concept (Topic, §11) and ten architectural invariants (§22), all evidence-traced. Explicitly, disclosedly defers streaming/continuous observation, replay, and workflow/durable execution signalling (§21), and explicitly identifies two Mandatory Candidate ADRs this Draft does not itself resolve (Topic capability-authorization granularity, §11; the Runtime execution-model dependency underlying continuous event sources, treated as an out-of-document precondition, §21). No architecture prior to this document is amended, redesigned, or reinterpreted. |
| 0.1.1 | 2026-07-29 | Denver Jacobs (AI-assisted) | PATCH correction pass applying exactly the six Minor findings ARB-002 (Architecture Review Board Session 002) accepted with amendment required — no Critical or Major findings existed, and no architectural, normative, or scope content changes. **Finding 1** (§11): corrected an illustrative citation from a nonexistent `SqlError::TimerNotFound` to the SQLite Provider's own actual `SqlError::UnknownHandle`. **Finding 2** (§5): corrected two citations from a nonexistent "ARR-002 §5.2"/"§5.4" subsection form to "ARR-002 §5 (item 2)"/"(item 4)," reflecting that ARR-002 §5 is a flat numbered list, not a sub-headed section. **Finding 3** (§9): corrected a misattributed citation from "ARR-002 §9" (SynapseOS Opportunities, which does not contain the cited finding) to "ARR-002 §5 (item 3)" (Common Architectural Concepts, where the finding actually appears). **Finding 4** (§17): removed an extraneous, unrelated "ACR-002 §9" reference from the dead-letter citation, retaining the accurate "§5/§12." **Finding 5** (§2): corrected an internal cross-reference from "§17/§22" (Failure Semantics and Architectural Invariants, neither of which discusses the streaming/replay/workflow deferral reasoning) to "§15 (for Replay specifically) and §21 (comprehensively)," where that reasoning actually resides. **Finding 6** (§2, §21): aligned §2's own deferred-item enumeration with §4's already-correct, task-specified five-item list; added a lead-in clarification and an inline row label to §21's Future Compatibility table distinguishing the four genuinely unresolved deferrals from the one already-confirmed-compatible item (Distributed / cluster-spanning communication), without altering any underlying architectural conclusion. No concept, principle, invariant, Provider or Runtime responsibility, capability rule, Effect rule, audit rule, or failure rule was added, removed, or altered by this correction pass. A disclosed, uncorrected issue remained: a third occurrence of the same "ARR-002 §5.2" citation-format defect Finding 2 addressed, located in §9's first normative bullet, outside ARB-002's own Finding 2 scope (which named only two bullets in §5) — recorded for a future, separately authorized correction rather than silently fixed or silently ignored. |
| 1.0.0 | 2026-07-29 | Denver Jacobs (Founder) | Founder Approval (FA-002). `status` transitions from `Draft` to `Approved`; MAJOR version set to 1.0.0 marking this as SynapseOS's first approved, operative Communication & Messaging Architecture, per the Founder's own explicit versioning decision in FA-002 (distinct from, and not bound by, ARCH-008's own historical practice of remaining within the 0.x series across its own successive approved dispositions). Governing basis: the complete governance chain ARR-002 → RSR-002 → ACR-002 → ARCH-010 authoring (v0.1.0) → ARB-002 (Architecture Review Board Session 002: 0 Critical, 0 Major, 6 Minor findings, `APPROVE WITH REQUIRED AMENDMENTS`) → correction pass (v0.1.1, resolving exactly the six accepted findings) → this Founder Approval. No architectural, normative, or scope content changed by this disposition — approval is a governance act recording a disposition, not a content edit (GOV-013 §6.16's own principle, applied here on the identical mutable-Approval-Status-table basis ARCH-008 and every approved EWO in this repository already use, per §26 below). The single disclosed, uncorrected citation-format issue noted in the v0.1.1 entry above is acknowledged as an outstanding, non-blocking matter — confirmed not to affect architectural correctness or any normative statement — left open for resolution under normal documentation governance rather than blocking adoption. Implementation of Communication Providers, and Runtime changes explicitly required by approved future Communication work orders, are hereby authorized; no implementation may intentionally diverge from this architecture without an approved ADR or a subsequent architecture amendment. |

## 26. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted (v0.1.0), Corrected (v0.1.1) | 2026-07-29 |
| Independent Architecture Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available; GOV-003 §3.4, §3.5) | `ARB-002 COMPLETE — ARCH-010 INDEPENDENTLY REVIEWED — READY FOR FOUNDER APPROVAL`: 0 CRITICAL, 0 MAJOR, 6 MINOR findings; recommendation **APPROVE WITH REQUIRED AMENDMENTS**; all six findings resolved by the v0.1.1 correction pass, applying exactly those six findings with no architectural, normative, or scope content change | 2026-07-29 |
| Approval Authority | Denver Jacobs, Founder, exercising the interim Class B (architectural) approval default under GOV-003 §3.2 (Chief Architect role vacant) | **Approved** — Founder Approval recorded as FA-002; the single citation-format issue disclosed outside ARB-002's own six-finding correction scope (§25, v0.1.1 entry) is acknowledged as outstanding, confirmed not to affect architectural correctness, any normative statement, or this approval decision, and left open for resolution under normal documentation governance rather than blocking adoption | 2026-07-29 |

This document is now **Approved** (v1.0.0), following Founder Approval recorded directly above, on the identical "ordinary, mutable Approval Status convention" this repository's engineering-tier and most-recently-approved documents use throughout (demonstrated: ARCH-008 itself, and EWO-014/015/016's own Approval Status tables, each populated in place). ARCH-010 is hereby adopted as the authoritative Communication & Messaging Architecture for SynapseOS: future implementation shall conform to it; future architectural modification shall occur only through the established governance process (GOV-013, STD-031); no implementation may intentionally diverge from it without an approved ADR or a subsequent architecture amendment.
