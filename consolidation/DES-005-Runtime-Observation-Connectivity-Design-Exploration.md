---
document_id: DES-005
title: "SynapseOS Runtime Observation / Connectivity: Design Exploration"
version: 0.2.0
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
  and DES-001/DES-002/DES-003/DES-004 already occupy it: an
  evidence-to-decision synthesis document that precedes, and directly
  informs, a later binding artifact (a future Runtime Observation /
  Connectivity Architecture) without itself being architecture,
  governance, or an engineering authorization. This placement is a
  disclosed, narrow convenience, not a documentation-hierarchy
  redesign; formal registration of a "DES" family in STD-001, if ever
  wanted, is a separate, future, independently-authorized task, not
  performed here or implied by this placement.
reviewed_by: >
  Design Approval Review of DES-005 v0.1.0 (conversational record;
  not a filed repository document), conducted per GOV-013 §6.5 —
  verdict: REVISION REQUIRED (0 Critical, 2 Major [DAR-005-F02 —
  ROC-05's Should-level priority was inconsistent with this
  document's own stated sensitive-data risk rationale; DAR-005-F04 —
  effect-payload minimization was left as an unenforceable body-text
  recommendation rather than a traceable requirement], 4 Minor
  [DAR-005-F01 — ROC-02's separation from a future Runtime Control
  API was not reinforced as a closed, enumerable operation set,
  leaving a generic-extensibility erosion path open; DAR-005-F03 —
  ROC-08's query/request-response requirement was worded as an
  exclusivity mandate the evidence did not support; DAR-005-F05 —
  ROC-11's multiple-simultaneous-client requirement was worded as a
  Phase 1 mandate the evidence did not support; DAR-005-F06 — ROC-01
  was misclassified Inherited when applying an existing
  Control-Centre-scoped rule to a new, not-yet-conceived boundary
  required genuine design judgment], 0 Observation). Narrow Design
  Correction (v0.1.0 -> v0.2.0) applied exactly DAR-005-F01 through
  F06: ROC-02 now requires a closed, enumerable read-only operation
  set, not a generic/extensible dispatch mechanism (F01); ROC-05
  raised from Should to Must, with mutation authority and observation
  authorization explicitly distinguished as separate concerns (F02);
  ROC-08 reworded from an exclusivity mandate to a sufficiency
  requirement, preserving Architecture Authoring's own freedom to add
  optional future push (F03); ROC-19 added, requiring effect-payload
  minimization by default at Must level (F04); ROC-11 reworded from a
  mandate to a non-foreclosure rule (F05); ROC-01 reclassified
  Inherited -> Derived (F06). Net requirement count: nineteen
  (eighteen retained, one added). Narrow Design Re-Review of v0.2.0
  (conversational record) verdict — PASS: all six findings
  independently re-verified as Resolved via direct re-reading of the
  corrected sections and their cross-references; zero
  correction-introduced or newly-revealed defects found; Alternative
  A, the ownership model, the standalone product model, local-only
  scope, and the Runtime identity/continuity rules confirmed
  unaltered; zero architecture or technology leakage found in any of
  the six corrected sections. Denver Jacobs, Founder, accepted
  DES-005 v0.2.0 as the final Runtime Observation / Connectivity
  Design Exploration / requirements baseline on 2026-08-10, confirmed
  Alternative A (Query-oriented local observation service) as the
  preferred design direction for subsequent Architecture Authoring,
  and separately authorized this Repository Filing and Runtime
  Observation / Connectivity Architecture Authoring as the next
  post-publication workstream; this acceptance is not Architecture
  Approval and does not itself authorize technology selection, SDK or
  Runtime implementation, Control Centre implementation, or any
  correction, re-review, or approval of EWO-028.
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.4-§6.7, the Design Exploration / Design Approval Review / Design Correction / Narrow Design Re-Review stages this document's own lifecycle follows exactly)
  architecture:
    - ARCH-016 (v0.2.1, Approved — Control Centre Foundation Architecture; §13/§14/§28 — the staleness, reconnection, and no-identity-mechanism rules this exploration inherits rather than re-derives)
    - ARCH-015 (v0.2.0, Approved — Developer Platform Boundary Architecture; cross-cutting invariants — Runtime Authority, State of Record, Capability Security, Generalization — extended throughout, never redefined)
    - ARCH-014 (v0.8.0, Approved — Synapse SDK Architecture; SDK mediation, tier, and error/compatibility vocabulary consumed, never duplicated)
  consolidation:
    - DES-004 (v0.2.1, Draft, Founder-accepted, filed — Control Centre Design Exploration; the product requirement this gap blocks)
  work-orders:
    - EWO-028 (Draft, unfiled — Phase 1 Read-Only Control Centre GUI Engineering Work Order; its Independent Engineering Work Order Review found IER-028-F01 (Blocking) and IER-028-F02 (Minor), the direct evidentiary trigger for this exploration; remains blocked pending this workstream's own architecture and implementation)
  adrs:
    - ADR-0020 (v0.1.0, Approved — Disposition of the Unrecoverable ARCH-013 Draft)
  source_artifacts:
    - "Direct source inspection of synapse-runtime (EWO-028's own Independent Engineering Work Order Review, re-confirmed this engagement): runtime/Cargo.toml, runtime/src/main.rs, core/capability-authority/src/lib.rs:56 (Capability's own lack of Serialize/Deserialize derivation), exhaustive repository-wide IPC/RPC search, services/http-provider's confirmed outbound-only nature"
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# DES-005 — SynapseOS Runtime Observation / Connectivity: Design Exploration

> **Founder Acceptance Notice (2026-08-10).** Denver Jacobs, Founder, has reviewed and accepted this document, at this version (v0.2.0), as the current Runtime Observation / Connectivity requirements and design baseline, following the complete review/correction/re-review lifecycle recorded in full in the `reviewed_by` field above. Consistent with `DES-001`–`004`'s own established precedent, this document's own tracked `status:` remains **Draft** — Design Exploration output in this corpus carries no independent approval authority of its own and uses no `Approved` status or formal Approval Status table; Founder disposition is recorded here and in `reviewed_by` instead. **Alternative A — Query-oriented local observation service — is accepted as the preferred design direction for subsequent Architecture Authoring; no transport, protocol, serialization, IPC technology, framework, library, endpoint representation, or implementation mechanism is selected. This acceptance is the requirements/design baseline only. It is not Architecture Approval and does not authorize Runtime Observation / Connectivity Architecture Authoring, technology selection, SDK implementation, Runtime implementation, Control Centre implementation, or any correction, re-review, or approval of EWO-028.** Runtime Observation / Connectivity Architecture Authoring is separately authorized as the next post-publication workstream; it does not begin as part of, or automatically following, this filing.

*(Section numbering below begins at §2 and preserves the numbering used throughout this document's own engagement and review history — §1, §60, and §61 were process/reporting sections specific to the chat-delivered engagement [Document Control, Revision History, Approval Status], not part of the design content itself, and are represented instead by the frontmatter, this Founder Acceptance Notice, and the Revision History section at the end, per `DES-001`–`004` precedent.)*

## 2. Purpose

Answer, at design level only, `GOV-013` §6.4's question — *what should be built?* — for the architectural gap `IER-028-F01` discovered: how an authorized external developer tool may observe an independently running local SynapseOS Runtime, without acquiring mutation authority, and without the Control Centre becoming the Runtime host.

## 3. Trigger

`EWO-028`'s own Independent Engineering Work Order Review found, through direct source inspection (`runtime/Cargo.toml`, `runtime/src/main.rs`, exhaustive repository-wide IPC/RPC search, `services/http-provider`'s confirmed outbound-only nature), that no mechanism exists anywhere in Approved architecture or current implementation for a separate process to observe a running Runtime. The Founder directed that the standalone product model be preserved and this gap resolved through a narrow architecture path, not by redefining the Control Centre as an in-process Runtime host.

## 4. Authority

`DES-004` v0.2.1 (the product requirement this gap blocks); `ARCH-015` v0.2.0 (the cross-cutting boundary this new boundary must extend, not violate); `ARCH-016` v0.2.1 (the Control Centre architecture whose §13/§14/§28 already, correctly anticipated staleness/reconnection/no-identity-mechanism rules this exploration inherits rather than re-derives); `ARCH-014` v0.8.0 (SDK mediation and tier rules); `EWO-028` and its Independent Engineering Work Order Review (the direct evidentiary trigger).

## 5. Background

`ARCH-016` §13 characterized "protocol/transport/IPC mechanism" as a downstream implementation decision. `EWO-028`'s review found this characterization itself insufficiently skeptical: the deeper question — whether *any* mechanism exists, even conceptually, for cross-process Runtime observation — was never asked at any prior stage of this lifecycle (`DES-004`, `ARCH-016` authoring, or either of `ARCH-016`'s own review passes). Direct source inspection confirms the answer is no.

