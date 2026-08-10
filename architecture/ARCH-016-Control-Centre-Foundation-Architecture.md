---
document_id: ARCH-016
title: SynapseOS Control Centre Foundation Architecture (Phase 1: Read-Only)
project: SynapseOS
specification: SynapseOS — the minimum coherent architecture necessary for a read-only SynapseOS Control Centre Phase 1, a developer-facing projection surface over Runtime/SDK authority already established by ARCH-007/008/011/012/014 and bound by ARCH-015's cross-cutting Developer Platform Boundary invariants
version: 0.2.1
status: Approved
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.2 interim Chief Architect authority (GOV-010 §5, Class B) — no Chief Architect is currently appointed, the identical basis ARCH-007, ARCH-008, ARCH-011, ARCH-012, ARCH-014, and ARCH-015 each already record for themselves.
created: 2026-08-10
last_updated: 2026-08-10
classification: Public
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.8-§6.11, the Architecture Authoring / Independent Architecture Review / Architecture Correction / Narrow Architecture Re-Review lifecycle this document's own history follows exactly)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution)
    - ACT-005 (v0.1.0, Approved — Developer Platform Era; §6 Control Centre boundary, §7 personas)
  roadmap:
    - ROAD-001 (v0.1.0, Approved — SynapseOS Platform Strategy and Roadmap)
  research:
    - RES-008 (v0.2.0, Founder-accepted — Developer Platform Landscape and Developer Workflow Research)
  architecture:
    - ARCH-007 (v0.5.2, Approved — Persistent Actor Architecture)
    - ARCH-008 (v0.5.0, Approved — Effect Runtime Architecture; §17, §29, §30 — the single strongest evidentiary basis for this document's own Control Centre compatibility framing and the confirmed absence of a Runtime Control API)
    - ARCH-011 (v0.1.3, Approved — Durable Storage Mechanics)
    - ARCH-012 (v0.2.0, Approved — Durable DomainState Encoding Architecture)
    - ARCH-014 (v0.8.0, Approved — Synapse SDK Architecture; error/compatibility vocabulary consumed via ARCH-015, never duplicated)
    - ARCH-015 (v0.2.0, Approved — Developer Platform Boundary Architecture; the cross-cutting invariants this document inherits in full and directly extends at the Control Centre boundary)
  consolidation:
    - DES-004 (v0.2.1, Draft, Founder-accepted, filed — SynapseOS Control Centre: Design Exploration; the requirements/design baseline this document transforms into architecture)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-016 — SynapseOS Control Centre Foundation Architecture (Phase 1: Read-Only)

> **Filing Notice.** This document was authored, independently reviewed, corrected, re-reviewed, further corrected, and re-reviewed a second time entirely as a chat-delivered artifact before this Repository Filing — the identical lifecycle shape `ARCH-007`, `ARCH-012`, `ARCH-014`, and `ARCH-015` each established for themselves. Version 0.1.0 was the initial Architecture Authoring Draft, transforming `DES-004` v0.2.1's Founder-accepted requirements into a narrowly-scoped, read-only Phase 1 Control Centre foundation. Its Independent Architecture Review found `REVISION REQUIRED` (0 Blocking, 0 Major, 3 Minor [`IAR-016-F01`–`F03`], 1 Observation [`IAR-016-F04`]), Founder-accepted in full. Version 0.2.0 applied exactly those four findings; its own Narrow Architecture Re-Review returned `FAIL` — `IAR-016-F02` only Partially Resolved, one new Minor finding (`NAR-016-F01`, a termination-vs-durable-deletion ambiguity). Version 0.2.1 applied exactly `NAR-016-F01`, evidence-confirmed by direct inspection of `synapse-runtime` source (`core/actor-host`, `core/lifecycle-guardian`, `services/persistence/src/internal.rs`, `runtime/src/lib.rs:3170`); its Second Narrow Architecture Re-Review returned `PASS` — zero remaining findings of any severity across the entire review chain. Founder Architecture Approval is recorded in full below (§49, Disposition); this Filing (v0.2.1) records that approval and constitutes this document's own Repository Filing. It does not itself authorize a GUI Engineering Work Order, GUI technology selection, or implementation of any kind.

## 1. Document Control

| Field | Value |
|---|---|
| Document ID | ARCH-016 |
| Title | SynapseOS Control Centre Foundation Architecture (Phase 1: Read-Only) |
| Version | 0.2.1 |
| Status | **Approved** — Founder Architecture Approval recorded below (§49) |
| Author | Denver Jacobs (AI-assisted) |
| Approval authority | Chief Architect (Class B, per `GOV-010` §5), vacant; Founder (interim), per `GOV-003` §3.2 |
| Created | 2026-08-10 |
| Classification | Public |
| Requirements baseline | `DES-004` v0.2.1 (Founder-accepted, filed, commit `13ccb1d`) |
| Constitutional authority | `GOV-018`, `ACT-005`, `ARCH-015` (Approved v0.2.0), `ARCH-014` (Approved v0.8.0), `ARCH-007`, `ARCH-008`, `ARCH-011`, `ARCH-012` |
| Numbering | `ARCH-016` — freshly verified at Authoring; unaffected by either correction |
| Review requirement (v0.1.0, complete) | Architecture Authoring → Independent Architecture Review → **REVISION REQUIRED** (0 Blocking, 0 Major, 3 Minor, 1 Observation) — Founder-accepted, correction authorized |
| Review requirement (v0.2.0, complete) | Narrow Architecture Correction (`IAR-016-F01`–`F04`) → Narrow Architecture Re-Review → **FAIL** (`F02` Partially Resolved; `NAR-016-F01`, Minor) — Founder-accepted, further correction authorized |
| Review requirement (v0.2.1, complete) | Further Narrow Architecture Correction (`NAR-016-F01`, evidence-confirmed) → Second Narrow Architecture Re-Review → **PASS** (zero remaining findings) → Founder Architecture Approval |

## 2. Executive Summary

This document establishes the minimum coherent architecture necessary for a **read-only** SynapseOS Control Centre Phase 1 — a developer-facing projection surface over Runtime/SDK authority already established by `ARCH-007`/`008`/`011`/`012`/`014` and bound by `ARCH-015`'s cross-cutting Developer Platform Boundary invariants. It authorizes no Runtime mutation, no persistent audit-history subsystem, no global actor discovery, and no authoritative Application concept — each is explicitly, architecturally deferred, its own future extension point preserved without being designed here. It directly implements the sequencing conclusion of the preceding Control Centre Architecture Dependency & Sequencing Assessment: a narrowly-scoped architecture now, amended narrowly later as `D1` (enumeration), `D2` (control operations), and `D4` (persistent audit) are each separately, independently resolved by their own Runtime-tier architecture workstreams.

Version 0.2.0 applied exactly `IAR-016-F01`–`F04`: precised the `CC-18` requirement-mapping split (`F01`); added durable-actor-enumeration semantics (`F02`, partially — inclusion, ordering, staleness); established one cross-cutting partial-data-disclosure rule applied uniformly across every partial projection (`F03`); added an explicit Runtime-restart scenario to the Failure Model (`F04`). Its own Narrow Architecture Re-Review found `F02` only Partially Resolved — the termination-vs-durable-deletion distinction was implied but not explicit — and returned `FAIL`. Version 0.2.1 closed that gap with direct, positive source confirmation (`core/actor-host`, `core/lifecycle-guardian`, `services/persistence/src/internal.rs`, `runtime/src/lib.rs:3170`): actor termination and durable-state deletion are structurally independent operations; termination alone never changes durable-enumeration membership. The Second Narrow Architecture Re-Review confirmed this resolved, with zero remaining findings anywhere in the review chain, and returned `PASS`. Founder Architecture Approval is recorded in full below (§49). No SDK, Runtime, CLI, Documentation Platform, or Control Centre implementation is designed or authorized by this document.

## 3. Context

`DES-004` v0.2.1 (Founder-accepted) established fifty Control Centre requirements, each classified against real, code-level Runtime/SDK evidence as Supported now, Requires exposure, Requires architecture, or Deferred. This document is the `GOV-013` §6.8 Architecture Authoring stage for that accepted design, scoped — per the Founder's own explicit authorization — to exactly the subset `DES-004` §35 already identified as coherent without `D1`/`D2`/`D3`(full)/`D4`: a read-only foundation.

## 4. Authority and Constitutional Basis

`GOV-018` §4/§5 (constitutional properties, capability-security principle); `ACT-005` §6 (Control Centre boundary — no implementation authorization, a compatible future path only) and §7 (personas); `ARCH-015` (the cross-cutting Developer Platform Boundary this document inherits in full — `DPB-INV-01` through `DPB-INV-10`, `DPB-INV-12`, and §24's own Control Centre Boundary narrative, all directly binding here); `ARCH-014` (SDK error/compatibility vocabulary, consumed via `ARCH-015`, never duplicated); `ARCH-007`/`008`/`011`/`012` (Runtime-tier authority for actor lifecycle, effects, capability, durability — none redefined here). `ARCH-002`/`ARCH-001`/`ADR-0016` remain Draft, disclosed, not relied upon directly, matching `ARCH-015`'s own established citation discipline.

## 5. Scope

The architectural boundaries required for a read-only Control Centre Phase 1: Runtime connection and status; durable-actor browsing and detail (identity, lifecycle, capabilities, effects, diagnostics); state-of-record and staleness presentation; the internal domain/presentation boundary; security/authority projection for read operations; cross-platform and presentation-technology independence. Nothing beyond this.

## 6. Non-Goals

Runtime redesign; SDK redesign; global actor enumeration architecture; Runtime Control API architecture; persistent audit-history architecture; authoritative Application architecture; Control Centre implementation; GUI technology selection; UI visual design; packaging implementation; cloud architecture; Enterprise Edition; AI Workforce Platform; domain-specific commercial applications; any other `ACT-005` domain.

## 7. Architectural Principles

1. **Projection, not authority.** Every fact the Control Centre displays is a projection of Runtime/SDK truth, never an independently manufactured one (`ARCH-015` §10/§11).
2. **Honesty over completeness.** Where information is unavailable, unsupported, or partial, the architecture requires accurate disclosure of that limitation over a fabricated or implied completeness.
3. **Minimal architecture.** Only decisions that genuinely require durable, cross-cutting resolution are settled here (`ARCH-015` §9's Minimal Architecture Principle, directly reused).
4. **Narrow, disclosed deferral.** Every excluded capability names its own future extension point rather than silently foreclosing it.

## 8. System Context

```
Runtime (ARCH-007/008/011/012)  — sole authority for execution, capability, effect, durability, supervision
        │  (via SDK, ARCH-014)
        ▼
Synapse SDK — stable programmatic interface, never bypassed
        │
        ▼
Control Centre Domain (this document)
   ├── Runtime Connection Boundary
   ├── Developer Projection/Domain Boundary
   ├── State Observation Boundary
   ├── Capability/Authority Projection
   ├── Diagnostics Projection
   └── Presentation Boundary  ── (technology-independent, not designed here)
```

The Control Centre never connects to the Runtime except through the SDK (`ARCH-015` §12/§13 inherited directly — no CLI, no undocumented internal API).

## 9. Authority Model

**Invariant, restated from `ARCH-015` §10/§11, applied concretely here:** the Control Centre may observe, project, explain, and navigate legitimate SDK/Runtime operations; it may never independently create execution authority, capability authority, or a competing system of record. Every displayed fact must be traceable to a specific SDK/Runtime query (`ARCH-015` §11's seven-question Authority Projection test, unmodified).

## 10. Runtime / SDK / Control Centre Boundary

Tested directly against `ARCH-014`/`ARCH-015` rather than assumed: the Control Centre **must** consume Runtime state exclusively through the SDK — no direct Runtime-internal API access, matching `ARCH-015` §12's own SDK-boundary rule applied one layer further outward. No inspection surface bypasses the SDK; where the SDK does not yet expose something the Runtime legitimately possesses (§34's own Gap Analysis, inherited from `DES-004`), that is a **Requires exposure** dependency, resolved by future SDK work under `ARCH-014`'s own amendment process if the underlying data is not already reachable — never by the Control Centre reaching around the SDK. This preserves SDK evolution compatibility and prevents the Control Centre from becoming a privileged backdoor (`ARCH-015` §24, directly inherited).

## 11. Control Centre Internal Boundaries

Six conceptual boundaries, not implementation classes, crates, or frameworks — this architecture does not mandate any particular code structure, only that these concerns remain conceptually separable:

- **Runtime Connection Boundary** — owns the connection/session relationship to one local Runtime (§13).
- **Developer Projection/Domain Boundary** — transforms platform concepts into developer-facing Control Centre concepts without altering their authority (`ARCH-015` `DPB-INV-06` inherited).
- **State Observation Boundary** — represents observed Runtime state and its own synchronization/staleness lifecycle (§14).
- **Capability/Authority Projection** — represents what the connected context may legitimately inspect, never grants anything (§18).
- **Diagnostics Projection** — represents available error/diagnostic information, live only (§21).
- **Presentation Boundary** — renders developer-facing information; technology-independent (§30).

## 12. State Ownership

| State | Authoritative Owner | Control Centre Treatment |
|---|---|---|
| Actor identity | Runtime (`ActorDirectory`/`ActorHost`) | Presented, never owned; sourced only from an already-known or durably-enumerable `ActorId` |
| Actor lifecycle | Runtime (`LifecycleGuardian`) | Presented where SDK exposure exists; never inferred or cached as truth beyond its own freshness window |
| Durable actor population | Runtime (Persistence Service) | Presented via `known_actors()`; **explicitly, permanently disclosed as partial**, never represented as complete actor discovery |
| Capability grants | Runtime (Capability Authority) | Presented per-actor; never granted, revoked, or inferred by the Control Centre |
| Effect state | Runtime (Effect Coordinator) | Presented per-known-id; never mutated, cancelled, or retried in Phase 1 |
| Supervision state | Runtime (Supervisor) | Presented only to the extent the Supervisor's own public surface allows (`is_registered`/`parent_of`); restart/failure history explicitly out of Phase 1 scope, a disclosed deliberate boundary, not a Control Centre omission |
| Durable state (`DomainState`) | Runtime (Persistence Service/`StorageBackend`) | Durability classification presented only; content never displayed or edited |
| Runtime status | Runtime itself | Presented; the Control Centre holds only a local, explicitly stale-markable copy (§14) |
| Diagnostics | Runtime/SDK error vocabulary (`ARCH-014` §12) | Presented, live/ephemeral only; never treated as persistent history (§21) |
| GUI navigation state | Control Centre (GUI-local) | Owned entirely by the Control Centre; no Runtime relevance |
| GUI preferences | Control Centre (GUI-local) | Owned entirely by the Control Centre |
| GUI grouping (Application) metadata | Control Centre (GUI-local, non-authoritative) | Owned by the Control Centre; explicitly never claims Runtime truth (§23) |

## 13. Connection Model

Phase 1 addresses **local Runtime connection only** — a single, already-known-location Runtime instance. Requirements: connection establishment; Runtime identity confirmation; connection loss detection; reconnection; incompatible-version detection (via `ARCH-014` §14's tier vocabulary, never independently inferred, `ARCH-015` §21 inherited); graceful degradation where an inspection capability is unavailable (§34's own Gap Analysis directly gates which capabilities can even be attempted). No protocol, transport, IPC mechanism, or serialization format is selected — each remains a downstream implementation decision.

## 14. Observation and Synchronization

Acceptable conceptual mechanisms, none mandated over another: initial snapshot on connection; explicit or periodic refresh; reconnect-triggered re-snapshot; stale-state detection and marking. **Binding guarantee the Control Centre may make:** displayed state reflects what was true as of its own last successful observation, explicitly timestamped or marked as such. **Guarantee it may never make:** that displayed state is current, unquestionable Runtime truth once any observation lag, disconnection, or partial-failure condition exists — direct, concrete application of `ARCH-015` `DPB-INV-12` (State of Record).

## 15. Runtime Discovery

Differentiated explicitly from Runtime *connection*: discovery (automatically finding available Runtime instances) is **not required for Phase 1** — `DES-004` does not establish a first-version need for it, and no evidence supports it as blocking. Preserved as a named future architectural capability (§35), not designed here.

## 16. Actor Discovery

Direct application of `DES-004` §35's own reconciled distinction. **Available:** durable-actor enumeration via `known_actors()`. **Not established:** global enumeration across durable and transient actors. The architecture requires the Control Centre to communicate this incompleteness honestly wherever a durable-actor list is presented — this is a binding presentation obligation, not merely a documentation note, directly enforcing `ARCH-015` §9's Non-Second-Runtime-adjacent honesty principle at the UX level.

**Enumeration semantics.**

- **Inclusion.** `known_actors()`/`known_durable_actors()` is understood, from its own accessor semantics as documented in `DES-004` §34's own evidence ("tells you which actors currently have durable state"), to reflect current durable-state possession at the moment of observation.
- **Ordering.** No evidence establishes any ordering guarantee. **Consumers MUST NOT derive semantic meaning from the order in which enumerated actors are returned or presented.**
- **Staleness.** Direct application of §14's general model: the enumeration represents the result of its most recent successful observation, not a continuously authoritative live registry. No time-based staleness bound is asserted — none is evidenced.
- **Completeness.** Durable actor enumeration is not global actor enumeration, and the Control Centre MUST continue to disclose this distinction wherever a durable-actor list is presented (§21's cross-cutting rule applies here as the general case).

**Termination vs. durable-state deletion (evidence-confirmed).** Actor execution/lifecycle status and durable-state possession are architecturally distinct dimensions, and `synapse-runtime`'s own current implementation confirms this precisely, not merely by inference:

- **Actor termination** (`ActorHost::terminate_instance`, `core/actor-host/src/internal.rs`) removes only the actor's mailbox and instance-level registration (`instances`, `live_instance_of`, `mailboxes`, `behaviors`). It has no reference to, and no effect on, durable-state storage — confirmed by exhaustive inspection: zero references to `persistence` anywhere in the `core/actor-host` crate.
- **Lifecycle-state transition to `Terminated`** (`LifecycleGuardian`, `core/lifecycle-guardian/`) governs only legal lifecycle-state progression. It likewise has no reference to, and no effect on, durable-state storage.
- **Durable-state deletion** (`Persistence::delete`) is invoked in production code exclusively through `Runtime::delete_actor_state` (`runtime/src/lib.rs:3170`) — a distinct, explicit, capability-authorized operation (requiring an explicit `Capability` argument, checked via a dedicated `authorize_persistence_operation` step) that produces its own complete audit trail (`persistence_deletion_requested`/`denied`/`authorized`/`completed` events). No code path connects actor termination to this operation automatically.

**Governing rule:** actor termination alone, without a separate, explicitly-authorized durable-state deletion, does not remove an actor from durable enumeration (`known_actors()`/`known_durable_actors()`). The Control Centre MUST NOT infer "actor no longer running" as equivalent to "actor removed from durable enumeration" — the two facts are independent, and only an explicit deletion (itself a distinct, separately capability-gated operation, unaffected by this clarification) changes durable-enumeration membership. Symmetrically, presence in durable enumeration does not imply the actor is currently running.

**Evidentiary basis, disclosed precisely:** `core/actor-host/src/lib.rs:149`, `internal.rs:183` (termination); `core/lifecycle-guardian/src/lib.rs` (lifecycle-state transitions); `services/persistence/src/internal.rs:231–237` (`PersistenceImpl::delete`/`known_actors`); `runtime/src/lib.rs:3170–3224` (`Runtime::delete_actor_state`, the sole production caller of `persistence.delete`). This is direct, positive source confirmation, independently re-verified twice this engagement's own review lifecycle — not conservative inference.

## 17. Actor Detail Projection

Given an already-known or durably-enumerated `ActorId`: identity and lifecycle projection require SDK-level exposure work confirmed by `DES-004` §34 to not yet exist (`Requires exposure` — no accessor is fabricated here; this document only establishes that such exposure, once built, projects legitimately through this boundary).

## 18. Capability Projection

The Control Centre may inspect and explain capability state (`CapabilityAuthority::bound_capabilities`, confirmed `Supported now`, `DES-004` §34) but MUST NOT grant, revoke, infer authority beyond Runtime truth, or create capability policy. Phase 1 is inspection/explanation only; the architecture's own extension point (§32) preserves a path for future capability-management operations without pre-designing them.

Subject to §21's cross-cutting partial-data-disclosure rule — where capability visibility is incomplete or unavailable for any reason, the Control Centre MUST NOT represent that absence as "no capability held." Absence-from-view and absence-of-authority are distinct facts; only the Runtime, never the Control Centre's own incomplete view, is authoritative for which is true.

## 19. Effect Projection

The Control Centre may present known effects, status, and relationship information given a known id (`EffectCoordinator`'s query surface, confirmed `Supported now`). Cancellation, retry, and re-execution are explicitly **not** Phase 1 functionality — each belongs behind future Runtime Control API architecture (§32).

Subject to §21's cross-cutting rule — where only some effect information is exposed, absence of a given effect from the Control Centre's own projection MUST NOT be represented as proof no such effect exists, unless the underlying SDK/Runtime source is known to guarantee completeness for that specific query.

## 20. Supervision Projection

`DES-004` §16 established that the Supervisor crate's own restart-count/failure-history absence is a **deliberate, documented design boundary**, not missing exposure. This architecture does not redesign supervision and does not attempt to work around that boundary. Only `is_registered`/`parent_of`-derived relationship information may be legitimately projected; restart/failure history remains unavailable and deferred pending a separate future decision about whether to revisit that boundary at all (`DES-004` §42).

The Control Centre MUST NOT present available relationship data (e.g., `parent_of`) in a way that implies a complete supervision picture. **Specifically: displaying a supervisor relationship without restart-count or failure-history context MUST be accompanied by an explicit indication that this history is unavailable by design**, not silently absent — the single most consequential application of §21's cross-cutting rule, since this is the projection most likely to mislead a developer into inferring completeness that does not exist.

## 21. Diagnostics

Live/current error and diagnostic information, consuming `ARCH-014` §12's developer-facing error vocabulary directly (`ARCH-015` `DPB-INV-09` inherited), may be presented. **The proposed live-session activity feed is excluded from Phase 1** — no Approved architecture currently establishes the semantics required to treat `AuditEmitter::drain`'s own destructive read as a legitimate, repeatable GUI data source, and this document does not invent them merely to add the feature. Diagnostics are never treated as, or silently converted into, persistent audit history.

**Cross-Cutting Partial-Data Disclosure Rule (extends `AI-12`, referenced by §16, §18, §19, §20 above).** *Whenever the Control Centre presents a partial projection of platform state — durable-actor enumeration (§16), capability visibility (§18), effect visibility (§19), supervision relationships (§20), or diagnostics (this section) — it MUST NOT imply that dimensions of Runtime truth outside its own current projection are absent from the underlying platform. Absence from the Control Centre's own view is never equivalent to absence of fact, unless the specific underlying SDK/Runtime source is independently known to guarantee completeness for that query.* This is stated once, here, as the general rule; §16/§18/§19/§20 each reference it rather than restating it, consistent with this document's own Minimal Architecture Principle (§7 Principle 3).

## 22. Audit Boundary

Persistent, GUI-queryable audit history remains a later architectural capability (`D4`, per the Sequencing Assessment), explicitly deferred. This architecture preserves future compatibility — it does not store a second, competing authoritative history merely for GUI convenience, and it does not design the eventual mechanism. Future compatibility is not authorization.

## 23. Application Grouping

Non-authoritative only, per `DES-004` `CC-01`. For Phase 1: **Application grouping is omitted or, if included, purely presentational**, using GUI-local metadata whose own authoring mechanism is explicitly, architecturally undecided (`DES-004` §42's own unresolved question, not solved here). No manifest, registry, Runtime-owned Application object, or authoritative application database is created.

## 24. Progressive Disclosure

Direct inheritance of `ARCH-015` `DPB-INV-05`/`DES-004` `CC-38`/`39`: simple first-use comprehension (the durable-actor browse path, `DES-004` §35) must be reachable without confronting supervision internals, raw audit event types, or `DomainState` encoding details; nothing learned at an overview level may require unlearning at a detail level.

## 25. Error Model

Direct consumption of `ARCH-014` §12 via `ARCH-015` `DPB-INV-09`. The Control Centre projects, never redefines, established error meaning. Presentation-specific contextualization (navigation, remediation copy) is permitted; a competing error taxonomy is not.

## 26. Compatibility

Reuses `ARCH-014` §14's tier vocabulary and `ARCH-015` `DPB-INV-10`'s own tier-vs-version-range distinction directly. The Control Centre must be able to state which Runtime/SDK version range it is valid for; it must not infer compatibility from a stability tier alone.

## 27. Security

No authority escalation; no implicit capability grant; no hidden mutation pathway (Phase 1 is read-only by architecture, §31, not merely by UI convention); no diagnostic information exposed beyond what the connected context's own legitimate authority already permits; least-authority projection as the default presentation posture. Authentication technology and remote-trust mechanisms remain out of scope — local-only for Phase 1 (§13) sidesteps, without resolving, the deeper remote-trust question.

## 28. Failure Model

Runtime unavailable; connection failure; Runtime shutdown mid-session; an actor disappearing between list observation and detail inspection; stale information; unsupported inspection (per §34's own Gap Analysis); permission/authority denial; SDK incompatibility; partial data availability; diagnostic source unavailable.

**Runtime restart** — distinguished explicitly from generic shutdown/unavailability. No evidence establishes a Runtime identity or continuity mechanism across a restart. **Governing rule:** the Control Centre MUST NOT assume continuity of any previously-observed state merely because it successfully reconnects to the same endpoint or environment after a Runtime restart. All previously cached/observed state remains subject to §14's own stale-state rules until successfully re-observed and revalidated against the (possibly new) Runtime instance. This document does not invent a Runtime identity protocol, generation counter, session token, or handshake mechanism to resolve this more strongly.

**Governing rule, throughout this section:** the architecture favors truthful incomplete state over fabricated certainty in every failure case — direct application of Principle 2 (§7).

## 29. Cross-Platform Boundary

Direct architectural requirement, sourced from `ARCH-008` §29's own explicit compatibility statement (Windows/Linux/macOS). No architecture decision here may unnecessarily bind the Control Centre domain to one operating system or presentation framework.

## 30. Presentation Technology Independence

The six internal boundaries (§11) are defined precisely so that domain logic never depends on any particular UI framework. No GUI framework, web framework, or UI toolkit is selected — Tauri, Electron, React, Vue, Svelte, Flutter, Qt, Avalonia, .NET UI, any Rust GUI framework, and browser-only architecture are all equally unselected by this document.

## 31. Phase 1 Read-Only Architecture

**Precise definition of "read-only," beyond hiding buttons:** no functionality within Phase 1 scope provides a GUI-originated pathway to Runtime mutation. This is enforced structurally, not by convention — every operation this architecture authorizes (§18–§21) is drawn exclusively from confirmed-`Supported-now`/`Requires-exposure` *observation* capabilities (`DES-004` §34); no operation requiring `ARCH-015` §11's Authority Projection test to pass for a *mutating* action is included in scope at all.

## 32. Future Runtime Control Integration

Extension point preserved, not designed: when a future Runtime Control API architecture (`D2`) is separately authored, reviewed, and approved, it integrates at the **Capability/Authority Projection** and **State Observation** boundaries (§11) precisely because those boundaries are already defined generically enough to accept new operation types without redesign — no API shape is specified here.

## 33. Future Global Discovery

Extension point preserved: when a future Runtime enumeration/discovery architecture (`D1`) is separately resolved, it integrates at the **Runtime Connection**/**State Observation** boundaries, extending the existing durable-actor-only observation model rather than replacing it.

## 34. Future Persistent Audit

Extension point preserved at the **Diagnostics Projection** boundary: when persistent audit-history architecture (`D4`) is separately resolved, it extends, rather than replaces, the live-diagnostics-only model this document establishes (§21–§22).

## 35. Future Multi-Runtime Support

Phase 1 assumes a single active Runtime connection (§13), directly supported by `DES-004` §22's own evidence. The **Runtime Connection Boundary** (§11) is defined as a boundary *type*, not a hard-coded single-instance assumption baked into other boundaries. Multi-Runtime orchestration itself is not architected here.

## 36. Architecture Invariants

| ID | Statement | Source |
|---|---|---|
| AI-01 | Runtime authority remains authoritative; the Control Centre never overrides it | `ARCH-015` §10 |
| AI-02 | The Control Centre is not a second Runtime | `ARCH-015` `DPB-INV-01` |
| AI-03 | Phase 1 is read-only by architectural scope, not UI convention | §31 |
| AI-04 | GUI state cannot become authoritative platform state | `ARCH-015` `DPB-INV-12` |
| AI-05 | Durable actor enumeration is not global actor enumeration, and must be disclosed as such wherever presented | `DES-004` §16/§34 |
| AI-06 | Known-id inspection is not discovery | `DES-004` §11 |
| AI-07 | Capabilities remain Runtime-authoritative; the Control Centre inspects, never grants | §18 |
| AI-08 | Effects remain Runtime-authoritative; the Control Centre inspects, never mutates, in Phase 1 | §19 |
| AI-09 | Live diagnostics are not automatically persistent audit history | §21 |
| AI-10 | Application grouping is non-authoritative unless a future, separate architecture establishes otherwise | §23 |
| AI-11 | Presentation technology does not own platform semantics | §30 |
| AI-12 | Unsupported or unavailable information must be represented truthfully, never fabricated — **and this obligation applies uniformly across every partial projection (§16, §18, §19, §20, §21), not only where a section happens to restate it** | §21's cross-cutting rule; §28 |
| AI-13 | Future control capability requires separately governed authority (`D2`) | §32 |
| AI-14 | Future discovery capability requires separately governed authority (`D1`) | §33 |
| AI-15 | Future audit capability requires separately governed authority (`D4`) | §34 |
| AI-16 | The Control Centre reaches SDK/Runtime operations directly; it must never be architecturally required to route through the CLI | `ARCH-015` §13 |

`AI-12`'s own restart-continuity application (§28) is a direct consequence of this same truthful-representation obligation, not a separate invariant.

## 37. Architecture Decision Table

| Decision | Selected Direction | Alternatives Considered | Evidence | Consequence |
|---|---|---|---|---|
| Phase 1 mutability | Read-only, structurally scoped | Include a small set of "safe" mutations — rejected, no genuine partial-mutation set survived the Authority Projection test | `DES-004` §26/§34 | No control-operation code path exists to secure or gate later |
| Connection scope | Local Runtime only | Local + remote from day one — rejected, no first-version evidence requires it | `DES-004` §22 | Remote-trust questions deferred, not resolved |
| Actor discovery | Durable-only, explicitly disclosed as partial | Fabricate a "best-effort" global list from partial data — rejected, violates `AI-12` | `DES-004` §35 | Actor Explorer remains genuinely incomplete until `D1` |
| Application grouping | Non-authoritative, GUI-local, mechanism undecided | Invent a manifest/config format now — rejected, exceeds this document's own authority | `DES-004` §9/§42 | Metadata-sourcing question remains open |
| Diagnostics scope | Live/ephemeral only | Include a session-scoped activity feed via `AuditEmitter::drain` — rejected, no Approved architecture establishes the semantics | `DES-004` §19 | Any activity-feed feature waits for `D4` |
| Presentation independence | Six-boundary domain/presentation split | Allow presentation-layer code direct Runtime/SDK access — rejected | §11/§30 | Framework selection remains fully deferred without domain rework risk |

## 38. DES-004 Requirement Mapping

| DES-004 Requirement Group | Disposition |
|---|---|
| `CC-02`/`CC-03` (Runtime status/compatibility) | Architecturally satisfied; implementation/exposure required |
| `CC-01`/`CC-04`/`CC-05` (Application grouping, active-actor counts, global listing) | `CC-01` architecturally satisfied at non-authoritative scope; `CC-04`/`CC-05` later architecture dependency (`D1`) |
| `CC-06`–`09` (actor identity/lifecycle, given known id) | Architecturally satisfied; implementation/exposure required |
| `CC-10`/`CC-11`/`CC-15`/`CC-17` (capability/effect display, current/live) | Architecturally satisfied, `Supported now` at data layer |
| `CC-16` (retry/idempotency classification) | Should priority, feasibility not independently confirmed beyond `CC-15`'s base surface |
| `CC-18` (capability/effect denial explanation) | **Split explicitly.** Current/display aspect: architecturally satisfied, `Supported now`, within the existing Phase 1 inspection boundary. Historical aspect (a persistent record of past denials): **Requires architecture** — depends on the separately deferred persistent audit architecture (`D4`), not resolved by this document |
| `CC-12`/`CC-20`/`CC-21` (supervision) | Architecturally satisfied for available relationship data only; restart/failure history explicitly outside Phase 1 |
| `CC-13`/`CC-22`/`CC-23` (durability) | Architecturally satisfied for classification/enumeration; no editing, ever |
| `CC-14`/`CC-24`–`27` (diagnostics/audit) | `CC-24`/`25` architecturally satisfied; `CC-26`/`27` later architecture dependency (`D4`) |
| `CC-19` (capability modification) | Outside Phase 1, deferred |
| `CC-28` (refresh/reconnect) | Architecturally satisfied |
| `CC-29`–`31` (cancel/restart/recovery/delete/app start-stop) | Later architecture dependency (`D2`), outside Phase 1 |
| `CC-32`–`34` (state-of-record) | Architecturally satisfied |
| `CC-35`/`36` (connectivity) | `CC-35` architecturally satisfied; `CC-36` deferred |
| `CC-37` (search) | Deferred |
| `CC-38`/`39` (Progressive Disclosure) | Architecturally satisfied |
| `CC-40`–`43` (security UX) | Architecturally satisfied |
| `CC-44` (cross-platform) | Architecturally satisfied |
| `CC-45`/`46` (accessibility) | Attribute-level requirement carried forward unmodified; standard question outside this document's own scope |
| `CC-47`/`48` (performance/Runtime-safety) | Architecturally satisfied in principle, no numeric target set |
| `CC-49` (no CLI substrate) | Architecturally satisfied (`AI-16`) |
| `CC-50` (documentation deep-link) | Outside Phase 1, deferred |

No requirement is silently dropped; every one of the fifty is accounted for above.

## 39. Phase 1 Capability Matrix

| Capability | Architecture Status | Runtime/SDK Support | Exposure Required | Phase 1 Eligible? | Later Dependency |
|---|---|---|---|---|---|
| Runtime status | Architected | Partial | Yes | Yes | — |
| Runtime identity | Architected | Partial | Yes | Yes | — |
| Durable actor listing | Architected | `known_actors()` | Yes | **Yes** | — |
| Global actor listing | Not architected here | None | — | No | `D1` |
| Actor detail | Architected | Partial | Yes | Yes | — |
| Lifecycle state | Architected | Not exposed | Yes | Yes | — |
| Capabilities | Architected | **Exists** | No | **Yes** | — |
| Effects | Architected | **Exists** | No | **Yes** | — |
| Supervision (relationship only) | Architected, bounded | Partial, deliberate limit | Yes | Yes, bounded | Restart/failure history: separate decision |
| Diagnostics/errors (live) | Architected | Sound (vocabulary) | No | Yes | — |
| Persistent audit history | Not architected here | Write-only | — | No | `D4` |
| Restart | Not architected here | Does not exist | — | No | `D2` |
| Recovery | Not architected here | Individual op only | — | No | `D2` |
| Deletion | Not architected here | Individual op only | — | No | `D2` |
| Effect cancellation | Not architected here | Semantics exist, exposure unconfirmed | — | No | `D2` |
| Application grouping | Architected, non-authoritative | None | Optional | Yes, if omitted or minimal | Full concept: separate decision |
| Runtime discovery | Not architected here | N/A | — | No | Future |
| Multi-Runtime support | Not architected here | N/A | — | No | Future |

## 40. Risks

GUI coupling to Runtime internals (mitigated: §10's strict SDK-only boundary); accidental second source of truth (mitigated: `AI-04`/`AI-05`/`AI-06`, §14's staleness discipline); stale state presented as current (mitigated: §14, `DPB-INV-12`); capability leakage (mitigated: §18, §27); premature control operations (mitigated: §31's structural, not conventional, read-only scoping); local-only assumptions becoming permanently unquestioned (mitigated: §35's boundary-type framing); GUI technology contaminating domain architecture (mitigated: §11/§30's six-boundary split); **partial durable-actor enumeration mistaken for complete discovery — understood as one instance of the general partial-data-disclosure risk §21's cross-cutting rule addresses, also covering capability/effect/supervision/diagnostics projections** (mitigated: `AI-05`, §21); diagnostic data mistaken for durable audit history (mitigated: `AI-09`, §21–§22); presentational Application grouping becoming accidental platform authority (mitigated: `AI-10`, §23); architecture overreach (mitigated: §6 Non-Goals); architecture under-specification (mitigated: §36's explicit invariant set plus four extension points, §32–§35).

## 41. Architecture Acceptance Tests

1. Can the first GUI display durable actors without claiming it displays all actors? — **Yes** (`AI-05`).
2. Can the GUI disappear entirely without affecting Runtime correctness? — **Yes** (`AI-01`/`AI-02`).
3. Can presentation technology be replaced without redefining Runtime authority? — **Yes** (§11/§30).
4. Can a future Runtime Control API be integrated without bypassing capability enforcement? — **Yes** (§32).
5. Can unavailable information be represented without inventing state? — **Yes** (`AI-12`, §28).
6. Can a future global-discovery capability be added without discarding the durable-actor-only model? — **Yes** (§33).
7. Can a future persistent-audit capability be added without redesigning the Diagnostics Projection boundary? — **Yes** (§34).
8. Does any Phase 1 capability require the CLI to be present? — **No** (`AI-16`).
9. Does the architecture require re-observation rather than assumed continuity after a Runtime restart? — **Yes** (§28, directly enforcing `AI-12`).

## 42. Deferred Architecture

Global actor/effect/capability enumeration architecture (`D1`); Runtime Control API architecture (`D2`); persistent audit-history architecture (`D4`); the Application-metadata-sourcing mechanism, if the Founder ever wants richer-than-minimal grouping (`D3`, conditional); a Runtime identity/continuity mechanism across restarts, if stronger-than-re-observation guarantees are ever wanted (§28); Runtime discovery mechanism; multi-Runtime orchestration; remote-Runtime connection/trust model; the specific accessibility compliance standard (`DES-002` R15, unmodified); presentation technology selection; packaging/distribution.

## 43. External Dependencies

`DES-004`'s own external dependencies (Diagnostics/Observability workstream, Documentation Platform, Developer Portal — boundary only) remain unchanged. New: the SDK exposure work §17/§38 name (identity/lifecycle projection), properly routed through `ARCH-014`'s own amendment process where genuinely new SDK surface is required, never invented here.

## 44. Governance Impact

This document occupies the same governance tier `ARCH-015` established for itself: architecture governing one Developer Platform component (Control Centre, Phase 1 only), subordinate to `GOV-018`'s constitutional layer and `ARCH-015`'s own cross-cutting boundary, superior to any future Engineering Work Order that implements it. No existing `ARCH`, `ADR`, `GOV`, or `STD` document is amended, superseded, or contradicted by this Filing.

## 45. Independent Review Readiness

*(Historical record, v0.1.0.)* Self-checked at Authoring against: `DES-004` completeness; `ARCH-014`/`ARCH-015` compatibility; Runtime authority; capability/effect/persistence/supervision boundaries; architecture leakage; technology leakage; future compatibility; Phase 1 scope integrity. Superseded in effect by the complete Independent Architecture Review and both Narrow Re-Reviews recorded in this document's own history (§49).

## 46. References

`GOV-018`, `ROAD-001`, `ACT-005`, `RES-008`, `ADR-0020`, `DES-003`, `DES-004` v0.2.1, `ARCH-015` v0.2.0, `ARCH-014` v0.8.0, `ARCH-007` v0.5.2, `ARCH-008` v0.5.0 (§17, §29, §30), `ARCH-011` v0.1.3, `ARCH-012` v0.2.0, the Control Centre Architecture Dependency & Sequencing Assessment, the ARCH-016 v0.1.0 Independent Architecture Review, the ARCH-016 v0.2.0 Narrow Architecture Re-Review, the ARCH-016 v0.2.1 Second Narrow Architecture Re-Review (all this engagement's own conversational record).

## 47. Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Architecture Authoring Draft, transforming `DES-004` v0.2.1's Founder-accepted requirements into eleven invariants and six decisions, per `GOV-013` §6.8. Independent Architecture Review: `REVISION REQUIRED` (0 Blocking, 0 Major, 3 Minor [`IAR-016-F01`–`F03`], 1 Observation [`IAR-016-F04`]). |
| 0.2.0 | 2026-08-10 | Denver Jacobs (Founder) | Narrow Architecture Correction applying exactly `IAR-016-F01`–`F04`: precised the `CC-18` requirement-mapping split; added durable-actor-enumeration semantics (inclusion, ordering, staleness); established one cross-cutting partial-data-disclosure rule (§21, extending `AI-12`), applied uniformly across §16/§18/§19/§20; added an explicit Runtime-restart scenario to the Failure Model. Narrow Architecture Re-Review: **FAIL** — `IAR-016-F02` only Partially Resolved (termination-vs-durable-deletion distinction implied but not explicit); one new Minor finding, `NAR-016-F01`. |
| 0.2.1 | 2026-08-10 | Denver Jacobs (Founder) | Further Narrow Architecture Correction applying exactly `NAR-016-F01`, evidence-confirmed by direct inspection of `synapse-runtime` source (`core/actor-host`, `core/lifecycle-guardian`, `services/persistence/src/internal.rs`, `runtime/src/lib.rs:3170`): §16 now explicitly states that actor termination alone does not remove an actor from durable enumeration — only a separate, capability-authorized `delete_actor_state` call does. Second Narrow Architecture Re-Review: **PASS** — zero remaining findings of any severity across the entire review chain, independently re-verified from primary source. Records Founder Architecture Approval (§49, Founder Declaration quoted verbatim) and constitutes this document's own Repository Filing. `status` transitions from `Draft` to **`Approved`**. |

## 48. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author (v0.1.0) | Denver Jacobs (AI-assisted) | Drafted | 2026-08-10 |
| Independent Architecture Review (v0.1.0) | — | `REVISION REQUIRED` — 0 Blocking, 0 Major, 3 Minor, 1 Observation | 2026-08-10 |
| Founder Disposition (v0.1.0 review) | Denver Jacobs, Founder | Accepted in full; Narrow Architecture Correction authorized | 2026-08-10 |
| Author (v0.2.0, Correction) | Denver Jacobs (Founder) | Corrected — four findings applied | 2026-08-10 |
| Narrow Architecture Re-Review (v0.2.0) | — | **FAIL** — `F02` Partially Resolved; `NAR-016-F01` (Minor) | 2026-08-10 |
| Founder Disposition (v0.2.0 review) | Denver Jacobs, Founder | Accepted in full; Further Narrow Architecture Correction authorized | 2026-08-10 |
| Author (v0.2.1, Further Correction) | Denver Jacobs (Founder) | Corrected — `NAR-016-F01` applied, evidence-confirmed | 2026-08-10 |
| Second Narrow Architecture Re-Review (v0.2.1) | — | **PASS** — zero remaining findings | 2026-08-10 |
| Approval Authority | Denver Jacobs, Founder (interim, per `GOV-003` §3.2 vacancy) | **Approved** (verbatim Founder Declaration recorded in §49) | 2026-08-10 |

## 49. Disposition

**Approved.** Architecture Authoring (v0.1.0) → Independent Architecture Review (`REVISION REQUIRED`, 3 Minor, 1 Observation) → Narrow Architecture Correction (v0.2.0) → Narrow Architecture Re-Review (`FAIL`, one Minor remaining) → Further Narrow Architecture Correction (v0.2.1) → Second Narrow Architecture Re-Review (`PASS`, zero remaining findings) → Founder Architecture Approval, recorded in full below.

**Founder Architecture Approval granted.** Denver Jacobs, Founder, 2026-08-10, recorded verbatim:

> "Founder Architecture Approval and Controlled Publication Authorization. ARCH-016 v0.2.1 — SynapseOS Control Centre Foundation Architecture (Phase 1: Read-Only) — receives Founder Architecture Approval and is accepted as the authoritative Phase 1 architecture for the SynapseOS Control Centre. The complete architecture review lineage is accepted: Architecture Authoring; Independent Architecture Review; Narrow Architecture Correction; first Narrow Architecture Re-Review — FAIL; Further Narrow Architecture Correction; Second Narrow Architecture Re-Review — PASS. IAR-016-F01, IAR-016-F02, IAR-016-F03, IAR-016-F04, and NAR-016-F01 are accepted as Resolved. No Blocking, Major, or Minor architecture-review finding remains open. Runtime remains authoritative. The Control Centre is a projection domain and does not become a second Runtime or an alternative persistence, lifecycle, capability, effect, supervision, or audit authority. Phase 1 remains structurally read-only. This Architecture Approval does not authorize the Control Centre to terminate, restart, recover, delete, grant, revoke, cancel, retry, or otherwise mutate Runtime state. Durable actor enumeration is accepted only within the boundaries established by ARCH-016. Durable enumeration is not global Runtime actor enumeration and must not be presented as complete Runtime actor truth. Actor execution/lifecycle state and durable-state possession remain distinct dimensions. Actor termination alone does not imply removal from durable enumeration, and presence in durable enumeration does not imply that an actor is currently running. The ARCH-016 partial-projection rule is accepted: information that is unavailable, unsupported, unexposed, or not observed must not be represented by the Control Centre as absent from Runtime truth, applied across actor discovery, capabilities, effects, supervision, and diagnostics. The ARCH-016 Runtime restart/reconnection model is accepted: previously observed state must be re-observed or revalidated following Runtime loss/reconnection; reconnection does not by itself establish Runtime continuity; no Runtime identity protocol, generation identifier, session token, or equivalent mechanism is approved by this decision. This approval does not approve or pre-authorize: authoritative/global Runtime enumeration or discovery architecture; Runtime Control API architecture; persistent audit-history architecture; authoritative Runtime-owned Application architecture; Runtime identity/continuity architecture beyond the re-observation model established by ARCH-016; multi-Runtime orchestration/control architecture — each remains a separately governed future workstream. The identified SDK exposure requirements are accepted as downstream exposure/implementation prerequisites rather than unresolved ARCH-016 architecture defects; their existence does not reopen ARCH-016. No GUI technology is selected by this approval — no approval is granted here for Tauri, Electron, Flutter, Qt, React, Vue, Svelte, another desktop/web framework, transport mechanism, persistence technology, or equivalent implementation choice. The positive GUI EWO gate assessment is accepted: following successful controlled publication of ARCH-016 v0.2.1, the first Phase 1 read-only Control Centre GUI Engineering Work Order may be considered as the next downstream engagement, subject to the identified SDK exposure prerequisites and normal governance; this declaration does not itself author or approve that EWO. GUI implementation is not authorized by this declaration. Controlled Publication is authorized: ARCH-016 v0.2.1 may proceed to Controlled Publication as the next separate engagement, authorized only to perform the mechanically necessary publication work required to materialize this Founder Architecture Approval faithfully, validate the resulting artifact, stage the intended file only, create the controlled commit, perform remote-safety verification, push normally without force, and verify the published remote artifact. Controlled Publication must not alter the substantive architecture approved here."

This Filing (v0.2.1) records that approval and constitutes this document's own Repository Filing. Concrete Control Centre GUI implementation remains future, separately authorized engineering (§42); no EWO is authorized by this Filing.
