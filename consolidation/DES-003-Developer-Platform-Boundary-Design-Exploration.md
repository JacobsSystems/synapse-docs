---
document_id: DES-003
title: "Developer Platform Boundary: Design Exploration"
version: 0.1.0
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
  and DES-001/DES-002 already occupy it: an evidence-to-decision
  synthesis document that precedes, and directly informs, a later
  binding artifact (a future replacement Developer Platform Boundary
  Architecture) without itself being architecture, governance, or an
  engineering authorization. This placement is a disclosed, narrow
  convenience, not a documentation-hierarchy redesign; formal
  registration of a "DES" family in STD-001, if ever wanted, is a
  separate, future, independently-authorized task, not performed
  here or implied by this placement.
reviewed_by: >
  Independent Design Approval Review of DES-003 v0.1.0 (conversational
  record; not a filed repository document), conducted per GOV-013
  §6.5 — verdict: PASS, confirmed sufficient to proceed, no Design
  Correction required. Findings: 0 Blocking, 0 Major, 2 Minor (M1 —
  a Domain Ownership Matrix completeness gap concerning Reference
  Applications and Community/Commercial Adoption Foundations,
  self-corrected inline during drafting via an explicit disclosure
  note, §20; M2 — DPB-17's evidentiary basis, "general engineering
  practice," is weaker than every other DPB requirement's sourcing,
  accepted as a disclosed, non-blocking limitation, not corrected
  further), 1 Observation (O1 — developer identity/account ownership
  is a genuine, disclosed gap with no current owner anywhere in the
  repository, carried to Architecture Questions §34, not
  force-resolved). Denver Jacobs, Founder, accepted DES-003 v0.1.0 as
  the current cross-cutting Developer Platform requirements and
  design baseline on 2026-08-10, accepted the PASS verdict and
  findings exactly as reported, and separately authorized this
  Repository Filing; this acceptance is not Architecture Approval
  and does not authorize Architecture Authoring.
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.4-§6.5, the Design Exploration / Design Approval Review stages this document's own lifecycle follows exactly)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution; §2, §4, §5 principle 2 — binding constraints throughout)
    - ACT-005 (v0.1.0, Approved — Developer Platform Era; §4 eleven programme domains, §4.H Testing Tooling boundary, §6 Control Centre boundary)
  roadmap:
    - ROAD-001 (v0.1.0, Approved — SynapseOS Platform Strategy and Roadmap; §2 Developer Experience pillar)
  research:
    - RES-008 (v0.2.0, Founder-accepted — Developer Platform Landscape and Developer Workflow Research; the evidence baseline for every hedged claim in this document)
  architecture:
    - ARCH-014 (v0.8.0, Approved — Synapse SDK Architecture; the SDK-side authority this document never duplicates, §7, §29)
  adrs:
    - ADR-0020 (v0.1.0, Approved — Disposition of the Unrecoverable ARCH-013 Draft; the source of the orphaned cross-cutting principles this document explores, §3, §6, §8-§9)
  consolidation:
    - DES-001 (v0.2.0, Draft — Persistent Actor Design Exploration; the demonstrated Design Exploration / Design Approval Review precedent this document follows structurally)
    - DES-002 (v0.2.1, Draft, Founder-accepted — Documentation Platform and SDK Documentation: Design Exploration; the direct dependency-map partner, §28)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# DES-003 — Developer Platform Boundary: Design Exploration

> **Founder Acceptance Notice (2026-08-10).** Denver Jacobs, Founder, has reviewed and accepted this document, at this version (v0.1.0), as the current cross-cutting Developer Platform Boundary requirements and design baseline, following the Independent Design Approval Review recorded in full in the `reviewed_by` field above. Consistent with `DES-001`/`DES-002`'s own established precedent, this document's own tracked `status:` remains **Draft** — Design Exploration output in this corpus carries no independent approval authority of its own and uses no `Approved` status or formal Approval Status table; Founder disposition is recorded here and in `reviewed_by` instead. **This acceptance is the requirements/design baseline only. It is not Architecture Approval and does not authorize Architecture Authoring, technology selection, Control Centre work, or implementation of any kind.**

## 1. Purpose

`ADR-0020` identified that certain cross-cutting Developer Platform principles historically carried by the unrecoverable `ARCH-013` draft (CLI boundary, general Progressive Disclosure, general SDK/CLI generalization test) presently lack a complete Approved architectural home. This document answers `GOV-013` §6.4's question — *what should be built?* — for a **narrow cross-cutting Developer Platform boundary layer**, not a full umbrella architecture. It is new design work, constrained by surviving Approved architecture (`GOV-018`, `ARCH-014`) and current evidence (`RES-008`), never a reconstruction of `ARCH-013`.

## 2. Scope and Non-Goals

**In scope:** the boundary between Runtime, SDK, cross-cutting Developer Platform, and domain-specific Developer Platform components; candidate requirements for the cross-cutting layer only.

**Explicitly out of scope** (§27 gives the complete list; verified unviolated during Design Approval Review, §36 of the originating engagement report): Runtime redesign; SDK layer redesign; stable SDK v1.0.0 API redesign; Documentation Platform technology/framework/hosting/search selection; CLI implementation or framework selection; packaging implementation; Developer Portal, Reference Application, or Control Centre implementation; Distributed Runtime; Synapse Cloud; Enterprise Edition; AI Workforce Platform; commercial application design.

