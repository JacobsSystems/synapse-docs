---
document_id: ARCH-009
title: Effect Provider Architecture
project: SynapseOS
specification: SynapseOS — the architecture governing every present and future Effect Provider (ARCH-008 §11), covering exactly the ground ARCH-008 itself reserves or leaves as a future extension point — provider registration/discovery, provider classification, and provider extension rules — without restating what ARCH-008 §§9–31 already, generically define
version: 0.1.0
status: Draft
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Denver Jacobs, Founder, exercising the interim Class B (architectural) approval default under GOV-003 §3.2 (Chief Architect role vacant) — identical basis to ARCH-008's own approval authority
created: 2026-07-29
last_updated: 2026-07-29
classification: Public
related_documents:
  governance:
    - GOV-013 (Engineering Lifecycle, Approved, commit `3ac7cfb`) — this document is an Architecture Authoring output under GOV-013 §6.8, subject to Architecture Review (§6.9) before any approval
  standards:
    - STD-001 (Documentation Standards) — identifier, filing, and versioning rules this document defers to throughout
    - STD-031 (Engineering Work Order Lifecycle Standard, v0.2.1, Approved) — governs any future Engineering Work Order authored against this document; this document is architecture, not an EWO, and is not itself governed by STD-031
  architecture:
    - ARCH-001 (Constitutional Architecture) — actor isolation, capability model, non-amplification
    - ARCH-002 (Runtime Architecture) — Actor Directory, replaceable-service model, Provider Architecture deferral (§23)
    - ARCH-008 (Effect Runtime Architecture, v0.4.3, Approved) — the primary, governing architecture this document complements; see §3 for the complete, load-bearing disclosure of exactly how
  acts:
    - ACT-003 (Act 3 Authorization and Charter, Approved) — names Effect Providers, provider lifecycle, and Runtime external capability as in-scope Act 3 engineering
  engineering:
    - EWO-017 (Reference Effect Provider Framework, implemented, commit `397dded`) — the sole demonstrated evidence this document generalizes from; every claim below is checked directly against this implementation, never assumed
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-009 — Effect Provider Architecture

*Filename pattern: `ARCH-009-Effect-Provider-Architecture.md` (per STD-001 §7–§8).*

> **Status notice.** This document is **Draft**. No Architecture Review (GOV-013 §6.9) has occurred. It is submitted as architecture documentation for review, not as an approved specification.

> **Load-bearing disclosure, stated before anything else in this document.** ARCH-008 (Effect Runtime Architecture, v0.4.3, Approved) already defines, generically and provider-agnostically, almost everything this document was asked to define: Provider Actor Model (§11), Provider Isolation (§12), Effect Request and Result Flow (§13), Capability Architecture (§14), Effect Identity (§15), Effect Lifecycle (§16), Audit Architecture (§17), Failure Semantics (§18), Retry Architecture (§19), Timeout Architecture (§20), Cancellation Architecture (§21), Security Boundaries (§27), and Future Compatibility (§29) — the last of which already names filesystem, database, AI, email, message-broker, cloud, hardware, and plugin providers by name as already-compatible future extensions, requiring no redesign. **This document does not restate any of it.** Doing so would recreate, at a far larger scope, the exact "duplicate lifecycle definition / dual source of truth" defect this project's own Independent Engineering Review of STD-031 (IER-STD-031, F01) already identified and corrected once. See §3 for the complete accounting of what ARCH-008 already governs versus what this document actually adds.

## 1. Purpose

This document defines the architecture governing **provider registration and discovery**, **provider classification by nature** (as distinct from Effect classification by operation-nature, ARCH-008 §24), and **provider extension rules** — the three concerns ARCH-008 itself explicitly reserves, defers, or leaves unaddressed (ARCH-008 §11, §14, §30, §33) — so that a future engineer adding a new Effect Provider has one complete, non-duplicated reference spanning both documents. HTTP (EWO-017) is treated throughout as the one demonstrated example proving the pattern, never as the architectural focus.

## 2. Scope

**In scope:** provider registration and discovery; provider classification by nature (Stateless/Stateful/Streaming/Long-running/Local/Remote/Persistent/Ephemeral) and its architectural implications; provider extension rules (required interfaces, tests, documentation, engineering evidence); a generalized, non-binding description of the typed-request/typed-result pattern EWO-017 demonstrates; organizing (never deciding) the provider concurrency design space ARCH-008 §13/§33 already, deliberately leaves open; and a single, consolidated set of architectural invariants spanning both this document and ARCH-008.

**Out of scope (governed elsewhere, cited not restated):** provider invocation mechanics, capability integration, effect lifecycle, failure/retry/timeout/cancellation semantics, and audit event requirements — all ARCH-008 §§13–21, §17. Any concrete provider implementation (ARCH-008 §4, §30). Any Engineering Work Order (STD-031). Any governance, standards, or ADR change.

## 3. Relationship to ARCH-008 — Consistency Review