## 6. Problem Statement

> **How may an authorized external developer tool observe an independently running local SynapseOS Runtime without acquiring Runtime mutation authority and without making the Control Centre itself the Runtime host?**

## 7. Existing Runtime Execution Model

A `Runtime` object is created via `Runtime::bootstrap`/`bootstrap_with_config` and owned entirely by the process that constructs it — every existing usage (`DX-001`, every test in `runtime/src/lib.rs`) is in-process. The Runtime stays alive only as long as its owning process retains the object; shutdown is an explicit, synchronous call. **No persistent Runtime daemon/server exists** — the sole executable (`runtime/src/main.rs`) bootstraps and immediately shuts down. No process-identity, endpoint, listener (local or remote), request/response protocol, subscription mechanism, client-authentication mechanism, or cross-process capability representation exists anywhere in the current implementation.

## 8. Existing Observation Surfaces

| Information | Authoritative Owner | Existing Query Surface | In-Process? | SDK Exposed? | Cross-Process Accessible? |
|---|---|---|---|---|---|
| Runtime state | Runtime | `Runtime::state()` | Yes | Partial | **No** |
| Durable actor enumeration | Persistence Service | `known_actors()`/`known_durable_actors()` | Yes | No | **No** |
| Actor identity (forward) | Actor Directory | `ActorDirectory::lookup(name)` | Yes | No | **No** |
| Actor lifecycle (current state) | Lifecycle Guardian | Internal only, not public | Yes | No | **No** |
| Capabilities | Capability Authority | `bound_capabilities(actor)` | Yes | No | **No** |
| Effects | Effect Coordinator | Full query surface | Yes | No | **No** |
| Supervision | Supervisor | `is_registered`/`parent_of` | Yes | No | **No** |
| Diagnostics/errors | SDK error vocabulary (`ARCH-014` §12) | Live, in-process | Yes | Partial | **No** |
| Runtime version/count | None | Does not exist | — | No | **No** |

