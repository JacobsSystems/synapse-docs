---
document_id: GOV-013
title: Engineering Lifecycle
version: 0.1.0
status: Draft
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.1/§5 — a Governance-tier document, approved at the Governance tier on the same basis as GOV-002, GOV-003, GOV-004, and GOV-010.
created: 2026-07-26
last_updated: 2026-07-26
classification: Public
related_documents:
  governance:
    - GOV-001 (Draft — legacy .docx; Project Charter, "Documentation before implementation" founding principle)
    - GOV-002 (v0.1.0, Draft — Vision and Mission)
    - GOV-003 (v0.1.0 — Operative, Act 2 Approved; roles and approval authority this document's stages draw on without redefining)
    - GOV-004 (v0.1.0, Draft — Engineering Principles; §1 "Documentation Before Implementation" is the principle this document's stage ordering exists to enforce)
    - GOV-010 (v0.1.0 — Operative, Act 2 Approved; Decision Framework this document's Change Control section defers to)
    - GOV-011 (v0.1.1, Draft — Architecture Review Board Charter; the demonstrated mechanism realizing this document's Architecture Review stage at foundational scale)
    - GOV-012 (v0.1.1, Draft — Architecture Review Board Session 001; evidence of the Architecture Review stage in practice)
  standards:
    - STD-001 (v0.1.0 and v0.2.0 Approved via normal-governance disposition; current v0.4.0 content not separately approved — the sole authority for document families, identifiers, and evidence representation this document defers to throughout)
  architecture:
    - ARCH-001 (Draft — cited as an example Architecture Authoring output)
    - ARCH-004 (Draft — cited as an example Architecture Authoring output and Architecture Review subject)
    - ARCH-005 (Draft — cited as an example Architecture Authoring output)
    - ARCH-007 (Draft — cited as the most recently demonstrated full Architecture Authoring / Architecture Review / Architecture Correction / Narrow Architecture Re-Review cycle)
  rfcs: None
  adrs:
    - ADR-0011 (Draft, Act 1/Act 2 effective — source of the Publication evidence model this document's Publication stage cites)
    - ADR-0012 (Approved — corrective refinement of the same evidence model)
  roadmap: None
  research:
    - RES-001 through RES-006 (Draft — cited as example Research-stage outputs)
  operational: None
  consolidation:
    - RSS-001 (v0.1.2, Draft — cited as the demonstrated Research Review mechanism at multi-document scale)
    - ACR-001 (v0.2.0, Draft — cited as the demonstrated architecture-versus-evidence consolidation mechanism)
    - AFR-001 (v0.1.1, Draft — cited as the demonstrated certification-gate mechanism)
    - DES-001 (v0.2.0, Draft — cited as the demonstrated Design Exploration / Design Correction mechanism)
  engineering:
    - EWO-001, EWO-002, EWO-007, EWO-009, EWO-010, EWO-011 (cited as example Engineering Work Order outputs, including two non-standard historical-reconstruction instances)
    - ER-001, ER-007, ER-008, ER-009, ER-010 (cited as example Engineering Report outputs and evidence of the Independent Implementation Review step)
  source_artifacts: None
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# GOV-013 — Engineering Lifecycle

> **Status notice.** This document is **Draft**. No GOV-013-specific approval act has occurred. Drafting, saving, staging, committing, or pushing this document does not itself constitute its approval. See §14 (Approval Status).

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 §33 (AI-Assisted Documentation) and GOV-010 §15. AI output is not automatically authoritative.

> **A note on identifier assignment.** This document was originally requested under the identifier "GOV-003." Repository verification found `governance/GOV-003-Governance-Model.md` already exists, is a different document (Governance Model: roles, decision process, document hierarchy), and is genuinely, verifiably Approved under Bootstrap Act 2 (independently hash-verified against commit `43a49b4` during this document's own preparation: SHA-256, git blob ID, byte size, and line count all match exactly, and the artifact is unchanged since approval). STD-001 §7 ("Identifiers are permanent. A retired or superseded identifier must not be reassigned to a different document.") forecloses reassigning GOV-003. This content is filed under GOV-013, the next unused identifier in the GOV family (GOV-001 through GOV-012 are all taken), following disclosure to and confirmation by the document's requester.

## 1. Purpose

This document defines the mandatory engineering lifecycle used for every major SynapseOS capability: the sequence of stages a capability passes through from initial idea to published, evidenced completion; the question each stage answers; who is responsible for it; what it may and may not produce; and how correction, review, and publication are carried out.

This document governs **how engineering work progresses**. It does not define any specific technical capability, any architecture, or any implementation. It documents a lifecycle that has already been demonstrated, repeatedly, in this project's own history — it does not invent one.

## 2. Scope

**In scope:** the mandatory lifecycle stages from Idea through Publication; the single question each stage answers; each stage's permitted activities, prohibited activities, required outputs, entry criteria, and exit criteria; the discipline governing independent review, findings classification, correction, and re-review; the engineering principles this lifecycle exists to enforce; the relationships between Governance, Research, Design, Architecture, Engineering Work Orders, Implementation, Reviews, and Engineering Reports; and change control — when correction suffices, when a narrow re-review suffices, and when a stage must be reopened.

**Out of scope:** any specific technical capability, architecture, or implementation; the definition or amendment of controlled-document families, identifiers, metadata, or file conventions (STD-001's exclusive domain); the governance roles and decision-authority framework themselves (GOV-003 and GOV-010's domain, which this document cites and does not restate); capability-specific engineering rules of any kind. Where this document names a document family (RES, ARCH, EWO, ER, RSS, ACR, AFR), it does so only to describe how that already-registered family is used within the lifecycle — it registers no new family and amends no existing one.

## 3. Relationship to GOV-001, GOV-002, GOV-003, GOV-004, and GOV-010

- **GOV-001** (Project Charter) states the founding principle this entire document exists to operationalize: "Documentation before implementation... Claude Code implements approved specifications." This document defines the stages through which that principle is actually carried out.
- **GOV-002** (Vision and Mission) states enduring principles, including "Architecture that evolves through evidence rather than opinion" (§6.5) — the same principle this document's Research and Research Review stages exist to satisfy.
- **GOV-003** (Governance Model) defines *who* — Founder, Chief Architect, Document Author, Reviewer, Approval Authority, Implementer — and *what authority* each role holds. This document does not redefine any role or authority; every reference to "review," "approval," or "author" in this document draws its meaning from GOV-003 §3.
- **GOV-004** (Engineering Principles) states principles this document's stages exist to enforce mechanically: "Production code MUST NOT be written without an approved architectural specification" (§1) is why Architecture Authoring precedes Implementation in the mandatory sequence below; "Every implementation MUST map back to ARCH, RFC and ADR documents" (§11) is why Engineering Work Orders cite their authorizing architecture and Engineering Reports cite their authorizing Work Order.
- **GOV-010** (Decision Framework) defines *how* a decision is processed once a role with authority must act: the decision classes (§4), the distinction between review and approval as separate acts (§5), the evidence standard (§9), and the lifecycle-and-disposition model (§21) this document's own stages produce inputs for. This document's Change Control (§12) explicitly defers to GOV-010 rather than restating it.

**Division of responsibility, stated plainly:** GOV-003 assigns authority to roles. GOV-010 defines how a role exercises that authority to reach a decision. **This document defines the sequence of engineering stages a capability must pass through before a Class B (Architectural) or Class E (Implementation) decision under GOV-010 is even ready to be made or implemented.** It is process scaffolding beneath GOV-010, not a competing decision framework.

## 4. Core Principle — The One-Question Rule

Every engineering stage answers **one question only**. A stage must never answer a question assigned to another stage. This is not a stylistic preference; it is the same separation-of-concerns discipline SynapseOS's own architecture applies to itself (ARCH-001 §9's mechanism/policy test; ARCH-004 §8's policy/mechanism separation for supervision) applied one level up, to the process that produces architecture in the first place.

| Stage | Question |
|---|---|
| Idea | Is there a need worth investigating? |
| Research | What does the evidence show? |
| Research Review | Is the evidence complete and correctly interpreted, and — where architecture already exists — is it supported by that evidence? |
| Design Exploration | What should be built? |
| Design Approval Review | Is this the correct design? |
| Design Correction | Have the identified design findings, and only those findings, been resolved? |
| Narrow Design Re-Review | Were the identified findings actually resolved, without new drift? |
| Architecture Authoring | What must always be true? |
| Architecture Review | Does the architecture faithfully express the approved design? |
| Architecture Correction | Have the identified architecture findings, and only those findings, been resolved? |
| Narrow Architecture Re-Review | Were the identified findings actually resolved, without new drift? |
| Engineering Work Order | How will the approved architecture be implemented? |
| Implementation | Has the work been implemented? |
| Independent Implementation Review | Does the implementation faithfully satisfy the Engineering Work Order? |
| Engineering Report | What evidence proves completion? |
| Publication | Is the approved artifact now a durable, evidenced part of the repository? |

A stage that answers a question belonging to another stage has exceeded its mandate, regardless of good intent. A Research document that recommends a specific design has exceeded Research's mandate (compare RES-001 §8's own explicit "Non-Recommendation Statement"). An Engineering Work Order that redefines architecture has exceeded its mandate (GOV-004 §1; every demonstrated EWO's own "Engineering Authority" section instead *cites* specific ARCH sections rather than restating or altering them). A correction pass that redesigns anything beyond its named findings has exceeded its mandate (§8.4 below).

## 5. Mandatory Lifecycle

```text
Idea
    │
    ▼
Research
    │
    ▼
Research Review
    │
    ▼
Design Exploration
    │
    ▼
Design Approval Review
    │
    ▼
Design Correction (if required)
    │
    ▼
Narrow Design Re-Review
    │
    ▼
Architecture Authoring
    │
    ▼
Architecture Review
    │
    ▼
Architecture Correction (if required)
    │
    ▼
Narrow Architecture Re-Review
    │
    ▼
Engineering Work Order
    │
    ▼
Implementation
    │
    ▼
Independent Implementation Review
    │
    ▼
Engineering Report
    │
    ▼
Publication
```

Every major SynapseOS capability passes through this sequence. A stage may be skipped only where its own entry criteria (§6) are already satisfied by prior work — never by omission. The "(if required)" stages are conditional exactly once: if the preceding review returns a verdict requiring no change, the correction and re-review stages do not occur, and the lifecycle proceeds directly to the next unconditional stage.

**A note on demonstrated variation, disclosed rather than concealed (§13.1).** Not every stage below currently corresponds to a registered STD-001 controlled-document family, and the depth at which some stages are realized scales with the significance of the capability under development. Where this is so, it is stated explicitly in that stage's own definition (§6) rather than presented as uniform when the evidence shows otherwise.

## 6. Stage Definitions

Each stage below states: Purpose; Permitted Activities; Prohibited Activities; Required Outputs; Entry Criteria; Exit Criteria.

### 6.1 Idea

**Purpose.** Identify a need, gap, or problem sufficient to justify future Research or Architecture Authoring.

**Permitted.** Informal problem statements; identifying a gap already disclosed by existing architecture (for example, ARCH-002 §23's Deferred Architecture table naming "Storage Architecture" as a future need, which this project's own history shows became the origin point for the Persistent Actors capability).

**Prohibited.** Committing to a design, technology, or architecture; authorizing any engineering work; treating an idea as itself a decision.

**Required outputs.** None mandatory. May be recorded informally.

**Entry criteria.** None — this is the lifecycle's origin point.

**Exit criteria.** A specific, bounded question exists that Research or Architecture Authoring can address.

### 6.2 Research

**Purpose.** What does the evidence show?

**Permitted.** Comparative study of existing systems, precedent, and documented design models; explicit disclosure of sourcing method and limitations; explicit non-recommendation.

**Prohibited.** Recommending a specific SynapseOS design; architecture; implementation. RES-001 through RES-006 each close with an explicit "Non-Recommendation Statement" and disclose that "any such recommendation must be raised through its own EWO/ADR/ARCH-conformance process, citing this document as evidentiary context only" (RES-001 §8) — this is the demonstrated, correct posture for this stage, not an optional courtesy.

**Required outputs.** For a capability whose research need is broad or foundational, one or more Research (RES) documents, registered per STD-001 Appendix B (demonstrated: RES-001 through RES-006, each examining a distinct facet of SynapseOS's architecture against existing precedent). For a narrower, single-capability research need, comparative analysis may instead be embedded directly within the Design Exploration stage's own document (demonstrated: DES-001's own "Comparative Analysis" section, studying seven external persistence models inline rather than as separate RES documents). The choice between these two forms is not a shortcut — it is scaled to the reach of what is being investigated, disclosed explicitly in the resulting document either way.

**Entry criteria.** An Idea-stage question exists.

**Exit criteria.** Evidence is documented, sourced, and disclosed together with its own limitations; no design recommendation has been made.

### 6.3 Research Review

**Purpose.** Is the evidence complete and correctly interpreted — and, where architecture already exists, is that architecture supported by the evidence?

**Permitted.** Consolidating multiple Research outputs into one synthesized evidence base without adding new findings (demonstrated: RSS-001, which "performs no new comparative research... consolidates the evidence established by RES-001 through RES-006"); evaluating existing architecture against that evidence base, classifying each material claim as Supported, Partially Supported, Unsupported, Contradicted, or Outside Research Scope (demonstrated: ACR-001, which is explicitly "deliberately adversarial in method" and treats absence of contradiction as requiring scrutiny, not comfort); determining whether the evidence and architecture together are ready to serve as a certified baseline (demonstrated: AFR-001's five-gate certification model).

**Prohibited.** Introducing new research; introducing or amending architecture; treating implementation quality, design intent, or the sheer existence of documentation as proof of architectural validity (ACR-001 §3's own explicit exclusion).

**Required outputs.** For a multi-document Research programme, a Research Synthesis Review (RSS) and, where existing architecture must be checked against it, an Architecture Consolidation Review (ACR) and an Architecture Freeze Review (AFR) — per STD-001 §50–§52. For a narrower, single-capability Research effort realized inline within a Design Exploration document, independent verification of that evidence is instead performed as part of Design Approval Review (§6.5) rather than as a separately filed document — this is the demonstrated pattern for the Persistent Actors capability, where DAR-001 (not a standalone RSS/ACR) independently checked DES-001's own comparative analysis.

**Entry criteria.** One or more Research outputs exist.

**Exit criteria.** The evidence base is confirmed complete and correctly interpreted; where architecture already exists, every material claim checked has a disclosed disposition — including "insufficient evidence to conclude" or "outside research scope," which are legitimate, non-deficient outcomes (ACR-001 §2), never silently resolved one way or the other.

### 6.4 Design Exploration

**Purpose.** What should be built?

**Permitted.** Comparing design alternatives against explicit criteria (a decision matrix); recommending a specific design with disclosed trade-offs; separating decisions the exploration itself settles from questions explicitly deferred to Architecture Authoring (demonstrated: DES-001's own "Already decided by DES-001" / "Deferred to ARCH-007" split).

**Prohibited.** Normative architecture language (MUST/MUST NOT statements binding future implementation); implementation detail of any kind — APIs, types, method names, pseudocode, storage technologies.

**Required outputs.** A Design Exploration document. **This stage's output is not currently a registered STD-001 controlled-document family.** Where an output is filed, it uses the narrowest existing, purpose-consistent repository location and identifier convention already available, with the placement's own rationale disclosed inside the document itself — exactly as DES-001 did (filed in `consolidation/`, alongside RSS/ACR/AFR, with an explicit `document_family_note` stating this is "a disclosed, narrow convenience, not a documentation-hierarchy redesign"). This document does not register "DES" as a family; that decision, if ever wanted, belongs to a future, separately authorized amendment to STD-001, not to this document or to the act of filing any single Design Exploration output.

**Entry criteria.** Research and Research Review are complete for the capability in question.

**Exit criteria.** A recommended design exists, with alternatives considered, trade-offs disclosed, and an explicit list separating decisions already made from questions deferred to Architecture Authoring.

### 6.5 Design Approval Review

**Purpose.** Is this the correct design?

**Permitted.** Independent, skeptical evaluation of the Design Exploration output against the evidence base and existing architecture; findings classified by severity; a recorded verdict.

**Prohibited.** Redesigning the capability; proposing architecture or implementation; reopening a question the Design Exploration output did not itself raise.

**Required outputs.** A recorded verdict, with findings precisely identified where the verdict requires change. This stage's own output is not currently a registered, separately-filed document family; the demonstrated practice (DAR-001) records the review as an independently conducted, evidence-based act whose disposition — verdict, findings, and their eventual resolution — is preserved in the Design Exploration document's own frontmatter (`reviewed_by`) and Revision History, not lost to an unfiled conversation.

**Entry criteria.** A Design Exploration output exists.

**Exit criteria.** A verdict is reached: either the design is confirmed sufficient to proceed to Architecture Authoring, or specific, named findings require Design Correction.

### 6.6 Design Correction (if required)

**Purpose.** Have the identified design findings, and only those findings, been resolved?

**Permitted.** Applying exactly the corrections named by Design Approval Review; disclosing precisely what changed and why.

**Prohibited.** Redesigning beyond the named findings; reopening decisions the review did not flag; introducing new research, new alternatives, or new design questions.

**Required outputs.** A corrected Design Exploration document (a new version), with a Correction Notes or Revision History entry naming each finding and its exact fix (demonstrated: DES-001 v0.1.0 → v0.2.0, applying "exactly the four findings of DAR-001... No other content changed").

**Entry criteria.** Design Approval Review returned a verdict requiring change.

**Exit criteria.** Every named finding has been addressed; nothing else in the document has changed.

### 6.7 Narrow Design Re-Review

**Purpose.** Were the identified findings actually resolved, without introducing new drift?

**Permitted.** Re-checking exactly the corrected items; confirming the correction itself introduced no new defect.

**Prohibited.** Reopening the design; introducing findings unrelated to the original review; repeating the full Design Approval Review.

**Required outputs.** A narrow verdict (demonstrated: the DAR-001 narrow re-review of DES-001 v0.2.0, explicitly scoped to "strictly to the four corrections").

**Entry criteria.** Design Correction is complete.

**Exit criteria.** The narrow verdict is reached. The design proceeds to Architecture Authoring, or — only if the correction itself was deficient — returns to Design Correction, not to a full Design Approval Review, unless the deficiency reveals a defect the original review itself should have caught.

### 6.8 Architecture Authoring

**Purpose.** What must always be true?

**Permitted.** Transforming an approved design's decisions into normative architecture — responsibilities, ownership, invariants, guarantees; citing and extending existing architecture without amending it.

**Prohibited.** Implementation detail of any kind (APIs, types, method names, storage technologies, serializer names, pseudocode, implementation-behavior sequence diagrams); reopening the approved design; inventing a design decision the approved Design Exploration output did not itself make.

**Required outputs.** An ARCH document, registered and located per STD-001 §7/§10, satisfying STD-001 §15's Required Core Structure and the structural precedent already established by this project's own ARCH documents (Document Control and Status; Purpose; Scope; Non-Goals; Existing Architectural Context; Design Principles; the capability's own model sections; Ownership; per-component Responsibilities; Invariants; Deferred Decisions; Conformance Requirements; References; Change History; Approval Status — demonstrated by ARCH-002, ARCH-004, ARCH-005, and ARCH-007).

**Entry criteria.** A design has passed Design Approval Review (or Narrow Design Re-Review, if corrected).

**Exit criteria.** The architecture document is internally consistent, faithfully transforms every approved design decision, introduces no new design, and is ready for independent Architecture Review.

### 6.9 Architecture Review

**Purpose.** Does the architecture faithfully express the approved design?

**Permitted.** Independent, skeptical evaluation against the approved design and existing architecture; checking specifically for implementation leakage, ownership ambiguity, invariant contradiction, and architectural drift; findings classified by severity (Critical / Major / Minor / Observation); a recorded verdict.

**Prohibited.** Redesigning the architecture; proposing implementation; reopening the approved design.

**Required outputs.** A recorded verdict with precisely identified findings where change is required. At foundational scale, this stage is realized through a convened, chartered mechanism (demonstrated: GOV-011's Architecture Review Board, and its first session, GOV-012). At single-capability scale, it is realized as an independently conducted, evidence-based review whose disposition is preserved in the architecture document's own Change History (demonstrated: the Architecture Review, Targeted Correction Pass, and Narrow Architecture Re-Review this project performed against ARCH-007). Neither realization is a lesser substitute for the other; each is proportionate to what it reviews.

**Entry criteria.** Architecture Authoring is complete.

**Exit criteria.** A verdict is reached: Approved (ready for an Engineering Work Order), or Architecture Changes Required with findings precisely identified.

### 6.10 Architecture Correction (if required)

**Purpose.** Have the identified architecture findings, and only those findings, been resolved?

**Permitted.** Applying exactly the corrections named by Architecture Review; disclosing precisely what changed and why, including a complete internal-cross-reference validation pass where the review's own findings warrant one.

**Prohibited.** Reopening the approved design; redesigning the architecture beyond the named findings; introducing new architectural concepts, ownership boundaries, or invariants.

**Required outputs.** A corrected ARCH document (a new version), with a Change History entry naming each finding and its exact fix (demonstrated: the ARCH-007 targeted correction pass, v0.1.0 → v0.2.0).

**Entry criteria.** Architecture Review returned a verdict requiring change.

**Exit criteria.** Every named finding has been addressed; every architectural invariant, ownership boundary, and guarantee that was not the subject of a finding remains unchanged in meaning.

### 6.11 Narrow Architecture Re-Review

**Purpose.** Were the identified findings actually resolved, without introducing new drift?

**Permitted.** Re-checking exactly the corrected items, independently re-derived from the current document state rather than trusting the correction's own report; confirming no regression against the architecture's constitutional guarantees, ownership model, or invariants.

**Prohibited.** Reopening the architecture; introducing findings unrelated to the original review; repeating the full Architecture Review.

**Required outputs.** A narrow verdict (demonstrated: the narrow re-review of ARCH-007 v0.2.0, scoped strictly to the five corrections the review identified).

**Entry criteria.** Architecture Correction is complete.

**Exit criteria.** The narrow verdict is reached. The architecture proceeds to an Engineering Work Order, or — only if the correction itself was deficient — returns to Architecture Correction.

### 6.12 Engineering Work Order

**Purpose.** How will the approved architecture be implemented?

**Permitted.** Citing the specific architecture sections that authorize the work (an "Engineering Authority" statement); scoping implementation; defining mandatory validation gates, a Definition of Done, explicit stop/escalation conditions for when an architectural decision is required mid-implementation, and any Bounded Design Decisions the architecture deliberately left open.

**Prohibited.** Redefining architecture; proceeding without an approved, faithfully-cited architectural specification (GOV-004 §1: "Production code MUST NOT be written without an approved architectural specification").

**Required outputs.** An EWO document, registered per STD-001 §46, citing its authorizing architecture and, where one exists, its predecessor EWO (demonstrated by every EWO reviewed in this project's history: 100% cite specific authorizing ARCH sections).

**Entry criteria.** Architecture has passed Architecture Review (or Narrow Architecture Re-Review, if corrected).

**Exit criteria.** The EWO is internally consistent, cites its authorizing architecture precisely, and defines objective, testable completion and validation criteria.

### 6.13 Implementation

**Purpose.** Has the work been implemented?

**Permitted.** Writing code, tests, and documentation strictly within the EWO's stated scope; escalating — stopping and reporting rather than resolving unilaterally — when an architectural decision is required mid-implementation (demonstrated: the stop/escalate clause present in every reviewed EWO).

**Prohibited.** Exceeding EWO scope; making an architecture-level decision without escalation; treating an EWO's own Bounded Design Decisions as license to make an unbounded one.

**Required outputs.** Working code and tests satisfying the EWO's Mandatory Validation and Acceptance Criteria.

**Entry criteria.** An approved EWO exists.

**Exit criteria.** Every EWO requirement is met, or the EWO is returned with disclosed, specific reasons why it cannot be completed as written.

### 6.14 Independent Implementation Review

**Purpose.** Does the implementation faithfully satisfy the Engineering Work Order?

**Permitted.** Independent, skeptical re-verification of test results, architecture compliance, and completion criteria, re-derived from the repository rather than restated from the implementer's own report; a recorded verdict (demonstrated: Approved, Approved with Minor Observations, and Changes Required verdicts recorded across ER-007 through ER-010, including one case — ER-009 — of six successive independent reviews before publication).

**Prohibited.** Redesigning the implementation; expanding EWO scope; substituting for the EWO's own Mandatory Validation gates.

**Required outputs.** A review verdict, recorded within the Engineering Report. Where no separate reviewer is available, this must be disclosed, not silently omitted — demonstrated by ER-010 §10's own disclosure that "no separate human or second-agent reviewer was available for this task," with the report's own validation performed as a genuinely independent re-run in its place, and by ER-001's own explicit distinction between "architectural review performed" (true) and "formal approval evidence recorded" (not true, and not claimed).

**Entry criteria.** Implementation reports completion.

**Exit criteria.** A verdict is reached. Changes Required routes back to Implementation — not to a new Engineering Work Order — unless the required change is itself architecture-level, in which case it routes back to Architecture Review (§6.9).

### 6.15 Engineering Report

**Purpose.** What evidence proves completion?

**Permitted.** Recording exact repository state before and after (branch, HEAD, diff); test results with reconciled counts; an architecture-compliance table checked against the authorizing EWO's own cited sections; disclosed mid-task corrections and how they were resolved; the Independent Implementation Review's verdict.

**Prohibited.** Authorizing, approving, or publishing anything on its own. An Engineering Report is informational only and creates no new requirement (STD-001 §47) — it records that work happened; it does not make that work operative.

**Required outputs.** An ER document, registered per STD-001 §47, citing its authorizing EWO (`reports_on`, never omitted in demonstrated practice).

**Entry criteria.** Independent Implementation Review has returned a verdict permitting completion to be reported.

**Exit criteria.** The ER is complete and evidentially self-consistent — test counts and repository state independently re-derived, not merely restated from a prior claim (demonstrated: ER-007's own arithmetic reconciliation of a 14→13 test-count discrepancy it found in its own predecessor material, disclosed rather than silently corrected).

### 6.16 Publication

**Purpose.** Is the approved artifact now a durable, evidenced part of the repository?

**Permitted.** Staging, committing, and pushing the approved artifact; recording approval evidence through a separate, content-non-mutating evidence commit, per the ADR-0011/ADR-0012 model (STD-001 §31) — verified by exact artifact identity (git blob ID, SHA-256 hash, byte size, line count), independently reproducible by any later party (demonstrated, and independently re-verified during this document's own preparation: the GOV-003 Act 2 Approval Evidence Record, commit `43a49b4`, whose cited hash, blob ID, byte size, and line count all match the current artifact exactly).

**Prohibited.** Mutating the approved artifact's own content to reflect its approval (the artifact stays byte-identical to what was reviewed — this is why an approved document's own internal "Approval Status" table commonly still reads "Not yet approved": that placeholder text is itself part of the approved, byte-identical artifact, and the true disposition lives in the evidence commit, not in the document's own prose); treating staging, committing, or pushing as itself an approval act; force-pushing over or otherwise bypassing verification.

**Required outputs.** A pushed commit, independently hash-verifiable against the evidence record — or, where remote publication is genuinely unavailable, an explicit, disclosed record of that fact (demonstrated throughout this project's history: repeated, disclosed `Permission denied (publickey)` outcomes, with the local, valid commit preserved and no workaround attempted).

**Entry criteria.** The artifact has an effective Approval disposition under GOV-010 §21.

**Exit criteria.** The evidence commit is published to `origin/main` and independently hash-verifiable by any later reviewer.

## 7. Non-Goals

This document does not:

- redesign, re-specify, or evaluate any specific SynapseOS capability;
- define any implementation technique, API, data structure, or technology choice;
- define, register, or amend any STD-001 controlled-document family, identifier convention, or metadata requirement;
- introduce any rule specific to a particular capability (persistence, supervision, temporal execution, or any other);
- redefine any GOV-003 role or GOV-010 decision class, process, or authority assignment;
- claim that every stage in §6 is currently realized as a separately registered, filed document family — several are demonstrated, disclosed, functional practices without a dedicated STD-001 registration (§6.4, §6.5, §6.9–§6.11), and this document does not attempt to create one.

## 8. Review Discipline

### 8.1 Independent review

Every review stage (§6.3, §6.5, §6.7, §6.9, §6.11, §6.14) is conducted independently of the work it reviews wherever an independent reviewer exists, and its independence — or the disclosed absence of one — is stated plainly (GOV-003 §3.4: "must disclose any conflict of interest, including reviewing content they also authored"; GOV-010 §15: AI-generated critique does not, by itself, satisfy an independent human review requirement). A review re-derives its findings from the artifact and the repository directly; it does not trust a prior report's own characterization of what it did (demonstrated throughout: every Architecture Review, Design Approval Review, and Independent Implementation Review in this project's history re-reads the reviewed artifact, re-runs the relevant verification commands, or re-derives the relevant hash — never accepting a predecessor's summary as sufficient).

### 8.2 Evidence-based findings

A finding states what was checked, what was found, and why it matters — never an unsupported assertion. Where a review finds no defect, that outcome is itself reported as the result of applying the method, not assumed as a starting expectation (ACR-001 §3's treatment of its own "zero contradictions found" result).

### 8.3 Finding classifications

Findings are classified by severity, using a consistent, small vocabulary demonstrated across every review stage in this project's history:

- **Critical** — the reviewed artifact is unsound and cannot safely proceed.
- **Major** — correction is required before the artifact may proceed.
- **Minor** — an editorial or narrow clarification is needed; does not block proceeding once scheduled for correction.
- **Observation** — no correction required.

A review concludes with exactly one verdict, stated unambiguously (for example: Approved; Design/Architecture Changes Required; Approved with Minor Observations).

### 8.4 Correction process

A correction pass — Design Correction (§6.6) or Architecture Correction (§6.10) — applies **only** the findings a review identified. This is not a convenience; it is the mechanism that keeps a correction pass auditable and prevents an approved design or architecture from silently drifting under the guise of "fixing" it. A correction pass that finds an additional defect while correcting a named one discloses that additional finding explicitly (as its own, separately identified item) rather than folding it silently into the requested scope — and either corrects it under an explicit, narrow extension of the same pass (as this project's own Architecture Correction of ARCH-007 did, when a requested cross-reference sweep genuinely required checking every citation, not only the ones the review had named, and disclosed doing so) or defers it back to a future review, but never conceals it.

### 8.5 Narrow re-review process

A narrow re-review (§6.7, §6.11) confirms only that the named findings were resolved and that the correction introduced no new drift against the artifact's constitutional properties (its ownership model, invariants, or guarantees). It is not a second full review. Where a narrow re-review's own independent check surfaces something the correction pass should have caught but did not, it is reported as a new finding of the narrow re-review itself — not silently absorbed as though the correction pass had already covered it.

### 8.6 Approval criteria

A review's Approved verdict requires: every finding from the preceding review (if any) resolved; no new finding introduced by the correction; every constitutional or foundational property named in that artifact's own governing precedent unchanged; and no implementation, design, or governance content out of the reviewed stage's own scope. Approval, once reached, is a review-stage verdict — it becomes an *effective, operative* decision only once GOV-010 §5's Approval Authority act occurs and is evidenced per §6.16 above; a review's own "Approved" wording never substitutes for that act.

## 9. Engineering Principles

The following principles are not new; each is already stated or directly implied by GOV-001, GOV-002, GOV-004, or this project's own demonstrated practice. This document collects them here because they are what the lifecycle in §5–§6 exists to enforce mechanically, stage by stage:

1. **Evidence before decisions.** No design, architecture, or implementation decision precedes the Research and Research Review that inform it (§6.2–§6.3; GOV-002 §6.5).
2. **Separation of concerns.** Each stage answers exactly one question and no other (§4).
3. **Single responsibility per document.** A Research document establishes evidence; a Design Exploration document recommends a design; an Architecture document states what must always be true; an Engineering Work Order scopes implementation; an Engineering Report records completion. None substitutes for another.
4. **Traceability.** Every Engineering Work Order cites the architecture that authorizes it; every Engineering Report cites the Engineering Work Order it reports on; every Architecture document cites the design it codifies (GOV-004 §11).
5. **Constitutional architecture is not implementation's to change.** An implementer holds no automatic authority to approve the specifications it implements (GOV-003 §3.6) and is obligated to implement only decisions that are currently effective (GOV-003 §3.6; GOV-010 §21).
6. **Implementation follows approved architecture — never the reverse.** GOV-004 §1's rule is the load-bearing constraint behind the entire stage ordering in §5.
7. **Truthful engineering history.** A correction preserves the original finding and adds a dated correction; it does not silently rewrite or delete what was previously recorded (demonstrated throughout this project's governance and architecture corpus wherever a document has required correction).
8. **Independent verification.** A review re-derives its own findings from the artifact and repository directly, never from a predecessor's characterization alone (§8.1).
9. **Minimal correction scope.** A correction pass resolves exactly what was found, and discloses, rather than silently absorbs, anything further it happens to notice (§8.4).
10. **Publication only after approval — and approval is never inferred from mere existence.** Drafting, staging, committing, or pushing a document does not, by itself, approve it (demonstrated by the status notice present in every governance and architecture document reviewed for this task); approval requires an evidenced act under GOV-010 §5 and §21, represented per STD-001 §31 (§6.16).

## 10. Document Relationships

```text
Governance (GOV-001–GOV-013)
   defines: who may decide, how a decision is processed, and — in this
   document — the stage sequence a capability must pass through
        │
        ▼
Research (RES) ──▶ Research Review (RSS / ACR / AFR, or embedded
   evidence,          verification within Design Approval Review)
   never a
   recommendation
        │
        ▼
Design Exploration (DES-pattern) ──▶ Design Approval Review ──▶
   what should be built                is this correct?
        │
        ▼  (after any required Design Correction / Narrow Re-Review)
Architecture Authoring (ARCH) ──▶ Architecture Review ──▶
   what must always be true            faithful to the design?
        │
        ▼  (after any required Architecture Correction / Narrow Re-Review)
Engineering Work Order (EWO)
   how will the approved architecture be implemented?
        │
        ▼
Implementation ──▶ Independent Implementation Review ──▶
   has it been built?    faithful to the EWO?
        │
        ▼
Engineering Report (ER)
   what evidence proves completion?
        │
        ▼
Publication
   durable, evidenced repository record
```

Governance sits above every other layer without dictating any layer's content (§3). Research and Design inform Architecture but confer no authority of their own — a Research document is never cited as authorizing anything (§6.2's own demonstrated non-recommendation posture); a Design Exploration output authorizes nothing until it has passed Design Approval Review and Architecture Authoring has transformed its decisions into normative form. An Engineering Work Order derives its authority exclusively from the architecture it cites, never from a Research or Design document directly (demonstrated: 0 of the EWOs reviewed for this document cite a RES document as an authorizing source; every one cites specific ARCH sections). An Engineering Report carries no independent authority at all (STD-001 §47) — it is evidence that authorized work occurred, not a further authorization.

## 11. Change Control

This section states when each response below is appropriate; it does not redefine GOV-010's own decision classes or authority assignments, which govern who may make each of these calls.

- **A correction pass (§6.6, §6.10) is sufficient** when a review has identified specific, nameable findings against an otherwise-sound design or architecture, and resolving them does not require changing any decision the review did not itself flag.
- **A narrow re-review (§6.7, §6.11) is sufficient** to confirm a correction pass, and never substitutes for a full review of anything the correction pass did not touch.
- **Design or Architecture must be reopened — a new Research, Design Exploration, or Architecture Authoring cycle, not a correction pass — when:** a review finds a Critical defect indicating the underlying approach is unsound; a correction pass would require changing a decision the governing review never evaluated; or new evidence emerges that contradicts a foundational assumption the existing design or architecture depends on (GOV-010 §19's own decision-review triggers apply identically here: an assumption becomes false, a material cost changes, or a new standard becomes relevant).
- **A new engineering cycle (returning to Idea, §6.1) is required** when the capability's own governing question — not merely one design or architecture decision within it — turns out to be wrong, or when a materially different capability is actually needed.
- **Amending an already-approved document of any kind** — governance, architecture, or otherwise — requires the same evidenced Approval act any first approval requires (GOV-010 §21; §6.16 above), regardless of how minor the amendment; a PATCH-level editorial fix and a MAJOR-level substantive change differ in review depth (STD-001 §13), never in whether an evidenced approval act is required at all.
- **Which decision class governs a given change** is determined by GOV-010 §4, not by this document — a change to architecture is ordinarily Class B; a change confined to implementation choices within already-approved architecture is ordinarily Class E. This document does not reclassify any decision.

## 12. Conformance

Compliance with this document's mandatory lifecycle (§5–§6) is required for every future major SynapseOS capability, from the point this document becomes effective (GOV-010 §21) forward. This document does not retroactively reclassify, invalidate, or require rework of any capability whose engineering already occurred before this document's own effective date — consistent with GOV-010 §21's own prospective-application principle for newly effective process documents. Where a future capability's own scale genuinely does not warrant a given stage's full formal weight (for example, a documentation-only correction requiring no Engineering Report, as EWO-010 itself demonstrates), that proportionality is itself part of conformance, not an exception to it — provided it is disclosed, exactly as EWO-010 discloses it, rather than silently assumed.

## 13. Open Questions

13.1. **Whether "Design Exploration," "Design Approval Review," "Architecture Review," "Architecture/Design Correction," "Narrow Re-Review," and "Independent Implementation Review" should ever become registered STD-001 controlled-document families**, with their own identifier series, rather than remaining functionally demonstrated practices realized within existing families' own conventions (§6.4, §6.5, §6.9–§6.11). This document takes no position and makes no recommendation — that decision, if ever wanted, belongs to a future, separately authorized STD-001 amendment, evaluated on its own evidence.

13.2. **How the Architecture Review Board (GOV-011) and the single-capability Architecture Review pattern demonstrated for ARCH-007 relate at scale** — whether every future capability's Architecture Review should be routed through the Board, or whether the lighter, single-capability pattern remains appropriate for narrower capabilities. Not resolved here; both are cited in §6.9 as legitimate, already-demonstrated realizations of the same stage.

13.3. **Whether a capability may legitimately skip Research Review entirely** (as opposed to realizing it inline within Design Approval Review, §6.5) when the underlying Research itself was extremely narrow. Not resolved here; no such instance has yet been demonstrated in this project's history, so this document does not assert an answer either way.

## 14. References

Internal:

- GOV-001 — Project Charter
- GOV-002 — Vision and Mission
- GOV-003 — Governance Model
- GOV-004 — Engineering Principles
- GOV-010 — Decision Framework
- GOV-011 — Architecture Review Board Charter
- GOV-012 — Architecture Review Board Session 001
- STD-001 — Documentation Standards (§7, §10, §13, §31, §46, §47, §50, §51, §52)
- ADR-0011 — Bootstrap Approval Authority
- ADR-0012 — Corrective Founder Approval Evidence Record basis

Source evidence (read in full or independently verified during this document's preparation, not restated from memory):

- `research/RES-001-Comparative-Execution-Model-Analysis.md` through `RES-006` (Research stage)
- `consolidation/RSS-001-Research-Synthesis-Review.md`, `ACR-001-Architecture-Consolidation-Review.md`, `AFR-001-Architecture-Freeze-Review.md` (Research Review stage, multi-document scale)
- `consolidation/DES-001-Persistent-Actor-Design-Exploration.md` (Design Exploration / Design Correction stage)
- `architecture/ARCH-001-Constitutional-Architecture.md`, `ARCH-004-Local-Actor-Supervision-Architecture.md`, `ARCH-005-Temporal-Runtime-Architecture.md`, `ARCH-007-Persistent-Actor-Architecture.md` (Architecture Authoring / Architecture Review / Architecture Correction / Narrow Architecture Re-Review stages)
- `work-orders/EWO-001, EWO-002, EWO-007, EWO-009, EWO-010, EWO-011` (Engineering Work Order stage, including two non-standard historical-reconstruction instances, EWO-009 and EWO-010)
- `engineering-reports/ER-001, ER-007, ER-008, ER-009, ER-010` (Independent Implementation Review and Engineering Report stages)
- `governance/GOV-001_Project_Charter_v0.1.docx`, `GOV-002-Vision-and-Mission.md`, `GOV-004-Engineering-Principles.md`, `GOV-010-Decision-Framework.md` (governance context, §3)
- `governance/GOV-003-Governance-Model.md` and git commit `43a49b4` (independently re-verified during this document's preparation: SHA-256, git blob ID, byte size, and line count all match the current tracked artifact exactly, confirming genuine Act 2 approval and the Publication evidence model cited in §6.16)

## 15. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-26 | Denver Jacobs (AI-assisted) | Initial Draft. Documents the mandatory Engineering Lifecycle demonstrated across this project's governance, research, design, architecture, and engineering history: sixteen stages from Idea through Publication, each stated as the single question it answers, its permitted and prohibited activities, required outputs, and entry/exit criteria. Filed as GOV-013 rather than GOV-003 after repository verification found GOV-003 already exists, is a different, unrelated document (Governance Model), and is genuinely Approved (independently hash-verified against commit `43a49b4` during this document's own preparation) — disclosed to, and confirmed by, the document's requester before filing. Documents two forms of Research/Research Review realization (multi-document RES/RSS/ACR/AFR at foundational scale; embedded, single-document comparative analysis at single-capability scale) and discloses that several stages (Design Exploration, Design Approval Review, Architecture Review, Architecture/Design Correction, Narrow Re-Review) are demonstrated, functional practices not currently registered as their own STD-001 document families, rather than presenting a uniform formality the evidence does not support. No architecture, implementation, capability-specific rule, or STD-001 amendment is introduced. |

## 16. Approval Status

### Document State

| Field | Value |
|-------|-------|
| Lifecycle status | Draft |
| Normal-governance disposition | Not yet approved |
| Disposition date | Not yet assigned |
| Approving actor | Not yet assigned |

### Immutable Approval Evidence

| Field | Value |
|-------|-------|
| Document ID | GOV-013 |
| Repository path | governance/GOV-013-Engineering-Lifecycle.md |
| Version | 0.1.0 |
| Artifact commit | Not yet created |
| Git blob ID | Not yet created |
| SHA-256 | Not yet created |
| Approver | Not yet assigned |
| Approver capacity | Not yet assigned |
| Approval-authority source | Not yet assigned (Governance-tier normal-governance disposition per GOV-003 §3.1 and §5, on the same basis as GOV-002, GOV-004, and GOV-010) |
| Approval type | Not yet assigned |
| Review evidence | Not yet created |
| Independent-review status | Not yet created |
| Self-approval / conflict-of-interest disclosure | Not yet created |
| Rationale | Not yet created |
| Known limitations | Not yet created |
| Unresolved issues | Not yet created |
| Disposition | Not yet approved |
| Effective date | Not yet assigned |
| Evidence commit | Not yet created |
| Evidence publication state | Not yet published |
| Relevant evidence | Not yet created |

No field in this section may be populated until the corresponding act has genuinely occurred and is evidenced per ADR-0011 §14 and ADR-0012 §9. This table does not, and must not be read to, claim that GOV-013 has been approved.