**This is the primary, load-bearing consistency-review finding for this document, disclosed rather than silently avoided, per this task's own instruction not to silently resolve a conflict.**

Direct textual comparison against ARCH-008 §§9–31 shows near-total topical overlap with what this document was asked to define:

| Required section (this task) | Already governed by | This document's own treatment |
|---|---|---|
| Provider Definition | ARCH-008 §11 | Cited (§4 below), not restated |
| Provider Invocation | ARCH-008 §13 | Cited (§5), not restated |
| Capability Integration | ARCH-008 §14 | Cited (§6), not restated |
| Provider Lifecycle | ARCH-002 §15 (`ActorState`) and ARCH-008 §16 (Effect/Attempt lifecycle) — two *existing*, distinct state machines | Disentangled, not redefined (§7) — see the conflation risk noted there |
| Failure Model | ARCH-008 §18 | Cited (§8), not restated |
| Retry Model | ARCH-008 §19–§19.4 | Cited (§9), not restated |
| Timeout Model | ARCH-008 §20 | Cited (§10), not restated |
| Cancellation Model | ARCH-008 §21 | Cited (§11), not restated |
| Audit Architecture | ARCH-008 §17 | Cited (§12), with one genuine, disclosed gap noted |
| Concurrency Model | ARCH-008 §13, §31 invariant 16, §33 (deliberately undecided) | Organizes the design space; decides nothing ARCH-008 left open (§14) |
| Result Model | ARCH-008 §13 (ordinary reply message), §24 (classification, no concrete type) | Generalizes EWO-017's demonstrated pattern, non-binding (§13) |
| Provider Registration | **Not governed** — ARCH-008 §11/§14/§30 explicitly state "no new provider registry is introduced" and defer discovery | **New content** (§15) |
| Provider Classification (by nature) | **Not governed** — ARCH-008 §24 classifies the *Effect* (Pure/Read/Write/Transactional/Streaming/LongRunning), never the *Provider* itself | **New content** (§16) |
| Provider Extension Rules | **Not governed** — no existing document states how a future provider is added | **New content** (§17) |

**A further, directly on-point precedent, found during this review and disclosed rather than omitted:** ARCH-008's own frontmatter records a prior task, informally named *"SynapseOS — ARCH-009 Architecture Investigation: Retry Architecture,"* whose own conclusion was **"recommending an ARCH-008 amendment rather than a new document"** — its findings were folded into ARCH-008 v0.4.3 directly; no document named ARCH-009 was ever filed as a result. That investigation concerned only the Retry Architecture; this document was tasked with a scope an order of magnitude larger, spanning nearly every section ARCH-008 already has. The identical reasoning that produced "amend, don't duplicate" for retry alone applies with even greater force here.

**Resolution, disclosed rather than silently applied:** this document proceeds, as explicitly instructed, as a new document named ARCH-009 — but scoped narrowly to genuinely new ground (§§15–17 below), citing ARCH-008 for everything it already, correctly governs. No conflicting or competing definition is introduced anywhere in this document. **This document explicitly recommends that Architecture Review consider whether the genuinely new content in §§15–17 should instead be folded into ARCH-008 as a further amendment (its own established pattern, demonstrated four times already: v0.2.0, v0.3.0, v0.4.0–v0.4.2, v0.4.3), rather than retained as a separate document — consistent with the identical precedent the prior retry investigation already set.** This document does not decide that question itself; it names it explicitly so Architecture Review can.

## 4. Provider Definition

Governed by ARCH-008 §11, unchanged: a Provider is an ordinary, capability-scoped `synapse_api::Actor` — not a Trusted Core component, not a Runtime-privileged construct, no new actor category. Responsibilities, boundaries (the Mandatory Provider Isolation Rule, ARCH-008 §12), lifetime, and ownership are exactly as ARCH-008 §9's ownership table and §11 already state them. `synapse-http-provider`'s own `HttpProvider` (EWO-017) is a direct, conformant instance of this existing definition — nothing here required, or requires, any change to it.

## 5. Provider Invocation

Governed by ARCH-008 §13, unchanged: Effect requests and outcomes are ordinary messages through the existing, unmodified admission pipeline; no second, provider-specific dispatch path exists or may be introduced. EWO-017's `dispatch_http` test helper and `HttpProvider::handle` both confirm this directly — dispatch to the Reference Provider uses `Runtime::request_effect` and ordinary `step()`/`run_until_idle()`, nothing else.

## 6. Capability Integration

Governed by ARCH-008 §14, unchanged: `effect.<domain>.<operation>` capability strings, operation-specific least privilege, ordinary issuance/binding/attenuation/revocation, fresh validation at every dispatch including retries, destination-bound targets. EWO-017 registers `effect.http.get/post/put/patch/delete` — five ordinary instances of the existing `effect.<domain>.<operation>` pattern ARCH-007 §14/§17 already established the granularity for; no new capability mechanism was introduced or is proposed here.

