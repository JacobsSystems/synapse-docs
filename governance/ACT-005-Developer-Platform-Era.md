---
document_id: ACT-005
title: Developer Platform Era
version: 0.1.0
status: Approved
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.1 — a governance-tier programme authorization, on the same basis ACT-003 established for itself; not itself a Class B Architectural decision.
created: 2026-08-10
last_updated: 2026-08-10
classification: Public
related_documents:
  governance:
    - GOV-003 (Operative, Act 2 Approved — Governance Model; §3.1 Founder authority)
    - GOV-010 (Operative, Act 2 Approved — Decision Framework)
    - GOV-013 (Approved — Engineering Lifecycle; the unmodified downstream lifecycle this charter's workstreams follow)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution; this charter's own constitutional basis)
    - GOV-014 (v0.1.0, Draft, never filed — Roadmap Adoption; recommended a future "ACT-004" scoped to "Establish Durable Runtime State," an objective repository evidence indicates was substantively achieved via EWO-025/ARCH-007/ARCH-012 without a formal charter; this document does not resolve GOV-014 and creates no retroactive ACT-004 — see Numbering Note, below)
    - ACT-003 (Approved, 2026-07-29 — Act 3 Authorization and Charter; the established Act format this document follows)
  roadmap:
    - ROAD-001 (v0.1.0, Approved — SynapseOS Platform Strategy and Roadmap; names Developer Platform the most immediately unblocked strategic era, §3-§4; this charter is subordinate to it)
  standards:
    - STD-001 (Draft; §5 Controlled Document Families — no new family created by this charter, §8 below)
    - STD-031 (Approved, v0.2.1 — Engineering Lifecycle Standard; the unmodified downstream lifecycle §9 below cites)
  architecture:
    - ARCH-008 (Approved, v0.4.3 — Effect Runtime Architecture; §29-§30, the Control Centre boundary evidence, §6 below)
    - ARCH-014 (Approved, v0.7.0 — Synapse SDK Architecture; the stable SDK v1.0.0 surface this charter's documentation/tooling domains target and never redesign)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ACT-005 — Developer Platform Era

> **Status notice.** This document is **Approved**, effective 2026-08-10, per the Founder Approval recorded in the Approval Status section below, using the ordinary mutable Approval Status convention `ACT-003` already established for itself (version unchanged from its own initial Draft, `0.1.0`, exactly as `ACT-003` itself remained at `0.1.0` through its own approval — only `status` transitions).

> **Filing Notice.** This document was authored, independently reviewed, and Founder-approved entirely as chat-delivered material across several stages of this engagement, before this Repository Filing. No substantive content was altered by this filing — identifier, frontmatter, Approval Status, and this Filing Notice are the only additions. This charter is programme authorization only; it does not itself authorize implementation of any of the eleven programme domains it names (§3), and it does not open any future programme (Operations Platform, Distributed Runtime, Synapse Cloud, Enterprise Platform, AI Workforce applications) it names for compatibility purposes only (§17).

> **Numbering Note.** `GOV-014` (Draft, never filed, created 2026-07-30) recommended a future charter be filed as `ACT-004`, scoped to "Establish Durable Runtime State." Direct repository evidence indicates that objective was substantively achieved via `EWO-025` (Durable Actor Recovery), `ARCH-007` (Persistent Actor Architecture, v0.5.2), and `ARCH-012` (Durable DomainState Encoding, v0.2.0) — all Approved by 2026-08-09 — without a formal `ACT-004` charter. This conflict was disclosed and put to the Founder directly rather than silently resolved; the Founder's decision preserved `ACT-004` for `GOV-014`'s own recommendation and numbered this charter `ACT-005`. This charter does not resolve `GOV-014` and does not imply a formal `ACT-004` exists. `GOV-014` itself was not modified by this filing.

## 1. Purpose

`GOV-018` establishes SynapseOS's identity; `ROAD-001` names Developer Platform as the most immediately unblocked strategic era. This charter authorizes that era's own investigation, requirements definition, research, architecture, and review across eleven programme domains (§4). It does not itself authorize implementation of any of them.

## 2. Mission

**Transform SynapseOS from a stable Runtime and public SDK into a developer platform developers can discover, learn, build with, test, package, operate, and adopt successfully.**

The central programme test — *would this make developers choose SynapseOS over the alternatives?* — is translated into the measurable outcomes of §12, not left as marketing language. Every workstream under this charter must trace to at least one of: developer productivity, onboarding, comprehension, confidence, application quality, testing quality, packaging/distribution, ecosystem participation, commercial adoption, or long-term maintainability. No workstream may exist merely because it is technically interesting.

## 3. Authorization Boundary

**This is programme authorization, not implementation authorization.** A domain's appearance in §4 means: *this area is within the legitimate scope of the Developer Platform Era* — investigation, requirements definition, research, architecture, and independent review may begin now. It does **not** mean: *implementation of this area is already approved.* Every domain still requires its own future, separately authored and approved `EWO` (or, where architectural in nature, its own `ARCH`/`ADR` reaching Approved status first) before a single line of code is written, following the full downstream lifecycle of §9. This charter authorizes a *programme*; it delegates every *component* decision to that programme's own future, individually reviewed artifacts.

## 4. Authorized Programme Domains

**A. Synapse CLI** — project creation, scaffolding, local execution, diagnostics, testing integration, packaging integration, developer workflows. *No command is designed here.*
**B. Documentation Platform** — architecture, conceptual documentation, guides, navigation, discoverability, reference and release documentation. *No documentation technology is selected here.*
**C. SDK Documentation** — stable public API reference, examples, conceptual and compatibility guidance, discoverability. *Reflects the approved SDK; never a mechanism for silently redesigning it.*
**D. Interactive Tutorials** — first application, actors, capabilities, effects, supervision, durable execution, AI-native workloads where appropriate. *No tutorial implementation platform is designed here.*
**E. Official Templates** — production-quality starting points. *Must demonstrate approved architecture, never invent alternative architectural patterns; no final catalogue defined here.*
**F. Reference Applications** — proof that real applications can be built on SynapseOS; learning resources, architecture examples, integration evidence, testing assets, adoption assets. *No domain application becomes part of core SynapseOS.*
**G. Developer Portal** — onboarding, documentation access, project resources, ecosystem discovery. *Assumes none of the following exist unless separately evidenced and authorized: Synapse Cloud, required accounts, hosted services, a marketplace.*
**H. Testing Tooling** — actor testing, capability mocking, failure simulation, deterministic validation, replay, integration/workload testing. *Testing convenience must never weaken Runtime semantics — any proposal to do so is an Engineering Stop (`GOV-017` §6), not a testing-tooling decision.*
**I. Packaging & Distribution** — packaging, dependency consumption, compatibility, versioning, distribution, signing, possible future registries. *No format, registry, or container model is preselected without dedicated evidence (§12).*
**J. Community & Ecosystem Foundations** — contributor onboarding, contribution standards, examples, extension ecosystem, community templates, ecosystem governance. *Mentioning ecosystem growth never itself authorizes a marketplace.*
**K. Commercial Adoption Foundations** — adoption friction, enterprise evaluation, production-readiness and support expectations, integration requirements, sustainable platform models. *No pricing defined; no Enterprise Edition or Synapse Cloud authorized; commercial pressure may never silently weaken a constitutional guarantee — any such proposal routes through `GOV-018`'s own Amendment process (`GOV-018` §10), never resolved inside this workstream.*

## 5. Explicit Exclusions

This charter authorizes **no implementation** of: Runtime redesign; stable SDK redesign; Distributed Runtime; Synapse Cloud; Enterprise Edition; AI Workforce Platform; AI-employee products; trading applications or any other domain-specific commercial application as core SynapseOS; a provider marketplace without its own separate approval; the full Control Centre (§6); or any architectural change justified by developer convenience alone. **These may appear in strategic compatibility discussions (`ROAD-001` §3). Compatibility is not authorization (§17). No future programme begins merely because this charter, or `ROAD-001`, anticipates it.**

## 6. Control Centre Boundary

Verified directly against `ARCH-008` §3/§4/§30 and `ACT-003` §2 (which already excluded "a production Runtime Control API or Control Centre user interface" from Act 3, citing `ARCH-008` §29–30 as "explicitly deferred"): no repository document grants authority to build the Control Centre; `ARCH-008` §29 preserves a compatible future path only. Accordingly, this charter authorizes work needed to ensure Developer Platform interfaces do not unnecessarily obstruct that future path — requirements research, developer/operations interface analysis, Runtime Control API compatibility considerations, observability requirements relevant to future tooling. **This charter does not authorize building the Control Centre.**

## 7. Developer Personas

**Adopted, narrowly, as analytical tools only — not new authority classes, not rigid product categories, not decoration:**

1. **Application Developer** — primary audience for CLI, Documentation, Tutorials, Templates, Testing Tooling.
2. **Contributor / Platform Engineer** — primary audience for Community & Ecosystem Foundations.
3. **Enterprise Evaluator/Operator** — primary audience for Commercial Adoption Foundations.

Each persona's purpose is limited to requirements-traceability within §4's domains; none carries decision authority, and none may justify a workstream that does not otherwise trace to §2's measurable outcomes.

## 8. Downstream Workstream Lifecycle

No parallel process is introduced. Every workstream follows the existing `GOV-013`/`STD-031` lifecycle: Research → Requirements/Design → Architecture Authoring → Independent Architecture Review → Amendment (if required) → Re-Review → Founder Architecture Approval → Engineering Work Order → Implementation → Independent Engineering Review → Amendment (if required) → Re-Review → Founder Implementation Approval → Engineering Report → Publication. No new controlled document family is created by this charter — `STD-001` §5 already registers every family these workstreams need (`RES`, `ARCH`/`ADR`, `EWO`/`ER`, `RSS`/`ACR`/`AFR` where volume warrants consolidation). Workstreams may be referenced conceptually ("the CLI workstream") without an unsupported identifier family such as `DX-NNN`/`PRD-NNN`/`TAS-NNN`; if a genuine future need for one emerges, that is a `STD-001` amendment requirement, named here, not created here.

## 9. Governance Proportionality

Applying full architecture-tier governance to every documentation typo or minor example fix would itself become an adoption barrier. `GOV-013` §12 already establishes the required mechanism — a narrower, disclosed-and-justified proportional realization of the full lifecycle remains permitted (demonstrated precedent: `EWO-010`, a documentation-only correction requiring no full Engineering Report). This charter does not amend that mechanism; it directs that Documentation, SDK Documentation, Tutorials, and Templates workstreams apply it deliberately for low-risk editorial changes, while Architecture, public-API, security/capability, compatibility, and major-interface decisions receive the full lifecycle without exception. No governance change is made here; none is required.

## 10. Dependency Model

```
                    ROAD-001 (Developer Platform priority)
                              │
                              ▼
                    Developer Requirements
                    (personas, §7; baseline
                     evidence, §11–§12)
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        Synapse CLI    Documentation     Testing Tooling
                        Platform (+ SDK
                        Documentation)
              │               │               │
              └───────┬───────┴───────┬───────┘
                      ▼               ▼
                 Templates      Interactive
                                 Tutorials
                      │               │
                      └───────┬───────┘
                              ▼
                    Reference Applications
                              │
                              ▼
                 Packaging & Distribution
                              │
                              ▼
              Community & Ecosystem Foundations
                              │
                              ▼
              Commercial Adoption Foundations
                              │
                              ▼
                     Developer Portal
```

CLI, Documentation Platform (+ SDK Documentation), and Testing Tooling are each independently buildable against the stable v1.0.0 SDK and share no dependency on each other — parallel first tier. Templates and Tutorials both consume CLI/Documentation output — second tier. Reference Applications follow both, since a credible reference application should demonstrate the CLI/template/tutorial path actually works, not bypass it. Packaging & Distribution follows Reference Applications, since packaging strategy is best evidenced by what real applications actually need. Community/Ecosystem and Commercial Adoption both depend on there being something real to onboard contributors or evaluators against. The Developer Portal, an aggregation surface, is last. Not every domain need execute in strict sequence — the diagram marks genuine dependencies, not a schedule.

## 11. Evidence Programme

Required before major decisions in each stage:

- **Before requirements:** developer workflow studies; a documented baseline for every §12 metric category currently lacking one (all except Maintainability).
- **Before architecture:** competitive/platform analysis (§12, below); prototype evidence where a domain's design space is genuinely open (packaging format, CLI UX pattern).
- **Before implementation:** the completed architecture's own Independent Review; reference-application experience where a domain's real-world fit is otherwise unproven.
- **Before declaring programme success:** external developer evaluation; documentation usability testing; packaging experiments exercised outside the core team; operational feedback from the Reference Applications workstream.

## 12. Competitive Evaluation Requirement

Required before Documentation Platform, CLI, and Packaging/Distribution architecture proceeds, following this project's own demonstrated `RES`/`RSS` comparative-research methodology. Candidate comparators: Rust/Cargo, Docker, Kubernetes, Temporal, Dapr, Ray, actor frameworks generally, AI-agent frameworks, serverless developer platforms. Required questions: which developer expectations have become standard; where existing platforms create friction; what SynapseOS should deliberately do differently; which advantages follow from SynapseOS's own actual, already-approved architecture rather than from an unsupported superiority claim. **This research does not authorize architectural imitation** — a competitor doing something a particular way is evidence to weigh, never grounds to copy without SynapseOS-specific justification, and no claim of SynapseOS superiority may be made without that same evidentiary basis.

## 13. Success Metrics Framework

No target below is fabricated; every category lacking a baseline requires baseline research as its own first deliverable.

| Category | Measure | Baseline status |
|---|---|---|
| Onboarding | Time for a competent developer to reach a successful first application | No baseline exists |
| Productivity | Difficulty of common development workflows | No baseline exists |
| Diagnostics | Whether developers can understand and resolve failures unaided | No baseline exists |
| Documentation | Whether developers can discover and correctly use the stable SDK without undocumented internals | No baseline exists |
| Testing | Whether developers can reliably validate applications before production | No mechanism currently exists |
| Packaging | Reproducibility of packaging/consumption | No mechanism currently exists |
| Reference Applications | Whether realistic applications can be built using only approved public interfaces | Zero reference applications exist under this charter's own scope |
| External Validation | Whether external developers can build meaningful applications without core-team assistance | Not yet attempted |
| Maintainability | Whether Developer Platform evolution preserves Runtime/SDK stability and clarity | Baseline exists: `ARCH-014` v0.7.0's four-tier Public API Architecture, though its tier-marking mechanism remains deferred (`ARCH-014` §19) |

## 14. Commercial Discipline

Investigation required (§4.K): why organizations would adopt SynapseOS; migration friction; operational confidence requirements; support expectations; integration requirements; total developer/operational cost. **No pricing is defined. No Enterprise Edition is authorized. No Synapse Cloud is authorized. No revenue forecast is made.**

## 15. Programme Risks

| Risk | Consequence | Mitigation Principle | Evidence Needed |
|---|---|---|---|
| Excellent Runtime, poor developer experience | Adoption fails despite sound architecture | Every workstream traces to §2's measurable outcomes, never technical interest alone | §13 baseline + post-launch measurement |
| Excessive conceptual complexity | Onboarding failure | `ARCH-014`'s own Curation Test, generalized to every developer-facing surface | Onboarding time-to-success measurement |
| Weak onboarding | Developers abandon before first success | §13 Onboarding baseline required before any target | Time-to-first-application measurement |
| Poor diagnostics | Developers cannot self-resolve failures | Testing Tooling and CLI held to the same quality-gate discipline as Runtime/SDK engineering | Diagnostic-resolution measurement |
| Weak or stale documentation | Developer trust erosion | Documentation tied to SDK stability tier (§4.C); no undocumented Stable-tier item permitted | SDK coverage measurement |
| Unstable developer tooling | Productivity loss, abandonment | CLI/Testing Tooling held to the same quality gates as Runtime/SDK | Tooling defect/regression tracking |
| Over-engineering | Wasted effort, delayed real developer value | §13 baseline-first discipline; every workstream justified against measured, not assumed, need | Baseline + ongoing developer feedback |
| Premature Cloud/Distributed Runtime work smuggled in via "developer platform" framing | Architectural risk taken without the required foundation | §5/§17's explicit exclusions and non-goals, enforced at every workstream's Architecture Trace Matrix | None outstanding — structural |
| Premature ecosystem expansion outrunning review capacity | Governance debt, quality erosion | `ROAD-001` §7's own named risk; proportional governance (§9) required before wide contribution is invited | Contribution-volume vs. review-capacity tracking |
| Building features developers do not need | Wasted engineering investment | §13's baseline-first, evidence-before-opinion discipline | Ongoing developer feedback loop, not yet established |
| Insufficient real-world reference applications | Production-readiness claims go unproven | §10's dependency placement (Reference Applications after CLI/Templates/Tutorials) | Reference-application count and authorship independence |
| Governance too burdensome for ordinary contributions | Contributor friction, ecosystem stagnation | §9 Governance Proportionality, deliberately applied | Time-to-review measurement for small contributions |
| Developer convenience eroding architecture | Slow loss of constitutional guarantees | §4.H's explicit prohibition on Runtime-semantics changes for testing convenience, generalized | Architecture Trace Matrix audit per workstream |
| Vendor/model coupling via flagship-workload tooling | Violates `GOV-018` §7 | Every template/tutorial reviewed against `GOV-018` §2/§7 before publication | None outstanding — structural |
| Unstable packaging/distribution | Broken or irreproducible install experience | §12 competitive evidence required before format/registry choice | Packaging experiment results |
| Commercial pressure distorting technical priorities | Constitutional guarantee weakened for revenue | §4.K's explicit routing requirement through `GOV-018`'s Amendment process; `ROAD-001` §6's own zero-tolerance tracked indicator | None outstanding — structural |

## 16. Completion Criteria

The Developer Platform Era is complete when SynapseOS has crossed from *stable Runtime + public SDK* to *independently usable developer platform* — not when every imaginable feature exists. Evidence required across all of:

- Coherent, measured onboarding meeting a Founder-reviewed bar set only after §13's baseline research.
- Stable developer tooling (CLI, Testing Tooling) passing the same quality gates as Runtime/SDK engineering.
- Comprehensive documentation of the stable public SDK, with validated learning-path completion.
- Reproducible packaging/distribution, independently exercised.
- A Founder-reviewed minimum set of production-quality reference applications, spanning more than one category, at least one substantively built or evaluated outside the core team.
- At least one external developer successfully building without direct core-team assistance, friction documented.
- No dependency on undocumented internal APIs anywhere in the Developer Platform surface.
- Evidence that Operations Platform and Ecosystem Growth (`ROAD-001` §3) can begin independently on Developer Platform's own output.

Reviewed the same way an Architecture Freeze Review (`STD-001` §52) recommends a disposition without holding independent approval authority — the actual completion determination remains Founder-tier.

## 17. Future Programme Boundaries

This charter preserves strategic compatibility with Operations Platform, Distributed Runtime, Synapse Cloud, Enterprise Platform, and AI Workforce-class applications, per `ROAD-001` §3. **Future compatibility is not future programme authorization.** This charter cannot authorize those programmes indirectly, regardless of how naturally they might seem to follow.

## 18. Governance Relationship

`GOV-018` (canonical identity) → `ROAD-001` (subordinate strategy/roadmap) → `ACT-005` (this charter — programme authorization, subordinate to both) → future requirements/research/architecture (§4, §8) → Independent Review → Founder Approval where required → Engineering authorization/`EWO` → Implementation → Engineering Review → Publication. No tier below `GOV-018` or `ROAD-001` may contradict either; nothing in this charter authorizes any specific implementation directly.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial Draft. Numbered `ACT-005` per explicit Founder decision resolving the `GOV-014`/`ACT-004` identifier conflict. Authorizes investigation/research/architecture/review only, across eleven programme domains; implementation of any component requires its own future `EWO`. Independently reviewed: zero Blocking, zero Major, zero Minor findings; one non-blocking Observation (`GOV-014`'s own unresolved `ACT-004` recommendation, disclosed in the Numbering Note above, not corrected by this charter). |
| 0.1.0 (this filing) | 2026-08-10 | Denver Jacobs (Founder) | Repository Filing. Records genuine Founder Approval (below) of the 0.1.0 text unchanged — version not incremented, mirroring `ACT-003`'s own identical precedent (approval alone does not require a version transition). No substantive content altered by this filing. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-08-10 |
| Independent Review | — | Blocking: 0; Major: 0; Minor: 0; Observation: 1 (`GOV-014`/`ACT-004`, non-blocking, disclosed, not resolved by this charter) | 2026-08-10 |
| Founder Approval | Denver Jacobs, Founder, exercising `GOV-003` §3.1 authority for a governance-tier programme authorization | **Approved** | 2026-08-10 |

**Founder Approval, recorded as declared:**

> "APPROVED. I hereby approve: ACT-005 — Developer Platform Era v0.1.0 as the programme authorization for the SynapseOS Developer Platform Era. I approve its mission: Transform SynapseOS from a stable Runtime and public SDK into a developer platform developers can discover, learn, build with, test, package, operate, and adopt successfully. I confirm the authority chain: GOV-018 → ROAD-001 → ACT-005 → separately governed downstream work. I approve the eleven programme domains defined by ACT-005 as legitimate areas of investigation, requirements definition, research, architecture, review, and subsequent engineering work only through the established downstream governance lifecycle. This approval does not constitute blanket implementation authorization for the eleven programme domains. A programme domain appearing in ACT-005 means that the domain is legitimately within the Developer Platform Era. It does not mean implementation of that domain is already approved. Downstream work must continue through the applicable SynapseOS governance, architecture, review, Founder approval, Engineering Work Order, implementation, engineering review, and publication processes defined by the existing governance baseline. This approval does not authorize: Runtime redesign; stable SDK redesign; Distributed Runtime implementation; Synapse Cloud implementation; Enterprise Edition implementation; AI Workforce Platform implementation; AI employee products; domain-specific commercial applications as core SynapseOS; marketplace implementation without separate authority; full Control Centre implementation; architectural changes justified merely by developer convenience. Future compatibility is not future programme authorization. I reaffirm the previously made Founder decision: ACT-004 remains historically associated with GOV-014's unfiled recommendation concerning 'Establish Durable Runtime State.' No retroactive ACT-004 is created by this approval. The Developer Platform Era is correctly identified as ACT-005. GOV-014's unresolved recommendation remains a separate governance-hygiene matter and does not block ACT-005. I acknowledge the reported Independent Review result: Blocking findings: 0; Major findings: 0; Minor findings: 0; Observations: 1. The GOV-014 observation is accepted as non-blocking and carried forward for separate governance treatment if required. Upon controlled filing and publication of this approved ACT-005, the SynapseOS Developer Platform Era may be considered formally opened. This approval does not itself authorize bypassing the controlled filing/publication step."

This Filing is genuinely **Approved** on the ordinary, mutable Approval Status convention `ACT-003` and this engagement's own `EWO-026.x`/`GOV-018`/`ROAD-001` filings already use. Upon this Filing's own controlled commit, push, and remote verification, the SynapseOS Developer Platform Era is formally open for its own, separately governed downstream work — no domain named in §4 is thereby implementation-authorized (§3, binding).
