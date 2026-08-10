---
document_id: DES-002
title: "Documentation Platform and SDK Documentation: Design Exploration"
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
  and DES-001 already occupy it: an evidence-to-decision synthesis
  document that precedes, and directly informs, a later binding
  artifact (a future Documentation Platform + SDK Documentation
  architecture) without itself being architecture, governance, or an
  engineering authorization. This placement is a disclosed, narrow
  convenience, not a documentation-hierarchy redesign; formal
  registration of a "DES" family in STD-001, if ever wanted, is a
  separate, future, independently-authorized task, not performed
  here or implied by this placement.
reviewed_by: >
  DAR-002 — Design Approval Review of DES-002 v0.1.0 (conversational
  record; not a filed repository document); verdict —
  REVISION REQUIRED (three findings: DAR-002-F01, Major — complete
  normative requirements set not actually delivered, only a
  representative subset; DAR-002-F02, Minor — R08 misclassified as a
  requirement rather than a Constraint; DAR-002-F03, Minor —
  Getting-Started dependency framing overstated the CLI/scaffolding
  dependency). Design Correction (v0.1.0 -> v0.2.0) applied exactly
  these three findings. First Narrow Design Re-Review (conversational
  record) verdict — FAIL (one new finding, NDR-002-F01, Minor: the
  original Section 26 Traceability Matrix retained a stale R08 row
  and omitted the thirteen requirements the F01 correction added).
  A further, narrow Design Correction (v0.2.0 -> v0.2.1) applied
  exactly this one finding, reconciling the Traceability Matrix and
  correcting one mechanically-dependent, previously-misstated
  requirement count (26 normative + 1 Deferred = 27 tracked
  identifiers, not 24 + 1 as originally reported). Second Narrow
  Design Re-Review (conversational record) verdict — PASS. Denver
  Jacobs, Founder, accepted DES-002 v0.2.1 as the final Design
  Exploration / requirements baseline on 2026-08-10 and separately
  authorized this Repository Filing; this acceptance is not
  Architecture Approval and does not authorize Architecture
  Authoring.
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.4-§6.7, the Design Exploration / Design Approval Review / Design Correction / Narrow Design Re-Review stages this document's own lifecycle follows exactly)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution; §2 Platform Identity, §5.2 capability security — binding constraints throughout)
    - ACT-005 (v0.1.0, Approved — Developer Platform Era; §4.B/C Documentation Platform / SDK Documentation domains, §9 Governance Proportionality)
  roadmap:
    - ROAD-001 (v0.1.0, Approved — SynapseOS Platform Strategy and Roadmap; §2 Developer Experience pillar)
  research:
    - RES-008 (v0.2.0, Founder-accepted — Developer Platform Landscape and Developer Workflow Research; the primary evidence baseline for every requirement in this document)
  consolidation:
    - DES-001 (v0.2.0, Draft — Persistent Actor Design Exploration; the demonstrated Design Exploration / Design Correction / Narrow Design Re-Review precedent this document follows structurally throughout)
  source_artifacts:
    - "DX-001 — Hello Durable World reference application (sdk/examples/hello_durable_world.rs, synapse-runtime; uncommitted as of this document's authorship, disclosed accurately, used as direct evidence for R02's own CLI-independence)"
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# DES-002 — Documentation Platform and SDK Documentation: Design Exploration

> **Founder Acceptance Notice (2026-08-10).** Denver Jacobs, Founder, has reviewed and accepted this document, at this version (v0.2.1), as the final Design Exploration / requirements baseline for the SynapseOS Documentation Platform + SDK Documentation workstream, following the complete review/correction lifecycle recorded in full in the `reviewed_by` field above. Consistent with `DES-001`'s own established precedent, this document's own tracked `status:` remains **Draft** — Design Exploration output in this corpus carries no independent approval authority of its own and uses no `Approved` status or formal Approval Status table; Founder disposition is recorded here and in `reviewed_by` instead. **This acceptance is the requirements/design baseline only. It is not Architecture Approval and does not authorize Architecture Authoring, technology selection, or implementation of any kind.**

## 1. Purpose

`GOV-018` establishes SynapseOS's identity; `ROAD-001` names Developer Platform as the most immediately unblocked strategic era; `ACT-005` authorizes investigation, requirements, research, architecture, and review for the Documentation Platform and SDK Documentation domains; `RES-008` is the accepted evidence baseline. This document answers `GOV-013` §6.4's own question — *what should be built?* — for this workstream, without answering *how*.

## 2. Scope and Non-Goals

