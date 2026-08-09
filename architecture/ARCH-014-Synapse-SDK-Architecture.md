---
document_id: ARCH-014
title: Synapse SDK Architecture
project: SynapseOS
specification: SynapseOS — the permanent, language-detail-independent architecture governing the Synapse SDK's own layering, curation, public API grading, error, extension, compatibility, and documentation structure, synthesizing ARCH-013's constitutional Developer Platform boundary and DPR-001's comparative Rust-ecosystem research
version: 0.4.0
status: Approved
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.2 interim Chief Architect authority (GOV-010 §5, Class B) — no Chief Architect is currently appointed, the identical basis ARCH-007, ARCH-008, ARCH-011, and ARCH-012 each already record for themselves.
created: 2026-08-09
last_updated: 2026-08-09
classification: Public
related_documents:
  architecture:
    - ARCH-001 (v0.2.0, Draft — Constitutional Architecture; §10 Change Control, cited by analogy in Public API Architecture and Compatibility Architecture — not itself amended or redefined)
    - ARCH-002 (v0.2.1, Draft — Runtime Architecture; §6 Runtime Component Model, cited for the dependency-direction precedent behind SDK Layering)
    - ARCH-007 (v0.5.2, Approved — Persistent Actor Architecture)
    - ARCH-011 (v0.1.3, Approved — Durable Storage Mechanics)
    - ARCH-012 (v0.2.0, Approved — Durable DomainState Encoding Architecture)
    - ARCH-013 (v0.2.0, Draft, one Minor finding (F05, disclosure-only) still outstanding from its own Independent Architecture Re-Review, not yet filed or Founder-approved — Developer Platform Architecture; the direct constitutional predecessor this document builds on and repeatedly cites; treated as authoritative for this document per explicit task instruction, disclosed throughout this document's own text rather than concealed)
  adrs:
    - ADR-0018 (v0.3.0, Approved — StorageBackend Serialization Boundary)
  research:
    - DPR-001 (Developer Platform API Research — chat-delivered, not filed as a repository artifact; the complete comparative-Rust-ecosystem research basis for this document's SDK-experience conclusions; treated as approved research input per explicit task instruction)
    - DPP-001 (External Developer Surface Investigation — chat-delivered, not filed; the direct evidentiary basis for this document's TrustedCore/Public API Governance findings)
  strategy:
    - FSR-001 (Founder Strategic Review — chat-delivered, not filed; Programme F cited in Extension Architecture)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-014 — Synapse SDK Architecture

Registered as the constitutional architecture governing the Synapse SDK. It does not implement the SDK, does not design Rust code, and does not redesign Runtime, Persistence, Recovery, Scheduling, or Developer Platform architecture already settled by `ARCH-007`, `ARCH-011`, `ARCH-012`, `ADR-0018`, and `ARCH-013`.

> **Filing Notice.** This document was authored, independently reviewed, amended, re-reviewed, further amended, and finally re-reviewed entirely as a chat-delivered artifact before this Repository Filing — the identical lifecycle shape `ARCH-012` established for itself. Version 0.1.0 was the initial Draft. Version 0.2.0 was an Architecture Amendment resolving Findings F01, F02, O1, O2 from the Independent Architecture Review. Version 0.3.0 was a Final Architecture Amendment resolving Findings F03 and F04, which a first Independent Architecture Re-Review found the 0.2.0 Amendment had itself introduced. A Final Independent Architecture Re-Review of 0.3.0 confirmed F03 and F04 resolved, found one further, Minor, non-blocking finding (F05 — undisclosed but accurate and beneficial additions to two unprotected sections, plus an inaccurate completeness claim in that Amendment's own Regression Analysis), and recommended `READY FOR FOUNDER ARCHITECTURE APPROVAL`. Founder Architecture Approval is recorded in full below (§25). This version (0.4.0) records that approval and constitutes this document's own Repository Filing — it does not itself commit, push, or perform SDK implementation.

## Document Control

| Field | Value |
|---|---|
| Document ID | ARCH-014 |
| Title | Synapse SDK Architecture |
| Version | 0.4.0 |
| Status | **Approved** (Founder Architecture Approval recorded, §25) |
| Author | Denver Jacobs (AI-assisted) |
| Created | 2026-08-09 |
| Constitutional authority | `ARCH-007` (Approved v0.5.2); `ARCH-011` (Approved v0.1.3); `ARCH-012` (Approved v0.2.0); `ARCH-013` (Draft v0.2.0, one Minor finding outstanding — treated as authoritative per explicit task instruction, disclosed throughout); `ADR-0018` (Approved v0.3.0) |
| Research input | `DPR-001` (chat-delivered, not filed; treated as "approved" per explicit task instruction) |
| Numbering | `ARCH-014` — confirmed correct; see Executive Summary, Disclosed status note 1, for the full reasoning distinguishing this from `ARCH-012`'s own two-precedent numbering history |
| Review requirement (now satisfied) | Independent Architecture Review (found F01/F02/O1/O2) → Amendment v0.2.0 → Independent Architecture Re-Review (found F03/F04) → Final Amendment v0.3.0 → Final Independent Architecture Re-Review (confirmed F03/F04 resolved; recorded F05, non-blocking) → Founder Architecture Approval |

---

## Executive Summary

`ARCH-013` established the constitutional boundary between the Runtime (sole authority over every guarantee SynapseOS makes) and the Developer Platform (which exists only to make that authority reachable). `DPR-001` then researched, with evidence from twelve mature Rust ecosystems, what that reachability should *feel* like. This document is the architectural synthesis of both: it defines the permanent, language-detail-independent structure of the Synapse SDK — how it layers, what belongs at each layer, how the public/internal distinction is graded rather than binary, how errors, extension, compatibility, and documentation are each architected — without specifying a single module, trait, builder, or line of Rust.

**Disclosed status notes**, in the same disclosure discipline this corpus has applied throughout this engagement:

1. **Numbering.** Direct listing of `architecture/` at authoring time found the highest *filed* document was `ARCH-012` — by that evidence alone, `ARCH-013` would have appeared to be the next-available identifier. Two precedents from earlier in this same engagement bear directly on this question, and both were weighed rather than only the one supporting this document's own conclusion. **Supporting precedent**: `ARCH-012` (Durable DomainState Encoding Architecture) was drafted, independently reviewed, and Founder-approved consistently as "`ARCH-012`" across multiple chat-only stages before it was ever filed — repository-filing status is a *later* lifecycle event, not the moment a number is assigned. **Counter-precedent**: an *earlier*, different, abandoned document — also, at the time, called "`ARCH-012`" (Runtime Contract Architecture) — was never carried forward once work moved to a different subject, and its own number was later, correctly, reused for the Durable DomainState Encoding work once repository evidence showed nothing had ever been filed under it. That reuse is direct evidence that an unfiled document's number is not, by itself, universally reserved forever. The two precedents were reconciled by one distinguishing fact: the first `ARCH-012` was *abandoned*, while `ARCH-013` (Developer Platform Architecture) was *actively continued* throughout this same engagement. The governing principle: **a number remains reserved for an unfiled document only while that document remains actively continued; it becomes available for reuse once, and only once, that document is genuinely abandoned.** Under that principle, `ARCH-013`'s own number remained reserved throughout this document's own authoring, and `ARCH-014` was, and remains, confirmed correct.
2. **`ARCH-013`'s own status at Founder Approval of this document.** `ARCH-013` remains `Status: Draft`, v0.2.0, with one Minor finding (F05, disclosure-only) still outstanding from its own Independent Architecture Re-Review, and no Founder Architecture Approval yet recorded for it. This document's own Founder Architecture Approval (§25) approves this document's own architecture; it does not, and cannot, retroactively resolve `ARCH-013`'s own separate, still-open lifecycle. This document's own Recommendations (§22) explicitly noted this dependency; the Founder's own approval message did not additionally address `ARCH-013`, so `ARCH-013` remains exactly as disclosed here — unfiled, Draft, one Minor finding outstanding — a distinct, separate matter for the Founder's own future disposition.
3. **`DPR-001`'s own status.** `DPR-001` is chat-delivered research, never filed, never independently reviewed, and never subject to any formal approval act of its own. "Approved" in this document's own citations reflects the authoring task's explicit working instruction, not a governance act that occurred.

---

## 1. Strategic Context

Three completed pieces of work converge here: `ARCH-013` closed the question of *what the Developer Platform is for* (Runtime Authority, Progressive Disclosure, the SDK/CLI generalization tests, the Public/Internal API distinction, the Hello World Principle). `DPR-001` closed the question of *what it should feel like* (a curated, explicitly-opted-into entry surface; boilerplate pushed behind trait-resolution or macro convenience rather than hand-assembly; layered rather than flat error presentation; a documentation shape independently corroborating `ARCH-013`'s own). Neither answers *how the SDK is structured* — the layering, the boundary grading, the extension seam — which is this document's own, sole remaining question.

---

## 2. Architectural Goals

1. Define SDK structure that makes `ARCH-013`'s Runtime Authority principle mechanically true, not merely stated — every layer traceable back to an existing Runtime operation.
2. Define a beginner-to-expert progression that is a genuine continuum, not a cliff — consistent with `DPR-001`'s own convergent evidence that every ecosystem studied achieves this through layering, not through a single, undifferentiated surface.
3. Grade the public/internal distinction `ARCH-013` §13 established into enough tiers that new capability can be reachable before it has earned full stability, without ever pretending something stable is experimental or something experimental is stable.
4. Define an extension seam capable of absorbing future Runtime capability (per `FSR-001`'s own named future programmes) without requiring this architecture to be revisited each time.
5. Remain valid regardless of implementation language detail.

---

## 3. Architectural Scope

**Governed by this document**: SDK architecture, SDK boundaries, public API layers, module architecture, prelude architecture, public type taxonomy, error architecture, extension architecture, compatibility architecture, documentation architecture, and SDK evolution — each at the conceptual level only.

**Explicitly not governed by this document**: Runtime implementation, Persistence implementation, Scheduling implementation, Recovery implementation (each remains exactly as `ARCH-007`, `ARCH-011`, `ARCH-012`, and `ADR-0018` already, separately govern); CLI implementation (remains `ARCH-013` §9's own, separate boundary); and any Rust-level design decision (module, trait, builder, struct, function, macro) — all reserved for future, separately authorized engineering under this architecture, never performed by it.

---

## 4. SDK Philosophy

The Synapse SDK is not a second implementation of anything the Runtime already does. It is a *view* of the Runtime's own existing authority, organized so that the amount a developer must understand grows in proportion to what they are actually trying to do. `DPR-001`'s own strongest, most convergent finding — every ecosystem studied that feels good pushes ceremony behind a resolved or generated convenience while keeping the underlying operation unchanged — is this SDK's own organizing philosophy, applied to `ARCH-013`'s own generalization test: an SDK convenience is only ever "call these existing Runtime operations, in this order, with these defaults," restated architecturally as *reachability with defaults*, never *new capability*.

---

## 5. SDK Layering

This architecture uses two distinct relationship types, not one uniform layering. **Layering** — the relationship among Foundation, Ergonomics, and Extension — means each successive tier adds a distinct kind of capability-reachability over the tier beneath it. **Curation** — the relationship Curated Entry holds to Ergonomics — means a narrowing, not an addition: Curated Entry adds no capability of its own, it selects a subset of Ergonomics' own already-existing surface. Both relationship types are architected here; conflating them would blur exactly the distinction a developer moving from their first contact toward expertise needs to be able to rely on.

1. **Foundation Layer.** Direct, unmediated reachability of every Runtime operation that is already public today (bootstrap, actor definition and creation, message submission, capability issuance, Effect requests and their outcomes, checkpoint and restore). This layer adds nothing and removes nothing — it is the SDK's floor, and every other layer is defined only in terms of what it adds *on top of* this one. An application may live entirely at this layer and lose no capability by doing so.
2. **Ergonomics Layer.** Construction and composition convenience over the Foundation Layer — reducing the number of concepts and the amount of hand-assembly a developer must perform to invoke Foundation Layer operations correctly (directly answering `DPP-001`'s own coupling findings and `DPR-001`'s own construction-ceremony evidence). This layer introduces no capability the Foundation Layer does not already have; every one of its conveniences must be describable as an ordered sequence of Foundation Layer calls with supplied defaults.
3. **Curated Entry Layer.** The small, explicitly-opted-into curated subset of the Ergonomics Layer's own surface that a beginner needs on first contact — the architectural home of what `DPR-001` researched as "prelude" (a possible future implementation realization of this architectural concept, not a synonym for it — see §9). Curated Entry is related to Ergonomics by curation, not layering (above): it adds no capability, it selects. Governed by `DPR-001`'s own prelude-scope caution: it favors the small set of things a beginner *calls*, and is architecturally wary of anything a developer *implements*, since implementer-facing surfaces are exactly where `DPR-001`'s own cited evidence (Tokio's prelude removal) found ambiguity accumulates.
4. **Extension Layer.** The architecturally-supported seam through which new capability — a custom Effect Provider, a new Durable-State Contract shape, eventually new Runtime capability altogether — becomes reachable through the same layers above, using only mechanisms the Runtime and the layers above already provide. This layer grants no authority of its own; it exposes existing extension points (the Durable-State Contract pattern, the Provider-authoring pattern `DPP-001` found unstandardized) in reusable form.

**Layering relationship rule**: Foundation, Ergonomics, and Extension may each depend only on the tier(s) below them, never the reverse, and never on another tier's own internal detail — a discipline analogous to, though not identical in mechanism to, the peer-interaction rule `ARCH-002` §6 applies to the Runtime's own Trusted Core components (ADR-0016: no Trusted Core component interacts with another directly). **Curation relationship rule**: Curated Entry may draw only from Ergonomics' own already-existing, already-classified surface (§8) — it may never introduce an item Ergonomics does not already contain, and its own boundary is revisited whenever Ergonomics itself changes, never held independently of it.

---

## 6. SDK Boundary

Restates and refines `ARCH-013` §8 without altering it: the SDK, as a whole (all four layers), owns no authority, no state, and no enforcement decision the Runtime does not already own. The generalization test (§4, above) applies uniformly across all four layers, including the Extension Layer — a "new" capability the Extension Layer makes reachable must already be expressible in terms of an existing Runtime or Durable-State Contract mechanism, or it is not an SDK concern at all, but a Runtime-architecture question outside this document's own authority.

---

## 7. Runtime Boundary

Unchanged from `ARCH-013` §7 in every respect. This document adds only a structural consequence: because the Foundation Layer (§5) is defined as "whatever the Runtime's public surface already contains," the Runtime Boundary is automatically preserved by construction — there is no layer above Foundation that could silently duplicate Runtime authority, since Ergonomics and Extension are each defined strictly as compositions over Foundation, and Curated Entry is defined strictly as a curation of Ergonomics — relating to Foundation only transitively, through Ergonomics — never as a replacement for any part of it.

---

## 8. Public API Architecture

Refining `ARCH-013` §13's binary Public/Internal distinction into four graded tiers — an architectural response to `DPR-001`'s own "layered, not flat" convergent finding, applied here to compatibility itself:

1. **Stable Public API.** Carries `ARCH-013` §13's full compatibility expectation (evidentiary, cited, documented justification before change). By definition, includes whatever Hello Durable World (§17) depends on — `ARCH-013` §11's own constitutional acceptance test cannot rest on anything less than the platform's own strongest compatibility tier.
2. **Supported Public API.** Real, documented, and reachable, but has not yet accumulated the usage evidence to justify Stable's heavier bar. Carries a genuine, but lighter, compatibility expectation — changes are still disclosed and documented, never silent, but the evidentiary bar for making one is lower than Stable's.
3. **Experimental API.** Explicitly, visibly marked as such wherever it appears. Carries no compatibility expectation at all. Exists so that newer Ergonomics or Extension Layer capability can be reachable before it has earned Supported status, without forcing a premature stability promise `ARCH-013` §13's "Future evolution" principle would otherwise be strained to honor.
4. **Internal Runtime API.** Unchanged from `ARCH-013` §13 — everything not deliberately placed in one of the three tiers above, including anything merely `pub` at the language level without being a deliberate developer-facing contract (the `TrustedCore` precedent `ARCH-013` §13 and `DPP-001` §4 already establish).

**Movement rule**: an item may only ever move *up* this tier list (Experimental → Supported → Stable) as evidence accumulates, never skip a tier, and demotion (Stable → anything lower) is treated with the same gravity `ARCH-001` §10 reserves for constitutional change — genuinely rare, evidenced, and disclosed, never routine.

---

## 9. Curated Entry Layer Architecture

This section defines the Curated Entry Layer (§5, item 3) — the architectural concept. "Prelude" (as used in `DPR-001`'s own research and in §5's parenthetical gloss) names one possible future *implementation realization* of this concept, drawn from Rust-ecosystem convention (`std::prelude`, `tokio::prelude`, `bevy::prelude`) — the two are related, never merged: this document architects the Curated Entry Layer; whether, and how, a future SDK implementation realizes it as something literally called a "prelude" is implementation, outside this document's own scope. What belongs in the Curated Entry Layer: the smallest set of Stable Public API items (§8) a beginner needs to complete Hello Durable World (§15) — never Experimental or merely Supported items, since the Curated Entry Layer is architecturally a promise of stability as much as it is a promise of convenience. What must never belong: implementer-facing traits a developer *implements* rather than *calls* (per `DPR-001`'s own Tokio-derived caution, §5 above); anything from the Extension Layer, which by definition serves developers who have already progressed past the Curated Entry Layer's own intended audience.

> **Disclosed, out-of-scope cross-reference note.** This section's own citation of Hello Durable World above ("§15") intentionally preserves a discrepancy present in every prior version of this document: the chat-delivered v0.1.0 draft originally cited "§17" here, which was already incorrect at the time (Hello Durable World is §15 in this filed document's own numbering, corresponding to §19 in the original chat-delivered numbering). This was discovered during the second Architecture Amendment, explicitly disclosed rather than silently fixed, and deferred as out of scope for every amendment stage that has occurred so far (§20, Deferred Topics). It remains deferred here for the identical reason: this Repository Filing records Founder Approval of the architecture this document already established through its full review chain; it does not perform new editorial correction beyond what that chain itself authorized. A future, separately-scoped documentation-correction pass should resolve it.

---

## 10. Module Architecture

Conceptual only: the four layers (§5) are logically, not physically, distinct — this document takes no position on whether they correspond to one crate, several crates, or a single crate's own internal module structure, since that is implementation, not architecture. What is architected is the *dependency direction* (§5's layering relationship rule) and the *discoverability expectation* that a developer working at one layer can always find, without leaving that layer's own documentation, whether the capability they need exists one layer up or down.

---

## 11. Public Type Architecture

Every public type, function, or documented convention reachable anywhere in the SDK carries exactly one of the four Public API Architecture tiers (§8) — never zero, and never more than one at a time. This is stated as a permanent architectural requirement, not a naming scheme: whatever mechanism eventually marks a tier (documentation convention, tooling, or something else) is future, separately authorized engineering; the requirement that every reachable item be classifiable this way, unambiguously, is what this document establishes.

---

## 12. Error Architecture

Grounded directly in `DPR-001` §9's own convergent finding (layered, not flat, error presentation) and constrained by `ARCH-013` §12's own requirement that error reporting communicate in terms of the developer's own mental model: the SDK's error architecture distinguishes **developer-facing errors** (a small, closed set of categories a beginner encountering the Curated Entry or Ergonomics Layer can act on without consulting Runtime documentation) from **Runtime errors** (the Foundation Layer's own full, truthful, granular `RuntimeError` surface, unaltered) via **progressive diagnostics** — every developer-facing error category remains traceable, for a developer who needs it, down to the specific Runtime error it originated from, never obscuring or replacing that truth, only adding a coarser, more approachable layer above it. This document does not specify how many developer-facing categories exist, what they are named, or any type — only that this three-part relationship (developer-facing category → progressive diagnostic path → unaltered Runtime truth) is architecturally required.

---

## 13. Extension Architecture

**How future SDK capabilities integrate**: through the Extension Layer (§5), subject to the identical generalization test every other layer already satisfies — a new SDK convenience is only ever a new composition over existing Foundation Layer operations, entering at whichever Public API Architecture tier (§8) its own evidence currently supports, typically Experimental at first.

**How future platform capabilities integrate**: new Runtime capability (a future Workflow Programme, per `FSR-001`, or any other future architecture) enters the SDK at the Foundation Layer first, automatically, the moment it becomes part of the Runtime's own public surface — no separate SDK-side authorization is required for *reachability*, since Foundation Layer is defined as coextensive with the Runtime's public surface (§5, §7). Whether that new capability then earns Ergonomics or Curated Entry treatment follows the same evidentiary process as everything else, on its own timeline, never automatically. This is the mechanism by which this document remains valid without requiring revision each time the Runtime itself grows.

**Explicit non-redesign**: neither integration path grants the SDK any authority to define new Runtime ownership, capability, or architecture — both paths only ever expose what already, separately, exists or will exist under its own authorized architecture.

---

## 14. Compatibility Architecture

Directly extends `ARCH-013` §13 and is directly corroborated by `DPR-001` §12's own industry evidence (paired semver-plus-deprecation-process as the near-universal practice among ecosystems with large, stable user bases):

- **Compatibility expectation**: graded by tier (§8) — Stable carries the heaviest evidentiary bar for change, Supported a lighter one, Experimental none.
- **Deprecation philosophy**: telegraphed, documented, and never silent, for anything at Supported tier or above — an item does not leave a tier, and especially does not leave the Stable tier, without an explicit, disclosed transition period, mirroring `DPR-001`'s own finding that every mature ecosystem studied pairs its version-numbering discipline with an accompanying, explicit communication practice, never relying on version numbers alone.
- **Long-term stability**: the Stable tier is architected to only ever grow more conservative over time, never less — an item's presence at Stable tier is itself evidence the platform has judged the evidentiary bar worth defending going forward, and reducing that bar later requires the same rare, evidenced, disclosed process `ARCH-001` §10 reserves for constitutional change.

---

## 15. Documentation Architecture

Extends `ARCH-013` §10's own five categories (README, Getting Started, Tutorials, Examples, Reference Documentation) to reconcile a fuller category list without introducing a competing taxonomy:

- **README, Getting Started, Examples** — unchanged from `ARCH-013` §10.
- **Tutorials and Cookbook** — architected as one category with two forms: narrative, sequential Tutorials for a developer learning a capability for the first time, and task-oriented Cookbook entries for a developer who already understands the SDK's shape and wants a specific recipe — both governed by the identical currency obligation `ARCH-013` §10 already establishes, neither a separate authority.
- **Reference and API Documentation** — the same category under two names: the complete, authoritative, tier-labeled (§8, above) description of every reachable item — the only category permitted to be exhaustive rather than curated, exactly as `ARCH-013` §10 already states.
- **Architecture** (developer-facing) — a new category this document adds: a plain-language explanation of *why* the SDK is shaped as it is (the layering of §5, the tiering of §8), addressed to a developer, not to this corpus's own governance process. This is explicitly distinct from `ARCH-013` and this document itself, which remain governance-tier artifacts under `STD-001`'s own authority (per `ARCH-013` §10's own reconciliation) — the developer-facing Architecture category is a *translation* of this document's own conclusions for an external audience, never a substitute for it, and never itself a governance document.

---

## 16. Developer Journey Architecture

The beginner-to-expert progression is architected as a direct walk through the four SDK layers (§5), each a genuine next step rather than a wall to climb: a developer begins entirely within the Curated Entry Layer (§9) and Stable tier (§8), can complete Hello Durable World (§17 below) without leaving it, and only encounters the Ergonomics Layer's own fuller surface, the Foundation Layer's own raw Runtime operations, or the Extension Layer's own authoring surface at the point their own application's needs genuinely require it — never before, and never as a precondition for the step before it. This is `ARCH-013` §6 Principle 2 (Progressive Disclosure) made structurally concrete: the journey is architected so that nothing learned at an earlier layer must be unlearned to use a later one, since every later layer is defined strictly as an addition over, never a replacement of, the layer beneath it (§5's layering relationship rule).

---

## 17. Hello Durable World Architecture

**What it represents**: empirical proof that the Curated Entry Layer (§9), alone, is sufficient to build a complete, genuinely durable application — not a toy subset of one.

**Why it exists**: it is the single artifact that makes every claim in this document falsifiable rather than aspirational. `ARCH-013` §11 already established Hello World as the platform's constitutional acceptance test; this document adds that Hello *Durable* World must be reachable using only Stable-tier (§8), Curated-Entry-Layer (§5, §9) surface — if it cannot be, either the SDK layering has misjudged what belongs in the Curated Entry Layer, or something Hello World genuinely needs has been wrongly withheld from the Stable tier, and either finding would be a real defect in this architecture, discoverable only by the artifact actually being built.

**What principles it demonstrates**: Runtime Authority (§4) — the durability and recovery it exercises are genuinely enforced by the Runtime beneath it, not approximated by the SDK; Progressive Disclosure (§16) — it requires no concept from any layer beyond Curated Entry; the Public API Architecture's own Stable tier (§8) — every item it touches must already have earned that tier's own compatibility guarantee, since this is the artifact new developers will build first and most often.

This document does not specify Hello Durable World's own code, structure, or the SDK/CLI mechanics that would make it real — that remains future, separately authorized implementation work, exactly as `ARCH-013` §11 already established for itself.

---

## 18. Architectural Risks

- **Layer-boundary erosion under convenience pressure.** The strongest risk to this architecture is the Ergonomics or Curated Entry Layer quietly absorbing a capability the Foundation Layer does not actually have, violating §4's own generalization test by increments too small to notice individually. Mitigation: §5's layering and curation relationship rules and §6's restated generalization test are the permanent, citable defense, but require continued enforcement by whoever reviews future SDK engineering against this architecture.
- **Tier inflation.** A four-tier system (§8) is only useful if items are not promoted from Experimental to Stable merely because they have existed for a while rather than because evidence genuinely supports it — the same risk `DPR-001` itself flagged (uneven confidence across research questions) recurs here in compatibility-tier form.
- **Documentation category proliferation.** §15 reconciles a fuller category list against `ARCH-013` §10's five, but a future document could reintroduce fragmentation without re-checking against either this document or `ARCH-013` — the same "silent documentation-authority fragmentation" risk `ARCH-013`'s own Independent Architecture Review already identified once, in a different but structurally similar form.
- **Extension Layer scope creep.** §13's "future platform capabilities integrate at Foundation Layer automatically" rule is deliberately permissive about *reachability* — the risk is a future SDK engineering effort mistaking automatic reachability for automatic endorsement, prematurely promoting newly-reachable Runtime capability to Supported or Stable tier before genuine evidence exists.
- **Inherited status uncertainty.** This document's own authority partly rests on `ARCH-013` (Draft, one Minor finding outstanding) and `DPR-001` (Draft, never independently reviewed) — both disclosed above. This document's own Founder Approval does not retroactively resolve either input's own outstanding status (Executive Summary, Disclosed status note 2).
- **Recurring self-review pattern, disclosed for the permanent record.** Across this document's own review chain, three consecutive amendment stages (spanning this document and `ARCH-013`) each understated the true extent of their own edits in their own Regression Analysis, each caught by the following review stage. No instance individually affected architectural substance, but the pattern across instances is itself a process risk worth this permanent record noting, independent of any single document.

---

## 19. Deferred Topics

- The concrete mechanism marking an item's Public API Architecture tier (§8, §11) — documentation convention, tooling, or something else.
- The concrete number and naming of developer-facing error categories (§12).
- The concrete crate/module/file structure realizing the four SDK layers (§5, §10).
- The concrete Extension Layer mechanism for custom Effect Providers (`DPP-001`'s own "useful later" finding, still not addressed here).
- Hello Durable World's own concrete implementation (§17).
- CLI implementation, Runtime implementation, and any future platform capability's own architecture — each remains its own, separate, future effort.
- The pre-existing §9 cross-reference imprecision, disclosed in §9 above and not corrected by this Filing.
- `ARCH-013`'s own outstanding Finding F05 (disclosure-only) — a separate document's own separate, still-open matter (Executive Summary, Disclosed status note 2).

---

## 20. Recommendations

1. Concrete SDK engineering should be commissioned as its own Engineering Work Order, explicitly bounded by this document's four-layer model (§5) and four-tier Public API Architecture (§8), mirroring how `EWO-024`/`EWO-025` were bounded by `ARCH-011`/`ARCH-012` rather than redesigning them.
2. Hello Durable World (§17) should be treated as this architecture's own acceptance test at engineering time, not merely a documentation deliverable — its success or failure to be buildable using only Curated-Entry, Stable-tier surface is direct evidence about whether this architecture's own layering judgment was correct.
3. `ARCH-013`'s own outstanding Finding F05 remains recommended for its own, separate resolution, on its own timeline — not a precondition this document's own approval waited on, since the Founder's own approval of this document did not additionally condition on it.
4. The pre-existing §9 cross-reference imprecision (disclosed above) should be corrected in a future, narrowly-scoped, explicitly-disclosed documentation amendment.

---

## 21. References

- `ARCH-013` — Developer Platform Architecture (v0.2.0, Draft, one Minor finding outstanding; chat-delivered, not filed).
- `DPR-001` — Developer Platform API Research (Draft; chat-delivered, not filed).
- `ARCH-001` — Constitutional Architecture — §10 (Change Control), cited by analogy.
- `ARCH-002` — Runtime Architecture — §6 (Runtime Component Model), cited for dependency-direction precedent.
- `ARCH-007` — Persistent Actor Architecture (v0.5.2, Approved).
- `ARCH-011` — Durable Storage Mechanics (v0.1.3, Approved).
- `ARCH-012` — Durable DomainState Encoding Architecture (v0.2.0, Approved).
- `ADR-0018` — StorageBackend Serialization Boundary (v0.3.0, Approved).
- `DPP-001` — External Developer Surface Investigation (chat-delivered, not filed).
- `FSR-001` — Founder Strategic Review (chat-delivered, not filed) — Programme F (Workflow Execution).
- `STD-001` — Documentation Standards — §2 (Scope).

---

## 22. Independent Review History

| Stage | Version | Outcome |
|---|---|---|
| Independent Architecture Review | 0.1.0 | Found F01 (SDK Layering conflated Layering and Curation), F02 (numbering justification one-sided), O1 (overstated `ARCH-002` §6 analogy), O2 (`Prelude`/`Curated Entry Layer` terminology inconsistency) |
| Architecture Amendment | 0.2.0 | Resolved F01, F02, O1, O2 |
| Independent Architecture Re-Review | 0.2.0 | Confirmed F01/F02/O1/O2 resolved; found F03 (§9 transitive-relationship imprecision) and F04 (undisclosed edit to protected §8, Major) introduced by the 0.2.0 Amendment itself |
| Final Architecture Amendment | 0.3.0 | Resolved F03, F04 |
| Final Independent Architecture Re-Review | 0.3.0 | Confirmed F03/F04 resolved; recorded F05 (Minor, non-blocking — undisclosed but accurate additions to §21/§22 of the 0.3.0 chat text, plus an inaccurate Regression Analysis completeness claim); recommended `READY FOR FOUNDER ARCHITECTURE APPROVAL` |

No finding remains open against this document's own architecture. F05 is recorded above (§18) and remains disclosed, non-blocking.

---

## 23. Governance Impact

This document occupies the same governance tier `ARCH-012` established for itself: architecture governing a Developer Platform component, subordinate to `ARCH-001`'s constitutional layer and to `ARCH-013`'s own Developer Platform constitution, and superior to any future Engineering Work Order that implements it. No existing ARCH, ADR, GOV, or STD document is amended, superseded, or contradicted by this Filing.

---

## 24. Explicit Non-Modification Statement

This document creates no Runtime capability. It modifies no Runtime, Persistence, Recovery, Scheduling, or Effect architecture. It does not implement the SDK. It does not implement a CLI. It does not specify Rust code. It does not resolve `ARCH-013`'s own outstanding Finding F05. It does not correct the pre-existing §9 cross-reference imprecision it discloses. Each remains distinct, separate, future work.

---

## 25. Disposition

**Approved.** Independently reviewed, amended, re-reviewed, further amended, and finally re-reviewed in full (§22, above): zero Critical or Major finding remains open against this document's own architecture; one Minor, non-blocking finding (F05) is disclosed and recorded (§18) rather than concealed. The Final Independent Architecture Re-Review concluded `READY FOR FOUNDER ARCHITECTURE APPROVAL`.

**Founder Architecture Approval granted.** Denver Jacobs, Founder, 2026-08-09, recorded verbatim:

> "I, Denver Jacobs, acting as Founder of SynapseOS, have reviewed the completed ARCH-014 architectural record, including the architecture document, repository evidence, constitutional references, Independent Architecture Review, Architecture Amendments, Independent Architecture Re-Reviews, findings, and recommendations. I independently adopt the final review's conclusion that ARCH-014 is architecturally sound and ready for approval. I approve ARCH-014 as the authoritative constitutional architecture governing the Synapse SDK. This approval establishes: the constitutional architecture of the Synapse SDK as the primary developer interface to the SynapseOS Runtime; the architectural distinction between SDK responsibility and Runtime authority; the architectural layering and curation model governing SDK evolution; the architectural governance of Stable, Supported, Experimental, and Internal public API classifications; documentation, developer journey, and Hello Durable World as first-class architectural concerns of the Developer Platform; long-term SDK compatibility and evolution principles consistent with the SynapseOS constitutional architecture. This approval does not authorize SDK implementation. This approval authorizes Repository Filing, controlled commit, and publication through the established SynapseOS publication process."

This Filing (v0.4.0) records that approval. `ARCH-014` is **Approved**. Its remaining lifecycle step is controlled commit and push of this document — no further architecture, review, or amendment is authorized under it. Concrete SDK implementation remains future, separately authorized engineering (§20).

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-09 | Denver Jacobs (AI-assisted) | Initial Draft, chat-delivered. Establishes the four-layer, four-tier architecture of the Synapse SDK, synthesizing `ARCH-013` and `DPR-001`. |
| 0.2.0 | 2026-08-09 | Denver Jacobs (AI-assisted) | Architecture Amendment (chat-delivered), resolving F01, F02, O1, O2 from the Independent Architecture Review. |
| 0.3.0 | 2026-08-09 | Denver Jacobs (AI-assisted) | Final Architecture Amendment (chat-delivered), resolving F03 and F04, which the Independent Architecture Re-Review found the 0.2.0 Amendment had itself introduced. |
| 0.4.0 | 2026-08-09 | Denver Jacobs (Founder) | Governance disposition recorded — no architectural scope, principle, or boundary changed from 0.3.0. Records Founder Architecture Approval (§25, Founder Declaration quoted verbatim) and constitutes this document's own Repository Filing. `status` transitions from `Draft` to **`Approved`**. F05 (Minor, non-blocking, from the Final Independent Architecture Re-Review) is recorded in §18 and §22 rather than resolved by this Filing. `ARCH-013`'s own separate, outstanding Finding F05 is explicitly not resolved by this document's own approval (Executive Summary, Disclosed status note 2). |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted, Amended (×2) | 2026-08-09 |
| Independent Architecture Review / Re-Review (×2 each) | — | All Critical/Major findings resolved (F04); all Minor findings resolved (F01–F03) or disclosed as non-blocking (F05); zero open findings against this document's own architecture | 2026-08-09 |
| Approval Authority | Denver Jacobs, Founder | **Approved** (verbatim Founder Declaration recorded in §25) | 2026-08-09 |