## 3. Historical ARCH-013 Boundary

`ARCH-013` — Developer Platform Architecture v0.2.0 Draft — is confirmed unrecoverable (`ADR-0020`). Nothing below reconstructs it, fabricates its missing text, presents new design as recovered `ARCH-013` content, reuses its identifier, or assigns it independent authority. Every candidate principle below is justified directly against currently Approved architecture (`GOV-018`, `ARCH-014`) and current accepted evidence (`RES-008`), not against any claim about what `ARCH-013` used to say beyond what `ADR-0020` §4 already, separately records as orphaned.

## 4. The Core Developer Platform Boundary

**Boundary statement:** *The Developer Platform is the layer that makes legitimate Runtime/SDK operations reachable, learnable, and consistent across every developer-facing surface, without inventing execution authority, duplicating SDK architecture, or making any single surface's tooling a precondition for building a correct application.*

Four zones, tested against `GOV-013`'s own Fundamental Design Rule (defined once vs. defined per-domain):

- **Runtime** — execution authority and mechanisms (`ARCH-007`, `ARCH-008`, `ARCH-011`, `ARCH-012`). Not touched by this document.
- **SDK** — the stable programmatic interface (`ARCH-014`, SDK v1.0.0's four layers). Not touched by this document.
- **Cross-Cutting Developer Platform** — the subject of this document: rules that must hold identically across CLI, Documentation, Tutorials, Templates, Testing Tooling, and beyond, because defining them independently per-domain would produce exactly the failure modes `GOV-013`'s own test names (inconsistent mental models, conflicting authority, duplicated policy, accidental Runtime semantics, inconsistent security posture).
- **Domain-Specific** — everything that fails that test: technology choices, presentation, command syntax, page layout.

## 5. A Structural Finding: One Principle, Several Applications

Self-disclosed during drafting, not concealed: four of the boundary questions this exploration was asked to investigate (Runtime Authority, SDK Boundary, Capability-Security, Auditability) resolve to **the same underlying rule** — *Developer Platform tooling may expose, compose, simplify, validate, explain, or orchestrate legitimate Runtime/SDK operations, but must never become a second Runtime.* Treating these as four independently-justified principles would itself be the "over-centralizing"/"premature abstraction" risk named in §33. They are presented below as one structural principle (`DPB-01`) with three domain-specific corollaries (`DPB-04` CLI, `DPB-09`–`DPB-11` capability-security, `DPB-12` auditability), each still independently sourced, not merely restated for volume.

## 6. Runtime Authority Boundary

Candidate rule (`DPB-01`, below): Developer Platform surfaces must not independently invent execution authority the Runtime does not possess. Tested against CLI (must translate every command to real SDK/Runtime operations, no side channel), templates (must use only approved SDK patterns), documentation examples (must compile against the real stable SDK — already `DES-002` R06), testing tools (mocking must not silently disable capability checks in a way that misrepresents production behavior — test doubles must be structurally distinguishable from real grants), Developer Portal (actions must map to real operations, no shadow state), future Control Centre (out of scope per `ACT-005` §6, but the principle pre-applies if that boundary is ever crossed). Not a Runtime redesign — the Runtime's own authority is unchanged; this only binds what may be built *on top of* it.

## 7. SDK Boundary

`ARCH-014` already, exclusively owns SDK architecture. This document does not duplicate it. Developer Platform surfaces may assume SDK v1.0.0's four layers, four-tier Public API Architecture, and generalization test (`ARCH-014` §4) are stable and authoritative. The dividing line between legitimate "convenience" and an "unofficial parallel SDK": **the verbosity test** — can the same effect be achieved, only more verbosely, using documented Stable-tier SDK calls alone? If yes, it is tooling convenience. If no — if it requires semantics no existing SDK call expresses — it is an SDK proposal requiring `ARCH-014`'s own amendment process, never silently added as tooling magic (`DPB-03`).

## 8. CLI Boundary

`ADR-0020` §4 identified the general CLI boundary as orphaned. Candidate principle: *the CLI is a developer-facing control and workflow surface over legitimate platform operations, not an independent authority system.* What it may own: scaffolding conventions, local workflow sequencing, diagnostic presentation, ergonomics. What it must delegate: all execution authority, all capability semantics, all durable-state semantics (`DPB-04`) — it must call into SDK-exposed operations, never reimplement them. What it must never own: independent execution authority, a parallel state store, a parallel capability model. CLI commands may encode *orchestration* policy (step ordering) but never *security/capability* policy. `DES-002` R02 already establishes that Getting Started must remain reachable through the manual Cargo workflow, independent of any future CLI; this generalizes directly to `DPB-05`: **CLI availability must never become a precondition for core SynapseOS application correctness** — not because CLI is discouraged, but because architectural dependence on it, not inconvenience, is the actual risk.

## 9. Generalization Principle

Formalizing `ADR-0020` §4's orphaned "SDK/CLI generalization test" as `DPB-02`: *a Developer Platform convenience is legitimate only if fully explainable as a simpler or safer composition of existing, legitimately-owned platform operations, introducing no new execution semantics; removing the convenience must leave the underlying application model valid.* Applied via a six-question test: (1) what underlying platform operation exists; (2) who owns it; (3) what authority is required; (4) does the convenience merely compose/expose it; (5) does it introduce new semantics; (6) would removing it leave the underlying application model valid. This is `DPB-01`'s own enforcement mechanism, not an independently-motivated rule — its Developer Choice value is indirect (§30): it exists to keep `DPB-01` auditable at review time, not because a developer directly benefits from its existence.

## 10. Progressive Disclosure

`ADR-0020` §4 identified the general form as orphaned; `ARCH-014` §16 carries only the SDK-specific instance. Not copied — generalized independently. Two structural rules, both already present in narrower form in existing Approved/accepted material and generalized here:

- **No omission** (`DPB-06`): no domain's own minimum-success path may omit or soften a security/correctness concept to appear simpler — directly generalizing `DES-002` §4's binding rule (capability grants stay visible at the mandatory tier because `GOV-018` §5.2 makes them non-optional) beyond documentation to every onboarding-relevant domain (CLI, Tutorials, Templates).
- **No unlearning** (`DPB-07`): nothing learned at an earlier stage may need to be unlearned later — directly generalizing `ARCH-014` §16's SDK-specific journey rule.

What remains UX, not architecture: page ordering, visual design, specific tutorial narrative content. What must be invariant: the two rules above, and that a definable minimum-success path exists per onboarding-relevant domain. A violation example: a future "quick start" that hides the capability-grant step, or a CLI scaffold pattern the developer must discard once they need the next SDK layer.

## 11. Developer Mental Model

An originating-task-suggested example chain (Actor → capability → operation/effect → durable state → supervision → observable result) was deliberately **not adopted automatically**. Independently deriving from Approved sources: `GOV-018` §4's four non-negotiable properties (identity, explicit capability, governed messaging, observable/auditable action) plus the constitutional mechanisms that already carry them (`ARCH-007` supervision, `ARCH-011`/`ARCH-012` durable state, `ARCH-008` effects/audit) are exactly the concepts every current Approved artifact treats as constitutional. That this independently-derived set substantially overlaps the suggested example is disclosed explicitly as **convergence from a shared constitutional source, not adoption of the example** — and notably, the example's specific *ordering* is not itself evidenced by anything Approved, so no ordering requirement is derived (`DPB-08` binds terminology/relationship consistency only, never presentation sequence).

## 12. Capability-Security Boundary

`GOV-018` §5 principle 2 ("explicit authority before implicit trust... every capability is minted, attenuated, or revoked explicitly, never assumed") is already binding constitutional text. Corollaries for Developer Platform surfaces, none redesigning capability architecture itself:

- **No silent grant** (`DPB-09`): tooling must never create a path that grants capability without an explicit, visible, attributable step.
- **Least-authority default** (`DPB-10`): official templates must default to the minimum capability set their own demonstrated purpose requires.
- **Capability visibility** (`DPB-11`): any surface executing a capability-requiring operation must make that requirement discoverable from that surface.

The "must not silently grant/hide/bypass" rules are structural and testable — architecture. The exact visual/textual mechanism for displaying a requirement is UX, not specified here.

## 13. Auditability Boundary

`GOV-018` §4 item 4 ("every action is observable, and every decision is auditable") is unconditional, not limited to Runtime-internal actions. `DPB-12`: any Developer Platform action causing a platform-visible change (a CLI-driven capability change, a future deployment/package action, a Developer Portal or Control Centre configuration change) must route through the same audited Runtime/SDK operations already producing the audit trail — never a parallel, tooling-level log. This is `DPB-01`'s auditability-specific corollary (§5), not an independently invented audit subsystem.

## 14. Error Model

`ARCH-014` §12 already carries the SDK-specific error architecture (developer-facing categories over the full `RuntimeError` surface, via progressive diagnostics). `DPB-13` generalizes only the **vocabulary-consistency** requirement — every developer-facing failure presentation, in any domain, must remain traceable to that same underlying vocabulary; no domain may invent a parallel, disconnected error taxonomy. It does not redefine what the categories are, does not specify types or codes (Architecture Question, §34).

## 15. Manual Path / Tool Independence

`DES-002` R02 (Getting Started independent of CLI) generalizes to `DPB-05`/`DPB-14`: **core SynapseOS application correctness must not depend on optional developer convenience tooling.** Test, per domain (CLI, templates, code generators, Developer Portal, Control Centre, packaging helpers): is what the tool produces also achievable, less conveniently, through documented Stable-tier SDK usage alone? If a genuine correctness-relevant capability exists with *no* manual path, that is a structural dependency, and must be disclosed as a Design Risk (§33), not silently accepted. `DPB-14` requires every domain to disclose this per major feature. The issue is architectural dependence, not convenience parity — a manual path need not be pleasant, only exist.

## 16. Automation and Non-Human Consumers

`RES-008` §16 Minor-1 (non-human documentation-traffic finding) is Medium confidence with no traceable primary source — too weak to elevate into a Must-priority architectural requirement, and doing so would also risk `GOV-018` §2's binding workload-agnostic rule by letting a specific consumer category drive design. `DPB-17` is deliberately **not** sourced from that finding: it rests only on ordinary automation/CI-compatibility grounds (a mechanical operation should not structurally require an interactive human, independent of who or what is asking) — Should-priority, weakest-evidenced requirement in this set, disclosed as such rather than concealed (§36, Finding M2, of the originating review). No requirement anywhere optimizes specifically for AI-agent or IDE-tooling consumption — that would be speculation this document declines to make.

## 17. Machine-Readable Surfaces

`DES-002` §10 already leaves the concrete machine-consumable representation format as an open Architecture Question, scoped to Documentation Platform architecture specifically. **Not absorbed here.** If a genuine cross-cutting need later emerges (e.g., shared compatibility-version metadata), it is already covered by `DPB-15` (§18, below), not a separate machine-readability principle.

## 18. Version Compatibility

`ARCH-014` §14 already governs SDK API compatibility tiers. `DPB-15`: any Developer Platform surface presenting or depending on version-specific SDK/Runtime behavior must be able to state the version range it is valid for, using `ARCH-014`'s own existing vocabulary — not a new compatibility scheme. Purely a "what must be true"; no metadata format, file, or mechanism is chosen (Architecture Question, §34).

## 19. Stable vs. Experimental Developer Surfaces

Investigated, not copied: `ARCH-014`'s four SDK tiers grade *API surface*, not *tooling features*. A CLI command can wrap only Stable-tier SDK calls yet still itself be an experimental, actively-changing CLI feature — a genuinely distinct axis. `DPB-16` (new, no `ARCH-014` equivalent): every domain surface (CLI command, template, testing-tool feature) must be able to disclose its own stability status, independent of, and not automatically inherited from, the SDK tier of what it wraps. No specific labels or mechanism chosen here.

## 20. Domain Ownership Matrix

| Concern | Runtime | SDK | Cross-Cutting Developer Platform | Domain-Specific |
|---|---|---|---|---|
| Execution semantics | ✅ Owner | — | — | — |
| Capabilities | ✅ Owner | Exposed via SDK | `DPB-09`–`11` (visibility/least-authority rules only) | — |
| Durable state | ✅ Owner (`ARCH-011`/`012`) | Exposed via SDK | — | — |
| Effects | ✅ Owner (`ARCH-008`) | Exposed via SDK | — | — |
| Supervision | ✅ Owner (`ARCH-007`) | Exposed via SDK | — | — |
| SDK ergonomics | — | ✅ Owner (`ARCH-014`) | — | — |
| CLI workflow | — | — | `DPB-01`/`02`/`04`/`05` (boundary only) | ✅ Owner (commands, syntax) |
| Onboarding | — | — | `DPB-06`/`07`/`08` (rules) | ✅ Owner (presentation, per domain) |
| Documentation presentation | — | — | `DPB-06`/`07`/`13`/`15` (rules consumed) | ✅ Owner (Documentation Platform) |
| Compatibility | — | ✅ Owner (vocabulary, `ARCH-014` §14) | `DPB-15` (contract requirement) | Owner (mechanism, per domain) |
| Errors | — | ✅ Owner (vocabulary, `ARCH-014` §12) | `DPB-13` (consistency requirement) | Owner (presentation, per domain) |
| Auditability | ✅ Owner (mechanism) | — | `DPB-12` (never-bypass rule) | — |
| Templates | — | — | `DPB-02`/`09`/`10` (constraints) | ✅ Owner |
| Packaging | — | — | Unexplored — `ACT-005` §12 defers to its own future competitive evaluation | ✅ Owner (eventual) |
| Testing | — | — | `DPB-01` (never weaken Runtime semantics, restating `ACT-005` §4.H directly) | ✅ Owner |
| Search/discovery | — | — | — | ✅ Owner (Documentation Platform/Developer Portal) |
| Developer identity/account | — | — | **Genuinely unowned — flagged, §34** | Unclear — Developer Portal, unconfirmed |
| Future Control Centre | Out of scope (`ACT-005` §6) | — | `DPB-01` pre-applies if ever built | — |

**Disclosed completeness note (self-caught during drafting, Design Approval Review Finding M1):** Reference Applications (`ACT-005` §4.F) and Community & Ecosystem / Commercial Adoption Foundations (`ACT-005` §4.J/K) are not given their own matrix rows. Investigated directly: no cross-cutting boundary question specific to these three domains was found that does not already reduce to `DPB-01`/`02` (Generalization), `DPB-06`/`07` (Progressive Disclosure), or `DPB-14` (Tool Independence) once they eventually produce tooling or content — the same rules apply to them as to any domain, not a gap, a disclosed conclusion. No new principle is proposed for them by this document.

## 21. Inherited Constraints

| Constraint | Source | Consequence | Modifiable by this exploration? |
|---|---|---|---|
| Workload-agnostic architecture; no AI-specific Runtime/SDK/tooling construct | `GOV-018` §2 (binding) | No `DPB` requirement may justify itself by AI-workload positioning | **No** |
| Explicit authority before implicit trust | `GOV-018` §5 principle 2 | Source of `DPB-09`/`10`/`11` | **No** |
| Every action observable, every decision auditable | `GOV-018` §4 item 4 | Source of `DPB-12` | **No** |
| Testing convenience must never weaken Runtime semantics | `ACT-005` §4.H | Restated directly as `C-DPB-04`, not reinterpreted | **No** |
| No Control Centre implementation authorized | `ACT-005` §6 | Bounds Domain Ownership Matrix's Control Centre row to principle-only | **No** |
| SDK v1.0.0 four-layer, four-tier architecture | `ARCH-014` §5, §8 | Source of `DPB-03`/`07`/`15`/`19` boundary | **No** |
| `ARCH-013`-derived principles now rest on `ARCH-014`'s own Approved status only | `ADR-0020` §2 item 6 | No requirement here may cite `ARCH-013` as independent authority | **No** |
| CLI-independent Getting Started | `DES-002` R02 | Direct source of `DPB-05`/`14` | **No** (DES-002 itself unmodified) |
| Capability grants visible at mandatory tier, never softened | `DES-002` §4 | Direct source of `DPB-06` | **No** |
| Zero SynapseOS external-user evidence exists | `RES-008` §15 | Bounds every Developer Choice Test claim to hypothesis (§32) | **No** |
| Documentation machine-readable format left open | `DES-002` §10 | Not absorbed here (§17) | **No** |

## 22. New Design Proposals

Each entry: problem → rule → rationale → affected domains → evidence → alternatives considered → risk → Architecture Authoring implication.

- **`DPB-01`/`04`/`12` — Non-Second-Runtime family.** *Problem:* nothing currently states, cross-domain, that Developer Platform tooling must not accrete independent authority. *Rule:* §5, §6, §8, §13. *Alternative considered:* leave each domain to discover this independently — rejected, exactly the inconsistent-security-posture failure mode `GOV-013`'s own test warns against. *Risk:* over-centralization if treated as four separate rules rather than one — mitigated by explicit disclosure (§5). *Architecture Authoring:* required eventually, to become binding rather than candidate.
- **`DPB-02` — Generalization Test.** Enforcement mechanism for the above; *alternative considered:* a purely case-by-case review with no named test — rejected as unauditable.
- **`DPB-03` — SDK Non-Duplication (verbosity test).** *Problem:* tooling convenience could silently accrete into an unofficial parallel SDK. *Alternative:* require every convenience to route through a formal SDK-change proposal regardless of triviality — rejected as disproportionate; the verbosity test is the proportional middle path.
- **`DPB-05`/`14` — CLI Non-Mandatory / Tool Independence Disclosure.** *Problem:* generalizes a single documented precedent (`DES-002` R02) into a durable cross-domain rule before it silently erodes. *Risk if omitted:* CLI becomes a de facto dependency by accumulation, never by a single visible decision.
- **`DPB-06`/`07` — Progressive Disclosure (no omission / no unlearning).** Directly generalizes existing binding text (`DES-002` §4, `ARCH-014` §16); minimal independent invention.
- **`DPB-08` — Conceptual Consistency.** *Alternative considered:* mandate a single canonical named model diagram — rejected, since presentation should not be frozen; terminology/relationship consistency chosen instead as the minimal binding form.
- **`DPB-09`–`11` — Capability-Security corollaries.** Direct corollaries of already-binding `GOV-018` text; genuinely low-risk, high-confidence proposals.
- **`DPB-13` — Error Vocabulary Consistency.** Generalizes `ARCH-014` §12 without redefining it.
- **`DPB-15` — Compatibility Contract.** Generalizes `ARCH-014` §14 without redefining it.
- **`DPB-16` — Domain Stability Disclosure.** Genuinely new; no existing source. *Alternative considered:* reuse `ARCH-014`'s SDK tier labels directly for tooling — rejected, since API stability and tooling-feature maturity are genuinely distinct axes without evidence they should share one label set.
- **`DPB-17` — Non-Interactive Path.** Weakest-evidenced; *alternative considered:* omit entirely given weak sourcing — retained at Should-priority only, explicitly decoupled from `RES-008`'s own weak finding, disclosed as the weakest item in the set.

## 23. Complete Requirements Set

| ID | Requirement | Class | Priority | Source/Rationale | Affected Domains | Acceptance Method |
|---|---|---|---|---|---|---|
| DPB-01 | Developer Platform surfaces must not invent execution authority the Runtime does not possess; every operation must be explainable as composition of existing Runtime/SDK operations | Architectural | Must | `ADR-0020` §5; `GOV-018` §4/§5 | CLI, Templates, Testing Tooling, Developer Portal, future Control Centre | Review: cited composed operation(s) per surface |
| DPB-02 | A convenience is legitimate only if fully explainable as composition, introducing no new semantics; removal must leave the application model valid | Architectural/Acceptance Test | Must | `ADR-0020` §5 (generalization test) | CLI, Templates, Testing Tooling | Review: six-question test applied |
| DPB-03 | Any convenience requiring semantics not expressible via existing Stable/Supported-tier SDK calls is an SDK proposal, not a Developer Platform decision | Architectural | Must | `ARCH-014` §6/§8; `ADR-0020` §2 item 8 | CLI, Templates | Review: verbosity test |
| DPB-04 | CLI must delegate all execution authority, capability, and durable-state semantics; must not reimplement or shadow any of them | Architectural | Must | `ADR-0020` §4 (orphaned CLI boundary) | CLI | Review: no CLI-internal capability/state store |
| DPB-05 | CLI availability must not become a precondition for core application correctness | Functional | Must | `DES-002` R02, generalized | CLI, Templates, code generators | Review: manual path exists per correctness-relevant capability |
| DPB-06 | No domain's minimum-success path may omit or soften a security/correctness concept | Security | Must | `DES-002` §4, generalized; `GOV-018` §5.2 | CLI, Docs, Tutorials, Templates | Review: mandatory-tier content audit |
| DPB-07 | Nothing learned at an earlier stage may need to be unlearned later | Content | Must | `ARCH-014` §16, generalized | All onboarding-relevant domains | Review: journey audit |
| DPB-08 | Developer-facing references to a constitutional concept must use terminology/relationships consistent with `GOV-018` §4 and its defining architecture | Content | Should | `GOV-018` §4 (`[INF]` synthesis) | All | Review: terminology audit |
| DPB-09 | No path may grant capability without an explicit, visible, attributable step | Security | Must | `GOV-018` §5 principle 2 | CLI, Templates, Testing Tooling | Review: grant-path audit |
| DPB-10 | Official templates must default to minimum required capability | Security | Must | `GOV-018` principle 2; `ACT-005` §4.E | Templates | Review: per-template capability audit |
| DPB-11 | Any capability-requiring operation's requirement must be discoverable from the surface executing it | Security | Must | `GOV-018` §4 | CLI, Docs, Templates | Review: discoverability check |
| DPB-12 | Platform-visible-change actions must route through existing audited operations, never a parallel log | Architectural | Must | `GOV-018` §4 item 4 | CLI, future Developer Portal/Control Centre | Review: audit-path trace |
| DPB-13 | Every failure presentation must remain traceable to `ARCH-014` §12's error vocabulary; no parallel taxonomy | Content/Architectural | Must | `ARCH-014` §12, generalized | CLI, Docs examples, Testing Tooling | Review: error-mapping audit |
| DPB-14 | Every domain must disclose, per major feature, whether a manual/tool-independent path exists | Governance | Must | `DES-002` R02, generalized | All | Review: disclosure present |
| DPB-15 | Version-dependent surfaces must state their valid SDK/Runtime version range using `ARCH-014`'s existing vocabulary | Compatibility | Must | `ARCH-014` §14, generalized | CLI, Templates, Docs, Testing Tooling | Review: version-range statement present |
| DPB-16 | Every domain surface must disclose its own stability status, independent of the SDK tier it wraps | Content | Should | New — no existing source | CLI, Templates, Testing Tooling | Review: stability marker present |
| DPB-17 | Mechanical operations should not structurally require interactive human presence | Functional | Should | General engineering/CI practice — explicitly not `RES-008`-sourced | CLI, Testing Tooling | Review: non-interactive equivalent exists |

## 24. Constraints

- **C-DPB-01**: No Developer Platform surface may become a second Runtime or duplicate `ARCH-014`'s SDK architecture (umbrella of `DPB-01`/`03`/`04`/`12`).
- **C-DPB-02**: No AI-specific Developer Platform construct is permitted (`GOV-018` §2, restating `DES-002` C02 directly, not a new constraint).
- **C-DPB-03**: This exploration selects no technology, format, label vocabulary, or specific mechanism (§27).
- **C-DPB-04**: Testing convenience must never weaken Runtime semantics (`ACT-005` §4.H, restated, not reinvented).
- **C-DPB-05**: `DPB-05` (CLI non-mandatory) must not be read as CLI being discouraged — only that architectural dependence on it is prohibited.

## 25. Deferred Items

- Exact stability-label mechanism (`DPB-16`) — implementation.
- Exact compatibility-metadata mechanism (`DPB-15`) — implementation.
- Developer identity/account ownership — genuinely unexplored (§20, §34).
- Whether a formally documented "minimum success path" artifact is required per domain, or an informal review criterion suffices (§34).
- Packaging & Distribution's own boundary — largely unexplored; `ACT-005` §12 defers to its own future competitive evaluation.

## 26. External Dependencies

Same base set `DES-002` §9 already names (Diagnostics/Observability workstream; Reference Application catalogue; capability-grant ergonomic work, if pursued; the future validation mechanism), plus: the future CLI, Testing Tooling, and Templates architectures (each will consume the `DPB` requirements directly); the resumed Documentation Platform + SDK Documentation Architecture Authoring (§28, dependency map); the future Control Centre requirements/design and architecture lifecycle, which must inherit this boundary rather than invent a separate one.

## 27. Explicit Non-Goals

Runtime redesign; SDK layer redesign; stable SDK v1.0.0 API redesign; Documentation Platform technology/framework/hosting/search-engine selection; CLI implementation or framework selection; packaging implementation; Developer Portal, Reference Application, or Control Centre implementation; Distributed Runtime; Synapse Cloud; Enterprise Edition; AI Workforce Platform; AI-employee products; trading or other domain-specific commercial applications; commercial application design. Future compatibility is not implementation authorization. Verified none violated during Design Approval Review.

## 28. DES-002 Dependency Map

| DES-002 Requirement | Required Cross-Cutting Principle | Proposed Owner |
|---|---|---|
| R02 (CLI independence) | `DPB-05`, `DPB-14` | This layer defines the rule; Documentation Platform architecture consumes it directly — **now sufficiently defined** |
| R21 (disclosure tiering) | `DPB-06`, `DPB-07` | This layer defines the rule; tier-*marking mechanism* remains Documentation Platform's own open question (`DES-002` §10) — rule sufficiently defined, mechanism not |
| R04/R25 (capability grant/denial documented) | `DPB-09`, `DPB-10`, `DPB-11` | This layer defines the visibility/least-authority rule; Documentation Platform still authors the actual content — **now sufficiently defined** |
| R07 (version-awareness) | `DPB-15` | This layer defines the contract; mechanism deferred | Rule sufficiently defined |
| R11/R26 (baseline measurement) | None | Evidence-methodology requirements, not architecture — remain Documentation Platform-owned, no cross-cutting principle needed |
| R09 (RuntimeError catalogue) | `DPB-13` | This layer requires vocabulary consistency; Documentation Platform authors the catalogue content — **now sufficiently defined** |
| §10 (machine-readable representation) | None (declined to absorb, §17) | Remains Documentation Platform's own open Architecture Question, unresolved, not blocking |
| C02 (no AI-specific construct) | `C-DPB-02` | Direct restatement, no new principle |
| C03 (capability semantics authority) | Consistent with `DPB-09` | No conflict |

**Conclusion**: R02, R04, R09, R21 (rule-level), R25, and R07 dependencies are now sufficiently defined to potentially unblock resumption of Documentation Platform + SDK Documentation Architecture Authoring for those specific items. The machine-readable-representation question and baseline-measurement methodology remain outside this layer's scope, unresolved, and — per `DES-002` itself — not blocking, since `DES-002` never made either a precondition of R02/R21 satisfaction.

## 29. ARCH-014 Compatibility Map

| Candidate Principle | ARCH-014 Relationship |
|---|---|
| DPB-01/03/04/12 | Compatible — SDK-side half already owned (`ARCH-014` §6/§7); this is the Developer-Platform-side generalization of the identical rule, not a redefinition |
| DPB-02 | SDK-specific application of broader principle — `ARCH-014` §4's own generalization test is SDK-scoped; this generalizes it, does not alter it |
| DPB-06/07 | SDK-specific application of broader principle — `ARCH-014` §16 carries the SDK-specific form only |
| DPB-08 | Genuinely new — extends, does not override |
| DPB-09/10/11 | Compatible but not owned — sourced from `GOV-018` directly, constitutionally above `ARCH-014`; no `ARCH-014` content touched |
| DPB-12 | Same as above |
| DPB-13 | SDK-specific application of broader principle — `ARCH-014` §12 |
| DPB-15 | SDK-specific application of broader principle — `ARCH-014` §14 |
| DPB-16 | Genuinely new — no `ARCH-014` equivalent (its four tiers grade API surface, not tooling features) |
| DPB-05/14 | Compatible but not owned — `ARCH-014` does not address CLI at all, consistent with its own non-goals |
| DPB-17 | Genuinely new — no `ARCH-014` equivalent |

**Zero potentially-conflicting proposals found.**

## 30. Developer Choice Test

| Principle group | Answer | Explanation |
|---|---|---|
| DPB-01/03/04/12 (Non-Second-Runtime) | Indirectly | Prevents drift that would erode the platform's own structural differentiator (`RES-008` §13) over time; absence would be felt, presence is invisible |
| DPB-02 (Generalization Test) | No meaningful independent value | Internal enforcement mechanism for `DPB-01`, retained for that reason, not because it "sounds sophisticated" |
| DPB-05/14 (CLI independence) | Yes | Removes lock-in risk, a concrete friction category `RES-008` §5 already names |
| DPB-06/07 (Progressive Disclosure) | Indirectly | Reduces cognitive/relearning cost, matching `RES-008` §11's Should-priority candidate principle |
| DPB-08 (Conceptual Consistency) | Indirectly | Plausible cross-surface friction reducer, `[INF]` |
| DPB-09/10/11 (Capability-Security) | Yes, for enterprise-evaluator persona specifically | Concretizes `RES-008` §13's own "enforced by construction" claim; **not yet evidenced for general application-developer persona** |
| DPB-13 (Error Vocabulary) | Indirectly | Reduces per-surface relearning cost, plausible not evidenced |
| DPB-15/16 (Compatibility/Stability disclosure) | No, but necessary | Hygiene — absence actively repels, presence is not itself a choice-driver |
| DPB-17 (Non-Interactive Path) | Indirectly | CI/automation table-stakes for Contributor/Platform-Engineer persona |

## 31. Simplicity Test

**"Could a developer build a basic SynapseOS application without understanding this document?"** — **Yes**, directly testable: `DES-002` R02 and `DX-001` already demonstrate a complete durable-actor application via documented Stable-tier SDK usage and the manual Cargo workflow alone, with zero CLI, Documentation Platform, Testing Tooling, or templates — none of which exist yet. This is not an accident of the current pre-tooling state: `DPB-05`/`14` are precisely what structurally guarantees this remains true once that tooling exists.

## 32. External Developer Evidence Limitation

`RES-008` §15's limitation is unchanged and directly binding here: **zero SynapseOS external-user evidence exists.** Every "Indirectly" or `[INF]`-tagged conclusion above (`DPB-06`/`07`/`08`/`13`/`17`, and the enterprise-evaluator claim under `DPB-09`–`11`) is a **hypothesis requiring future developer/enterprise validation once an onboarding path exists** — not a measured fact, not to be used in unqualified marketing language, matching `RES-008` §13's own explicit hedge rather than strengthening it.

## 33. Design Risks

| Risk | Mitigation |
|---|---|
| Over-centralizing Developer Platform architecture | Only 17 requirements, each independently sourced; `DPB-01`/`03`/`04`/`12` explicitly disclosed as one principle, not four (§5) |
| Creating a second Runtime | The central risk this document exists to prevent — `DPB-01` + `DPB-02` enforcement |
| Duplicating ARCH-014 | Explicit Compatibility Map (§29), zero conflicts |
| Over-specifying UX as architecture | `DPB-15`/`16`/`13` mechanisms explicitly deferred (§34) |
| Premature abstraction | `DPB-17` disclosed as weakest-evidenced, Should-priority only |
| Architecture driven by hypothetical AI-agent consumers | Explicitly declined (§16); `RES-008` Minor-1 disclosed as insufficient basis |
| Coupling domains unnecessarily | Ownership Matrix (§20) defaults Domain-Specific, cross-cutting only where the Fundamental Design Rule test is met |
| Making CLI mandatory | Directly prevented by `DPB-05` |
| Weakening capability security for convenience | Directly prevented by `DPB-09`–`11`; `C-DPB-04` restates `ACT-005` §4.H |
| Inconsistent concepts across surfaces | `DPB-08`/`13` |
| Requiring developers to understand this document before using the platform | Simplicity Test (§31) — Yes, structurally guaranteed by `DPB-05`/`14` |

## 34. Architecture Questions (deferred, not answered here)

- What formally constitutes a "Developer Platform surface" for `DPB-01`/`02`'s own boundary test?
- What compatibility-metadata mechanism satisfies `DPB-15`?
- What is the complete CLI authority model (specific legitimate commands, not just the boundary principle)?
- What stability-marking mechanism satisfies `DPB-16`?
- What error-contract types/codes satisfy `DPB-13` beyond "must be traceable"?
- What concrete auditability mechanism satisfies `DPB-12`?
- Which concepts require machine-readable representation (left entirely to Documentation Platform architecture, `DES-002` §10 — not solved here)?
- Who owns developer identity/account concerns? **Genuinely unresolved** — no current Approved artifact addresses this.
- Should Progressive Disclosure require a formally documented "minimum success path" artifact per domain, or is an informal review criterion sufficient?

## 35. Traceability Matrix

| Item | GOV-018 | ARCH-014 | ADR-0020 | DES-002 | RES-008 |
|---|---|---|---|---|---|
| DPB-01/04/12 | §4/§5 | §6/§7 | §5 | — | — |
| DPB-02/03 | — | §4, §6/§8 | §5 | — | — |
| DPB-05/14 | — | — | §4 | R02 | §5 friction |
| DPB-06/07 | §5.2 | §16 | §4 | §4 | §11 |
| DPB-08 | §4 | — | — | — | — |
| DPB-09/10/11 | §5 principle 2 | — | — | R04/R25 | §13 |
| DPB-13 | — | §12 | — | R09 | — |
| DPB-15 | — | §14 | — | R07 | — |
| DPB-16/17 | — | — | — | — | (17: not RES-008-sourced, disclosed) |

## Acceptance Framework

This exploration is adequately specified when: every proposed `DPB` requirement has an explicit class, priority, source, affected-domain list, and acceptance method (§23); every inherited constraint is separated from new proposal (§21 vs. §22); the `ARCH-014` and `DES-002` relationships are each mapped with zero unresolved conflicts (§28–§29); every non-goal is verified unviolated (§27); every evidence-thin claim is explicitly hedged (§32). This document's own completion is not self-declared — that was the Design Approval Review's role (`reviewed_by`, above), and ultimately the Founder's.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Design Exploration Draft. Following `GOV-013` §6.4, answering "what should be built?" for the cross-cutting Developer Platform Boundary, scoped narrowly to the principles `ADR-0020` §4 identified as orphaned by `ARCH-013`'s unrecoverability (CLI boundary, general Progressive Disclosure, general SDK/CLI generalization test). Seventeen `DPB` requirements, five constraints, five deferred items. Independent Design Approval Review: 0 Blocking, 0 Major, 2 Minor (both resolved by disclosure), 1 Observation — verdict PASS. |
| 0.1.0 (this filing) | 2026-08-10 | Denver Jacobs (Founder) | Repository Filing. Records genuine Founder acceptance of this document, at this version, as the current cross-cutting Developer Platform Boundary requirements and design baseline (`reviewed_by`, above), including explicit acceptance of the PASS verdict and both disclosed Minor findings exactly as reported. No substantive content altered by this filing — identifier, frontmatter, this Founder Acceptance Notice, and this Revision History entry are the only additions. Does not authorize Architecture Authoring, Control Centre work, or implementation of any kind. |
