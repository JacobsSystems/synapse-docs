---
document_id: ARCH-015
title: Developer Platform Boundary Architecture
project: SynapseOS
specification: SynapseOS — the narrowest cross-cutting architecture necessary to keep Developer Platform surfaces (CLI, Documentation Platform, SDK Documentation, Tutorials, Templates, Testing Tooling, Developer Portal, and the future Control Centre) coherent with the Runtime, the SDK, and each other, without becoming an umbrella architecture for any of them
version: 0.2.0
status: Approved
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.2 interim Chief Architect authority (GOV-010 §5, Class B) — no Chief Architect is currently appointed, the identical basis ARCH-007, ARCH-008, ARCH-011, ARCH-012, and ARCH-014 each already record for themselves.
created: 2026-08-10
last_updated: 2026-08-10
classification: Public
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.8-§6.11, the Architecture Authoring / Independent Architecture Review / Architecture Correction / Architecture Re-Review lifecycle this document's own history follows exactly)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution; §2, §4, §5 principle 2 — binding constraints throughout)
    - ACT-005 (v0.1.0, Approved — Developer Platform Era; §4 eleven programme domains, §4.H Testing Tooling boundary, §6 Control Centre boundary)
  roadmap:
    - ROAD-001 (v0.1.0, Approved — SynapseOS Platform Strategy and Roadmap)
  research:
    - RES-008 (v0.2.0, Founder-accepted — Developer Platform Landscape and Developer Workflow Research)
  architecture:
    - ARCH-007 (v0.5.2, Approved — Persistent Actor Architecture)
    - ARCH-008 (v0.5.0, Approved — Effect Runtime Architecture)
    - ARCH-011 (v0.1.3, Approved — Durable Storage Mechanics)
    - ARCH-012 (v0.2.0, Approved — Durable DomainState Encoding Architecture)
    - ARCH-014 (v0.8.0, Approved — Synapse SDK Architecture; the sole, exclusive authority for SDK structure this document never duplicates)
  adrs:
    - ADR-0020 (v0.1.0, Approved — Disposition of the Unrecoverable ARCH-013 Draft; the decision this document is the directly-authorized "narrower replacement" of, per its own §5/§7)
  consolidation:
    - DES-003 (v0.1.0, Draft, Founder-accepted, filed — Developer Platform Boundary: Design Exploration; the requirements/design baseline this document transforms into architecture)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-015 — Developer Platform Boundary Architecture

> **Filing Notice.** This document was authored, independently reviewed, corrected, and re-reviewed entirely as a chat-delivered artifact before this Repository Filing — the identical lifecycle shape `ARCH-007`, `ARCH-012`, and `ARCH-014` each established for themselves. Version 0.1.0 was the initial Architecture Authoring Draft, transforming `DES-003` v0.1.0's Founder-accepted candidate requirements into eleven invariants and six decisions. Its Independent Architecture Review returned `REVISION REQUIRED` (0 Critical, 2 Major, 5 Minor, 2 Observation), Founder-accepted in full. Version 0.2.0 was a strictly bounded Architecture Correction applying exactly the seven accepted findings (`IAR-015-F01`, `F02`, `F03`, `F05`, `F06`, `F09`, `F10`); `F07`/`F08` remained Observations, not converted into requirements. Its own Architecture Re-Review found zero remaining or correction-induced findings of any severity and returned `PASS`. Founder Architecture Approval is recorded in full below (§39, Disposition); this Filing (v0.2.0) records that approval and constitutes this document's own Repository Filing. It does not itself authorize CLI, Documentation Platform, Developer Portal, Control Centre, or any other domain's implementation.

## Document Control

| Field | Value |
|---|---|
| Document ID | ARCH-015 |
| Title | Developer Platform Boundary Architecture |
| Version | 0.2.0 |
| Status | **Approved** — Founder Architecture Approval recorded below (§39); the corrected, Re-Reviewed architecture, superseding v0.1.0's own unapproved Draft state |
| Author | Denver Jacobs (AI-assisted) |
| Approval authority | Chief Architect (Class B, per `GOV-010` §5), vacant; Founder (interim), per `GOV-003` §3.2 |
| Created | 2026-08-10 |
| Classification | Public |
| Requirements baseline | `DES-003` v0.1.0 (Founder-accepted, filed, commit `2631f51`) |
| Constitutional authority | `GOV-018` (Approved v0.2.0); `ACT-005` (Approved v0.1.0); `ARCH-014` (Approved v0.8.0); `ARCH-007` (Approved v0.5.2); `ARCH-008` (Approved v0.5.0); `ARCH-011` (Approved v0.1.3); `ARCH-012` (Approved v0.2.0); `ADR-0020` (Approved v0.1.0) |
| Numbering | `ARCH-015` — freshly verified at Authoring; unaffected by Correction or Re-Review |
| Review requirement (v0.1.0, complete) | Architecture Authoring → Independent Architecture Review → **REVISION REQUIRED** (0 Critical, 2 Major [`F01`,`F02`], 5 Minor [`F03`,`F05`,`F06`,`F09`,`F10`], 2 Observation [`F07`,`F08`]) — Founder-accepted in full |
| Review requirement (v0.2.0, complete) | Architecture Correction applying exactly `F01`, `F02`, `F03`, `F05`, `F06`, `F09`, `F10` → Architecture Re-Review → **PASS** (zero remaining/correction-induced findings) → Founder Architecture Approval |

---

## Executive Summary

This document establishes the narrowest Approved architecture necessary to keep SynapseOS Developer Platform surfaces coherent with each other and with the Runtime and SDK, without becoming an umbrella architecture for any of them. Version 0.1.0 transformed `DES-003`'s Founder-accepted candidate requirements into eleven invariants and six decisions; its Independent Architecture Review returned `REVISION REQUIRED` (2 Major, 5 Minor, 2 Observation, Founder-accepted in full). Version 0.2.0 applied exactly the seven accepted findings: corrected a defective citation to Draft `ARCH-002` (`F01`); added one new invariant, `DPB-INV-12` (State of Record), closing a genuine gap the review found — no Developer Platform Surface may become an independent authoritative source of truth for Runtime/SDK-owned state (`F02`); clarified that the CLI must never become a hidden mandatory substrate for *other* Developer Platform surfaces, not only for applications (`F03`); clarified that a stability tier is not proof of version compatibility (`F05`); clarified that Packaging remains bound by the general invariants despite having no packaging-specific rule yet (`F06`); downgraded `DPB-INV-11` (Stability Disclosure) from a binding invariant to non-binding guidance, `NBG-01` (`F09`); and softened an overstated `DES-002` R21 dependency-resolution claim (`F10`). Net invariant count remained eleven (`DPB-INV-01`–`10`, `DPB-INV-12`) — one removed, one added, disclosed as coincidental. Its Architecture Re-Review found zero remaining or correction-induced defects and returned `PASS`. Founder Architecture Approval is recorded in full below (§39). No SDK, Runtime, CLI, Documentation Platform, or Control Centre architecture is designed or implemented by this document.