**In scope:** what the Documentation Platform and SDK Documentation must accomplish for developers, treated as one requirements workstream. **Explicitly out of scope, verified not violated anywhere below:** static-site generator or any specific tool selection; hosting/domain selection; search-provider selection; UI/CSS design; rustdoc integration mechanics; SDK/Runtime API changes; tutorial/reference-application authoring; CLI documentation commands; an AI documentation assistant; any documentation-specific Runtime feature.

## 3. Developer Journey Requirements

Mapped against `RES-008` §4's own 15-stage journey: documentation-owned stages are 1-3, 9-11, 16-17; jointly-owned are 6-7 (create/understand first project). **Stages 6-7 are achievable today via `DX-001`'s own demonstrated manual Cargo workflow, independent of the CLI/scaffolding workstream (`ACT-005` priority 3).** CLI/scaffolding, once it exists, may improve this journey's ergonomics but is not a prerequisite for R02's own Must-tier satisfaction. Not documentation-owned, recorded as External Dependencies (§9): stages 4, 8, 12-15.

## 4. Information Model, Progressive Disclosure, and SDK Coverage

Ten information categories evaluated (Orientation, Getting Started, Concepts, Guides, Tutorials-interface, API Reference, Examples, Troubleshooting, Architecture Explanation, Compatibility/Versioning, Contribution). Disclosure tiers: mandatory-before-first-success (actor identity, one capability grant, one message send — matching `DX-001`); immediately-after (durable state/checkpoint/restore); intermediate (multi-actor messaging, capability denial, Effect Providers, `ConstraintSet`/secure patterns); advanced (supervision, Extension-layer authoring); contributor/internal (Foundation-layer direct usage, Runtime internals). **Binding constraint:** progressive disclosure must never omit or soften a security/correctness concept to make onboarding appear simpler — capability grants remain visible at the mandatory tier precisely because `GOV-018` §5.2 makes them non-optional. All four stable SDK layers (Foundation, Ergonomics, Prelude, Extension) require documentation coverage; Foundation is explicitly audience-differentiated to the contributor/internal tier.

## 5. Source-of-Truth, Versioning, Discoverability, and Quality

API-reference content must be sourced solely from SDK source, never hand-duplicated; conceptual documentation must cite its `GOV`/`ARCH` source directly and be staleness-checkable by version comparison. A developer must always be able to determine which documentation version applies to their SDK/Runtime version. Concept discovery, API lookup, task lookup, and troubleshooting lookup must each be achievable within a bounded, baseline-measured number of navigation steps. Quality attributes (correctness, completeness, consistency, freshness, testability) are satisfied structurally by every requirement's own class/priority/acceptance method (§7), not stated as a separate, unverifiable "documentation should be excellent" requirement.

## 6. Examples, Troubleshooting, and Security Documentation

Documentation-owned examples are small, single-concept, and link to — never duplicate — the separate Reference Application catalogue (Strategy C, `RES-008` §14), which this document does not author. Troubleshooting distinguishes documentation problems (information exists but is unexplained — in scope) from tooling problems (no diagnostic surface exists — External Dependency, §9). Capability grants, `ConstraintSet`/scoping, denial, and secure-vs-dangerous patterns must be documented accurately without ever redefining capability semantics — approved architecture remains sole authority.

## 7. Complete Normative Requirements Set

**26 normative requirements, plus `R15` explicitly Deferred — 27 tracked identifiers total.**