**Every row shares the identical gap: the data exists, in-process; none of it has ever crossed a process boundary.** This confirms `IER-028-F01`'s own finding precisely — the problem is uniformly the missing boundary, not any individual missing data source.

## 9. Product Model

Per Founder direction: Runtime and Control Centre are independently running processes, connected by a to-be-architected read-only observation boundary. In-process Runtime hosting by the Control Centre is rejected for this workstream.

## 10. Scope

Design-level exploration only: the conceptual boundary, its ownership, its authority model, its lifecycle semantics, and the requirement set Architecture Authoring must satisfy. No protocol, transport, or technology is selected.

## 11. Non-Goals

Runtime mutation; Runtime Control API (`D2`); actor terminate/restart/recover; durable-state deletion; capability grant/revoke; effect cancel/retry/re-execute; persistent audit streaming/history; remote Runtime management; multi-Runtime orchestration; cloud connectivity; Runtime clustering/placement; global actor enumeration (`D1`); authoritative Application architecture (`D3`); GUI technology selection; GUI implementation; specific transport selection; serialization-format selection.

## 12. Runtime Authority

Unchanged, directly inherited: the Runtime remains sole authority for execution, capability, effect, durability, supervision. The observation boundary transports projections of that authority; it creates no second state of record, no competing authority of any kind (`ARCH-015` §10/§11, `ARCH-016` §9, both directly extended, never redefined).

## 13. Observation vs. Control

**Structural, not conventional**, per the identical discipline `ARCH-016` §31 already established for the Control Centre's own read-only scope. The Observation Boundary's own interface contract defines **zero mutating operations** — not "mutating operations gated behind a permission flag," but a protocol that literally has no verb capable of changing Runtime state. This separation is robust against future erosion, not merely correct at time of writing: **the Runtime Observation Boundary exposes a closed, enumerable set of explicitly defined read-only observation operations. It MUST NOT expose a generic arbitrary-command, generic method-dispatch, generic capability-invocation, or equivalent extensible operation capable of being repurposed into Runtime mutation** (`ROC-02`). A future Runtime Control API (`D2`) is never created by adding a mutating case to this boundary's own dispatch surface — it must be an entirely separate boundary/protocol (§45), never an extension of this one with elevated permissions added later. No operation name, protocol message, or implementation endpoint is defined here; this is an enforceability requirement, not API design.

## 14. Observation Boundary

A first-class architectural concept is warranted: a **Runtime Observation Service** — Runtime-side, responsible for accepting an observation connection, determining compatibility, answering read-only queries, and refusing (structurally, by having no such operation) anything else. Client-side, a corresponding SDK-mediated observation client abstraction (§36).

## 15. Ownership

Per `ARCH-015` §7's own ownership model, tested against §39 (Generalization): the Runtime-side serving component belongs to the Runtime tier (a new, narrowly-scoped service, analogous in kind to Persistence/Supervisor/Effect Coordinator — each an existing, independent Runtime-tier service). The client-facing API belongs to the SDK (`ARCH-014`), mediating access exactly as it already mediates every other Runtime capability. The Control Centre (and any future Developer Platform client) consumes only the SDK's own client abstraction — never the Runtime-side service directly. This is `ARCH-016` §10's own "Runtime → SDK → Control Centre" chain, with this new boundary slotting in beneath the SDK layer as the mechanism that makes cross-process access possible at all.

## 16. Least Authority

Direct source confirmation: `Capability` (`core/capability-authority/src/lib.rs:56`) carries no `Serialize`/`Deserialize` derivation anywhere — it is exclusively an in-process Rust value with no existing cross-process representation. Given §13's own structural read-only guarantee, **no capability representation needs to cross the process boundary for Phase 1 at all** — since the protocol itself contains no mutating operation, there is nothing a capability could authorize that the protocol doesn't already unconditionally refuse to offer.

This resolves *mutation authority* only. It does not, and must not be read to, imply that *observation itself* requires no authorization. Two structurally distinct questions are involved: mutation authority is absent structurally (§13) — no mechanism exists to grant it, because no operation exists to authorize; observation authorization remains mandatory (§17, `ROC-05`) — because read-only Runtime information may itself be sensitive, a separate concern this resolution must not collapse into the first. `IER-028-F02` is resolved at design level by these two requirements jointly (disposition: §58), not returned to `EWO-028` as a narrow correction.