---

## 1. Context

`ADR-0020` disposed of the unrecoverable `ARCH-013` draft and identified that certain cross-cutting Developer Platform principles it once carried now lack a complete Approved architectural home. `DES-003` (Founder-accepted 2026-08-10) explored this gap as new design work and produced seventeen candidate requirements. This document is the Architecture Authoring stage `GOV-013` §6.8 defines for that accepted design: transforming decisions already made into normative architecture, introducing no new design question `DES-003` did not itself raise.

## 2. Scope

The relationship between Runtime, SDK, cross-cutting Developer Platform concerns, and domain-specific Developer Platform components. Nothing more.

## 3. Non-Goals

Runtime redesign; SDK layer redesign; stable SDK v1.0.0 API redesign; Documentation Platform, CLI, Developer Portal, Reference Application, Testing Tooling, Packaging, or Control Centre implementation or technology selection; Distributed Runtime; Synapse Cloud; Enterprise Edition; AI Workforce Platform; any commercial application design. Verified unviolated at Authoring, Correction, and Re-Review alike.

## 4. Method

Every invariant satisfies the Minimal Architecture Principle test (§9): genuinely needed by more than one domain; would produce meaningful inconsistency if left domain-specific; protects a real Runtime/SDK/security boundary; is architecture, not UX; cannot be safely left to domain ownership alone. `DPB-INV-11`'s failure to clearly satisfy this test is precisely why it no longer appears as a binding invariant (§22).

---

## 5. Authority and Dependencies

### 5.1 Authority Classification

| Tier | Documents | Status |
|---|---|---|
| Constitutional / Strategic | `GOV-018`, `ROAD-001`, `ACT-005` | Approved |
| Existing Approved Architecture | `ARCH-007`, `ARCH-008`, `ARCH-011`, `ARCH-012`, `ARCH-014` | Approved |
| Approved ADR Decisions | `ADR-0020` | Approved |
| Research Evidence | `RES-008` | Founder-accepted, `status: Draft` |
| Design Requirements | `DES-003` | Founder-accepted, `status: Draft` |
| This document | `ARCH-015` | **Approved**, v0.2.0 |

**Disclosed finding, not a STOP condition.** `ARCH-001`, `ARCH-002`, and `ADR-0016` remain `status: Draft`, never Founder Architecture Approved — a pre-existing repository condition. **This document does not rely on `ARCH-001`, `ARCH-002`, or `ADR-0016` as authority for any invariant it states** — every invariant's citation was independently re-checked at Correction (`F01`) and again at Re-Review to confirm it cites only the independently, separately Approved downstream tier. *(Corrected v0.2.0: v0.1.0's §18 broke this rule by citing `ARCH-002` §18 directly — see §18, below. This is the only place in the document where the `ARCH-002`/`ARCH-001`/`ADR-0016` relationship is discussed; no other section relies on them, independently re-verified at Re-Review.)*

### 5.2 ARCH-013 Historical Boundary

`ADR-0020` controls. `ARCH-013` was Draft, never filed, never Founder Architecture Approved, is confirmed unrecoverable, and its identifier is permanently retired. It provides **no independent architectural authority**. Where this document's principles historically trace to it, the wording used throughout is: *"a principle historically associated with the unrecoverable `ARCH-013` draft survives through Approved `ARCH-014`"* or equivalent — never *"`ARCH-013` requires..."* as a source of current authority. This document is new architecture; it claims no textual continuity with `ARCH-013`. Independently re-verified clean at both Independent Architecture Review and Architecture Re-Review.

---

## 6. Architectural Mission

**What cross-cutting architectural rules must all SynapseOS Developer Platform surfaces obey so that they remain coherent with the Runtime, SDK, security model, and each other?** This document does not architect any individual domain. It exists only to prevent domains independently inventing authority, semantics, compatibility rules, developer concepts, security behavior, stability expectations, or control mechanisms — and only where a shared rule is genuinely necessary (§9).

## 7. Architectural Ownership Model

Four zones:

- **Runtime** — execution authority and mechanisms. Owned exclusively by `ARCH-007`/`008`/`011`/`012` and their own successors. Not touched here.
- **SDK** — the stable programmatic interface. Owned exclusively by `ARCH-014`. Not touched here.
- **Developer Platform Boundary** (this document) — invariants that must hold identically across every Developer Platform Surface (§8), because defining them independently per-surface would produce inconsistent authority, security posture, or developer mental models (§9).
- **Domain-Specific** — everything else: technology, presentation, command syntax, layout, content. Owned by each domain's own future architecture.

## 8. Developer Platform Surface Definition

A component is a **Developer Platform Surface**, and therefore subject to this architecture, if and only if it satisfies both:

1. It is built for use by SynapseOS application developers, contributors, or operators — never a tenant application's own end-user; and
2. It exposes, composes, presents, or acts upon legitimate Runtime/SDK operations, concepts, or their results, in any form (command, page, generated code, test harness, GUI action, package artifact).

**Explicitly included, once built:** CLI, Documentation Platform, SDK Documentation, Interactive Tutorials, Official Templates, Testing Tooling, Developer Portal, Control Centre developer-facing functions, Packaging & Distribution tooling that developers interact with directly.

**Explicitly excluded:** the Runtime and SDK themselves (owned elsewhere, §7); tenant applications built on SynapseOS (not part of the platform); internal engineering tooling not distributed to or used by external developers.

## 9. Minimal Architecture Principle

For every candidate rule: (1) does more than one domain genuinely need it; (2) would independent per-domain definitions create meaningful inconsistency; (3) does it protect a real Runtime/SDK/security boundary; (4) is it architecture rather than UX; (5) could a domain safely own this itself? Applied literally throughout §26.

---

## 10. Runtime Authority Boundary

**Normative rule:** Developer Platform Surfaces MUST NOT independently create execution authority or execution semantics the underlying Runtime/SDK does not already possess. Every operation a surface exposes MUST be traceable to a specific, existing Runtime or SDK operation (§11, Authority Projection). This is the direct architectural consequence of the Trusted-Core-exclusivity pattern as it survives, restated, through the independently-Approved `ARCH-007` §5/`ARCH-008` §5 ("this document amends no prior authority... extends, and is bound by...") and `ARCH-014` §7 ("the SDK, as a whole, owns no authority, no state, and no enforcement decision the Runtime does not already own"). This document generalizes that same rule one layer further outward, to every Developer Platform Surface, not only the SDK. **Tested against:** CLI (every command must translate to real SDK/Runtime operations); documentation examples (must compile against the real stable SDK, `DES-002` R06); templates (must use only approved SDK patterns); testing tools (mocking must not silently disable capability checks in a way that misrepresents production behavior — test doubles MUST be structurally distinguishable from real grants); Developer Portal (actions must map to real operations, no shadow state); future Control Centre (out of scope per `ACT-005` §6, but this rule pre-applies the moment that boundary is crossed, §24). **The Developer Platform Boundary MUST NOT become a second Runtime.**

