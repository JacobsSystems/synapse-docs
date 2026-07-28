---
document_id: STD-031
title: Engineering Lifecycle Standard
version: 0.1.0
status: Draft
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.2 interim Chief Architect authority (GOV-010 §5, Class B) — no Chief Architect is currently appointed.
created: 2026-07-28
last_updated: 2026-07-28
classification: Public
related_documents:
  governance:
    - GOV-003 (Governance Model) — roles and authority this standard draws on without redefining
    - GOV-004 (Engineering Principles) — "documentation before implementation," the principle this standard's stage ordering exists to enforce operationally
    - GOV-010 (Decision Framework) — decision classes and authority this standard applies, never reclassifies
    - GOV-013 (Engineering Lifecycle, v0.1.0) — **Approved** via Normal-Governance disposition (evidence commit `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6`, effective 2026-07-26), independently confirmed via that commit's own recorded artifact hash/blob-ID/byte-size/line-count, all matching the current tracked artifact exactly. GOV-013's own tracked text still displays `status: Draft` and an unpopulated Approval Status table — by the deliberate exact-byte-identity convention this repository's constitutional-tier documents use (ADR-0011 §14–§15): true disposition lives in the separate evidence commit, never edited into the approved artifact's own bytes. This standard is the governance-tier foundation §4 below builds on; it is not restated in full here, and nothing in this standard amends, reclassifies, or supersedes it.
  standards:
    - STD-001 (Documentation Standards) — the sole authority for document families, identifiers, versioning, and evidence representation this standard defers to throughout
  architecture:
    - ARCH-008 (v0.4.3, Approved) — cited as the demonstrated Architecture Amendment / Independent Architecture Review / Correction / Re-Review cycle this standard's Architecture-stage citations draw on
  adrs:
    - ADR-0011 (Bootstrap Approval Authority) — the exact-byte-identity evidence model this standard's own Publication and Founder Approval stages reuse
    - ADR-0012 (Corrective Founder Approval Evidence Record basis)
  predecessor: GOV-013 (Engineering Lifecycle) — the governance-tier document whose own §13.1 Open Question this standard resolves for the Engineering-Work-Order-centered portion of the lifecycle
  engineering:
    - EWO-014, EWO-015, EWO-016 (cited throughout as the demonstrated evidence for every EWO-centered stage this standard defines)
    - ER-015, ER-016, ER-017 (cited as demonstrated Engineering Report evidence)
---

# STD-031 — Engineering Lifecycle Standard

> **Status notice.** This document is **Draft**. No STD-031-specific approval act has occurred. Drafting, saving, staging, committing, or pushing this document does not itself constitute its approval. See §17 (Approval Status).

Registered per STD-001 §7/§10. Filed as **STD-031** — not "STD-002," which is already `standards/STD-002-Coding-Standards.md`, a different, unrelated, genuinely existing controlled document — following independent repository verification (`ls standards/`, confirming STD-001 through STD-030 are each already claimed, at minimum as a named bootstrap draft, for a specific, unrelated topic) and STD-001 §7's own rule that identifiers are permanent and a claimed one must not be reassigned. STD-031 is the next entirely unclaimed number, chosen on the identical precedent GOV-013 itself already established when an analogous collision was found: GOV-013 was originally requested under "GOV-003," found already taken by an unrelated, genuinely Approved document, and filed instead under the next unused sequential identifier in its family (13, not an internal gap) — this standard follows that same resolution, using STD-031 rather than the unclaimed-but-unexplained STD-021 gap, which sits among a thematically distinct cluster of AI/autonomous-systems standards (STD-020 Memory and Knowledge Governance, STD-022 Tool and Action Execution) and may carry an intent this standard has no evidence to displace.

## 1. Purpose

This standard defines the mandatory, repeatable engineering lifecycle governing every future SynapseOS Engineering Work Order, from architectural authorization through implementation, independent review, evidenced completion, Founder Acceptance, and baseline establishment. It exists so that engineering work is never authorized, implemented, or considered complete except through a fixed, auditable sequence of stages — each with a defined purpose, defined authority, defined evidence, and a defined exit condition — rather than through ad hoc practice that varies unpredictably between milestones.