## 17. Trust Boundary

A distinct, real question survives even with zero mutation risk: **should observation itself be unconditionally available to any local process?** Runtime state (capability lists, effect payloads, diagnostics) may be sensitive even read-only. **`ROC-05`: an observation client MUST pass an appropriate local trust/authorization check before access to Runtime observation projections is granted.** This requirement remains technology-neutral — no OS-user permission scheme, token, certificate, credential, shared secret, capability, filesystem-permission mechanism, or authentication library is chosen; each remains an Architecture/Technology question (§53/§54).

## 18. Local-Only Boundary

Phase 1 is explicitly same-machine only. This materially simplifies trust (same-machine, same-OS-user is a defensible minimal boundary), discovery (no network discovery needed), and attack surface (no network hardening required). Remote observation remains a disclosed, undesigned future extension (§46).

## 19. Discovery

Distinguished from actor discovery (`D1`, unrelated). Given Phase 1's local-only scope and `DES-004`'s own Core User Journey (developer starts Runtime, *then* opens Control Centre — both user-initiated), **automatic Runtime discovery is not required.** A well-known local endpoint, supplied by the user or by launch tooling, is sufficient (`ROC-06`). The exact mechanism (fixed local address, file-based handle, environment variable) is a Technology Question (§54), not decided here.

## 20. Connection Lifecycle

Required conceptual states: `Disconnected`, `Connecting`, `Connected`, `Incompatible`, `Unauthorized`, `Unavailable`, `Reconnecting`. Each is a semantic state, not a UI label — `ARCH-016` §14/§28's own staleness and restart rules apply directly to transitions among them.

## 21. Runtime Identity / Continuity Boundary

Unchanged from `ARCH-016` §28: this boundary supplies enough information to establish "I am currently reachable to a Runtime," never "this is the same Runtime instance as before." No identity/generation mechanism is designed here, exactly as `ARCH-016` itself declined to invent one.

## 22. Observation Sessions

A lightweight session concept is warranted, minimally: the scope within which compatibility has been negotiated and observation requests are considered valid. **No authority or capability content belongs in a session** (§16). Ending a connection ends the session; no state persists across sessions — directly consistent with §21's own non-continuity rule.

## 23. Query / Subscription Model

**Phase 1 functionality MUST be achievable using query/request-response alone; no Phase 1 capability may depend on a push/subscription mechanism** (`ROC-08`). This preserves Architecture Authoring's own freedom to include an optional push/subscription capability alongside query/response, should it later find good reason to, without making any Phase 1 capability depend on it. Justification: matches `ARCH-016` §14's own already-established "snapshot + explicit/periodic refresh" model exactly, requires no new Runtime-side push infrastructure (none exists today, confirmed §7-§8), and is not required by any `DES-004` requirement (every Phase 1 requirement tolerates staleness-marked, refresh-driven data). Subscription/push remains an explicit, undesigned future extension (§44) a query-based protocol can gain later without redesign — the reverse is not as clean.

## 24. Projection Semantics

Each query result is independently answered; no cross-domain atomic snapshot is promised (no Runtime mechanism provides one, and none is required — `ARCH-016` §14 already established this exact discipline for the in-process case, now extended unmodified across a real process boundary).

## 25. Partiality / Staleness

`ARCH-016` §21's cross-cutting partial-data-disclosure rule must survive the boundary intact for every projection category it transports — empty, unavailable, unsupported, partial, stale, disconnected must each remain independently representable, never collapsed into a single null/empty/false (`ROC-10`).

## 26. Compatibility

The boundary must communicate protocol/SDK/Runtime compatibility sufficient for a client to know whether it can safely interpret responses (`ROC-15`), consuming `ARCH-014` §14's existing tier vocabulary and `ARCH-016` §26's tier-vs-version-range distinction — not inventing a new versioning scheme. Exact negotiation mechanism deferred (§54).

## 27. Error Semantics

Errors crossing the boundary must be classifiable consistently with `ARCH-014` §12's developer-facing error principles: connection failure, authorization failure, unsupported query, unavailable information, Runtime-side failure, compatibility failure, malformed request, timeout — distinguished categories, no competing taxonomy invented (`ROC-16`).

## 28. Durable Actor Projection

Transported, not redefined: `ARCH-016` §16's full semantics (durable-state possession, not liveness; not global truth; termination ≠ deletion; no ordering guarantee; potentially stale) carry through this boundary unmodified.

## 29. Capability Projection

Transported read-only; §16's own least-authority resolution applies — the projection itself carries no authority, only information. `ARCH-016` §18's "absence ≠ none" rule is preserved.

## 30. Effect Projection

Transported read-only; no cancel/retry/re-execute path exists in the protocol at all (§13). **Effect payload content MUST NOT be transmitted across the Runtime Observation Boundary by default. Observation is limited to effect metadata (identity, status, provider, actor association, timestamps where established) necessary for legitimate developer inspection, unless a future, separately authorized design/architecture decision explicitly permits additional payload disclosure** (`ROC-19`). No exact field schema is frozen here, and no claim is made that every listed metadata field currently exists — the binding rule is that payload disclosure is not the default.