Execution authority (this section) and state-of-record authority (§10.1, below) are related but distinct risks: the first governs what operations a surface may *perform*; the second governs what a surface may *claim as currently true*. Both are corollaries of the same underlying Non-Second-Runtime principle (§5 of this document's own authoring history).

### 10.1 State of Record

**`DPB-INV-12`.** No Developer Platform Surface — including CLI, Developer Portal, Control Centre, documentation tooling, testing tooling, automation tooling, or any future surface — may become an independent authoritative source of truth for platform state that is owned by the Runtime, the SDK, or another legitimate subsystem (§7, Architectural Ownership Model). A Developer Platform Surface MAY cache, project, index, present, or derive state where useful. It MUST NOT allow such a representation to silently replace or override the legitimate system of record: any consumer that requires *current* truth MUST be able to obtain it, ultimately, from the owning subsystem itself, not merely from a surface's own derived copy, however convenient.

**Rationale.** The Independent Architecture Review's Second-Runtime Test and Control Centre Scenario C both constructed hypothetical designs — a CLI or Developer Portal whose local cache of "known actors" becomes the thing other tooling trusts instead of the Runtime itself — that complied with every v0.1.0 invariant while recreating exactly the confused-deputy risk pattern `RES-008` §9 (S7) documents in a comparable external system (MCP's CVE-2025-49596). This invariant closes that gap.

**Tested against:** CLI (a local project-status cache is fine; treating that cache as authoritative for a *different* tool's decision-making is not); Developer Portal (a dashboard may display cached actor state; it must not become the system another surface queries instead of the Runtime); future Control Centre (§24 — this is the specific rule Control Centre Scenario C required and v0.1.0 did not previously have); testing tooling (a test harness's own simulated state is explicitly not platform state at all, and is therefore outside this invariant's scope, not a violation of it — test doubles remain governed by §10's own existing rule on structural distinguishability).

**Scope discipline.** This invariant is architectural only. It selects no storage, synchronization, database, caching, or eventing technology, and specifies no mechanism by which "current truth" must be obtainable (polling, push, query API, or otherwise) — that remains future, domain-specific architecture (§29, Deferred Architecture).

## 11. Authority Projection

Every operation a Developer Platform Surface exposes MUST be answerable against seven questions: (1) what underlying Runtime/SDK operation actually occurs; (2) which subsystem owns it (§7); (3) what authority permits it; (4) what capability is required; (5) what observable/auditable result follows (§21); (6) what error occurs when authority is absent (§22); (7) what is the current, authoritative source of record for any resulting state, and how would a consumer needing current truth reach it (§10.1). This is §10/§10.1's own operational, testable form — distinct from the Generalization Test (§14), the acceptance procedure applied to a *proposed new convenience before it is built*.

## 12. SDK Boundary