This standard formalizes, at the standards tier, a lifecycle segment that has now been independently demonstrated, correctly and repeatedly, across multiple completed engineering programmes (ARCH-008's v0.4.2 and v0.4.3 amendments; EWO-014/ER-015; EWO-015/ER-016; EWO-016/ER-017) — it does not invent a new process. Where GOV-013 (Engineering Lifecycle, Approved) already documents a stage, this standard cites it rather than restating it (§4). Where demonstrated practice has since established a stage GOV-013 does not yet name — Independent Engineering Review of an Engineering Work Order itself, Founder Approval as its own distinct act, Founder Acceptance, and Baseline Update — this standard defines that stage for the first time at controlled-document tier, resolving — for the Engineering-Work-Order-centered portion of the lifecycle only — the question GOV-013 §13.1 itself left open (in substance: whether stages such as these should ever become registered STD-001 controlled-document families, rather than remaining functionally demonstrated but unregistered practice). The answer this standard gives is: yes, standardized as a standard, but not as a new registered *document family* — no new document family is created (see §2).

## 2. Scope

**In scope:** the mandatory sequence of stages from an approved architecture through a Founder-accepted, closed Engineering Work Order and the baseline update that follows it; the single question each stage answers; each stage's authority, required evidence, permitted and prohibited actions, and exit criteria; the mandatory verification and validation gates that must pass before implementation, publication, and acceptance; the responsibilities of each repository this lifecycle touches; and how this lifecycle itself may be amended.

**Out of scope, deliberately:**

- The stages preceding an approved architecture (Idea, Research, Research Review, Design Exploration, Design Approval Review, Design Correction, Narrow Design Re-Review, Architecture Authoring, Architecture Review, Architecture Correction, Narrow Architecture Re-Review) — GOV-013 §6.1–§6.11 already defines these; this standard cites them (§4) and does not restate, amend, or duplicate them.
- The definition or amendment of any controlled-document family, identifier convention, versioning rule, or metadata requirement — exclusively STD-001's own domain (STD-001 §1, §7, §13).
- The governance roles and decision-authority framework themselves — exclusively GOV-003's and GOV-010's domain (§8 below cites, never redefines, them).
- Any specific technical capability, architecture, or implementation technology. This standard is written to remain valid regardless of the Runtime's own implementation language or toolchain; where a current-toolchain example is given (§10), it is disclosed explicitly as an illustrative, non-binding current realization of a generic requirement, not a permanent technology dependency.
- Registration of any new controlled-document family. This standard governs the use of the EWO and ER families STD-001 §46/§47 already register; it registers no "IER," "IIR," or "FA" family of its own — those stage outputs are recorded within the EWO's or ER's own text (§6), exactly as GOV-013 §7 already discloses for its own currently-unregistered stages (Design Exploration, Design Approval Review, Architecture Review, Architecture/Design Correction, Narrow Re-Review — GOV-013 §6.4, §6.5, §6.9–§6.11).

## 3. Relationship to GOV-013 and to Governance Generally

GOV-013 (Approved) states the complete engineering lifecycle from Idea through Publication and the principles it exists to enforce; it is the governance-tier constitution this standard operates under, never around. This standard does not compete with it. The division of responsibility, stated plainly:

- **GOV-003** assigns authority to roles.
- **GOV-010** defines how a role exercises that authority to reach a decision.
- **GOV-013** defines the complete stage sequence a capability passes through, at the level of principle, and states explicitly (§13.1) that whether any stage becomes a registered, standards-tier practice is a question for "a future, separately authorized STD-001 amendment."
- **This standard (STD-031)** is that future amendment, scoped to exactly the portion of GOV-013's own lifecycle this project's own history has most repeatedly demonstrated in operational detail — architecture authorization through Engineering Work Order, implementation, review, report, acceptance, and baseline update. It adds no principle GOV-013 does not already state or directly imply (§4); it adds operational specificity — required evidence fields, repository-by-repository permitted/prohibited actions, and mandatory verification/validation gates — GOV-013 itself does not attempt, consistent with its own stated Non-Goals (GOV-013 §7).

Where this standard names a stage GOV-013 does not (Independent Engineering Review of an EWO, Founder Approval, Founder Acceptance, Baseline Update — §6.2, §6.3, §6.8, §6.9), it does so as an *additive* standardization of demonstrated practice, not a contradiction of anything GOV-013 states: GOV-013 §8.6 already establishes that "Approval, once reached, is a review-stage verdict — it becomes an effective, operative decision only once GOV-010 §5's Approval Authority act occurs," which is precisely the Founder Approval act this standard now names explicitly; GOV-013 simply predates (2026-07-26) the first demonstrated instance of a symmetric, post-implementation Founder Acceptance act (EWO-016, 2026-07-28), which is why GOV-013 could not yet name it.

## 4. Engineering Principles

The following are not new principles; each is already stated in GOV-001, GOV-002, GOV-004, or GOV-013 §9, and is restated here only because it is what §5–§9 exist to enforce mechanically for the Engineering-Work-Order-centered lifecycle segment:

1. **Truthful engineering history.** A correction preserves what was originally found and adds a dated correction; it never silently rewrites or deletes prior record (demonstrated: every Revision History entry across ARCH-008, EWO-014, EWO-015, and EWO-016 discloses its own corrections in full, in the order they occurred).
2. **Evidence before conclusions.** No review, approval, or acceptance verdict is recorded without independently re-derived evidence — re-running verification commands, re-reading source, re-computing test totals — never accepted solely on a predecessor's own characterization (demonstrated: every Independent Engineering Review and Independent Implementation Review in this lifecycle's own history re-derives its findings directly, and this standard's own §9 makes that mandatory).
3. **Independent verification.** A review is independent of the work it reviews wherever an independent reviewer exists; where none does, that absence is disclosed plainly, never concealed or implied otherwise (GOV-013 §8.1).
4. **Immutable published history.** An approved or published artifact's own tracked content is not silently rewritten to reflect a later disposition; where an evidence-commit model is used (§7, §13), the artifact's own bytes remain exactly as reviewed, and the disposition lives in the separate, content-non-mutating evidence record (ADR-0011 §14–§15).
5. **Minimal authorized change.** A correction pass resolves exactly the findings identified and discloses, rather than silently absorbs, anything further it happens to notice (GOV-013 §8.4). An implementation stays strictly within its authorizing Engineering Work Order's own stated scope (§6.5).
6. **Repository discipline.** Each repository this lifecycle touches has a defined, bounded role (§7); a task governing one repository does not modify another except where this standard or the authorizing artifact explicitly permits it.
7. **Repeatability.** Every stage's required evidence is independently reproducible by a later party from repository state alone — commit hashes, diff statistics, test totals, hash-verifiable evidence records — never from an unreproducible private claim.
8. **Auditability.** Every stage produces a durable, traceable record of what was decided, by whom, on what evidence, and when.
9. **Deterministic governance.** Given the same repository state and the same authorizing artifact, the lifecycle's own stage sequence and exit criteria produce the same procedural outcome — this standard defines a process, not a matter of individual judgment about which stage applies.

## 5. Lifecycle Overview

```text
                    (GOV-013 §6.1–§6.11 — governed there, cited not restated)
Idea → Research → Research Review → Design Exploration → Design Approval Review
   → Design Correction (if required) → Narrow Design Re-Review
   → Architecture Authoring → Architecture Review
   → Architecture Correction (if required) → Narrow Architecture Re-Review
                                │
                                ▼
                    (this standard, STD-031 — §6 below)
                    Engineering Work Order
                                │
                                ▼
                    Independent Engineering Review
                                │
                                ▼
                    Founder Approval
                                │
                                ▼
                    Publication
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
                    Founder Acceptance
                                │
                                ▼
                    Baseline Update
```

An architecture must have exited Architecture Review (or Narrow Architecture Re-Review, if corrected) under GOV-013 before Engineering Work Order authoring begins (§6.1's own Inputs). The "(if required)" stages are conditional exactly once: if a review returns a verdict requiring no correction, the correction and re-review stages for that finding set do not occur, and the lifecycle proceeds directly to the next stage. This applies identically within §6 below: Independent Engineering Review may require a Correction and Re-Review cycle (demonstrated: EWO-015's own two-round correction cycle) before Founder Approval; Independent Implementation Review may likewise require returning to Implementation (§6.6's own Exit Criteria) before an Engineering Report may be authored.

## 6. Stage Definitions

Each stage below states: Purpose; Inputs; Outputs; Exit Criteria; Responsible Authority; Evidence Required; Repository Affected; Permitted Changes; Prohibited Actions.

### 6.1 Engineering Work Order (EWO)

**Purpose.** How will the approved architecture be implemented?

**Inputs.** An architecture that has exited Architecture Review, or Narrow Architecture Re-Review if corrected (GOV-013 §6.9–§6.11), with a recorded Approved verdict.

**Outputs.** An EWO document, registered per STD-001 §46, in `synapse-docs/work-orders/`.

**Exit Criteria.** The EWO is internally consistent; cites its authorizing architecture sections precisely (never restating or amending them); defines an implementation scope, a Definition of Done, mandatory validation gates (§10), and explicit stop/escalation conditions for when an architectural decision is required mid-implementation; is version `0.1.0`, `status: Draft`.

**Responsible Authority.** Author (GOV-003 §3.3); no independent approval authority is exercised at this stage.

**Evidence Required.** A repository-evidence-derived document identifier (never assumed in advance — STD-001 §7); direct citation of the specific authorizing architecture sections, independently verified against the current architecture text, not restated from memory.

**Repository Affected.** Documentation only. Runtime remains untouched and unmodified throughout authoring.

**Permitted Changes.** Creating one new, uncommitted or freshly drafted EWO document.

**Prohibited Actions.** Redefining or amending architecture; authorizing implementation before this stage's own Exit Criteria are met; assuming a document identifier without independent repository verification.

### 6.2 Independent Engineering Review

**Purpose.** Is the Engineering Work Order itself sound, internally consistent, and faithful to its authorizing architecture, before any implementation is authorized against it?

**Inputs.** A drafted EWO (§6.1).

**Outputs.** A classified findings list (Critical / Major / Minor / Observation, GOV-013 §8.3) and exactly one recorded verdict.

**Exit Criteria.** Zero unresolved Critical or Major findings. Where correction is required, the EWO is corrected — applying only the findings identified (GOV-013 §8.4) — and re-reviewed narrowly (GOV-013 §8.5) before this stage's exit condition is met; this cycle repeats until zero unresolved Critical or Major findings remain (demonstrated: EWO-015's own review → correction → re-review → regression found → correction → focused re-review chain, fully disclosed in its own Revision History).

**Responsible Authority.** Independent Reviewer (GOV-003 §3.4); where no independent human reviewer exists, this is disclosed explicitly, never presented as independent human review (GOV-013 §8.1; demonstrated in every reviewed EWO's own Approval Status table).

**Evidence Required.** Every finding independently re-derived from the EWO's own text and the current repository state — never accepted from the draft's own self-description; a version increment recording the correction (STD-001 §13), with the correction's own Revision History entry naming exactly what changed and why.

**Repository Affected.** Documentation only.

**Permitted Changes.** Correcting the EWO directly, strictly to the findings identified, with a version increment.

**Prohibited Actions.** Approving an EWO with an unresolved Critical or Major finding; expanding the review into a redesign; silently absorbing a newly noticed issue outside the corrected scope without disclosing it as its own, separate finding.

### 6.3 Founder Approval

**Purpose.** Does the reviewed Engineering Work Order become an effective, operative authorization to implement?

**Inputs.** An EWO that has exited Independent Engineering Review (§6.2) with zero unresolved Critical or Major findings.

**Outputs.** A recorded Founder Approval disposition.

**Exit Criteria.** `status` transitions from `Draft` to `Approved`; the document's own Approval Status table records the Approval Authority, the exact decision-class basis (GOV-010 §4–§5), and the date. GOV-013 §8.6's own principle governs here directly: "a review's own 'Approved' wording never substitutes for" this act — Independent Engineering Review's own verdict (§6.2) is a recommendation; only this stage's own Approval Authority act makes the EWO operative.

**Responsible Authority.** Approval Authority (GOV-003 §3.5), ordinarily the Founder exercising Class E (Implementation) decision authority (GOV-010 §4) within already-approved architecture, per GOV-010 §5's delegation provision — disclosed explicitly where exercised directly by the Founder in the absence of an identified delegate (demonstrated identically across EWO-014, EWO-015, and EWO-016's own Approval Status tables).

**Evidence Required.** A dedicated version reserved exclusively for this governance disposition, changing no engineering-scope content (demonstrated: EWO-014 and EWO-015 both reserved a final version exclusively for this act; EWO-016 did so at 0.1.2, having already used an earlier version for its own Independent Engineering Review correction, disclosed as the reason its "pure disposition" version differs numerically from EWO-014/015's own).

**Repository Affected.** Documentation only.

**Permitted Changes.** `status`/`version` transition, Approval Status table completion, a Revision History entry recording the disposition — no engineering-scope, requirement, or exclusion content may change at this stage.

**Prohibited Actions.** Approving an EWO carrying an unresolved Critical or Major finding; altering any technical requirement while recording approval; treating drafting, staging, committing, or pushing as itself this act (GOV-013 §6.16's own identical prohibition applies here).

### 6.4 Publication

**Purpose.** Is the approved Engineering Work Order now a durable, evidenced part of the repository?

**Inputs.** An EWO with an effective Founder Approval disposition (§6.3).

**Outputs.** A pushed Documentation-repository commit.

**Exit Criteria.** The commit is present on `origin/main`; local and remote HEAD match; ahead/behind reads `0/0`.

**Responsible Authority.** Implementer or Author, acting on an already-effective approval (GOV-003 §3.6) — publication is a mechanical act, not a further decision.

**Evidence Required.** A truthful commit message; a staged-file list containing only the authorized artifact; `git diff --check` clean; pre- and post-push HEAD comparison against `origin/main`.

**Repository Affected.** Documentation. Runtime remains untouched.

**Permitted Changes.** Staging, committing, and pushing exactly the approved EWO.

**Prohibited Actions.** Publishing an EWO without an effective Founder Approval disposition; staging any unrelated file, including pre-existing repository drift; force-pushing.

### 6.5 Implementation

**Purpose.** Has the approved Engineering Work Order been faithfully realized in the Runtime repository?

**Inputs.** A published, Approved EWO.

**Outputs.** Runtime source and test changes satisfying the EWO's own stated scope, Definition of Done, and mandatory validation gates.

**Exit Criteria.** Every EWO requirement is met and independently verified (§9, §10) from a forced-clean state, or the EWO is returned with disclosed, specific reasons why it cannot be completed as written (GOV-013 §6.13's identical exit condition).

**Responsible Authority.** Implementer (GOV-003 §3.6) — obligated to implement only decisions currently effective, and to escalate rather than resolve unilaterally any architectural decision the EWO did not itself bound.

**Evidence Required.** Repository verification before and after implementation (branch, HEAD, working-tree cleanliness, ahead/behind against origin); a specification-to-code trace mapping every EWO requirement to its realized location; full workspace verification (§10) passing from a forced-clean state; independently re-derived test totals, by direct summation, never asserted from a single aggregate figure alone.

**Repository Affected.** Runtime. Documentation remains unmodified by this stage — any Documentation change belongs to a separately governed task.

**Permitted Changes.** Source, test, and example changes strictly within the EWO's own authorized file scope; a mechanical, purely syntactic migration where the EWO explicitly authorizes one, verified line-by-line to contain no semantic change.

**Prohibited Actions.** Exceeding EWO scope; making an architecture-level decision without escalation; modifying Documentation; committing or staging unrelated repository drift.

### 6.6 Independent Implementation Review

**Purpose.** Does the implementation faithfully satisfy the Engineering Work Order?

**Inputs.** A completed implementation reporting exit from §6.5.

**Outputs.** A classified findings list and exactly one recorded verdict, independently re-derived from source — never accepted from the implementer's own report (demonstrated: this exact discipline caught a genuine focused-test-count misreporting error during EWO-016's own Independent Implementation Review, corrected before publication in the resulting Engineering Report).

**Exit Criteria.** Zero unresolved Critical or Major findings. A Major or Critical finding returns the work to Implementation (§6.5) — not to a new Engineering Work Order — unless the required change is itself architecture-level, in which case it routes back to Architecture Review under GOV-013 §6.9 (GOV-013's identical routing rule applies unchanged here).

**Responsible Authority.** Independent Reviewer (GOV-003 §3.4), independence disclosed or its absence disclosed, on the identical basis as §6.2.

**Evidence Required.** Direct re-inspection of the changed source, not the implementation's own description of it; an independent re-run of every mandatory verification and validation gate (§9, §10); a scope audit classifying every changed file as within or outside the EWO's authorized scope, with any scope creep reported as a finding.

**Repository Affected.** Runtime is evidence only for this stage — read, never modified, unless a genuine defect is found, in which case any correction remains strictly within the authorizing EWO's own scope and is itself independently re-verified before the review concludes.

**Permitted Changes.** A minimal, disclosed, in-scope defect correction, where a genuine implementation defect is found — never a scope expansion.

**Prohibited Actions.** Accepting an implementation report's own claims without independent re-verification; redesigning the implementation; expanding EWO scope.

### 6.7 Engineering Report (ER)

**Purpose.** What evidence proves completion?

**Inputs.** An implementation that has exited Independent Implementation Review (§6.6) with zero unresolved Critical or Major findings.

**Outputs.** An ER document, registered per STD-001 §47, in `synapse-docs/engineering-reports/`, citing its authorizing EWO (`reports_on`, never omitted).

**Exit Criteria.** The ER is evidentially self-consistent: repository state, diff statistics, and test totals are independently re-derived at authoring time, not restated from a prior claim (demonstrated: this exact re-derivation caught and corrected a misreported focused test count during ER-017's own authoring, rather than propagating it). The ER records the Independent Implementation Review's own findings and verdict truthfully and proportionately — a corrected reporting error is disclosed as exactly that, never overstated as an implementation defect nor concealed.

**Responsible Authority.** Author (GOV-003 §3.3). An Engineering Report carries no independent authority of its own — it is informational only and creates no new requirement (STD-001 §47; GOV-013 §6.15's identical principle) — it records that work happened; it does not make that work operative or closed.

**Evidence Required.** Exact repository baselines (both repositories, branch, HEAD, ahead/behind, working-tree state) at authoring time; per-file diff statistics; independently re-run workspace and focused verification (§9, §10); the Independent Implementation Review's findings, classified and reconciled.

**Repository Affected.** Documentation only. Runtime remains evidence, unmodified.

**Permitted Changes.** Creating exactly one new ER document.

**Prohibited Actions.** Claiming Founder Acceptance or closure within the ER itself — that is a distinct, subsequent act (§6.8); modifying the authorizing EWO, any prior ER, architecture, ADRs, or standards; repeating a prior report's own unverified figures without independent re-derivation.

### 6.8 Founder Acceptance

**Purpose.** Does the Founder accept the completed, evidenced implementation as final, closing the Engineering Work Order?

**Inputs.** A published Engineering Report (§6.7) recording zero unresolved Critical or Major findings from Independent Implementation Review.

**Outputs.** A recorded Founder Acceptance disposition on the governing EWO.

**Exit Criteria.** The EWO's own `status` transitions to `Implemented` (STD-001 §12 — "optional status for specifications whose implementation is verified"; not an invented value — this is the first use of that already-defined status in this repository, chosen over an unregistered term precisely because STD-001 already defines it for this exact purpose). The EWO's own Approval Status table gains a Founder Acceptance row recording the acceptance and the implementation commit it accepts. The EWO's own Disposition section states plainly that it is closed and that no further work is authorized under it.

**Responsible Authority.** Approval Authority (GOV-003 §3.5), the same basis as §6.3 — Founder Acceptance and Founder Approval are related but distinct acts of the same authority, at different lifecycle points, exactly as GOV-010 §5 already distinguishes "authorship, review, recommendation, approval... are distinct acts."

**Evidence Required.** Independent re-confirmation, immediately before recording acceptance, that both repositories remain at their expected, evidenced state; that the Engineering Report's own findings genuinely contain zero unresolved Critical or Major items; a dedicated version reserved exclusively for this disposition, changing no engineering-scope content, on the identical "reserved pure-disposition version" basis §6.3 already establishes.

**Repository Affected.** Documentation only. Runtime remains unmodified and is cited only as historical evidence.

**Permitted Changes.** `status` transition, Approval Status table completion, Disposition rewrite, Revision History entry — no technical requirement changes.

**Prohibited Actions.** Accepting an implementation while any Critical or Major finding remains unresolved; reopening implementation, architecture, or any prior review's own conclusion; modifying the Engineering Report.

### 6.9 Baseline Update

**Purpose.** Does the now-closed Engineering Work Order's own implementation commit become the reference point future engineering work is measured against?

**Inputs.** A Founder-accepted, closed EWO (§6.8), naming a specific Runtime implementation commit and a specific Documentation publication commit.

**Outputs.** No new document. The closed EWO's own recorded commit identities (Runtime implementation commit; Documentation acceptance commit) become the "expected baseline" future Engineering Work Orders, reviews, and reports cite and verify against — exactly the "Expected Runtime baseline" / "Expected Documentation baseline" fields every reviewed EWO and Engineering Report in this lifecycle's own history already opens with.

**Exit Criteria.** The next Engineering Work Order authored against this lifecycle segment correctly cites the newly closed EWO's own commit identities as its own starting baseline, independently re-verified — never assumed — at that future EWO's own Phase 1 Repository Verification.

**Responsible Authority.** Author of the next Engineering Work Order (GOV-003 §3.3), verifying rather than assuming the baseline.

**Evidence Required.** A fresh `git rev-parse HEAD` / `git status --short` / `git fetch` verification, at the start of the next engineering task, confirming the cited baseline commit is genuinely the current repository state (or that any legitimate advance since is disclosed and investigated, never silently reset to).

**Repository Affected.** Both, as read-only reference. This stage performs no write of its own.

**Permitted Changes.** None. This stage is definitional, not an action — it states what "baseline" means and how it propagates; it does not itself modify any repository.

**Prohibited Actions.** Resetting either repository to a prior commit to "match" a stated baseline; treating a baseline citation as license to skip Phase 1 Repository Verification in the next engineering task; assuming a baseline is current without independently re-fetching and re-checking it.

## 7. Repository Responsibilities

**Runtime repository.** Holds only implementation source, tests, and examples. Modified exclusively during the Implementation stage (§6.5) and, narrowly, during Independent Implementation Review (§6.6) where a genuine, in-scope defect is found. Evidence-only at every other stage. Never modified during a governance task (Founder Approval, Founder Acceptance, or this standard's own authoring).

**Documentation repository.** Holds every controlled document this lifecycle produces: architecture (`architecture/`), standards (`standards/`), governance (`governance/`), ADRs, Engineering Work Orders (`work-orders/`), Engineering Reports (`engineering-reports/`), and research/consolidation outputs, per STD-001 §10's Repository Location Standard. A task governing one document family stages and commits only the files that family's own stage authorizes (§6) — never an unrelated document, and never pre-existing, previously disclosed repository drift.

**Architecture documents (`architecture/`).** Authored and reviewed under GOV-013 §6.8–§6.11 (out of this standard's own scope, §2); cited, never restated or amended, by every Engineering Work Order this standard governs (§6.1).

**Standards documents (`standards/`).** Authored and amended per STD-001 §13's versioning rules and this standard's own §13 (Change Control). This standard is itself one such document.

**Engineering Work Orders and Engineering Reports (`work-orders/`, `engineering-reports/`).** Registered per STD-001 §46/§47; their own full lifecycle is what §6 of this standard defines in operational detail.

## 8. Engineering Authorities

- **Founder (GOV-003 §3.1).** Retains final authority for Class A decisions; exercises Class B interim authority in the absence of an appointed Chief Architect (GOV-003 §3.2); is, in this project's current, disclosed state, also the Approval Authority (§3.5) exercising the Class E decision authority (GOV-010 §4) this standard's Founder Approval (§6.3) and Founder Acceptance (§6.8) stages require — a delegable authority (GOV-010 §5), exercised directly only in the disclosed absence of an identified delegate.
- **Architecture Review Board / Architecture Review (GOV-011, GOV-013 §6.9).** Out of this standard's own scope (§2); this standard's Engineering Work Order stage (§6.1) requires only that architecture has already exited this review with an effective Approved disposition.
- **Independent Reviewer (GOV-003 §3.4).** Conducts Independent Engineering Review (§6.2) and Independent Implementation Review (§6.6); discloses independence or its disclosed absence in every case; re-derives every finding from source, never from a predecessor's own report.
- **Implementation Engineer / Implementer (GOV-003 §3.6).** Carries out only currently-effective decisions; implements strictly within EWO scope (§6.5); escalates rather than resolves any architecture-level question unilaterally.
- **Acceptance Authority.** The same Approval Authority (GOV-003 §3.5) exercising Founder Acceptance (§6.8) — a distinct act from Founder Approval (§6.3), at a later lifecycle point, requiring its own independent evidence (§6.8's own Evidence Required) rather than being inferred from the earlier approval.

**Decision authority is never reclassified by this standard.** Whether a specific change is Class B (Architectural) or Class E (Implementation) is determined by GOV-010 §4 alone (GOV-013 §11's identical principle); this standard states which stage's own act corresponds to which already-defined class, and introduces no new class.

## 9. Mandatory Verification

Before Implementation begins (§6.5):

- Repository verification, both repositories: current branch, HEAD, working-tree cleanliness, ahead/behind against `origin/main`, and disclosure of any pre-existing, unrelated drift left deliberately untouched.
- Confirmation that the governing EWO carries an effective Founder Approval disposition (§6.3) and has been published (§6.4).
- A specification-to-code trace, mapping every EWO requirement to its intended implementation location, before any source file is edited.
- Baseline verification: the full validation suite (§10) run once, before implementation, to establish the exact pre-implementation state independently re-derived counts are compared against.

Before Publication (§6.4), Founder Approval (§6.3), and Founder Acceptance (§6.8):

- Independent re-verification that the artifact being published, approved, or accepted carries zero unresolved Critical or Major findings from its own governing review.
- Independent re-verification of both repositories' current state, immediately before the act — never relying on a verification performed earlier in the same task if repository state could plausibly have changed since.
- For Founder Acceptance specifically: independent confirmation that the cited Engineering Report and the cited Independent Implementation Review outcome are genuinely consistent with the current repository state, not merely restated from the Engineering Report's own text.

## 10. Required Validation

Before Founder Approval, Implementation completion, Publication, and Founder Acceptance, the following categories of validation are mandatory. The current Runtime toolchain's concrete commands are given as the present, disclosed realization of each category — not a permanent technology dependency; should the Runtime's own implementation language or toolchain ever change, this standard's own requirement is unchanged, and only the illustrative command changes:

- **Formatting conformance** — currently: `cargo fmt --all -- --check`.
- **Static analysis / lint conformance, warnings treated as failures** — currently: `cargo clippy --workspace --all-targets --all-features -- -D warnings`.
- **Full workspace build** — currently: `cargo build --workspace --all-targets --all-features`.
- **Full workspace test execution, with an independently re-derived total** (by direct summation across every test binary, never asserted from a single aggregate figure alone) — currently: `cargo test --workspace --all-targets --all-features`.
- **Whitespace/diff hygiene** — currently: `git diff --check`.
- **Repository-state verification** — `git status --short`, `git branch --show-current`, `git rev-parse HEAD`, `git fetch origin`, `git rev-list --left-right --count HEAD...origin/main`, both repositories, at every stage boundary named in §9.
- **Published-specification verification** — independent confirmation, from the artifact's own frontmatter and Approval Status table (or, for constitutional-tier documents using the evidence-commit model, independent recomputation of the cited hash/blob-ID/byte-size/line-count against a separate evidence commit — §3's own worked example), that the specification being implemented, reviewed, or accepted is genuinely in the state assumed, never taken on faith.
- **Engineering Report re-derivation** — every figure an Engineering Report states (test totals, diff statistics, finding counts) independently re-run and re-computed at that report's own authoring time, never copied forward from an earlier draft or claim.
- **Acceptance-time re-confirmation** — the full validation suite above, re-run or its most recent passing result independently re-verified as still current, before Founder Acceptance is recorded.

## 11. Engineering Rules

1. Architecture before implementation. No Engineering Work Order may authorize implementation against architecture that has not itself exited Architecture Review with an effective Approved disposition (GOV-013 §6.12's identical rule).
2. No implementation without an approved, published Engineering Work Order (GOV-004 §1; §6.1, §6.3, §6.4 above).
3. No Runtime modification during a governance-only task (Founder Approval, Publication, Founder Acceptance, or standards authoring) — Runtime is evidence only at every stage except Implementation (§6.5) and, narrowly, Independent Implementation Review (§6.6).
4. No undocumented engineering work. Every Runtime change traces to a specific, cited Engineering Work Order (GOV-004 §11; GOV-013 §9 principle 4).
5. No engineering outside an approved EWO's own stated scope, at any stage (§6.1, §6.5, §6.6).
6. Independent review is required before Founder Approval (§6.2) and before an Engineering Report may be authored (§6.6) — its absence, where genuinely unavailable, is disclosed, never concealed or implied.
7. Truthful engineering history throughout: a correction discloses what was wrong and what changed; a misreported figure is corrected and the correction itself disclosed, never silently absorbed (demonstrated: §6.7's own worked example).
8. Evidence over assumptions at every stage boundary (§9): a document's identifier, version, status, and content are independently re-verified from the repository, never assumed from a prior task's memory of them.
9. Minimal authorized change: a correction pass, at any stage, resolves exactly what its governing review identified (§6.2's own Exit Criteria).
10. Publication, Founder Approval, and Founder Acceptance are each distinct acts; none is inferred from another, and none is inferred from the mere existence, staging, committing, or pushing of a document (§6.3, §6.4, §6.8; GOV-013 §6.16's identical principle).

## 12. Baseline Management

A **baseline** is the specific, named commit identity — in both the Runtime and Documentation repositories — that a Founder-accepted, closed Engineering Work Order (§6.8) leaves behind as the reference point every subsequently authored Engineering Work Order must independently verify itself against before proceeding (§6.9).

A baseline is established exactly once per closed Engineering Work Order, at the moment Founder Acceptance is recorded (§6.8) — never before, since the implementation commit it names is not final until that act occurs, and never retroactively, since STD-001 §4/§28's own immutability principle ("approved documents are not silently rewritten"; "approved documents must not have substantive history erased") forbids rewriting a prior baseline's own recorded identity after the fact.

A baseline record must state, at minimum: the Runtime implementation commit hash and its subject line; the Documentation commit hash at which the governing Engineering Report was published; the Documentation commit hash at which Founder Acceptance itself was recorded; and the exact Independent Implementation Review outcome (finding counts by severity) that Acceptance relied on.

Future engineering work begins from a baseline by independently re-verifying it, never assuming it: the next Engineering Work Order's own Phase 1 Repository Verification (§9) must confirm the cited baseline commit is genuinely the current repository state, or disclose and investigate any legitimate advance since — never reset either repository to force a match (§6.9's own Prohibited Actions).

## 13. Change Control

This standard follows STD-001 §13's versioning rule: PATCH for editorial correction, MINOR for backward-compatible addition or clarification, MAJOR for a change to obligation, stage ordering, or authority assignment.

A finding against this standard, once this standard is Founder-approved, is corrected via the identical discipline §6.2 already establishes for an Engineering Work Order: a review identifies specific, nameable findings; a correction pass resolves exactly those findings; a narrow re-review confirms resolution without new drift. Which decision class governs a proposed change to this standard is determined by GOV-010 §4, not by this document (identical to GOV-013 §11's own rule) — an amendment altering stage ordering or authority is ordinarily Class B; a purely editorial fix is a lesser-review-depth PATCH, never requiring less than an evidenced approval act regardless of size (GOV-013 §11's identical principle, restated here for this standard specifically).

This standard does not retroactively reclassify, invalidate, or require rework of any engineering work completed before this standard's own effective date (GOV-010 §21's prospective-application principle; GOV-013 §12's identical conformance rule) — including the EWO-014, EWO-015, and EWO-016 lifecycles this standard was itself derived from observing.

## 14. Compliance

All future Engineering Work Orders, and every stage named in §6, must comply with this standard from the point it becomes effective (GOV-010 §21) forward, unless superseded by a later approved standard. Where a future capability's own scale genuinely does not warrant a given stage's full formal weight, that proportionality is itself part of conformance, provided it is disclosed exactly as such — never silently assumed (GOV-013 §12's identical principle, adopted here without change).

## 15. References

Internal:

- GOV-003 — Governance Model
- GOV-004 — Engineering Principles
- GOV-010 — Decision Framework
- GOV-013 — Engineering Lifecycle (Approved via Normal-Governance disposition, evidence commit `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6`)
- STD-001 — Documentation Standards (§7, §12, §13, §31, §46, §47)
- ADR-0011 — Bootstrap Approval Authority
- ADR-0012 — Corrective Founder Approval Evidence Record basis

Source evidence (independently re-verified during this standard's own preparation, not restated from memory):

- `governance/GOV-013-Engineering-Lifecycle.md` and its evidence commit `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6` (independently re-read in full)
- `architecture/ARCH-008-Effect-Runtime-Architecture.md` v0.4.3 and its approval commit `2bd2595912f3ba1acf30d80684501c95bc4903fd`
- `work-orders/EWO-014-Provider-Idempotency-Registration.md` (commit `1c51f5e`) and `engineering-reports/ER-015-Provider-Idempotency-Registration.md` (commit `b1439cb`)
- `work-orders/EWO-015-Retry-Architecture-Implementation.md` (commit `3ca11f2`) and `engineering-reports/ER-016-Retry-Architecture-Implementation.md` (commit `1c50d55`)
- `work-orders/EWO-016-ConstraintSet-Based-Retry-Policy.md` (Founder Approval commit `1038e8a`; Runtime implementation commit `29ea55ced6348490f90bd7baeb08d3d4705f19ab`; Founder Acceptance commit `a316874`) and `engineering-reports/ER-017-ConstraintSet-Based-Retry-Policy.md` (commit `9ea9fa6`)
- `standards/STD-001-Documentation-Standards.md` §7, §12, §13, §46, §47 (read directly, not paraphrased from memory)
- `standards/STD-002-Coding-Standards.md` (read directly to confirm the identifier collision this standard's own filing note, above, discloses)

## 16. Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Denver Jacobs (AI-assisted) | Initial Draft. Formalizes, at the standards tier, the Engineering-Work-Order-centered segment of the engineering lifecycle GOV-013 (Approved) already establishes at the governance-principle level — resolving GOV-013's own §13.1 Open Question for that segment specifically. Adds four stages GOV-013 does not yet name, each grounded in directly cited, independently re-verified repository evidence: Independent Engineering Review of an Engineering Work Order itself (demonstrated: EWO-014, EWO-015's two-round correction cycle, EWO-016); Founder Approval as a distinct act from a review's own verdict (demonstrated: EWO-014 v0.1.1, EWO-015 v0.1.1, EWO-016 v0.1.2, each a dedicated pure-disposition version); Founder Acceptance as a distinct, post-Engineering-Report act (demonstrated for the first time by EWO-016 v0.1.3, closing that EWO via STD-001 §12's own `Implemented` status); and Baseline Update (the propagation mechanism by which a closed EWO's own commit identities become the next EWO's own starting baseline). Filed as STD-031 after independently verifying that "STD-002" — the identifier originally requested — is already `standards/STD-002-Coding-Standards.md`, an unrelated, genuinely existing document, and that STD-001 through STD-030 are each already claimed for a specific, unrelated topic; STD-031 is the next unclaimed sequential number, chosen on the identical precedent GOV-013 itself established when it resolved its own analogous "GOV-003" collision by moving to the next unused sequential family number rather than reusing an internal gap. No architecture, implementation, governance role, or decision class is redefined; no new controlled-document family is registered. |

## 17. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-28 |
| Independent Engineering Review | TBD | Pending | |
| Approval Authority | TBD | Pending | |

This standard is genuinely **Draft** — no Independent Engineering Review or Founder Approval act, and no separate evidence commit of any kind, has occurred for it. It is drafted and published to the repository as a candidate standard, ready to be cited and used going forward on that disclosed basis. This is distinct from GOV-013's own state (§3, related_documents above): GOV-013 is actually **Approved**, via a Normal-Governance evidence commit that leaves its own tracked text unedited (ADR-0011's exact-byte-identity convention) — its internal `Draft` appearance is a bookkeeping artifact of that convention, not a genuinely unapproved state. This standard carries no such evidence commit yet; its own `Draft` status is exactly what it appears to be. Drafting, staging, committing, or pushing this document does not itself constitute its approval (§4 principle 10; GOV-013 §6.16's identical prohibition).