## 31. Supervision Projection

Transported read-only, bounded exactly as `ARCH-016` §20 already establishes (`is_registered`/`parent_of` only; restart/failure history remains unavailable, disclosed).

## 32. Diagnostics Projection

Transported, live/current only, per `ARCH-016` §21 — this boundary is explicitly not a streaming audit system (§Non-Goals).

## 33. Runtime Safety

The Runtime-side observation service must be protected against resource exhaustion from query volume, large actor populations, multiple simultaneous observers, and abandoned/slow clients (`ROC-13`). This protection requirement applies regardless of how many observers are ultimately supported — even a single client can issue harmful query volume; §35's own non-foreclosure framing does not weaken this requirement. No numeric limit is invented here — the requirement is protection, not a specific threshold.

## 34. Failure Isolation

A crashed, hung, disconnected, or malformed-request-sending observation client must not affect Runtime availability, correctness, or performance beyond §33's own resource-protection bounds (`ROC-14`). The Runtime must not become dependent on any observer's own liveness.

## 35. Multiple Clients

**The architecture MUST NOT preclude multiple simultaneous read-only observation clients, but Phase 1 implementation is not required to support more than one active observation client unless later requirements establish that need** (`ROC-11`). Read-only queries carry no conflict risk with each other; foreclosing multiple observers would arbitrarily block legitimate future CLI/diagnostics-tool use without any authority-based justification, while not mandating immediate multi-client correctness avoids imposing a present engineering obligation no current requirement demands.

## 36. SDK Role

The SDK mediates: it owns the client-facing observation API, connection-lifecycle abstraction, compatibility handling, projection types, and error translation — it does not itself own Runtime truth, only its own faithful transport (`ARCH-014`, unchanged).

## 37. Runtime Role

The Runtime owns: authoritative query execution, connection/authorization validation (§17), and the actual projections served. It owns no GUI or Control-Centre-specific presentation concept.

## 38. Control Centre Role

The Control Centre initiates and uses an observation connection via the SDK, maintains non-authoritative observed state (`ARCH-016` §12, unchanged), and displays freshness/partiality truthfully. It does not define Runtime semantics, own authoritative state, become a protocol authority, or bypass the SDK.

## 39. Developer Platform Generalization

Tested directly against `ARCH-015` §35's own generalization principle: **yes**, this boundary makes identical architectural sense for a future CLI, IDE plugin, or diagnostics tool as it does for the Control Centre — nothing in its design is Control-Centre-specific. Architected as a general Developer Platform observation boundary, not a private Control-Centre tunnel (`ROC-17`), consistent with `ARCH-015` itself declining to design anything CLI-specific that a broader rule already covers.

## 40. Cross-Platform Constraints (Technology-Neutral)

Same-machine process communication; request/response (minimum) with room for future push; connection lifecycle; structured typed information; classifiable errors; compatibility negotiation; local trust enforcement; bounded resource use; Windows/Linux/macOS support; testability. No mechanism selected.

## 41. Security Requirements

Least authority (achieved structurally, §16, distinguished from observation authorization, §17); local access restriction (`ROC-05`); no mutation authority (§13, enforced by a closed, enumerable operation set, `ROC-02`); sensitive-data-minimization for effect payloads (`ROC-19`); denial-of-service resistance (§33); malformed-request isolation (§34); no implicit blanket trust merely from being local, absent an explicit basis (§17).

## 42. Deployment / Enablement

Whether observation is always-present, developer-mode-only, or explicitly opt-in is a genuine open question — recorded as an Architecture Question (§53), not resolved here; no evidence in `DES-004`/`ARCH-016` mandates one answer over another.

## 43. Public API Considerations

Whether the resulting SDK-facing observation client belongs at Public, Supported, Experimental, or Internal tier (`ARCH-014` §8) is an Architecture Authoring decision, not made here (`ROC-18`).

## 44. Evolution

Compatibility negotiation (§26) is the mechanism by which future protocol evolution (new projections, deprecated fields) avoids forcing lockstep upgrades — exact versioning scheme deferred, principle only stated here.

## 45. Relationship to D2

The Observation Boundary and the future Runtime Control API (`D2`) **must remain architecturally separate** (§13). This separation is enforced by the closed-operation-set requirement (`ROC-02`) — `D2`, when it exists, cannot be created by adding a mutating case to this boundary's own dispatch surface, because this boundary's own dispatch surface is required to be closed and enumerable, not generically extensible. They may share underlying connection/session/lifecycle *concepts* at a future Architecture Authoring stage's own discretion, but `D2` must never be implemented as "the observation protocol plus a write permission" — that would silently collapse the structural Observation/Control separation this exploration's entire design rests on. `D2` remains its own, entirely separate future boundary/protocol. **This is itself a binding requirement (`ROC-02`), not merely a stylistic preference.**

## 46. Relationship to Future Remote Runtime

