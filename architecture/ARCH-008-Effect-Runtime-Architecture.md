---
document_id: ARCH-008
title: Effect Runtime Architecture
project: SynapseOS
specification: SynapseOS — the framework through which actors request externally observable or non-deterministic operations without directly owning external resources, realizing the "Provider Architecture" ARCH-002 §23 defers and the "effect-runtime consumer" ARCH-006 §11 already anticipates
version: 0.5.0
status: Approved
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available; GOV-003 §3.4, §3.5)
approval_authority: Denver Jacobs, Founder, exercising the interim Class B (architectural) approval default under GOV-003 §3.2 (Chief Architect role vacant)
created: 2026-07-27
last_updated: 2026-07-29
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
    - ARCH-001 (Draft — constitutional foundation; §6 foundational Runtime mechanisms list; §11 Runtime/Infrastructure-layer scoping)
    - ARCH-002 (Draft — Runtime architecture; §6, §9, §19, §22, §23 directly realized by this document)
    - ARCH-004 (Draft — Local Actor Supervision Architecture; §21 explicit Effect-system future-compatibility precedent)
    - ARCH-005 (Draft — Temporal Runtime Architecture; §4, §9, §21, §22 explicit Effect Runtime scope exclusion, component-placement, and future-compatibility precedent; timer-reuse precedent for timeout scheduling)
    - ARCH-006 (Draft — Runtime Actor Execution Architecture; §9.1, §11, §14 explicit anticipation of an effect-runtime consumer as a future message origin)
    - ARCH-007 (Draft — Persistent Actor Architecture; §8, §9, §17 precedent for ActorId-keyed replaceable-service state, execution-state exclusion, and deletion-coordination ordering)
    - ARCH-009 (Superseded by this version — Effect Provider Architecture, v0.1.0, Draft; its own §3 recommended, and Independent Architecture Review AR-009 confirmed, that its genuinely new content be folded into this document rather than retained as a standalone architecture; see §11.1–§11.4, §13.1–§13.2, §17.1, §18.1, invariants 46–51, and §38.4)
  rfcs: None
  adrs:
    - ADR-0015 (Approved — Audit Emitter Failure Semantics)
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
    - ADR-0017 (Approved — Bootstrap Capability Trust Root)
  roadmap: None
  research: None
  operational: None
  reports:
    - ER-011 (Effect Runtime Foundation — Engineering Report; published `synapse-docs` commit `57f67e11ce739d7ce7a2dd1bc37af356d96329a7`; verified predecessor baseline)
    - ER-012 (Provider Actor Integration — Engineering Report; published `synapse-docs` commit `bc75bab77c8827c9aa947637f20c617194e78c6e`)
    - ER-013 (Effect Timeout Integration — Engineering Report; published `synapse-docs` commit `cfd82e4db5ce90eb8ed800ec3f66f1300b173cc5`)
    - ER-014 (Effect Cancellation on Actor Termination — Engineering Report; published `synapse-docs` commit `4f981ecdec5b4f9e116acd2e6d99d713bf2fd7b7`)
    - ER-015 (Provider Idempotency Registration — Engineering Report; published `synapse-docs` commit `b1439cb3ac8fbc279e9ba13aeba18a986b2139da`; current verified implementation baseline this amendment's evidence is checked against)
  engineering:
    - EWO-017 (Reference Effect Provider Framework, implemented, Runtime commit `397dded110bde75bdbcfcb4389c703d6fa7077dc`) — the sole demonstrated Provider implementation this amendment generalizes from; no ER-017/ER-018 exists for it in this repository as of this amendment (disclosed gap, not concealed — see §38.4)
  source_artifacts:
    - synapse-runtime @ 397dded110bde75bdbcfcb4389c703d6fa7077dc (published; adds `synapse-http-provider` and the additive `EFFECT_PROVIDER_RESULT_FAILED` dispatch-outcome wiring on top of the 4256b44 baseline; confirms `record_retry_scheduled` and the `RetryScheduled` state remain bookkeeping only, unchanged by this amendment)
  review: "SynapseOS — Effect Runtime Architecture Review (primary analytical basis for the initial 0.1.0 draft, concluding EFFECT RUNTIME ARCHITECTURE APPROVED FOR FORMAL DESIGN); SynapseOS — Independent Review of ARCH-008 Effect Runtime Architecture (concluding ARCH-008 EFFECT RUNTIME ARCHITECTURE REQUIRES CORRECTION; governing basis for the 0.2.0 correction); SynapseOS — Final Independent Review of Corrected ARCH-008 Effect Runtime Architecture (concluding ARCH-008 EFFECT RUNTIME ARCHITECTURE REQUIRES FURTHER CORRECTION; governing basis for this 0.3.0 correction, §37); SynapseOS Act II — Idempotency Architecture Investigation (concluding IDEMPOTENCY ARCHITECTURE INVESTIGATION COMPLETE — READY FOR ARCHITECTURE AMENDMENT; sole analytical basis for the 0.4.0 amendment, §23.1–§23.5); SynapseOS Act II — Independent Architecture Review of the ARCH-008 Idempotency Amendment (concluding ARCH-008 IDEMPOTENCY REVIEW REQUIRES AMENDMENT; two MAJOR findings, one MINOR finding; governing basis for the 0.4.1 revision); SynapseOS Act II — Independent Architecture Re-Review of the ARCH-008 Idempotency Amendment (concluding ARCH-008 IDEMPOTENCY RE-REVIEW COMPLETE — READY FOR PUBLICATION; confirmed all three findings resolved with no regression — the governing basis for the 0.4.2 approval); SynapseOS — ARCH-009 Architecture Investigation: Retry Architecture (concluding ARCH-009 ARCHITECTURE INVESTIGATION COMPLETE — READY FOR ARCHITECTURE AMENDMENT, recommending an ARCH-008 amendment rather than a new document; sole analytical basis for this 0.4.3 amendment, §19.1–§19.4 and invariant 45); SynapseOS — Independent Architecture Review of the ARCH-008 v0.4.3 Retry Architecture Completion (concluding ARCH-008 v0.4.3 INDEPENDENT ARCHITECTURE REVIEW REQUIRES CORRECTION; two MAJOR findings, two MINOR findings; governing basis for the pre-publication corrections folded into this same 0.4.3 entry, §37); SynapseOS — Independent Architecture Re-Review of the ARCH-008 v0.4.3 Retry Architecture Completion (concluding ARCH-008 v0.4.3 INDEPENDENT ARCHITECTURE RE-REVIEW COMPLETE — READY FOR FOUNDER APPROVAL; confirmed all four findings resolved with no regression); SynapseOS — Founder Approval, ARCH-008 v0.4.3 (concluding FOUNDER APPROVAL GRANTED — ARCH-008 v0.4.3 APPROVED FOR PUBLICATION — the governing basis for this publication, §38.3); SynapseOS — ARCH-009 Effect Provider Architecture (Draft, v0.1.0, authored following EWO-017; concluding its own content should be integrated into ARCH-008 rather than retained as a standalone document, §3); SynapseOS — Independent Architecture Review of ARCH-009 Effect Provider Architecture (AR-009; concluding ARCH-009 ARCHITECTURE REVIEW COMPLETE — READY FOR FOUNDER APPROVAL, zero Critical, zero Major findings, two Minor findings, recommending Option B — merge into ARCH-008 — over Option A; sole analytical basis for this 0.5.0 amendment); SynapseOS — Founder Approval, ARCH-008 v0.5.0 (concluding ARCH-008 UPDATED — PROVIDER ARCHITECTURE INTEGRATED — the governing basis for this publication, §38.4)"
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-008 — Effect Runtime Architecture

*Filename pattern: `ARCH-008-Effect-Runtime-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Document Control and Status

| Field | Value |
|---|---|
| Document ID | ARCH-008 |
| Title | Effect Runtime Architecture |
| Version | 0.5.0 |
| Status | **Approved** (content through 0.3.2, §38.1; content added by 0.4.0/0.4.1, approved as 0.4.2, §38.2; content added by 0.4.3 — §19.1–§19.4, invariant 45 — approved as 0.4.3, §38.3; content added by 0.5.0 — §11.1–§11.4, §13.1–§13.2, §17.1, §18.1, invariants 46–51 — approved as recorded below, §38.4) |
| Author | Denver Jacobs |
| Approval authority | Denver Jacobs, Founder, exercising the interim Class B (architectural) approval default under GOV-003 §3.2 (Chief Architect role vacant) |
| Created | 2026-07-27 |
| Approved | 2026-07-27 (content through 0.3.2); 2026-07-28 (content added by 0.4.0/0.4.1, approved as 0.4.2; content added by 0.4.3, approved and published that same date); 2026-07-29 (content added by 0.5.0, approved and published this same date) |
| Classification | Public |

This document is **Approved** in full: content through version 0.3.2 (§38.1), content added by 0.4.0/0.4.1 (§23.1–§23.5, invariants 42–44, approved as 0.4.2, §38.2), content added by 0.4.3 (§19.1–§19.4, invariant 45, §38.3), and content added by 0.5.0 (§11.1–§11.4, §13.1–§13.2, §17.1, §18.1, invariants 46–51, §38.4) — each disposition recorded separately, per STD-001 §31.5's content-non-mutating model, by Denver Jacobs, Founder, acting as the interim Approval Authority for Class B (architectural) decisions under GOV-003 §3.2 while the Chief Architect role remains vacant, with the disclosed self-review/self-approval basis stated explicitly in §38 per GOV-003 §3.5. The 0.5.0 disposition (§38.4) follows the complete governance chain: Architecture Authoring of ARCH-009 (a separate, sibling document, GOV-013 §6.8) → Independent Architecture Review of ARCH-009 (AR-009, `COMPLETE — READY FOR FOUNDER APPROVAL`, zero Critical, zero Major, two Minor findings, recommending integration into ARCH-008 over standalone status) → Founder acceptance of that recommendation → Architecture Amendment (this version, correcting both Minor findings in the process) → Founder Approval (`GRANTED`). Unlike 0.4.3, this cycle's own Independent Architecture Review evaluated content while it still lived in a separate, already-published sibling document (ARCH-009 v0.1.0) rather than a pre-publication draft of this document itself — disclosed here because it is a genuinely different governance shape from every prior amendment, not because it is irregular (§37, 0.5.0 entry, states the full reasoning). The existing 0.3.2, 0.4.2, and 0.4.3 Approval Status table entries and Approval Evidence Records (§38.1–§38.3) remain completely unmodified below, as the truthful, permanent record of what was approved and when. This document introduces no implementation and authorizes none directly; it establishes architecture only, to be realized by a future Engineering Work Order (STD-001 §46) — EWO-001/ER-011 through EWO-016/ER-017, and EWO-017 (implemented, no ER published for it — a disclosed gap, §38.4), already realize the portion of it published to date (§36 References).

This document is the authoritative source ARCH-002 §23 already anticipated and named: ARCH-002 §23's Deferred Architecture table lists "Provider Architecture: Adapter patterns, retry and circuit-breaking policy" against the contract "Provider adapters as ordinary, capability-scoped actors, never ambient." This document is that Provider Architecture document, generalized to the complete framework governing every externally observable or non-deterministic operation an actor may request — not merely a narrow adapter-pattern note. It is also the document ARCH-005 §21 and ARCH-006 §11 each independently, textually anticipated as a future, compatible extension requiring no redesign of either document's own architecture.

This document codifies, and introduces no design beyond, the Effect Runtime Architecture Review, which concluded `EFFECT RUNTIME ARCHITECTURE APPROVED FOR FORMAL DESIGN` after finding no BLOCKER findings, no required redesign of any published architecture, and strong, convergent, four-document evidentiary precedent for the ownership model this document formalizes. Every architectural decision recorded below traces to a specific review finding or a specific, cited piece of existing, published architecture; none is newly invented here.

Version 0.3.0 applied the two accepted corrections identified by a final, fully independent review of the 0.2.0 correction, which concluded `ARCH-008 EFFECT RUNTIME ARCHITECTURE REQUIRES FURTHER CORRECTION` on the basis of two MAJOR findings — both self-contained internal inconsistencies the 0.2.0 correction introduced while establishing the Effect ID / Effect Attempt ID split, neither reopening any of the six MAJOR findings the 0.2.0 correction had already, genuinely resolved (§37, Change History). The prior version (0.2.0) applied the accepted corrections identified by an earlier, independent adversarial review of the initial 0.1.0 draft, which concluded `ARCH-008 EFFECT RUNTIME ARCHITECTURE REQUIRES CORRECTION` on the basis of six MAJOR findings, none requiring redesign of this document's own approved foundation. Every correction applied across both versions is additive: it clarifies, discloses, or extends an existing section; none reopens an ownership decision the 0.1.0 draft established.

Version 0.3.1 applied no technical or architectural change of any kind relative to 0.3.0 — every ownership decision, lifecycle rule, and invariant described throughout this document remained exactly as 0.3.0 established. 0.3.1 was a governance-preparation and metadata-correction pass only, prepared ahead of formal review by this project's Approval Authority (§37, §38).

Version 0.3.2 likewise applied no technical or architectural change of any kind relative to 0.3.0/0.3.1 — it recorded only the formal governance disposition itself (§37, §38.1): the Approval Authority's decision, the completed Approval Status table, and the Approval Evidence Record. Versions 0.4.0/0.4.1 completed the Idempotency deferral §23 itself already named (§23.1–§23.5, invariants 42–44); 0.4.2 recorded that completion's own governance disposition (§38.2).

Version 0.4.3 completed the Retry deferral §19 itself already named, on the sole analytical basis of the ARCH-009 Architecture Investigation: Retry Architecture, which concluded `ARCH-009 ARCHITECTURE INVESTIGATION COMPLETE — READY FOR ARCHITECTURE AMENDMENT` and recommended this exact vehicle — an ARCH-008 amendment, not a new document — precisely to preserve ARCH-008 as the single constitutional owner of the entire Effect lifecycle. **Strictly additive (STD-001 §13): no existing ownership decision, lifecycle rule, Trusted Core boundary, capability model, idempotency rule, or prior invariant (1–44) is reopened, redefined, or reinterpreted.** It adds four new subsections to §19 — §19.1 Retry Eligibility, §19.2 Retry Authority, §19.3 Idempotency-Class Retry Permission, §19.4 Retry Limit Ownership — and one new invariant (45, deterministic retry decisions). A pre-publication Independent Architecture Review of this content (§37) found two MAJOR and two MINOR findings, corrected before this version was ever committed; one correction clarified §9's own ownership-table wording for "domain-level retry intent" to state directly the general case §19.2 depends on (restoration remains named there as one specific case of it) — the requesting actor's ownership of retry intent itself is unchanged; only the completeness of its description in §9 was corrected. A subsequent Independent Architecture Re-Review confirmed all four findings resolved with no regression (`ARCH-008 v0.4.3 INDEPENDENT ARCHITECTURE RE-REVIEW COMPLETE — READY FOR FOUNDER APPROVAL`), and Founder Approval was then granted (`FOUNDER APPROVAL GRANTED — ARCH-008 v0.4.3 APPROVED FOR PUBLICATION`). Unlike the 0.4.0→0.4.2 idempotency cycle, which recorded its governance disposition under a separate version (0.4.2), the complete review-correction-re-review-approval cycle for Retry is recorded entirely under this same version, 0.4.3, since no intermediate state was ever committed or published — `status` transitions directly from `Draft` to **`Approved`** for this version, per GOV-013 §11's requirement that the amendment receive its own evidenced Approval Authority act, satisfied here by §38.3, below. The existing 0.3.2 and 0.4.2 Approval Status table entries and Approval Evidence Records (§38.1, §38.2) are left completely unmodified as the truthful, permanent record of what was approved and when.

The current version, 0.5.0, integrates the genuinely new architectural content of ARCH-009 — Effect Provider Architecture (v0.1.0, Draft), a sibling document authored after EWO-017 (Reference Effect Provider Framework, the first concrete Effect Provider implemented against this architecture). ARCH-009's own §3 found, by direct comparison, that ARCH-008 §§9–31 already governed nearly everything it had been asked to define, and disclosed a directly on-point precedent: this document's own frontmatter already records a prior "ARCH-009 Architecture Investigation: Retry Architecture" that reached the identical conclusion for a narrower scope (§37, 0.4.3 entry) — amend ARCH-008, do not duplicate it in a new document. An Independent Architecture Review (AR-009) of the standalone ARCH-009 document confirmed this: zero Critical and zero Major findings against its content, two Minor findings (a citation misattribution and a structural-completeness gap against GOV-013 §6.8's own named ARCH-document elements), and a recommendation for "Option B" — integrate ARCH-009's genuinely new content into ARCH-008 rather than retain it as a standalone, separately-versioned document — over "Option A," retaining it independently. The Founder accepted AR-009's conclusion and Option B in full. This version accordingly adds: §11.1 (disentangling the Provider Actor's own instance lifecycle, `ActorState`, from the Effect Attempt's own outcome — a clarification, not a new lifecycle state); §11.2 (Provider Registration and Discovery, the `ActorId`-direct model ARCH-008 previously stated only negatively as "no registry is introduced," now stated normatively); §11.3 (Provider Classification by nature — Stateless/Stateful/Streaming/Long-running/Local/Remote/Persistent/Ephemeral — an axis orthogonal to, and independent of, §24's existing Effect Classification); §11.4 (Provider Extension Rules, codifying EWO-017's own demonstrated pattern as the process for adding a future Provider); §13.1 (Result Model, generalizing EWO-017's typed-request/typed-result pattern, non-binding); §13.2 (Concurrency Model, organizing — never deciding — the design space §13's own forward-progress constraint and §33's own deferral already leave open); §17.1 (a disclosed gap: the current `AuditEvent` structure carries no timestamp or correlation-identifier field of its own); §18.1 (the Provider error model — `EFFECT_PROVIDER_RESULT_FAILED` — generalized as the canonical pattern for a Provider to report an ordinary, retry-eligible failure without being misclassified as `ProviderLost`); and invariants 46–51 (§31). **Strictly additive (STD-001 §13): no existing ownership decision, lifecycle rule, Trusted Core boundary, capability model, retry/idempotency rule, or prior invariant (1–45) is reopened, redefined, or reinterpreted.** Both Minor findings AR-009 identified against the standalone ARCH-009 document are corrected in the course of this integration (§37, 0.5.0 entry, states exactly how) rather than carried forward. ARCH-009 itself is marked `Superseded` by this version (its own frontmatter, updated) and is retained in the repository, unmodified in substance, as the truthful historical record of how this content was originally authored and reviewed (STD-001 §4/§28's own immutability principle) — it is not deleted, and its own analytical content is not restated verbatim here where a citation back to it suffices.

## 2. Purpose

This document defines the authoritative architecture for the **Effect Runtime**: the framework through which an actor requests an externally observable or non-deterministic operation — an HTTP call, a filesystem read or write, a SQL query, an AI-inference request, an email send, a message-queue publish, a cloud-API call, a shell execution, a hardware interaction, or a future plugin-provided operation — without directly owning the external resource that operation touches. It defines who may request an Effect, who authorizes the request, who executes it, how its outcome is reported, how it is audited, how it fails, how it may be retried, timed out, or cancelled, how it interacts with Persistent Actor determinism, and what remains explicitly deferred.

This document does not select, implement, or design any concrete provider (no HTTP client, filesystem API, SQL driver, AI-model adapter, email client, cloud SDK, or shell-execution mechanism); does not define implementation APIs; and does not authorize implementation. It is architecture: what must be true, who owns what, and why — consistent with ARCH-002's own stated method (`ARCH-002 §1`: "precise enough that independent engineering teams could implement recognizably conformant runtimes without identical internal code").

## 3. Scope

**In scope:** the architectural placement of Effect requesting, tracking, authorization, execution, and outcome-reporting within the existing Runtime component model; the ownership boundary between Runtime (sole orchestrator), the Effect Coordinator (bookkeeping only), and Provider Actors (execution only); the capability-authorization model governing Effect requests; the message-admission model governing Effect requests and outcomes; the Effect lifecycle and its terminal states; the mandatory prohibition on direct Provider-to-Provider invocation; the reserved (not yet implemented) Effect Classification model; the audit obligations Effects introduce; the failure, retry, timeout, and cancellation architectures; the interaction between in-flight Effects and Persistent Actor determinism; idempotency as provider-declared metadata; the compensation boundary; resource-governance ownership; the security boundary around provider credentials; and future compatibility with Workflow Runtime, Distributed Runtime, the Runtime Control API, and the SynapseOS Control Centre.

**Out of scope:** any concrete provider implementation (HTTP client, filesystem API, SQL driver, email client, AI-model adapter, cloud SDK, shell-execution mechanism, broker client); provider discovery or a plugin marketplace; distributed Effect transport; streaming-protocol mechanics; long-running-operation polling mechanics; distributed transactions; compensation execution; numeric retry or timeout defaults or policy; Runtime Control API endpoints; Control Centre user interfaces; any Rust struct, trait, enum, method signature, or field layout. See §30 for the complete Explicit Non-Goals statement.

## 4. Non-Goals

This document does not define, and takes no position on:

- an HTTP client, filesystem API, SQL driver, email client, AI-model adapter, cloud SDK, shell-execution mechanism, or message-broker client of any kind;
- a plugin marketplace or generalized plugin-loading mechanism;
- provider discovery beyond the existing, unmodified Actor Directory contract (ARCH-002 §6);
- distributed Effect transport, cross-host provider routing, or clustering;
- streaming-protocol mechanics or long-running-operation polling mechanics;
- distributed transactions of any kind;
- compensation execution (compensation *ownership* is addressed, §25; compensation *mechanics* are not);
- numeric retry policy, backoff, jitter, or maximum-attempt defaults;
- numeric timeout-duration defaults;
- Runtime Control API endpoint design;
- Control Centre user-interface design;
- any Rust struct, trait, enum, method signature, function name, or field layout beyond what is necessary to state ownership;
- any new Trusted Core component (ARCH-002 §5–§6 is unchanged; see §9);
- any new actor lifecycle state beyond ARCH-002 §15's existing set;
- any new constitutional guarantee beyond ARCH-001's four (non-forgery, integrity, enforcement-at-invocation, revocation-state enforcement).

## 5. Architectural Context

This document amends no prior authority. It extends, and is bound by, the following without redefinition:

- **ARCH-000** established SynapseOS's whole-system introduction; this document inherits it without restatement.
- **ARCH-001** fixes the four constitutional guarantees and confines Runtime mechanics to the Runtime/Infrastructure layer (`ARCH-001 §11`). Its §6 "foundational Runtime mechanisms" list — capability enforcement, audit emission, scheduling, time observation, transport, bootstrap — does not include, and gives no basis for adding, "Effect execution": this is the direct textual basis for §9's ownership recommendation that Effect execution belongs in Actor World, never in Trusted Core.
- **ARCH-002** remains the sole authority for Trusted Core component responsibility and the fixed Replaceable-services list (`ARCH-002 §6`), fresh-never-cached capability validation (`ARCH-002 §9`), the Extension and Replaceability Model (`ARCH-002 §19`), the minimum audit-event set (`ARCH-002 §18`), and the Deferred Architecture table (`ARCH-002 §23`), whose "Provider Architecture" row — "Provider adapters as ordinary, capability-scoped actors, never ambient" — this document completes directly (§1 above). This document amends none of it.
- **ARCH-004** established the precedent this document follows for a new replaceable service (narrow responsibility, Runtime-mediated reach only) and independently, explicitly named the exact model this document formalizes: "this session's own 'actor emissions are admission requests, not sent facts' discipline... is the model a future effect system should follow" (`ARCH-004 §21`).
- **ARCH-005** established the `ActorId`-keyed replaceable-service-state precedent this document reuses for Effect Coordinator state, established the timer-reuse pattern this document reuses for timeout scheduling (§20), and explicitly confirmed compatibility in advance: "an effect-scheduling actor is an ordinary consumer of ordinary timer-fired messages" (`ARCH-005 §21`), while explicitly excluding "a workflow engine or generalized effect-scheduling system" from its own scope (`ARCH-005 §4`, §22).
- **ARCH-006** established the single, shared, Runtime-owned admission pipeline (`admit_message`) this document reuses without modification (§13), and is the single most directly on-point precedent this document depends on: its §11 states, verbatim, that "any future milestone introducing a new message origin (a durable-timer replay, a workflow-orchestration actor's own emission, **an effect-runtime consumer**) inherits this same convergence requirement without needing to redesign it," and its §14 explicitly excludes "a generalized effect-scheduling abstraction" from its own scope for a precise, evidenced reason this document now resolves.
- **ARCH-007** established the `ActorId`-keyed persistent-state model this document extends by exclusion (§22): its §9's exhaustive rule that execution state "MUST remain opaque and non-persisted" is the direct basis for this document's own rule that in-flight Effects are execution state, not domain state. Its §17 deletion-coordination-ordering pattern (validate authority → cancel dependent state → audit → attempt the definitive act → audit outcome) is the direct precedent this document reuses for Effect cancellation on durable deletion (§21).
- **ADR-0015** (Audit Emitter Failure Semantics) governs every audit-emission failure behavior this document assumes: a mandatory audit emission that fails causes the *reporting* operation to fail, without rollback of already-committed component-level state.
- **ADR-0016** (Trusted Core Interaction Rule) is the specific authority this document depends on most directly. Its two rules — Runtime as sole cross-component composer (Rule 1) and no Trusted-Core-adjacent component owning a direct peer path (Rule 2) — are extended, not amended, by this document to a new participant (Effect Coordinator, §10): it connects to every other component exactly as Supervisor, Temporal Runtime, and Persistence Service already do, through the Runtime, never directly.
- **ADR-0017** (Bootstrap Capability Trust Root) governs how any Effect capability ultimately comes to exist: through the ordinary, ADR-0017-rooted `issue_capability` path, never a new issuance mechanism.
- **Governance (GOV-003, GOV-010)** and **STD-001** govern this document's own review, approval, and evidentiary process; they are not architectural inputs to its content.

## 6. Governing Architectural Precedents

Four already-published, already-drafted architecture documents converge, independently, on the same answer this document formalizes. This convergence — not invention — is the primary basis for every ownership decision below.

| Document | Citation | Convergent finding |
|---|---|---|
| ARCH-002 | §23 | "Provider adapters as ordinary, capability-scoped actors, never ambient." |
| ARCH-004 | §21 | "This session's own 'actor emissions are admission requests, not sent facts' discipline... is the model a future effect system should follow." |
| ARCH-005 | §21, §4, §22 | "An effect-scheduling actor is an ordinary consumer of ordinary timer-fired messages," while explicitly excluding "a workflow engine or generalized effect-scheduling system" from its own scope. |
| ARCH-006 | §11, §14 | The shared admission pipeline "inherits this same convergence requirement without needing to redesign it" for "an effect-runtime consumer" as a future message origin; explicitly excludes "a generalized effect-scheduling abstraction" from its own scope. |

Per STD-001's own evidence-precedence discipline, this is not a design question with multiple live options addressed here for the first time — it is a design question with an already-published answer this document's own job is to formalize, not reopen. No finding in the governing Effect Runtime Architecture Review required redesigning any of the four documents above; each remains fully intact and unamended by this document.

## 7. Definitions

- **Effect** — a request, made by an actor, for an externally observable or non-deterministic operation to be performed on its behalf: one whose outcome depends on a resource outside the Runtime's own deterministic control (a network, a filesystem, a database, an AI-inference service, an email transport, a message broker, a cloud API, a shell, a hardware device, or a future plugin).
- **Effect request** — the specific, capability-authorized ask that one such operation be performed, identified by an Effect ID (§15).
- **Effect Coordinator** — the new, narrow, non-Trusted-Core, Runtime-composed replaceable service this document introduces (§10), owning Effect-request bookkeeping, lifecycle tracking, and timeout/retry/cancellation coordination.
- **Provider Actor** — an ordinary, capability-scoped actor that executes a provider-specific operation on behalf of an Effect request (§11). "Effect Provider" and "Provider" are used interchangeably with "Provider Actor" throughout this document and elsewhere in this corpus; all three denote the identical concept — an ordinary actor, never a Trusted Core or Runtime-privileged construct.
- **Requesting actor** — the ordinary, capability-scoped actor that initiates an Effect request; may itself be a Provider Actor requesting a further Effect (subject to §12's isolation rule).
- **Effect ID** — the Runtime-scoped, opaque identifier for one logical Effect request, stable across every attempt made to satisfy it (§15).
- **Effect Attempt ID** — the Runtime-scoped, opaque identifier for one concrete execution attempt of an Effect. A retry (§19) mints a new Effect Attempt ID while remaining associated with the same, unchanged Effect ID; an Effect that is never retried has exactly one attempt. Every provider result, timeout, cancellation signal, retry trigger, and late completion is attributable to a specific Effect Attempt ID, never to the Effect ID alone (§15).
- **Effect Classification** — the reserved (not yet implemented) taxonomy of orthogonal, independently combinable metadata dimensions — `Pure`, `Read`, `Write`, `Transactional`, `Streaming`, `LongRunning` — none of which is mutually exclusive with any other (§24).
- **Idempotency metadata** — provider-declared classification of an operation as `Idempotent`, `NonIdempotent`, or `Unknown` (§23).
- **Terminal outcome** — for an Effect Attempt, one of `Completed`, `Failed`, `Cancelled`, `TimedOut`, `ProviderLost` (§16.2), reached at most once per attempt; for the Effect itself, either `Denied` (no attempt was ever created, §16.1) or the single *accepted* logical terminal outcome reached once no further retry follows an attempt's own terminal outcome (§16.3, §19).
- **RetryScheduled** — an Effect-level, transitional state (§16.1) describing the Effect's own condition after one attempt has reached its own terminal outcome and before a new attempt's own authorization succeeds; never an Effect Attempt state, and never a reopening of the attempt that preceded it.

## 8. Architectural Principles

The following principles govern every decision in this document, each a direct application of existing authority:

- **Runtime orchestrates; it never executes.** Every cross-component Effect sequence is Runtime-composed (ADR-0016 Rule 1); Runtime performs no provider-specific I/O itself (ARCH-001 §6).
- **Providers are ordinary actors, never a privileged category.** No Trusted Core boundary is crossed by introducing a Provider Actor (ARCH-002 §23).
- **Actor emissions are admission requests, never sent facts — extended to Effects.** An Effect request and an Effect result are ordinary messages, converging through the single shared admission pipeline (ARCH-006 §11), exactly as ARCH-004 §21 anticipated.
- **Mechanism and meaning remain separated.** The Effect Coordinator tracks that a request exists and where it stands; it never decides whether the request is authorized, and never performs the operation itself (§10).
- **Authorization is never inferred from mechanism, possession, or classification.** Capability Authority alone authorizes; classification metadata is descriptive, never authorizing (§24).
- **Truthfulness over convenience**, extending ADR-0015's own precedent: no audit record may claim an Effect completed before it genuinely did, and no terminal outcome may be silently overwritten by a later, stale signal (§16).
- **Determinism is preserved across restart.** In-flight Effect execution is Runtime execution state, never actor domain state (§22), extending ARCH-007 §9's exhaustive exclusion.
- **No redesign of existing architecture.** Every mechanism this document reuses — the admission pipeline, the capability model, the audit-event shape, the timer service — is reused unmodified; this document introduces exactly one new component (§10) and no new Trusted Core concept.

## 9. Ownership Model

Every responsibility this architecture introduces has exactly one owner. No responsibility below may be duplicated, shared, or silently assumed by a component other than the one named.

| Responsibility | Owner |
|---|---|
| Deciding when an Effect request begins; obtaining or verifying authorization; admitting Effect request and outcome messages through the ordinary pipeline; coordinating timeout and cancellation actions; informing the Effect Coordinator of a Provider Actor lifecycle-loss event; emitting mandatory audit events | **Runtime** (§9.1), sole orchestrator (ADR-0016 Rule 1) |
| Effect and Effect Attempt identity; request bookkeeping; lifecycle-state tracking per Effect and per attempt; timeout coordination; cancellation coordination; Provider Actor lifecycle-loss reaction; retry-decision coordination; terminal-outcome tracking; queryable Effect/attempt status | **Effect Coordinator** (§10) |
| Authorizing every capability-gated Effect operation; fresh, never-cached validation | **Capability Authority**, exactly as ARCH-002 §9 already establishes generally |
| Executing the provider-specific operation; owning provider-specific external-resource integration and credentials | **Provider Actor** (§11) |
| Requesting an Effect; supplying operation input; consuming the Effect outcome; deciding domain-level retry intent — including, as one specific case, whether to request an Effect again after restoration (§22) | **Requesting actor** (which may itself be a Provider Actor requesting a further Effect, subject to §12) |

This table is the single authoritative statement of ownership for this architecture. No component gains a responsibility not listed against it here.

### 9.1 Runtime Responsibilities

Runtime MUST remain the sole cross-component composer of every Effect-related sequence this document introduces, on the identical basis ADR-0016 Rule 1 already establishes for every other Trusted-Core-adjacent sequence in the system.

Runtime:

- MUST mediate every Effect request and every Effect outcome through the existing, unmodified message-admission pipeline (§13);
- MUST obtain or verify the required authorization from Capability Authority before an Effect request is dispatched to a Provider Actor;
- MUST coordinate, through the Effect Coordinator, timeout and cancellation actions, without itself becoming a second Effect-tracking authority;
- MUST inform the Effect Coordinator when a Provider Actor instance handling an active attempt is restarted, stopped, terminated, or otherwise made unavailable by ordinary supervision, so the affected attempt can be driven to a truthful terminal outcome (§18, §21) — the concrete mechanism is an implementation decision (§33), but the responsibility to do so is Runtime's own, not optional;
- MUST cause every mandatory Effect-related audit event this document requires (§17) to be truthfully emitted, in the order the underlying facts genuinely occurred;
- MUST NOT itself perform provider-specific external operations;
- MUST NOT itself decide authorization — that remains Capability Authority's own responsibility.

Runtime gains no new decision authority of its own by virtue of this document; it gains exactly one new orchestration responsibility, on the same basis it already orchestrates supervision (ARCH-004), temporal delivery (ARCH-005), and persistence (ARCH-007).

## 10. Effect Coordinator

**Effect Coordinator** is a new, narrow, Runtime-composed replaceable service, positioned architecturally parallel to Scheduler, Supervisor, Temporal Runtime, and Persistence Service (`ARCH-002 §6`'s "Replaceable services" table; ARCH-004 §9.1, ARCH-005 §9, ARCH-007 §12's identical precedent, each independently). It is not a Trusted Core component and introduces no new constitutional concept.

The Effect Coordinator owns exactly, and only:

- **Effect and Effect Attempt identity** (§15) — minting and tracking the opaque Effect ID for each logical request and the opaque Effect Attempt ID for each concrete execution attempt of it;
- **request bookkeeping** — the requesting `ActorId`, the presented capability reference, the provider destination, and correlation information where supplied, per Effect and per attempt;
- **lifecycle tracking** (§16) — the current state of each Effect and each of its attempts, and the ordering discipline preventing more than one accepted terminal outcome per attempt and per Effect;
- **timeout coordination** (§20) — scheduling and enforcing timeouts, per attempt, by reusing the existing Timer Service and Temporal Runtime, never a new timer subsystem of its own;
- **cancellation coordination** (§21) — ensuring a cancelled attempt cannot later be reported as completed;
- **Provider Actor lifecycle-loss awareness** (§21) — recording, when Runtime informs it that the Provider Actor instance handling an active attempt has been restarted, stopped, terminated, or otherwise made unavailable by ordinary supervision, that the affected attempt can no longer reach a genuine completion signal, and driving that attempt to a truthful terminal coordination outcome — never itself observing or deciding the underlying supervision outcome, which remains exclusively Supervisor's and Runtime's own (§18, §21);
- **retry-decision coordination** (§19) — deciding, from actor-supplied intent, provider-declared idempotency, and capability constraints, whether a new attempt may be made, never silently retrying a `NonIdempotent` or `Unknown` Effect, and never silently re-executing an attempt lost to Provider Actor restart without a new, explicitly authorized and scheduled attempt (§19, §21);
- **terminal-outcome tracking and queryable Effect/attempt status** — a durable-for-the-runtime-session record of each Effect's and each attempt's current and final state, exposable to a future Runtime Control API (§29) without exposing Provider Actors directly.

The Effect Coordinator MUST NOT:

- become a second Runtime — it decides no authorization and orchestrates no cross-component sequence Runtime itself has not already initiated;
- authorize capabilities itself — Capability Authority remains the sole authorization authority (§14);
- perform external operations of any kind — all provider-specific I/O remains Provider Actors' own responsibility (§11);
- directly dispatch actors outside the Runtime — dispatch remains Runtime/Execution Coordinator's own mechanism, unmodified (§13);
- own provider credentials — credentials remain exclusively Provider Actors' own responsibility (§27);
- implement its own timer system — timeout and retry scheduling reuse the existing Timer Service and Temporal Runtime without exception (§19, §20);
- implement a second admission pipeline — Effect requests and outcomes converge through the single, existing `admit_message` pipeline (§13);
- silently retry a `NonIdempotent` or `Unknown` Effect (§19, §23);
- persist or resurrect in-flight Effect execution — in-flight Effects are Runtime execution state, never checkpointed, never automatically replayed (§22);
- supervise Provider Actors — Supervisor and Runtime's own existing lifecycle machinery remain the sole, unmodified authority over restart, stop, and escalation decisions for any actor, including a Provider Actor; the Effect Coordinator only ever *reacts*, after the fact, to a lifecycle outcome Runtime already reached through that unmodified machinery, never participates in reaching it (§21).

The Effect Coordinator is reachable only through Runtime, and MUST NOT establish or use any direct interaction path to any other component (ADR-0016 Rule 2, extended to this new participant exactly as ARCH-004, ARCH-005, and ARCH-007 already extend it to Supervisor, Temporal Runtime, and Persistence Service respectively).

## 11. Provider Actor Model

A **Provider Actor** is an ordinary, capability-scoped actor (`api::Actor`, unmodified) that executes a provider-specific operation on behalf of an Effect request. It is not a Trusted Core component, not a Runtime-privileged construct, and introduces no new actor category — this is ARCH-002 §23's own already-published answer ("Provider adapters as ordinary, capability-scoped actors, never ambient"), applied here without modification.

Examples of future provider domains include HTTP, filesystem, database, AI-inference, email, message-broker, cloud, hardware, and plugin providers — none designed by this document (§4, §30).

Provider Actors:

- execute provider-specific operations and own provider-specific external-resource integration, entirely outside this document's architectural concern beyond the boundary this section fixes;
- receive Effect requests, and return Effect outcomes, through ordinary Runtime admission (§13) — never through a synchronous call, an unrestricted actor reference, or any path bypassing admission;
- remain subject to the ordinary actor lifecycle, capability, audit, scheduling, and supervision rules every other actor is already subject to — no new lifecycle state, no new supervision exception, no new audit exemption;
- own their own provider-specific credentials exclusively (§27) — never shared with, or visible to, the requesting actor, the Effect Coordinator, or Runtime;
- are looked up by `ActorId` through the existing Actor Directory (ARCH-002 §6), exactly as any other actor — no new provider registry is introduced (§30);
- are replaceable and mockable exactly as any other actor already is, since a Provider Actor is simply a differently-behaving actor under the same capability-authorized operation strings (§14) — no new architectural machinery is required to make this true, subject to the capability target-binding consequence stated in §14;
- remain subject to ordinary Supervisor restart/stop/escalation decisions exactly as any other actor, with no special exemption and no second supervision mechanism (§18, §21) — a Provider Actor's own execution failure is handled by the identical, unmodified machinery any other actor's execution failure already is;
- MUST NOT block the Runtime's own forward progress for the duration of a genuine external operation (§13) — this is a constraint on Provider Actor implementations, not a property this document can grant automatically.

**One Provider Actor may internally perform more than one external step only when those steps together constitute a single, declared logical provider operation** (for example, an operation that internally authenticates before performing its one externally meaningful action). Distinct, independently invocable external operations remain distinct Effects and MUST route through the Runtime exactly as any other Effect request would (§12) — packaging multiple, independently invocable operations inside one actor or one internal library does not create permission to bypass the Provider Isolation Rule. This document does not mandate exactly one Provider Actor per operation; it mandates that no internally-reachable, independently-qualifying Effect ever bypasses admission, regardless of how many operations one Provider Actor implements.

Provider-specific implementation — the HTTP client, the SQL driver, the filesystem API, and so on — remains entirely outside this document's scope (§4).

### 11.1 Provider Instance Lifecycle vs. Effect Attempt Lifecycle (0.5.0)

A Provider Actor participates in exactly two, distinct, already-existing state machines, which MUST NOT be merged, conflated, or represented as one "provider lifecycle":

1. **The Provider Actor's own instance lifecycle** — governed by the existing, unmodified `ActorState` (ARCH-002 §15): `Defined -> Initializing -> Idle -> Ready -> Executing -> {Suspended | Failed | Stopping -> Terminated}`. This is the *actor instance's* own state — whether it exists, is live, and can currently execute — identical for a Provider Actor and for any other actor. Nothing about being a Provider changes this state machine.
2. **The Effect Attempt's own outcome** — governed by §16.2: `Dispatched -> Executing -> {Completed | Failed | Cancelled | TimedOut | ProviderLost}`. This is the *attempt's* own terminal outcome — one specific dispatched operation's own fate — tracked by the Effect Coordinator, never by the Provider Actor's own instance state.

A Provider Actor instance remains `Executing` (state 1) for the duration of exactly one `handle()` call; an Effect Attempt it is currently handling separately, independently reaches its own terminal outcome (state 2), which says nothing about whether the *instance* itself remains `Idle`/`Ready` afterward (ordinarily yes) or has been separately lost (`ProviderLost` — precisely the case where instance state 1 and attempt state 2 diverge: the instance left `Executing` via restart/termination, and *that same fact* is what attempt state 2 records as `ProviderLost`, §18, §21). A completed or failed Effect Attempt does not itself dispose of the Provider Actor instance that handled it. No new lifecycle state is introduced by this subsection; it is a clarification of an existing conflation risk, extending the attempt/Effect distinction §31 invariants 9–10 already establish by one further level.

### 11.2 Provider Registration and Discovery (0.5.0)

A Provider is looked up by `ActorId` through the existing Actor Directory (ARCH-002 §6), identically to any other actor. "Registration" of a Provider consists of exactly two ordinary, pre-existing Runtime operations — defining an actor and creating a live instance with behavior — performed by whatever embedding code composes the Runtime; there is no provider-specific registration API, manifest, or discovery protocol, and this document introduces none. A capability naming an `effect.<domain>.<operation>` operation remains bound to the specific `ActorId` the capability was issued against (§14); discovering *which* `ActorId` currently serves a given operation is the embedding code's own responsibility, unassisted by any Runtime mechanism.

A Provider Actor carries no version identifier of its own. Replacing a Provider Actor implementation is, architecturally, replacing the actor's own behavior under an unchanged `ActorId` (compatible with existing capabilities, no reissuance required) or introducing a new `ActorId` (requiring capability reissuance, attenuation, or rebinding, §14). This subsection introduces no provider-version field, no compatibility-negotiation protocol, and no manifest format.

A future logical-provider-identity or discovery mechanism that decouples "authority to use an operation" from "the specific actor currently serving it" remains exactly as undesigned as §14 already states (§33). This subsection does not close that gap — it states the current, working, `ActorId`-direct model normatively for the first time, so a future extension effort has a stated baseline to extend from.

### 11.3 Provider Classification (0.5.0)

**Distinct from, and never to be merged with, Effect Classification (§24).** §24 classifies an *Effect* — the operation being performed (`Pure`/`Read`/`Write`/`Transactional`/`Streaming`/`LongRunning`) — and explicitly reserves it as non-enforced, future metadata. This subsection classifies the *Provider itself* — an orthogonal axis. A single Effect Classification trait may correspond to Providers of any of the classes below, and vice versa.

| Class | Description | Architectural implication |
|---|---|---|
| **Stateless** | Retains no state across dispatches | Trivially safe to have multiple concurrent instances; no special disposal concern |
| **Stateful** | Retains state across dispatches (a connection-pooling database provider, for example) | Subject to the identical, unmodified actor domain-state and persistence-opt-in rules (ARCH-007 §7) as any other stateful actor; no new persistence mechanism is introduced for this reason |
| **Streaming** | Produces or consumes an ongoing sequence rather than one terminal response | Corresponds to, but does not itself define, §24's `Streaming` Effect-classification dimension; result-shape mechanics remain deferred there (§13.1) |
| **Long-running** | An operation whose own execution substantially outlives ordinary message handling | Corresponds to §24's `LongRunning` dimension; concurrency mechanics remain as organized, not decided, in §13.2 |
| **Local** | The external resource is on the same host as the Runtime process (filesystem, local database) | No distinct architectural treatment beyond §13.2's own concurrency organization; typically lower-latency, but this document fixes no numeric assumption |
| **Remote** | The external resource is reached over a network | Subject to the identical capability, admission, audit, retry, timeout, and cancellation rules as any other Provider — "remote" changes nothing architecturally; §29 already states remote providers are compatible without redesign |
| **Persistent** | The Provider Actor instance itself is expected to remain live across many dispatches (a connection-holding provider) | Subject to ordinary Supervisor restart/stop/escalation rules (ARCH-004) unchanged; a restart of a Persistent provider still produces `ProviderLost` for any attempt in flight at the time (§16, §21) |
| **Ephemeral** | A fresh instance may be created per dispatch or per short-lived batch of dispatches | No distinct architectural treatment; ordinary actor creation/termination (ARCH-002) already accommodates this without a new mechanism |

These eight names are independent, freely combinable descriptive tags, not a mutually exclusive enum — the identical "independent dimensions, not one closed axis" discipline §24 already establishes for Effect Classification, applied here for consistency. **None of the eight grants, withholds, or modifies capability authority, retry eligibility, timeout behavior, or audit requirements.**

### 11.4 Provider Extension Rules (0.5.0)

Codifying EWO-017's own demonstrated pattern — the first concrete Provider implemented against this architecture — as the process for adding a future Provider Actor:

- **MUST** be implemented as an ordinary crate under `services/`, implementing `synapse_api::Actor`, with no dependency on `synapse-runtime` itself and no dependency this workspace does not already carry, unless a new external dependency is itself raised as a separate, explicit architectural decision (§33's own deferred-decision discipline; this subsection does not pre-authorize one).
- **MUST** publish a `README.md` stating: its responsibility and boundary (§11); its capability operations, in `effect.<domain>.<operation>` form (§14); its Provider Classification (§11.3); its concurrency realization (§13.2), disclosed explicitly rather than left implicit; and any known limitation.
- **MUST** use the reserved provider-error-model message type (§18.1) for any ordinary, retry-eligible failure that leaves the Provider Actor instance itself healthy — never `Err` from `handle()` for such a case, which remains reserved for a genuine instance-level fault (§18).
- **MUST** be covered by tests exercising, at minimum: successful completion; an ordinary failure via the provider error model; timeout interaction; cancellation interaction; retry success and retry exhaustion; an invalid-capability rejection; a malformed-input rejection that fails safely rather than panicking; and an end-to-end demonstration of the complete path from actor through Provider to external resource and back.
- **MUST NOT** introduce a second admission path, a second capability-authorization mechanism, a second timer subsystem, or a second supervision mechanism (§31 invariants 18, 27–28, 32; unchanged, restated here only as an extension-rule checkpoint).
- **SHOULD** be authored, reviewed, approved, published, implemented, reviewed again, reported on, and accepted through the complete STD-031 Engineering Work Order lifecycle, exactly as EWO-017 was — this document does not require a specific EWO structure, since that remains STD-031's own domain.

Required engineering evidence for a new Provider, at minimum: a passing test suite meeting the bullet above; formatting, lint, build, and test validation all clean, exactly as STD-031 §11 already requires generally; and a truthful Engineering Report (STD-001 §47) disclosing what was implemented, what was deferred, and any known limitation.

## 12. Mandatory Provider Isolation Rule

> **Provider Actors MUST NOT invoke other Effect Providers directly.**

A Provider Actor requiring another Effect MUST submit a new Effect request through the Runtime exactly as an ordinary requesting actor would.

**The required path:**

```text
Provider Actor
    │
    │ submits new Effect request
    ▼
Runtime
    │
    ├── capability authorization
    ├── Effect Coordinator tracking
    ├── audit
    └── ordinary message admission
    ▼
Target Provider Actor
```

**The prohibited path:**

```text
Provider Actor
    │
    │ direct provider call
    ▼
Another Provider Actor
```

This prohibition covers, without exception, direct invocation through: unrestricted actor references; provider handles; provider registries; direct library callbacks; shared service objects; hidden internal queues; sockets or channels bypassing Runtime admission; or provider-specific orchestration shortcuts.

**Rationale.** Direct Provider-to-Provider invocation would create a hidden execution path bypassing capability authorization, audit emission, Effect identity, timeout enforcement, cancellation, retry controls, quotas, correlation, supervision boundaries, future distributed routing, and Runtime Control API observability — precisely the failure mode ARCH-006 §11 already names as constitutional: "if a second, origin-specific admission path existed... any origin routed around it would silently escape every guarantee this architecture claims to hold universally." This rule is the direct, necessary application of that same constitutional property to the one place a future implementer might otherwise be tempted to shortcut it — two providers believed to be trusted, colocated, or commonly maintained.

**This rule applies even when** both providers are trusted, built into the same crate, run in the same process, or are maintained by the same developer. Trust, colocation, and common maintenance are not architectural properties this document recognizes as grounds for bypassing admission — the identical reasoning ARCH-002 §9 already applies to capability validity ("fresh, never cached, never assumed") applies here to admission itself.

**Permitted:** internal, deterministic helper-library calls inside one Provider Actor's own implementation, provided they do not constitute invocation of another Effect Provider or another externally observable Effect. A provider computing a checksum, formatting a request body, or parsing a response using an ordinary in-process library is not "invoking another provider" — it has performed no externally observable operation on another provider's behalf. The dividing line is externally observable effect, not code organization: if the called code itself would independently qualify as an Effect under this document's own definition (§7), routing to it directly is prohibited regardless of how it is packaged — including where the two operations are packaged inside one Provider Actor or one shared internal library (§11's composite-provider clarification).

## 13. Effect Request and Result Flow

Effect requests and Effect outcomes remain ordinary messages. They converge through the existing, unmodified, Runtime-owned admission pipeline (`admit_message`, ARCH-006 §11) exactly as external submissions, actor-emitted messages, and timer-generated messages already do — a fourth message origin, requiring no redesign of the pipeline itself, exactly as ARCH-006 §11 explicitly anticipated.

This document prohibits:

- a second, Effect-specific queue that bypasses admission;
- direct provider dispatch by the requesting actor;
- synchronous provider invocation of any kind;
- unrestricted provider references held by any actor;
- ambient provider access granted merely by running inside the Runtime process;
- any provider-specific admission path.

Both initial requests and retried requests MUST undergo fresh capability validation at the moment of dispatch — never cached from an earlier point, never inherited from a prior attempt (ARCH-002 §9; ARCH-005 §14's identical fire-time-only precedent, applied here). Effect results return through ordinary, authorized message delivery, dispatched to the requesting actor exactly as any other message would be.

Execution itself follows the ordinary dispatch model ARCH-006 §10 already establishes: a Provider Actor's `handle()` is invoked by Execution Coordinator, under Lifecycle Guardian's existing `Executing`-state discipline, one message at a time (ARCH-002 §12), exactly as any other actor's execution already is. No new Runtime-owned execution primitive, synchronous or asynchronous, is introduced.

**Runtime forward progress.** This document does not claim, and must not be read as claiming, that genuine external I/O latency is automatically invisible to the Runtime merely because Effects are modeled as ordinary messages. What this modeling *does* achieve — and the full extent of what it achieves — is that external latency and provider-specific mechanics never become an actor's own domain-state concern (§22): the requesting actor observes only an ordinary, eventual reply message, never a socket, a thread, or a blocking call of its own. Whether that reply arrives promptly is a separate, genuine implementation property this document does impose a constraint on:

> **Provider execution MUST NOT prevent the Runtime from continuing to make bounded forward progress for unrelated actors, timers, Effect coordination, and lifecycle processing.**

This is a constitutional constraint on any conformant implementation, not a mandate for any particular concurrency technology. It does not select, require, or prohibit threads, an async runtime, worker processes, or any other specific mechanism (§4, §33) — the concrete mechanism by which a Provider Actor's own `handle()` initiates a genuinely long-latency operation without itself blocking unrelated Runtime progress is an implementation decision for a future Engineering Work Order. What is fixed here is the property that implementation must achieve: a Provider Actor performing slow external I/O MUST NOT be capable of stalling delivery, scheduling, timeout enforcement, or supervision for any actor other than itself.

### 13.1 Result Model (0.5.0)

Effect results return through ordinary, authorized message delivery (above); this subsection generalizes EWO-017's own demonstrated pattern for a Provider's own typed result, non-binding on any future Provider: a Provider's typed result is encoded into the reply `Message.payload` using a private, provider-owned wire format; the requesting actor decodes it using the identical, provider-published decode function. Metadata (for example, an HTTP status code) travels inside that same typed result, never as a separate architectural concept. Streaming and partial results are explicitly not designed here — §24 already reserves `Streaming` as an independent, combinable Effect-classification dimension whose execution semantics are deferred, and this subsection does not narrow that deferral. A Provider requiring a materially different result shape is free to choose its own wire representation, provided it still returns through ordinary message delivery and the requester alone interprets it — no architectural gate is added here for a specific result shape.

### 13.2 Concurrency Model — Organizing the Design Space (0.5.0)

This subsection organizes, without deciding, the design space the forward-progress constraint above and §33's own deferral already leave open:

- **Blocking-synchronous** (EWO-017's own realization): a Provider performs its entire external operation within one `handle()` call, bounded by its own resource-safety timeout. Simple, fully consistent with every other actor's existing synchronous execution model, but does not, by itself, satisfy the forward-progress constraint above for an operation whose latency is large relative to other actors' own scheduling needs, since the single-threaded Runtime makes no progress on unrelated work for the duration of that one blocking call.
- **Cooperative, non-blocking, multi-step polling**: already structurally supported by the existing, unmodified Effect Coordinator — `Executing` is a distinct, separately reachable attempt-level state (§16.2), and a coordinator-level method already exists to record it independent of this subsection. A Provider using non-blocking I/O could perform one bounded quantum of work per `handle()` invocation and rely on a subsequent, externally-driven Runtime step to continue — no new Runtime mechanism is required to support this pattern.
- **A future thread-based or async-runtime mechanism**: remains exactly as undecided as §33 already leaves it. This subsection does not select, require, or prohibit one.

Whichever a future Provider chooses, it MUST still satisfy the forward-progress constraint above and MUST NOT introduce a second dispatch or admission path to achieve it (§31 invariant 18). Thread safety and shared-resource questions are unchanged: a Provider Actor is an ordinary actor instance, subject to Runtime's own existing "at most one concurrent `handle()` call per instance" guarantee (ARCH-002 §12).

## 14. Capability Architecture

Effect capability operation strings follow the structure:

```text
effect.<domain>.<operation>
```

Examples (illustrative only, not a closed list): `effect.http.get`, `effect.http.post`, `effect.filesystem.read`, `effect.filesystem.write`, `effect.sql.query`, `effect.sql.execute`, `effect.ai.infer`, `effect.email.send`. New domains and operations may be added without any architectural change — this is the direct, deliberate extension of ARCH-007 §14's/§17's already-established `persistence.checkpoint`/`persistence.restore`/`persistence.delete` operation-string granularity.

This architecture establishes:

- **Operation-specific least privilege** — each `effect.<domain>.<operation>` string is a distinct, separately-grantable capability. Possessing one MUST NOT imply possessing another: `effect.filesystem.read` MUST NOT imply `effect.filesystem.write` or `effect.filesystem.delete`. Dangerous operations remain independently grantable, exactly as ARCH-007 §14/§20/§30 already requires "materially stronger, separately auditable authorization" for deletion relative to ordinary checkpointing.
- **Ordinary Capability Authority issuance** — Effect capabilities are minted through the existing `issue_capability` path (ADR-0017), no new issuance mechanism.
- **Ordinary binding** — bound to the requesting actor exactly as any other capability already is (ARCH-002 §9).
- **Attenuation-only delegation** — a Provider Actor or a downstream requester may only ever narrow, never widen, an Effect capability it delegates (ARCH-001 §5.2/§6).
- **Revocation** — Effect capabilities are revocable exactly as any other capability (ARCH-002 §9).
- **Fresh validation at execution time, never cached** — every Effect dispatch, including a retry (§19), revalidates the presented capability at the moment of dispatch.
- **No provider-owned authorization decisions** — a Provider Actor never decides whether a request reaching it was authorized; that determination has already been made, exclusively, by Capability Authority before admission (§13).
- **No possession of one operation implying another** — restated for emphasis: this is the load-bearing least-privilege property this entire capability structure exists to provide.

**Target binding, disclosed explicitly.** The existing, unmodified Capability model binds every capability to a specific target (ARCH-001 §5.2). This document introduces no exception: an `effect.<domain>.<operation>` capability that authorizes delivery to a Provider Actor is bound to that specific Provider Actor's `ActorId`, exactly as any other capability targeting any other actor already is. A concrete consequence follows directly, and is stated here rather than left for an implementer to discover:

- holding `effect.http.get` does **not** grant ambient authority to invoke *any* actor that happens to implement an HTTP-get operation — authority remains bound to the specific, authorized destination the capability names, under the existing admission model (§13);
- replacing a Provider Actor with a different `ActorId` MAY require the affected capabilities to be reissued, attenuated, or rebound to the new target — this document does not assume such a replacement is transparent;
- distributing load across multiple Provider Actor identities implementing the same operation cannot be assumed to work transparently under the current, single-target capability model;
- a future logical provider identity, provider discovery mechanism, or capability-rebinding facility that decouples "authority to use an operation" from "the specific actor currently serving it" is a plausible future extension, but is not designed, assumed, or required by this document (§33) — no such mechanism exists in the current architecture, and this document introduces none;
- no provider registry of any kind is introduced by this document (§11, §30).

## 15. Effect Identity

Effect identity has exactly two levels, and they MUST NOT be conflated.

**Effect ID** — a Runtime-scoped, opaque identifier for the logical Effect an actor requested. It is minted once and remains stable across every attempt made to satisfy it, including every retry (§19). It retains or associates:

- the requesting `ActorId` — never an ephemeral `ActorInstanceId`, for the identical durability reason ARCH-002 §7, ARCH-004 §12, ARCH-005 §11, and ARCH-007 §8 each independently already establish for their own respective identity models;
- the capability or operation reference originally presented (§14);
- a Runtime-established causation reference, established the same way ARCH-006 §10/§13 already establishes causation for any other emitted message — never self-asserted by the requester;
- correlation information where supplied by the requesting actor, carried as ordinary `Message` payload data (ARCH-002 §8, unmodified) — no new constitutional concept;
- the current logical lifecycle state (§16);
- the accepted terminal outcome, once one exists (§16, §19) — at most one, regardless of how many attempts were made.

**Effect Attempt ID** — a Runtime-scoped, opaque identifier for one concrete execution attempt of an Effect, minted by the Effect Coordinator only once authorization for that specific try has genuinely succeeded (§16.2) — the same atomic fact the existing admission pipeline (§13) already establishes as one event, whether described as "authorized" or "dispatched." A request whose authorization fails receives no Effect Attempt ID at all: that outcome (`Denied`) belongs exclusively to the Effect's own lifecycle, never to an attempt's (§16.1). Each retry mints a fresh Effect Attempt ID, once its own fresh authorization succeeds, while remaining associated with the same, unchanged Effect ID (§16.1, §19). It retains or associates:

- the Effect ID it belongs to;
- the provider destination for this specific attempt (the target Provider Actor's `ActorId`, §14) — an attempt retried after a Provider Actor replacement may target a different `ActorId` than an earlier attempt of the same Effect;
- the dispatch time and, where truthfully observable, the execution-start time for this attempt (§16, §17);
- the timeout applicable to this attempt (§20);
- this attempt's own terminal outcome, once one exists (§16) — at most one per attempt;
- audit linkage specific to this attempt (§17).

Every provider result, timeout, cancellation signal, retry trigger, and late completion MUST be attributable to a specific Effect Attempt ID, never to the Effect ID alone. This is required so that a late result belonging to an earlier attempt cannot overwrite the accepted outcome of a later attempt of the same Effect (§16, §19, §20), so duplicate completions of the same attempt can be rejected, and so a Provider Actor's own restart or loss can be resolved against the specific attempt it affected without disturbing any other attempt or any other Effect (§21).

Neither identifier's concrete format (counter, UUID, or otherwise) is fixed here, and neither is required to be persisted across a Runtime-process restart beyond what §22/§28 already require of Effect Coordinator bookkeeping generally. A distributed identifier representation is explicitly not designed here, consistent with this corpus's own long-standing deferral of distributed identity representation generally (ARCH-002 §21/§23). The two-level identity model above is extensible without replacement: it adds no field or property that would need to change shape to accommodate a future distributed Effect ID or Effect Attempt ID.

## 16. Effect Lifecycle

Effect lifecycle has exactly two levels, corresponding to the two-level identity model (§15), and they MUST NOT be conflated: the **Effect-level lifecycle** governs the logical Effect as a whole, across zero or more attempts; the **Attempt-level lifecycle** governs one Effect Attempt, which comes into existence only once authorization for that specific try has genuinely succeeded.

### 16.1 Effect-level lifecycle

```text
Requested -> Denied
Requested -> [an Effect Attempt is dispatched]
[an Effect Attempt reaches its own terminal outcome] -> RetryScheduled -> [a new Effect Attempt is dispatched]
[an Effect Attempt reaches its own terminal outcome] -> [Effect's own accepted logical terminal outcome]
```

| State | Kind |
|---|---|
| Requested | Transitional |
| Denied | **Terminal** — authorization for even a first attempt failed; no Effect Attempt was ever created and no Effect Attempt ID was ever minted for it (§15) |
| RetryScheduled | Transitional — the Effect Coordinator has decided (§19) that a new attempt will be made and has scheduled it via the existing Timer Service (§19, §20); no Effect Attempt exists for this specific retry cycle until its own authorization succeeds and it is dispatched |

`Denied` MUST NOT be treated as an Attempt-level outcome under any circumstance: an authorization failure — whether on the Effect's first try or on any subsequent retry's own fresh authorization check (§14, §19) — means dispatch never occurred for that try, so no Effect Attempt, and no Effect Attempt ID, ever came into existence for it. `RetryScheduled` MUST NOT be treated as an Attempt-level state: it describes the Effect's own condition while awaiting a new attempt, strictly after a prior attempt has already reached its own terminal outcome (§16.2) and strictly before a new Effect Attempt ID is minted for the next one. No attempt is reopened, extended, or reused to represent this waiting period.

### 16.2 Attempt-level lifecycle

```text
Authorized -> Dispatched -> Executing -> {Completed | Failed}
Authorized -> Dispatched -> {Completed | Failed}
Dispatched -> {TimedOut | Cancelled | ProviderLost}
Executing -> {TimedOut | Cancelled | ProviderLost}
```

| State | Kind |
|---|---|
| Authorized | Transitional — MAY be truthfully collapsed into `Dispatched` (see below); exists only where authorization for this specific try succeeded, which is precisely the condition under which an Effect Attempt (and its Effect Attempt ID) comes into existence |
| Dispatched | Transitional |
| Executing | Transitional (conditionally observed — see §17); MAY be truthfully collapsed into `Dispatched` where the underlying implementation cannot distinguish dispatch from execution start, in which case `Dispatched` transitions directly to `Completed`/`Failed`/`TimedOut`/`Cancelled`/`ProviderLost` |
| Completed | **Terminal** |
| Failed | **Terminal** for this attempt (a retry, if any, is a fresh attempt reached through the Effect-level `RetryScheduled` transition, §16.1, §19) |
| Cancelled | **Terminal** |
| TimedOut | **Terminal** |
| ProviderLost | **Terminal** — the Provider Actor instance handling this attempt was restarted, stopped, terminated, or otherwise made unavailable by ordinary Runtime supervision before a genuine completion signal could be observed (§18, §21) |

**Authorized/Dispatched collapse, permitted explicitly.** The existing, unmodified message-admission pipeline this document reuses (§13) performs capability validation and admission as a single, atomic operation with no truthfully observable intermediate state between "authorized" and "admitted." Where an implementation genuinely cannot distinguish the moment authorization succeeded from the moment dispatch began, `Authorized` and `Dispatched` MAY be represented, and audited, as one combined transition — exactly the same accommodation this section already grants `Executing` relative to `Dispatched`, applied here for the identical underlying reason. This collapse MUST NOT be achieved by introducing a second, separate authorization check ahead of the existing pipeline (§9, §10, §31 invariants 17–18) — it is a representational simplification of one genuinely atomic Runtime fact, never a new mechanism. This collapse never affects `Denied`: a failed authorization check has no `Dispatched` counterpart to collapse into, precisely because no attempt was ever created (§16.1).

**Terminal outcomes for one attempt:** `Completed`, `Failed`, `Cancelled`, `TimedOut`, `ProviderLost` — `Denied` is never among them (§16.1). **A given Effect Attempt reaches no more than one terminal outcome.** The Effect Coordinator MUST enforce this as an invariant of its own tracked state (§10, §31 invariant 9): once a terminal outcome is recorded for an attempt, no later signal attributed to that same attempt — a late provider completion, a stale timeout, a duplicate cancellation — may overwrite it (§17, §20, §21). A late signal arriving after its own attempt's terminal outcome is already recorded MUST be discarded and truthfully audited as discarded, never silently applied and never silently dropped without record. A signal attributed to an *earlier* attempt of the same Effect MUST NOT overwrite a *later* attempt's own outcome, and MUST NOT be mistaken for a signal belonging to the later attempt (§15, §19).

### 16.3 Composition

**The Effect's own logical lifecycle** is the composition of §16.1 and §16.2: `Requested` once, for the Effect itself; then either `Denied` (no attempt ever created) or a first Effect Attempt is dispatched (§16.2); upon that attempt reaching its own terminal outcome, the Effect either accepts it as its own final outcome, or transitions to `RetryScheduled` (§16.1, §19) and, once the scheduled retry's own fresh authorization succeeds, a new Effect Attempt begins under a new Effect Attempt ID (§15) — repeating until exactly one attempt reaches `Completed`, or until no further retry is authorized and the last attempt's own terminal outcome becomes the Effect's own accepted logical terminal outcome. **The Effect itself reaches no more than one accepted logical terminal outcome**, even though several of its attempts may each have reached their own, distinct attempt-level terminal outcome first (§19, §31 invariants 9–10).

This lifecycle is a named set of architecturally recognized transitions, not a numeric policy; the concrete mechanism realizing each transition (a specific enum, a specific state-machine implementation) remains an implementation-phase concern for the eventual Engineering Work Order, on the same basis ARCH-005 §12 and ARCH-007 §16 already leave their own lifecycle mechanics to implementation.

## 17. Audit Architecture

The following facts MUST be truthfully, distinctly auditable, using the existing, unmodified `AuditEvent` structure — new `event_type` string values only, never a new field — extending, never contradicting, the minimum audit-event set ARCH-002 §18 already establishes and the identical discipline ARCH-004, ARCH-005, and ARCH-007 have each already applied:

**Mandatory, attributed to the relevant Effect ID and, where the fact concerns one attempt specifically, the relevant Effect Attempt ID (§15):**

- `effect.requested` (Effect-level)
- `effect.dispatched` (attempt-level)
- `effect.completed` (attempt-level, and Effect-level once accepted as the Effect's own logical outcome, §16)
- `effect.failed` (attempt-level)
- `effect.cancelled` (attempt-level)
- `effect.timeout` (attempt-level)
- `effect.provider_lost` (attempt-level) — the truthful record that the Provider Actor instance handling this attempt was restarted, stopped, terminated, or otherwise made unavailable by ordinary supervision before completion could be observed (§18, §21)

**Mandatory, Effect-level:** `effect.denied` — attributed to the Effect ID only, never to an Effect Attempt ID, since a denied request never produces an attempt (§16.1). `effect.denied` is always genuinely distinct from any dispatch event, since a denied request is never dispatched at all, and remains unconditionally, separately mandatory — it is never subject to the collapse allowance below, which applies only to `Authorized`/`Dispatched` for an attempt that genuinely came into existence.

**Mandatory, but permitted to collapse (attempt-level):** `effect.authorized` is a mandatory, truthfully distinct fact wherever an implementation can genuinely observe authorization as separate from dispatch, for an attempt that did come into existence. Where the existing, reused admission pipeline (§13) makes authorization and dispatch one atomic, inseparable operation, `effect.authorized` MAY be represented by the same audit record as `effect.dispatched` (§16.2's collapse allowance) — this MUST NOT be achieved by adding a second, separate authorization check purely to manufacture a distinct event (§9, §10).

**Conditionally mandatory:** `effect.executing` — mandatory only if, and only where, the Runtime can truthfully observe that transition as a distinct fact (e.g., the moment Execution Coordinator's own dispatch genuinely begins, ARCH-006 §10); it MUST NOT be fabricated or inferred if the underlying implementation cannot truthfully distinguish "dispatched" from "executing" as separate observable moments (ADR-0015's truthfulness discipline, applied here). Where an implementation cannot truthfully distinguish them, `effect.dispatched` alone satisfies this document's own audit requirement.

**Retry-related, required when retry support is implemented, concrete identifiers deferred:** a fact equivalent to a retry being scheduled MUST be truthfully, distinctly auditable once retry is implemented (illustrative identifier: `effect.retry_scheduled`), attributed to the Effect ID and describing the Effect-level `RetryScheduled` transition (§16.1) — never to any Effect Attempt ID, since no new attempt yet exists at that point. Facts equivalent to the new attempt being dispatched and its own completion or failure MUST likewise be truthfully, distinctly auditable (illustrative identifiers: `effect.retry_dispatched`, `effect.retry_completed`, `effect.retry_failed`), attributed to that new attempt's own, freshly minted Effect Attempt ID once it exists. Their exact names are an implementation-phase concern, consistent with ARCH-005 §24 and ARCH-007 §19's own established precedent for deferring concrete audit-event identifiers, but the *requirement* that retry activity be truthfully observable, and correctly attributed to the Effect ID before an attempt exists and to the correct Effect Attempt ID once one does, is architectural and not deferred.

**Ordering requirement**, extending ARCH-004 §18's and ARCH-007 §19's own established discipline: no audit record may claim an Effect requested, authorized, dispatched, completed, failed, cancelled, or timed out before the underlying fact has genuinely, verifiably occurred. Authorization-decision events (`effect.authorized`/`effect.denied`) and operation-outcome events (`effect.completed`/`effect.failed`/etc.) MUST remain distinct: a recorded authorization is not itself evidence the operation occurred, and a recorded outcome is not itself evidence it was properly authorized — the identical separation ARCH-007 §21's invariant 12 already requires for persistence operations.

A requested, denied, timed-out, failed, or cancelled Effect MUST NOT be represented as successfully completed under any circumstance, including a late provider signal arriving after a terminal outcome is already recorded (§16, §20, §21).

ADR-0015's audit-failure semantics apply unmodified: a mandatory Effect-related audit emission that fails causes the reporting operation to fail, without rollback of already-committed component-level state.

### 17.1 Disclosed Gap: Timestamp and Correlation-Identifier Fields (0.5.0)

Found during Independent Architecture Review AR-009 and disclosed here rather than left to be silently discovered: the current, tracked `AuditEvent` structure carries exactly four fields — `event_type`, `actor`, `capability`, `message` — **no timestamp field, and no explicit correlation/trace-identifier field of its own** (correlation is recoverable only indirectly, by cross-referencing the named `message`'s own `MessageId` against Effect Coordinator bookkeeping, §15). The ordering requirement above can be satisfied through emission sequence alone — but a consumer wanting a genuine wall-clock timestamp, or a directly-carried correlation identifier on the event itself, cannot obtain either from `AuditEvent` today. **This is not resolved here.** Concrete audit-event field additions remain, consistent with §4/§30's own repeated pattern, an implementation-phase concern for a future, separately authorized Engineering Work Order.

## 18. Failure Semantics

Provider failure returns a truthful Effect failure outcome (`effect.failed`) through the ordinary admission/dispatch failure path — this is the direct, load-bearing extension of ARCH-005 §17's own normative statement, "Timer delivery failures are admission failures. They are not supervision failures... never routed to Supervisor," applied here to Effect failures of every kind.

Effect failure MUST NOT automatically:

- terminate the Runtime;
- terminate the requesting actor;
- invoke Supervisor as though the requesting actor had crashed;
- authorize a retry (retry remains a distinct, explicit decision, §19);
- trigger compensation (compensation remains a distinct, future-deferred concern, §25).

The following failure categories are architecturally distinct and MUST NOT be conflated in audit records or in Effect Coordinator state:

| Category | Description |
|---|---|
| Provider business/operation failure | The provider genuinely attempted the operation and it did not succeed on its own terms (e.g., an HTTP 404, a query returning a constraint violation) — an ordinary `effect.failed` outcome for this attempt (§16, §17); the Provider Actor itself is entirely unaffected |
| Provider actor execution failure | The Provider Actor's own `Actor::handle()` returned `Err` before or during the attempt — an ordinary actor execution failure, routed through Runtime's own existing, unmodified execution-failure path exactly as any other actor's would be (ARCH-006 §9.1). Where the Provider Actor is registered with Supervisor, this MAY genuinely result in an ordinary `Restart`, `Stop`, or `Escalate` decision, made entirely by Supervisor and Runtime's own existing machinery, unmodified by this document. Runtime MUST inform the Effect Coordinator that the attempt this Provider Actor instance was handling can no longer reach a genuine completion signal, so the attempt can be driven to `ProviderLost` (§16, §21) rather than left indefinitely `Dispatched`/`Executing`. The Effect Coordinator neither observes nor decides the underlying supervision outcome — it only reacts to Runtime's own truthful notification that the outcome occurred (§10, §21) |
| Admission failure | The Effect request or outcome message itself failed the ordinary admission pipeline (envelope, authority) before ever reaching a provider |
| Authorization denial | Capability Authority declined to authorize the request (`effect.denied`) — distinct from every failure category above, since the operation was never attempted; an Effect-level outcome, never an attempt-level one (§16.1) |
| Timeout | No result arrived within the coordinated deadline (§20) |
| Cancellation | The request was explicitly or coordinately ended before completion (§21) |
| Provider Actor lost | The Provider Actor instance handling this attempt was restarted, stopped, terminated, replaced, or otherwise made unavailable by ordinary supervision before completion could be observed (§21) — a distinct category from provider actor execution failure above precisely because it is the *consequence*, not the triggering fault: the triggering fault is always one of "provider actor execution failure" or an externally-driven stop/termination; "provider lost" is the truthful attempt-level outcome that follows from it |
| Audit-infrastructure failure | The mandatory audit emission itself failed — governed by ADR-0015, unmodified |
| Runtime-infrastructure failure | A genuine Runtime-level fault unrelated to the Effect's own outcome |

Each category integrates with existing Runtime failure and supervision rules exactly as its nearest existing precedent already does: provider actor execution failure follows ARCH-006's own actor-failure handling and, where applicable, ARCH-004's own unmodified Supervisor restart/stop/escalate mechanics; admission failure follows the ordinary admission-rejection path; audit-infrastructure failure follows ADR-0015 unmodified. **No new supervision exception, and no second supervision mechanism, is introduced by this document** — the Effect Coordinator supervises nothing; it is only ever informed, by Runtime, of a supervision outcome Supervisor and Runtime's own existing, unmodified machinery already reached (§10, §21).

### 18.1 Provider Error Model (0.5.0)

Prior to EWO-017, this document's own automatic dispatch-outcome wiring admitted exactly two paths from a Provider Actor's own `Actor::handle()` return: `Ok(_)` -> `Completed`; `Err(_)` -> `ProviderLost`. No path existed for a Provider to report an ordinary, retry-eligible "provider business/operation failure" (above table) — for example, a refused network connection — without being misclassified as `ProviderLost`, which incorrectly implies the Provider Actor instance itself is broken (rather than the operation's own external target).

EWO-017 closed this additively, and this subsection generalizes the result as the canonical pattern every future Provider uses: a reserved `Message::message_type` value, carried on one message included in a Provider's own successful (`Ok`) return, causes Runtime to record `Failed` instead of `Completed` for the correlated attempt — the Provider Actor instance itself remains genuinely healthy and available for a future attempt, exactly as "provider business/operation failure" (above) already requires. Every pre-existing `Ok`/`Err` path, for every Provider and test predating this subsection, is completely unaffected. The specific reserved value and its exact wiring are implementation, not architecture (§4); the *requirement* that such a path exist, and that it never be conflated with `ProviderLost`, is architectural as of this subsection.

## 19. Retry Architecture

**Retry Policy** (maximum attempts, delay, backoff, jitter) is numeric policy and remains outside this constitutional architecture, owned by the requesting actor or a policy-bearing capability constraint — never Runtime, never the Effect Coordinator — on the identical basis ARCH-004 §4/§21, ARCH-005 §4/§22, and ARCH-007 §24 have each already, independently, refused to fix any numeric timing or backoff policy at the architecture level.

**Retry Decision** — whether a given attempt's failure, timeout, or provider-lost outcome should be followed by a new attempt at all — is coordinated by the Effect Coordinator (§10), using: explicit actor-supplied retry intent; provider-declared idempotency metadata (§23); capability constraints; and future policy input not designed here. **The Effect Coordinator MUST NOT silently retry a `NonIdempotent` or `Unknown` Effect** — this is a hard architectural prohibition (§31 invariant 33), not a default that may be overridden by convenience. **The Effect Coordinator MUST NOT silently re-execute an attempt lost to Provider Actor restart, stop, or termination** (`ProviderLost`, §16, §21, §31 invariant 5) — a new attempt following a lost one is subject to the identical retry-decision discipline as a new attempt following any other failure, never an automatic, unauthorized re-dispatch.

**Retry Scheduling** reuses the existing Timer Service and Temporal Runtime directly (ARCH-005) — a scheduled retry is architecturally identical to "wake me at T," and this document introduces no Effect-specific timer subsystem of any kind (§31 invariant 32). Once the Retry Decision is made, the Effect transitions to `RetryScheduled` (§16.1) — an Effect-level state, not an attempt state, since no new Effect Attempt exists yet — for the duration between that decision and the new attempt's own authorization succeeding.

**Retry Execution** begins once the scheduled timer fires and the new attempt's own fresh authorization succeeds; only at that point does it mint a fresh Effect Attempt ID (§15, §16.2) under the same, unchanged Effect ID — it is a new attempt, never a continuation of the prior one, and never a reopening of the `RetryScheduled` Effect-level state, which ends the moment this new attempt's Effect Attempt ID is minted. It MUST:

- use ordinary Runtime admission (§13);
- undergo fresh capability validation (§14), never inherited from the original attempt;
- receive its own distinct Effect Attempt ID (§15), while remaining associated with the original Effect ID;
- emit truthful, distinct retry audit events, attributed to its own Effect Attempt ID (§17);
- be dispatched to whatever Provider Actor destination is currently valid for the presented capability, which MAY differ from the destination an earlier attempt of the same Effect used (§14, §15) if the Provider Actor has since been replaced.

A signal belonging to an earlier attempt MUST NOT be interpreted as belonging to, or permitted to overwrite the outcome of, a later attempt of the same Effect (§15, §16).

This decomposition reuses Temporal Runtime for scheduling and the existing admission pipeline for execution without exception; the Effect Coordinator adds no new timer mechanism and no new admission mechanism of its own.

### 19.1 Retry Eligibility

The following completes, rather than changes, the retry-eligibility fact already implied by §16.2's own per-outcome annotations:

- **Retry-eligible** (an attempt-level terminal outcome that MAY be followed by a Retry Decision, §19): `Failed`, `TimedOut`, `ProviderLost`, and no other attempt-level terminal outcome.
- **Never retry-eligible** (an attempt-level terminal outcome that MUST NOT be followed by any Retry Decision for that same attempt): `Completed` (the Effect already succeeded — retrying a completed attempt would risk a duplicate operation the idempotency model itself governs at the operation level, §23, not by re-attempting a specific attempt that already succeeded) and `Cancelled` (the attempt was deliberately ended; a cancelled attempt is never eligible for automatic or actor-authorized retry — a requesting actor wishing to proceed after cancellation submits a new, independent Effect request, never a "retry" of the cancelled one).
- **`Denied` is not an attempt-level outcome at all** (§16.1, §31 invariant 40) and is therefore categorically outside this section's own retry-eligibility question: no Effect Attempt, and no Effect Attempt ID, ever existed to retry. A requesting actor whose Effect was `Denied` may submit a fresh, independent Effect request once the authorization defect is corrected; this is an ordinary new Effect, never architecturally a "retry" of the denied one, since the two share no Effect Attempt lineage.

This subsection introduces no new lifecycle state, no new terminal outcome, and reopens no decision recorded in §16.

### 19.2 Retry Authority

Retry-related responsibility is exhaustively partitioned across exactly four roles, none of which may supply another's input or perform another's decision:

| Role | Responsibility | Explicitly excluded |
|---|---|---|
| **Provider Actor** | The idempotency classification informing the Retry Decision (§19.3) describes this role's own operation (§23, §23.1) — "provider-declared" denotes whose operation is described, never which component technically performs registration (§23.1's own disclosure, unmodified here) | Never expresses, supplies, or infers retry intent; never decides or triggers a retry; never observes or participates in the Retry Decision itself (§10's existing "no provider-owned authorization decisions" principle, §14, extended here without modification) |
| **Requesting actor** | The sole source of retry intent, per §9's own ownership-table entry ("deciding domain-level retry intent — including, as one specific case, whether to request an Effect again after restoration") | Never the subject of an idempotency classification (that describes the Provider Actor's own operation, §23.1, never the requesting actor's); never enforces a capability constraint; never itself dispatches a retry |
| **Runtime** | Orchestrates the retry-decision sequence and the retry dispatch through ordinary admission (§9.1, §13), exactly as it orchestrates every other Effect-related sequence | Never decides whether a retry occurs — that determination belongs exclusively to the Effect Coordinator (§9, §10); gains no new decision authority by virtue of this subsection |
| **Effect Coordinator** | The sole decision-maker: combines retry eligibility (§19.1), the requesting actor's retry intent (this section), the Provider's idempotency declaration (§23, §19.3), and any applicable capability constraint (§19.4) into one Retry Decision (§19) | Never a source of retry intent itself; never an idempotency authority; never a capability authority; decides no authorization of its own (§10) |

**Retry intent is expressed once, as part of the original Effect request, never re-solicited per attempt.** The Effect Coordinator is bookkeeping-only and non-interactive (§10): it has no mechanism to "ask" a requesting actor mid-lifecycle whether a specific failed attempt should be retried, and this document introduces none. Retry intent is therefore domain-level, actor-declared data the requesting actor supplies at request time — carried as ordinary request payload or a capability-declared constraint (§14) — never inferred from the absence of a stated preference, and never defaulted to "yes." §9's own ownership table already names the requesting actor as the owner of "domain-level retry intent," restoration named there as one specific case of it, not the whole of it; this subsection completes that single, authoritative statement with the retry-decision mechanics it did not itself need to restate, without altering the restoration case's own, already-correct treatment (§22).

### 19.3 Idempotency-Class Retry Permission

Completing the interaction §19's existing prohibition only states negatively (Effect Coordinator "MUST NOT silently retry a `NonIdempotent` or `Unknown` Effect"), the constitutional retry permission for each idempotency class (§23) is:

| Idempotency class | Retry permission |
|---|---|
| `Idempotent` | Retry **permitted**, subject to retry eligibility (§19.1) and the requesting actor's own expressed retry intent (§19.2) — an `Idempotent` declaration never itself compels or automatically triggers a retry the actor did not request; it removes the *safety* objection to retrying, never the *intent* requirement |
| `NonIdempotent` | Retry **prohibited by default**; permitted only where the requesting actor's retry intent (§19.2) includes an explicit, distinct acceptance of the risk for that specific Effect request — never inferred, never satisfied by a general-purpose "always retry" preference, and never triggered by the idempotency declaration alone |
| `Unknown` | Identical treatment to `NonIdempotent`, per §23's existing conservative-default rule ("`Unknown` MUST be treated conservatively — equivalent to `NonIdempotent`... until a provider affirmatively declares otherwise") |

An idempotency declaration only ever **narrows** what the Retry Decision may permit; it never independently **authorizes** a retry the requesting actor did not, itself, request (§19.2 remains the sole source of retry intent, in every idempotency class without exception). This subsection reopens no rule stated in §23; it completes the retry-specific consequence §23 already names as informing, without itself resolving, the Retry Decision.

### 19.4 Retry Limit Ownership

§19's own Retry Policy statement, above, leaves retry-limit ownership as an unresolved disjunction ("the requesting actor **or** a policy-bearing capability constraint"). This is resolved, without introducing any new mechanism, by direct analogy to the existing, already-approved capability attenuation model (§14: "a Provider Actor or a downstream requester may only ever narrow, never widen, an Effect capability it delegates"):

- A capability-declared retry-limit constraint, where one is presented, is a **hard ceiling**: the Effect Coordinator MUST NOT permit a number of retry attempts exceeding it, regardless of the requesting actor's own expressed retry intent (§19.2) — "permit" here is the Effect Coordinator's own bookkeeping-level enforcement of an already-declared numeric constraint (§10, §20's identical timeout-enforcement precedent), never a capability-authorization decision, which remains exclusively Capability Authority's own act (§8, §14).
- A requesting actor's own retry-limit preference MAY further **narrow** (reduce) the effective limit below any applicable capability-declared ceiling, on the identical narrowing-only basis §14 already establishes for capability delegation generally; it MUST NOT widen the effective limit beyond that ceiling.
- Where no capability-declared ceiling is presented, the requesting actor's own retry-limit preference alone governs.
- Where neither a capability-declared ceiling nor an actor-supplied preference is presented, no architecture-level bound exists for that Effect; this document still fixes no numeric default of any kind (§4, §33) — only ownership and precedence between the two possible sources is resolved here.

This subsection introduces no numeric retry policy, no backoff or jitter calculation, and no new capability, capability operation string, or authorization step (§14, §23.5's identical "no new gate" reasoning applies here without modification) — it resolves an ownership-precedence ambiguity using a mechanism (attenuation-only narrowing) this document already, independently established for a different numeric constraint.

## 20. Timeout Architecture

| Concern | Ownership |
|---|---|
| Timeout policy (the numeric duration) | Requesting actor or capability-declared constraint — never Runtime, never the Effect Coordinator, on the same basis as retry policy (§19) |
| Timeout scheduling | Reuses the existing Timer Service directly, per attempt (§15) — no new mechanism |
| Timeout enforcement | Effect Coordinator, upon the scheduled timer firing, transitions the specific attempt to `TimedOut` if it has not already reached a terminal state |
| Timeout cancellation | If an attempt completes before its own timeout fires, the Effect Coordinator cancels the corresponding timer registration for that attempt |
| Timeout audit | `effect.timeout`, attributed to the specific Effect Attempt ID (§15, §17) |
| Timeout propagation | The requesting actor receives an ordinary reply message carrying the truthful `TimedOut` outcome — never a silent drop, extending ARCH-007 §18's "no partial or ambiguous outcome may be represented as fully determinate" |

**Timer cancellation and the completion/firing race, disclosed explicitly.** The Timer Service exposes two distinct cancellation operations with different failure behavior: a per-timer cancellation, which may report that the targeted timer has already fired or already transitioned out of its pending state, and a cancel-all-for-actor operation, which cancels every currently pending timer for a given `ActorId` and cannot itself fail. Whichever operation an implementation uses for per-attempt timeout cancellation, a genuine race between an attempt's own completion and its timeout independently firing is expected and MUST be treated as harmless: if cancellation reports that the timer had already fired, this is not a new failure category requiring escalation — it is resolved exactly as any other late signal already is (below), by discarding whichever of the two truthfully-later-recorded signals loses the race, never by treating the cancellation attempt itself as an error condition.

**A late provider completion after timeout MUST NOT overwrite the timeout terminal state.** Once `TimedOut` is recorded for an attempt, the Effect Coordinator MUST discard a subsequently arriving provider result for that same attempt without applying it, and MUST truthfully audit that a late result was received and discarded (an implementation-phase concrete event name is not fixed here, consistent with §17's own deferral of concrete identifiers) — that attempt's own outcome for every purpose visible to the requesting actor and to audit remains `TimedOut`.

**External outcome uncertainty.** `TimedOut` means the Runtime stopped waiting under the applicable timeout for this attempt. It does not, by itself, prove that the provider or the external system it was calling actually stopped, or that no external side effect occurred (§21 states the identical principle for cancellation, generally).

## 21. Cancellation Architecture

Cancellation is Runtime-coordinated, applying to:

- **explicit actor cancellation** — an ordinary, capability-authorized request to end a specific Effect;
- **requesting-actor termination** — the requesting actor's own instance-level termination;
- **Provider Actor lifecycle loss** — the Provider Actor instance handling an active attempt is restarted, stopped, terminated, replaced, or otherwise made unavailable by ordinary Runtime supervision (§18). This is distinct from every trigger above: it originates from the *destination* side of an attempt, not the requester's own state, and it is a genuinely new trigger this correction adds. Runtime MUST cause any attempt so affected to transition to `ProviderLost` (§16) rather than remain indefinitely `Dispatched` or `Executing` — this is a constitutional requirement of this document, stated without prescribing a concrete notification API: how Runtime learns of and propagates the underlying supervision outcome to the Effect Coordinator (a direct call following `route_actor_execution_failure`-equivalent processing, an internal event, or an equivalent Runtime-orchestrated mechanism) is an implementation decision for a future Engineering Work Order (§33). What is fixed here is that Provider Actor supervision and Effect coordination MUST be integrated through Runtime-owned orchestration — never through a second supervision mechanism, and never through the Effect Coordinator supervising Provider Actors itself (§10, §18). A Provider Actor restart following such a loss MUST NOT silently re-execute the lost attempt; a new attempt requires the identical, explicit retry-decision and retry-scheduling discipline §19 already establishes for any other failure (§19, §31 invariant 5);
- **Persistent Actor durable deletion** — reusing, without redesigning, the deletion-coordination-ordering pattern ARCH-007 §17 already establishes (validate authority → cancel dependent state → audit → attempt the definitive act → audit outcome), extended here to pending Effects exactly as ARCH-007 §17/ARCH-005 §23 already extended it to pending timer registrations. Durable deletion MUST cancel or terminally resolve every pending Effect associated with the deleted `ActorId` before deletion is reported complete (§31 invariant 20) — resolving Effect coordination in this sense means every pending attempt reaches a terminal coordination outcome (most commonly `Cancelled`); it does not mean, and MUST NOT be represented as meaning, that the Runtime has thereby learned or certified whether any external side effect occurred (see "External outcome uncertainty," below);
- **Runtime shutdown** — no new Effect dispatch begins once Runtime is no longer accepting new work, mirroring the `RuntimeState::Running`-guard pattern already used throughout the Persistent Actor implementation;
- **capability revocation** — handled by the existing fire-time/dispatch-time revalidation (§14) without new machinery, exactly as timer firing already handles it;
- **provider unavailability** — best-effort; the Effect Coordinator may still record cancellation or failure even where the underlying external system cannot itself be signaled to stop;
- **future distributed migration** — deferred (§29), not designed here.

Provider cancellation support MAY be best-effort where the external system does not support cancellation; the Effect Coordinator's own bookkeeping obligation is unaffected either way. **The Effect Coordinator MUST still prevent a cancelled attempt from later becoming successfully completed in Runtime state** — the identical late-signal discipline §20 establishes for timeouts applies here without modification: a late provider completion after cancellation is discarded and truthfully audited, never applied.

**External outcome uncertainty, stated as one general principle.** Runtime terminal coordination state is not proof of external-world state. Specifically:

- `Cancelled` means Runtime coordination for an attempt has been cancelled or terminally resolved; it does not, by itself, mean the external system performed no operation;
- `TimedOut` means the Runtime stopped waiting under the applicable timeout (§20); it does not, by itself, prove the provider or the external system it was calling actually stopped;
- `ProviderLost` means the Runtime can no longer observe a genuine completion signal for an attempt; the external operation the lost Provider Actor instance may have already initiated could have completed, partially completed, or never started — the Runtime does not, and cannot, know which;
- durable deletion resolving its owned Effect coordination (above) does not create knowledge the Runtime does not otherwise possess about whether a pending Effect's own external side effect occurred.

This document does not introduce a new `Unknown`, `Indeterminate`, or `OutcomeUncertain` lifecycle state. The terminal outcomes already defined (§16) are truthful as *Runtime-coordination* facts; this paragraph exists so that no future reader mistakes a Runtime-coordination fact for a certified claim about physical, external reality, which none of them are or were ever intended to be.

This document does not redesign the already-published deletion architecture (ARCH-005 §23, ARCH-007 §17); it extends the identical ordering and failure-semantics discipline to a new dependent-state category (pending Effects) exactly as that architecture already extended it once, from pending timers, without requiring either document to change.

## 22. Determinism and Persistence

> **In-flight Effect execution is Runtime execution state, not actor domain state.**

This is the direct, required extension of ARCH-007 §9's own exhaustive rule — "any data scoped to one in-progress execution cycle... MUST remain opaque and non-persisted" — to a data category ARCH-007 could not have anticipated (a request whose outcome depends on an external, non-deterministic resource). Therefore:

- in-flight Effects MUST NOT be included in actor checkpoints;
- restore MUST NOT automatically recreate or redispatch an in-flight Effect;
- the Persistence Service MUST NOT infer that an external Effect did or did not occur;
- actors MAY persist domain-level intent or acknowledgement state as their own ordinary domain data (e.g., "I asked for X, and have not yet heard back") — this is the actor's own affair to encode, never something the Effect Coordinator or Persistence Service does on the actor's behalf, mirroring the identical discipline this corpus's own timer-cleanup work already established for timer re-arming after restoration;
- actors, not the Runtime, are responsible for deciding whether to request an Effect again after restoration;
- `NonIdempotent` and `Unknown` Effects MUST NOT be silently replayed under any circumstance, including as a side effect of restoration (§23);
- Effect Coordinator bookkeeping (§10), including all per-attempt tracking (§15), MUST NOT become actor domain state — it is Runtime-session-scoped tracking, not a durable record an actor may read as its own data;
- an Effect result received after the requesting actor's deletion, or in an otherwise invalid restoration context, MUST be handled without resurrecting the actor — the result is discarded and truthfully audited (§17), on the identical late-signal discipline §20/§21 already establish, never treated as a reason to recreate a live instance.

This section's own determinism/persistence interaction is the one genuinely novel synthesis in this document — no prior architecture addressed Effects-meets-Persistence directly — and is stated here normatively, with explicit `MUST NOT` language, precisely so that no future implementer must re-derive it by inference.

## 23. Idempotency

Provider-declared idempotency metadata is reserved as: `Idempotent`, `NonIdempotent`, `Unknown`.

- Idempotency is separate from Effect Classification (§24) — a classification never implies an idempotency value.
- `Read` does not automatically mean idempotent — a read against a system with side-effecting read-triggers, rate-limited quotas, or non-deterministic ordering is not safely idempotent merely because it is a read.
- `Pure` does not automatically mean replayable unless its inputs and environment are genuinely deterministic — a nominally "pure" computation depending on wall-clock time or external configuration is not safely replayable.
- `Transactional` does not automatically mean safe to retry — a transaction that partially committed before failure may not be safe to retry blindly.
- `LongRunning` and `Streaming` do not determine idempotency — duration and delivery shape are orthogonal to whether repeating the operation is safe.
- `Unknown` MUST be treated conservatively — equivalent to `NonIdempotent` for every retry and replay decision (§19, §22) until a provider affirmatively declares otherwise.
- Retry (§19) and restoration behavior (§22) MAY depend on idempotency metadata; neither may depend on classification alone (§24).
- Providers MUST NOT falsely claim idempotency to obtain automatic retries — a false claim is a provider-implementation defect, not an architectural gap this document could close by mechanism; the architecture's own obligation is to never retry without a declared basis, which it satisfies regardless of whether any individual provider's declaration is itself honest.

### 23.1 Registration Model

Idempotency metadata is declared at **operation granularity**, never at the granularity of one individual Effect request. A classification is registered once per `(Provider Actor `ActorId`, operation identifier)` pair — the identical `effect.<domain>.<operation>` granularity the capability model (§14) already uses — not per dispatched attempt, and not per retry. This is a completion of the reservation already stated above ("provider-declared idempotency metadata," not "per-request idempotency metadata"), not a new decision: §19 already frames idempotency as informing a Retry *Decision* about an operation's own safety, never as a per-call fact that could legitimately vary attempt-to-attempt for the identical operation.

A declaration's registration is not itself an Effect request, undergoes no Effect lifecycle (§16) transition, and mints no Effect ID or Effect Attempt ID. This section states the resulting fact's own granularity and ownership (§23.2) normatively; it does not state which component performs the registration act. Consistent with the passive, reactive Provider Actor model this document already establishes elsewhere (§14: "No provider-owned authorization decisions — a Provider Actor never decides whether a request reaching it was authorized"), the concrete registration mechanism — whether Runtime-mediated, established at Provider definition time, or otherwise — is deferred to implementation (§33), consistent with how this document already defers comparable mechanism questions (for example, the Provider-lifecycle-loss notification mechanism, §21). "Provider-declared," as used throughout this section and §23, denotes *whose operation the classification describes*, not a claim about which component technically initiates the registration act.

### 23.2 Runtime Ownership

The Effect Coordinator (§10) is the sole owner of registered idempotency declarations, on the identical basis it already owns every other Effect-related correlation fact this document assigns it (§15, §20, §21). This ownership is bookkeeping only (§10, §31 invariants 29–31): the Effect Coordinator records and answers queries about a declaration; it does not itself decide, from a declaration alone, whether a retry occurs — that remains the Retry Decision's own responsibility (§19), unchanged by this section.

A registered declaration is Runtime-session-scoped Effect Coordinator bookkeeping, not actor domain state, on the same basis §22 already, generally establishes for all Effect Coordinator bookkeeping: it MUST NOT be included in actor checkpoints, MUST NOT be treated as data any actor may read as its own, and MUST NOT survive a Runtime process restart as a durable record. A Provider re-declares on a fresh Runtime session, exactly as a Provider Actor instance itself is freshly created rather than restored as domain state (§22).

### 23.3 Failure and Default Semantics

- A Provider that has registered no declaration for a given operation is treated identically to an explicit `Unknown` declaration — the existing conservative default (above) already covers the no-declaration case without requiring a distinct architectural rule.
- Across a Runtime restart, all registered declarations are lost, exactly as §23.2 requires; every operation reverts to the undeclared (`Unknown`-equivalent) state until the responsible Provider re-registers within the new session. This is not data loss requiring recovery — it is the same, already-accepted session-scoping every other piece of Effect Coordinator bookkeeping already has (§22).
- A Provider Actor restart, stop, or replacement (§21) does not, by itself, invalidate or require re-declaration of that Provider's own registered classifications within the same Runtime session — the declaration is scoped to the operation and the Runtime session, not to any one Provider Actor instance, consistent with capability targets themselves remaining meaningful across a Provider Actor's own replacement (§14).
- A Provider that registers a new declaration for an operation it has already declared replaces the prior declaration for that operation; this is an ordinary, always-legal registration, never a rejected or erroneous act — it introduces no new failure mode.
- Nothing in this section alters the Retry Decision's own existing failure semantics (§19) or the late-signal discipline (§16.2, §20, §21) — a declaration informs whether a retry may be attempted at all; it never determines, or interferes with, how any individual attempt's own outcome is recorded.

### 23.4 Provider Correlation

Every Effect-request message Runtime admits to a Provider Actor already carries the requesting Effect's own Effect ID as that message's `correlation` value (ARCH-002 §8), stable and unchanged across every retry attempt of the same Effect (§15, §19). **This is the one Message field an Effect-request message's own construction does not leave to the requesting actor to supply.** §15 already, generally states that an Effect's own "correlation information" is "supplied by the requesting actor" — that general statement continues to govern ordinary, non-Effect messages and any actor-supplied correlation data an Effect request's own payload may separately carry; for the specific `Message.correlation` field of the Effect-request message Runtime itself constructs and admits, Runtime populates it directly with the Effect ID, so that every attempt of the same Effect — including every retry — carries an identical, stable value regardless of what the requesting actor did or did not supply. This document now states normatively what was already, structurally true of the existing, unmodified admission pipeline: **a Provider MAY treat the correlation value it receives as a stable, Runtime-supplied token identifying "this specific logical Effect," usable for the Provider's own external-system-facing request-deduplication purposes, entirely at the Provider's own discretion.** This is not a new mechanism — no new field, no new message shape, and no new admission behavior is introduced; it is a disclosure of an existing capability of the correlation value the admission pipeline already carries. It is also explicitly distinct from, and MUST NOT be conflated with, the idempotency declaration this section otherwise governs: the correlation value identifies *which logical Effect* a given attempt belongs to; the idempotency declaration states *whether that Effect's own operation is safe to retry at all*. A Provider may use one, both, or neither, and using the correlation value for external deduplication requires no capability beyond what dispatching to that Provider already requires (§23.5).

### 23.5 Capability Model

No new capability, capability operation string, or capability-authorization step is introduced by this section. This document's existing, unmodified capability model already, fully governs the operation an idempotency declaration describes: an `effect.<domain>.<operation>` capability is bound to **the requesting actor** and targets the Provider Actor authorized to receive that operation (§14) — the Provider Actor is never itself a capability holder, exactly as §14 already, generally establishes ("No provider-owned authorization decisions — a Provider Actor never decides whether a request reaching it was authorized"). A declaration is therefore recorded **against the same `(Provider `ActorId`, operation)` pair the existing capability structure already, independently gates** — no second, parallel authorization concept is introduced for registration, and no new gate is needed to reach the identical conclusion the existing capability architecture already provides for who may legitimately cause that operation to be dispatched at all. Introducing a second, independent authorization gate purely for registration would create a capability boundary this document has no evidence-based reason to add, and would not, by itself, prevent a dishonest declaration in any case (already addressed above: a false declaration is a provider-implementation defect, not something a capability check can detect).

Concrete metadata types, storage, and enforcement mechanics remain deferred to implementation (§33).

## 24. Reserved Effect Classification Model

Effect Classification is reserved as a future architectural extension point. It is **not implemented, not enforced, and not required by the first Effect Runtime implementation milestone** — an implementation MAY treat it as reserved metadata or omit runtime enforcement of it entirely, provided future adoption does not require redesign.

**The six reserved names are independent, combinable metadata dimensions, not one mutually exclusive enum, and MUST NOT be represented as a single closed classification axis.** A single Effect MAY carry more than one of these traits simultaneously. Illustrative combinations that describe entirely ordinary real-world operations: `Write` + `Transactional` (an operation that changes external state within a provider-defined transactional boundary); `Read` + `Streaming` (an operation that retrieves external information as an ongoing sequence rather than one terminal response); `Write` + `Transactional` + `LongRunning` (a transactional write whose own execution substantially outlives ordinary actor-message handling). This document does not decide, and future work remains free to decide, whether the eventual representation is a set of independent flags, a set of traits, separate metadata fields, or a structured descriptor (§33) — no concrete Rust type is defined here (§4).

**The six names describe different underlying concerns, grouped as follows, precisely so a future implementer does not collapse them into one axis:**

| Class | Concern it describes | Conceptual meaning |
|---|---|---|
| `Pure` | Externally observable effect | No external side effect at all. A genuinely `Pure` operation may ultimately not require a Provider Actor — the classification is reserved for future policy and optimization decisions, not a current requirement that every Pure operation route through this architecture. |
| `Read` | Externally observable effect | Intended to retrieve external information without intentionally changing the external system. Does not imply determinism, repeatability, or safety to retry (§23). |
| `Write` | Externally observable effect | Intended to change external state. Not assumed safe to retry or replay (§23). |
| `Transactional` | Transactional behavior/guarantees | Executed within, or participating in, a provider-defined transactional boundary. Does not require the Runtime to implement distributed transactions (§4). Combinable with `Read` or `Write`. |
| `Streaming` | Result/interaction shape | Produces or consumes an ongoing sequence of data rather than one terminal response. Streaming execution semantics are deferred (§33). Combinable with `Read` or `Write`. |
| `LongRunning` | Execution duration/coordination shape | May substantially outlive ordinary actor-message handling, or require later status observation. Long-running execution mechanics are deferred (§33). Combinable with any of the above. |

`Pure`/`Read`/`Write` describe *whether, and how, the external world is observably affected* — at most one of these three ordinarily applies to a given Effect, since they describe mutually exclusive observable-effect characterizations. `Transactional`, `Streaming`, and `LongRunning` describe orthogonal properties of *how* an Effect executes, independent of whether it reads, writes, or does neither, and MAY freely combine with each other and with the applicable `Pure`/`Read`/`Write` trait.

**Constraints, binding on this and every future document that adopts classification:**

- classification is provider-declared metadata, never a type-system enforcement mechanism fixed here;
- classification is not an authorization mechanism — it neither grants nor withholds capability authority (§14);
- classification does not replace capability operation strings — the two are orthogonal;
- classification does not itself prove idempotency, retry safety, or replay safety (§23);
- classification does not authorize compensation (§25);
- classification does not create new execution paths — every classified Effect still converges through the identical admission pipeline (§13);
- classification MUST NOT alter the ordinary admission pipeline in any way;
- classification MAY later inform: retry eligibility, timeout defaults, concurrency controls, audit detail, approval requirements, compensation planning, Control Centre presentation, Workflow Runtime behavior, distributed-placement decisions, and provider health policy — each identified here as a deferred possibility, not current behavior.

## 25. Compensation Boundary

Compensation remains deferred, for the identical reason this exact concept has already been deferred three separate times in this repository (ARCH-004 §5, ARCH-005 §4, ARCH-006 §5, each naming "workflow compensation" or "generalized effect-retry frameworks" as out of scope).

- Compensation is not automatic rollback — no mechanism in this document, or implied by it, reverses a completed Effect automatically.
- Compensation belongs to future workflow or orchestration policy, not this document.
- A future Workflow Runtime may request compensating Effects through this same Effect Runtime — an ordinary consumer of ordinary Effect-outcome messages, exactly as ARCH-005 §21 already anticipated for a workflow engine generally.
- Compensating Effects MUST remain independently capability-authorized and audited, exactly as any other Effect request (§14, §17) — a compensation is a fresh Effect request, never a privileged shortcut.
- Provider Actors MUST NOT secretly perform cross-provider compensation — this would itself be a Provider-to-Provider invocation, prohibited by §12 without exception.
- The Effect Coordinator MUST NOT become a workflow engine — its own ownership boundary (§10) is exhaustive and does not include compensation planning or execution.

## 26. Resource Governance

| Concern | Ownership |
|---|---|
| Quotas and rate limits | Effect Coordinator (bookkeeping-level, using its own in-flight-request tracking) or the Provider Actor itself |
| Per-actor concurrency | Effect Coordinator, or the existing bounded-mailbox mechanism (ARCH-002 §13) where sufficient |
| Per-provider concurrency | Provider Actor's own mailbox capacity, reused unmodified — no new concurrency-limiting mechanism is required at the architecture level |
| Provider exhaustion and circuit breakers | Provider-adapter-owned, exactly as ARCH-002 §23's own "Provider Architecture" deferred-architecture row already assigns ("retry and circuit-breaking policy") |
| Mailbox back-pressure | The existing, unmodified bounded mailbox and `Overflow` rejection (ARCH-002 §13) |
| Long-running Effect limits | Deferred (§33) — a concern of the `LongRunning` classification's own future adoption (§24) |
| Streaming resource limits | Deferred (§33) — a concern of the `Streaming` classification's own future adoption (§24) |

**Runtime-level governance MUST NOT grant authorization that Capability Authority denied.** A quota, rate limit, or concurrency control may narrow what is permitted; none may widen it. This is the direct extension of the general non-amplification principle already governing every capability-adjacent mechanism in this corpus (ADR-0017 §"why bootstrap grants strengthen, rather than weaken" point 3).

## 27. Security Boundaries

- Providers own their own provider-specific credentials exclusively. Runtime, the Effect Coordinator, and requesting actors never receive them.
- Requesting actors never receive provider credentials, directly or indirectly — a requesting actor holds a capability naming an *operation*, never a credential the provider uses to perform it, exactly as ARCH-002 §22's existing prohibition on exposing provider credentials to ordinary actors already establishes, extended here without modification.
- The Effect Coordinator never stores provider credentials — its own ownership boundary (§10) explicitly excludes them.
- Credentials MUST NOT enter actor-visible audit payloads (§17) — an audit event may record that an Effect occurred, never the secret material used to perform it.
- Provider Actors remain ordinary actor trust boundaries for the purpose of actor-to-actor authority, message admission, and Runtime orchestration — governed by ARCH-001 §5.1's existing actor-isolation guarantee, which this document extends without alteration.
- Capability operation strings (§14) are the authorization boundary. Classification metadata (§24) is explicitly not an authorization boundary.
- Direct Provider-to-Provider invocation is prohibited without exception (§12).
- Future plugin providers remain subject to every rule in this document exactly as any other Provider Actor — no exception is created for third-party-authored provider code.
- No ambient filesystem, network, shell, database, or model access is granted merely because code runs inside the Runtime process — every such access requires a presented, valid, capability-authorized Effect request.

**Actor and capability isolation is not process, operating-system, or host-level sandboxing — disclosed explicitly, not left to be discovered.** This document's own mechanisms (capability authorization, message admission, Runtime orchestration, auditability, Runtime-owned lifecycle coordination, §12's Provider Isolation Rule) govern *which requests reach which Provider Actor* and *whether that reaching is authorized and observable*. They do not, by themselves, and cannot, prevent a Provider Actor's own implementation code from making arbitrary host-level calls once that code is legitimately executing inside the SynapseOS process. ARCH-001 §5.1's actor-isolation guarantee concerns an actor's own private, in-memory state — it is not, and was never intended as, a claim about operating-system-level resource sandboxing. Ordinary Provider Actor status, exactly as this document defines it, does not by itself create: process isolation; container isolation; syscall filtering; filesystem sandboxing; network sandboxing; operating-system user separation; hardware isolation; or secret-store enforcement. **Provider implementations therefore remain trusted to comply with their own architecturally granted scope** unless and until stronger, host-level enforcement is introduced by a future, separate architecture effort (§33) — this document does not design process isolation, sandboxing, syscall restriction, remote provider execution, or capability-aware host enforcement, and the absence of such a mechanism today does not weaken, and MUST NOT be used to weaken, any capability or Provider-isolation rule this document does establish (§12, §14).

This document does not prescribe concrete secret-store technology (§4, §30) — that remains a permitted implementation variation, on the same basis ARCH-002 §22 already permits variation in persistence technology generally.

## 28. Runtime State and Shutdown Behavior

- Effect dispatch follows the existing `RuntimeState::Running` guard discipline already established for checkpoint, restore, and delete operations in the Persistent Actor implementation: no new Effect dispatch begins once Runtime is no longer `Running`.
- Effects already in flight at the point shutdown begins are handled through the ordinary cancellation-coordination path (§21) — Runtime shutdown is one of the cancellation triggers this document already names, not a separate mechanism.
- No Effect Coordinator state is required to survive a Runtime-process restart under this document (§22) — in-flight Effects are Runtime execution state, and a Runtime restart is architecturally equivalent, for this purpose, to the loss of any other execution-state category ARCH-002 §10/ARCH-004 §14 already disclose as non-preserved across restart.
- This document does not define Runtime-startup Effect-reattachment policy — consistent with ARCH-007 §24's own identical deferral for Runtime-startup restoration policy generally (§33).

## 29. Future Compatibility

| Future capability | Compatible? | Basis |
|---|---|---|
| Workflow Runtime | Yes | An ordinary consumer of ordinary Effect-outcome messages (ARCH-005 §21's own explicit, direct precedent); compensation requests flow through the same Effect Runtime (§25) |
| Distributed Runtime / Cluster Runtime | Yes, structurally; not solved operationally | `ActorId`-keying (used throughout this document) is already location-transparent by contract (ARCH-002 §7), the identical property already keeping Temporal Runtime and Persistence Service distributable-in-principle without redesign |
| Runtime Control API | Yes | The Effect Coordinator's own tracked state (§10, §16) is exactly the kind of queryable, `ActorId`-keyed record every other replaceable service already exposes through Runtime-mediated access; the Control Centre observes, queries, and manages authorized Effect activity through this surface without directly contacting Provider Actors |
| SynapseOS Control Centre (Windows, Linux, macOS) | Yes | Consumes the same audit stream (§17) and Runtime Control API surface every other subsystem already does; no bespoke integration required; this document preserves a clean path toward it by never coupling Effect observability to any Provider Actor's own internal state |
| Intelligence Core | Yes | An ordinary requester of Effects, exactly like any other actor — AI inference is one of this document's own named examples (§2, §11) and requires no special-casing |
| Plugin providers | Yes | "Provider Actor is an ordinary actor" (§11) already accommodates a plugin as "an actor defined by third-party code," subject to every rule in this document without exception (§27) |
| Remote providers | Yes | The identical `ActorId`-keying and admission-pipeline reuse already accommodates a provider whose own implementation happens to be remote, without any change to this document's own ownership model |
| Cross-platform Runtime hosts | Yes | This document introduces no platform-specific mechanism of any kind |

No redesign of this document is anticipated for any of these eight future capabilities under the ownership model established here.

## 30. Explicit Non-Goals

Restated in full for clarity, consistent with §3/§4: this document does not design or implement an HTTP client; a filesystem API; an SQL driver; an email client; an AI-model adapter; a cloud SDK; shell execution; a broker client; a plugin marketplace; provider discovery; distributed Effect transport; streaming-protocol mechanics; long-running-operation polling mechanics; distributed transactions; compensation execution; numeric retry defaults; numeric timeout defaults; Runtime Control API endpoints; or Control Centre user interfaces.

## 31. Architectural Invariants

Any implementation of this architecture MUST preserve the following invariants:

1. The Runtime remains the sole cross-component orchestrator for every Effect-related sequence.
2. Provider Actors are ordinary capability-scoped actors, never a Trusted Core or Runtime-privileged construct.
3. Provider Actor supervision (restart, stop, escalation) remains owned exclusively by the existing, unmodified Supervisor and Runtime lifecycle machinery; the Effect Coordinator introduces no second supervision mechanism and never itself supervises a Provider Actor (§18, §21).
4. Loss of the Provider Actor instance handling an active attempt — restart, stop, termination, replacement, or other supervision-driven unavailability — MUST be reflected in that attempt's own Effect coordination state (`ProviderLost`, §16, §21), rather than left indefinitely `Dispatched` or `Executing`.
5. A Provider Actor restart MUST NOT silently re-execute a lost attempt; a new attempt requires the same explicit, authorized retry decision and scheduling as any other retry (§19, §21).
6. Every retry mints a new, distinct Effect Attempt ID; no retry reuses a prior attempt's own identity (§15, §19).
7. The Effect ID remains stable across every attempt made to satisfy it (§15).
8. Every late signal (a late completion, a stale timeout, a duplicate cancellation) is evaluated, and if applicable discarded, against the specific Effect Attempt ID it is attributed to — never against the Effect ID alone, and never permitted to overwrite a different attempt's own recorded outcome (§15, §16, §19, §20).
9. A given Effect Attempt reaches at most one terminal outcome (§16).
10. The Effect itself reaches at most one accepted logical terminal outcome, even where several of its attempts each reached their own attempt-level terminal outcome first (§16, §19).
11. Effect capabilities remain destination-bound under the existing, unmodified Capability model; holding an `effect.<domain>.<operation>` capability does not grant ambient authority over any actor implementing that operation other than the specific, authorized target (§14).
12. Replacing a Provider Actor under a different `ActorId`, or distributing load across multiple Provider Actor identities, MAY require capability reissuance, attenuation, or rebinding; this document does not assume such transparency (§14).
13. The six Effect Classification names are independent, combinable metadata dimensions, never a single mutually exclusive enum (§24).
14. Effect Classification grants no authority of any kind.
15. Actor and capability isolation, as this document defines them, are not process-, operating-system-, or host-level sandboxing; a Provider Actor's own implementation code is trusted to comply with its architecturally granted scope absent future, separate host-level enforcement (§27).
16. Provider execution MUST NOT prevent the Runtime from continuing to make bounded forward progress for unrelated actors, timers, Effect coordination, and lifecycle processing (§13); the concrete concurrency mechanism achieving this is an implementation decision, not fixed here.
17. Separate lifecycle states and audit events (specifically `Authorized`/`Dispatched`, and conditionally `Executing`) MAY be represented as one combined, truthful transition where the underlying Runtime mechanism cannot truthfully observe them as separate facts; this collapse MUST NOT be achieved by introducing a second, duplicate authorization or observation mechanism (§16, §17).
18. No second, Effect-specific admission or authorization pipeline exists, has ever existed, or may be introduced without itself becoming a new architectural decision.
19. A Runtime-recorded cancellation does not, by itself, prove that the external operation the affected attempt targeted was itself cancelled or did not occur (§21).
20. Durable deletion cannot report success while owned Effect coordination for the deleted `ActorId` remains unresolved (§21).
21. Durable deletion resolving its owned Effect coordination does not, by itself, constitute or claim certainty about whether a pending Effect's external side effect occurred (§21).
22. In-flight Effects and Effect Attempts are Runtime execution state, never actor domain state, and are never included in actor checkpoints.
23. Restoration never automatically replays an Effect or re-dispatches a lost attempt.
24. Provider Actors MUST NOT invoke other Effect Providers directly (§12).
25. A Provider Actor requiring another Effect MUST request it through the Runtime, exactly as an ordinary requesting actor would (§12).
26. This document introduces no new Trusted Core component and alters no existing Trusted Core component's own ARCH-002 §6 responsibility.
27. Capability Authority remains the sole authorization mechanism for every protected Effect operation.
28. Effect authorization is validated fresh, at the moment of dispatch (including every retry), and is never cached.
29. The Effect Coordinator does not perform provider-specific I/O of any kind.
30. The Effect Coordinator does not own provider credentials.
31. The Effect Coordinator does not authorize capabilities.
32. No Effect-specific timer subsystem exists; timeout and retry scheduling reuse the existing Timer Service and Temporal Runtime without exception.
33. `NonIdempotent` or `Unknown` Effects are never silently retried.
34. Effect Classification remains a reserved extension point and does not require immediate implementation.
35. Compensation is never owned by the Effect Coordinator.
36. Every mandatory lifecycle-transition audit event (§17) remains truthfully auditable, in genuine-occurrence order, subject to the collapse allowance in invariant 17.
37. Provider credentials are never exposed to requesting actors, the Effect Coordinator, or Runtime.
38. Future distributed execution must preserve the identical authorization, audit, and admission rules this document establishes — no distributed extension may introduce a second path around any of them.
39. No component this document touches gains capability-issuing authority it did not already, independently, hold.
40. `Denied` is an Effect-level outcome, never an Effect Attempt outcome; a request whose authorization fails — on its first try or on any subsequent retry's own fresh authorization check — never produces an Effect Attempt or an Effect Attempt ID (§16.1).
41. `RetryScheduled` is an Effect-level, transitional state, never an Effect Attempt state; it MUST NOT reopen, extend, or reuse an already-terminated attempt, and ends the moment a new Effect Attempt ID is minted for the next attempt (§16.1, §19).
42. An idempotency declaration is registered at `(Provider Actor `ActorId`, operation identifier)` granularity, never per individual Effect request or Effect Attempt (§23.1).
43. A registered idempotency declaration is Effect Coordinator bookkeeping, Runtime-session-scoped; it MUST NOT be included in actor checkpoints, MUST NOT be treated as actor domain state, and MUST NOT survive a Runtime process restart (§22, §23.2).
44. No idempotency-declaration registration act introduces a capability, capability operation string, or authorization step beyond the existing `effect.<domain>.<operation>` grant already governing that operation (§14, §23.5).
45. Given identical Runtime history — the same sequence of Effect requests, attempt-level terminal outcomes, requesting-actor-supplied retry intent, Provider-declared idempotency classifications, and applicable capability constraints — the Effect Coordinator's Retry Decisions (§19, §19.1–§19.4) MUST be identical. A Retry Decision is a pure function of Effect Coordinator tracked state (§10) and the inputs named in §19.1–§19.4 (retry eligibility, retry authority/intent, idempotency-class permission, and retry-limit ownership); it MUST NOT depend on wall-clock time, process identity, thread scheduling, or any other factor not itself part of that tracked, replayable history.
46. A Provider Actor's own instance lifecycle (`ActorState`, ARCH-002 §15) and an Effect Attempt's own outcome (§16.2) are distinct state machines and MUST NOT be merged, conflated, or represented as one "provider lifecycle" (§11.1).
47. An ordinary, retry-eligible Provider failure that leaves the Provider Actor instance itself healthy MUST be reported via the reserved provider-error-model message type, never via `Err` from `handle()` (§18.1).
48. Provider Classification (§11.3) and Effect Classification (§24) are independent taxonomies describing different subjects (the provider; the operation) and MUST NOT be merged into one classification scheme.
49. No Provider Classification tag (§11.3) grants, withholds, or modifies capability authority, retry eligibility, timeout behavior, or audit requirements.
50. Provider registration (§11.2) remains `ActorId`-direct, through the existing Actor Directory, until and unless a future, separately authorized architecture effort introduces a discovery mechanism; no provider manifest, registry, or discovery protocol is introduced by this document.
51. A future Provider's own concurrency realization (§13.2) MUST satisfy §13's forward-progress constraint and MUST NOT introduce a second dispatch or admission path to do so; this document selects no specific mechanism.

## 32. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| The Effect Coordinator becomes a second orchestrator, deciding *whether* to authorize rather than only tracking | Explicit ownership boundary (§9, §10) naming this a MUST NOT; the identical discipline already resisted successfully by Temporal Runtime and Persistence Service |
| Providers cache capabilities across dispatches | Explicit fresh-validation requirement, including for retries (§14, §19) |
| Retry or timeout policy leaks into the architecture as numeric defaults | Explicit policy/mechanism separation (§19, §20), following the identical, already-proven ARCH-004/ARCH-005/ARCH-007 pattern |
| Determinism drift — Persistence Service or the Effect Coordinator silently resurrects an in-flight Effect on restore | Explicit `MUST NOT` language in §22, stated normatively rather than left to inference, precisely to close this identified highest-risk area |
| A future implementer treats trusted/colocated providers as exempt from §12's isolation rule | §12 explicitly states the rule applies "even when both providers are trusted, built into the same crate, run in the same process, or are maintained by the same developer" |
| Effect Classification (§24) is mistaken for an authorization or safety mechanism | §23 and §24 each explicitly, repeatedly state classification proves neither idempotency nor authorization |
| An Effect dispatched to a Provider Actor that is later restarted or stopped is left permanently `Dispatched`/`Executing`, with no terminal outcome ever recorded | Explicit `ProviderLost` terminal outcome and Runtime-notification requirement (§16, §18, §21, §31 invariants 3–5) |
| A future implementer builds a real async/threading execution model to satisfy the forward-progress constraint, mistakenly believing this document mandates one | §13 and §31 invariant 16 explicitly state the constraint is implementation-independent and prescribes no specific concurrency technology |
| Effect Classification (§24) is implemented as a single closed enum, silently forcing an Effect to carry only one of the six traits | §24 explicitly states the six names are independent, combinable dimensions with worked combination examples |
| Actor isolation (§27) is mistaken for host-level sandboxing, leading to unwarranted trust in Provider Actor code exceeding its granted capability scope | §27 explicitly disclaims OS/process-level enforcement and states Provider implementations remain trusted absent future host-level enforcement |
| A late signal from an earlier Effect Attempt is applied against a later attempt's own outcome | Explicit Effect ID / Effect Attempt ID distinction (§15) with attempt-scoped late-signal discard discipline (§16, §19, §20, §31 invariant 8) |
| `RetryScheduled` is modeled as an attempt state, causing an already-terminated attempt to be reopened or reused to represent the waiting period before a retry | Explicit Effect-level placement (§16.1, §31 invariant 41); `RetryScheduled` exists only between an attempt's own terminal outcome and a new Effect Attempt ID being minted, never as a state of any attempt itself |
| `Denied` is modeled as an attempt outcome, causing an Effect Attempt ID to be minted for a request whose authorization never succeeded | Explicit Effect-level placement (§16.1, §31 invariant 40); an Effect Attempt, and its Effect Attempt ID, come into existence only once authorization for that specific try has genuinely succeeded (§15) |
| A future reader merges the Provider Actor's own instance lifecycle with the Effect Attempt's own outcome into one "provider lifecycle" | Explicit disentanglement (§11.1, §31 invariant 46) |
| A future Provider author invents a new failure-signaling convention instead of reusing the one this document now names, fragmenting the failure model across Providers | §18.1's explicit generalization and §31 invariant 47; §11.4's explicit MUST |
| Provider Classification (§11.3) is mistaken for an authorization mechanism, or merged with Effect Classification (§24) | §31 invariants 48–49; §11.3's own explicit "none of the eight" statement |

## 33. Deferred Decisions

The following are explicitly, deliberately deferred to future architecture or authorized implementation work, and are not resolved by this document:

- concrete provider implementations of every kind beyond EWO-017's own HTTP reference (§4, §30);
- a future provider discovery, versioning, or compatibility-negotiation mechanism beyond the current `ActorId`-direct model §11.2 now states normatively; any plugin-loading or plugin-marketplace mechanism;
- distributed Effect transport, cross-host provider routing, and clustering;
- streaming-protocol mechanics, long-running-operation polling mechanics, and any concrete streaming/partial-result wire representation (§13.1, §24, §26);
- distributed transactions and compensation execution mechanics (§25);
- numeric retry policy — the concrete maximum-attempt count, backoff calculation, and jitter — and numeric timeout durations (§19, §20; §19.1–§19.4 establish retry eligibility, role partition, idempotency-class permission, and limit-ownership precedence normatively, while the concrete numeric values and algorithms remain, as before, implementation-phase);
- concrete Effect-lifecycle, Effect ID, and Effect Attempt ID type shapes, and any `AuditEvent` field-layout change beyond new `event_type` string values — including the timestamp and correlation-identifier fields §17.1 discloses as currently absent (§15, §17, §17.1) — implementation-phase concerns;
- concrete idempotency-declaration type shape, registration transport mechanism, and internal storage/lookup structure (§23, §23.1–§23.5 establish registration granularity, ownership, session-scoping, and default/failure semantics normatively; the concrete representation remains implementation-phase, per this document's own established pattern for every comparable mechanism question);
- concrete audit-event identifiers for `effect.executing` and every retry-related event (§17);
- the concrete mechanism by which Runtime informs the Effect Coordinator of a Provider Actor lifecycle-loss event (§21) — a Runtime-orchestrated notification of some form is required; its precise shape is not;
- the concrete concurrency mechanism (threading, async execution, worker processes, or otherwise) by which Provider execution preserves Runtime forward progress — §13.2 organizes the design space EWO-017's own blocking-synchronous realization sits within, without deciding it (§13, §31 invariants 16, 51);
- a future logical provider identity or capability-rebinding facility decoupling Effect-capability authority from a specific Provider Actor `ActorId`, beyond the current, now-normatively-stated `ActorId`-direct model (§11.2, §14);
- Runtime-startup Effect-reattachment policy (§28);
- Effect Classification's own eventual concrete representation (flags, traits, a set, or a structured descriptor), enforcement mechanism, and which policies it informs (§24);
- future process isolation, syscall filtering, or other host-level sandboxing enforcement for Provider Actor implementations (§27);
- Runtime Control API endpoint design and Control Centre user-interface design (§29, §30);
- secret-store technology for provider credentials (§27).

None of these requires reopening any decision this document does record; each is a genuinely separate, bounded question left to the process — future architecture, ADR, or Engineering Work Order — that STD-001 already provides for it.

## 34. Consequences

**Positive consequences:** actors gain a uniform, capability-authorized way to request any externally observable or non-deterministic operation without ever directly owning the resource it touches; the entire framework is realized by reusing four already-existing mechanisms (the admission pipeline, the capability model, the audit-event shape, the Timer Service) rather than inventing new ones, minimizing both implementation risk and conceptual surface area; Persistent Actor determinism is preserved by explicit, normative exclusion of in-flight Effects from checkpointed state; the framework is structurally ready for a future Workflow Runtime, Distributed Runtime, and Control Centre without requiring this document to be revisited.

**Costs and risks, named explicitly:** the Effect Coordinator, like every other replaceable service before it, is a new component whose own bookkeeping-only discipline must be actively maintained in implementation, not merely declared here (§32); the Provider Isolation Rule (§12) requires ongoing implementation discipline to detect and prevent, since nothing in the ordinary Rust type system automatically prevents one actor's implementation from holding a reference to another; the late-signal discard discipline (§16, §20, §21) requires genuinely careful implementation to avoid a race between a terminal-outcome write and a late provider signal, now scoped per Effect Attempt ID rather than per Effect (§15); deferring numeric retry/timeout policy (§19, §20, §33) means the first implementation milestone must still make real choices, just not ones this document fixes; the Runtime-forward-progress constraint (§13) is a genuine implementation obligation that a naive, purely synchronous Provider Actor implementation would violate, and satisfying it will require real engineering effort in the eventual Engineering Work Order; integrating Provider Actor supervision outcomes with Effect coordination (§21) requires a genuine, if narrow, new Runtime-orchestrated notification path that did not previously need to exist; and the reliance on Provider Actor implementation trustworthiness for anything beyond message-admission-level authorization (§27) is a real, disclosed limitation this document cannot itself close.

## 35. Approval Criteria

This document, and any implementation claiming conformance to it, is approved for formal design and subsequent implementation only if all of the following hold, restated from the governing Effect Runtime Architecture Review's own approval criteria:

1. Clean integration with the existing admission pipeline, with no second pipeline introduced (§13, §31 invariant 18).
2. Runtime ownership preserved — Runtime remains sole orchestrator (§9, §31 invariant 1).
3. Provider independence — Provider Actors are ordinary, capability-scoped actors (§11, §31 invariant 2).
4. Capability authorization as the sole authorization mechanism, with no second authority (§14, §31 invariant 27, 28).
5. Runtime-owned, truthful auditing for every mandatory lifecycle transition, with truthful collapse permitted where facts are not separately observable (§17, §31 invariant 17, 36).
6. Clear retry ownership, separating policy from mechanism, with every retry attributable to its own Effect Attempt ID (§19, §31 invariant 6, 7).
7. Clear timeout ownership, separating policy from mechanism (§20).
8. Persistent Actor determinism preserved — in-flight Effects and attempts excluded from checkpoints, never automatically replayed (§22, §31 invariant 22–23).
9. The Provider Isolation Rule stated as an explicit, unconditional normative requirement (§12, §31 invariant 24, 25).
10. Effect Classification reserved as independent, combinable dimensions, without requiring immediate implementation or enforcement (§24, §31 invariant 13, 14, 34).
11. Idempotency treated conservatively by default, with no silent retry of `NonIdempotent`/`Unknown` Effects (§23, §31 invariant 33).
12. Compensation ownership excluded from the Effect Coordinator (§25, §31 invariant 35).
13. Provider Actor supervision remains owned exclusively by existing Supervisor/Runtime machinery, with Provider Actor loss reflected in Effect coordination rather than left unresolved (§18, §21, §31 invariant 3–5).
14. Capability-to-Provider target binding, and its consequence for Provider replacement and load distribution, is explicitly disclosed rather than assumed transparent (§14, §31 invariant 11–12).
15. Provider execution is constrained to preserve Runtime forward progress, without prescribing a concurrency technology (§13, §31 invariant 16).
16. Actor/capability isolation is explicitly distinguished from host-level sandboxing, with the current reliance on Provider Actor trustworthiness disclosed (§27, §31 invariant 15).
17. Future compatibility with Workflow Runtime, Distributed Runtime, Runtime Control API, Control Centre, and Intelligence Core, with no anticipated redesign (§29).
18. No unnecessary redesign of any existing published architecture (§6, §31 invariant 26).

All eighteen criteria are satisfied by this document, each by direct citation to already-published architecture, confirmed Runtime source evidence, or a specific section above; no criterion required inventing architecture beyond formalizing the governing review's own findings and the accepted corrections applied in this revision.

## 36. References

Internal:

- GOV-003 — Governance Model
- GOV-010 — Decision Framework
- GOV-004 — Engineering Principles
- STD-001 — Documentation Standards
- ARCH-000 — Introduction
- ARCH-001 — Constitutional Architecture (§5.1, §5.2, §5.3, §6, §11)
- ARCH-002 — Runtime Architecture (§1, §6, §7, §8, §9, §10, §12, §13, §18, §19, §21, §22, §23)
- ARCH-004 — Local Actor Supervision Architecture (§4, §5, §9.1, §12, §21)
- ARCH-005 — Temporal Runtime Architecture (§4, §9, §11, §12, §14, §17, §21, §22, §23, §24)
- ARCH-006 — Runtime Actor Execution Architecture (§5, §9.1, §10, §11, §13, §14)
- ARCH-007 — Persistent Actor Architecture (§8, §9, §14, §16, §17, §18, §19, §20, §24)
- ARCH-009 — Effect Provider Architecture (v0.1.0, Draft, Superseded by this version — retained as the historical record of how §11.1–§11.4, §13.1–§13.2, §17.1, §18.1, and invariants 46–51 were originally authored and reviewed)
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- STD-031 — Engineering Work Order Lifecycle Standard (v0.2.1, Approved) — governs EWO-017, the sole demonstrated Provider evidence for this amendment
- ACT-003 — Act 3 Authorization and Charter (Approved) — names Effect Providers and provider lifecycle as in-scope Act 3 engineering, the governing context for EWO-017 and this amendment

Source evidence (verified during the governing review and independently re-verified during this correction, not restated from memory):

- `runtime/src/lib.rs` (`admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`, `execute_message_capturing`, `route_actor_execution_failure`, `delete_actor_state`, `checkpoint_actor`, `restore_actor` — confirming the shared admission pipeline, the deletion-coordination-ordering pattern, the confirmed absence of any existing Effect implementation, and the confirmed real interaction between actor execution failure and Supervisor's restart/stop/escalate decisions)
- `api/src/lib.rs` (`Actor::handle` — confirming the existing execution contract is a fully synchronous, ordinary Rust function, with no async or threading construct anywhere in this or any other workspace crate)
- `services/timer/src/{lib,internal}.rs` (`register`, `cancel`, `cancel_all_for_actor`, `query_due` — confirming both cancellation operations' distinct fallibility)
- `core/capability-authority`, `core/lifecycle-guardian`, `core/actor-host`, `services/persistence`, `services/supervisor` (confirming each existing component's own current responsibility boundary, unmodified by this document)
- The complete, approved **Effect Runtime Architecture Review** (this engineering effort), concluding `EFFECT RUNTIME ARCHITECTURE APPROVED FOR FORMAL DESIGN` — the primary analytical basis for this document; every recommendation in that review is formalized here without reopening any of its conclusions.
- The complete **Independent Review of ARCH-008 Effect Runtime Architecture** (this engineering effort), concluding `ARCH-008 EFFECT RUNTIME ARCHITECTURE REQUIRES CORRECTION` — the governing input for every correction applied in this revision (§37).
- The complete **SynapseOS Act II — Idempotency Architecture Investigation** (this engineering effort), concluding `IDEMPOTENCY ARCHITECTURE INVESTIGATION COMPLETE — READY FOR ARCHITECTURE AMENDMENT` — the sole analytical basis for §23.1–§23.5 (0.4.0).
- `runtime/src/lib.rs` (`request_effect`, confirmed at the current published baseline: `correlation: Some(effect.0.clone())` on every dispatched Effect-request message) — the direct source evidence for §23.4's disclosure that the existing admission pipeline already carries a stable, provider-visible correlation token.
- The complete **SynapseOS — ARCH-009 Architecture Investigation: Retry Architecture** (this engineering effort), concluding `ARCH-009 ARCHITECTURE INVESTIGATION COMPLETE — READY FOR ARCHITECTURE AMENDMENT` and recommending an ARCH-008 amendment rather than a new document — the sole analytical basis for §19.1–§19.4 and invariant 45 (0.4.3).
- `services/effect-coordinator/src/{lib.rs,internal.rs}` (`record_retry_scheduled`, the `RetryScheduled`/`Requested` transition guard, and the `EffectCoordinatorImpl` internal state — confirmed, at the current published baseline `4256b4434447fb9ab43d0d901a5baf8476c024e3`, to implement only the bookkeeping transition itself, with no retry-decision, retry-eligibility, or retry-limit mechanism wired; `runtime/src/lib.rs`'s own `retry_scheduling_remains_unimplemented_by_this_milestone` test independently confirms this is a disclosed, deliberate gap, not drift between this document and the implementation it governs).
- **ARCH-009 — Effect Provider Architecture** (this engineering effort, v0.1.0, Draft), concluding, in its own §3, that its genuinely new content should be integrated into ARCH-008 rather than retained as a standalone document — the primary authoring basis for §11.1–§11.4, §13.1–§13.2, §17.1, and §18.1 (0.5.0).
- The complete **Independent Architecture Review of ARCH-009 Effect Provider Architecture** (AR-009, this engineering effort), concluding `ARCH-009 ARCHITECTURE REVIEW COMPLETE — READY FOR FOUNDER APPROVAL` — zero Critical, zero Major findings, two Minor findings (a citation misattribution and a structural-completeness gap, both corrected in this integration rather than carried forward), and a recommendation of Option B (integrate into ARCH-008) over Option A (retain standalone) — the sole analytical basis for the integration decision and for invariants 46–51 (0.5.0).
- `common/src/lib.rs` (`AuditEvent`, exactly four fields — `event_type`, `actor`, `capability`, `message` — confirming §17.1's disclosed gap directly; `EFFECT_PROVIDER_RESULT_FAILED`, confirming §18.1's reserved provider-error-model value exists and is documented); `runtime/src/lib.rs` (`execute_message_capturing`'s additive dispatch-outcome wiring, confirming §18.1 does not alter any pre-existing `Ok`/`Err` path); `services/http-provider/` in full (confirming EWO-017's own demonstrated Provider Classification — Stateless, Remote, Ephemeral — and its own disclosed concurrency realization, both cited in §11.3/§13.2 without restating the crate's own README verbatim) — all read directly at the current published baseline `397dded110bde75bdbcfcb4389c703d6fa7077dc`.

## 37. Change History

| Version | Date | Author | Description |
|---------|------|--------|--------------|
| 0.1.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Initial Draft. Establishes the Effect Runtime Architecture: ownership model (Runtime orchestrates, Effect Coordinator tracks, Provider Actors execute), the Effect Coordinator as a new, narrow, non-Trusted-Core replaceable service, the Provider Actor model, the mandatory Provider-to-Provider isolation rule, Effect request/result flow through the existing, unmodified admission pipeline, capability architecture (`effect.<domain>.<operation>`), Effect identity, lifecycle (with single-terminal-outcome and late-signal-discard discipline), audit architecture, failure semantics, retry and timeout architectures (policy/mechanism separated, reusing the existing Timer Service), cancellation architecture (extending the existing deletion-coordination-ordering pattern), determinism and persistence (in-flight Effects as execution state, never domain state — the one genuinely novel synthesis in this document), idempotency, the reserved (unimplemented) Effect Classification model, the compensation boundary, resource governance, security boundaries, Runtime state and shutdown behavior, future compatibility, explicit non-goals, twenty-seven architectural invariants, risks and mitigations, deferred decisions, consequences, and fourteen approval criteria. Completes the deferral recorded in ARCH-002 §23 ("Provider Architecture") and formalizes the convergent precedent independently established by ARCH-002 §23, ARCH-004 §21, ARCH-005 §4/§21/§22, and ARCH-006 §11/§14. Codifies the Effect Runtime Architecture Review's own conclusions without introducing new design beyond what expressing them normatively as architecture requires. |
| 0.2.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Targeted correction pass applying the accepted findings of the Independent Review of ARCH-008 Effect Runtime Architecture (verdict: REQUIRES CORRECTION, six MAJOR findings, no BLOCKER). No ownership decision, Trusted Core boundary, or reused-mechanism commitment from 0.1.0 was reopened; every change below is additive clarification, disclosure, or extension. (1) **Provider Actor lifecycle-loss integration** (§10, §16, §18, §21) — added `ProviderLost` as a named terminal attempt outcome; required Runtime to inform the Effect Coordinator when a Provider Actor instance handling an active attempt is restarted, stopped, terminated, replaced, or otherwise made unavailable by ordinary Supervisor/Runtime supervision, so the attempt is driven to a truthful terminal outcome rather than left indefinitely `Dispatched`/`Executing`; made explicit that a Provider Actor restart must not silently re-execute a lost attempt; added an explicit Effect Coordinator MUST-NOT against ever supervising Provider Actors itself. (2) **Capability-to-Provider target binding, disclosed** (§14) — stated explicitly that an Effect capability targets one specific Provider Actor `ActorId` under the existing, unmodified Capability model, with the resulting consequence for Provider Actor replacement, load distribution, and future provider-discovery/rebinding mechanisms named as deferred rather than assumed transparent. (3) **Effect Classification corrected to orthogonal, combinable dimensions** (§24) — replaced the single-list presentation of `Pure`/`Read`/`Write`/`Transactional`/`Streaming`/`LongRunning` with an explicit statement that they are independent, combinable metadata traits grouped by underlying concern (externally observable effect vs. transactional behavior vs. result shape vs. execution duration), with worked combination examples, closing the risk of an incorrectly modeled closed enum. (4) **Actor isolation distinguished from host-level sandboxing** (§27) — added an explicit disclosure that capability authorization and actor-state isolation govern message admission and dispatch only, do not themselves provide process/OS-level resource sandboxing, and that Provider Actor implementations remain trusted to comply with their own granted scope absent future, separately designed host-level enforcement. (5) **Effect ID / Effect Attempt ID introduced as a first-class distinction** (§7, §15, §16, §17, §19, §20, §21, §22) — the Effect ID now denotes the stable, logical request; the Effect Attempt ID denotes one concrete execution attempt, minted fresh on every retry; every provider result, timeout, cancellation, retry trigger, and late signal is now explicitly attributed to a specific attempt, closing the risk of a late signal from an earlier attempt overwriting a later attempt's own outcome. (6) **Runtime forward-progress constraint added; misleading latency claim corrected** (§13) — removed the 0.1.0 claim that external I/O latency is "invisible to the Runtime's own deterministic step engine" (contradicted by confirmed source evidence that `Actor::handle()` is a fully synchronous function with no async/threading construct anywhere in the workspace) and replaced it with an explicit, implementation-independent constitutional constraint: Provider execution MUST NOT prevent the Runtime from making bounded forward progress for unrelated actors, timers, Effect coordination, and lifecycle processing; no concurrency technology is prescribed. (7) **Authorized/Dispatched truthful-collapse permitted** (§16, §17) — extended the collapse allowance §17 already granted `Executing` to `Authorized`/`Dispatched`, so an implementation is not forced to fabricate a second, separately observable authorization moment the existing, reused admission pipeline's own atomicity does not truthfully provide, without permitting a second authorization mechanism of any kind. (8) **Minor corrections** — corrected the citation "ARCH-005 §12" (Timer Lifecycle) to "ARCH-005 §9" (Component Placement) in §10; disclosed, in §20, that the Timer Service's two cancellation operations have different fallibility, and that a cancellation/firing race is expected and harmless, resolved by the existing late-signal discipline, never a new failure category; cross-referenced durable deletion's coordination-resolution language in §21 to the same external-outcome-uncertainty principle already applying to ordinary cancellation, stated once, generally, as "Runtime terminal coordination state is not proof of external-world state"; clarified in §11/§12 that one Provider Actor may perform multiple internal steps only as one declared logical operation, and that independently invocable external operations remain distinct Effects regardless of packaging. Renumbered Architectural Invariants (§31) from twenty-seven to thirty-nine to accommodate the above without removing any prior invariant; updated Risks and Mitigations (§32), Deferred Decisions (§33), Consequences (§34), Approval Criteria (§35, fourteen to eighteen), and References (§36) accordingly; verified every internal section and invariant cross-reference after renumbering. No new `Unknown`/`Indeterminate`/`OutcomeUncertain` lifecycle state was introduced — the independent review's own conclusion that none is presently required was preserved. |
| 0.3.0 | 2026-07-27 | Denver Jacobs (AI-assisted) | Final, narrowly-scoped correction pass applying the two accepted findings of the Final Independent Review of Corrected ARCH-008 Effect Runtime Architecture (verdict: REQUIRES FURTHER CORRECTION, two MAJOR findings, no BLOCKER, both self-contained inconsistencies introduced by the 0.2.0 correction itself). No architecture outside these two findings was touched; the architectural freeze governing this task's own scope was observed throughout. (1) **`RetryScheduled` restored as an Effect-level state** (§16, now split into §16.1 Effect-level lifecycle, §16.2 Attempt-level lifecycle, §16.3 Composition; §7, §19, §31 invariant 41, new) — resolves the contradiction the 0.2.0 correction introduced, in which `effect.retry_scheduled` was required as a mandatory audit fact (§17) with no corresponding lifecycle state anywhere in §16 to describe. `RetryScheduled` is explicitly, exclusively an Effect-level, transitional state describing the interval between one attempt's own terminal outcome and a new attempt's own Effect Attempt ID being minted; it never reopens, extends, or reuses an already-terminated attempt, and no attempt exists while it holds. (2) **`Denied` corrected to an Effect-level outcome, never an Effect Attempt outcome** (§7, §16, §17, §18, §31 invariant 40, new) — resolves the contradiction between §15's statement that an Effect Attempt ID is minted "each time an attempt is dispatched" and the 0.2.0 text's listing of `Denied` — which occurs at `Requested -> Denied`, strictly before any dispatch — among "terminal outcomes for one attempt." `Denied` is now explicitly stated to mean no Effect Attempt, and no Effect Attempt ID, was ever created, whether the failed authorization check was the Effect's first or occurred on a later retry's own fresh authorization attempt; `effect.denied` is now explicitly attributed to the Effect ID alone (§17), and the Attempt-level lifecycle diagram and terminal-outcomes list no longer include it. (3) **Lifecycle diagram corrected to reflect the existing `Executing`-collapse allowance** (§16.2) — added the `Authorized -> Dispatched -> {Completed \| Failed}` path alongside the existing `Authorized -> Dispatched -> Executing -> {Completed \| Failed}` path, so the diagram no longer contradicts the prose's own, unchanged permission to collapse `Executing` into `Dispatched` where not truthfully observable. Two new invariants were appended (§31, invariants 40–41) rather than interleaved into the existing numbering, to avoid a second full renumbering sweep for a narrowly-scoped correction; two Risks and Mitigations rows were added (§32) for the two corrected defects. No lifecycle state was silently added or removed: `RetryScheduled` is restored, not newly invented, and `Denied` is relocated, not removed. Runtime ownership, Effect Coordinator ownership, the Provider Actor model, Provider isolation, capability architecture, the forward-progress requirement, Effect Classification, the Effect ID model, persistence rules, the external-uncertainty model, the security model, the durable-deletion model, Runtime orchestration, and Provider supervision integration are all unchanged from 0.2.0. |
| 0.3.1 | 2026-07-27 | Denver Jacobs (AI-assisted) | Governance-preparation and metadata correction only — **no technical or architectural change of any kind**; every ownership decision, lifecycle rule, and invariant from 0.3.0 is unchanged. Prepared ahead of formal review by the Approval Authority (GOV-003 §3.5), following a governance readiness review confirming ARCH-008 is technically complete, matches the now-published implementation, and that the sole remaining gap is the approval act itself. Corrected two stale frontmatter fields that predated the implementation this document governs: `reports` (was `None`; now cites ER-011 and ER-012) and `source_artifacts` (was `None`; now cites the published `synapse-runtime` commit implementing the Effect Runtime Foundation and Provider Actor Integration milestones). Added, below the existing Approval Status table (§38), a prepared-but-unrecorded Approval Evidence Record placeholder structured per STD-001 §31.1–§31.3, for the Approval Authority to complete — this addition does not itself constitute approval evidence and does not alter the Approval Status table's own existing, accurate "Pending" entries, consistent with STD-001 §31.5 (approval evidence, once genuinely recorded, is content-non-mutating with respect to this artifact; reconciling tracked metadata is a separate, later, distinct act). `status` remains `Draft`; no field asserting technical review completion or approval was altered, since neither has occurred. |
| 0.3.2 | 2026-07-27 | Denver Jacobs (Founder) | Governance disposition recorded — **no technical or architectural change of any kind**; every ownership decision, lifecycle rule, and invariant from 0.3.0/0.3.1 is unchanged. Records the Approval Authority's decision on the 0.3.1 governance approval package: `status` transitions from `Draft` to **`Approved`**; the §38 Approval Status table is completed (Technical Review and Approval Authority both recorded against Denver Jacobs, Founder, exercising the interim Class B approval default under GOV-003 §3.2 while the Chief Architect role remains vacant, with the resulting self-review/self-approval conflict explicitly disclosed rather than concealed, per GOV-003 §3.5); the Approval Evidence Record (§38, STD-001 §31.1–§31.3) is completed with disposition, approver identity, authority citation, effective date, and rationale — the three artifact-identity fields (§31.1) are deliberately left as reproducible derivations rather than fabricated values, since a commit cannot cite its own SHA from within its own tree; the exact values are recorded in the governing publication task's own Final Report. This approval disposes of ARCH-008 only, not of ARCH-001–ARCH-007, GOV-003, GOV-010, or STD-001, each of which remains independently Draft/Pending. |
| 0.4.0 | 2026-07-28 | Denver Jacobs (AI-assisted) | Architecture Amendment completing the deferral §23 itself already named ("concrete metadata types, storage, and enforcement mechanics are deferred to implementation"), on the sole analytical basis of the completed SynapseOS Act II Idempotency Architecture Investigation. **MINOR, additive-only change (STD-001 §13): no existing ownership decision, lifecycle rule, Trusted Core boundary, capability model, or prior invariant (1–41) is reopened, redefined, or reinterpreted.** Adds five new subsections to §23 — §23.1 Registration Model (idempotency is declared at `(Provider ActorId, operation)` granularity, never per Effect request or Attempt, completing rather than changing the document's own existing "provider-declared... metadata," never "per-request metadata," framing); §23.2 Runtime Ownership (Effect Coordinator bookkeeping only, Runtime-session-scoped, never actor domain state, never checkpointed — a direct, unmodified application of the general rule §22 already establishes); §23.3 Failure and Default Semantics (no declaration ≡ `Unknown`; lost on Runtime restart, not on Provider Actor restart within a session; re-registration replaces, never errors); §23.4 Provider Correlation (formalizes, as a disclosed existing capability rather than a new mechanism, that the `Message.correlation` value the admission pipeline already carries on every Effect dispatch — confirmed directly against the published `synapse-runtime` implementation, `request_effect`'s own `correlation: Some(effect.0.clone())` — MAY be used by a Provider as its own external-system request-deduplication token, explicitly distinguished from the idempotency declaration itself); §23.5 Capability Model (no new capability, operation string, or authorization gate — registration authority is already fully implied by the existing `effect.<domain>.<operation>` grant, §14). Adds invariants 42–44 (§31), appended without renumbering, on the identical basis the 0.3.0 correction already established for narrowly-scoped additions. Narrows one line of §33 (Deferred Decisions) to reflect that registration granularity, ownership, and session-scoping are now specified, while the concrete representation remains, as before, implementation-phase. Updates the stale `reports`/`source_artifacts` frontmatter fields to cite ER-013/ER-014 and the current published Runtime baseline (`3626a73288f31c1a97cdf4d1c8bca181d12c7d9b`), on the identical basis the 0.3.1 correction already did for ER-011/ER-012. **`status` reverts from `Approved` to `Draft`** for this version specifically — per GOV-013 §11, amending an already-approved document requires its own evidenced Approval Authority act regardless of scope; the existing 0.3.2 Approval Status table entries and Approval Evidence Record are left completely unmodified below as the truthful, permanent record of what was approved and when, rather than rewritten to imply this new content was covered by that same act. |
| 0.4.1 | 2026-07-28 | Denver Jacobs (AI-assisted) | Corrective revision applying the three findings of the Independent Architecture Review of the 0.4.0 amendment (verdict: `ARCH-008 IDEMPOTENCY REVIEW REQUIRES AMENDMENT`; two MAJOR findings, one MINOR finding, no BLOCKER). **PATCH-level, wording-only correction (STD-001 §13): no normative conclusion reached by 0.4.0 is reversed; every change below clarifies or reconciles existing text.** (1) **Finding A (§23.4 vs. §15)** — added an explicit sentence distinguishing the general rule (§15: Effect correlation information is "supplied by the requesting actor," continuing to govern ordinary messages and any actor-supplied correlation an Effect request's own payload separately carries) from the specific, narrower fact §23.4 states (Runtime itself populates the `Message.correlation` *field* of an Effect-request message with the Effect ID, the one field such a message's own construction does not leave to the requesting actor) — the two statements no longer silently coexist without acknowledging their different scope. (2) **Finding B (§23.1/§23.5 capability-holder language)** — reworded §23.1 ("A Provider declares its classification" → "A classification is registered..."; the section now states explicitly that "Provider-declared" denotes whose operation a classification describes, not which component technically performs registration) and §23.5 (removed the factually incorrect "already holding" branch and rebuilt the justification on the accurate, already-established §14 model: a capability is bound to the requesting actor and targets the Provider, which is never itself a capability holder — the registration act requires no new gate because the operation is already, independently gated by that existing structure, not because the Provider "holds" anything). (3) **Finding C (§38 table clarity)** — added an inline scope qualifier directly to the Author, Technical Review, and Approval Authority cells of the §38 table itself ("content through v0.3.2 only" / "v0.4.0/v0.4.1's own new content is not covered by this disposition"), so the table is self-sufficiently accurate even read in isolation from the disclosure paragraph above it. No architectural redesign, no new requirement, and no change outside these three findings' own scope — Retry Architecture, Effect lifecycle, the capability model's own rules, and the persistence/session-scoping model are all unchanged from 0.4.0. |
| 0.4.2 | 2026-07-28 | Denver Jacobs (Founder) | Governance disposition recorded — **no technical or architectural change of any kind**; every ownership decision, lifecycle rule, and invariant from 0.4.0/0.4.1 is unchanged. Records the Approval Authority's decision on the completed Idempotency Metadata Amendment review-correction-re-review cycle (Independent Architecture Review: `ARCH-008 IDEMPOTENCY REVIEW REQUIRES AMENDMENT`; Architecture Amendment Revision: 0.4.0 → 0.4.1, applying exactly the three named findings; Independent Architecture Re-Review: `ARCH-008 IDEMPOTENCY RE-REVIEW COMPLETE — READY FOR PUBLICATION`): `status` transitions from `Draft` back to **`Approved`**; §38 is reorganized into §38.1 (the existing, byte-unmodified v0.3.2 disposition) and a new §38.2 (this disposition), each with its own complete Approval Status table and Approval Evidence Record, per STD-001 §31.5's content-non-mutating model — neither section rewrites the other. This approval covers §23.1–§23.5 and invariants 42–44 specifically; it does not reopen, and is not represented as reopening, the §38.1 disposition of content through v0.3.2. |
| 0.4.3 | 2026-07-28 | Denver Jacobs (AI-assisted) | Architecture Amendment completing the deferral §19 itself already named, on the sole analytical basis of the completed SynapseOS ARCH-009 Architecture Investigation: Retry Architecture (concluding `ARCH-009 ARCHITECTURE INVESTIGATION COMPLETE — READY FOR ARCHITECTURE AMENDMENT`, and recommending this exact vehicle — an ARCH-008 amendment, not a new document — to preserve ARCH-008 as the single constitutional owner of the entire Effect lifecycle). **MINOR, additive-only change (STD-001 §13): no existing ownership decision, lifecycle rule, Trusted Core boundary, capability model, idempotency rule, or prior invariant (1–44) is reopened, redefined, or reinterpreted.** Adds four new subsections to §19 — §19.1 Retry Eligibility (names explicitly, for the first time, the three retry-eligible attempt-level terminal outcomes — `Failed`, `TimedOut`, `ProviderLost` — and confirms `Completed`, `Cancelled`, and the Effect-level-only `Denied` are never retry-eligible, completing rather than changing what §16.2's own per-outcome annotations already implied); §19.2 Retry Authority (exhaustively partitions retry responsibility across Provider Actor, requesting actor, Runtime, and Effect Coordinator, none of which may supply another's input or perform another's decision; resolves that retry intent is requesting-actor-supplied once, at original request time, never re-solicited per attempt, since the Effect Coordinator is bookkeeping-only and non-interactive, §10); §19.3 Idempotency-Class Retry Permission (completes the interaction §19's own existing negative prohibition only partially stated, with an explicit three-row permission table: `Idempotent` — permitted, subject to eligibility and actor intent; `NonIdempotent` — prohibited by default, permitted only by the actor's own explicit, distinct risk-acceptance for that specific Effect; `Unknown` — identical treatment to `NonIdempotent`, per §23's existing conservative default); §19.4 Retry Limit Ownership (resolves the "requesting actor or a policy-bearing capability constraint" disjunction §19's own original text left unresolved, by direct analogy to §14's already-approved attenuation-only capability model: a capability-declared limit, where present, is a hard ceiling; an actor-supplied limit may narrow but never widen it; absent a capability ceiling, the actor's own limit alone governs — introducing no numeric default, only ownership and precedence). Adds invariant 45 (§31, deterministic Retry Decisions — a pure function of Effect Coordinator tracked state and the §19.1–§19.4 inputs, never of wall-clock time, process identity, or thread scheduling), appended without renumbering, on the identical basis the 0.3.0/0.4.0 corrections already established for narrowly-scoped additions. Narrows one line of §33 (Deferred Decisions) to reflect that retry eligibility, role partition, idempotency-class permission, and limit-ownership precedence are now specified, while the concrete numeric values and backoff/jitter algorithms remain, as before, implementation-phase. Updates the stale `reports`/`source_artifacts` frontmatter fields to cite ER-015 and the current published Runtime baseline (`4256b4434447fb9ab43d0d901a5baf8476c024e3`), on the identical basis the 0.3.1/0.4.0 corrections already did for their own predecessors. **`status` reverts from `Approved` to `Draft`** for this version specifically — per GOV-013 §11, amending an already-approved document requires its own evidenced Approval Authority act regardless of scope; the existing 0.4.2 Approval Status table entries and Approval Evidence Record (§38.1, §38.2) are left completely unmodified below as the truthful, permanent record of what was approved and when, rather than rewritten to imply this new content was covered by that same act. **Pre-publication correction, applied before this version was ever committed:** an Independent Architecture Review of the initial 0.4.3 draft found two MAJOR and two MINOR findings — (1) §19.2's "Provider Actor" row originally read "Declares operation-level idempotency classification," reintroducing language the 0.4.1 correction's own Finding B had already replaced; reworded to state the classification describes the Provider Actor's own operation without attributing the registration act to it, per §23.1's own disclosure; (2) §19.2 originally claimed to "generalize" §9's ownership-table entry ("retry intent after restoration") without editing that table, contradicting §9's own "single authoritative statement of ownership" claim; resolved by editing §9's table row itself to state the general case directly (restoration named there as one specific case of it), and rewording §19.2 to reflect §9 as already, directly authoritative rather than claiming to generalize it; (3) invariant 45 cited only §19.2–§19.4 as its determinism inputs, omitting §19.1 (Retry Eligibility) despite §19.2's own text listing it as one of the four combined inputs; corrected to cite §19.1–§19.4; (4) §19.4 used "authorize" for the Effect Coordinator's own bookkeeping-level retry-limit enforcement, risking confusion with Capability Authority's reserved use of that term; reworded to "permit," with an explicit disclaimer distinguishing it from capability authorization. Since this version was never committed or published prior to correction, these four fixes are folded directly into this same 0.4.3 entry rather than recorded as a separate version, consistent with GOV-013's own stage separation between amendment and its own subsequent, still-pre-publication review. No Runtime code and no EWO were touched by either the original amendment or this correction; the correction additionally touched §9's ownership table (Correction 2, above) beyond the §19/§31/§33/§36/§37/§1 scope the original amendment confined itself to — disclosed here rather than concealed, since resolving the §9/§19.2 inconsistency required it. **Publication (this same date):** a subsequent Independent Architecture Re-Review confirmed all four findings resolved with no regression (`ARCH-008 v0.4.3 INDEPENDENT ARCHITECTURE RE-REVIEW COMPLETE — READY FOR FOUNDER APPROVAL`); Founder Approval was then granted (`FOUNDER APPROVAL GRANTED — ARCH-008 v0.4.3 APPROVED FOR PUBLICATION`). `status` now transitions from `Draft` to **`Approved`** for this version; §38 gains a new §38.3 disposition recording this approval, on the identical content-non-mutating basis (STD-001 §31.5) §38.2 already established for the idempotency amendment — §38.1 and §38.2 remain completely unmodified. |
| 0.5.0 | 2026-07-29 | Denver Jacobs (AI-assisted) | Architecture Amendment integrating the genuinely new architectural content of ARCH-009 — Effect Provider Architecture (v0.1.0, Draft), a sibling document authored following EWO-017 (Reference Effect Provider Framework, Runtime commit `397dded`). This amendment's own governance shape differs from every prior one: the content under review (ARCH-009) was, for the first time, a separately authored and published sibling document rather than a pre-publication draft of this document itself; Independent Architecture Review AR-009 evaluated it there, concluding `ARCH-009 ARCHITECTURE REVIEW COMPLETE — READY FOR FOUNDER APPROVAL` (zero Critical, zero Major findings; two Minor findings) and recommending Option B — integrate into ARCH-008 — over Option A — retain ARCH-009 as a standalone document — a recommendation ARCH-009's own §3 had itself already anticipated by name, citing this document's own prior, identical precedent ("ARCH-009 Architecture Investigation: Retry Architecture," 0.4.3 entry above) for a narrower scope. The Founder accepted AR-009 and Option B in full. **MINOR-tier, additive-only change (STD-001 §13): no existing ownership decision, lifecycle rule, Trusted Core boundary, capability model, retry/idempotency rule, or prior invariant (1–45) is reopened, redefined, or reinterpreted.** Adds: §11.1 Provider Instance Lifecycle vs. Effect Attempt Lifecycle (disentangles the existing `ActorState` instance lifecycle from the existing Effect Attempt outcome, ARCH-002 §15/§16.2 — a clarification, not a new state); §11.2 Provider Registration and Discovery (states normatively, for the first time, the existing `ActorId`-direct lookup-through-Actor-Directory model this document previously stated only negatively as "no registry is introduced"); §11.3 Provider Classification (Stateless/Stateful/Streaming/Long-running/Local/Remote/Persistent/Ephemeral — an axis orthogonal to, and independent of, §24's existing Effect Classification; grants no authority); §11.4 Provider Extension Rules (codifies EWO-017's own demonstrated pattern — crate structure, README disclosure, required test categories, engineering evidence — as the process for adding a future Provider); §13.1 Result Model (generalizes EWO-017's typed-request/typed-result pattern, non-binding, narrowing no existing Streaming deferral); §13.2 Concurrency Model (organizes, without deciding, the design space §13's own forward-progress constraint already leaves open — blocking-synchronous, cooperative non-blocking polling already structurally supported by the existing `Executing` attempt state, and a future thread/async mechanism); §17.1 (discloses, rather than silently assumes resolved, that the current `AuditEvent` carries no timestamp or correlation-identifier field of its own); §18.1 (generalizes EWO-017's own additive provider-error-model convention — a reserved message-type value causing `Failed` rather than `Completed` — as the canonical pattern for a Provider to report an ordinary, retry-eligible failure without misclassification as `ProviderLost`; the pre-existing `Ok`/`Err` paths are completely unaffected). Adds invariants 46–51 (§31), appended without renumbering, on the identical basis every prior narrowly-scoped addition already established. Adds three Risks and Mitigations rows (§32) and narrows five lines of Deferred Decisions (§33) to reflect what is now specified (the `ActorId`-direct registration model; Provider Classification's own independence from Effect Classification) versus what remains deferred (a future discovery/versioning mechanism; concrete streaming wire representation; the concrete concurrency mechanism; `AuditEvent` field additions). **Both Minor findings AR-009 identified against the standalone ARCH-009 document are corrected in this same integration, not carried forward:** (1) a citation in ARCH-009 §16 misattributed a quotation to `README.md` that in fact appeared in `src/lib.rs`'s own doc comment — this amendment's own §11.3 omits the specific illustrative quotation entirely, citing only the crate's own disclosed classification generally, rather than repeating an unverified attribution; (2) ARCH-009's own lack of labeled Non-Goals/Design-Principles/Ownership sections (GOV-013 §6.8's own named ARCH-document elements) is resolved structurally by this integration itself — this content now lives inside a document that already has §4 (Non-Goals), §8 (Architectural Principles), and §9 (Ownership Model), with no separate citation section required. ARCH-009 itself (v0.1.0) is marked `Superseded` by this version in its own frontmatter and retained, unmodified in substance, as the truthful historical record (STD-001 §4/§28); it is not deleted and its content is not restated here where a citation suffices. A disclosed, unresolved, out-of-scope gap, named rather than concealed: `services/http-provider/README.md` forward-references an "ER-018" that does not exist in this repository as of this amendment — EWO-017 was never followed by a published EWO-017/ER-017-equivalent work-order/report pair in `synapse-docs`, only a Runtime commit; this amendment does not create one, since doing so is outside an Architecture Amendment's own authority (STD-031, EWO/ER creation), and Runtime itself was read-only throughout this task. Updates the stale `source_artifacts`/`engineering` frontmatter fields to cite the current published Runtime baseline (`397dded110bde75bdbcfcb4389c703d6fa7077dc`) and EWO-017 directly. **`status` transitions directly from `Approved` to `Approved`** — recorded as `Draft` only internally during this task's own drafting, never separately committed or published in that state, exactly as the 0.4.3 cycle already established as acceptable when no intermediate state is ever published; §38 gains a new §38.4 disposition recording this approval — §38.1, §38.2, and §38.3 remain completely unmodified. |

## 38. Approval Status

This document has received two, separately evidenced Approval Authority dispositions, covering different content added at different times. Per STD-001 §31.5, neither disposition mutates the other; both are recorded in full, in the order they occurred.

### 38.1 Disposition of Content Through v0.3.2 (2026-07-27)

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-27 |
| Technical Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | Completed (self-review, disclosed per GOV-003 §3.5) | 2026-07-27 |
| Approval Authority | Denver Jacobs, Founder, interim Class B approval default (GOV-003 §3.2 — Chief Architect role vacant) | **Approved** | 2026-07-27 |

**Approval Evidence Record (STD-001 §31).** Recorded per STD-001 §31.1–§31.3. Consistent with STD-001 §31.5, completing this record does not itself require, and was not the sole justification for, the `status`/table changes above; the Approval Authority's disposition, recorded here, is what those changes reflect.

| STD-001 §31 field | Value |
|---|---|
| Document ID | ARCH-008 |
| Repository path | `architecture/ARCH-008-Effect-Runtime-Architecture.md` |
| Version | 0.3.2 |
| Artifact revision identifier (§31.1) | The `synapse-docs` commit whose subject is "Approve Effect Runtime architecture," publishing this exact Approval Status content. A commit cannot cite its own SHA from within its own tree; the exact commit SHA, tree SHA, and blob ID are recorded in that commit's governing publication task's own Final Report and are independently, reproducibly verifiable via `git log --follow -- architecture/ARCH-008-Effect-Runtime-Architecture.md` and `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md` against this exact path and byte content — satisfying §31.1's committed-content requirement without fabricating a value here |
| Content fingerprint (§31.1) | Reproducible as `git hash-object architecture/ARCH-008-Effect-Runtime-Architecture.md` at the commit identified above, rather than restated here, for the identical reason given above |
| Git blob ID (§31.1) | Reproducible as `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md`, for the identical reason given above |
| Disposition (§31.2) | Approved |
| Disposition type (§31.2) | Architecture approval (Class B, GOV-010 §4–§5) |
| Approver identity (§31.2) | Denver Jacobs |
| Authority citation (§31.2) | GOV-003 §3.2 — Chief Architect role vacant; Class B (architectural) approval authority defaults, on an interim basis, to the Founder |
| Effective date (§31.2) | 2026-07-27 |
| Review evidence actually available (§31.3) | Extensive AI-assisted analytical and adversarial review was conducted across this engineering effort (architecture authoring, two correction cycles, an approval-confirmation review, and a governance readiness review; see §36 References) — disclosed as AI-generated supporting evidence only (GOV-010 §9), not itself independent human review. The Approval Authority's own direct review of this content and its supporting evidence, disclosed as self-review above, is the human review basis for this disposition |
| Independent-review status (§31.3) | No independent *third-party* human review occurred — disclosed, not concealed. The interim Founder default (GOV-003 §3.2) applies precisely because the Chief Architect role is vacant and no independent reviewer was available |
| Self-approval / conflict-of-interest disclosure (§31.3) | Disclosed: Denver Jacobs acted as both the (disclosed) Technical Reviewer and the Approval Authority for this disposition, under the interim Founder default (GOV-003 §3.2); this is the same conflict-of-interest pattern GOV-003 §3.5 explicitly anticipates and requires be disclosed rather than concealed, not one it prohibits |
| Known limitations (§31.3) | See §32 (Risks and Mitigations) and §33 (Deferred Decisions). Separately: this approval disposes of ARCH-008 only; it does not, and is not represented as, a governance ratification of ARCH-001–ARCH-007, GOV-003, GOV-010, or STD-001, each of which remains independently `Draft`/`Pending` in its own right |
| Unresolved issues (§31.3) | None identified against ARCH-008 itself beyond the deferrals already disclosed in §33 |
| Rationale (§31.3) | Approved on the basis that the architecture is technically complete, internally consistent, and accurately matches the published implementation (EWO-001/ER-011, EWO-002/ER-012) across three independent review-and-correction cycles with no remaining BLOCKER or MAJOR finding, per the governing Governance Readiness Review |

### 38.2 Disposition of Content Added by v0.4.0/v0.4.1 — the Idempotency Metadata Amendment (recorded as v0.4.2, 2026-07-28)

Per GOV-013 §11, amending an already-approved document requires its own evidenced Approval Authority act "regardless of how minor the amendment." The §23.1–§23.5 completion and invariants 42–44 (added by 0.4.0, corrected by 0.4.1) accordingly received their own, separate review-and-approval sequence, distinct from — and not covered by — the 38.1 disposition above.

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-28 |
| Independent Architecture Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `ARCH-008 IDEMPOTENCY REVIEW REQUIRES AMENDMENT` (two MAJOR findings, one MINOR finding, no BLOCKER) | 2026-07-28 |
| Architecture Amendment Revision | — | Applied exactly the three named findings (0.4.0 → 0.4.1); no unrelated content changed | 2026-07-28 |
| Independent Architecture Re-Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `ARCH-008 IDEMPOTENCY RE-REVIEW COMPLETE — READY FOR PUBLICATION` (all three findings independently confirmed resolved; no regression) | 2026-07-28 |
| Approval Authority | Denver Jacobs, Founder, interim Class B approval default (GOV-003 §3.2 — Chief Architect role vacant) | **Approved** | 2026-07-28 |

**Approval Evidence Record (STD-001 §31).**

| STD-001 §31 field | Value |
|---|---|
| Document ID | ARCH-008 |
| Repository path | `architecture/ARCH-008-Effect-Runtime-Architecture.md` |
| Version | 0.4.2 |
| Artifact revision identifier (§31.1) | The `synapse-docs` commit publishing this exact 0.4.2 Approval Status content, once created — a commit cannot cite its own SHA from within its own tree; the exact commit SHA, tree SHA, and blob ID are independently, reproducibly verifiable via `git log --follow -- architecture/ARCH-008-Effect-Runtime-Architecture.md` and `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md` against this exact path and byte content once published, satisfying §31.1's committed-content requirement without fabricating a value here |
| Content fingerprint (§31.1) | Reproducible as `git hash-object architecture/ARCH-008-Effect-Runtime-Architecture.md` at the commit identified above, once published, for the identical reason given above |
| Git blob ID (§31.1) | Reproducible as `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md`, once published, for the identical reason given above |
| Disposition (§31.2) | Approved |
| Disposition type (§31.2) | Architecture amendment approval (Class B, GOV-010 §4–§5) |
| Approver identity (§31.2) | Denver Jacobs |
| Authority citation (§31.2) | GOV-003 §3.2 — Chief Architect role vacant; Class B (architectural) approval authority defaults, on an interim basis, to the Founder |
| Effective date (§31.2) | 2026-07-28 |
| Review evidence actually available (§31.3) | The complete Idempotency Architecture Investigation, Architecture Amendment, Independent Architecture Review, Architecture Amendment Revision, and Independent Architecture Re-Review (§36 References) — disclosed as AI-generated supporting evidence only (GOV-010 §9), not itself independent human review. The Approval Authority's own direct review of this content and its supporting evidence, disclosed as self-review above, is the human review basis for this disposition |
| Independent-review status (§31.3) | No independent *third-party* human review occurred — disclosed, not concealed, on the identical basis as §38.1. The interim Founder default (GOV-003 §3.2) applies for the identical reason |
| Self-approval / conflict-of-interest disclosure (§31.3) | Disclosed: Denver Jacobs acted as both the (disclosed) Technical/Architecture Reviewer and the Approval Authority for this disposition, under the interim Founder default (GOV-003 §3.2); the identical conflict-of-interest pattern already disclosed at §38.1 |
| Known limitations (§31.3) | See §32 (Risks and Mitigations) and §33 (Deferred Decisions) — the concrete idempotency-declaration type shape, registration transport mechanism, and internal storage/lookup structure remain deferred to a future Engineering Work Order; this approval disposes of the §23.1–§23.5/invariants 42–44 architecture only, not of that future implementation work |
| Unresolved issues (§31.3) | None identified against the 0.4.1 content itself; the Independent Architecture Re-Review found all three prior findings resolved with no new issue introduced |
| Rationale (§31.3) | Approved on the basis that the amendment completes a deferral this document itself already named (§23, §33), introduces no new architectural concept, capability, lifecycle state, or Trusted Core responsibility, preserves the constitutional and Runtime-architecture boundaries established by ARCH-001/ARCH-002, and passed a full review-correction-re-review cycle with no remaining Critical, Major, or unresolved finding |

### 38.3 Disposition of Content Added by v0.4.3 — the Retry Architecture Completion (2026-07-28)

Per GOV-013 §11, amending an already-approved document requires its own evidenced Approval Authority act "regardless of how minor the amendment." The §19.1–§19.4 completion and invariant 45 (added by 0.4.3) accordingly received their own, separate review-and-approval sequence, distinct from — and not covered by — the §38.1 or §38.2 dispositions above.

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-28 |
| Independent Architecture Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `ARCH-008 v0.4.3 INDEPENDENT ARCHITECTURE REVIEW REQUIRES CORRECTION` (two MAJOR findings, two MINOR findings, no BLOCKER) | 2026-07-28 |
| Architecture Amendment Correction | — | Applied exactly the four named findings, folded into the same 0.4.3 entry since this version was never committed or published beforehand; no unrelated content changed | 2026-07-28 |
| Independent Architecture Re-Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `ARCH-008 v0.4.3 INDEPENDENT ARCHITECTURE RE-REVIEW COMPLETE — READY FOR FOUNDER APPROVAL` (all four findings independently confirmed resolved; no regression) | 2026-07-28 |
| Approval Authority | Denver Jacobs, Founder, interim Class B approval default (GOV-003 §3.2 — Chief Architect role vacant) | **Approved** (`FOUNDER APPROVAL GRANTED — ARCH-008 v0.4.3 APPROVED FOR PUBLICATION`) | 2026-07-28 |

**Approval Evidence Record (STD-001 §31).**

| STD-001 §31 field | Value |
|---|---|
| Document ID | ARCH-008 |
| Repository path | `architecture/ARCH-008-Effect-Runtime-Architecture.md` |
| Version | 0.4.3 |
| Artifact revision identifier (§31.1) | The `synapse-docs` commit publishing this exact 0.4.3 Approval Status content, once created — a commit cannot cite its own SHA from within its own tree; the exact commit SHA, tree SHA, and blob ID are independently, reproducibly verifiable via `git log --follow -- architecture/ARCH-008-Effect-Runtime-Architecture.md` and `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md` against this exact path and byte content once published, satisfying §31.1's committed-content requirement without fabricating a value here |
| Content fingerprint (§31.1) | Reproducible as `git hash-object architecture/ARCH-008-Effect-Runtime-Architecture.md` at the commit identified above, once published, for the identical reason given above |
| Git blob ID (§31.1) | Reproducible as `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md`, once published, for the identical reason given above |
| Disposition (§31.2) | Approved |
| Disposition type (§31.2) | Architecture amendment approval (Class B, GOV-010 §4–§5) |
| Approver identity (§31.2) | Denver Jacobs |
| Authority citation (§31.2) | GOV-003 §3.2 — Chief Architect role vacant; Class B (architectural) approval authority defaults, on an interim basis, to the Founder |
| Effective date (§31.2) | 2026-07-28 |
| Review evidence actually available (§31.3) | The complete ARCH-009 Architecture Investigation, Architecture Amendment, Independent Architecture Review, Architecture Amendment Correction, and Independent Architecture Re-Review (§36 References) — disclosed as AI-generated supporting evidence only (GOV-010 §9), not itself independent human review. The Approval Authority's own direct review of this content and its supporting evidence, disclosed as self-review above, is the human review basis for this disposition |
| Independent-review status (§31.3) | No independent *third-party* human review occurred — disclosed, not concealed, on the identical basis as §38.1/§38.2. The interim Founder default (GOV-003 §3.2) applies for the identical reason |
| Self-approval / conflict-of-interest disclosure (§31.3) | Disclosed: Denver Jacobs acted as both the (disclosed) Technical/Architecture Reviewer and the Approval Authority for this disposition, under the interim Founder default (GOV-003 §3.2); the identical conflict-of-interest pattern already disclosed at §38.1/§38.2 |
| Known limitations (§31.3) | See §32 (Risks and Mitigations) and §33 (Deferred Decisions) — numeric retry policy (maximum attempts, backoff, jitter) and the concrete registration/enforcement mechanism for retry-limit constraints remain deferred to a future Engineering Work Order; this approval disposes of the §19.1–§19.4/invariant 45 architecture only, not of that future implementation work |
| Unresolved issues (§31.3) | None identified against the corrected 0.4.3 content itself; the Independent Architecture Re-Review found all four prior findings resolved with no new issue introduced |
| Rationale (§31.3) | Approved on the basis that the amendment completes a deferral this document itself already named (§19, §33), introduces no new architectural concept, capability, lifecycle state, or Trusted Core responsibility, preserves the constitutional and Runtime-architecture boundaries established by ARCH-001/ARCH-002, and passed a full review-correction-re-review cycle with no remaining Critical, Major, or unresolved finding |

### 38.4 Disposition of Content Added by v0.5.0 — the Effect Provider Architecture Integration (2026-07-29)

Per GOV-013 §11, amending an already-approved document requires its own evidenced Approval Authority act "regardless of how minor the amendment." §11.1–§11.4, §13.1–§13.2, §17.1, §18.1, and invariants 46–51 (added by 0.5.0) accordingly received their own, separate review-and-approval sequence, distinct from — and not covered by — the §38.1, §38.2, or §38.3 dispositions above.

This disposition's own governance shape is genuinely different from §38.1–§38.3, disclosed rather than presented as identical: the Independent Architecture Review named below evaluated the content while it lived in a separately authored and published sibling document (ARCH-009 v0.1.0, Draft) rather than a pre-publication draft of this document itself. This does not weaken the disposition — GOV-013 §6.9's own Architecture Review stage is satisfied identically regardless of which document the reviewed content currently resides in — but it is named explicitly so no future reader mistakes this cycle's shape for every prior one's.

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs (AI-assisted) | Drafted (as ARCH-009 v0.1.0, a separate sibling document) | 2026-07-29 |
| Independent Architecture Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `ARCH-009 ARCHITECTURE REVIEW COMPLETE — READY FOR FOUNDER APPROVAL` (AR-009; zero Critical, zero Major findings; two Minor findings — a citation misattribution and a structural-completeness gap, both corrected in this same integration, §37; recommended Option B, integrate into ARCH-008, over Option A, retain standalone) | 2026-07-29 |
| Architecture Amendment | — | ARCH-009's approved content integrated into this document as §11.1–§11.4, §13.1–§13.2, §17.1, §18.1, and invariants 46–51; both Minor findings corrected in the process; ARCH-009 itself marked `Superseded` | 2026-07-29 |
| Approval Authority | Denver Jacobs, Founder, interim Class B approval default (GOV-003 §3.2 — Chief Architect role vacant) | **Approved** (`ARCH-008 UPDATED — PROVIDER ARCHITECTURE INTEGRATED`) | 2026-07-29 |

**Approval Evidence Record (STD-001 §31).**

| STD-001 §31 field | Value |
|---|---|
| Document ID | ARCH-008 |
| Repository path | `architecture/ARCH-008-Effect-Runtime-Architecture.md` |
| Version | 0.5.0 |
| Artifact revision identifier (§31.1) | The `synapse-docs` commit publishing this exact 0.5.0 Approval Status content, once created — a commit cannot cite its own SHA from within its own tree; the exact commit SHA, tree SHA, and blob ID are independently, reproducibly verifiable via `git log --follow -- architecture/ARCH-008-Effect-Runtime-Architecture.md` and `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md` against this exact path and byte content once published, satisfying §31.1's committed-content requirement without fabricating a value here |
| Content fingerprint (§31.1) | Reproducible as `git hash-object architecture/ARCH-008-Effect-Runtime-Architecture.md` at the commit identified above, once published, for the identical reason given above |
| Git blob ID (§31.1) | Reproducible as `git ls-tree <that commit> -- architecture/ARCH-008-Effect-Runtime-Architecture.md`, once published, for the identical reason given above |
| Disposition (§31.2) | Approved |
| Disposition type (§31.2) | Architecture amendment approval (Class B, GOV-010 §4–§5) |
| Approver identity (§31.2) | Denver Jacobs |
| Authority citation (§31.2) | GOV-003 §3.2 — Chief Architect role vacant; Class B (architectural) approval authority defaults, on an interim basis, to the Founder |
| Effective date (§31.2) | 2026-07-29 |
| Review evidence actually available (§31.3) | The complete ARCH-009 Effect Provider Architecture (v0.1.0, Draft) and Independent Architecture Review AR-009 (§36 References) — disclosed as AI-generated supporting evidence only (GOV-010 §9), not itself independent human review. The Approval Authority's own direct review of this content and its supporting evidence, disclosed as self-review above, is the human review basis for this disposition |
| Independent-review status (§31.3) | No independent *third-party* human review occurred — disclosed, not concealed, on the identical basis as §38.1–§38.3. The interim Founder default (GOV-003 §3.2) applies for the identical reason |
| Self-approval / conflict-of-interest disclosure (§31.3) | Disclosed: Denver Jacobs acted as both the (disclosed) Architecture Reviewer (of the standalone ARCH-009) and the Approval Authority for this disposition, under the interim Founder default (GOV-003 §3.2); the identical conflict-of-interest pattern already disclosed at §38.1–§38.3 |
| Known limitations (§31.3) | See §32 (Risks and Mitigations) and §33 (Deferred Decisions) — a future provider discovery/versioning mechanism, concrete streaming/partial-result wire representation, the concrete Provider concurrency mechanism, and `AuditEvent` timestamp/correlation fields all remain deferred to future, separately authorized work; this approval disposes of the §11.1–§11.4/§13.1–§13.2/§17.1/§18.1/invariants 46–51 architecture only. Separately disclosed: `services/http-provider/README.md`'s own forward-reference to a non-existent "ER-018" remains unresolved — out of this amendment's own authority (Runtime read-only; no EWO/ER creation) |
| Unresolved issues (§31.3) | None identified against the integrated content itself; AR-009 found zero Critical and zero Major findings, and both Minor findings are corrected by this same integration (§37) |
| Rationale (§31.3) | Approved on the basis that the integration completes ARCH-009's own §3 recommendation and AR-009's own independent confirmation of it, introduces no new architectural concept beyond what ARCH-009 itself already proposed and AR-009 already reviewed, corrects both identified Minor findings in the process, preserves every existing ownership decision, lifecycle rule, and invariant (1–45) unchanged, and results in a single, non-duplicated authoritative architecture governing both the Effect Runtime and every present and future Effect Provider |
