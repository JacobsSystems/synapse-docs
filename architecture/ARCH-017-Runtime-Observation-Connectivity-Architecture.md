---
document_id: ARCH-017
title: "SynapseOS Runtime Observation / Connectivity Architecture (Phase 1: Local, Query-Only)"
project: SynapseOS
specification: SynapseOS — the minimum coherent architecture necessary to resolve IER-028-F01, establishing a Runtime Observation Service and SDK-mediated Observation Client connected by a local-only, structurally read-only Observation Boundary
version: 0.2.2
status: Approved
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.2 interim Chief Architect authority (GOV-010 §5, Class B) — no Chief Architect is currently appointed, the identical basis ARCH-007, ARCH-008, ARCH-011, ARCH-012, ARCH-014, ARCH-015, and ARCH-016 each already record for themselves.
created: 2026-08-10
last_updated: 2026-08-10
classification: Public
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.8-§6.11, the Architecture Authoring / Independent Architecture Review / Architecture Correction / Narrow Architecture Re-Review lifecycle this document's own history follows exactly)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution; §4/§5 capability-security-first principle, directly governing §32's deployment-default decision)
  architecture:
    - ARCH-014 (v0.8.0, Approved — Synapse SDK Architecture; §5/§8/§13 — the Foundation/Ergonomics layering and Public API tier system §31 classifies this document's own SDK surface against)
    - ARCH-015 (v0.2.0, Approved — Developer Platform Boundary Architecture; the cross-cutting invariants — Runtime Authority, Generalization, Tool Independence — extended throughout, never redefined)
    - ARCH-016 (v0.2.1, Approved — Control Centre Foundation Architecture; §13/§14/§21/§26/§28 — the connection-lifecycle, staleness, partial-projection, compatibility, and restart-continuity rules this document extends across a genuine process boundary, never re-derives)
  consolidation:
    - DES-005 (v0.2.0, Draft, Founder-accepted, filed — Runtime Observation / Connectivity: Design Exploration; the requirements/design baseline this document transforms into architecture)
  work-orders:
    - EWO-028 (Draft, unfiled — Phase 1 Read-Only Control Centre GUI Engineering Work Order; its Independent Engineering Work Order Review found IER-028-F01 (Blocking) and IER-028-F02 (Minor), the direct evidentiary trigger for this architecture; remains blocked pending this architecture's own implementation)
  adrs:
    - ADR-0020 (v0.1.0, Approved — Disposition of the Unrecoverable ARCH-013 Draft)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-017 — SynapseOS Runtime Observation / Connectivity Architecture (Phase 1: Local, Query-Only)

> **Filing Notice.** This document was authored, independently reviewed, corrected, re-reviewed, further corrected, re-reviewed again, further corrected once more (a single-sentence editorial fix), and finally re-reviewed a fourth time — entirely as a chat-delivered artifact before this Repository Filing, the identical lifecycle shape `ARCH-007`, `ARCH-012`, `ARCH-014`, `ARCH-015`, and `ARCH-016` each established for themselves. Version 0.1.0 was the initial Architecture Authoring Draft, transforming `DES-005` v0.2.0's Founder-accepted requirements into a narrowly-scoped, local, query-only Phase 1 Runtime Observation / Connectivity foundation. Its Independent Architecture Review found `REVISION REQUIRED` (0 Critical, 3 Major [`IAR-017-F01`–`F03`], 4 Minor [`IAR-017-F04`–`F07`]). Version 0.2.0 applied exactly those seven findings; its own Narrow Architecture Re-Review confirmed all seven Resolved but recorded two new Minor findings (`NAR-017-F01` — an SDK Foundation/Ergonomics scoping imprecision; `NAR-017-F02` — a stale cross-reference introduced by the `F02` correction itself) and returned `FURTHER NARROW CORRECTION REQUIRED`. Version 0.2.1 applied exactly those two findings; its own Further Narrow Architecture Re-Review confirmed both Resolved and all seven original findings still closed, but found one further, narrowly-scoped Minor finding (`FNAR-017-F01` — a malformed, leftover editorial fragment the `F02` correction had itself left inside §14) and again returned `FURTHER CORRECTION REQUIRED`. Version 0.2.2 applied exactly that one finding; the Final Single-Finding Further Narrow Architecture Re-Review confirmed it Resolved, confirmed every prior finding across the entire chain remained closed, and returned `PASS` — zero remaining findings of any severity anywhere in the review chain. Founder Architecture Approval is recorded in full below (§50, Disposition); this Filing (v0.2.2) records that approval and constitutes this document's own Repository Filing. It does not itself authorize implementation, technology selection, an Engineering Work Order, or `EWO-028`'s own unblocking.

## 1. Document Control

| Field | Value |
|---|---|
| Document ID | ARCH-017 |
| Title | SynapseOS Runtime Observation / Connectivity Architecture (Phase 1: Local, Query-Only) |
| Version | 0.2.2 |
| Status | **Approved** — Founder Architecture Approval recorded below (§50) |
| Author | Denver Jacobs (AI-assisted) |
| Approval authority | Chief Architect (Class B, per `GOV-010` §5), vacant; Founder (interim), per `GOV-003` §3.2 |
| Created | 2026-08-10 |
| Classification | Public |
| Requirements baseline | `DES-005` v0.2.0 (Founder-accepted, filed, commit `7196108`) |
| Constitutional authority | `GOV-018`, `ARCH-014` (Approved v0.8.0), `ARCH-015` (Approved v0.2.0), `ARCH-016` (Approved v0.2.1) |
| Numbering | `ARCH-017` — freshly verified at Authoring; unaffected by any correction |
| Review requirement (v0.1.0, complete) | Architecture Authoring → Independent Architecture Review → **REVISION REQUIRED** (0 Critical, 3 Major, 4 Minor) — Founder-accepted, correction authorized |
| Review requirement (v0.2.0, complete) | Narrow Architecture Correction (`IAR-017-F01`–`F07`) → Narrow Architecture Re-Review → **FURTHER NARROW CORRECTION REQUIRED** (all seven Resolved; 2 new Minor) — Founder-accepted, further correction authorized |
| Review requirement (v0.2.1, complete) | Further Narrow Architecture Correction (`NAR-017-F01`–`F02`) → Further Narrow Architecture Re-Review → **FURTHER CORRECTION REQUIRED** (both Resolved, all prior closed; 1 new Minor) — Founder-authorized, single-sentence correction authorized |
| Review requirement (v0.2.2, complete) | Single-Sentence Further Narrow Architecture Correction (`FNAR-017-F01`) → Final Single-Finding Further Narrow Architecture Re-Review → **PASS** (zero remaining findings) → Founder Architecture Approval |

## 2. Executive Summary

This document establishes the minimum coherent architecture necessary to resolve `IER-028-F01` — the confirmed absence of any mechanism for a separate process to observe an independently running SynapseOS Runtime. It introduces a **Runtime Observation Service** (Runtime-tier, new) and a corresponding **SDK-mediated Observation Client** (SDK-tier), connected by a **local-only, structurally read-only Observation Boundary** exposing a **closed, enumerable set of seven query operations** (§17). It selects no transport, protocol, serialization, or IPC technology. It authorizes no Runtime Control API, no mutation, no remote observation, no multi-Runtime orchestration, no global actor enumeration, and no persistent audit history — each a disclosed, separately governed future extension point, not designed here.

Across its full review lifecycle, this architecture: replaced an unsupported mandatory Runtime-observation timestamp with provenance-honest freshness metadata; made explicit the transport-class consequence of its own same-OS-user trust model; established that the Runtime Observation Service's own lifecycle is fully independent of Runtime bootstrap and operation; classified the SDK observation surface precisely against `ARCH-014`'s own two independent axes (Foundation Layer, Experimental API tier); reversed its deployment default to explicit opt-in after direct evaluation against `GOV-018`'s capability-security posture; disclosed precisely what its own trust model does and does not protect against; and restated its Runtime-safety requirement as a safety property rather than an implementation-prescriptive isolation mandate. Every one of these was a genuine, independently re-verified correction, not a formality — the complete review lineage is recorded in the Filing Notice above and in this document's own Revision History (§48).

## 3. Context

`DES-005` v0.2.0 (Founder-accepted, filed) established the requirements baseline for this workstream, triggered directly by `EWO-028`'s own Independent Engineering Work Order Review. Alternative A — a query-oriented local observation service — was accepted as the preferred design direction. This document is the `GOV-013` §6.8 Architecture Authoring stage for that accepted design.

## 4. Authority and Constitutional Basis

`GOV-018` §4/§5 (constitutional properties, capability-security principle — directly governing §32's own deployment-default decision); `ACT-005` §6 (Control Centre boundary — a compatible future path, not an implementation authorization); `ARCH-015` (the cross-cutting Developer Platform Boundary this document inherits in full — `DPB-INV-01` Runtime Authority, `DPB-INV-05` Progressive Disclosure, `DPB-INV-06` Conceptual Consistency, `DPB-INV-09` Error Vocabulary Consistency, `DPB-INV-10` Compatibility Contract, `DPB-INV-12` State of Record, and §35's Generalization/§39's Tool-Independence principles all directly binding here); `ARCH-014` (SDK error/compatibility vocabulary and tier system, consumed via `ARCH-015`, never duplicated); `ARCH-016` (§13/§14/§21/§26/§28 — the connection-lifecycle, staleness, partial-projection, compatibility, and restart-continuity rules this document extends across a genuine process boundary, never re-derives); `DES-005` v0.2.0 (the requirements baseline this document transforms into architecture).

## 5. Scope

The architectural boundaries required for local, read-only, cross-process Runtime observation: the Runtime Observation Service's own placement, responsibilities, and lifecycle independence from Runtime bootstrap/operation; the SDK Observation Client's own placement and layer/tier classification; the trust/authorization model and its transport-capability consequence; the connection and session lifecycle; the closed operation set and its query/response contract shape; the compatibility-negotiation model; the projection contracts for Runtime status, durable actors, actor detail, capabilities, effects, supervision, and diagnostics; Runtime-side resource protection; the deployment/enablement default. Nothing beyond this.

## 6. Non-Goals

Runtime Control API (`D2`) design; any mutation operation; remote Runtime observation or management; multi-Runtime orchestration; global actor enumeration (`D1`); persistent audit-history architecture (`D4`); authoritative Application architecture (`D3`); Control Centre GUI implementation; GUI technology selection; concrete transport/protocol/serialization/IPC technology selection; push/subscription protocol design; capability grant/revoke; effect cancel/retry/re-execute; durable-state deletion.

## 7. Architectural Principles

1. **Closed over extensible.** The observation surface is a fixed, enumerable operation set, never a generic dispatch mechanism — `DES-005` `ROC-02`, elevated here to the architecture's own organizing principle.
2. **Projection, not authority.** Every fact this boundary transports is a projection of Runtime truth already established by `ARCH-007`/`008`/`011`/`012`/`016`, never independently manufactured (`ARCH-015` `DPB-INV-01`).
3. **Fail closed on trust and on presence.** No projection is served, and no Runtime detail beyond a bare authorization-failure signal is disclosed, until the trust/authorization check succeeds (§14). This same fail-closed posture governs the Observation Service's own existence — it is present only where explicitly configured, not by default (§32).
4. **Minimal architecture.** Only decisions that genuinely require durable, cross-cutting resolution are settled here (`ARCH-015` §9's own principle, directly reused).
5. **Narrow, disclosed deferral.** Every excluded capability names its own future extension point (§35–§38) rather than silently foreclosing it.

## 8. System Context

```
Runtime Process                              Client Process
┌────────────────────────────┐               ┌──────────────────────────────┐
│ Runtime (ARCH-007/008/     │               │ Synapse SDK (ARCH-014)       │
│ 011/012) — sole authority   │               │   └─ Observation Client      │
│                              │               │       (this architecture)    │
│ Runtime Observation Service  │◄══ Observation ══►                          │
│ (this architecture, new)     │    Boundary    │  Control Centre / future    │
│  - closed 7-operation set    │  (local-only,  │  CLI / IDE plugin —         │
│  - trust/authorization check │   technology   │  consumer only, never       │
│  - compatibility negotiation │   unselected)  │  bypasses the SDK           │
└────────────────────────────┘               └──────────────────────────────┘
```

The Observation Client never reaches the Runtime Observation Service except through the SDK (`ARCH-015` §12/§13, `ROC-12`, inherited directly).

## 9. Authority Model

Direct application of `ARCH-015` `DPB-INV-01` at this new boundary: the Runtime Observation Service transports projections of Runtime authority; it never becomes a second authority for execution, capability, effect, durability, or supervision. Every field a projection carries must be traceable to a specific, already-authoritative Runtime query (`ARCH-016` §16/§18/§19/§20's own semantics, extended unmodified across the process boundary).

## 10. Runtime / SDK / Observation Client Boundary

The Observation Client **must** consume the Observation Service exclusively through the SDK — no Developer Platform tool constructs its own connection to the Runtime Observation Service directly (`ROC-12`). This preserves `ARCH-015` §12's SDK-boundary rule at a second layer and prevents the SDK from becoming bypassable infrastructure rather than genuine mediation.

## 11. Internal Boundaries

Four conceptual boundaries, not implementation classes or crates:

- **Runtime Observation Service** — Runtime-tier; owns connection acceptance, authorization, compatibility negotiation, and the closed operation set's own execution (§13).
- **Trust / Authorization Boundary** — governs which connecting processes may establish a session at all (§14).
- **Compatibility Boundary** — governs whether an established connection may proceed to querying (§19).
- **SDK Observation Client** — SDK-tier; owns the client-facing API, connection lifecycle, projection typing, and error translation (§31).

## 12. State / Projection Ownership

| State | Authoritative Owner | Observation Boundary Treatment |
|---|---|---|
| Runtime status/reachability | Runtime | Projected via `GetRuntimeStatus`; version/build fields remain investigation-gated (`EWO-028` Founder Direction), not resolved here |
| Durable actor enumeration | Persistence Service | Projected via `ListDurableActors`; `ARCH-016` §16's full semantics (partial, not global, termination ≠ deletion) carried unmodified |
| Actor identity/lifecycle | Actor Directory / Lifecycle Guardian | Projected via `GetActorDetail`, given an already-known or enumerated id; blocked on the existing `Requires exposure` SDK gap (`ARCH-016` §17), not created here |
| Capabilities | Capability Authority | Projected via `ListCapabilities`; no capability representation crosses the boundary (`ROC-04`) |
| Effects | Effect Coordinator | Projected via `ListEffects`; metadata only, payload excluded by default (`ROC-19`, §25) |
| Supervision | Supervisor | Projected via `GetSupervisionRelationship`; bounded exactly as `ARCH-016` §20 establishes |
| Diagnostics | SDK error vocabulary | Projected via `GetDiagnostics`; live/current only |
| Session/connection state | Observation Boundary itself | No authority content; ends with the connection (§16) |

## 13. Connection Model

Phase 1 addresses local Runtime connection only. Connection sequence, a fixed order: **(1)** transport-level connect (mechanism unselected, §15); **(2)** trust/authorization check (§14) — failure transitions to `Unauthorized`, disclosing nothing beyond that outcome; **(3)** compatibility negotiation (§19) — failure transitions to `Incompatible`; **(4)** session establishment (§16); **(5)** operations from the closed set (§17) become permitted. States: `Disconnected`, `Connecting`, `Connected`, `Incompatible`, `Unauthorized`, `Unavailable`, `Reconnecting` — `ARCH-016` §14/§28's staleness and restart rules apply directly to every transition.

`Unavailable` covers both "the Runtime process itself is gone" and "the Runtime is running but its Observation Service has failed or was never configured" — a connecting client cannot, and is not required to, distinguish these from the wire alone (§29). Whether the Observation Service was ever configured at all is a deployment-time fact (§32), not a distinct wire-visible connection state.

## 14. Trust and Authorization Model

Model, not mechanism — the concrete OS API/library remains a Technology Question. **Decision:** the default trust boundary is same local OS user as the Runtime process — a connecting process is authorized only if it can be established, via a platform-appropriate peer-identity check, to run as the same OS user that owns the Runtime process, or as a user explicitly configured as additionally trusted.

This trust model carries an explicit architectural transport requirement: **the Phase 1 transport MUST support trustworthy local peer-principal identification, or an equivalent supplementary authorization mechanism MUST be separately justified and shown architecturally compatible with this trust model (§44).** This is not satisfied by a transport that cannot deliver any peer-identity signal to the listening end without such a supplementary mechanism. No transport is named or selected here; this is a mandatory Technology Evaluation input, not a technology choice.

This is an explicit, accepted Phase 1 trust boundary, stated precisely, not an accidental security guarantee: same-OS-user authorization protects against processes belonging to other local OS users, and against clients that cannot establish the required trusted principal at all. **It does NOT isolate the Runtime Observation Service from malicious or compromised software already executing as the Runtime's own trusted OS user.** No stronger protection is claimed. A future, stronger trust model — per-application authorization, a capability-scoped credential, or similar — is a separately governed future architectural evolution (§44), not invented here.

Fails closed: absent a successful check, the connection is refused with `Unauthorized`, never partially served.

## 15. Discovery / Endpoint Boundary

**Model:** a single well-known local endpoint, one per Runtime process, supplied to launch tooling or the connecting client out-of-band (environment variable, file-based handle, or fixed local address — the exact mechanism is a Technology Question, `ROC-06`). No discovery protocol, no registry, no broadcast/announcement mechanism is architected — consistent with `DES-005` §19's own finding that automatic discovery is not required given `DES-004`'s user-initiated Core User Journey.

## 16. Observation Sessions

A session is established only after §13's steps 1–3 succeed. It carries exactly one fact: "this connection passed the trust and compatibility checks as of session establishment." Phase 1 sessions carry no differentiated permission level — a session either exists (full closed-operation-set access) or does not (`Unauthorized`); no per-operation authorization granularity is architected, since no operation in the closed set (§17) is more sensitive than the overall trust boundary already accounts for. Ending the connection ends the session; no session state persists across reconnection (`ARCH-016` §28, `ROC-07`, directly inherited).

## 17. Closed Operation Set

Exactly seven operations, satisfying `ROC-02`'s own closed-and-enumerable requirement. No operation mutates Runtime state; none is generic dispatch. Each takes a typed request and returns a typed projection carrying explicit freshness and partiality metadata (`ROC-09`/`10`). Extending this set requires a future architecture amendment to this document, never a protocol-level addition made unilaterally by an implementation.

| Operation | Input | Returns | Projection Domain |
|---|---|---|---|
| `GetRuntimeStatus` | none | Reachability, compatibility tier/version-range | §21 |
| `ListDurableActors` | none | Durable actor enumeration (`known_actors()`-equivalent) | §22 |
| `GetActorDetail` | known/enumerated `ActorId` | Identity, lifecycle state | §23 |
| `ListCapabilities` | known `ActorId` | Bound capability projection | §24 |
| `ListEffects` | known `ActorId` (or effect id) | Effect metadata (no payload) | §25 |
| `GetSupervisionRelationship` | known `ActorId` | `is_registered`/`parent_of` relationship | §26 |
| `GetDiagnostics` | none, or known `ActorId` | Live/current diagnostic and error information | §27 |

No operation named here defines a wire message, endpoint path, or serialization shape — that remains a Technology Question. This is an operation catalogue, not an API specification.

## 18. Query / Response Contract Shape

**Model:** every operation is a synchronous request/response exchange (`ROC-08`). **Every response MUST carry sufficient freshness/observation metadata to support truthful `Complete`/`Partial`/`Unavailable`/`Unsupported` interpretation** (`ARCH-016` §21's cross-cutting rule, extended here). A timestamp is optional, not mandatory, and MAY be included only where its provenance is explicit:

- **Runtime-owned timestamp** — permitted only if grounded in an authoritative Runtime time source that actually exists for the data in question; none currently does anywhere in `synapse-runtime` production code. This architecture does not create such a source.
- **SDK-local receipt timestamp** — permitted, but only if explicitly identified as SDK/client-local receipt time, never represented as Runtime observation time, and never used to imply Runtime-side atomicity or continuity.
- **No timestamp** — an explicit non-temporal freshness/observation marker (e.g., a per-session monotonic observation sequence number) satisfies this requirement equally.

No cross-operation atomic snapshot is promised — each of the seven operations remains independently answered (`ROC-09`, `ARCH-016` §14 inherited).

## 19. Compatibility Negotiation Model

**Model:** at session establishment (§13 step 3), the Runtime and the connecting SDK client each present their own compatibility tier and version-range information, consuming `ARCH-014` §14's tier vocabulary and `ARCH-016` §26's tier-vs-version-range distinction directly — no new versioning scheme invented. If the presented information indicates incompatibility, the session enters `Incompatible` and no operation from §17 is permitted. The concrete wire encoding of this exchange is a Technology Question (`ROC-15`).

## 20. Reconnection and Continuity

Direct, unmodified extension of `ARCH-016` §28: reconnection — including after a Runtime restart — establishes only that the Observation Service is currently reachable and has passed §13's checks again; it never establishes that the reconnected Runtime is the same instance previously observed. All previously observed projections remain subject to §18's staleness marking until re-observed. No identity, generation, or session-continuity token is introduced (`ROC-07`).

## 21. Runtime Status Projection

`GetRuntimeStatus` projects reachability and compatibility information only. Version/build/count fields remain gated behind the existing, separately disclosed investigation (`EWO-028` Founder Direction) — this architecture does not resolve what data exists, only how already-existing data would cross the boundary once it does.

## 22. Durable Actor Projection

`ListDurableActors` projects exactly `ARCH-016` §16's own semantics unmodified: inclusion reflects current durable-state possession at observation time; no ordering guarantee; potentially stale; explicitly not global actor enumeration; actor termination alone does not remove an entry, only a separate, capability-authorized deletion does.

## 23. Actor Detail Projection

`GetActorDetail`, given an already-known or `ListDurableActors`-enumerated `ActorId`, projects identity and lifecycle state. The underlying SDK exposure this depends on is confirmed not yet to exist (`ARCH-016` §17, `Requires exposure`) — this architecture defines how that data, once exposed, crosses the boundary; it does not itself create the exposure.

## 24. Capability Projection

`ListCapabilities` projects bound-capability information read-only; no `Capability` value or any cross-process capability representation crosses the boundary (`ROC-04`, `ARCH-016` §18's absence-≠-none rule preserved).

## 25. Effect Projection

`ListEffects` projects **effect metadata only — identity, status, provider, actor association, and timestamps where established. Effect payload content (arguments, results, error detail bodies) is explicitly excluded from this projection by architecture, not merely by default configuration** (`ROC-19`). No field schema beyond this metadata boundary is frozen here. No cancel/retry/re-execute path exists in this operation set at all (§17).

## 26. Supervision Projection

`GetSupervisionRelationship` projects exactly `ARCH-016` §20's own bounded semantics (`is_registered`/`parent_of` only); restart/failure history remains unavailable, a disclosed deliberate boundary this architecture does not attempt to work around.

## 27. Diagnostics Projection and Error Model

`GetDiagnostics` projects live/current diagnostic information only, consuming `ARCH-014` §12's developer-facing error vocabulary directly (`ROC-16`). Errors crossing the boundary at the connection/session level are classified into eight fixed categories: connection failure, authorization failure, unsupported query, unavailable information, Runtime-side failure, compatibility failure, malformed request, timeout. No competing taxonomy may be introduced by an implementation.

## 28. Runtime Safety and Resource Protection

**Observation workload MUST NOT starve, materially interfere with, or become an availability dependency of Runtime actor execution, regardless of implementation structure.** This is satisfiable by dedicated threads/executors, by bounded scheduling on shared infrastructure, or by any other mechanism an implementation chooses — none is mandated here. The architecture requires: bounded observation work; bounded concurrent in-flight work per connection; bounded response/result size; rejection or failure behavior once those bounds are exceeded; and detection/forced disconnection of an abandoned or unresponsive client (`ROC-13`). No specific numeric threshold is set.

## 29. Failure Isolation and Service Lifecycle Independence

**Client failure isolation:** a crashed, hung, disconnected, or malformed-request-sending Observation Client must not affect Runtime availability, correctness, or performance beyond §28's own bounds (`ROC-14`). The Runtime Observation Service must detect and forcibly disconnect an abandoned or unresponsive client rather than holding resources indefinitely on its behalf.

**Observation Service lifecycle independence: Runtime bootstrap and continued Runtime operation MUST NOT depend on successful Observation Service initialization or continued availability.**

1. Observation Service initialization failure (e.g., inability to bind its own local endpoint) must not by itself fail Runtime bootstrap.
2. The Runtime may operate correctly, indefinitely, with observation unavailable — whether because it was never configured (§32) or because it failed after starting.
3. Observation Service failure or restart must not require a Runtime restart.
4. Any existing observation sessions terminate if the service becomes unavailable; this is a normal `Unavailable` transition (§13), not a Runtime-level failure.
5. Clients observe the service as unavailable/disconnected exactly as they would observe any other connection loss — no distinct wire-level signal is required to distinguish "service failed" from "Runtime gone" (§13).
6. Runtime-owned actor/lifecycle/effect/capability/persistence/supervision semantics remain entirely unaffected by Observation Service state.
7. If the service later becomes available, clients establish a fresh connection/session and revalidate per §20's existing reconnection rules — no continuity is assumed.

This architecture does not define the Observation Service's own restart mechanism (implementation).

## 30. Multiple Clients

The Runtime Observation Service's own internal design must not assume a single connection slot — its architecture must support concurrent sessions. Phase 1 implementation is not required to exercise this beyond what a straightforward concurrent-session-capable design already provides; no additional multi-client coordination, fairness, or prioritization mechanism is architected (`ROC-11`, satisfied as non-foreclosure).

## 31. SDK Role and Compatibility Tier

The SDK observation surface has two distinct layer responsibilities (`ARCH-014` §5):

- **Foundation-layer surface (this document's own classification target).** The direct, unmediated reachability of the seven closed operations (§17) and their raw projection/error contract (§18, §21–§27) — invoking each operation, receiving its typed projection, interpreting its freshness/partiality metadata and error category directly. This is the surface `ROC-18` requires this document to classify, and the surface classified below.
- **Any Ergonomics-layer convenience (not designed here, not mandated).** Connection-lifecycle orchestration, compatibility-check orchestration, reconnect convenience, or composition of multiple Foundation calls with supplied defaults — to the extent any such convenience is genuinely describable as an ordered sequence of Foundation-layer calls with supplied defaults (`ARCH-014` §5 item 2's own defining test), it would be Ergonomics-layer convenience *over* the Foundation surface, not a redefinition of it. This document does not design, mandate, or preclude such convenience; if and when it is built, its own tier classification is a separate, later decision under `ARCH-014` §8. No consumer is required to use an Ergonomics wrapper — the Foundation surface alone is sufficient and complete for satisfying `ROC-12`/§10's SDK-mediation requirement.

**Decision, resolving `ROC-18`:** the Foundation-layer observation surface is classified **Foundation Layer, Experimental API tier**, per `ARCH-014` §5/§8/§13.

- **Layer — Foundation.** Per `ARCH-014` §13's own automatic-entry rule ("new Runtime capability... enters the SDK at the Foundation Layer first, automatically, the moment it becomes part of the Runtime's own public surface"): the Observation Service becomes part of the Runtime's own public surface once implemented, and its direct SDK reachability accordingly enters at Foundation.
- **Tier — Experimental.** Per `ARCH-014` §8 item 3: "carries no compatibility expectation at all... exists so that newer... capability can be reachable before it has earned Supported status." No wire protocol or concrete implementation yet exists to stabilize a Public/Supported contract against.

**Layer and Tier remain two independent classification axes** (`ARCH-014` §5 vs. §8) — Layer describes *where* in the SDK's own structure a surface enters; Tier describes *what compatibility commitment* currently attaches to it. Promotion beyond Experimental tier is a future, separately evaluated decision (`ARCH-014` §8's Movement rule); Foundation-layer placement is not expected to change, since it follows from where the underlying Runtime capability itself lives, not from maturity.

## 32. Deployment / Enablement Model

**Decision, evaluated explicitly against `GOV-018`'s capability-security-first constitutional posture, favoring minimal attack surface by default:** `synapse-runtime` currently has zero IPC/listening surface of any kind (`IER-028-F01`'s own central finding) — the Observation Service would be the first such surface ever introduced. A listening endpoint's own connection-acceptance and compatibility-negotiation code (§13 steps 1–3) is exposed to any connecting process *before* the authorization outcome is known, regardless of how strong that authorization check later proves to be — mandatory authorization narrows *who gains access after connecting*, it does not shrink *the presence of the listening surface itself*.

**Selected default: the Runtime Observation Service is present only when explicitly configured at Runtime bootstrap; it is absent by default.** No separate "developer mode" flag is introduced beyond this single opt-in configuration point. The friction of explicit opt-in is one composition-time configuration call at Runtime bootstrap, matching the same pattern already established for other optional Runtime-tier services — invisible to `DES-004`'s own Core User Journey persona, who does not hand-write `Runtime::bootstrap()` calls directly but launches through existing tooling that can pass this configuration once, on the Control Centre's own behalf.

What remains configurable: an embedding application (or Control-Centre-oriented launch tooling) explicitly opts in at bootstrap; declining, or simply not opting in, leaves zero observation surface present, structurally, not merely access-denied. What is preserved regardless: mandatory trust/authorization when the service *is* configured (§14); no mutation surface ever (§17); Runtime operation is fully independent of whether observation is configured, present, or failed (§29); SDK/client state is truthfully reported as `Unavailable` whenever observation is not reachable, whether by absence, decline, or failure (§13, §29).

## 33. Cross-Platform Constraints

Same-machine process communication; peer-identity-based trust check (§14); local endpoint addressing (§15); synchronous request/response (§18); compatibility negotiation (§19); classifiable errors (§27); bounded resource use (§28); Windows/Linux/macOS support (`ARCH-008` §29, directly inherited); testability. No mechanism selected.

## 34. Security

Least authority achieved structurally by the closed, mutation-free operation set (§17), never by a cross-process capability system (§24); mandatory, fail-closed local trust/authorization when configured (§14) — explicitly bounded to the same-OS-user threat model disclosed there, not a defense against already-compromised same-user software; effect-payload minimization by architecture, not configuration (§25); denial-of-service resistance via the non-starvation property (§28); malformed-request isolation (§29); no implicit blanket trust merely from being local (§14); minimal attack surface by default — the Observation Service does not exist unless explicitly configured (§32).

## 35. Relationship to D2 (Runtime Control API)

The Runtime Observation Service and any future Runtime Control API remain architecturally separate boundaries, never variants of one another. This is structurally enforced two ways: `D2` cannot be created by adding a case to §17's closed operation set, because that set is fixed by this document, not extensible by an implementation; and `D2`, when separately architected, would require its own trust model, its own operation catalogue, and its own architecture review — sharing, at most, underlying connection/session *concepts* (§13, §16) at that future architecture's own discretion.

## 36. Relationship to Future Remote Runtime

Local-only for Phase 1 (§13, §15). Nothing in §16's session model or §18's query/response shape is inherently local-only in its semantics — only §14's peer-OS-user trust model is local-specific. A future remote extension would require its own trust/authentication architecture entirely; it would not automatically inherit §14's model.

## 37. Relationship to Future Push/Subscription

Not designed here (§18, `ROC-08`). The closed operation set (§17) and query/response contract shape are structured so that a future push/subscription capability could be added as new, separately architected operations without altering the seven existing ones — an extension point, not a design.

## 38. Relationship to D1 (Global Discovery) and D4 (Persistent Audit)

Not designed here. `ListDurableActors` (§22) remains durable-only, exactly as `ARCH-016` §16/§33 already establish; a future `D1` architecture would extend, not replace, this projection. `GetDiagnostics` (§27) remains live/current only, exactly as `ARCH-016` §21/§34 already establish; a future `D4` architecture would extend the Diagnostics Projection, not this document.

## 39. Architecture Invariants

| ID | Statement | Source |
|---|---|---|
| ROI-01 | The Observation Service exposes exactly the closed, enumerable operation set of §17; no generic/extensible dispatch mechanism exists | `ROC-02`, §17 |
| ROI-02 | No operation defined by this architecture can mutate Runtime state | `ROC-01`/`04`, §13 |
| ROI-03 | A future Runtime Control API (`D2`) is a wholly separate boundary, never an extension of this one | `ROC-02`, §35 |
| ROI-04 | An observation client must pass the peer-OS-user trust check, over a transport capable of trustworthy peer-principal identification or an equivalent justified supplement, before any session is established | `ROC-05`, §14 |
| ROI-05 | No cross-process capability representation is introduced by this architecture | `ROC-04`, §24 |
| ROI-06 | Phase 1 operations are synchronous query/request-response only; none may depend on push/subscription | `ROC-08`, §18 |
| ROI-07 | Every response carries explicit freshness/observation metadata sufficient for truthful interpretation; no mandatory Runtime-owned timestamp is required absent an authoritative source | `ROC-09`, §18 |
| ROI-08 | Every partial/unavailable/unsupported projection state is independently, truthfully representable | `ROC-10`, `ARCH-016` §21 |
| ROI-09 | The Observation Service's own internal design does not structurally foreclose multiple simultaneous clients | `ROC-11`, §30 |
| ROI-10 | The Observation Client surface is exclusively SDK-mediated | `ROC-12`, §10 |
| ROI-11 | Observation workload must not starve, materially interfere with, or become an availability dependency of actor execution, regardless of implementation structure | `ROC-13`, §28 |
| ROI-12 | A failing/abandoned observation client cannot degrade Runtime availability or correctness | `ROC-14`, §29 |
| ROI-13 | Session establishment requires successful compatibility negotiation before any query is permitted | `ROC-15`, §19 |
| ROI-14 | Errors crossing the boundary are classified per the established taxonomy, never a competing one | `ROC-16`, §27 |
| ROI-15 | This boundary's design is general-purpose across any legitimate Developer Platform observation client, never Control-Centre-exclusive | `ROC-17`, §11 |
| ROI-16 | Effect payload content is never transmitted by default; only the defined effect-metadata fields are | `ROC-19`, §25 |
| ROI-17 | Runtime identity/continuity is never asserted beyond current reachability; reconnection requires re-observation | `ARCH-016` §28, `ROC-07` |
| ROI-18 | Runtime bootstrap and continued Runtime operation never depend on Observation Service initialization, availability, or restart | §29 |

## 40. Architecture Decision Table

| Decision | Selected Direction | Alternatives Considered | Evidence | Consequence |
|---|---|---|---|---|
| Operation set shape | Closed, seven named operations | Generic request/response dispatch by method name — rejected, violates `ROC-02` directly | `DES-005` §13/§50, `ROC-02` | New operations require architecture amendment, not implementation extension |
| Trust model | Same local OS user (peer identity), configurable additional trust, with explicit transport-capability requirement and disclosed same-user threat boundary | No trust check — rejected, contradicts `DES-005` §17's own risk rationale; full capability/token system — rejected, exceeds Phase 1 need | `DES-005` §17/§41, `ROC-05` | Technology Evaluation inherits an explicit constraint and an explicit, honestly bounded security claim |
| Session permission granularity | Uniform — session exists or does not | Per-operation authorization tiers — rejected, no evidence any one of the seven operations is more sensitive than the trust boundary already covers | Principle 4 (Minimal Architecture) | Simpler session model; revisit only if evidence emerges |
| Connection sequence ordering | Authorize before compatibility-negotiate | Compatibility-negotiate before authorize — rejected, would disclose Runtime detail before trust is established | Principle 3 (Fail Closed) | Unauthorized clients learn nothing about Runtime compatibility |
| SDK classification | Foundation Layer (direct operation reachability), Experimental tier; any Ergonomics-layer convenience left undesigned and separately tiered later | A single undifferentiated "SDK Observation Client" tier/layer statement — rejected, conflated two independent `ARCH-014` axes | `ARCH-014` §5/§8/§13, `ROC-18` | `ROC-18` resolved precisely on both axes, without over-designing an Ergonomics surface this document has no evidence basis to specify |
| Deployment default | Explicit opt-in at Runtime bootstrap; absent unless configured | Present by default, declinable — rejected, tested only against authorization redundancy, not `GOV-018`'s minimal-attack-surface posture; always-on, non-declinable — rejected, removes legitimate embedder control | `DES-005` §42; `GOV-018` §4/§5; `IER-028-F01` | No observation listening surface exists in any deployment that does not explicitly request it |
| Observation Service lifecycle | Independent of Runtime bootstrap/operation | Coupled — Runtime bootstrap fails if Observation Service fails — rejected, violates Runtime independence (`ARCH-015` `DPB-INV-01`) | — | Runtime remains fully operable with observation absent or failed |
| Multi-client support | Architecturally required not to foreclose; not required to exercise beyond natural design | Mandatory concurrent-client correctness testing — rejected, no current requirement demands it | `ROC-11` | Future CLI/IDE reuse preserved without present engineering burden |

## 41. DES-005 Requirement Mapping

| ROC Requirement | Disposition |
|---|---|
| `ROC-01` | Architecturally satisfied — §17 closed operation set is the read-only observation surface |
| `ROC-02` | Architecturally satisfied — §17, §35 |
| `ROC-03` | Architecturally satisfied — §13, §15, §36 (local-only) |
| `ROC-04` | Architecturally satisfied — §24, no capability representation introduced |
| `ROC-05` | Architecturally satisfied — §14 (model resolved; concrete mechanism remains Technology Question) |
| `ROC-06` | Architecturally satisfied — §15 (model resolved; concrete addressing remains Technology Question) |
| `ROC-07` | Architecturally satisfied — §20 |
| `ROC-08` | Architecturally satisfied — §18, §37 |
| `ROC-09` | Architecturally satisfied — §18 |
| `ROC-10` | Architecturally satisfied — §18, per-operation §21–§27 |
| `ROC-11` | Architecturally satisfied — §30 |
| `ROC-12` | Architecturally satisfied — §10 |
| `ROC-13` | Architecturally satisfied — §28 |
| `ROC-14` | Architecturally satisfied — §29 |
| `ROC-15` | Architecturally satisfied — §19 |
| `ROC-16` | Architecturally satisfied — §27 |
| `ROC-17` | Architecturally satisfied — §11 |
| `ROC-18` | Architecturally satisfied — §31 (Foundation Layer, Experimental tier, both required axes resolved) |
| `ROC-19` | Architecturally satisfied — §25 |

No requirement silently dropped; all nineteen accounted for. No requirement cites `ARCH-013`.

## 42. Risks

Accidentally designing `D2` under cover of an operation-set extension (mitigated: §17's own closed, amendment-only-extensible design, `ROI-01`/`03`); false freshness/timestamp precision — an implementation fabricating an authoritative-seeming Runtime timestamp where none exists (mitigated: §18's provenance-honest requirement); transport-class exclusion discovered late in Technology Evaluation (mitigated: §14's explicit disclosure); trust model bypassed by a misconfigured additional-trust allow-list (mitigated: §14's fail-closed default); same-OS-user compromise treated as mitigated when it is not (mitigated: §14/§34's explicit, honest threat-model boundary); effect-payload leakage via a future metadata-field misclassification (mitigated: §25's explicit architectural exclusion); resource exhaustion from observation load (mitigated: §28); Runtime instability from a hung client (mitigated: §29); accidental coupling between Observation Service health and Runtime bootstrap/availability (mitigated: §29's explicit lifecycle-independence rule); SDK tier promoted prematurely before implementation experience exists (mitigated: §31's explicit Experimental starting tier); deployment default judged wrong once real usage exists (mitigated: §32's explicit, disclosed decline mechanism, revisitable without re-architecting); operation catalogue judged incomplete once `EWO-028`'s SDK exposure work proceeds (mitigated: §17's own amendment-required-not-forbidden framing); resource-protection wording over-constraining implementation concurrency choice (mitigated: §28's property-based restatement).

## 43. Architecture Acceptance Tests

1. Can a client be refused observation access without learning anything about Runtime compatibility or state? — **Yes** (§13 step 2, §14).
2. Can the closed operation set be enumerated exhaustively from this document alone? — **Yes** (§17).
3. Can a future Runtime Control API be added without modifying this document's own operation set? — **Yes** (§35, `ROI-03`).
4. Can effect payload ever cross the boundary without a separate, future authorization? — **No** (§25).
5. Can a hung or malicious client degrade Runtime actor execution? — **No** (§28, §29).
6. Can the SDK client surface be implemented without selecting a transport? — **Yes** (§18/§19 define the model, not the wire shape).
7. Can multi-client support be added later without redesigning the Observation Service? — **Yes** (§30).
8. Does reconnection after a Runtime restart ever assert continuity? — **No** (§20).
9. Can freshness be communicated without fabricating an unbacked Runtime timestamp? — **Yes** (§18).
10. Is the transport's required trusted-peer-principal property (or its justified equivalent) an explicit Technology Evaluation input rather than a silent assumption? — **Yes** (§14, §44).
11. Does Runtime bootstrap succeed even when the Observation Service cannot initialize? — **Yes** (§29).
12. Does Observation Service restart or unavailability avoid requiring a Runtime restart? — **Yes** (§29).
13. Is the SDK Observation Client classified on both required `ARCH-014` axes (Layer and Tier)? — **Yes** (§31).
14. Is the same-user trust boundary disclosed accurately rather than implied to be stronger than it is? — **Yes** (§14, §34).
15. Can observation load be bounded without mandating a specific concurrency implementation? — **Yes** (§28).
16. Does effect-payload exclusion hold even through diagnostic/error paths? — Disclosed as an implementation-verification obligation (§25) — not resolved by architecture alone, correctly left to implementation verification.

## 44. Deferred Architecture

The concrete peer-principal identification mechanism satisfying §14's transport-capability requirement, or its justified supplementary alternative; the Observation Service's own concrete restart mechanism (§29); any future Ergonomics-layer convenience over the Foundation observation surface (§31), including its own tier classification if and when built; an eighth-or-later operation, should `EWO-028`'s own SDK exposure work reveal a genuine gap in the seven defined here; SDK tier promotion beyond Experimental; the concrete endpoint-addressing mechanism (§15); the concrete wire protocol/serialization (all sections); Runtime Control API architecture (`D2`); remote Runtime observation/trust architecture; multi-Runtime orchestration; global actor enumeration architecture (`D1`); persistent audit-history architecture (`D4`); push/subscription protocol.

## 45. External Dependencies

`DES-005`'s own dependencies (`ARCH-014`, `ARCH-015`, `ARCH-016`) remain unchanged. New: the existing `Requires exposure` SDK gap (`ARCH-016` §17) that `GetActorDetail` (§23) depends on, routed through `ARCH-014`'s own amendment process, not invented here; the existing Runtime-status investigation gate (`EWO-028` Founder Direction) that `GetRuntimeStatus` (§21) depends on for version/build fields.

## 46. Governance Impact

Occupies the same governance tier `ARCH-016` established for itself: architecture governing one cross-cutting Runtime-tier capability (Runtime Observation), subordinate to `GOV-018`'s constitutional layer and `ARCH-015`'s own cross-cutting boundary, superior to any future Engineering Work Order implementing it (including, once unblocked, `EWO-028`). No existing `ARCH`, `ADR`, `GOV`, or `STD` document is amended, superseded, or contradicted by this Filing.

## 47. References

`GOV-013` §6.8, `GOV-018`, `ACT-005`, `ARCH-014` v0.8.0, `ARCH-015` v0.2.0, `ARCH-016` v0.2.1, `DES-005` v0.2.0 (commit `7196108`), `IER-028-F01`/`F02`, `EWO-028` and its own Independent Engineering Work Order Review, this document's own complete Independent Architecture Review / Narrow Architecture Re-Review / Further Narrow Architecture Re-Review / Final Single-Finding Further Narrow Architecture Re-Review chain (all this engagement's own conversational record).

## 48. Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Architecture Authoring Draft, transforming `DES-005` v0.2.0's nineteen `ROC` requirements into seventeen invariants, a seven-operation closed operation set, and seven architectural decisions, per `GOV-013` §6.8. Independent Architecture Review: `REVISION REQUIRED` (0 Critical, 3 Major [`IAR-017-F01`–`F03`], 4 Minor [`F04`–`F07`]). |
| 0.2.0 | 2026-08-10 | Denver Jacobs (Founder) | Narrow Architecture Correction applying exactly `IAR-017-F01`–`F07`: mandatory-timestamp requirement replaced with provenance-honest freshness metadata; trust model's transport-capability requirement and same-user threat boundary made explicit; Observation-Service-lifecycle-independence rule added (`ROI-18`); SDK classification completed with the Foundation Layer axis; deployment default reversed to explicit opt-in against `GOV-018`'s minimal-attack-surface posture; resource-protection wording restated as a safety property. Narrow Architecture Re-Review: `FURTHER NARROW CORRECTION REQUIRED` — all seven Resolved; two new Minor findings (`NAR-017-F01`, `NAR-017-F02`). |
| 0.2.1 | 2026-08-10 | Denver Jacobs (Founder) | Further Narrow Architecture Correction applying exactly `NAR-017-F01`–`F02`: §31 precised to distinguish Foundation raw-operation reachability from optional, undesigned Ergonomics convenience; §14's transport-capability citation corrected from the unrelated §37 to §44. Further Narrow Architecture Re-Review: `FURTHER CORRECTION REQUIRED` — both Resolved, all seven original findings remained closed; one new Minor finding (`FNAR-017-F01` — a malformed editorial fragment left inside §14). |
| 0.2.2 | 2026-08-10 | Denver Jacobs (Founder) | Single-Sentence Further Narrow Architecture Correction applying exactly `FNAR-017-F01`: replaced the malformed trailing sentence in §14 with clean normative prose. Final Single-Finding Further Narrow Architecture Re-Review: **PASS** — zero remaining findings of any severity across the entire four-stage review chain, independently re-verified at each stage. Records Founder Architecture Approval (§50, Founder Declaration quoted verbatim) and constitutes this document's own Repository Filing. `status` transitions from `Draft` to **`Approved`**. |

## 49. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author (v0.1.0) | Denver Jacobs (AI-assisted) | Drafted | 2026-08-10 |
| Independent Architecture Review (v0.1.0) | — | `REVISION REQUIRED` — 0 Critical, 3 Major, 4 Minor | 2026-08-10 |
| Founder Disposition (v0.1.0 review) | Denver Jacobs, Founder | Accepted in full; Narrow Architecture Correction authorized | 2026-08-10 |
| Author (v0.2.0, Correction) | Denver Jacobs (Founder) | Corrected — seven findings applied | 2026-08-10 |
| Narrow Architecture Re-Review (v0.2.0) | — | `FURTHER NARROW CORRECTION REQUIRED` — `F01`–`F07` Resolved, 2 new Minor | 2026-08-10 |
| Founder Disposition (v0.2.0 review) | Denver Jacobs, Founder | Accepted in full; Further Narrow Architecture Correction authorized | 2026-08-10 |
| Author (v0.2.1, Further Correction) | Denver Jacobs (Founder) | Corrected — two findings applied | 2026-08-10 |
| Further Narrow Architecture Re-Review (v0.2.1) | — | `FURTHER CORRECTION REQUIRED` — `NAR-017-F01`/`F02` Resolved, `IAR-017-F01`–`F07` closed, 1 new Minor | 2026-08-10 |
| Founder Disposition (v0.2.1 review) | Denver Jacobs, Founder | Accepted; Single-Sentence Further Narrow Architecture Correction authorized | 2026-08-10 |
| Author (v0.2.2, Single-Sentence Correction) | Denver Jacobs (Founder) | Corrected — one finding applied | 2026-08-10 |
| Final Single-Finding Further Narrow Architecture Re-Review (v0.2.2) | — | **PASS** — zero remaining findings | 2026-08-10 |
| Approval Authority | Denver Jacobs, Founder (interim, per `GOV-003` §3.2 vacancy) | **Approved** (verbatim Founder Declaration recorded in §50) | 2026-08-10 |

## 50. Disposition

**Approved.** Architecture Authoring (v0.1.0) → Independent Architecture Review (`REVISION REQUIRED`, 3 Major, 4 Minor) → Narrow Architecture Correction (v0.2.0) → Narrow Architecture Re-Review (`FURTHER NARROW CORRECTION REQUIRED`, 2 new Minor) → Further Narrow Architecture Correction (v0.2.1) → Further Narrow Architecture Re-Review (`FURTHER CORRECTION REQUIRED`, 1 new Minor) → Single-Sentence Further Narrow Architecture Correction (v0.2.2) → Final Single-Finding Further Narrow Architecture Re-Review (`PASS`, zero remaining findings) → Founder Architecture Approval, recorded in full below.

**Founder Architecture Approval granted.** Denver Jacobs, Founder, 2026-08-10, recorded verbatim:

> "Founder Architecture Approval — ARCH-017 v0.2.2. I accept the Final Single-Finding Further Narrow Architecture Re-Review PASS verdict and grant Founder Architecture Approval to ARCH-017 v0.2.2 — SynapseOS Runtime Observation / Connectivity Architecture (Phase 1: Local, Query-Only). Founder: Denver Jacobs. Decision: APPROVED. Date: 2026-08-10."

This Filing (v0.2.2) records that approval and constitutes this document's own Repository Filing. It does not authorize implementation of the Runtime Observation Service, the SDK Observation Client, any transport, or any authorization mechanism. It does not select GUI technology and does not authorize Control Centre implementation. It does not unblock `EWO-028` — that remains conditioned on this architecture's own eventual implementation, not on approval alone.