Local-only for Phase 1 (§18); nothing in this design's own conceptual boundary (connection/session/compatibility/projection) is inherently local-only in its *semantics*, only in its *initial scope* — a future remote extension would extend, not replace, these concepts. Not designed further here.

## 47. Design Alternatives

**Alternative A — Query-oriented local observation service** *(recommended, §49)*: Runtime hosts a narrow, read-only, local-only request/response service; SDK provides a client wrapper; no subscription; minimal new Runtime-side surface.

**Alternative B — Session-oriented observation service with optional update stream**: adds Runtime-side push/subscription infrastructure for live updates. More responsive UX, but requires wholly new Runtime-side notification infrastructure (confirmed absent, §7-§8) for a capability no `DES-004` requirement demands — every Phase 1 requirement already tolerates refresh-driven staleness.

**Alternative C — SDK-mediated local observation gateway with a broker component**: functionally similar to A, but introduces an intermediary broker process/component between SDK and Runtime. Adds a component and a hop not justified by any current requirement.

| | Architectural fit | Runtime authority | Least authority | SDK fit | Control Centre fit | Complexity | Runtime impact | Testability | Cross-platform | Future CLI/IDE reuse | Future D2 compatibility | Security |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| A | Strong | Preserved | Free (§16) | Direct | Direct | Lowest | Lowest new-surface risk | High | No inherent issue | Strong (§39) | Clean separation | Strong |
| B | Strong, but adds scope | Preserved | Free | Direct | Direct, better UX | Highest — new infra | Highest — persistent connections | Moderate — async complexity | No inherent issue | Strong | Clean separation | Strong, larger surface |
| C | Moderate — extra component | Preserved | Free | Indirect (extra hop) | Indirect | Higher than A | Isolated by broker, but new component to secure | Moderate | No inherent issue | Strong | Clean separation | New component to secure |

## 48. Trade-off Matrix

See §47's own table — Alternative A dominates on complexity, Runtime impact, and directness without sacrificing any Phase 1 requirement; B and C each add scope or components unjustified by current evidence.

## 49. Preferred Direction

**Alternative A**, preferred because: (1) it is the minimum architecture necessary to unblock `EWO-028`'s actual acceptance criteria, none of which requires push/subscription; (2) it introduces the smallest new Runtime-side surface, minimizing future Architecture Review risk; (3) it preserves a clean path to Alternative B's own subscription capability later, without requiring rework. **Unresolved, left to Architecture Authoring:** the exact Runtime-side service boundary's own internal shape; the trust/authorization mechanism (§17); the discovery-endpoint mechanism (§19); the compatibility-negotiation mechanism (§26); the SDK tier classification (§43). **Left to a later Technology Evaluation:** the concrete transport (§54).

## 50. Normative Requirements

| ID | Statement | Class | Priority | Source |
|---|---|---|---|---|
| ROC-01 | Runtime MUST expose a local, read-only observation surface reachable by a separate process, structurally incapable of mutation | Architectural | Must | `IER-028-F01`; `ARCH-016` §31 generalized |
| ROC-02 | The observation boundary MUST remain architecturally separate from any future Runtime Control API (`D2`), and MUST expose a closed, enumerable set of explicitly defined read-only operations — not a generic/extensible dispatch mechanism | Architectural | Must | New, §13/§45 |
| ROC-03 | Observation MUST be local-only for Phase 1; remote observation explicitly deferred | Functional | Must | Derived, §18 |
| ROC-04 | No cross-process capability representation is required for Phase 1, since the protocol itself defines no mutating operation | Security | Must | Resolves `IER-028-F02` (mutation-authority component) |
| ROC-05 | An observation client MUST pass an appropriate local trust/authorization check before access to Runtime observation projections is granted | Security | Must | New, §17; resolves `IER-028-F02` (observation-authorization component) |
| ROC-06 | Automatic Runtime discovery is NOT required; a well-known local endpoint suffices | Functional | Must | Derived, §19 |
| ROC-07 | Connection MUST NOT be represented as establishing Runtime identity/continuity beyond current reachability | Architectural | Must | Inherited, `ARCH-016` §28 |
| ROC-08 | Phase 1 functionality MUST be achievable using query/request-response alone; no Phase 1 capability may depend on a push/subscription mechanism | Architectural | Must | Derived, §23 |
| ROC-09 | Each query result MUST carry its own freshness information; no atomic cross-domain snapshot is promised | Architectural | Must | Inherited, `ARCH-016` §14 |
| ROC-10 | The boundary MUST preserve every `ARCH-016` §21 partial-projection distinction, for every transported category | Content/Architectural | Must | Inherited |
| ROC-11 | The architecture MUST NOT preclude multiple simultaneous read-only observation clients, but Phase 1 implementation is not required to support more than one active client unless later requirements establish that need | Architectural | Must (non-foreclosure only) | New, §35/`ARCH-015` §39 |
| ROC-12 | The client-facing API MUST be SDK-mediated; no Developer Platform tool accesses the Runtime-side service by any other means | Architectural | Must | Inherited, `ARCH-015` §12/§13 |
| ROC-13 | The Runtime-side service MUST be protected against resource exhaustion from observation load, regardless of client count | Architectural | Must | New, §33 |
| ROC-14 | A failing observation client MUST NOT affect Runtime availability/correctness beyond `ROC-13`'s own bounds | Architectural | Must | New, §34 |
| ROC-15 | The boundary MUST communicate compatibility sufficient for safe client interpretation | Compatibility | Must | Derived, `ARCH-016` §26 |
| ROC-16 | Errors crossing the boundary MUST be classifiable per `ARCH-014` §12 principles | Content | Must | Inherited, extended |
| ROC-17 | The boundary MUST be designed generally enough for any legitimate Developer Platform observation client, not Control-Centre-exclusive | Architectural | Must | New, `ARCH-015` §39 |
| ROC-18 | SDK tier classification (Public/Supported/Experimental/Internal) MUST be determined per `ARCH-014` §8 at Architecture Authoring time | Governance | Must | New, §43 |
| ROC-19 | Effect payload content MUST NOT be transmitted across the Runtime Observation Boundary by default. Observation is limited to effect metadata (identity, status, provider, actor association, timestamps where established) necessary for legitimate developer inspection, unless a future, separately authorized design/architecture decision explicitly permits additional payload disclosure | Security | Must | New, §30/§56 |