| ID | Statement | Class | Priority | Persona(s) | Acceptance Method |
|---|---|---|---|---|---|
| R01 | Orientation content must state what problem SynapseOS solves and what it deliberately does not solve | Content | Must | All | Review: both halves present |
| R02 | A Getting Started path must reach durable-actor success using currently available mechanisms (manual Cargo workflow), independent of CLI | Functional | Must | App Developer | Review: path reachable without a CLI step |
| R03 | Every stable-tier public SDK item (Foundation, Ergonomics, Prelude, Extension) must have documented purpose, usage, parameters, return/error behavior | Content | Must | App Developer | Audit: 100% stable-tier coverage |
| R04 | Capability grants and the capability-denial pattern must be documented at the mandatory-before-first-success tier | Security | Must | App Developer | Review: present at mandatory tier |
| R05 | Orientation content must state the workload-agnostic architecture / AI-native flagship-workload distinction explicitly | Content | Must | All | Review: both halves present |
| R06 | Every published example must be validated to compile against the current stable SDK | Validation | Must | App Developer | Automated/procedural check |
| R07 | A developer must be able to determine which documentation version applies to their SDK/Runtime version | Compatibility | Must | App Developer, Enterprise Evaluator | Review: version marker present |
| R09 | A catalogue of known `RuntimeError` variants and their meaning must be documented | Content | Should | App Developer | Audit: catalogue exists and covers `DX-001`-exercised variants |
| R10 | Content must use clear headings and complete, unambiguous code blocks | Quality | Should | App Developer | Review: style conformance |
| R11 | Baseline measurement of time-to-first-documented-success must be established before any numeric onboarding target is set | Adoption/Usability | Should | App Developer | Procedural: baseline study conducted |
| R12 | Content must be keyboard-navigable, semantically structured, sufficiently contrasted, screen-reader compatible, and responsive | Accessibility | Should | All | Review: each attribute present |
| R13 | Cross-linking between a concept, its API reference, and its example must exist for every documented concept | Content | Should | App Developer | Review: link-presence audit |
| R14 | Offline/local documentation access could be supported | Functional | Could | App Developer | N/A pending evidence of need |
| R16 | Dedicated conceptual documentation must exist for durable actors, capabilities, effects, supervision, and state/recovery | Content | Must | App Developer | Audit: each concept has dedicated content |
| R17 | Task-oriented guides should exist for the Must-tier friction points identified in the developer-journey evidence | Content | Should | App Developer | Audit: guide exists per identified friction point |
| R18 | A tutorial interface (linking convention between Concepts, Examples, and the future Reference Application catalogue) must be defined, without authoring the catalogue itself | Content | Must | App Developer | Review: interface/linking convention documented |
| R19 | A bounded architecture explanation must exist, sufficient to reason correctly without requiring the full constitutional corpus | Content | Must | App Developer, Enterprise Evaluator | Review: present, does not restate full `GOV`/`ARCH` text |
| R20 | Minimal, proportional contribution-path documentation should exist | Content | Should | Contributor | Review: guide exists at proportional depth |
| R21 | Every documented concept (not only capability grants) must be assigned a disclosure tier | Content | Must | App Developer | Review: every concept in R16 has an assigned tier |
| R22 | Foundation-layer content should be explicitly marked contributor/internal-tier, distinct from Prelude/Ergonomics | Content | Should | App Developer, Contributor | Review: explicit tier marking present |
| R23 | API-reference content must be sourced solely from SDK source, never hand-duplicated | Governance | Must | App Developer | Review: no manually-duplicated API description found |
| R24 | Conceptual/architectural documentation must cite its `GOV`/`ARCH` source directly and be staleness-checkable by version comparison | Governance | Must | App Developer | Review: citation + version marker present |
| R25 | `ConstraintSet`/scoping and secure-vs-dangerous capability patterns must be documented at the intermediate tier | Security | Must | App Developer | Review: present, does not redefine capability semantics |
| R26 | Concept discovery, API lookup, task lookup, and troubleshooting lookup should each be achievable within a bounded, baseline-measured number of navigation steps | Adoption/Usability | Should | App Developer | Procedural: baseline navigation-step study |
| R27 | A documentation-completeness check must be part of future release readiness | Governance | Must | App Developer, Enterprise Evaluator | Review: check exists in a future release process |
| R28 | Broken links, stale API references, uncompilable examples, missing stable-tier documentation, orphaned pages, and terminology inconsistency must each be automatable/procedurally detectable | Validation | Must | App Developer | Procedural: validation mechanism reports each category |
| R15 | The specific accessibility compliance level is Deferred | Accessibility | Deferred | All | N/A — pending external research and Founder decision |

## 8. Constraints

- **C01**: documentation affecting architecture meaning, security semantics, public API meaning, or compatibility must receive review proportional to the underlying change (`ACT-005` §9, existing, binding governance).
- **C02**: no AI-specific Runtime or SDK documentation construct permitted (`GOV-018` §2, binding).
- **C03**: documentation must not redefine or narrow capability semantics; approved architecture remains sole authority (`GOV-018` §5.2).
- **C04**: no SDK/Runtime API change may be introduced via documentation work.
- **C05**: no new governance tier is invented for documentation review; `ACT-005` §9's existing proportionality is reused, not replaced.

## 9. External Dependencies