## 7. Provider Lifecycle — Disentangling Two Existing State Machines, Not Defining a Third

This task's own request names a lifecycle — Construction, Ready, Executing, Cancelling, Completed, Failed, Disposed — that, read literally, would conflate two state machines ARCH-008 itself takes considerable care to keep separate (ARCH-008 §15, §16: "Effect identity has exactly two levels, and they MUST NOT be conflated"; the identical discipline extends to a *third* level this task's phrasing risks merging them with):

1. **The Provider Actor's own instance lifecycle** — governed by the existing, unmodified `ActorState` (ARCH-002 §15): `Defined -> Initializing -> Idle -> Ready -> Executing -> {Suspended | Failed | Stopping -> Terminated}`. This is the *actor instance's* own state — whether it exists, is live, and can currently execute — identical for a Provider Actor and for any other actor. Nothing about being a Provider changes this state machine.
2. **The Effect Attempt's own outcome** — governed by ARCH-008 §16.2: `Dispatched -> Executing -> {Completed | Failed | Cancelled | TimedOut | ProviderLost}`. This is the *attempt's* own terminal outcome — one specific dispatched operation's own fate — tracked by the Effect Coordinator, never by the Provider Actor's own instance state.

**These MUST NOT be merged into one "provider lifecycle."** A Provider Actor instance remains `Executing` (state 1) for the duration of exactly one `handle()` call; an Effect Attempt it is currently handling separately, independently reaches its own terminal outcome (state 2) — `Completed`, `Failed`, `Cancelled`, `TimedOut`, or `ProviderLost` — which says nothing about whether the *instance* itself remains `Idle`/`Ready` afterward (ordinarily yes) or has been separately lost (`ProviderLost`, precisely the case where instance state 1 and attempt state 2 diverge: the instance left `Executing` via restart/termination, and *that same fact* is what attempt state 2 records as `ProviderLost`, ARCH-008 §18, §21). "Disposed," similarly, is an instance-level fact (`Terminated`, `ActorState`) — a *completed* or *failed* Effect Attempt does not itself dispose of the Provider Actor instance that handled it, and conflating the two would incorrectly imply every dispatch terminates its own provider.

No new lifecycle state is introduced by this section. This is a clarification of an existing conflation risk, not a new architectural decision (ARCH-008 §31 invariants 9–10 already establish the attempt/Effect distinction this section extends by one further level).

## 8. Failure Model

Governed by ARCH-008 §18, unchanged: provider business/operation failure, provider actor execution failure, admission failure, authorization denial, timeout, cancellation, provider-lost, audit-infrastructure failure, and Runtime-infrastructure failure are the nine architecturally distinct categories, and MUST NOT be conflated. This task's own requested categories map onto them directly: "operational failure" = provider business/operation failure; "capability failure" = authorization denial; "transport failure" and "provider failure" = provider business/operation failure and provider actor execution failure respectively (the exact distinction EWO-017's own provider error model, §8 below, exists to preserve); "cancellation," "timeout," and "retry exhaustion" (an Effect-level outcome once the Retry Decision becomes `Accept`, ARCH-008 §19, §16.1) map directly; "internal defect" corresponds to Runtime-infrastructure or audit-infrastructure failure, unchanged. No new category is required or introduced.