Nineteen tracked identifiers (`ROC-01`–`19`).

## 51. Traceability Matrix

| Requirement | Source | Derived / Inherited | Notes |
|---|---|---|---|
| ROC-07, 09, 10, 12, 16 | `ARCH-016`/`ARCH-014`/`ARCH-015` | Inherited | Direct extension of already-Approved rules across a new boundary |
| ROC-01 | `IER-028-F01`; `ARCH-016` §31 | Derived | Applying an existing Control-Centre-scoped rule to a new, not-yet-conceived boundary required genuine design judgment |
| ROC-02, 03, 04, 05, 06, 08, 11, 13, 14, 15, 17, 18 | This exploration's own analysis | New/Derived | Genuinely new candidate requirements, disclosed as such, not silently promoted |
| ROC-04, 05 | `IER-028-F02` | Resolves the Minor finding directly — `ROC-04` (mutation-authority component) and `ROC-05` (observation-authorization component) jointly | — |
| ROC-19 | §30/§56 own risk analysis | New | Elevates a prior recommendation to a traceable requirement |
| ROC-01–19, collectively | `IER-028-F01` | The exploration's own reason for existing | — |

No requirement cites the lost `ARCH-013` as authority.

## 52. Inherited / Derived / New Classification

**Inherited** (already required by Approved architecture, restated here at the new boundary): `ROC-07`, `09`, `10`, `12`, `16`. **Derived** (necessary engineering/design consequence of inherited requirements): `ROC-01`, `03`, `06`, `08`, `13`, `14`, `15`. **New candidate requirements** (introduced by this exploration): `ROC-02`, `04`, `05`, `11`, `17`, `18`, `19`.

## 53. Architecture Questions (deferred, not answered here)

Architectural owner — confirm the Runtime-tier service's exact placement (§15); Runtime-side service's own internal shape and lifecycle; session semantics beyond the minimal framing in §22; the local trust/authorization mechanism satisfying `ROC-05` (§17); compatibility-negotiation mechanism (§26); the exact query/response contract shape; discovery-endpoint mechanism (§19); error-boundary types/codes; SDK Public/Internal tier classification (§43); deployment/enablement default (§42); evolution/versioning mechanism (§44); the exact effect-metadata field boundary satisfying `ROC-19` (§30).

## 54. Deferred Technology Questions

Concrete IPC/transport mechanism; serialization/wire encoding; transport library; async-runtime integration; endpoint naming/addressing scheme; OS-specific implementation differences; GUI-framework-side integration specifics.

## 55. Feasibility Matrix

| Capability | Classification |
|---|---|
| Runtime reachability | Requires new observation boundary |
| Compatibility | Requires new observation boundary (mechanism); vocabulary (`ARCH-014` §14) already exists |
| Durable actors | Supported by Runtime now (data); requires new observation boundary (transport) |
| Actor details | Requires SDK exposure (in-process, `EWO-028` §9.3) **and** new observation boundary (transport) |
| Capabilities | Supported by Runtime now (data); requires new observation boundary (transport) |
| Effects | Supported by Runtime now (data); requires new observation boundary (transport) |
| Supervision | Supported by Runtime now (data, bounded); requires new observation boundary (transport) |
| Diagnostics | Supported by Runtime now (data, live); requires new observation boundary (transport) |
| Runtime version/status | Requires architecture (data itself, per `EWO-028` §9.3) **and** new observation boundary (transport) |

## 56. Risks

