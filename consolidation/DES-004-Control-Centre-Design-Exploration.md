---
document_id: DES-004
title: "SynapseOS Control Centre: Design Exploration"
version: 0.2.1
status: Draft
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
created: 2026-08-10
last_updated: 2026-08-10
classification: Public
document_family_note: >
  "DES" (Design Exploration) is not currently a registered controlled
  document family in STD-001 Appendix B. This document is placed in
  `consolidation/` — the narrowest existing, purpose-consistent
  location (STD-001 §10), on the same functional basis RSS/ACR/AFR
  and DES-001/DES-002/DES-003 already occupy it: an evidence-to-decision
  synthesis document that precedes, and directly informs, a later
  binding artifact (a future Control Centre Architecture) without
  itself being architecture, governance, or an engineering
  authorization. This placement is a disclosed, narrow convenience,
  not a documentation-hierarchy redesign; formal registration of a
  "DES" family in STD-001, if ever wanted, is a separate, future,
  independently-authorized task, not performed here or implied by
  this placement.
reviewed_by: >
  Design Approval Review of DES-004 v0.1.0 (conversational record;
  not a filed repository document), conducted per GOV-013 §6.5 —
  verdict: REVISION REQUIRED (0 Critical, 1 Major [DAR-004-F01 —
  the Runtime Evidence Integration correction was delivered as a
  separate overlay message, never merged into the base document's
  own consolidated tables], 1 Minor [DAR-004-F02 — the evidence-
  corrected "identity/detail-driven" v1 scope was left unreconciled
  against Progressive Disclosure and the Beginner-stage persona],
  2 Observation [DAR-004-F03 — the Supervisor crate's deliberate
  non-exposure of restart/failure history was not elevated into
  Architecture Questions; DAR-004-F04 — the Application-metadata
  sourcing question was not made explicit]). Design Correction
  (v0.1.0 -> v0.2.0) applied exactly DAR-004-F01 through F04: merged
  the Runtime evidence audit into §9, §11-14, §16-17, §19, §34-35,
  §38, §42, §45-46 as one coherent artifact; reconciled the First
  Useful Version (§35) into a durable-actor-browse path (evidence-
  supported, via Persistence::known_actors()) distinguished
  explicitly from unavailable global actor discovery; elevated the
  supervision-boundary and Application-metadata-sourcing questions
  into §42. Narrow Design Re-Review (conversational record) of
  v0.2.0 verdict — PASS: all four findings independently confirmed
  resolved; one further, Minor, non-blocking finding recorded
  (NDR-004-F01 — §45's Feasibility Matrix did not individually
  cross-reference every CC-ID §38 classifies, a completeness gap,
  not a contradiction). A further, narrow Design Correction
  (v0.2.0 -> v0.2.1) applied exactly this one finding: added one
  clarifying sentence to §45 stating it is a representative
  architectural-feasibility summary, with §38 as the authoritative
  per-requirement source — no requirement, priority, feasibility
  classification, or Runtime evidence altered. Denver Jacobs,
  Founder, accepted DES-004 v0.2.1 as the final Control Centre
  Design Exploration / requirements baseline on 2026-08-10 and
  separately authorized this Repository Filing; this acceptance is
  not Architecture Approval and does not authorize Runtime
  Enumeration/Discovery Architecture, Runtime Control API
  Architecture, Application Architecture, audit-history architecture,
  Control Centre Architecture, GUI technology selection, or
  implementation.
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.4-§6.7, the Design Exploration / Design Approval Review / Design Correction / Narrow Design Re-Review stages this document's own lifecycle follows exactly)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution; §2, §4, §5 principle 2 — binding constraints throughout)
    - ACT-005 (v0.1.0, Approved — Developer Platform Era; §4.G Developer Portal boundary, §6 Control Centre boundary, §7 personas)
  roadmap:
    - ROAD-001 (v0.1.0, Approved — SynapseOS Platform Strategy and Roadmap)
  research:
    - RES-008 (v0.2.0, Founder-accepted — Developer Platform Landscape and Developer Workflow Research; the evidence baseline for every hedged claim in this document)
  architecture:
    - ARCH-015 (v0.2.0, Approved — Developer Platform Boundary Architecture; the cross-cutting invariants — Runtime Authority, State of Record, Capability Security, Auditability, CLI non-substrate, Progressive Disclosure, Error Vocabulary, Compatibility — this exploration's requirements consume throughout, never redefine)
    - ARCH-014 (v0.8.0, Approved — Synapse SDK Architecture; error/compatibility vocabulary consumed via ARCH-015, never duplicated)
    - ARCH-008 (v0.5.0, Approved — Effect Runtime Architecture; §17 Audit Architecture, §29-§30 — the single strongest evidentiary basis for this exploration's own Control Centre compatibility framing and the confirmed absence of a Runtime Control API)
    - ARCH-007, ARCH-011, ARCH-012 (Approved — Persistent Actor / Durable Storage / Durable DomainState architectures; supervision and durability vocabulary consumed for mental-model consistency)
  adrs:
    - ADR-0020 (v0.1.0, Approved — Disposition of the Unrecoverable ARCH-013 Draft)
  consolidation:
    - DES-002 (v0.2.1, Draft, Founder-accepted — Documentation Platform and SDK Documentation: Design Exploration; direct precedent for R12/R15 accessibility framing, §28)
    - DES-003 (v0.1.0, Draft, Founder-accepted, filed — Developer Platform Boundary: Design Exploration; the immediate design predecessor ARCH-015 was authored from)
  source_artifacts:
    - "Runtime introspection capability audit (independent code-level research against synapse-runtime, this engagement; every citation in §34 traces to it directly, not to inference)"
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# DES-004 — SynapseOS Control Centre: Design Exploration

> **Founder Acceptance Notice (2026-08-10).** Denver Jacobs, Founder, has reviewed and accepted this document, at this version (v0.2.1), as the current Control Centre requirements and design baseline, following the complete review/correction/re-review lifecycle recorded in full in the `reviewed_by` field above. Consistent with `DES-001`/`DES-002`/`DES-003`'s own established precedent, this document's own tracked `status:` remains **Draft** — Design Exploration output in this corpus carries no independent approval authority of its own and uses no `Approved` status or formal Approval Status table; Founder disposition is recorded here and in `reviewed_by` instead. **This acceptance is the requirements/design baseline only. It is not Architecture Approval and does not authorize Runtime Enumeration/Discovery Architecture, Runtime Control API Architecture, Application Architecture, audit-history architecture, Control Centre Architecture, GUI technology selection, prototyping, or implementation of any kind.**

*(Section numbering below begins at §4 and ends at §48, preserved exactly as used throughout this document's own engagement and review history — §1-§3 and §49-§52 were process/reporting sections specific to the chat-delivered engagement, not part of the design content itself, and are omitted here per `DES-001`/`DES-002`/`DES-003` precedent.)*

## 4. Product Vision

**The SynapseOS Control Centre makes the Runtime's own already-enforced authority — actor identity, capability, effect, supervision, durable state, audit — visually understandable and, where legitimate, actionable, without ever becoming a second source of that authority.** Long-term: a professional, cross-platform (Windows/Linux/macOS) console — `ARCH-008` §29's own explicit compatibility statement is the authoritative source for this cross-platform intent, not merely product framing. It is closer to an operational/developer control surface than a decorative dashboard, but it earns that status by projecting real, legitimate operations (`ARCH-015` §10/§11), never by inventing convenience ones.

## 5. Product Boundary

**Is:** a Runtime/Application Inspector, combined with a narrowly-scoped Operational Console for legitimate, capability-checked, audited operations. **Is not:** an IDE (no code editing); an account/organization/billing administration system (Developer Portal's domain, `ACT-005` §4.G); a marketplace; a step-through debugger (it observes Runtime-truthful state, it does not halt/single-step Runtime execution — no such mechanism exists in any Approved architecture, §34). A deliberate combination — Runtime Inspector (primary) + limited Operational Console (secondary, legitimate-operations-only).

## 6. Personas

`ACT-005` §7 already establishes three personas "narrowly, as analytical tools only — not new authority classes." This document reuses them rather than inventing a fourth: **Application Developer** (primary), **Contributor/Platform Engineer** ("Platform Developer" for Control-Centre-specific framing, same persona), **Enterprise Evaluator/Operator** ("Operator"). **"Beginner" is deliberately not adopted as a fifth, separate persona** — it is a *stage* within the Application Developer's own journey, governed by `ARCH-015` §15 (Progressive Disclosure), not a distinct authority class.

## 7. Core User Journeys

**Primary** (evidence-derived from `DES-002`/`DX-001`'s own demonstrated flow, extended): developer starts Runtime → opens Control Centre → sees Runtime connection status → sees application(s) → opens actor → observes capability/effect/supervision/durability information → understands a failure → returns to code to fix it. **First-run variants:** no Runtime available; Runtime available, no application yet; Runtime version incompatible; connection fails mid-session — each requires an explicit, non-blank empty/error state (folded into §38 requirements).

## 8. Conceptual Information Model

Candidate first-class visible concepts, tested against `ARCH-015` §16 (`DPB-INV-06`, meaning-consistency): **Runtime**, **Actor**, **Effect**, **Capability**, **Supervision**, **Durability/Recovery**, **Error**, **Authority** — each already a constitutional concept in Approved architecture (`GOV-018` §4; `ARCH-007`/`008`/`011`/`012`/`014`). **Application is explicitly *not* currently a first-class Runtime/SDK concept** (`ARCH-002` §2/§4 places it outside the Runtime's own boundary entirely, independently confirmed at the code level, §34) — a genuine, disclosed architecture gap, not an oversight of this document.

## 9. Application Requirements

Since no Application concept currently exists at Runtime/SDK level (confirmed both architecturally, `ARCH-002` §2/§4, and at the code level — zero matches for "Application" anywhere across the entire `synapse-runtime` workspace), every requirement below is marked **Requires architecture**, not Supported now. `CC-01`: the Control Centre SHOULD present a grouping concept above individual actors, for developer orientation, sourced initially from developer-supplied metadata rather than invented Runtime authority. **Unresolved sourcing question, made explicit (§42):** no configuration/manifest concept currently exists anywhere in the Runtime or SDK either — where such metadata would actually be authored is itself unanswered, not assumed. No application registry, manifest format, or metadata store is invented here.

## 10. Runtime Overview Requirements

`CC-02`: the Control Centre MUST display Runtime connectivity status (connected/disconnected/reconnecting) as the single most primary piece of information — direct consequence of `ARCH-015` `DPB-INV-12` (§21). `CC-03`: SHOULD display Runtime version/compatibility context, consuming `ARCH-014` §14's tier vocabulary via `DPB-INV-10` — MUST NOT independently infer compatibility (`ARCH-015` §21's own tier-vs-version-range distinction, directly binding here). `CC-04`: MAY display active-actor counts and failure indicators where the underlying data is genuinely available (§34 gates this, blocked on the enumeration gap).

## 11. Actor Explorer Requirements

**Split explicitly by feasibility, not treated as one undifferentiated group.** `CC-05` (global actor listing/search): **Requires architecture** — confirmed, no enumeration API exists anywhere in `synapse-runtime`; `ActorDirectory::lookup(name)` (`services/actor-directory/src/lib.rs:18`) and `ActorHost::live_instance(id)` (`core/actor-host/src/lib.rs:34`) are both single-key lookups only. `CC-06` (actor identity display, given an already-known `ActorId`): **Requires exposure** — `ActorDirectory::lookup` gives name→id only, no reverse id→name/type mapping exists; `ActorId`/`ActorInstanceId` carry no metadata accessors. `CC-07` (lifecycle-state display, given a known id): **Requires exposure** — `LifecycleGuardian`'s internal `current_state` (`internal.rs:91`) is not exposed publicly; only `validate_transition` (a yes/no on a *proposed* transition) exists. `CC-08` (parent/child/supervision relationships): see §16, below — partial, deliberately limited.

## 12. Actor Detail Requirements

Given an already-known `ActorId`, the six candidate panels are **not equally feasible**: `CC-09` (Overview: identity/type/lifecycle) — Requires exposure (§11). `CC-10`/`CC-11` (Capabilities, Effects) — **Supported now**, per-known-id (§13/§14 — the strongest-supported panels in the entire product). `CC-12` (Supervision) — Requires architecture, deliberately limited (§16). `CC-13` (Durability) — Requires exposure, though the underlying durable-actor set is at least enumerable (§17). `CC-14` (Diagnostics) — Requires architecture for genuine audit-trail history (§19); real-time error-vocabulary consumption itself is architecturally sound now (`ARCH-014` §12 via `DPB-INV-09`), only the historical-query surface is missing.

## 13. Effect Visibility Requirements

Grounded directly in `ARCH-008` §17's own real, Approved `AuditEvent` architecture. `CC-15` (effect identity/provider/originating-actor/lifecycle-status, given a known id): **Supported now** — `EffectCoordinator` (`services/effect-coordinator/src/lib.rs:166-478`) exposes `effect_status`, `attempt_status`, `actor_of`, `provider_of`, `attempts_for_actor(actor)`, with `EffectStatus`/`AttemptStatus`/`AttemptOutcome` enums — the richest, most GUI-ready subsystem in the codebase. `CC-16` (retry/idempotency-classification display): Should priority, per `ARCH-008` §19, not independently re-derived; feasibility not independently confirmed beyond the base `effect_status`/`attempt_status` surface (§45's own representative-summary scope, not separately broken out). No global "all in-flight effects" listing exists — the same enumeration gap named in §11, **Requires architecture**.

## 14. Capability Requirements

`CC-17` (capability display, given a known `ActorId`): **Supported now** — `CapabilityAuthority::bound_capabilities(actor) -> Vec<Capability>` (`core/capability-authority/src/lib.rs:155`), with `id()`/`target()`/`operations()`/`constraints()` accessors, deliberately including revoked capabilities (not filtered), directly useful for denial-explanation purposes too. `CC-18` (denial explanation): **Supported now** at the vocabulary level (`ARCH-015` `DPB-INV-11`); genuine per-denial diagnostic *history* depends on §19's own audit-history gap — **Requires architecture** for that component specifically. No global capability listing exists independent of already knowing an actor — the same enumeration gap. The GUI MUST NOT ever present a capability as an optional warning — it remains an enforcement boundary, a binding product requirement, not merely a design preference.

## 15. Capability Modification (Deferred)

**Investigated, not adopted as a first-version requirement.** Whether the Control Centre should ever *request* capability-management operations (grant/revoke) — as opposed to only displaying capability state — requires architecture not currently Approved: no Approved artifact defines a legitimate "Control Centre requests a capability change" operation, its own authority basis, or its audit treatment. **Classified: Architecture Question (§42), not a requirement.** No administrative bypass is invented in its place.

## 16. Supervision Requirements

`CC-20` (supervisor/supervised relationships, restart/failure history): **partial, and — unlike most other gaps in this document — deliberately, not accidentally, limited.** `Supervisor::is_registered`/`parent_of` exist (`services/supervisor/src/lib.rs:99,106`), but the crate's own doc comment states outright: *"No numeric restart-limit, backoff, or timing value is exposed by this type or by any other part of this crate's public surface."* This is a disclosed design boundary, not missing exposure — flagged distinctly here and carried into §42 as its own Architecture Question, since revisiting it is a materially different kind of future decision than simply adding an accessor. `CC-21` (visual supervision graph): Could, contingent on `CC-20`'s own resolution — no graph is useful without the underlying relationship data first existing.

## 17. Durable State and Recovery Requirements

`CC-22` (durable/non-durable classification, recovery status): **Requires exposure**, but with one important, evidence-established exception: `Persistence::known_actors()` / `Runtime::known_durable_actors()` (`services/persistence/src/lib.rs:91`; `runtime/src/lib.rs:2914`) is **the one genuine enumeration primitive that already exists in the entire codebase** — it returns every actor currently holding durable state. This is *not* global actor discovery (transient, non-durable actors remain invisible to it), and this document does not conflate the two. `retrieve`/`checkpoint`/`delete` exist; no recovery-timestamp or failure-history is retained anywhere. `CC-23` (no arbitrary durable-state editing): Must, hard boundary — no Approved architecture authorizes this operation at all.

## 18. Diagnostics and Error Requirements

`CC-24`: error presentation MUST remain traceable to `ARCH-014` §12's developer-facing-error vocabulary, via `ARCH-015` `DPB-INV-09` — the Control Centre may improve presentation, explanation, navigation, and contextual remediation guidance, but MUST NOT reinterpret semantic meaning. `CC-25`: error display SHOULD include source subsystem, affected actor/application, time, relevant capability context, and navigation to the affected resource.

## 19. Activity / Audit Requirements

`CC-26`: audit evidence, diagnostic events, and ordinary GUI activity history MUST be visibly, structurally distinct — never conflated, per `ARCH-015` `DPB-INV-08`'s auditability boundary. `CC-27`: any audit evidence the Control Centre displays MUST be sourced from the actual Runtime/`ARCH-008`-defined `AuditEvent` stream, never a GUI-invented log. **The underlying audit mechanism is write-only.** `AuditEmitter::drain` (`core/audit-emitter/src/lib.rs:18-30`) is a *destructive* remove-and-return, not a stable, repeatable query; `AuditPipeline::consume` (`services/audit-pipeline/src/lib.rs:19`) is a one-way sink with no read-back method at all; `AuditEvent` (`common/src/lib.rs:300`) carries no timestamp field. **`CC-26`/`CC-27` are therefore classified `Requires architecture`, not `Requires exposure`** — any genuine audit-trail *history* display (as opposed to a live, ephemeral event feed) requires new Runtime-level audit-storage/query architecture, named explicitly in §42. A genuinely unified cross-subsystem "activity surface" (actor lifecycle + effects + supervision + capability denials + recovery + errors, combined) is classified an **External Dependency** on the still-undesigned Diagnostics/Observability workstream (`RES-008` §12 priority 4, §41).

## 20. Operations Requirements

Every candidate operation (start/stop application, restart actor, request recovery, cancel effect, delete durable actor, refresh, reconnect) is tested against `ARCH-015` §11's own seven-question Authority Projection test. `CC-28` (refresh/reconnect) — **Supported now**, no underlying authority question, pure GUI-local behavior. `CC-29` (cancel effect) — `ARCH-008` §16's own cancellation semantics exist architecturally; exposure requires implementation/architecture confirmation — Requires architecture. `CC-30` (restart actor, request recovery, delete durable actor) — each requires a corresponding legitimate Runtime Control API operation `ARCH-008` §29 names as *anticipated* but explicitly *not yet designed* (§30 Non-Goals) — **Requires architecture**, not silently manufactured. `CC-31` (start/stop application) — blocked entirely on §9's own unresolved Application-concept gap; cannot legitimately exist until that architecture question is resolved.

## 21. State-of-Record Requirements

Direct, binding consequence of `ARCH-015` `DPB-INV-12` (§10.1). `CC-32`: the Control Centre MUST visibly distinguish live/fresh data from cached/stale data. `CC-33`: MUST clearly indicate disconnected state and MUST NOT present last-known state as current truth once disconnected. `CC-34`: every displayed value originating from a local cache/projection MUST remain traceable back to "the owning subsystem is the source of truth." No polling, push, WebSocket, database, cache, or synchronization technology is selected.

## 22. Connectivity Requirements

`CC-35`: MUST support a single local Runtime connection as the first-version baseline. `CC-36` (Should): a single remote Runtime connection. **Multiple simultaneous Runtime instances, enterprise fleet management: explicitly Future Scope (§36), not first-version** — no evidence in `ACT-005`, `ROAD-001`, or `RES-008` supports first-version multi-Runtime scope.

## 23. Search / Navigation Requirements

`CC-37` (Should): entity search (application, actor, effect, capability, error) — a future enhancement, not confirmed Must-priority absent developer-workflow evidence (`RES-008` §15's own limitation applies directly).

## 24. Progressive Disclosure Requirements

Direct application of `ARCH-015` `DPB-INV-05`. `CC-38`: a beginner-stage developer MUST be able to understand basic Runtime/application/actor state without confronting supervision internals, raw audit event types, or DomainState encoding details. `CC-39`: nothing learned at an overview level may need to be unlearned at a detail level — every deeper panel must be an addition over, never a replacement of, the overview's own concepts. **Reconciled explicitly against the evidence-corrected first-use experience in §35, below — see that section for how this is satisfied despite the confirmed absence of global actor discovery.**

## 25. Security UX Requirements

Direct application of `ARCH-015` `DPB-INV-07`/`09`/`11`. `CC-40`: any state-changing action MUST be visually, structurally distinguished from read-only/observational actions. `CC-41`: required capability MUST be displayed before a privileged action is attempted, not only after denial. `CC-42`: destructive operations (§26) MUST require explicit confirmation.

## 26. Destructive Operations Requirements

`CC-43`: every destructive operation exposed by the Control Centre MUST first satisfy `ARCH-015` §11's full seven-question Authority Projection test — per §20's own findings, most destructive candidates do not yet have a confirmed legitimate underlying Runtime Control API operation, and are therefore **not yet eligible for GUI exposure at all**, independent of any confirmation-dialog UX question.

## 27. Cross-Platform Requirements

`CC-44`: MUST behave substantially consistently across Windows, Linux, and macOS — direct product requirement, sourced from `ARCH-008` §29's own explicit, Approved-architecture-level compatibility statement. No framework selected.

## 28. Accessibility Requirements

Direct precedent from `DES-002` R12/R15 (Founder-accepted, both still binding). `CC-45` (Should): keyboard-navigable, semantically structured, sufficiently contrasted, screen-reader compatible, scalable UI. `CC-46` (Deferred, matching `DES-002` R15 exactly): the specific compliance standard is **not selected here** — pending the same external research and Founder decision `DES-002` R15 already deferred, not independently re-opened.

## 29. Performance / Runtime-Safety Requirements

Direct consequence of `ARCH-015` `DPB-INV-01`. `CC-47`: the Control Centre MUST NOT issue inspection queries at a volume or frequency that materially degrades Runtime operation — no numeric target invented. `CC-48`: expensive information SHOULD be fetched intentionally (on navigation/request), not eagerly by default.

## 30. Offline / Disconnected Requirements

Consolidated with §21 above (`CC-32`/`CC-33`) — presenting them as one requirement set avoids manufactured duplication.

## 31. CLI Relationship

Direct, binding consequence of `ARCH-015` §13's own CLI-non-substrate clarification. `CC-49`: the Control Centre MUST reach legitimate SDK/platform operations directly — it MUST NOT be architecturally required to shell out to, wrap, or otherwise depend on the CLI as its own only path to any operation. The GUI and CLI MAY expose equivalent legitimate operations as independent surfaces.

## 32. SDK Relationship

No SDK modification proposed. Every gap identified in §34 that traces to missing SDK-level exposure (as opposed to missing Runtime-level capability entirely) is classified **Requires exposure**, a distinct, lighter category than **Requires architecture**. No missing GUI operation is assumed to automatically warrant SDK addition.

## 33. Developer Portal / Documentation Relationship

**Developer Portal:** `CC-50`-adjacent boundary only — Control Centre interacts with actual SynapseOS execution environments; Developer Portal (`ACT-005` §4.G) owns ecosystem/distribution/account/documentation/community concerns. No dependency either direction. **Documentation:** `CC-50` (Could, Future-Scope-adjacent, not first-version-blocking): context-sensitive "Learn more" links from a Control Centre concept to the corresponding Documentation Platform content, once it exists. Does **not** require Documentation Platform implementation as a precondition — the two workstreams remain independently sequenced, per `ACT-005` §10's own parallel, not sequential, dependency model.

## 34. Runtime Introspection Gap Analysis

*(Final, single table — the authoritative source for every feasibility claim elsewhere in this document.)*

| Desired Capability | Existing Support | Evidence | Gap? | Classification |
|---|---|---|---|---|
| Actor listing/discovery (global) | Does not exist | No enumeration API; `ActorDirectory::lookup`/`ActorHost::live_instance` are single-key lookups only | Yes | **Requires architecture** |
| Actor identity inspection (given known id) | Partial | `ActorDirectory::lookup` name→id only; no reverse mapping, no metadata accessors | Yes | Requires exposure |
| Actor lifecycle-state query | Does not exist as a query | `LifecycleGuardian`'s internal state not publicly exposed; only proposed-transition validation exists | Yes | Requires exposure |
| Capability inspection, given known actor | **Exists** | `CapabilityAuthority::bound_capabilities(actor)`, full accessor set | No | **Supported now** |
| Effect visibility, given known id | **Exists (rich)** | `EffectCoordinator`'s full query surface | No | **Supported now** |
| Supervision inspection | Partial, **deliberately** limited | `is_registered`/`parent_of` exist; restart/failure history explicitly excluded by the crate's own documented design boundary | Yes | **Requires architecture** (boundary revisit, §42) |
| Durable state/recovery status | Partial — **the one true enumeration primitive** | `known_actors()`/`known_durable_actors()`; no recovery timestamp/failure history | Yes, partial | Requires exposure |
| Audit trail read/query | Partial, **write-only** | `drain()` is destructive, not a query; no read-back method; no timestamp field | Yes | **Requires architecture** |
| Application-level concept | Confirmed absent | Zero code-level matches; `ARCH-002` boundary statement | Yes | Requires architecture |
| Runtime-level status/identity | Partial | `Runtime::state()` exists; no version/connectivity/uptime/count | Yes | Requires exposure |
| Runtime Control API | Does not exist | `ARCH-008` §30's own explicit Non-Goal | Yes | Requires architecture |

**Governing structural finding:** with the sole exception of `known_actors()`/`known_durable_actors()`, no enumeration/discovery primitive exists anywhere in `synapse-runtime` — every other capability is a point lookup keyed by an already-known id.

## 35. First Useful Version Scope

Two honestly distinct tiers, not one blurred v1 claim, directly reconciling `§24`'s own Progressive Disclosure requirement against the evidence above:

- **What v1 can offer a developer who does *not* yet know a specific `ActorId`:** a **Durable Actors** browse list, sourced from `known_actors()`/`known_durable_actors()` — the one genuine, evidence-supported enumeration primitive that exists. Durable actors are visible and browsable; **transient/non-durable actors are not**, and this document does not represent otherwise. Clicking into a durable actor from this list reaches §12's Actor Detail panels, where Capability and Effect data are already `Supported now`.
- **What v1 cannot offer:** a complete "browse every actor" experience, or any listing of transient actors, effects, or capabilities independent of already knowing an id. **This is an explicit architectural dependency (§42, §48), not solved by this document.**

This satisfies `CC-38`/`CC-39` for the durable-actor path (a beginner can browse and drill in without prior error-log knowledge) while remaining honest that the *complete* discovery experience — reaching a transient actor a developer only knows about from an error message — still requires already having that actor's id today. No Runtime enumeration mechanism is invented to close this gap.

## 36. Future Scope

Multiple/remote Runtime instances beyond one; capability modification (§15); a unified cross-subsystem activity surface (§19); Application-concept-dependent features (§9, §20); visual supervision graphs (§16, Could); Documentation Platform deep-linking (§33); Developer Portal integration; distributed/fleet management; Synapse Cloud; Enterprise administration; AI Workforce management; marketplace; billing.

## 37. Explicit Exclusions

Runtime redesign; SDK redesign; Distributed Runtime implementation; Synapse Cloud; Enterprise Edition; AI Workforce Platform; marketplace; billing; broad organization administration; Developer Portal implementation; Documentation Platform implementation; GUI framework/technology selection; implementation or prototype coding of any kind.

## 38. Complete Normative Requirements Set

Fifty tracked identifiers (`CC-01`–`CC-50`):

| ID | Statement (abbreviated) | Class | Priority | Feasibility |
|---|---|---|---|---|
| CC-01 | GUI-level Application grouping, non-authoritative | Functional | Should | Requires architecture |
| CC-02 | Runtime connectivity status | Functional | Must | Requires exposure |
| CC-03 | Version/compatibility context | Compatibility | Should | Requires exposure |
| CC-04 | Active-actor counts/failure indicators | Functional | Could | Requires architecture |
| CC-05 | Global actor list/search | Functional | Should | **Requires architecture** |
| CC-06–07 | Actor identity/lifecycle, given known id | Functional | Should | Requires exposure |
| CC-08 | Actor supervision relationships | Functional | Should | Requires architecture |
| CC-09 | Actor Overview panel | Functional | Should | Requires exposure |
| CC-10–11 | Capability/Effect panels | Functional | Should | **Supported now** |
| CC-12 | Supervision panel | Functional | Should | Requires architecture |
| CC-13 | Durability panel | Functional | Should | Requires exposure |
| CC-14 | Diagnostics panel | Functional | Should | Requires architecture (history) |
| CC-15 | Effect visibility, given known id | Functional | Should | **Supported now** |
| CC-16 | Retry/idempotency classification | Functional | Should | Not independently confirmed beyond CC-15's base surface |
| CC-17 | Capability display | Security | Must | **Supported now** |
| CC-18 | Denial explanation | Security | Must | Supported now (display) / Requires architecture (history) |
| CC-19 | Capability modification | — | — | Deferred, §15 |
| CC-20–21 | Supervision display/graph | Functional | Should/Could | Requires architecture |
| CC-22 | Durability/recovery display | Functional | Should | Requires exposure (enumeration exists) |
| CC-23 | No arbitrary durable-state edit | Security | Must | Hard boundary |
| CC-24–25 | Error vocabulary consistency, error detail | Content | Must/Should | Sound now (vocabulary) |
| CC-26–27 | Audit/diagnostic/GUI-activity separation, real-source-only | Governance | Must | **Requires architecture** |
| CC-28 | Refresh/reconnect | Functional | Must | **Supported now** |
| CC-29 | Cancel effect | Functional | Should | Requires architecture |
| CC-30 | Restart/recovery/delete operations | Functional | Deferred (gated) | Requires architecture |
| CC-31 | Application start/stop | Functional | Deferred (gated) | Requires architecture |
| CC-32–34 | Stale/fresh distinction; disconnected indication; traceable source | Governance | Must | Requires exposure (mechanism-neutral) |
| CC-35–36 | Local Runtime (Must); remote Runtime (Should) | Functional | Must/Should | New |
| CC-37 | Entity search | Functional | Should | New, hedged |
| CC-38–39 | Progressive Disclosure no-omission/no-unlearning | Content | Must | Satisfied via §35's durable-actor path |
| CC-40–42 | Read/write distinction; capability-before-action; confirmation | Security | Must | New |
| CC-43 | Authority Projection test gates every destructive op | Security | Must | New |
| CC-44 | Cross-platform consistency | Functional | Must | New |
| CC-45–46 | Accessibility attributes (Should); standard (Deferred) | Accessibility | Should/Deferred | `DES-002` R12/R15 precedent |
| CC-47–48 | No Runtime destabilization; intentional expensive fetch | Performance | Must/Should | New |
| CC-49 | No architectural CLI dependency | Architectural | Must | New |
| CC-50 | Documentation deep-linking | Content | Could | New |

## 39. Constraints

`C-CC-01`: no operation may be exposed without passing `ARCH-015` §11's full seven-question Authority Projection test. `C-CC-02`: no capability-management (grant/revoke) operation is authorized. `C-CC-03`: no durable-state editing is authorized. `C-CC-04`: no GUI framework, transport, storage, or synchronization technology is selected. `C-CC-05`: no Application-concept-dependent feature may be built ahead of that architecture question's own resolution.

## 40. Deferred Items

`CC-19` (capability modification); `CC-46` (specific accessibility standard); multi-Runtime scope; visual supervision graph, pending concrete evidence; a unified cross-subsystem activity surface; Documentation Platform deep-linking implementation.

## 41. External Dependencies

Diagnostics/Observability workstream (`RES-008` §12 priority 4, §19); Documentation Platform (§33, non-blocking); Developer Portal (§33, boundary only); the still-undesigned Runtime Control API (§34, §42); the still-undesigned Application concept (§9, §34, §42); `RES-008`'s own zero-external-evidence limitation.

## 42. Architecture Questions (deferred, not answered here)

- **Runtime enumeration/discovery** — how would global actor (and eventually effect) discovery be architected, given today's point-lookup-only design? The single most foundational open question this exploration surfaces.
- **Runtime Control API** — the mechanism `ARCH-008` §29 anticipates but §30 explicitly does not design.
- **Application concept and metadata ownership** — including *where* non-authoritative GUI-level Application metadata would be authored or sourced, given no configuration/manifest concept exists today.
- **Supervision visibility boundary** — should the Supervisor crate's own deliberate non-exposure of restart-count/failure-history/backoff-timing be revisited for Control-Centre-facing diagnostics, and under what future architecture? This does not itself authorize changing that boundary.
- **Audit-history architecture** — a stable, non-destructive, timestamped, queryable audit/event store, distinct from the existing write-only `AuditEmitter`/`AuditPipeline`.
- GUI architecture/process model; local/remote connection model; state synchronization/caching mechanism; authentication; authorization projection for the GUI's own session; secure credential handling; Runtime discovery mechanism; multi-Runtime model; packaging/distribution; update mechanism; desktop/web boundary; cross-platform framework; persistence of GUI preferences; diagnostics transport.

## 43. Requirement Traceability Matrix

| Item | GOV-018 | ARCH-015 | ARCH-014 | ARCH-008 | ARCH-007/011/012 | DES-002 | RES-008 |
|---|---|---|---|---|---|---|---|
| CC-02, 32-34 | §4 | DPB-INV-12 | — | — | — | — | — |
| CC-03, 44 | — | DPB-INV-10 | §14 | §29 | — | — | — |
| CC-15-16 | — | DPB-INV-09 | §12 | §17, §19 | — | — | — |
| CC-17-18, 40-43 | §5 | DPB-INV-07/09/11 | — | — | — | — | §13 |
| CC-20-23 | — | DPB-INV-06 | — | — | ARCH-007, 011/012 | — | — |
| CC-26-27 | §4 item 4 | DPB-INV-08 | — | §17 | — | — | — |
| CC-37 | — | — | — | — | — | R26 (precedent) | §15 (hedge) |
| CC-45-46 | — | — | — | — | — | R12/R15 | — |
| CC-49 | — | §13 | — | — | — | R02 | — |

## 44. Approved Architecture Conflict Analysis

| Existing Architecture | Relationship | Conflict? |
|---|---|---|
| `ARCH-015` | Directly implementing (every requirement traces to or is bounded by it) | No |
| `ARCH-014` | Compatible, dependent (error/compatibility vocabulary consumed, never redefined) | No |
| `ARCH-008` | Compatible, directly anticipated (§17/§29 are this exploration's own strongest evidentiary basis) | No |
| `ARCH-007`/`011`/`012` | Compatible, dependent (supervision/durability vocabulary consumed) | No |
| `ARCH-002` (Draft, disclosed) | Overlapping but correctly subordinate — cited once, for its own boundary-exclusion statement only, not relied on as authority elsewhere | No |
| `GOV-018` | Compatible, constitutionally governed | No |

**Zero conflicts found.**

## 45. Feasibility Matrix

*(This table is a representative architectural-feasibility summary, organized by capability rather than by individual requirement. §38 — Complete Normative Requirements Set — is the authoritative source for per-requirement / per-`CC`-ID feasibility classification; an item's absence from this table reflects summary scope, never a missed or contradicted classification.)*

| Capability | Classification |
|---|---|
| Refresh/reconnect (`CC-28`) | **Supported now** |
| Capability display, given known actor (`CC-10`/`17`/`18`) | **Supported now** (display); Requires architecture (denial history) |
| Effect visibility, given known id (`CC-11`/`15`) | **Supported now** |
| Actor identity/lifecycle, given known id (`CC-06`/`07`/`09`) | **Requires exposure** |
| Durable-actor browse list (`CC-13`/`22`, via `known_actors()`) | **Requires exposure** — closest-to-buildable "list" experience |
| Runtime status/version display (`CC-02`/`03`) | Requires exposure |
| Global actor/effect/capability listing (`CC-04`/`05`) | **Requires architecture** |
| Supervision relationship/history (`CC-08`/`20`/`21`) | **Requires architecture** (deliberate boundary) |
| Audit-trail history (`CC-14`/`26`/`27`) | **Requires architecture** |
| Application grouping (`CC-01`) | **Requires architecture** |
| Restart/recovery/delete operations, app start/stop (`CC-29`–`31`) | **Requires architecture** |
| Capability modification (`CC-19`) | Deferred |
| Multi-Runtime, doc deep-linking (`CC-36`, `CC-50`) | Deferred |

## 46. GUI Build Readiness Assessment

(1) this Design Exploration (v0.2.1) → (2) Founder final Design Acceptance (granted, this Filing) → (3) **a dedicated Runtime enumeration/discovery architecture** — arguably the most foundational open item, since it blocks basic read/discovery before any write-operation question arises — and (4) **a dedicated Runtime Control API architecture**, both independently authorizable, neither authorized by this document → (5) an Application-concept architecture decision → (6) audit-history architecture → (7) Control Centre Architecture Authoring (`GOV-013` §6.8) → (8) Independent Architecture Review → (9) Founder Architecture Approval → (10) Engineering Work Order → (11) implementation. **Design completion is not build authorization.**

## 47. Design Approval Review Readiness

Complete — this document itself is the product of a full Design Approval Review → Design Correction → Narrow Design Re-Review cycle (`reviewed_by`, above), independently confirmed coherent at every stage.

## 48. Risks / Open Questions

Building GUI expectation ahead of Runtime Control API / enumeration architecture (mitigated: §45/§46 explicitly gate every operation-class and list-view requirement behind that architecture's own future existence); Application-concept scope creep if `CC-01` is misread as already-authoritative (mitigated: §9/§34 disclose this explicitly and repeatedly); zero external developer evidence for any UX-preference claim in this document (`RES-008` §15, unchanged); the durable-actor-only v1 discovery path (§35) may still read as incomplete to a developer expecting to find a transient actor they know exists by other means — an honest, disclosed v1 limitation, not eliminated by any correction performed.

## Acceptance Framework

This exploration is adequately specified when: every proposed `CC` requirement has an explicit class, priority, and feasibility classification traceable to either Approved architecture or direct code-level evidence (§38, §34); every inherited constraint is separated from new proposal; the `ARCH-014`/`ARCH-015` relationships are mapped with zero unresolved conflicts (§44); every non-goal is verified unviolated (§37); every evidence-thin claim is explicitly hedged (§41, §48). This document's own completion is not self-declared — that was the Design Approval Review's, Design Correction's, and Narrow Design Re-Review's own role (`reviewed_by`, above), and ultimately the Founder's.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Design Exploration Draft, per `GOV-013` §6.4, answering "what should the Control Centre be?" Fifty `CC` requirements, grounded in `ARCH-015`'s cross-cutting invariants and `ARCH-008` §17/§29-§30's own evidence. Design Approval Review found `DAR-004-F01` (Major), `F02` (Minor), `F03`/`F04` (Observation). |
| 0.2.0 | 2026-08-10 | Denver Jacobs (Founder) | Design Correction applying exactly `DAR-004-F01`–`F04`: merged the independent Runtime introspection code audit into §9, §11-14, §16-17, §19, §34-35, §38, §42, §45-46 as one coherent artifact; reconciled §35's First Useful Version into a durable-actor-browse path, explicit about what remains architecturally blocked; elevated the supervision-boundary and Application-metadata-sourcing questions into §42. Narrow Design Re-Review: PASS, one further Minor, non-blocking finding (`NDR-004-F01` — §45's incomplete per-ID cross-referencing against §38). |
| 0.2.1 | 2026-08-10 | Denver Jacobs (Founder) | Narrow Design Correction applying exactly `NDR-004-F01`: added one clarifying sentence to §45 stating it is a representative architectural-feasibility summary, with §38 as the authoritative per-requirement source. No requirement, priority, feasibility classification, or Runtime evidence altered. |
| 0.2.1 (this filing) | 2026-08-10 | Denver Jacobs (Founder) | Repository Filing. Records genuine Founder acceptance of this document, at this version, as the final Control Centre Design Exploration / requirements baseline (`reviewed_by`, above). No substantive content altered by this filing — identifier, frontmatter, this Founder Acceptance Notice, and this Revision History entry are the only additions. Does not authorize Runtime Enumeration/Discovery Architecture, Runtime Control API Architecture, Application Architecture, audit-history architecture, Control Centre Architecture, GUI technology selection, or implementation. |