**EWO-017's own provider error model, generalized as the demonstrated pattern for typed failure semantics.** ARCH-008 §16.2's own automatic dispatch-outcome wiring, prior to EWO-017, admitted exactly two paths: `Ok(_)` from `Actor::handle` -> `Completed`; `Err(_)` -> `ProviderLost`. No path existed for a Provider to report an ordinary, retry-eligible failure (ARCH-008 §18's "provider business/operation failure" or "provider actor execution failure" in its non-instance-threatening sense — for example, a refused TCP connection) without being misclassified as `ProviderLost`, which incorrectly implies the instance itself is broken. EWO-017 closes this additively: `synapse_common::EFFECT_PROVIDER_RESULT_FAILED`, a reserved `Message::message_type` value, causes Runtime to record `Failed` instead of `Completed` for the correlated attempt when a Provider's own successful (`Ok`) return includes a message carrying it — leaving the existing `Ok`/`Err` paths for every pre-existing Provider and test completely unaffected. **This document generalizes it as the canonical pattern every future Provider uses to report an ordinary failure that leaves the Provider Actor instance itself healthy** — the specific constant and its exact wiring are implementation, not architecture (ARCH-008 §4), but the *requirement* that such a path exist, and that it never be conflated with `ProviderLost`, is now stated architecturally, closing the one genuine gap this review found in ARCH-008 §18's own otherwise-complete failure taxonomy.

## 9. Retry Model

Governed by ARCH-008 §19–§19.4, unchanged: retry policy (numeric) is actor/capability-owned, never architecture-fixed; the Retry Decision is the Effect Coordinator's sole responsibility, combining eligibility, actor-supplied intent, provider-declared idempotency, and capability-declared limits; retry-eligible outcomes are exactly `Failed`, `TimedOut`, `ProviderLost`. EWO-017's own retry tests (`a_transient_transport_failure_recovers_on_retry`, `retries_are_exhausted_against_a_permanently_unreachable_endpoint`) exercise this existing machinery unmodified, registering `Idempotent` for its own GET/PUT/DELETE-style operations per the requesting actor's own domain knowledge — no new retry mechanism was introduced or is proposed.

## 10. Timeout Model

Governed by ARCH-008 §20, unchanged: timeout policy is actor/capability-owned; scheduling reuses the existing Timer Service; enforcement, cancellation-on-completion, and audit (`effect.timeout`) are the Effect Coordinator's own responsibility; a late provider completion after timeout is discarded, never applied. EWO-017's own timeout test confirms this directly against a genuine, working provider rather than a mechanical double for the first time. No new timeout mechanism was introduced or is proposed.

## 11. Cancellation Model

Governed by ARCH-008 §21, unchanged: cancellation is Runtime-coordinated (explicit actor cancellation, requester termination, Provider Actor lifecycle loss, durable deletion, shutdown, capability revocation); the Effect Coordinator's sole obligation is preventing a cancelled attempt from later completing; external outcome uncertainty is stated as a permanent, general principle, never a new "Unknown" state. EWO-017's own cancellation test confirms the identical late-signal discipline holds against a genuine provider whose real HTTP completion arrives after cancellation was already recorded. No new cancellation mechanism was introduced or is proposed.

## 12. Audit Architecture

Governed by ARCH-008 §17, unchanged in its own event-type requirements: `effect.requested`, `effect.dispatched`, `effect.completed`, `effect.failed`, `effect.cancelled`, `effect.timeout`, `effect.provider_lost`, `effect.denied`, and the retry-related events, each attributed to the correct Effect ID and, where attempt-specific, Effect Attempt ID.

**One genuine, disclosed gap, found during this review and not silently assumed resolved.** This task's own required content for Audit Architecture names "timing" and "correlation identifiers" as required. Direct inspection of the current, tracked `AuditEvent` structure (`synapse-common`) shows exactly four fields: `event_type`, `actor`, `capability`, `message` — **no timestamp field, and no explicit correlation/trace identifier field of its own** (correlation is instead recoverable only indirectly, by cross-referencing the named `message`'s own `MessageId` against Effect Coordinator bookkeeping, ARCH-008 §15). ARCH-008 §17 itself requires truthful *ordering* of events, which the current shape can satisfy through emission sequence alone — but a consumer wanting a genuine wall-clock timestamp, or a directly-carried correlation identifier on the event itself, cannot obtain either from `AuditEvent` today. **This is not resolved here.** Concrete audit-event field additions are, consistent with ARCH-008 §4/§30's own repeated pattern, an implementation-phase concern for a future, separately authorized Engineering Work Order — this document only names the gap precisely, rather than leaving it to be silently discovered later or mistaken for already-closed.

## 13. Result Model