Accidentally designing `D2` early under cover of "observation" (mitigated: `ROC-02`'s own binding separation requirement); exposing sensitive Runtime state (mitigated: `ROC-05` and `ROC-19`); overprivileged observation client (mitigated: `ROC-04` and `ROC-05` jointly — structural absence of mutation authority and mandatory observation authorization); Runtime instability from observation load (mitigated: `ROC-13`/`14`); transport becoming architecture by accident (mitigated: §54's explicit separation, no technology named anywhere in this document); platform-specific design (mitigated: §40's neutral constraint framing); protocol/client lockstep (mitigated: `ROC-15`/§44); SDK/Runtime ownership confusion (mitigated: §15/§36/§37's explicit split); observation state becoming a second state of record (mitigated: `ROC-09`/`10`, direct inheritance of `ARCH-016`'s own non-authoritative framing); designing only for the Control Centre (mitigated: `ROC-17`).

## 57. EWO-028 Unblocking Analysis

Once this exploration's preferred direction reaches Approved architecture and implementation: `EWO-028` §9.2 (Connection Model) would be rewritten to cite the concrete resulting architecture instead of treating protocol as an unexamined downstream detail; acceptance criteria 2 and 8 (§16 of `EWO-028`) become achievable; a new prerequisite dependency (this architecture, plus its own implementation) would need to precede or fold into `EWO-028`'s own Increment 0. **No other part of `EWO-028` requires substantive change** — §9.5–§9.13 (durable browser, actor detail, capability/effect/supervision/diagnostics treatment), testing strategy, and increments 1–6 were never the defect; only §9.2's own unexamined assumption was.

## 58. IER-028-F02 Disposition

> Absorbed as a normative requirement of this exploration — `ROC-04` (mutation-authority component: no cross-process capability representation is required, since the protocol itself defines no mutating operation) and `ROC-05` (observation-authorization component: an observation client MUST pass an appropriate local trust/authorization check) jointly, not returned as a narrow `EWO-028` correction. `IER-028-F02`'s design-level disposition is accepted (Founder Design Acceptance, `reviewed_by`, above); final engineering closure remains dependent on the eventual Approved architecture and implementation.

## 59. Design Approval Review Readiness

Complete — this document itself is the product of a full Design Approval Review → Narrow Design Correction → Narrow Design Re-Review cycle (`reviewed_by`, above), independently confirmed coherent at every stage. Narrow Design Re-Review verdict: PASS, zero remaining findings.

## 62. Non-Modification Statement

This document creates no Runtime capability, modifies no Runtime, SDK, or Control Centre architecture, selects no technology, and authorizes no implementation. It does not amend `DES-004`, `ARCH-014`, `ARCH-015`, or `ARCH-016`. `EWO-028` is not modified.

## Acceptance Framework

This exploration is adequately specified when: every proposed `ROC` requirement has an explicit class, priority, and traceability classification to either Approved architecture or direct code-level evidence (§50, §51); every inherited constraint is separated from new/derived proposal (§52); the `ARCH-014`/`ARCH-015`/`ARCH-016` relationships are extended with zero unresolved conflicts; every non-goal is verified unviolated (§11); the Observation/Control separation is structural, not conventional, and enforced by a closed, enumerable operation set (§13, `ROC-02`); every evidence-thin claim is explicitly hedged and left to Architecture Authoring or a later Technology Evaluation (§53, §54). This document's own completion is not self-declared — that was the Design Approval Review's, Narrow Design Correction's, and Narrow Design Re-Review's own role (`reviewed_by`, above), and ultimately the Founder's.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Design Exploration Draft, triggered by `IER-028-F01` (Blocking) and `IER-028-F02` (Minor), per `GOV-013` §6.4. Eighteen `ROC` requirements. Design Approval Review found `DAR-005-F01` (Minor), `F02` (Major), `F03` (Minor), `F04` (Major), `F05` (Minor), `F06` (Minor). |
| 0.2.0 | 2026-08-10 | Denver Jacobs (Founder) | Narrow Design Correction applying exactly `DAR-005-F01`–`F06`: `ROC-02` now requires a closed, enumerable operation set (`F01`); `ROC-05` raised from Should to Must, with mutation authority and observation authorization explicitly distinguished (`F02`); `ROC-08` reworded from an exclusivity mandate to a sufficiency requirement (`F03`); `ROC-19` added, requiring effect-payload minimization by default (`F04`); `ROC-11` reworded from a mandate to a non-foreclosure rule (`F05`); `ROC-01` reclassified Inherited → Derived (`F06`). Net requirement count: nineteen. Narrow Design Re-Review: PASS, zero remaining findings. |
| 0.2.0 (this filing) | 2026-08-10 | Denver Jacobs (Founder) | Repository Filing. Records genuine Founder acceptance of this document, at this version, as the final Runtime Observation / Connectivity Design Exploration / requirements baseline (`reviewed_by`, above), and Founder confirmation of Alternative A as the preferred design direction. No substantive content altered by this filing — identifier, frontmatter, this Founder Acceptance Notice, and this Revision History entry are the only additions, plus one mechanical cross-reference correction (former §16 cited a stale, non-existent "§65"; a v0.1.0 drafting artifact — this document has no section past §62 — corrected to §58, its evident intended target; no requirement, priority, classification, or design decision altered). Does not authorize Runtime Observation / Connectivity Architecture Authoring, technology selection, or implementation. |