`ARCH-014` remains the **sole, exclusive** authority for SDK layer structure, public API classification, extension architecture, ergonomics, and compatibility. This document owns none of it and amends none of it. The dividing line between legitimate Developer Platform convenience and an unofficial parallel SDK: **the verbosity test** (`ARCH-014` §4's own generalization test, applied here at the Developer Platform boundary) — can the same effect be achieved, only more verbosely, using documented Stable/Supported-tier SDK calls alone? If yes, it is tooling convenience, governed by this document. If no — if it requires semantics no existing SDK call expresses — it is an SDK proposal requiring `ARCH-014`'s own amendment process, never silently introduced as Developer Platform tooling.

## 13. CLI Boundary

The CLI is a control and workflow surface over legitimate platform operations, never an independent authority system. This is §10/§11/§12 applied to one specific, single domain — no separate architectural invariant is created for the CLI alone (§27, `AD-02`). What the CLI may own: scaffolding conventions, local workflow sequencing, diagnostic presentation, ergonomics — orchestration policy, never security/capability policy. `DES-002` R02's manual-Cargo-workflow independence is preserved and generalized by §19 (Tool Independence). No CLI framework, command syntax, argument parser, shell integration, or distribution mechanism is selected here.

**The CLI is an optional Developer Platform Surface.** No other Developer Platform Surface — future GUI/Control Centre, Developer Portal, Documentation Platform, Testing Tooling, Packaging, or automation tooling — may be architecturally *required* to shell out to, wrap, or otherwise depend upon the CLI merely to reach a legitimate SynapseOS operation that is also, independently, reachable through the SDK directly. This does not prohibit an implementation from *choosing* to reuse legitimate, shared, lower-level SDK/platform functionality that happens also to be exposed through the CLI — it prohibits the CLI becoming the *only* path, silently, by accumulation, exactly as `DPB-INV-04` (§19) already prohibits it becoming the only path *for applications*. This clarification does not authorize CLI architecture or implementation.

## 14. Generalization Principle

**Invariant `DPB-INV-02`**: a Developer Platform convenience is architecturally legitimate only if fully explainable as a simpler or safer composition of existing, legitimately-owned platform operations (§11), introducing no new execution semantics, such that removing the convenience leaves the underlying application model valid. **On failure:** the proposal does not become Developer Platform architecture by default — it routes to whichever existing lifecycle mechanism actually owns the gap (a domain's own future architecture; an `ARCH-014` amendment, if SDK-scoped per §12; a Runtime `ADR`/architecture proposal, if Runtime-scoped) or is rejected outright. No new governance mechanism is invented — `GOV-013`'s existing lifecycle is reused unmodified.

*(Observation, disclosed, not elevated: the verbosity test above is enforceable per-operation only; no invariant addresses whether a large, individually-compliant collection of conveniences could cumulatively function as a de facto alternate SDK entry point — carried forward as a watch item for future CLI/Templates domain architecture, per `IAR-015-F07`.)*

## 15. Progressive Disclosure

**Invariant `DPB-INV-05`**: increased Developer Platform convenience or abstraction MUST NOT create a fundamentally different platform model — a beginner and an expert MUST ultimately be interacting with the same underlying SynapseOS concepts (§16). Two testable clauses: **no omission** (no domain's minimum-success path may omit or soften a security/correctness concept — generalizing `DES-002` §4's binding rule, itself sourced from `GOV-018` §5.2) and **no unlearning** (nothing learned at an earlier stage may require unlearning at a later one — generalizing `ARCH-014` §16's SDK-specific journey rule). Screen layout, visual design, and specific tutorial content remain domain-specific.

## 16. Developer Mental Model

**Invariant `DPB-INV-06`**: developer-facing references to a constitutional concept MUST use terminology and relationships consistent with `GOV-018` §4's four non-negotiable properties (identity, explicit capability, governed messaging, observable/auditable action) and the Approved architecture that defines each (`ARCH-007` supervision, `ARCH-011`/`ARCH-012` durable state, `ARCH-008` effects/audit, `ARCH-014` §11 public type surface). This standardizes **meaning**, never presentation or ordering.

## 17. Capability Security

**Invariant `DPB-INV-07`**, three corollaries of the already-binding `GOV-018` §5 principle 2 rule that "every capability is minted, attenuated, or revoked explicitly, never assumed" — a rule this document consumes, never redefines: **no silent grant** (no Developer Platform path may grant capability without an explicit, visible, attributable step); **least-authority default** (official templates MUST default to the minimum capability set their own demonstrated purpose requires); **capability visibility** (any surface executing a capability-requiring operation MUST make that requirement discoverable from that surface).

## 18. Auditability

**`DPB-INV-08`**: any Developer Platform action causing a platform-visible change (a CLI-driven capability change, a future deployment/package action, a Developer Portal or Control Centre configuration change) MUST route through the same audited Runtime/SDK operations already producing the audit trail — never a parallel, tooling-level log. This is §10's auditability-specific corollary, not an independently invented audit subsystem. The actual audit mechanism's owner remains the Runtime; this document only binds Developer Platform surfaces never to bypass it. The authority for this invariant is `ARCH-008`'s own audit-event obligations directly (Approved, v0.5.0) — independently, separately Approved authority sufficient on its own.

## 19. Tool Independence

**Invariant `DPB-INV-04`**: core SynapseOS application correctness MUST NOT depend on optional Developer Platform convenience tooling (CLI, templates, generators, Developer Portal, Control Centre, packaging helpers). Every domain MUST disclose, per major feature, whether a manual/tool-independent path exists using only documented Stable-tier SDK usage. A manual path need not be pleasant, only exist — this is architectural dependence, not convenience parity. Direct generalization of `DES-002` R02, evidenced today by `DX-001`'s own demonstrated durable-actor application requiring zero Developer Platform tooling.

## 20. Error Architecture

**Invariant `DPB-INV-09`**: every developer-facing failure presentation, in any Developer Platform Surface, MUST remain traceable to `ARCH-014` §12's existing developer-facing-error-over-`RuntimeError` vocabulary — no domain may invent a parallel, disconnected error taxonomy. **Error semantics** (owned exclusively by `ARCH-014` §12, unchanged, unduplicated here) are separated from **error presentation** (domain-specific, unconstrained beyond vocabulary consistency). No error type, code, or schema is specified here (§29).

## 21. Compatibility Architecture

**`DPB-INV-10`**: any Developer Platform Surface presenting or depending on version-specific SDK/Runtime behavior MUST be able to state the version range it is valid for, using `ARCH-014` §14's existing compatibility-tier vocabulary — not a new scheme. No metadata format, file, or mechanism is chosen here (§29).

**A stability tier (§22) is not, by itself, proof of version compatibility.** `ARCH-014` §14 defines a compatibility-*expectation* vocabulary graded by tier (Stable/Supported/Experimental/Internal) and a deprecation philosophy — it does not define a version-*range* expression vocabulary. A surface stating "this operation is Stable-tier" has satisfied the tier half of this invariant; it has not thereby established which concrete Runtime/SDK version range it is compatible with. Compatibility truth MUST continue to derive from the legitimate owning architecture or mechanism, never independently inferred or guessed by a Developer Platform Surface itself. No compatibility-checking implementation mechanism is chosen here — tier and version-range are distinct obligations, both ultimately required, neither substituting for the other.

## 22. Stability / Disclosure

**`NBG-01` (Non-Binding Guidance, formerly `DPB-INV-11`).** On independent re-examination during the Independent Architecture Review, this concern — that non-SDK Developer Platform Surfaces might need a way to disclose their own stability status, independent of the SDK tier of what they wrap — was found to weakly or not clearly satisfy three of the Minimal Architecture Principle's five criteria (§9): it does not clearly protect a Runtime/SDK/security boundary, its architectural-versus-UX/product-policy character is genuinely borderline, and a domain could plausibly own it safely alone. The Founder accepted this finding and directed that it **no longer be treated as a binding cross-cutting architectural invariant**, on the same basis `DPB-17` (§23, `AD-03`) was already, independently, given identical treatment for materially similar reasons.

**Preserved, not deleted.** The underlying concern remains recorded as **non-binding architectural guidance**: a CLI command, template, or testing-tool feature *may* disclose its own stability status (e.g., stable/preview/experimental), independent of, and not automatically inherited from, the SDK tier of the operations it wraps. Any future domain architecture MAY adopt this guidance and elevate it to its own binding, domain-scoped rule if that domain's own evidence independently justifies it — this document neither requires nor forbids that. Full traceability to `DES-003` `DPB-16` is preserved in §32 (Traceability) and `AD-08` (§27).

## 23. Automation / Machine Consumers

No invariant is created here. `RES-008` §16 Minor-1's non-human documentation-traffic finding is Medium confidence with no traceable primary source — too weak to support a formal architectural obligation. `DES-003`'s own `DPB-17` (non-interactive path availability) is not elevated to an invariant (§27, `AD-03`) — it applies to only two domains, is sourced from general engineering practice rather than any SynapseOS-specific governing document, and remains better carried as non-binding future guidance (§29) pending either stronger evidence or its own domain architecture's independent judgment.

## 24. Control Centre Boundary

Not designed here (§3, Non-Goals; `ACT-005` §6 does not authorize it). The architectural rules the future Control Centre MUST eventually inherit, once it exists, are exactly §10-§22 above, applied without exception: it MUST NOT invent operations unavailable elsewhere (§10); MUST NOT bypass capabilities (§17); its own state MUST NOT become authoritative platform state (`DPB-INV-12`, §10.1); GUI workflows MUST NOT change Runtime semantics (§10); GUI actions MUST preserve auditability (§18); operations MUST map to legitimate underlying platform operations (§11, seven questions); applications MUST NOT require the GUI to function (§19); the Control Centre MUST NOT become an architecturally mandatory consumer of the CLI as its own only path to platform operations (§13); GUI version compatibility MUST relate to Runtime/SDK compatibility through the same contract, distinguishing tier from version-range (§21). **Governing principle:** the Control Centre becomes a powerful projection of SynapseOS capabilities, never a privileged backdoor into SynapseOS. No GUI framework, rendering technology, or transport mechanism is chosen here.

## 25. Documentation Platform Dependency

| DES-002 Dependency | Boundary Architecture Resolution | Owner | Status |
|---|---|---|---|
| R02 (CLI independence) | `DPB-INV-04` (§19), strengthened by §13's non-substrate clarification | This architecture | **Resolved by this new architecture** |
| R21 (disclosure tiering) | `DPB-INV-05` (§15) | This architecture (principle only); Documentation Platform (obligation + mechanism) | **Partially resolved.** `DPB-INV-05`'s no-omission/no-unlearning clauses establish the underlying principle R21 depends on, but do not themselves establish R21's own specific obligation that every documented concept must be assigned a disclosure tier — that per-concept assignment obligation, and its marking mechanism, both remain open, owned by Documentation Platform architecture |
| R04/R25 (capability grant/denial documented) | `DPB-INV-07` (§17) | This architecture (rule); Documentation Platform (content) | **Resolved by this new architecture** |
| R07 (version-awareness) | `DPB-INV-10` (§21), distinguishing tier from version-range | This architecture (contract); Documentation Platform (mechanism) | Contract resolved; mechanism Deferred |
| R11/R26 (baseline measurement) | None | Documentation Platform | Remains domain-specific |
| R09 (RuntimeError catalogue) | `DPB-INV-09` (§20) | This architecture (vocabulary); Documentation Platform (content) | **Resolved by this new architecture** |
| §10 (machine-readable representation) | None (declined) | Documentation Platform | Remains Deferred, not blocking |
| C02 (no AI-specific construct) | `GOV-018` §2 directly | Constitutional | Already resolved elsewhere |

## 26. Domain Ownership Matrix

| Concern | Runtime | SDK | Developer Platform Boundary | Domain |
|---|---|---|---|---|
| Execution semantics | ✅ Owner | — | — | — |
| Actor lifecycle | ✅ Owner (`ARCH-007`) | Exposed via SDK | — | — |
| Effects | ✅ Owner (`ARCH-008`) | Exposed via SDK | — | — |
| Durable state | ✅ Owner (`ARCH-011`/`012`) | Exposed via SDK | — | — |
| Supervision | ✅ Owner (`ARCH-007`) | Exposed via SDK | — | — |
| Capabilities | ✅ Owner | Exposed via SDK | `DPB-INV-07` (visibility/least-authority rules only) | — |
| State of record | ✅ Owner | Exposed via SDK | `DPB-INV-12` (§10.1 — surfaces may not become an independent source of truth) | Owner (own caches/projections, non-authoritative) |
| Audit semantics | ✅ Owner (mechanism, `ARCH-008`) | — | `DPB-INV-08` (never-bypass rule) | — |
| SDK ergonomics | — | ✅ Owner (`ARCH-014`) | — | — |
| CLI workflow | — | — | `DPB-INV-01`/`02`/`03`/`04` (boundary only, §13) | ✅ Owner (commands, syntax) |
| Documentation presentation | — | — | `DPB-INV-05`/`09`/`10` (rules consumed) | ✅ Owner |
| Developer mental model | — | — | `DPB-INV-06` (meaning only) | ✅ Owner (presentation) |
| Compatibility | — | ✅ Owner (tier vocabulary, `ARCH-014` §14) | `DPB-INV-10` (contract requirement; tier ≠ version-range, §21) | Owner (mechanism) |
| Stability disclosure | — | ✅ Owner (SDK tiers, `ARCH-014` §8) | `NBG-01` (non-binding, formerly `DPB-INV-11`, §22) | Owner, if adopted |
| Error semantics | — | ✅ Owner (`ARCH-014` §12) | `DPB-INV-09` (consistency requirement) | — |
| Error presentation | — | — | — | ✅ Owner |
| Templates | — | — | `DPB-INV-02`/`07` (constraints) | ✅ Owner |
| Testing | — | — | `DPB-INV-01` (never weaken Runtime semantics, `ACT-005` §4.H) | ✅ Owner |
| Packaging | — | — | `DPB-INV-01`/`07`/`08`/`12` apply generally — no packaging-specific rule exists yet (footnote, below) | ✅ Owner (eventual) |
| Search/discovery | — | — | — | ✅ Owner |
| Developer identity/account | — | — | Genuinely unowned — Deferred, §29 | Unclear, unconfirmed |
| Future Control Centre actions | Out of scope (`ACT-005` §6) | — | §10.1/§13/§24 pre-apply once built | — |

**Footnote.** *"Unexplored" (this document's own earlier working note for Packaging) meant, and means, only that no Packaging-*specific* cross-cutting rule has yet been identified — it never meant, and does not mean, that Packaging is exempt from the general invariants above. Every Developer Platform Surface satisfying §8's definition, including Packaging & Distribution tooling developers interact with directly, remains fully bound by `DPB-INV-01`/`07`/`08`/`12` and every other applicable invariant. This reopens no packaging format, technology, or `SRP-001` decision — none is touched here.*

*(Reference Applications and Community/Commercial Adoption Foundations remain without dedicated rows: no domain-specific boundary question was found for them that does not already reduce to the invariants above.)*

---

## 27. Architecture Decisions

**AD-01 — Boundary Scope.** *Problem:* whether to author a full umbrella Developer Platform architecture or a narrow boundary layer. *Decision:* narrow boundary only (§6, §9). *Rationale:* `ADR-0020` §5/§7 explicitly scoped the future replacement to orphaned cross-cutting principles, not a full umbrella. *Alternatives considered:* full Developer Platform Architecture covering all eleven `ACT-005` domains — rejected, `ADR-0020` §7 explicitly prohibits this. *Consequences:* every individual domain still requires its own future architecture. *Amendment trigger:* if a future domain architecture repeatedly needs a rule this document doesn't provide, that is evidence for a narrow future amendment here, not a redesign of the domain's own architecture.

**AD-02 — CLI Non-Invariant.** *Decision:* no dedicated CLI invariant — fully absorbed into `DPB-INV-01`/`03` applied to one domain (§13). *Rationale:* Minimal Architecture Principle (§9) — a single-domain-scoped rule already fully entailed by a cross-domain invariant adds no independent obligation. **Clarification (v0.2.0):** the Independent Architecture Review found this original claim did not address one narrower risk — a future Control Centre or other surface making CLI its *own* mandatory internal substrate, as opposed to making CLI mandatory for *applications*, which `DPB-INV-04` already prevents. §13's clarifying paragraph closes this gap without creating a dedicated CLI invariant, preserving AD-02's original core judgment while correcting its scope claim.

**AD-03 — Automation Guidance Deferral.** *Decision:* `DPB-17` deferred as non-binding guidance (§23, §29), not elevated. *Rationale:* weakest evidentiary basis of any `DES-003` item, applies to only two domains. Independently reviewed as **SOUND** at Independent Architecture Review — no correction required.

**AD-04 — No New Governance Mechanism for Generalization-Test Failures.** *Decision:* route Generalization Test failures to the existing, applicable `GOV-013` lifecycle mechanism, or reject outright — no new escalation process invented.

**AD-05 — Control Centre Projection Principle.** *Decision:* state only that §10-§22's invariants pre-apply once the Control Centre is built, framed as "projection, never backdoor" (§24) — no further design. *(v0.2.0: §24 now cites `DPB-INV-12` and §13's clarification directly, closing the gap the Independent Architecture Review found in this decision's original "inherits §10-§22" claim.)*

**AD-06 — Reference Applications / Community / Commercial Non-Rows.** *Decision:* no dedicated Domain Ownership Matrix rows for these three `ACT-005` domains (§26). *Rationale:* no domain-specific boundary question was found for them beyond what the existing invariants already cover.

**AD-07 — State of Record Invariant.** *Problem:* v0.1.0 stated a non-authoritative-state rule only narratively, for the Control Centre alone (§24), with no corresponding §26 invariant — a gap the Independent Architecture Review's Second-Runtime Test and Control Centre Scenario C both exploited. *Decision:* add `DPB-INV-12` (§10.1), generalizing the rule to every Developer Platform Surface. *Rationale:* the risk is not Control-Centre-specific; `RES-008` §9's own MCP confused-deputy CVE finding is direct external corroboration this class of risk is real and exploited elsewhere. *Alternatives considered:* leave the rule narrative-only in §24 — rejected; the Founder's own explicit decision required an invariant. *Consequences:* §11's Authority Projection test gained a seventh question. *Compatibility impact:* none — no existing Approved architecture touched. *Amendment trigger:* if a future domain architecture finds this invariant's "current, authoritative source of record" language insufficiently precise for its own mechanism design, that is grounds for a future narrow clarification, not a domain-level workaround.

**AD-08 — DPB-INV-11 Downgrade.** *Problem:* `DPB-INV-11` was found to weakly or not clearly satisfy three of the Minimal Architecture Principle's five criteria — the weakest justification of any v0.1.0 invariant, comparable to `DPB-17`, which AD-03 already deferred. *Decision:* downgrade to non-binding guidance, `NBG-01` (§22), preserving content and traceability rather than deleting it. *Rationale:* consistency — aligning treatment of two similarly weakly-evidenced items. *Alternatives considered:* retain as binding, disclosed as weakest-justified — available, but the Founder chose downgrade. *Amendment trigger:* a future domain architecture's own independent evidence could justify re-elevating this concern, domain-locally or, with genuinely cross-cutting evidence, via a future narrow amendment here.

## 28. Constitutional Acceptance Tests

1. **Runtime Test** — does this create execution semantics outside Runtime authority? (§10)
2. **Authority Projection Test** — can this exposed operation answer all seven §11 questions? (§11)
3. **Generalization Test** — is this convenience fully explainable as composition, with no new semantics, and would the application remain valid without it? (§14, `DPB-INV-02`)
4. **SDK Test** — does this duplicate or bypass `ARCH-014`'s SDK architecture? (§12, the verbosity test)
5. **Tool Independence Test** — would a valid application cease to be valid without this convenience? (§19, `DPB-INV-04`)
6. **Mental Model Test** — does this create a conflicting meaning for an established SynapseOS concept? (§16, `DPB-INV-06`)
7. **Capability Test** — does this silently grant, hide, or bypass capability? (§17, `DPB-INV-07`)
8. **Audit Test** — does this bypass required platform audit semantics? (§18, `DPB-INV-08`)
9. **Domain Ownership Test** — is this genuinely cross-cutting, or should the domain own it? (§9)
10. **Simplicity Test** — does this reduce developer complexity, or merely relocate internal complexity onto the developer? (§31)

## 29. Deferred Architecture

Exact CLI, Documentation Platform, Developer Portal, Control Centre, and Packaging architectures; the machine-readable documentation representation format; the specific accessibility compliance standard; external developer validation of every hedged hypothesis (§33); the stability-label mechanism, if a future domain architecture adopts `NBG-01`'s guidance (§22); the compatibility-metadata mechanism (`DPB-INV-10`, §21); the error-contract types/codes (`DPB-INV-09`); the concrete auditability mechanism (`DPB-INV-08`); the state-of-record reachability mechanism (`DPB-INV-12`/§10.1); developer identity/account ownership; whether Progressive Disclosure requires a formally documented "minimum success path" artifact; Distributed Runtime concerns; `DES-003`'s own `DPB-17` non-interactive-path guidance (§23, `AD-03`).

---

## 30. Developer Choice Assessment

| Invariant group | Classification | Explanation |
|---|---|---|
| `DPB-INV-01`/`02`/`03` | Indirect / Constitutional necessity | Prevents drift that would erode the platform's own structural differentiator (`RES-008` §13); the Generalization Test has no independent developer value, retained as `INV-01`'s enforcement mechanism |
| `DPB-INV-04` | **Direct** | Removes lock-in risk, strengthened by §13's CLI-substrate clarification |
| `DPB-INV-05` | Indirect | Reduces relearning cost, `RES-008` §11 |
| `DPB-INV-06` | Indirect, `[INF]` | Plausible, not evidenced |
| `DPB-INV-07` | **Direct**, enterprise-evaluator persona specifically | Concretizes `RES-008` §13's own claim; not yet evidenced for general application-developer persona |
| `DPB-INV-12` | Indirect | Prevents a class of trust erosion (a tool's own state silently diverging from platform truth); plausible, `[INF]`, not yet evidenced |
| `DPB-INV-08` | Constitutional necessity | Direct corollary of `GOV-018` §4 item 4 |
| `DPB-INV-09` | Indirect | Plausible relearning-cost reduction |
| `DPB-INV-10` | No meaningful independent value, but necessary | Hygiene — absence repels, presence is not itself a choice-driver; strengthened by tier/version-range distinction |
| `NBG-01` (relabeled) | No meaningful independent value, but potentially useful hygiene, if adopted | No longer binding; classification retained for whichever future domain architecture may adopt it |

## 31. Simplicity Assessment

**Test:** developers should not need to understand this document to build a basic SynapseOS application. **Result: satisfied**, directly testable — `DES-002` R02 and `DX-001` already demonstrate a complete durable-actor application via documented Stable-tier SDK usage and the manual Cargo workflow alone, zero Developer Platform tooling required. `DPB-INV-04`/`12` are precisely what structurally guarantee this remains true once CLI, templates, and Documentation Platform exist.

## 32. Traceability

| Item | GOV-018 | ARCH-014 | ARCH-007/008/011/012 | ADR-0020 | DES-003 | RES-008 |
|---|---|---|---|---|---|---|
| `DPB-INV-01` | §4/§5 | §6/§7 | §5 (each) | §5 | DPB-01 | — |
| `DPB-INV-02` | — | §4 | — | §5 | DPB-02 | — |
| `DPB-INV-03` | — | §6/§8 | — | §2 item 8 | DPB-03 | — |
| `DPB-INV-04` | — | — | — | §4 | DPB-05/14 | §5 |
| `DPB-INV-05` | §5.2 | §16 | — | §4 | DPB-06/07 | §11 |
| `DPB-INV-06` | §4 | — | ARCH-007/008/011/012 | — | DPB-08 | — |
| `DPB-INV-07` | §5 principle 2 | — | ARCH-002 §9 (disclosed, not relied on directly) | — | DPB-09/10/11 | §13 |
| `DPB-INV-12` | §4/§5 | — | — | — | *(none directly — identified by the Independent Architecture Review, `IAR-015-F02`, conceptually adjacent to `DPB-01`'s Non-Second-Runtime family but not itself an original `DES-003` item)* | §9 (S7), MCP confused-deputy CVE, external corroboration only |
| `DPB-INV-08` | §4 item 4 | — | ARCH-008 (audit events) | — | DPB-12 | — |
| `DPB-INV-09` | — | §12 | — | — | DPB-13 | — |
| `DPB-INV-10` | — | §14 | — | — | DPB-15 | — |
| `NBG-01` (relabeled) | — | §8 (distinguished from) | — | — | DPB-16 | — |

**Provenance disclosure.** `DPB-INV-12` is the one invariant in this document that does not trace to a `DES-003` requirement — it traces directly to the Independent Architecture Review's own finding (`IAR-015-F02`), Founder-accepted, disclosed explicitly rather than retrofitted to appear pre-anticipated.

## 33. External Developer Evidence Limitation

`RES-008` §15's limitation is unchanged and binding: **zero SynapseOS external-user evidence exists.** Every "Indirect"/`[INF]` classification in §30 is disclosed hypothesis pending future developer/enterprise validation, never fact.

## 34. Architecture Risks

| Risk | Mitigation |
|---|---|
| Second-Runtime risk | `DPB-INV-01`/`02` (§10, §14) |
| State-authority risk (a surface's own cache becoming trusted as truth) | `DPB-INV-12` (§10.1) |
| Parallel-SDK risk | `DPB-INV-03` (§12), verbosity test |
| Aggregate parallel-SDK risk (many individually-compliant conveniences cumulatively forming an alternate SDK) | Not fully mitigated — disclosed Observation (`IAR-015-F07`), carried forward as a watch item for future CLI/Templates domain architecture, not corrected further per explicit Founder instruction |
| Over-centralization | Eleven invariants; `DPB-INV-01`/`03`/`04`/`12` explicitly disclosed as one family, not four independently invented ones |
| Premature abstraction | `DPB-17` deferred (`AD-03`); `DPB-INV-11` downgraded to `NBG-01` (`AD-08`) |
| Domain capture | §3 Non-Goals; §26 defaults every concern Domain-Specific unless the Minimal Architecture Principle test is met |
| UX encoded as architecture | §20-§22 separate semantics (owned here) from presentation/mechanism (deferred, §29) |
| Excessive conceptual burden on developers | §31 Simplicity Assessment — satisfied |
| Capability-security weakening | `DPB-INV-07` (§17) |
| Audit bypass | `DPB-INV-08` (§18), correctly sourced to `ARCH-008` alone |
| Incompatible developer surfaces | `DPB-INV-06`/`09`/`10` (§16, §20, §21) |
| CLI dependence (application-level) | `DPB-INV-04` (§19) |
| CLI dependence (tool-on-tool substrate) | §13's clarification |
| Control Centre privilege escalation | §24, "projection, never backdoor," now backed by `DPB-INV-12` |
| Architecture driven by speculative AI-agent needs | Explicitly declined (§23) |
| Compatibility fragmentation | `DPB-INV-10` (§21), tier/version-range distinguished |

## 35. Conflict Analysis

| Compared against | Classification | Basis |
|---|---|---|
| `ARCH-014` (SDK) | **Compatible** | §12 preserves exclusive SDK ownership; zero conflicts found at Independent Architecture Review or Re-Review |
| `ARCH-007` (Persistent Actor/Supervision) | **Compatible, dependent** | `DPB-INV-06` cites its supervision concept for mental-model consistency only |
| `ARCH-008` (Effect Runtime) | **Compatible, dependent** | `DPB-INV-01`/`08` reference its capability-scoped-provider and audit-event patterns; `DPB-INV-08` now sourced to it exclusively (§18) |
| `ARCH-011`/`ARCH-012` (Durable Storage/DomainState) | **Compatible, dependent** | `DPB-INV-06` cites durable state for mental-model consistency only |
| `ADR-0020` | **Compatible, directly implementing** | This document is the "narrower replacement... under a freshly, independently verified identifier" `ADR-0020` §5/§7 anticipated |
| `ARCH-001`/`ARCH-002`/`ADR-0016` | **Overlapping but correctly subordinate — not relied upon directly** | Disclosed §5.1; the v0.1.0 citation defect (`IAR-015-F01`) corrected at §18, independently re-verified clean at Re-Review |
| `GOV-018` | **Compatible, constitutionally governed** | Every capability/audit/workload-agnosticism invariant traces directly to already-binding text |
| `ACT-005` | **Compatible, subordinate** | §4.H and §6 restated, not reinterpreted |

**No genuine conflict with any Approved architecture found**, independently confirmed at Authoring, Independent Architecture Review, and Architecture Re-Review alike.

## 36. Future Amendment Triggers

A future amendment is warranted if: a domain architecture repeatedly needs a rule this document does not provide and cannot safely define alone; `RES-008`'s weak evidence items are later strengthened by genuine SynapseOS-specific developer evidence; `ARCH-014` itself is amended in a way that changes the vocabulary this document consumes by reference; or a future domain architecture's own experience with `DPB-INV-12`'s "current, authoritative source of record" language reveals it needs sharper mechanical precision than this document deliberately declines to provide (§29).

## 37. Non-Modification Statement

This document creates no Runtime capability. It modifies no Runtime, Persistence, Recovery, Scheduling, Effect, Supervision, or SDK architecture — `ARCH-007`, `ARCH-008`, `ARCH-011`, `ARCH-012`, and `ARCH-014` remain entirely as they were before this document was drafted. It does not implement the CLI, Documentation Platform, Developer Portal, Testing Tooling, Packaging, or Control Centre. It does not specify Rust code, GUI framework, or any technology. It does not resolve `ARCH-013`'s permanently-unresolved F05 finding. It does not amend `ADR-0020`.

## 38. Governance Impact

This document occupies the same governance tier `ARCH-014` established for itself: architecture governing a Developer Platform component, subordinate to `GOV-018`'s constitutional layer and `ADR-0020`'s own disposition, superior to any future Engineering Work Order or domain architecture that implements it. No existing `ARCH`, `ADR`, `GOV`, or `STD` document is amended, superseded, or contradicted by this Filing.

## 39. Disposition

**Approved.** Architecture Authoring (v0.1.0) → Independent Architecture Review (`REVISION REQUIRED`, 2 Major, 5 Minor, 2 Observation) → Architecture Correction (v0.2.0, applying exactly `F01`, `F02`, `F03`, `F05`, `F06`, `F09`, `F10`) → Architecture Re-Review (`PASS`, zero remaining or correction-induced findings) → Founder Architecture Approval, recorded in full below.

**Founder Architecture Approval granted.** Denver Jacobs, Founder, 2026-08-10, recorded verbatim:

> "Founder Decision — ARCH-015 v0.2.0. ARCH-015 — Developer Platform Boundary Architecture v0.2.0 is hereby granted Founder Architecture Approval. It is accepted as the authoritative cross-cutting Developer Platform Boundary Architecture within its explicitly defined scope. The Architecture Re-Review PASS verdict is accepted in full: all seven Founder-authorized Architecture Correction findings are Resolved; zero Critical, Major, or Minor findings remain; zero new Observations were introduced; no correction-induced architectural drift or contradiction was found; no architecture leakage was found; no Runtime or SDK ownership boundary was violated; F07 and F08 remain Observations and were not elevated into normative requirements. DPB-INV-12 — State of Record — is accepted as part of the Approved architecture: Developer Platform Surfaces may maintain caches, projections, indexes, representations, or derived state where appropriate, but may not independently become the authoritative source of truth for state legitimately owned by the Runtime, SDK, or another subsystem; this approval does not select a storage, synchronization, database, caching, polling, push, eventing, or other implementation mechanism. The corrected CLI boundary is accepted: the CLI remains an optional Developer Platform Surface and must not become a hidden mandatory substrate through which other legitimate Developer Platform Surfaces are architecturally required to access SynapseOS functionality that is independently reachable through the SDK/platform; this does not authorize CLI architecture or implementation. The Control Centre boundary established by ARCH-015 is accepted: a future Control Centre must remain a projection of legitimate SynapseOS capabilities rather than becoming a second Runtime, an independent authority system, an authoritative source of Runtime-owned state, a capability bypass, an audit bypass, a mandatory application dependency, a mandatory CLI-dependent surface, or an independent source of compatibility truth; this approval does not authorize Control Centre Design Exploration, architecture, technology selection, GUI framework selection, or implementation. The corrected Documentation Platform dependency treatment is accepted: ARCH-015 establishes the required cross-cutting Developer Platform boundary foundation while correctly leaving domain-owned Documentation Platform matters — including the remaining DES-002 R21 per-concept disclosure-tier assignment and marking mechanism — to Documentation Platform architecture; Documentation Platform Architecture Authoring remains paused until ARCH-015 is successfully published and a separate Founder sequencing/authorization decision is made. The downgrade of former DPB-INV-11 to NBG-01 — Non-Binding Guidance — is accepted: the concern remains preserved and traceable but is not a binding cross-cutting architectural invariant; future domain architecture may adopt an equivalent domain-specific requirement only where independently justified. Packaging remains subject to all applicable general Developer Platform invariants but receives no packaging-specific architecture, format, technology, or implementation authorization through ARCH-015; SRP-001 remains unopened and unchanged. ARCH-015 remains deliberately narrow; this approval does not transform it into an umbrella architecture for all ACT-005 Developer Platform domains. This approval authorizes controlled publication as the next separate engagement. It does not authorize Runtime modification, SDK modification, CLI architecture or implementation, Documentation Platform Architecture Authoring or implementation, Control Centre Design Exploration or Architecture Authoring, GUI implementation or framework selection, Developer Portal work, Packaging implementation, Distributed Runtime work, Synapse Cloud, Enterprise Edition, AI Workforce Platform work, or any other ACT-005 domain work."

This Filing (v0.2.0) records that approval and constitutes this document's own Repository Filing.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Architecture Authoring Draft, transforming `DES-003` v0.1.0's Founder-accepted candidate requirements into eleven invariants and six decisions, per `GOV-013` §6.8. Independent Architecture Review: `REVISION REQUIRED` (0 Critical, 2 Major [`F01`,`F02`], 5 Minor [`F03`,`F05`,`F06`,`F09`,`F10`], 2 Observation [`F07`,`F08`]). Founder-accepted in full. |
| 0.2.0 | 2026-08-10 | Denver Jacobs (Founder) | Architecture Correction, applying exactly the seven accepted findings: removed defective `ARCH-002` §18 citation, sourcing `DPB-INV-08` to Approved `ARCH-008` alone (`F01`); added `DPB-INV-12` State of Record (§10.1) and a seventh Authority Projection question (§11), updating §24 Control Centre Boundary to cite it (`F02`); clarified §13 that the CLI must not become a mandatory substrate for other Developer Platform Surfaces, not only for applications (`F03`); clarified §21 that a stability tier is not proof of version compatibility (`F05`); clarified §26 that Packaging remains bound by the general invariants despite having no packaging-specific rule (`F06`); downgraded `DPB-INV-11` to non-binding guidance `NBG-01` (§22), preserving traceability (`F09`); softened §25's overstated R21 dependency-resolution claim to "partially resolved" (`F10`). `F07`/`F08` recorded as Observations, not converted into requirements. Architecture Re-Review: `PASS` — zero remaining or correction-induced findings of any severity. Records Founder Architecture Approval (§39, Founder Declaration quoted verbatim) and constitutes this document's own Repository Filing. `status` transitions from `Draft` to **`Approved`**. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author (v0.1.0) | Denver Jacobs (AI-assisted) | Drafted | 2026-08-10 |
| Independent Architecture Review (v0.1.0) | — | `REVISION REQUIRED` — 0 Critical, 2 Major, 5 Minor, 2 Observation | 2026-08-10 |
| Founder Disposition (v0.1.0 review) | Denver Jacobs, Founder | Accepted in full; Architecture Correction authorized, strictly bounded | 2026-08-10 |
| Author (v0.2.0, Correction) | Denver Jacobs (Founder) | Corrected — exactly seven findings applied | 2026-08-10 |
| Architecture Re-Review (v0.2.0) | — | **PASS** — zero remaining or correction-induced findings | 2026-08-10 |
| Approval Authority | Denver Jacobs, Founder (interim, per `GOV-003` §3.2 vacancy) | **Approved** (verbatim Founder Declaration recorded in §39) | 2026-08-10 |