Diagnostics/Observability workstream (tooling-problem troubleshooting); Reference Application catalogue (Strategy C, R18's own interface point); capability-grant ergonomic work, if ever pursued; CLI/scaffolding workstream (an ergonomic improvement, never a prerequisite for R02); the release-governance process itself (R27's dependency); the future validation mechanism (R06/R28's own architecture-question dependency).

## 10. Architecture Questions (deferred, not answered here)

Content/storage model; rendering/build system; SDK/rustdoc integration mechanism; documentation-versioning mechanism; search implementation; hosting/deployment; the automated validation pipeline; the source-synchronization mechanism; analytics/privacy considerations if any telemetry is ever considered; the concrete machine-consumable representation format.

## 11. Deferred Items

`R15` — specific accessibility compliance level, pending external standards research and a separate Founder decision. `R14` — offline/local access, Could-priority, pending evidence of actual developer need.

## 12. Traceability Matrix

| Item | GOV-018 | ROAD-001 | ACT-005 | RES-008 | SDK v1.0.0 |
|---|---|---|---|---|---|
| R01, R02 | §3 Vision | §2 Dev Experience pillar | §4.B/C | §4 journey, §5 friction | `hello_durable_world.rs` |
| R03, R22 | — | — | §4.C | §6 SDK audit | `prelude.rs`/`foundation.rs`/etc. |
| R04, R25 | §5.2 | — | — | §13 differentiation | `bootstrap_grant` pattern |
| R05 | §2 (binding) | — | — | §13 | — |
| R06, R28 | — | — | — | §6, §14 | doc comments |
| R07, R11, R26 | — | — | §13 Evidence Programme | §9/§12/§15 baseline requirements | `ARCH-014` §19 tier-marking |
| R09 | — | — | — | §6 (`UnknownTarget`) | `RuntimeError` variants |
| R10 | — | — | — | §12/§17 | — |
| R12 | — | — | — | (general baseline practice — no SynapseOS-specific evidence) | — |
| R13 | — | — | — | §10 comparator finding | — |
| R14 | — | — | — | (no evidence — Could, pending) | — |
| R15 | — | — | — | (Deferred — pending external research and Founder decision) | — |
| R16, R21 | §4 | — | — | §6, §8 | — |
| R17 | — | — | — | §5 friction inventory | — |
| R18 | — | §3-§4 (Strategy C) | — | §14 Strategy C | — |
| R19 | — | — | — | §6 (two-repo-split friction) | — |
| R20 | — | — | §9 Proportionality | — | — |
| R23, R24 | — | — | — | (`STD-001` §34, cited directly, §5 above) | SDK source (doc comments) |
| R27 | — | — | — | (self-justified, §9 above — no external citation fabricated) | — |
| C01 | — | — | §9 Proportionality | — | — |
| C02 | §2 (binding) | — | — | — | — |
| C03 | §5.2 | — | — | — | — |
| C04 | — | — | — | — | (SDK stability, §4 above) |
| C05 | — | — | §9 Proportionality | — | — |

*(The former `R08` — a proportional-governance-review requirement — was reclassified during Design Correction into `C01`, a Constraint rather than a normative requirement. It is retained in this matrix as `C01` for traceability completeness, not as `R08`.)*

## 13. Acceptance Framework

This requirements set is adequately specified when: every stable-tier SDK item has a traced documentation requirement (R03, §12); the Getting Started journey requirement is traceable to `DX-001`'s own demonstrated steps and does not depend on CLI (R02); every requirement has an explicit class, priority, and acceptance method (§7); every architecture-adjacent question is deferred explicitly (§10); every external dependency is named rather than absorbed (§9). The eventual Documentation Platform's own completion is not declared by this document — that belongs to a future Architecture Freeze Review-equivalent disposition and, ultimately, Founder-tier determination.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Design Exploration Draft. Following `GOV-013` §6.4, answering "what should be built?" for the Documentation Platform + SDK Documentation workstream, based on `RES-008` v0.2.0's accepted evidence. Original normative content presented as 15 representative requirements. |
| 0.2.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Design Correction applying exactly `DAR-002-F01`, `DAR-002-F02`, `DAR-002-F03`: expanded the representative requirement presentation into a complete, traceable set (adding R16-R28); relocated the former `R08` into Constraint `C01`; corrected the Getting-Started/CLI dependency framing to state explicit CLI-independence. No other content changed. |
| 0.2.1 | 2026-08-10 | Denver Jacobs (AI-assisted) | Narrow Design Correction applying exactly `NDR-002-F01`: reconciled the Section 26 (now §12) Traceability Matrix — removed the stale `R08` row, added rows for `R16`-`R28`, resolved a pre-existing `R05`/§16-section label collision incidental to the same pass, and corrected one mechanically-dependent requirement-count statement (26 normative + 1 Deferred = 27 tracked identifiers, not 24 + 1 as originally reported). No substantive requirement added, removed, strengthened, weakened, reprioritized, or reclassified. |
| 0.2.1 (this filing) | 2026-08-10 | Denver Jacobs (Founder) | Repository Filing. Records genuine Founder acceptance of this document, at this version, as the final Design Exploration / requirements baseline for the Documentation Platform + SDK Documentation workstream (`reviewed_by`, above). No substantive content altered by this filing — identifier, frontmatter, this Founder Acceptance Notice, and this Revision History entry are the only additions. Does not authorize Architecture Authoring. |