Governed generically by ARCH-008 §13 (Effect results "return through ordinary, authorized message delivery") and §24 (Effect Classification, reserved, no concrete type fixed). **Generalizing EWO-017's own demonstrated pattern, non-binding on any future Provider:** a Provider's typed result is encoded into the reply `Message.payload` using a private, provider-owned wire format (EWO-017's own hand-written length-prefixed encoding is one such realization, not a standard this document fixes); the requesting actor decodes it using the identical, provider-published decode function. Metadata (for example, an HTTP status code) travels inside that same typed result, never as a separate architectural concept. **Streaming and partial results are explicitly not designed here** — ARCH-008 §24 already reserves `Streaming` as an independent, combinable Effect-classification dimension whose "execution semantics are deferred," and this document does not narrow that deferral. Future extensibility: a Provider requiring a materially different result shape (a stream of partial results, for instance) is free to choose its own wire representation, provided it still returns through ordinary message delivery (§5) and the requester alone interprets it — no architectural gate is added or implied here for a specific result shape.

## 14. Concurrency Model

Governed by ARCH-008 §13 ("Provider execution MUST NOT prevent the Runtime from continuing to make bounded forward progress"), §31 invariant 16, and §33 (the concrete mechanism explicitly, deliberately undecided). **This document organizes the design space EWO-017's own disclosed choice sits within; it decides nothing ARCH-008 itself left open:**

- **Blocking-synchronous** (EWO-017's own realization): a Provider performs its entire external operation within one `handle()` call, bounded by its own resource-safety timeout. Simple, fully consistent with every other actor's existing synchronous execution model, and — disclosed plainly, exactly as `synapse-http-provider`'s own `README.md` already discloses — does not, by itself, satisfy ARCH-008 §13's forward-progress constraint for an operation whose latency is large relative to other actors' own scheduling needs, since the single-threaded Runtime makes no progress on unrelated work for the duration of that one blocking call.
- **Cooperative, non-blocking, multi-step polling**: already structurally supported by the *existing*, unmodified Effect Coordinator — `AttemptStatus::Executing` is a distinct, separately reachable state (ARCH-008 §16.2), and `record_executing` already exists as a Coordinator method independent of this document. A Provider using non-blocking I/O could perform one bounded quantum of work per `handle()` invocation and rely on a subsequent, externally-driven `step()` to continue — no new Runtime mechanism is required to support this pattern; EWO-017 simply does not exercise it.
- **A future thread-based or async-runtime mechanism**: remains exactly as undecided as ARCH-008 §33 already leaves it. This document does not select, require, or prohibit one.

Whichever a future Provider chooses, it MUST still satisfy ARCH-008 §13's own forward-progress constraint and MUST NOT introduce a second dispatch or admission path to achieve it (ARCH-008 §31 invariant 18). Thread safety and shared-resource questions are, likewise, unchanged: a Provider Actor is an ordinary actor instance, and Runtime's own existing "at most one concurrent `handle()` call per instance" guarantee (ARCH-002 §12) already applies without modification.

## 15. Provider Registration and Discovery

**Genuinely new content — not governed by any existing document.** ARCH-008 §11 states plainly, "no new provider registry is introduced"; §14 states a provider discovery mechanism "is a plausible future extension... but is not designed, assumed, or required by this document"; §30 lists "provider discovery" among its own explicit non-goals.

**Current, demonstrated model (EWO-017), stated normatively for the first time:** a Provider is looked up by `ActorId` through the existing Actor Directory (ARCH-002 §6), identically to any other actor. "Registration" of a Provider, today, consists of exactly two ordinary, pre-existing Runtime operations — `define_actor` and `create_actor_instance_with_behavior` — performed by whatever embedding code composes the Runtime; there is no provider-specific registration API, manifest, or discovery protocol, and this document introduces none. A capability naming an `effect.<domain>.<operation>` operation remains bound to the specific `ActorId` the capability was issued against (ARCH-008 §14); discovering *which* `ActorId` currently serves a given operation is the embedding code's own responsibility, unassisted by any Runtime mechanism.

**Versioning and compatibility:** a Provider Actor carries no version identifier of its own in the current architecture. Replacing a Provider Actor implementation is, architecturally, replacing the actor's own behavior under an unchanged `ActorId` (compatible with existing capabilities, no reissuance required) or introducing a new `ActorId` (requiring capability reissuance, attenuation, or rebinding, exactly as ARCH-008 §14 already discloses). This document does not introduce a provider-version field, a compatibility-negotiation protocol, or a manifest format.

**Future extensibility, disclosed as a genuine gap rather than assumed solved:** a future logical-provider-identity or discovery mechanism that decouples "authority to use an operation" from "the specific actor currently serving it" remains exactly as undesigned as ARCH-008 §14 already states. This document does not close that gap — it names the current, working, ActorId-direct model precisely enough that a future extension effort has a stated baseline to extend from, rather than an implicit one.

## 16. Provider Classification (by Nature)

**Genuinely new content, distinct from ARCH-008 §24's Effect Classification.** ARCH-008 §24 classifies an *Effect* — the operation being performed (`Pure`/`Read`/`Write`/`Transactional`/`Streaming`/`LongRunning`) — and explicitly reserves it as non-enforced, future metadata. This section classifies the *Provider itself* — an orthogonal axis ARCH-008 does not address. A single Effect Classification trait may correspond to Providers of any of the classes below, and vice versa; the two taxonomies MUST NOT be merged (ARCH-008 §24's own "classification is provider-declared metadata" language describes an Effect's own declared nature, not a statement about provider taxonomy at all — this section does not amend, narrow, or reinterpret §24 in any way).

| Class | Description | Architectural implication |
|---|---|---|
| **Stateless** | Retains no state across dispatches (EWO-017's own `HttpProvider` — a unit struct, `Copy`, holding nothing between calls) | Trivially safe to have multiple concurrent instances; no special disposal concern |
| **Stateful** | Retains state across dispatches (a connection-pooling database provider, for example) | Subject to the identical, unmodified actor domain-state and persistence-opt-in rules (ARCH-007 §7) as any other stateful actor; no new persistence mechanism is introduced for this reason |
| **Streaming** | Produces or consumes an ongoing sequence rather than one terminal response | Corresponds to, but does not itself define, ARCH-008 §24's `Streaming` Effect-classification dimension; result-shape mechanics remain deferred there (§13 above) |
| **Long-running** | An operation whose own execution substantially outlives ordinary message handling | Corresponds to ARCH-008 §24's `LongRunning` dimension; concurrency mechanics remain as organized, not decided, in §14 above |
| **Local** | The external resource is on the same host as the Runtime process (filesystem, local database) | No distinct architectural treatment beyond §14's own concurrency organization; typically lower-latency, but this document fixes no numeric assumption |
| **Remote** | The external resource is reached over a network (EWO-017's own HTTP provider) | Subject to the identical capability, admission, audit, retry, timeout, and cancellation rules as any other Provider (ARCH-008 §§13–21) — "remote" changes nothing architecturally; ARCH-008 §29 already states remote providers are compatible without redesign |
| **Persistent** | The Provider Actor instance itself is expected to remain live across many dispatches (a connection-holding provider) | Subject to ordinary Supervisor restart/stop/escalation rules (ARCH-004) unchanged; a restart of a Persistent provider still produces `ProviderLost` for any attempt in flight at the time (ARCH-008 §16, §21) |
| **Ephemeral** | A fresh instance may be created per dispatch or per short-lived batch of dispatches | No distinct architectural treatment; ordinary actor creation/termination (ARCH-002) already accommodates this without a new mechanism |

These eight names are independent, freely combinable descriptive tags, not a mutually exclusive enum — the identical "independent dimensions, not one closed axis" discipline ARCH-008 §24 already establishes for Effect Classification, applied here to Provider Classification for consistency. **None of the eight grants, withholds, or modifies capability authority, retry eligibility, timeout behavior, or audit requirements** — exactly as ARCH-008 §24 already states for its own classification model; this section introduces no authorization mechanism of any kind. `HttpProvider` (EWO-017) is classified here, for illustration, as **Stateless, Remote, Ephemeral** — consistent with its own `README.md` disclosure ("Stateless and deterministic except for the one external TCP round trip").

## 17. Provider Extension Rules

**Genuinely new content, codifying EWO-017's own demonstrated pattern as the first instance of it — formalizing demonstrated practice, GOV-013's own governing principle for standards-tier codification, applied here at the architecture tier for a Runtime-workspace engineering pattern.**

A future Provider Actor:

- **MUST** be implemented as an ordinary crate under `services/` (EWO-017's own `services/http-provider` precedent), implementing `synapse_api::Actor` with no dependency on `synapse-runtime` itself (avoiding a circular workspace dependency) and no dependency the rest of this workspace does not already carry, unless a new external dependency is itself raised as a separate, explicit architectural decision (ARCH-008 §33's own deferred-decision discipline; this document does not pre-authorize one).
- **MUST** publish a `README.md` stating: its responsibility and boundary (§4); its capability operations, in `effect.<domain>.<operation>` form (§6); its Provider Classification (§16); its concurrency realization and which category of §14 it falls into, disclosed explicitly rather than left implicit; and any known limitation.
- **MUST** use `EFFECT_PROVIDER_RESULT_FAILED` (or its successor, should one ever be introduced) for any ordinary, retry-eligible failure that leaves the Provider Actor instance itself healthy (§8) — never `Err` from `handle()` for such a case, which remains reserved for a genuine instance-level fault (ARCH-008 §18's "provider actor execution failure").
- **MUST** be covered by tests exercising, at minimum, the categories EWO-017 itself established as the demonstrated baseline: successful completion; an ordinary failure via the provider error model; timeout interaction; cancellation interaction (specifically, the late-signal-discard discipline against a genuine, working provider rather than a mechanical double); retry success and retry exhaustion; an invalid-capability rejection; a malformed-input rejection that fails safely rather than panicking; and a demonstration proving the full Actor -> Capability -> Effect -> Provider -> external resource -> typed result -> audit -> actor path end to end.
- **MUST NOT** introduce a second admission path, a second capability-authorization mechanism, a second timer subsystem, or a second supervision mechanism (ARCH-008 §31 invariants 18, 27–28, 32; unchanged, restated here only as an extension-rule checkpoint, not a new prohibition).
- **SHOULD** be authored, reviewed, approved, published, implemented, reviewed again, reported on, and accepted through the complete STD-031 Engineering Work Order lifecycle (STD-031 §6), exactly as EWO-017 was — this document does not require a specific EWO structure, since that remains STD-031's own domain, not this document's (§2).

Required engineering evidence for a new Provider, at minimum: a passing test suite meeting the bullet above; `cargo fmt`/`clippy -D warnings`/`cargo build`/`cargo test` all clean, exactly as STD-031 §11 already requires generally; and a truthful Engineering Report (STD-001 §47) disclosing what was implemented, what was deferred, and any known limitation — never silently omitted.

## 18. Architectural Invariants (Consolidated)

Every invariant ARCH-008 §31 already states (1–45) applies to every Provider Actor without exception or restatement here. This document adds exactly the following, new invariants, numbered independently to avoid renumbering ARCH-008's own list:

**A1.** A Provider Actor's own instance lifecycle (`ActorState`, ARCH-002 §15) and an Effect Attempt's own outcome (ARCH-008 §16.2) are distinct state machines and MUST NOT be merged, conflated, or represented as one "provider lifecycle" (§7).

**A2.** An ordinary, retry-eligible Provider failure that leaves the Provider Actor instance itself healthy MUST be reported via the reserved provider-error-model message type, never via `Err` from `handle()` (§8).

**A3.** Provider Classification (§16) and Effect Classification (ARCH-008 §24) are independent taxonomies describing different subjects (the provider; the operation) and MUST NOT be merged into one classification scheme.

**A4.** No Provider Classification tag (§16) grants, withholds, or modifies capability authority, retry eligibility, timeout behavior, or audit requirements.

**A5.** Provider registration (§15) remains `ActorId`-direct, through the existing Actor Directory, until and unless a future, separately authorized architecture effort introduces a discovery mechanism; no provider manifest, registry, or discovery protocol is introduced by this document.

**A6.** A future Provider's own concurrency realization (§14) MUST satisfy ARCH-008 §13's forward-progress constraint and MUST NOT introduce a second dispatch or admission path to do so; this document selects no specific mechanism.

## 19. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| This document is read as replacing, rather than complementing, ARCH-008 | §3's own explicit disclosure, stated first, before any substantive content |
| A future reader treats Provider Classification (§16) as an authorization mechanism | A4, and §16's own explicit "grants, withholds, or modifies... none of the eight" statement |
| A future reader conflates Provider Actor instance state with Effect Attempt outcome | §7's explicit disentanglement and A1 |
| A future Provider author invents a new failure-signaling convention instead of reusing EWO-017's | §17's explicit MUST, §8's generalization |
| Architecture Review concludes this document should not exist separately from ARCH-008 | Already disclosed as the expected, precedent-consistent outcome in §3 — not a risk this document treats as adverse |

## 20. Deferred Decisions

Explicitly deferred, not resolved here, consistent with ARCH-008's own identical pattern: a concrete provider-registration or discovery mechanism (§15); a concrete provider-version or compatibility-negotiation format (§15); a concrete streaming/partial-result wire representation (§13); a concrete thread-based or async concurrency mechanism (§14); any concrete audit-event timestamp or correlation-identifier field (§12); and every deferred decision ARCH-008 §33 already lists, unchanged.

## 21. Consequences

No existing Provider (EWO-017's `HttpProvider`) requires any change to remain conformant — every rule stated above either restates an existing ARCH-008 requirement it already satisfies, or is a new, prospective rule (§17's extension checklist) this document does not apply retroactively, on the identical basis STD-031 §14 already establishes for its own non-retroactive conformance principle. A future Provider gains a single, non-duplicated reference spanning this document and ARCH-008, rather than one document attempting, and risking inconsistency in, restating the other.

## 22. Approval Criteria

This document, and any Provider claiming conformance to it, is approved for reference use only if:

1. No content here conflicts with, restates in a diverging form, or silently amends any ARCH-008 §§9–31 requirement (§3).
2. Provider Classification (§16) grants no authority and is kept independent from Effect Classification (ARCH-008 §24).
3. The instance-lifecycle/attempt-outcome distinction (§7, A1) is preserved.
4. The provider error model generalization (§8, A2) is stated as the canonical, reusable pattern, not an HTTP-specific mechanism.
5. Architecture Review has explicitly considered, and recorded a disposition on, whether §§15–17 should instead be folded into ARCH-008 (§3's own named question).

## 23. Required Diagrams

### 23.1 Overall Provider Architecture

```text
Actor
    │
    ▼
Capability Validation      (ARCH-008 §14 — unchanged)
    │
    ▼
Effect Construction        (ARCH-008 §13, §15 — unchanged)
    │
    ▼
Effect Coordinator         (ARCH-008 §10 — unchanged)
    │
    ▼
Runtime Dispatch           ("Provider Dispatcher" is not a distinct
    │                        component — ARCH-008 §11 already
    │                        establishes Runtime's own existing,
    │                        unmodified dispatch as the mechanism;
    │                        no new component is introduced here)
    ▼
Provider                   (ARCH-008 §11; classified per §16 above)
    │
    ▼
External Resource          (outside this document's scope, §2)
    │
    ▼
Typed Result                (§13 — reply Message, provider-owned encoding)
    │
    ▼
Audit                      (ARCH-008 §17, one disclosed gap, §12)
    │
    ▼
Actor
```

### 23.2 Provider (Instance) Lifecycle — unchanged from ARCH-002 §15

```text
Defined -> Initializing -> Idle -> Ready -> Executing
Executing -> {Suspended | Failed | Stopping}
Stopping -> Terminated
```

### 23.3 Effect Attempt Lifecycle and Failure Transitions — unchanged from ARCH-008 §16.2

```text
Dispatched -> Executing -> {Completed | Failed}
Dispatched -> {TimedOut | Cancelled | ProviderLost}
Executing -> {TimedOut | Cancelled | ProviderLost}
```

### 23.4 Cancellation Flow — unchanged from ARCH-008 §21

```text
Cancellation trigger (explicit / requester termination / provider
lost / durable deletion / shutdown / capability revocation)
    │
    ▼
Effect Coordinator records Cancelled for the affected attempt
    │
    ▼
A later-arriving genuine provider completion for that same attempt
is discarded and truthfully audited — never applied (late-signal
discipline, ARCH-008 §16.2, §20, §21)
```

### 23.5 Retry Flow — unchanged from ARCH-008 §19

```text
Attempt reaches a retry-eligible terminal outcome
(Failed | TimedOut | ProviderLost)
    │
    ▼
Effect Coordinator's Retry Decision
(eligibility + actor intent + idempotency + capability limit)
    │
    ├── Accept  -> Effect's own accepted terminal outcome
    │
    └── Retry   -> Effect: RetryScheduled
                        │
                        ▼
                  Timer fires -> fresh capability validation
                        │
                        ▼
                  New Effect Attempt ID, same Effect ID
                        │
                        ▼
                  Dispatched (ordinary admission, §5)
```

### 23.6 Provider Interaction Sequence — unchanged from ARCH-008 §13, illustrated with EWO-017's own concrete evidence

```text
Requesting Actor          Runtime               Effect Coordinator      Provider Actor
      │                      │                          │                    │
      │  request_effect      │                          │                    │
      ├─────────────────────>│                          │                    │
      │                      │  capability validation    │                    │
      │                      │  (Capability Authority)   │                    │
      │                      │  record_requested/        │                    │
      │                      │  record_dispatched ───────>│                    │
      │                      │  admit_message ────────────────────────────────>│
      │                      │                          │                    │  handle()
      │                      │                          │                    │  (external
      │                      │                          │                    │   resource)
      │                      │<──────────────── reply message + outcome ──────┤
      │                      │  complete_effect /        │                    │
      │                      │  fail_effect ─────────────>│                    │
      │                      │  audit (effect.*)          │                    │
      │<───────── typed reply message ────────────────────                    │
```

## References

Internal:

- ARCH-008 — Effect Runtime Architecture (v0.4.3, Approved) — the primary, governing architecture this document complements throughout
- ARCH-002 — Runtime Architecture (`ActorState`, §15; Actor Directory, §6; Provider Architecture deferral, §23)
- ARCH-001 — Constitutional Architecture (actor isolation, capability model)
- ARCH-004 — Local Actor Supervision Architecture
- ARCH-007 — Persistent Actor Architecture (§7 persistence opt-in; §14/§17 operation-string granularity precedent)
- GOV-013 — Engineering Lifecycle (Architecture Authoring/Review stages, §6.8–§6.9)
- STD-031 — Engineering Work Order Lifecycle Standard (v0.2.1, Approved)
- ACT-003 — Act 3 Authorization and Charter (Approved)
- EWO-017 — Reference Effect Provider Framework (implemented, commit `397dded110bde75bdbcfcb4389c703d6fa7077dc`)

Source evidence (independently re-verified during this document's own preparation, not restated from memory):

- `architecture/ARCH-008-Effect-Runtime-Architecture.md`, read in full (§§1–38), including its own frontmatter's disclosed prior "ARCH-009 Architecture Investigation: Retry Architecture" precedent (§3 above)
- `synapse-runtime` @ `397dded110bde75bdbcfcb4389c703d6fa7077dc`: `common/src/lib.rs` (`AuditEvent`, `ActorState`, `EFFECT_PROVIDER_RESULT_FAILED`); `runtime/src/lib.rs` (`execute_message_capturing`'s dispatch-outcome wiring; `AttemptStatus`; `record_executing`); `services/http-provider/` in full (`README.md`, `src/lib.rs`, `src/internal.rs`)
- `governance/ACT-003-Act-3-Authorization-and-Charter.md`, read directly for its own named Act 3 scope

## Change History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-29 | Denver Jacobs (AI-assisted) | Initial Draft. Scoped narrowly to provider registration/discovery, provider classification (by nature, distinct from ARCH-008's own Effect Classification), and provider extension rules — the three concerns ARCH-008 §11/§14/§24/§30 explicitly reserves or does not address — after direct comparison against ARCH-008 §§9–31 found near-total overlap with the remainder of this task's own requested scope. Discloses a directly on-point precedent (ARCH-008's own frontmatter-recorded prior "ARCH-009 Architecture Investigation: Retry Architecture," which recommended an ARCH-008 amendment over a new document) and recommends Architecture Review consider the identical disposition for this document's own genuinely new content. Discloses one further gap in ARCH-008 §17's own audit model (no timestamp or correlation-identifier field on `AuditEvent`). No architecture is amended; no conflicting or competing definition is introduced. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-29 |
| Architecture Review | TBD | Pending — GOV-013 §6.9; see §22 item 5 above for the specific disposition question this review must record | |
| Approval Authority | TBD | Pending | |

This document is genuinely **Draft** — no Architecture Review and no Founder Approval act has occurred for it. It is drafted and published to the repository as a candidate architecture document, exactly as ARCH-002 through ARCH-007 remain candidate architecture pending their own review.
