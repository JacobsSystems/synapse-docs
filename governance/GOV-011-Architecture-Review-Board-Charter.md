---
document_id: GOV-011
title: Architecture Review Board Charter
version: 0.1.1
status: Draft
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.1 and §5 — GOV-011 is a Governance-tier document establishing a new permanent governance body, not a Class B (Architectural) content decision; it is therefore approved at the Governance tier on the same basis as GOV-003, GOV-010, and GOV-004, not delegated to the Chief Architect. Individual ARB decisions made once this charter is operative are ratified as ordinary Class B dispositions per GOV-010 §5 (Chief Architect, currently vacant — interim Founder default per GOV-003 §3.2). See §3 for the full reasoning.
created: 2026-07-17
last_updated: 2026-07-17
classification: Public
related_documents:
  governance:
    - GOV-003 (v0.1.0 — governance/GOV-003-Governance-Model.md; Operative — Act 2 Approved, content-hash-verified §3.1 below)
    - GOV-004 (v0.1.0 — governance/GOV-004-Engineering-Principles.md; Approved — normal-governance validation disposition, content-hash-verified §3.1 below)
    - GOV-010 (v0.1.0 — governance/GOV-010-Decision-Framework.md; Operative — Act 2 Approved, content-hash-verified §3.1 below)
  standards:
    - STD-001 (v0.4.0 — standards/STD-001-Documentation-Standards.md; Draft)
  architecture:
    - ARCH-000 (v0.1.0, Draft — architecture/ARCH-000-Introduction.md)
    - ARCH-001 (v0.2.0, Draft — architecture/ARCH-001-Constitutional-Architecture.md)
    - ARCH-002 (v0.2.1, Draft — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.5.0, Draft — architecture/ARCH-003-Runtime-Integration-Architecture.md)
    - ARCH-004 (v0.1.0, Draft — architecture/ARCH-004-Local-Actor-Supervision-Architecture.md)
    - ARCH-005 (v0.1.0, Draft — architecture/ARCH-005-Temporal-Runtime-Architecture.md)
    - ARCH-006 (v0.1.4, Draft — architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md)
  adrs:
    - ADR-0011 (v0.1.0, Draft — Act 1 effective — adrs/ADR-0011-Bootstrap-Approval-Authority.md)
    - ADR-0012 (v0.1.0, Draft — corrective Founder approval effective — adrs/ADR-0012-Content-Non-Mutating-Act-2-Approval-Evidence.md)
    - ADR-0013 (Draft — adrs/ADR-0013-Architectural-Evolution-of-SynapseOS.md)
    - ADR-0014 (Draft — adrs/ADR-0014-Engineering-Standards-Corpus-Reconciliation.md)
    - ADR-0015 (Draft — adrs/ADR-0015-Audit-Emitter-Failure-Semantics.md)
    - ADR-0016 (Draft — adrs/ADR-0016-Trusted-Core-Interaction-Model.md)
    - ADR-0017 (Draft — adrs/ADR-0017-Bootstrap-Capability-Trust-Root.md)
  consolidation:
    - RSS-001 (v0.1.2, Draft — consolidation/RSS-001-Research-Synthesis-Review.md)
    - ACR-001 (v0.2.0, Draft — consolidation/ACR-001-Architecture-Consolidation-Review.md)
    - AFR-001 (not yet authored — consolidation/AFR-001-Architecture-Freeze-Review.md; the ARB this charter establishes is the intended input mechanism for AFR-001's own future recommendation)
  research: None — the ARB consumes RES/RSS documents as evidentiary input (§6, §10); it does not author research.
  engineering: None — GOV-011 requires no authorizing Engineering Work Order. Establishing a governance body is a Governance-tier act under GOV-003 §5, not an engineering task; no EWO citation of any kind is used or required here.
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# GOV-011 — Architecture Review Board Charter

> **Status notice:** This document is **Draft**. No GOV-011-specific approval act has occurred. Drafting, saving, staging, committing, or pushing this charter does not itself constitute approval, on the same basis STD-001 §12 and ADR-0011 §7 already establish for every other document in this corpus. See §18 (Approval Status).

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 §33 (AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Purpose

This document establishes the **Architecture Review Board (ARB)** as SynapseOS's permanent architectural governance body, and defines the constitutional rules under which every future ARB — in Act 2, and in every act that follows it — shall operate.

The ARB exists to provide a formal, evidence-based decision process for architectural governance: to ensure that architectural evolution remains evidence-based, research-informed, internally consistent, historically truthful, and governed through documented decisions rather than implementation convenience.

The ARB is not an engineering team. It does not write runtime code. It does not author research. It does not rewrite architecture. Its responsibility is to evaluate architectural evidence already produced by the research, synthesis, and consolidation process (RES, RSS, ACR) and by proposed architectural or ADR changes, and to reach a governance disposition on that evidence — a disposition that, like every other disposition under GOV-010, becomes operative only through the approval-authority framework GOV-003 and GOV-010 already establish (§4, §16).

This charter itself creates no architecture, amends no architecture, and decides no architectural question. It establishes only the body, process, and rules by which such questions will be decided in the future.

## 2. Scope

This charter establishes: the ARB's authority (§4); the ARB's authority limitations (§5); the evidentiary hierarchy the ARB must apply (§6); the constitutional principles that bind every ARB decision (§7); the fixed decision classifications available to the ARB (§8); the ARB's decision process (§9); the minimum content of an ARB decision record (§10); the ARB's typical inputs (§11) and outputs (§12); the standing principles the ARB must observe (§13); the ARB's composition and meeting process (§14–§15); and this charter's future applicability (§16).

This charter does **not** conduct an Architecture Review Board meeting, does not evaluate any specific architectural claim, does not dispose of ACR-001 or any other document, and does not itself certify, approve, or freeze any architecture. The first substantive ARB session — whenever convened, under whatever future authorization commissions it — is a separate act, governed by this charter but not performed by it.

## 3. Relationship to Existing Governance

### 3.1 Governance basis, verified directly against repository evidence

This charter is authored against a governance foundation that this task independently re-verified, rather than assumed, before drafting began:

- **ADR-0011 Act 1** (Founding Act) is evidenced by a pushed, content-non-mutating commit (`f626499`, 2026-07-10) that this task confirmed is an ancestor of the current `origin/main` HEAD.
- **GOV-003 Act 2 Approval** is evidenced by a pushed commit (`43a49b4`, 2026-07-11, Disposition: Approved) confirmed as an ancestor of HEAD. This task independently recomputed the SHA-256 of the current, tracked `governance/GOV-003-Governance-Model.md` and confirmed it matches the fingerprint recorded in that evidence commit **exactly** — the currently operative GOV-003 content is the same content that was approved; nothing has drifted since.
- **GOV-010 Act 2 Approval** is evidenced by a pushed commit (`11cab8d`, 2026-07-11, Disposition: Approved), an ancestor of HEAD, with an independently recomputed SHA-256 match against the current tracked `governance/GOV-010-Decision-Framework.md`, confirmed exact.
- **GOV-004 normal-governance validation** is evidenced by a pushed commit (`1c1aaef`, 2026-07-11, Disposition: Approved, self-identified as "the Validation Act defined by ADR-0011 §18-19"), an ancestor of HEAD, with an independently recomputed SHA-256 match against the current tracked `governance/GOV-004-Engineering-Principles.md`, confirmed exact.

Applying ADR-0011 §13's own objective, evidence-based derivation table to this verified evidence — both GOV-003 and GOV-010 hold valid Act 2 approval, and a disposition of GOV-004 self-identified as the §18–§19 validation act has been recorded — the mechanical result is Bootstrap Authority State: **`Terminated`**, meaning normal governance under GOV-003/GOV-010 is now the sole approval path for every document in this repository, this charter included.

**A disclosed, unresolved inconsistency, not silently corrected.** The GOV-004 evidence commit's own text states, in its final line, "This act does not reactivate, further suspend, or terminate Bootstrap Authority State" — which appears to conflict with ADR-0011 §12.6 and §19's own non-discretionary rule that a first qualifying validation disposition *does* terminate Bootstrap Authority State automatically. This charter does not adjudicate that inconsistency: ADR-0011 §21 distinguishes predicate satisfaction from evidence recording, and resolving which of the two texts controls is a question for the governance-reconciliation process itself (an ARB, once convened under this charter, or an equivalent future task), not something GOV-011 may silently decide by treating one reading as authoritative. This charter proceeds on the basis that GOV-003 and GOV-010 are, at minimum, Operative under Act 2 (a conclusion both evidence commits agree on without qualification), which is sufficient governance foundation for this charter's own authorship; it records the Termination question as open rather than asserting it as settled fact.

**Tracked status fields remain unchanged by design.** GOV-003, GOV-010, and GOV-004 all still carry `status: Draft` in their own frontmatter. This is not evidence against their operative state — ADR-0012 §13 explicitly establishes that approval evidence is recorded in separate, content-non-mutating commits precisely so that a document's tracked bytes, including its `status:` field, are never required to change to reflect approval. A reader of this charter should not treat any document's frozen `status: Draft` field, alone, as proof that it lacks operative governance effect; the evidence commits verified above are the authoritative record.

### 3.2 Why GOV-011 is a Governance-tier, not Class B, decision

GOV-003 §5 states the Document Hierarchy as: Governance → Standards → Architecture → ADRs → RFCs → Implementation → Testing → Release. Establishing a permanent governance body is not, on its face, a Class B (Architectural) decision under GOV-010 §4 ("system boundaries, core protocols, persistence models and platform-wide technical choices") — it does not choose a system boundary, protocol, or technical design; it creates a new governance mechanism, a matter GOV-003 §5 places at the top of the hierarchy, not the third tier. Nor does it map cleanly onto GOV-010 §4's Class A ("mission, product direction, funding, licensing and major partnerships"). Rather than force this charter into a decision class that does not describe it, this charter is presented plainly as a Governance-tier document, approved on the same basis GOV-003, GOV-010, and GOV-004 themselves already were: Founder approval, per GOV-003 §3.1's retained final authority and §5's hierarchy — now exercised as an ordinary normal-governance act, since Bootstrap Authority State is, at minimum, no longer `Not Established` (§3.1), and Act 2 in any case never had authority over "any future GOV document" (ADR-0011 §8's exhaustive exclusion list).

Once this charter is itself approved and operative, the *individual decisions the ARB subsequently makes* — accepting or amending architecture, requiring a new ADR, commissioning research — are a different matter: those are genuine Class B (Architectural) content decisions, and this charter requires (§9) that they be ratified as such, by the Class B Approval Authority GOV-010 §5 already names (Chief Architect, vacant, interim Founder per GOV-003 §3.2). The ARB itself is not made the Class B Approval Authority by this charter; it is the evidence-based process that feeds that authority's decision, exactly as ACR-001 and AFR-001 already do for the narrower reviews STD-001 §51–§52 define.

### 3.3 Closing a previously identified gap

ACR-001 §17 (Non-Modification Statement) and its own "ARB consideration" fields throughout recorded, explicitly, that "no such body is currently defined in GOV-003, GOV-004, GOV-010, or STD-001," and used the term only prospectively. This charter is the act that closes that gap. Every "ARB consideration" question ACR-001 raised remains exactly as ACR-001 left it — a question, not an answer — and this charter does not resolve any of them; it only establishes the body now positioned to take them up in a future session.

## 4. Authority

The ARB may:

1. **Accept architecture** — record that a specific architectural claim, document, or set of claims is supported by the evidence presented and requires no further action.
2. **Require clarification** — direct that specific language, a specific citation, or a specific ambiguity in an architecture document, ADR, or review be clarified, without requiring a substantive change of position.
3. **Require architecture amendments** — direct that a specific architecture document be amended, identifying the specific claim requiring amendment and the evidentiary basis for that requirement. Consistent with STD-001 §51 (governing ACR, the review type an ARB session most often evaluates), this requirement is a recommendation the ARB records; the amendment itself still requires its own ADR and passage through ARCH-001 §10 (Change Control) before it takes effect (§5.7).
4. **Require new ADRs** — direct that a specific decision be recorded through a new ADR rather than resolved informally.
5. **Commission additional research** — direct that a specific, named evidentiary gap be closed through a new RES document or an extension of an existing one, before a claim can be finally classified.
6. **Classify future work** — determine that a specific topic is out of scope for the architecture currently under review, and record it as Deferred or Future Work rather than leaving its status ambiguous.
7. **Recommend architecture certification** — record a recommendation that a specific architecture, or the constitutional architecture as a whole, is ready for an Architecture Freeze Review disposition (AFR, per STD-001 §52). This is a recommendation only; it is not itself a certification (§5.8).

## 5. Authority Limitations

The ARB shall not:

1. **Modify runtime code** — the ARB has no access to, and no authority over, the `synapse-runtime` repository or any other source-code repository. Every finding in this session's own governing task explicitly restricted `synapse-runtime` to read-only reference material, and this charter preserves that restriction permanently, not merely for this session.
2. **Fabricate evidence** — an ARB decision record (§10) may cite only evidence that genuinely exists in the repository at the time of the decision. An ARB member (§14) who cannot verify a citation directly must say so, not assume it.
3. **Rewrite research** — the ARB may commission additional research (§4.5) but may not itself alter the findings, evidence, or conclusions of any RES document. RES documents are informational and investigative only (STD-001 §5) and are amended, if ever, only through their own authoring process.
4. **Ignore RSS** — an ARB decision that departs from RSS-001's (or a future RSS's) own synthesized evidence without citing a specific, identified reason is invalid under this charter's evidence hierarchy (§6) and constitutional principles (§7.2).
5. **Ignore ACR** — an ARB decision that departs from an already-completed ACR's own findings (for example, ACR-001's classifications, its Gap and Inconsistency Register, or its Architecture Claim Register) without citing a specific, identified reason is likewise invalid under §6 and §7.2.
6. **Bypass ADRs** — the ARB may require a new ADR (§4.4) but may not itself substitute its own decision record for the ADR process ARCH-001 §10 and STD-001 §5–§6 already require for an architecture amendment. An ARB "Architecture Amendment Required" decision (§8.3) creates an obligation to open an ADR; it does not itself amend anything.
7. **Bypass standards** — the ARB may not waive, override, or grant an exception to any STD-001 requirement, including the identifier, versioning, evidence-representation, and approval-evidence rules this charter itself is written under.
8. **Certify architecture directly** — the ARB may recommend certification (§4.7), but certification of the constitutional architecture as ready for implementation is an Architecture Freeze Review act, reserved to AFR under STD-001 §52, which itself states "An AFR does not itself possess approval authority" (STD-001's Appendix B Document Family Register separately summarizes this as an AFR "holds no independent approval authority") and becomes effective only through the GOV-003/GOV-010 approval-authority framework. The ARB does not perform, replace, or pre-empt an AFR.
9. **Publish engineering work** — the ARB does not author, authorize, or publish an EWO, ER, EMO, or EMR. Engineering authorization remains entirely outside this charter's scope, exactly as it is outside ACR's and RSS's (STD-001 §5–§6).

## 6. Evidence Hierarchy

When evaluating a specific architectural claim, the ARB weighs evidence in the following order:

1. **Standards** (STD-001 and any future STD-series document) — the mandatory rules governing how every other document in this hierarchy must be identified, versioned, evidenced, and represented. A claim that cannot be evaluated without violating a Standards requirement (for example, an unverifiable citation, or an invented identifier) fails on that basis before its substance is even reached.
2. **Governance** (GOV-series documents) — the rules defining who may decide what, and under what authority; governs the ARB's own process (§9–§10) and the effect of its decisions (§4, §5.8).
3. **Research** (RES-series documents) — the primary comparative evidence: what other systems actually do, and how they are documented to work.
4. **Research synthesis** (RSS-series documents) — the consolidated, cross-referenced synthesis of the RES corpus. Per §5.4, the ARB does not depart from an operative RSS's synthesis without a specifically identified reason.
5. **Architecture** (ARCH-series documents and this repository's operative ADRs, read together) — the claims under evaluation. Architecture is the object the ARB reviews, never itself evidence for its own correctness.
6. **Architecture reviews** (ACR-series documents) — the analytical evaluation of architecture against RSS evidence. Per §5.5, the ARB does not depart from an operative ACR's classifications without a specifically identified reason.
7. **ADRs** — the specific decision record for a narrower architectural question; read as evidence of what was decided and why, not as evidence that the decision was correct.
8. **Engineering reports** (ER-series documents, and EWO-series documents as context for them) — evidence of what was actually built and how, informative for understanding current implementation but never evidence that an implementation choice is architecturally correct, per the same rule ACR-001 §3 already applies: implementation conformance establishes only that a system conforms to its own architecture, never that the architecture itself is right.
9. **Runtime implementation** (the `synapse-runtime` repository directly) — the lowest-weight source in this hierarchy for the purpose of *architectural* evaluation. The ARB may consult it to confirm a factual claim about current behavior, but a runtime implementation detail is never, by itself, a justification for accepting or amending architecture (§5.1, §7.3).

**This hierarchy is distinct from, and does not override, GOV-003 §5's Document Hierarchy** (Governance → Standards → Architecture → ADRs → RFCs → Implementation → Testing → Release), which governs document *authority* — which document controls when two conflict — across the entire repository. The ordering above governs *evidentiary weight* specifically for the ARB's own evaluative task: which source the ARB should trust most when assessing whether a specific architectural claim is supported. The two orderings answer different questions and are not in tension: GOV-003 §5 determines what happens when two documents' instructions conflict; §6 above determines how much weight the ARB gives a source when weighing evidence for a single architectural question. Where a genuine conflict between the two orderings would matter to a specific decision, GOV-003 §5's authority ordering controls, because this charter is itself subordinate to GOV-003 (§3.2).

## 7. Constitutional Principles

Every ARB decision is bound by the following principles:

1. **Repository truth overrides convenience.** A decision that is easier to reach, faster to record, or more favorable to a prior conclusion is never preferred over one the evidence actually supports.
2. **Evidence overrides opinion.** An ARB member's technical judgment informs how evidence is read; it does not substitute for evidence where none exists (§5.2, §8.3–§8.5).
3. **Architecture overrides implementation.** Where architecture and current runtime implementation genuinely diverge, the ARB evaluates whether the *architecture* is correct — it does not treat the existence of a working implementation as proof that the architecture it happens to follow was the right choice (§6.9, consistent with ACR-001's own governing method).
4. **Research informs architecture.** Architecture is expected to be evaluable against RES/RSS evidence wherever that evidence exists; where it does not yet exist (as ACR-001 §7.12 and §7.15 found for supervision and temporal-runtime architecture specifically), the ARB records that honestly as Outside Research Scope rather than assuming either support or contradiction.
5. **Provenance shall be preserved.** The ARB does not invent replacement citations, authorizing work orders, or approval history where none exists — the same rule already applied twice in this corpus's history to the EWO-007 citation in RSS-001 and ACR-001, and the same rule this charter itself applies in declining to invent a resolution to the Bootstrap Authority State inconsistency (§3.1).
6. **Architectural history shall remain truthful.** A decision record (§10) documents what was actually found and decided, including disagreement, uncertainty, and unresolved questions — it is not edited after the fact to appear more conclusive than the original evaluation was.
7. **Future work shall not be represented as implemented capability.** A "Deferred to Future Scope" decision (§8.6) records that something is not yet built or not yet evaluated; it must never be phrased in a way a future reader could mistake for a claim that it already exists.
8. **Architectural certification requires completed governance.** The ARB cannot recommend certification (§4.7) for architecture whose own governing evidence — an operative RSS, and, where one exists, an operative ACR — has not itself been completed and verified current. A stale citation (for example, to a superseded RSS-001 version) invalidates a certification recommendation built on it.

## 8. Decision Classifications

Every finding the ARB evaluates receives exactly one of the following seven decisions:

1. **Accepted.** The claim is supported by the evidence presented (§6), no contradiction was identified, and no further action is required. This is the ARB equivalent of ACR-001's own "Supported" classification, applied at the level of a governance decision rather than an evidentiary classification.
2. **Accepted with Clarification.** The claim is substantively supported, but a specific textual, citation, or definitional ambiguity must be corrected for the claim to be unambiguously understood by a future reader. The required clarification must be stated precisely enough that whoever performs it does not need to re-derive what was meant.
3. **Architecture Amendment Required.** The evidence does not support the claim as currently written, and a change to the architecture itself — not merely its wording — is required. This decision creates an obligation to open a new ADR (§4.4, §5.6); it does not itself amend anything.
4. **New ADR Required.** The claim concerns a decision that has never been formally recorded, or that has been recorded informally (in an architecture document's prose, for instance) without the traceability an ADR provides. Distinct from Architecture Amendment Required: this classification does not necessarily mean the existing position is wrong, only that it has not yet been properly recorded as a decision.
5. **Additional Research Required.** The claim cannot presently be classified Supported, Partially Supported, Unsupported, or Contradicted (the ACR-001 evidentiary classification model this charter's decisions build on) because no adequate comparative evidence exists yet. This mirrors ACR-001's own "Outside Research Scope" finding for ARCH-004's supervision-policy design (ACR-001 §7.12, GAP-003) and is the correct classification for a similar future finding, not "Rejected."
6. **Deferred to Future Scope.** The claim concerns a topic the architecture, by its own text, has not yet addressed and does not need to address for the review currently underway. This mirrors ACR-001's own "Deferred Matters" (§14) and must cite the specific architecture section that defers the topic, where one exists.
7. **Rejected.** The evidence directly contradicts the claim, or the claim conflicts with an already-established constitutional law (ARCH-001 §6) or an already-approved governance rule, and no amendment path is being pursued. This is the ARB's strongest, least common disposition and requires the clearest evidentiary citation of the seven.

## 9. Decision Process

For every issue the ARB evaluates, the following eight steps apply, specializing GOV-010 §7's general eleven-step Decision Workflow for the ARB's own evidence-based, architecture-specific evaluative task:

1. **Evidence presented.** The specific architectural claim, its source document and section, and the evidence cited for or against it (per the hierarchy in §6) are recorded before discussion begins. This corresponds to GOV-010 §7 steps 1–3 (define the decision question, identify the decision owner, gather relevant evidence).
2. **Evidence evaluated.** Each cited source is checked directly — not merely cited from a prior document's own restatement of it — for whether it actually supports the claim as presented. Where this charter's own governing task history demonstrates the value of this step (ACR-001's independent re-verification of the ADR-0015/16/17 approval-status citations, and this charter's own independent SHA-256 verification of GOV-003/GOV-010/GOV-004 in §3.1), the ARB is expected to repeat it, not skip it because a prior document already made the same claim.
3. **Contradictory evidence considered.** Any evidence weighing against the claim, including evidence the claim's own proponent did not raise, is actively sought and recorded — never merely evidence that happens to already support a preferred outcome. This is the ARB's application of the adversarial, falsification-oriented method ACR-001's own governing task required and this charter's §7.1–§7.2 make a standing rule rather than a one-time instruction.
4. **Board reasoning.** The reasoning connecting the evidence (steps 1–3) to the eventual decision (step 5) is recorded in enough detail that a future reader can follow it without re-deriving it. Corresponds to GOV-010 §7 steps 4–7 (assumptions, constraints, alternatives, evaluation).
5. **Decision.** Exactly one of the seven classifications in §8 is recorded.
6. **Rationale.** The specific reason the chosen classification, rather than any of the other six, was selected — this is distinct from step 4's reasoning about the evidence itself, and addresses specifically why this decision and not an adjacent one.
7. **Required action.** The concrete next step the decision creates (opening an ADR, commissioning research, correcting a citation, or none, for Accepted) is stated precisely enough to be actioned without further interpretation.
8. **Closure criteria.** The specific, checkable condition under which this decision is considered resolved and does not require re-evaluation is stated explicitly — for example, "resolved when ADR-00NN reaches Approved status," or "resolved when RES-007 is published and the claim is re-evaluated against it."

This process corresponds to, and does not replace, GOV-010 §7's general decision workflow and §9's Evidence Standard; it is the ARB's own specialization of both for architectural evidence review specifically, on the same basis ACR-001 §5–§6 already specialized a review method from the same general governance framework.

## 10. Decision Records

Every ARB decision must be recorded with, at minimum, the following fields — an ARB-specific extension of GOV-010 §8's Required Decision Record and Appendix A Decision Record Template, adding the fields an evidence-based architecture review specifically requires:

1. **Issue identifier** — a stable, unique reference for this specific finding (for example, following the `ACR-001-CLM-NNN` convention ACR-001 §12 already established, or a new `ARB-NNN` series this charter does not itself mandate but does not preclude a future ARB session from adopting).
2. **Source document** — the exact document, version, and section the claim originates from.
3. **Evidence** — every source cited in §9 steps 1–3, with exact section references, not general document-level citations.
4. **Discussion summary** — the substance of §9 step 4's reasoning, condensed to what a future reader needs.
5. **Decision** — exactly one of the seven classifications in §8.
6. **Rationale** — §9 step 6.
7. **Required action** — §9 step 7.
8. **Blocking status** — whether this finding blocks a specific downstream act (for example, an AFR certification recommendation, mirroring ACR-001's own Gap and Inconsistency Register "Blocks AFR-001?" column) or does not.
9. **Follow-up work** — any commissioned research, required ADR, or other concrete work item this decision creates, distinct from Required Action's immediate next step where the two differ (for example, "Required action: open ADR-00NN" versus "Follow-up work: track ADR-00NN through GOV-010 §5 disposition").
10. **Closure condition** — §9 step 8.

## 11. Inputs

The ARB's typical inputs include, without limitation:

- **RSS documents** (Research Synthesis Reviews, STD-001 §50) — the primary consolidated evidence base.
- **ACR documents** (Architecture Consolidation Reviews, STD-001 §51) — analytical findings the ARB evaluates and, where warranted, acts on.
- **ADR proposals** — new or amended architectural decisions requiring disposition.
- **Architecture amendments** — proposed changes to an operative or Draft ARCH-series document.
- **Architecture reviews** — findings from any completed review of architecture against evidence, whether or not formally an ACR.
- **Significant engineering findings** — an ER or EMR that surfaces a material architectural question during implementation (for example, ARCH-004 §6's own disclosed Scheduler/Lifecycle-Guardian gap — a permanently-failing actor's second queued message is silently, permanently discarded with no retry, dead-letter, or halt — found during implementation and already recorded as an architecture-relevant finding, of exactly the kind a future ARB session would take up).

## 12. Outputs

The ARB's typical outputs include, without limitation:

- **Accepted findings** — claims classified Accepted or Accepted with Clarification (§8.1–§8.2).
- **Architecture amendments** — a recorded requirement (§8.3) that a specific ADR be opened; the amendment itself is produced by the ADR process this requirement triggers, not by the ARB directly.
- **ADR requests** — a recorded requirement (§8.4) that a decision be formally captured.
- **Research requests** — a recorded requirement (§8.5) that a specific evidentiary gap be closed.
- **Certification recommendations** — a recommendation (§4.7) forwarded to a future AFR, never a certification in its own right (§5.8).
- **Future work classifications** — claims classified Deferred to Future Scope (§8.6), each citing the specific architecture text that already defers the topic where one exists.

## 13. Board Principles

The ARB shall:

1. **Preserve architectural integrity** — evaluate whether a proposed change is consistent with ARCH-001's constitutional concepts and laws (§6) before evaluating whether it is otherwise well designed.
2. **Avoid unnecessary architectural change** — per ARCH-001 §10's own Change Control rule, the constitutional architecture is frozen except through demonstrated contradiction or significant improvement, never through novelty alone; the ARB applies this same standard to every amendment it considers.
3. **Avoid governance drift** — an ARB decision does not create governance authority beyond what §4–§5 grant it, on the same basis ADR-0011 §23 already requires of its own bootstrap mechanism: "not reusable by analogy to any other situation."
4. **Minimise architectural complexity** — where two evidence-supported options would satisfy a requirement, the ARB prefers the option that adds the least new mechanism, consistent with ARCH-001 §9's mechanism/policy separation test and the Minimal Runtime Profile discipline ARCH-002 §21 already establishes.
5. **Prefer evidence over intuition** — §7.2, restated here as a standing operational discipline, not merely a principle invoked when convenient.
6. **Document every significant decision** — an ARB evaluation that does not produce a decision record (§10) has no governance effect and creates no obligation on anyone, exactly as GOV-010 §21 already establishes for any undocumented disposition.

## 14. Composition

This charter does not invent ARB membership beyond what current, verified governance already establishes. As of this charter's authorship:

- The **Class B Approval Authority** for any architectural decision the ARB's findings feed into is the Chief Architect (GOV-003 §3.2), currently vacant, defaulting on an interim basis to the Founder, Denver Jacobs, under the same conflict-of-interest and self-approval disclosure requirements GOV-003 §3.5 already imposes on any holder of that role.
- **No independent reviewer currently exists in this repository.** This is not new to the ARB — it is the same limitation ADR-0011 §16, GOV-003 §3.4, and every Approval Status table in this corpus already disclose. An ARB session convened before an independent reviewer is appointed is, necessarily, a disclosed self-review process: the same person may author the material under review, evaluate it, and hold the Class B authority that ratifies the outcome. This charter does not conceal that limitation or treat AI-assisted analysis, however extensive, as satisfying it (GOV-003 §3.4, ADR-0011 §16) — an ARB decision record must disclose this exactly as every other self-approved document in this corpus already does (§10, field 4; GOV-003 §3.5).
- **Additional members** — technical reviewers, domain specialists, or an eventually-appointed Chief Architect — may participate in an ARB session as Reviewers under GOV-003 §3.4, producing recommendations; only the Class B Approval Authority's disposition is binding, per §3.2 above.
- This charter does not set a quorum requirement beyond the existing single-approver reality it discloses here; a future amendment to this charter, or a future GOV-003 amendment appointing a Chief Architect and additional named reviewers, may establish one once more than one participant is genuinely available.

## 15. Meeting Process

An ARB session is not required to be synchronous or convened as a live meeting; "meeting" in this charter means the bounded evaluative process defined by §9, applied to one or more specific findings, and closed by a recorded decision (§10) for each. A session:

1. is opened by identifying the specific document, claim set, or ACR/RSS output under evaluation;
2. proceeds through §9's eight steps for each issue in scope;
3. produces one decision record (§10) per issue, not a single undifferentiated summary for the whole session;
4. is closed by a session-level summary stating which issues were Accepted, which required further action, and which remain open, cross-referenced to each issue's own decision record; and
5. does not become effective as governance until the applicable Class B disposition (§3.2, §9 step 5 read together with GOV-010 §5) is recorded, exactly as any other Class B decision under GOV-010 requires.

## 16. Future Applicability

**This charter governs every future Architecture Review Board, not only a specific Act 2 session.** It is not exhausted by use, does not expire, and is not narrowed to the specific findings any one ARB session considers. A future ARB session — convened in Act 2, Act 3, or at any later point in this project's history — operates under this same charter unless and until it is formally amended through the same Governance-tier process that approves it (§3.2). Nothing in this charter is a one-time, non-reusable act in the sense ADR-0011 §7 uses that phrase for its own Founding Act; the ARB it establishes is, by design, permanent.

## 17. Non-Conclusions

This charter does not: certify any architecture; approve, reject, or reclassify any finding in ACR-001, RSS-001, or any RES document; convene or conduct an ARB session; authorize Act 3 or any future engineering work; amend ARCH-001 through ARCH-006 or any ADR; resolve the Bootstrap Authority State inconsistency disclosed in §3.1; or establish a quorum or membership beyond the single-approver reality disclosed in §14. It establishes the body and the rules under which all of the above may later occur, and nothing more.

## References

| Document ID | Title | Status | Path |
|---|---|---|---|
| GOV-003 | Governance Model | v0.1.0, Operative (Act 2 Approved) | `governance/GOV-003-Governance-Model.md` |
| GOV-010 | Decision Framework | v0.1.0, Operative (Act 2 Approved) | `governance/GOV-010-Decision-Framework.md` |
| GOV-004 | Engineering Principles | v0.1.0, Approved (normal-governance validation) | `governance/GOV-004-Engineering-Principles.md` |
| STD-001 | Documentation Standards | v0.4.0, Draft | `standards/STD-001-Documentation-Standards.md` |
| ADR-0011 | Bootstrap Approval Authority | v0.1.0, Draft — Act 1 effective | `adrs/ADR-0011-Bootstrap-Approval-Authority.md` |
| ADR-0012 | Content-Non-Mutating Act 2 Approval Evidence | v0.1.0, Draft — corrective Founder approval effective | `adrs/ADR-0012-Content-Non-Mutating-Act-2-Approval-Evidence.md` |
| RSS-001 | Research Synthesis Review | v0.1.2, Draft | `consolidation/RSS-001-Research-Synthesis-Review.md` |
| ACR-001 | Architecture Consolidation Review | v0.2.0, Draft | `consolidation/ACR-001-Architecture-Consolidation-Review.md` |
| ARCH-001 | Constitutional Architecture | v0.2.0, Draft | `architecture/ARCH-001-Constitutional-Architecture.md` |
| ARCH-002 | Runtime Architecture | v0.2.1, Draft | `architecture/ARCH-002-Runtime-Architecture.md` |

No external web references are used in this document. Git commit references cited in §3.1 (`f626499`, `43a49b4`, `11cab8d`, `1c1aaef`) refer to this repository's own pushed history on `origin/main`, independently verified during this task, not to any external source.

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-17 | Denver Jacobs (AI-assisted) | Initial Draft, establishing the Architecture Review Board as SynapseOS's permanent architectural governance body: authority and authority limitations (§4–§5); an evidence hierarchy explicitly reconciled with, and subordinate to, GOV-003 §5's existing Document Hierarchy (§6); eight constitutional principles (§7); seven fixed decision classifications extending ACR-001's own evidentiary classification model to governance decisions (§8); an eight-step decision process specializing GOV-010 §7's general decision workflow (§9); a ten-field decision record specializing GOV-010 §8's general decision record (§10); typical inputs and outputs (§11–§12); six standing board principles (§13); composition and meeting process, disclosing the current single-approver, no-independent-reviewer reality rather than assuming a multi-member board that does not yet exist (§14–§15); and explicit future applicability to every later ARB session, not only Act 2 (§16). Independently re-verified, rather than assumed, that GOV-003, GOV-010, and GOV-004 are genuinely Operative/Approved: located and read the full text of each Act 2 and normal-governance evidence commit in this repository's pushed history, confirmed each is an ancestor of the current `origin/main` HEAD, and independently recomputed the SHA-256 of each document's current tracked content against the fingerprint recorded in its evidence commit, confirming an exact match in all three cases (§3.1). Disclosed, without attempting to silently resolve, an apparent inconsistency between the GOV-004 evidence commit's own text and ADR-0011 §12.6/§19's non-discretionary termination rule (§3.1). Explicitly closes the "no ARB is currently defined" gap ACR-001 §17 recorded (§3.3). No architecture, ADR, RSS, ACR, GOV-003, GOV-010, GOV-004, or STD-001 content was modified by this document. |
| 0.1.1 | 2026-07-17 | Denver Jacobs (AI-assisted) | Corrected per an independent technical review's own findings (verdict: CHANGES REQUIRED), each independently re-verified against repository evidence before correction: (1) updated three architecture version citations gone stale after EWO-010's publication — ARCH-002 v0.2.0→v0.2.1, ARCH-003 v0.4.0→v0.5.0, ARCH-006 v0.1.3→v0.1.4 (frontmatter and §References; ARCH-001, ACR-001, and STD-001's own cited versions were independently re-checked and found still current, unaffected by EWO-010, and were left unchanged); (2) corrected the §3.3 quotation of ACR-001 §17, which had omitted "GOV-004" from the enumerated list of documents not defining an ARB — the source text reads "GOV-003, GOV-004, GOV-010, or STD-001," verified directly against ACR-001's current tracked text; (3) corrected the §5.8 quotation "holds no independent approval authority," previously attributed to STD-001 §52's own body text — that exact phrase in fact appears in STD-001's Appendix B (Document Family Register) and Revision History, not §52's body, which instead reads "An AFR does not itself possess approval authority"; §5.8 now cites §52's own body wording directly and separately notes the Appendix B phrasing as the source of the quoted alternative; (4) corrected §11's characterization of ARCH-004 §6's disclosed defect from "mailbox-overflow defect" to an accurate description — the defect ARCH-004 §6 documents is a Scheduler/Lifecycle-Guardian coordination gap causing a permanently-failing actor's second queued message to be silently, permanently discarded, not a mailbox-capacity overflow condition. No governance reasoning, authority model, decision, or substantive conclusion was reopened or changed by this revision — all four corrections are citation-precision fixes only. |

## Approval Status

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
| Document ID | GOV-011 |
| Repository path | governance/GOV-011-Architecture-Review-Board-Charter.md |
| Version | 0.1.1 |
| Artifact commit | Not yet created |
| Git blob ID | Not yet created |
| SHA-256 | Not yet created |
| Approver | Not yet assigned |
| Approver capacity | Not yet assigned |
| Approval-authority source | Not yet assigned (Governance-tier normal-governance disposition per GOV-003 §3.1 and §5, as reasoned in §3.2 above; not a Bootstrap Act 2 act, which never had authority over any future GOV document per ADR-0011 §8) |
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

No field in this section may be populated until the corresponding act has genuinely occurred and is evidenced per ADR-0011 §14 and ADR-0012 §9. This table does not, and must not be read to, claim that GOV-011 has been approved.
