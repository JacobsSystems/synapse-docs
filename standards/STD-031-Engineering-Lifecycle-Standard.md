---
document_id: STD-031
title: Engineering Lifecycle Standard (Engineering Work Order Lifecycle)
version: 0.2.1
status: Approved
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.2 interim Chief Architect authority (GOV-010 §5, Class B) — no Chief Architect is currently appointed.
created: 2026-07-28
last_updated: 2026-07-29
classification: Public
related_documents:
  governance:
    - GOV-003 (Governance Model) — roles and authority this standard draws on without redefining
    - GOV-004 (Engineering Principles) — "documentation before implementation," the principle this standard's stage ordering exists to enforce operationally
    - GOV-010 (Decision Framework) — decision classes and authority this standard applies, never reclassifies
    - GOV-013 (Engineering Lifecycle, v0.1.0) — **Approved** via Normal-Governance disposition (evidence commit `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6`, effective 2026-07-26), independently re-confirmed via that commit's own recorded artifact hash/blob-ID/byte-size/line-count, all matching the current tracked artifact exactly. GOV-013's own tracked text still displays `status: Draft` and an unpopulated Approval Status table — by the deliberate exact-byte-identity convention this repository's constitutional-tier documents use (ADR-0011 §14–§15): true disposition lives in the separate evidence commit, never edited into the approved artifact's own bytes. GOV-013 is the sole governing authority for every lifecycle stage through Engineering Work Order authoring (its own §6.1–§6.12); this standard does not restate, amend, or supersede any of them — it begins only where GOV-013 §6.12 ends (§2).
  standards:
    - STD-001 (Documentation Standards) — the sole authority for document families, identifiers, versioning, and evidence representation this standard defers to throughout
  architecture:
    - ARCH-008 (v0.4.3, Approved) — cited as the demonstrated Architecture Amendment / Independent Architecture Review / Correction / Re-Review cycle preceding the Engineering Work Orders this standard's stages draw evidence from
  adrs:
    - ADR-0011 (Bootstrap Approval Authority) — the exact-byte-identity evidence model this standard's own Publication stage reuses
    - ADR-0012 (Corrective Founder Approval Evidence Record basis)
  predecessor: GOV-013 (Engineering Lifecycle) — the governance-tier document whose own §13.1 Open Question this standard resolves for the post-Engineering-Work-Order portion of the lifecycle only
  engineering:
    - EWO-014, EWO-015, EWO-016 (cited throughout as the demonstrated evidence for every stage this standard defines)
    - ER-015, ER-016, ER-017 (cited as demonstrated Engineering Report evidence)
  reviews:
    - IER-STD-031 (Independent Engineering Review of this standard's own v0.1.0, concluding `RETURN TO AUTHOR FOR REVISION` — two MAJOR findings, F01 and F02, both resolved by the v0.2.0 revision; see §17 Revision History)
    - IER-STD-031-R2 (Independent Engineering Re-Review of v0.2.0, concluding `Approve with minor corrections` — 0 CRITICAL, 0 MAJOR, 1 MINOR (a Publication self-description over-claim, corrected by this v0.2.1 revision), 1 OBSERVATION (disposition recorded, §17 Revision History); see §17 Revision History and §18 Approval Status)
---

# STD-031 — Engineering Lifecycle Standard (Engineering Work Order Lifecycle)

> **Status notice.** This document is **Approved** (v0.2.1, following Founder Approval on 2026-07-29). Drafting, saving, staging, committing, or pushing this document did not itself constitute its approval — the recorded Founder Approval act (§18) did. See §18 (Approval Status).

> **Revision notice.** v0.2.0 was a MAJOR revision correcting two MAJOR findings (F01, F02) an Independent Engineering Review (IER-STD-031) identified against v0.1.0. v0.1.0 restated five stages GOV-013 already fully defines — including verbatim-identical Purpose text for three of them — while explicitly claiming not to, and placed a "Publication" stage at a lifecycle position conflicting with GOV-013's own placement of that stage. Both defects were architectural, not merely editorial: v0.2.0 narrowed this standard's own scope so that it no longer attempts to define any stage GOV-013 already governs, and resolved the Publication conflict by defining Publication once, as a single reusable act GOV-013 §6.16 already establishes, invoked at named points within this standard's own scope rather than redefined. v0.2.1 is a PATCH correcting one MINOR finding an Independent Engineering Re-Review (IER-STD-031-R2) identified against v0.2.0: §2 and §3 overstated the Publication resolution as adding nothing beyond GOV-013 §6.16, when §7.3 in fact adds necessary EWO-lifecycle-specific operational application; the wording is corrected, no normative meaning changed. See §17 for the complete disclosure.

Registered per STD-001 §7/§10. Filed as **STD-031** — not "STD-002" (already `standards/STD-002-Coding-Standards.md`, an unrelated, genuinely existing document) and not the unclaimed-but-unexplained STD-021 gap — for the reasons stated in v0.1.0's own filing note, unchanged by this revision: STD-031 is the next entirely unclaimed sequential number, chosen on the identical precedent GOV-013 itself established when it resolved its own analogous "GOV-003" collision.

## 1. Purpose

This standard defines the mandatory, repeatable **Engineering Work Order lifecycle**: the sequence of stages a SynapseOS capability passes through starting from the moment an Engineering Work Order has been authored (per GOV-013 §6.12) through Independent Engineering Review, Founder Approval, Publication, Implementation, Independent Implementation Review, Engineering Report, Founder Acceptance, and Baseline Update. It exists so that no engineering work is authorized, implemented, or considered complete except through this fixed, auditable sequence — each stage with a defined purpose, defined authority, defined evidence, and a defined exit condition.

This standard does **not** define the complete SynapseOS engineering lifecycle. The complete lifecycle — Idea through Publication — is GOV-013's own domain (Approved, evidence commit `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6`). GOV-013 §13.1 leaves open the question of whether any of its own stages should become a registered, standards-tier practice. This standard answers that question narrowly and only for the stages beginning after an Engineering Work Order exists: yes, standardized here, as an operational extension of GOV-013 — never a restatement, never a competing definition, and never a new registered document family (§2, §3).

Every stage this standard defines has been independently demonstrated, correctly and repeatedly, across completed engineering programmes (ARCH-008's v0.4.2 and v0.4.3 amendments; EWO-014/ER-015; EWO-015/ER-016; EWO-016/ER-017) before being formalized here. This standard formalizes demonstrated practice; it does not invent a new process.

## 2. Scope

**Entry precondition, not a stage this standard defines.** This standard's own scope begins only once GOV-013 §6.12's own Exit Criteria are satisfied: an Engineering Work Order exists, is internally consistent, cites its authorizing architecture precisely, and defines objective, testable completion and validation criteria. Everything preceding that point — Idea, Research, Research Review, Design Exploration, Design Approval Review, Design Correction, Narrow Design Re-Review, Architecture Authoring, Architecture Review, Architecture Correction, Narrow Architecture Re-Review, and the authoring of the Engineering Work Order itself — is governed exclusively by GOV-013 §6.1–§6.12 and is neither restated, amended, nor duplicated anywhere in this document. Where this standard needs to say anything about EWO authoring at all (identifier evidence, repository scope), it says so as a one-line cross-reference (§7), never as a competing stage definition.

**In scope:** the mandatory sequence of stages from a drafted Engineering Work Order through Independent Engineering Review, Founder Approval, Publication, Implementation, Independent Implementation Review, Engineering Report, Founder Acceptance, and Baseline Update; the single question each stage answers; each stage's authority, required evidence, permitted and prohibited actions, and exit criteria; the mandatory verification and validation gates that must pass before implementation, publication, and acceptance; the responsibilities of each repository this lifecycle touches; and how this lifecycle itself may be amended.

**Out of scope, deliberately:**

- Every stage GOV-013 already defines, in full, including Engineering Work Order authoring itself (GOV-013 §6.1–§6.12) **and** the three stages GOV-013 also already fully defines that this standard's own scope continues through — Implementation (§6.13), Independent Implementation Review (§6.14), and Engineering Report (§6.15). This standard does not restate any of these four stages' own Purpose, Permitted/Prohibited Activities, Required Outputs, or Entry/Exit Criteria. Where this standard has anything further to say about them, it says so as a clearly labeled *extension* citing the specific GOV-013 subsection it extends, never as an independent redefinition (§7.4–§7.6).
- **Publication as a generic concept.** GOV-013 §6.16 remains the sole authoritative definition of what Publication means and requires; this standard does not replace, contradict, or create a second definition of it. This standard does not redefine Publication — it identifies the specific points within the Engineering-Work-Order-execution lifecycle where GOV-013 §6.16's own generic stage is invoked, and adds only the EWO-lifecycle-specific operational application and evidence requirements that invoking it at those three named points requires (§7.3). Each of the three invocation points is a specialization of the single GOV-013-governed Publication act, never a competing one.
- The definition or amendment of any controlled-document family, identifier convention, versioning rule, or metadata requirement — exclusively STD-001's own domain (STD-001 §1, §7, §13).
- The governance roles and decision-authority framework themselves — exclusively GOV-003's and GOV-010's domain (§12 below cites, never redefines, them).
- Any specific technical capability, architecture, or implementation technology. This standard is written to remain valid regardless of the Runtime's own implementation language or toolchain; where a current-toolchain example is given (§11), it is disclosed explicitly as an illustrative, non-binding current realization of a generic requirement, not a permanent technology dependency.
- Registration of any new controlled-document family. This standard governs the use of the EWO and ER families STD-001 §46/§47 already register; it registers no "IER," "IIR," or "FA" family of its own — those stage outputs are recorded within the EWO's or ER's own text (§7), exactly as GOV-013 §7 already discloses for its own currently-unregistered stages.

## 3. Relationship to GOV-013

GOV-013 (Approved) states the complete engineering lifecycle from Idea through Publication; it is the governance-tier constitution this standard operates strictly beneath, never around and never in parallel with. The division of responsibility, stated plainly:

- **GOV-003** assigns authority to roles.
- **GOV-010** defines how a role exercises that authority to reach a decision.
- **GOV-013** defines the complete stage sequence a capability passes through, at the level of principle, including every stage through Engineering Work Order authoring, Implementation, Independent Implementation Review, Engineering Report, and Publication (§6.1–§6.16) — and states explicitly (§13.1) that whether any of its stages becomes a registered, standards-tier practice is a question for a future, separately authorized STD-001 amendment.
- **This standard (STD-031)** is that future amendment — but only for the portion of GOV-013's lifecycle beginning after Engineering Work Order authoring (GOV-013 §6.12) is complete. For every GOV-013 stage this standard's own scope continues through (Implementation §6.13, Independent Implementation Review §6.14, Engineering Report §6.15), this standard adds only what GOV-013's own generic schema does not itself specify — required evidence fields, repository-by-repository permitted/prohibited actions, and mandatory verification/validation gates — never a restated Purpose, Permitted/Prohibited Activity, or Entry/Exit Criterion (§7.4–§7.6). For Publication (GOV-013 §6.16), GOV-013 remains the authoritative definition of what Publication means; this standard adds only the EWO-lifecycle-specific operational application and evidence requirements for identifying where, within Engineering-Work-Order execution, that one generic stage is invoked (§7.3) — a specialization of GOV-013's own single act, never a second definition of it.
- **Only four stages in this standard have no GOV-013 equivalent at all**, and only these four are defined here in full: Independent Engineering Review of the Engineering Work Order itself (§7.1), Founder Approval (§7.2), Founder Acceptance (§7.7), and Baseline Update (§7.8). Each is additive, demonstrated practice this project's own recent history establishes, not a contradiction of anything GOV-013 states. GOV-013 §8.6 already establishes that "Approval, once reached, is a review-stage verdict — it becomes an effective, operative decision only once GOV-010 §5's Approval Authority act occurs" — precisely the Founder Approval act this standard names explicitly (§7.2). GOV-013 simply predates (2026-07-26) the first demonstrated instance of a symmetric, post-implementation Founder Acceptance act (EWO-016, 2026-07-28), which is why GOV-013 could not yet name it (§7.7).

**No stage defined by GOV-013 is redefined, restated, or given a competing schema anywhere in this document.** Where this standard's own text is silent on a stage's Purpose, Permitted Activities, Prohibited Activities, Required Outputs, or Entry/Exit Criteria, that silence is deliberate: the governing text is GOV-013's own, cited by section number, and nowhere else.

## 4. Terminology

This standard reuses GOV-013's own stage-definition vocabulary unchanged, for consistency across both documents: **Purpose; Permitted Activities; Prohibited Activities; Required Outputs; Entry Criteria; Exit Criteria** (GOV-013 §6's own schema, stated verbatim). Where this standard's own operational focus requires a field GOV-013's generic schema does not have, it adds exactly three, and no more: **Responsible Authority** (which GOV-003 role, and under which GOV-010 class, performs this stage — GOV-013 discusses authority narratively within Permitted Activities rather than as its own field); **Evidence Required** (what must be independently reproducible from repository state alone); **Repository Affected** (which of the two repositories this stage touches, and how). No other field name is introduced. No GOV-013 or STD-001 term is redefined or given an alternate meaning anywhere in this document.

## 5. Engineering Principles

The following are not new principles; each is already stated in GOV-001, GOV-002, GOV-004, or GOV-013 §9. They are listed here, attributed by number where GOV-013 §9 already states them, rather than re-derived in parallel prose:

1. **Evidence before decisions; truthful engineering history; independent verification; minimal correction scope** — GOV-013 §9 principles 1, 7, 8, 9, unchanged, applying identically to every stage this standard defines.
2. **Traceability** — GOV-013 §9 principle 4: every Engineering Work Order cites the architecture that authorizes it; every Engineering Report cites the Engineering Work Order it reports on.
3. **Publication only after approval, never inferred from mere existence** — GOV-013 §9 principle 10, applying identically to every publication point this standard names (§7.3).
4. **Repository discipline** (this standard's own addition — GOV-013 does not discuss repositories as a concept, since it predates the two-repository Runtime/Documentation split this standard's own stages must navigate). Each repository this lifecycle touches has a defined, bounded role (§7); a task governing one repository does not modify another except where this standard or the authorizing artifact explicitly permits it.
5. **Repeatability and auditability** (this standard's own addition, operationalizing GOV-013 §9 principle 8 for repository-evidence specifically). Every stage's required evidence is independently reproducible by a later party from repository state alone — commit hashes, diff statistics, test totals — never from an unreproducible private claim; every stage produces a durable, traceable record of what was decided, by whom, on what evidence, and when.

## 6. Lifecycle Overview

```text
        (GOV-013 §6.1–§6.12 — Idea through Engineering Work Order authoring;
         governed there in full; not restated, extended, or duplicated here)
                                │
                                ▼
              Engineering Work Order exists (GOV-013 §6.12's own Exit Criteria met)
                                │
                                ▼
              Independent Engineering Review (§7.1 — new)
                                │
                                ▼
              Founder Approval (§7.2 — new)
                                │
                                ▼
              Publication — invocation 1 of 3 (§7.3; GOV-013 §6.16)
                                │
                                ▼
              Implementation (§7.4; extends GOV-013 §6.13)
                                │
                                ▼
              Independent Implementation Review (§7.5; extends GOV-013 §6.14)
                                │
                                ▼
              Engineering Report (§7.6; extends GOV-013 §6.15)
                                │
                                ▼
              Publication — invocation 2 of 3 (§7.3; GOV-013 §6.16)
                                │
                                ▼
              Founder Acceptance (§7.7 — new)
                                │
                                ▼
              Publication — invocation 3 of 3 (§7.3; GOV-013 §6.16)
                                │
                                ▼
              Baseline Update (§7.8 — new)
```

Only four boxes in this diagram (Independent Engineering Review, Founder Approval, Founder Acceptance, Baseline Update) are stages this standard originates. Three (Implementation, Independent Implementation Review, Engineering Report) are GOV-013 stages this standard's own scope continues through, extended but not redefined. "Publication" is not three different stages — it is GOV-013 §6.16's own single stage, invoked three times because three distinct artifacts (the approved EWO, the Engineering Report, and the Founder Acceptance disposition) each require it in turn; it is defined once (§7.3) and referenced, not redefined, at each invocation.

The correction/re-review loop-back logic within this standard's own scope: Independent Engineering Review (§7.1) may require a Correction and Re-Review cycle (demonstrated: EWO-015's own two-round correction cycle) before Founder Approval; Independent Implementation Review (§7.5) may likewise require returning to Implementation (§7.4) — or, where the required change is architecture-level, to Architecture Review under GOV-013 §6.9 — before an Engineering Report may be authored.

## 7. Stage Definitions

### 7.0 Entry Precondition — Engineering Work Order Authoring (not a stage of this standard)

Governed entirely by GOV-013 §6.12. Not restated here. This standard's own evidence discipline (§10) requires only that, before Independent Engineering Review (§7.1) begins, the following is independently confirmed against the repository, not assumed: the EWO's own document identifier was derived from repository evidence at authoring time, never assumed in advance (STD-001 §7); the EWO cites its specific authorizing architecture sections, independently verified against the current architecture text; the EWO exists in the Documentation repository only — Runtime remains untouched and unmodified throughout authoring.

### 7.1 Independent Engineering Review

**Purpose.** Is the Engineering Work Order itself sound, internally consistent, and faithful to its authorizing architecture, before any implementation is authorized against it?

**Entry Criteria.** A drafted EWO exists (GOV-013 §6.12's own Exit Criteria met).

**Permitted Activities.** Independent, skeptical evaluation of the EWO against its cited architecture and the current repository state; classifying findings by severity (Critical / Major / Minor / Observation, GOV-013 §8.3); correcting the EWO directly, strictly to the findings identified, with a version increment, where correction is required.

**Prohibited Activities.** Approving an EWO with an unresolved Critical or Major finding; expanding the review into a redesign; silently absorbing a newly noticed issue outside the corrected scope without disclosing it as its own, separate finding.

**Required Outputs.** A classified findings list and exactly one recorded verdict.

**Exit Criteria.** Zero unresolved Critical or Major findings. Where correction is required, the EWO is corrected — applying only the findings identified (GOV-013 §8.4) — and re-reviewed narrowly (GOV-013 §8.5); this cycle repeats until zero unresolved Critical or Major findings remain (demonstrated: EWO-015's own review → correction → re-review → regression found → correction → focused re-review chain, fully disclosed in its own Revision History).

**Responsible Authority.** Independent Reviewer (GOV-003 §3.4); where no independent human reviewer exists, this is disclosed explicitly, never presented as independent human review (GOV-013 §8.1).

**Evidence Required.** Every finding independently re-derived from the EWO's own text and the current repository state — never accepted from the draft's own self-description; a version increment recording any correction (STD-001 §13), with a Revision History entry naming exactly what changed and why.

**Repository Affected.** Documentation only.

### 7.2 Founder Approval

**Purpose.** Does the reviewed Engineering Work Order become an effective, operative authorization to implement?

**Entry Criteria.** The EWO has exited Independent Engineering Review (§7.1) with zero unresolved Critical or Major findings.

**Permitted Activities.** `status`/`version` transition; Approval Status table completion; a Revision History entry recording the disposition.

**Prohibited Activities.** Approving an EWO carrying an unresolved Critical or Major finding; altering any technical requirement while recording approval; treating drafting, staging, committing, or pushing as itself this act (GOV-013 §6.16's own identical prohibition, applied here to the Approval act specifically).

**Required Outputs.** A recorded Founder Approval disposition.

**Exit Criteria.** `status` transitions from `Draft` to `Approved`; the document's own Approval Status table records the Approval Authority, the exact decision-class basis (GOV-010 §4–§5), and the date. GOV-013 §8.6 governs here directly: "a review's own 'Approved' wording never substitutes for" this act — Independent Engineering Review's own verdict (§7.1) is a recommendation; only this stage's own Approval Authority act makes the EWO operative.

**Responsible Authority.** Approval Authority (GOV-003 §3.5), ordinarily the Founder exercising Class E (Implementation) decision authority (GOV-010 §4) within already-approved architecture, per GOV-010 §5's delegation provision — disclosed explicitly where exercised directly by the Founder in the absence of an identified delegate (demonstrated identically across EWO-014, EWO-015, and EWO-016's own Approval Status tables).

**Evidence Required.** A dedicated version reserved exclusively for this governance disposition, changing no engineering-scope content (demonstrated: EWO-014 and EWO-015 both reserved a final version exclusively for this act; EWO-016 did so at 0.1.2, having already used an earlier version for its own Independent Engineering Review correction).

**Repository Affected.** Documentation only.

### 7.3 Publication

**Purpose.** Is an approved artifact now a durable, evidenced part of the repository? This is GOV-013 §6.16's own question, unchanged. This standard adds no new meaning to Publication — it identifies the three specific points within Engineering-Work-Order execution where this one, generic, already-defined stage is invoked.

**The three invocation points, within this standard's own scope:**

1. **After Founder Approval (§7.2)** — publishing the approved Engineering Work Order itself.
2. **After Engineering Report authoring (§7.6)** — publishing the Engineering Report.
3. **After Founder Acceptance (§7.7)** — publishing the acceptance disposition recorded on the EWO.

Each invocation is governed by the identical Entry/Exit Criteria, Permitted/Prohibited Activities, and Evidence Required stated once, here:

**Entry Criteria.** An artifact (an EWO, an ER, or an accepted EWO's own disposition) with an effective, already-recorded disposition (Founder Approval, completed authorship, or Founder Acceptance, respectively) requiring publication.

**Permitted Activities.** Staging, committing, and pushing exactly the authorized artifact for that invocation.

**Prohibited Activities.** Publishing an artifact without its own effective, already-recorded disposition; staging any unrelated file, including pre-existing repository drift; force-pushing.

**Required Outputs.** A pushed Documentation-repository commit.

**Exit Criteria.** The commit is present on `origin/main`; local and remote HEAD match; ahead/behind reads `0/0`.

**Responsible Authority.** Implementer or Author, acting on an already-effective disposition (GOV-003 §3.6) — publication is a mechanical act at every invocation, never a further decision.

**Evidence Required.** A truthful commit message; a staged-file list containing only the authorized artifact for that invocation; `git diff --check` clean; pre- and post-push HEAD comparison against `origin/main`.

**Repository Affected.** Documentation. Runtime remains untouched at every invocation.

### 7.4 Implementation

Governed by GOV-013 §6.13. Its Purpose ("Has the work been implemented?"), Permitted/Prohibited Activities, Required Outputs, and Entry/Exit Criteria are stated there and are not restated here. This standard adds only the following, which GOV-013's own generic schema does not itself specify:

**Evidence Required.** Repository verification before and after implementation (branch, HEAD, working-tree cleanliness, ahead/behind against origin); a specification-to-code trace mapping every EWO requirement to its realized location; full workspace verification (§11) passing from a forced-clean state; independently re-derived test totals, by direct summation, never asserted from a single aggregate figure alone.

**Repository Affected.** Runtime. Documentation remains unmodified by this stage — any Documentation change belongs to a separately governed task.

### 7.5 Independent Implementation Review

Governed by GOV-013 §6.14. Its Purpose ("Does the implementation faithfully satisfy the Engineering Work Order?"), Permitted/Prohibited Activities, Required Outputs, and Entry/Exit Criteria are stated there and are not restated here. This standard adds only the following:

**Evidence Required.** Direct re-inspection of the changed source, not the implementation's own description of it; an independent re-run of every mandatory verification and validation gate (§10, §11); a scope audit classifying every changed file as within or outside the EWO's authorized scope, with any scope creep reported as a finding (demonstrated: this exact discipline caught a genuine focused-test-count misreporting error during EWO-016's own Independent Implementation Review, corrected before publication in the resulting Engineering Report).

**Repository Affected.** Runtime is evidence only for this stage — read, never modified, unless a genuine defect is found, in which case any correction remains strictly within the authorizing EWO's own scope and is itself independently re-verified before the review concludes.

### 7.6 Engineering Report

Governed by GOV-013 §6.15. Its Purpose ("What evidence proves completion?"), Permitted/Prohibited Activities, Required Outputs, and Entry/Exit Criteria are stated there and are not restated here. This standard adds only the following:

**Evidence Required.** Exact repository baselines (both repositories, branch, HEAD, ahead/behind, working-tree state) at authoring time; per-file diff statistics; independently re-run workspace and focused verification (§10, §11); the Independent Implementation Review's findings, classified and reconciled (demonstrated: this exact re-derivation caught and corrected a misreported focused test count during ER-017's own authoring, rather than propagating it).

**Repository Affected.** Documentation only. Runtime remains evidence, unmodified.

**Additional Prohibited Activity, specific to this standard's own scope.** Claiming Founder Acceptance or closure within the ER itself — that is a distinct, subsequent act (§7.7).

### 7.7 Founder Acceptance

**Purpose.** Does the Founder accept the completed, evidenced implementation as final, closing the Engineering Work Order? This stage has no GOV-013 equivalent — GOV-013 (drafted 2026-07-26) predates the first demonstrated instance of it (EWO-016, 2026-07-28).

**Entry Criteria.** A published Engineering Report (§7.3 invocation 2; §7.6) recording zero unresolved Critical or Major findings from Independent Implementation Review.

**Permitted Activities.** `status` transition, Approval Status table completion, Disposition rewrite, Revision History entry.

**Prohibited Activities.** Accepting an implementation while any Critical or Major finding remains unresolved; reopening implementation, architecture, or any prior review's own conclusion; modifying the Engineering Report.

**Required Outputs.** A recorded Founder Acceptance disposition on the governing EWO.

**Exit Criteria.** The EWO's own `status` transitions to `Implemented` (STD-001 §12 — "optional status for specifications whose implementation is verified"; the first use of that already-defined status in this repository, chosen over an unregistered term precisely because STD-001 already defines it for this exact purpose). The EWO's own Approval Status table gains a Founder Acceptance row recording the acceptance and the implementation commit it accepts. The EWO's own Disposition section states plainly that it is closed and that no further work is authorized under it.

**Responsible Authority.** Approval Authority (GOV-003 §3.5), the same basis as §7.2 — Founder Acceptance and Founder Approval are related but distinct acts of the same authority, at different lifecycle points, exactly as GOV-010 §5 already distinguishes "authorship, review, recommendation, approval... are distinct acts."

**Evidence Required.** Independent re-confirmation, immediately before recording acceptance, that both repositories remain at their expected, evidenced state; that the Engineering Report's own findings genuinely contain zero unresolved Critical or Major items; a dedicated version reserved exclusively for this disposition, changing no engineering-scope content, on the identical "reserved pure-disposition version" basis §7.2 already establishes.

**Repository Affected.** Documentation only. Runtime remains unmodified and is cited only as historical evidence.

### 7.8 Baseline Update

**Purpose.** Does the now-closed Engineering Work Order's own implementation commit become the binding reference point all subsequently authored Engineering Work Orders must verify themselves against? This stage has no GOV-013 equivalent.

**Why this remains a distinct stage despite writing nothing to either repository.** Unlike every other stage in this document, Baseline Update performs no repository write of its own — it is not, in that narrow sense, an "action." It remains a distinct, mandatory stage because it establishes a binding obligation on all *future* engineering work: from the moment Founder Acceptance (§7.7) is recorded, every subsequently authored Engineering Work Order is required to independently re-verify the closed EWO's own recorded commit identities as its own starting baseline (§8), not merely to inherit them by assumption. Defining that obligation, and exactly what a baseline record must contain, is itself a normative act — establishing a rule that binds future conduct is a legitimate lifecycle stage even where it produces no artifact of its own, on the same basis GOV-013 §10 (Document Relationships) states a definitional relationship without itself constituting an action. This standard's own §8 (Baseline Management) is where the substantive content of this obligation lives; this stage exists in the lifecycle sequence only to mark the exact point — immediately after Founder Acceptance — at which that obligation begins to bind.

**Entry Criteria.** A Founder-accepted, closed EWO (§7.7), naming a specific Runtime implementation commit and a specific Documentation publication commit.

**Permitted Activities.** None in either repository. The only "activity" this stage authorizes is definitional: recognizing that the closed EWO's own commit identities are now the reference point future work must verify against (§8).

**Prohibited Activities.** Resetting either repository to a prior commit to "match" a stated baseline; treating a baseline citation as license to skip repository verification in the next engineering task; assuming a baseline is current without independently re-fetching and re-checking it.

**Required Outputs.** No new document and no repository write. The closed EWO's own recorded commit identities become the "expected baseline" every subsequently authored Engineering Work Order opens by citing and independently verifying — exactly the "Expected Runtime baseline" / "Expected Documentation baseline" fields every reviewed EWO and Engineering Report in this lifecycle's own history already opens with.

**Exit Criteria.** The next Engineering Work Order authored against this lifecycle correctly cites the newly closed EWO's own commit identities as its own starting baseline, independently re-verified — never assumed — at that future EWO's own repository verification (§10).

**Responsible Authority.** Author of the next Engineering Work Order (GOV-003 §3.3), verifying rather than assuming the baseline.

**Evidence Required.** A fresh `git rev-parse HEAD` / `git status --short` / `git fetch` verification, at the start of the next engineering task, confirming the cited baseline commit is genuinely the current repository state (or that any legitimate advance since is disclosed and investigated, never silently reset to).

**Repository Affected.** Both, as read-only reference.

## 8. Baseline Management

A **baseline** is the specific, named commit identity — in both the Runtime and Documentation repositories — that a Founder-accepted, closed Engineering Work Order (§7.7) leaves behind as the reference point every subsequently authored Engineering Work Order must independently verify itself against before proceeding (§7.8).

A baseline is established exactly once per closed Engineering Work Order, at the moment Founder Acceptance is recorded (§7.7) — never before, since the implementation commit it names is not final until that act occurs, and never retroactively, since STD-001 §4/§28's own immutability principle ("approved documents are not silently rewritten"; "approved documents must not have substantive history erased") forbids rewriting a prior baseline's own recorded identity after the fact.

A baseline record must state, at minimum: the Runtime implementation commit hash and its subject line; the Documentation commit hash at which the governing Engineering Report was published; the Documentation commit hash at which Founder Acceptance itself was recorded; and the exact Independent Implementation Review outcome (finding counts by severity) that Acceptance relied on.

Future engineering work begins from a baseline by independently re-verifying it, never assuming it: the next Engineering Work Order's own repository verification (§10) must confirm the cited baseline commit is genuinely the current repository state, or disclose and investigate any legitimate advance since — never reset either repository to force a match (§7.8's own Prohibited Activities).

## 9. Repository Responsibilities

**Runtime repository.** Holds only implementation source, tests, and examples. Modified exclusively during Implementation (§7.4) and, narrowly, during Independent Implementation Review (§7.5) where a genuine, in-scope defect is found. Evidence-only at every other stage. Never modified during a governance task (Founder Approval, Publication, Founder Acceptance, Baseline Update, or this standard's own authoring).

**Documentation repository.** Holds every controlled document this lifecycle produces: Engineering Work Orders (`work-orders/`) and Engineering Reports (`engineering-reports/`), per STD-001 §10's Repository Location Standard. A task governing one document family stages and commits only the files that family's own stage authorizes (§7) — never an unrelated document, and never pre-existing, previously disclosed repository drift.

**Engineering Work Orders and Engineering Reports.** Registered per STD-001 §46/§47; their own post-authoring lifecycle is what §7 of this standard defines. Their authoring itself, and every stage preceding it, remains GOV-013's own domain, cited here and nowhere restated.

## 10. Mandatory Verification

Before Independent Engineering Review begins (§7.1):

- Repository verification, both repositories: current branch, HEAD, working-tree cleanliness, ahead/behind against `origin/main`, and disclosure of any pre-existing, unrelated drift left deliberately untouched.
- Confirmation that the EWO under review genuinely satisfies GOV-013 §6.12's own Exit Criteria.

Before Implementation begins (§7.4):

- Repository verification, both repositories, repeated fresh — never relied on from an earlier stage in the same task if repository state could plausibly have changed since.
- Confirmation that the governing EWO carries an effective Founder Approval disposition (§7.2) and has been published (§7.3, invocation 1).
- A specification-to-code trace, mapping every EWO requirement to its intended implementation location, before any source file is edited.
- Baseline verification: the full validation suite (§11) run once, before implementation, to establish the exact pre-implementation state independently re-derived counts are compared against.

Before each Publication invocation (§7.3), Founder Approval (§7.2), and Founder Acceptance (§7.7):

- Independent re-verification that the artifact being published, approved, or accepted carries zero unresolved Critical or Major findings from its own governing review.
- Independent re-verification of both repositories' current state, immediately before the act.
- For Founder Acceptance specifically: independent confirmation that the cited Engineering Report and the cited Independent Implementation Review outcome are genuinely consistent with the current repository state, not merely restated from the Engineering Report's own text.

## 11. Required Validation

Before Founder Approval, Implementation completion, each Publication invocation, and Founder Acceptance, the following categories of validation are mandatory. The current Runtime toolchain's concrete commands are given as the present, disclosed realization of each category — not a permanent technology dependency:

- **Formatting conformance** — currently: `cargo fmt --all -- --check`.
- **Static analysis / lint conformance, warnings treated as failures** — currently: `cargo clippy --workspace --all-targets --all-features -- -D warnings`.
- **Full workspace build** — currently: `cargo build --workspace --all-targets --all-features`.
- **Full workspace test execution, with an independently re-derived total** (by direct summation across every test binary, never asserted from a single aggregate figure alone) — currently: `cargo test --workspace --all-targets --all-features`.
- **Whitespace/diff hygiene** — currently: `git diff --check`.
- **Repository-state verification** — `git status --short`, `git branch --show-current`, `git rev-parse HEAD`, `git fetch origin`, `git rev-list --left-right --count HEAD...origin/main`, both repositories, at every stage boundary named in §10.
- **Published-specification verification** — independent confirmation, from the artifact's own frontmatter and Approval Status table (or, for constitutional-tier documents using the evidence-commit model, independent recomputation of the cited hash/blob-ID/byte-size/line-count against a separate evidence commit — GOV-013's own worked example), that the specification being implemented, reviewed, or accepted is genuinely in the state assumed, never taken on faith.
- **Engineering Report re-derivation** — every figure an Engineering Report states (test totals, diff statistics, finding counts) independently re-run and re-computed at that report's own authoring time, never copied forward from an earlier draft or claim.
- **Acceptance-time re-confirmation** — the full validation suite above, re-run or its most recent passing result independently re-verified as still current, before Founder Acceptance is recorded.

## 12. Engineering Authorities

- **Founder (GOV-003 §3.1).** Retains final authority for Class A decisions; exercises Class B interim authority in the absence of an appointed Chief Architect (GOV-003 §3.2); is, in this project's current, disclosed state, also the Approval Authority (GOV-003 §3.5) exercising the Class E decision authority (GOV-010 §4) this standard's Founder Approval (§7.2) and Founder Acceptance (§7.7) stages require — a delegable authority (GOV-010 §5), exercised directly only in the disclosed absence of an identified delegate.
- **Independent Reviewer (GOV-003 §3.4).** Conducts Independent Engineering Review (§7.1) and Independent Implementation Review (§7.5); discloses independence or its disclosed absence in every case; re-derives every finding from source, never from a predecessor's own report.
- **Implementer (GOV-003 §3.6).** Carries out only currently-effective decisions; implements strictly within EWO scope (§7.4); escalates rather than resolves any architecture-level question unilaterally.
- **Acceptance Authority.** The same Approval Authority (GOV-003 §3.5) exercising Founder Acceptance (§7.7) — a distinct act from Founder Approval (§7.2), at a later lifecycle point, requiring its own independent evidence rather than being inferred from the earlier approval.

**Decision authority is never reclassified by this standard.** Whether a specific change is Class B (Architectural) or Class E (Implementation) is determined by GOV-010 §4 alone (GOV-013 §11's identical principle); this standard states which stage's own act corresponds to which already-defined class, and introduces no new class.

## 13. Engineering Rules

1. Architecture before implementation. No Engineering Work Order may authorize implementation against architecture that has not itself exited Architecture Review with an effective Approved disposition (GOV-013 §6.12's own Entry Criteria).
2. No implementation without an approved, published Engineering Work Order (GOV-004 §1; §7.2, §7.3 above).
3. No Runtime modification during a governance-only task (Independent Engineering Review, Founder Approval, Publication, Founder Acceptance, Baseline Update, or this standard's own authoring) — Runtime is evidence only at every stage except Implementation (§7.4) and, narrowly, Independent Implementation Review (§7.5).
4. No undocumented engineering work. Every Runtime change traces to a specific, cited Engineering Work Order (GOV-004 §11; GOV-013 §9 principle 4).
5. No engineering outside an approved EWO's own stated scope, at any stage (§7.4, §7.5).
6. Independent review is required before Founder Approval (§7.1) and before an Engineering Report may be authored (§7.5) — its absence, where genuinely unavailable, is disclosed, never concealed or implied.
7. Truthful engineering history throughout: a correction discloses what was wrong and what changed; a misreported figure is corrected and the correction itself disclosed, never silently absorbed (demonstrated: §7.6's own worked example).
8. Evidence over assumptions at every stage boundary (§10): a document's identifier, version, status, and content are independently re-verified from the repository, never assumed from a prior task's memory of them.
9. Minimal authorized change: a correction pass, at any stage, resolves exactly what its governing review identified (§7.1's own Exit Criteria).
10. Publication, Founder Approval, and Founder Acceptance are each distinct acts; none is inferred from another, and none is inferred from the mere existence, staging, committing, or pushing of a document (§7.2, §7.3, §7.7; GOV-013 §6.16's identical principle).

## 14. Change Control

This standard follows STD-001 §13's versioning rule: PATCH for editorial correction, MINOR for backward-compatible addition or clarification, MAJOR for a change to obligation, stage ordering, or authority assignment. This revision (0.1.0 → 0.2.0) is itself MAJOR, correcting a scope and stage-ordering defect an Independent Engineering Review identified (§17).

A finding against this standard, once Founder-approved, is corrected via the identical discipline §7.1 already establishes for an Engineering Work Order: a review identifies specific, nameable findings; a correction pass resolves exactly those findings; a narrow re-review confirms resolution without new drift. Which decision class governs a proposed change to this standard is determined by GOV-010 §4, not by this document (identical to GOV-013 §11's own rule).

This standard does not retroactively reclassify, invalidate, or require rework of any engineering work completed before this standard's own effective date (GOV-010 §21's prospective-application principle; GOV-013 §12's identical conformance rule) — including the EWO-014, EWO-015, and EWO-016 lifecycles this standard was itself derived from observing.

## 15. Compliance

All future Engineering Work Orders, and every stage named in §7, must comply with this standard from the point it becomes effective (GOV-010 §21) forward, unless superseded by a later approved standard. Where a future capability's own scale genuinely does not warrant a given stage's full formal weight, that proportionality is itself part of conformance, provided it is disclosed exactly as such — never silently assumed (GOV-013 §12's identical principle, adopted here without change).

## 16. References

Internal:

- GOV-003 — Governance Model
- GOV-004 — Engineering Principles
- GOV-010 — Decision Framework
- GOV-013 — Engineering Lifecycle (Approved via Normal-Governance disposition, evidence commit `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6`) — the sole authority for every stage through Engineering Work Order authoring, and for Implementation, Independent Implementation Review, Engineering Report, and Publication as generic concepts
- STD-001 — Documentation Standards (§1, §4, §7, §10, §12, §13, §28, §46, §47)
- ADR-0011 — Bootstrap Approval Authority
- ADR-0012 — Corrective Founder Approval Evidence Record basis

Source evidence (independently re-verified during this standard's own preparation and its revision, not restated from memory):

- `governance/GOV-013-Engineering-Lifecycle.md` and its evidence commit `3ac7cfb2e953004e891586e5ffef31e5d4fc87a6`, including direct side-by-side comparison of GOV-013 §6.1–§6.16 against this standard's own v0.1.0 text, which is what identified the two MAJOR findings this revision corrects
- `architecture/ARCH-008-Effect-Runtime-Architecture.md` v0.4.3 and its approval commit `2bd2595912f3ba1acf30d80684501c95bc4903fd`
- `work-orders/EWO-014-Provider-Idempotency-Registration.md` (commit `1c51f5e`) and `engineering-reports/ER-015-Provider-Idempotency-Registration.md` (commit `b1439cb`)
- `work-orders/EWO-015-Retry-Architecture-Implementation.md` (commit `3ca11f2`) and `engineering-reports/ER-016-Retry-Architecture-Implementation.md` (commit `1c50d55`)
- `work-orders/EWO-016-ConstraintSet-Based-Retry-Policy.md` (Founder Approval commit `1038e8a`; Runtime implementation commit `29ea55ced6348490f90bd7baeb08d3d4705f19ab`; Founder Acceptance commit `a316874`) and `engineering-reports/ER-017-ConstraintSet-Based-Retry-Policy.md` (commit `9ea9fa6`)
- `standards/STD-001-Documentation-Standards.md` §1, §4, §7, §10, §12, §13, §28, §46, §47 (read directly, not paraphrased from memory)
- `standards/STD-002-Coding-Standards.md` (read directly to confirm the identifier collision this standard's own filing note discloses)

## 17. Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Denver Jacobs (AI-assisted) | Initial Draft. Attempted to formalize the Engineering-Work-Order-centered segment of the lifecycle at the standards tier, citing GOV-013 throughout. |
| 0.2.0 | 2026-07-28 | Denver Jacobs (AI-assisted) | MAJOR revision, correcting two MAJOR findings from Independent Engineering Review IER-STD-031 (`RETURN TO AUTHOR FOR REVISION`). **F01 (duplicate lifecycle definitions) eliminated architecturally, not cosmetically**: v0.1.0 fully restated five GOV-013 stages (Engineering Work Order §6.12, Publication §6.16, Implementation §6.13, Independent Implementation Review §6.14, Engineering Report §6.15) — three with verbatim-identical Purpose text — while explicitly claiming not to. This revision removes Engineering Work Order authoring from this standard entirely (now a cited entry precondition, §2, §7.0, governed solely by GOV-013 §6.12); converts Implementation, Independent Implementation Review, and Engineering Report (§7.4–§7.6) from full restatements into short addenda that state only fields GOV-013's own schema lacks (Evidence Required, Repository Affected), explicitly deferring Purpose/Permitted/Prohibited/Required-Outputs/Entry-Exit-Criteria to GOV-013 by citation; and adopts GOV-013's own field vocabulary (Entry Criteria, Required Outputs, Permitted/Prohibited Activities) throughout (also resolving F04, terminology inconsistency), reserving only three new fields (Responsible Authority, Evidence Required, Repository Affected) with disclosed justification (§4). **F02 (Publication conflict) eliminated by reconciliation, not renaming**: Publication is no longer presented as this standard's own competing stage placed mid-lifecycle. It is defined exactly once (§7.3) as GOV-013 §6.16's own generic stage, invoked at three explicitly named points (after Founder Approval, after Engineering Report, after Founder Acceptance) rather than redefined three times or once in a conflicting position. §6's Lifecycle Overview diagram now shows this explicitly. Only four stages remain fully defined by this standard, because only four have no GOV-013 equivalent at all: Independent Engineering Review of the EWO (§7.1), Founder Approval (§7.2), Founder Acceptance (§7.7), and Baseline Update (§7.8) — Baseline Update's own no-repository-write nature is now explicitly explained (§7.8) rather than left as an unresolved oddity, and its substantive content is consolidated into the pre-existing §8 Baseline Management rather than duplicated across two sections. Title updated to disclose the narrowed scope ("Engineering Work Order Lifecycle") without renaming the document identifier or file, per STD-001 §7's identifier-permanence rule. F03 (an incorrect internal cross-reference in v0.1.0) is moot: the sentence it appeared in no longer exists in this restructured form. No architecture, implementation, governance role, or decision class is redefined; no new controlled-document family is registered; nothing GOV-013 or STD-001 already governs is restated. |
| 0.2.1 | 2026-07-29 | Denver Jacobs (AI-assisted) | PATCH correction, resolving the single MINOR finding from Independent Engineering Re-Review IER-STD-031-R2 (`Approve with minor corrections`; 0 CRITICAL, 0 MAJOR, 1 MINOR, 1 OBSERVATION). **IER-STD-031-R2-F01 (Publication self-description over-claim) corrected**: §2 and §3 previously stated this standard "states nothing about Publication that GOV-013 §6.16 does not already establish" — measurably too absolute, since §7.3 legitimately adds EWO-lifecycle-specific operational application and evidence requirements (the three named invocation points, their Entry/Exit Criteria, and Evidence Required) that GOV-013 §6.16's own generic text does not itself state. §2 and §3 are corrected to state plainly that GOV-013 §6.16 remains the sole authoritative definition of Publication, that this standard adds only operational application and evidence requirements for invoking it at three named points, and that each invocation is a specialization of GOV-013's single act, never a second definition. §7.3 itself is unchanged — the re-review's own recommendation was to correct the absolute wording rather than restructure an already-correct, non-conflicting stage definition. No normative meaning changes: Publication was, and remains, defined exactly once, governed by GOV-013 §6.16, invoked at the same three points. **IER-STD-031-R2-O01 (original-review observation disposition) recorded**: the original IER-STD-031 review (v0.1.0) also recorded three OBSERVATION-level findings alongside its two MAJOR findings. No standalone record of IER-STD-031 exists in this repository independent of this standard's own citation of it, so their specific content cannot be independently reconstructed at this revision. What can be truthfully stated: the v0.2.0 architectural revision was a comprehensive restructuring of the entire document, not a narrow patch confined to the two MAJOR findings, so any observation concerning duplicated content, terminology, or cross-references was necessarily subsumed by that restructuring; no observation was identified, during v0.2.0's authoring or this v0.2.1 correction, as requiring a separate blocking correction; and no observation is known to have been deliberately declined. Any such improvement not explicitly traceable to this Revision History remains non-normative and does not bind future revisions. Founder Approval recorded (§18). |

## 18. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted (v0.1.0), Revised (v0.2.0), Corrected (v0.2.1) | 2026-07-29 |
| Independent Engineering Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | v0.1.0: `IER-STD-031 FAILED — RETURN TO AUTHOR FOR REVISION` (two MAJOR findings, F01 and F02, both addressed by the v0.2.0 revision — see §17); v0.2.0 re-review: `IER-STD-031-R2 COMPLETE — READY FOR FOUNDER APPROVAL` (0 CRITICAL, 0 MAJOR, 1 MINOR — corrected by the v0.2.1 revision, see §17 — 1 OBSERVATION, disposition recorded in §17; recommendation: Approve with minor corrections) | 2026-07-29 |
| Approval Authority | Denver Jacobs, Founder, exercising Class E (Implementation) decision authority under GOV-010 §4–§5 in the absence of an identified delegate, per GOV-010 §5's delegation provision (no Chief Architect currently appointed) | **Approved** — minor correction (IER-STD-031-R2-F01) completed and incorporated in v0.2.1 prior to this approval, per the Founder's approval condition | 2026-07-29 |

This standard is now **Approved** (v0.2.1), following Founder Approval recorded directly above, on the identical "ordinary, mutable Approval Status convention" this repository's engineering-tier documents use throughout (demonstrated: EWO-014, EWO-015, and EWO-016's own Approval Status tables, each populated in place rather than left blank behind a separate evidence commit). This is distinct from GOV-013's own state (§3, related_documents above): GOV-013 is a governance-tier document using the exact-byte-identity evidence-commit convention (ADR-0011 §14–§15), under which its own tracked text remains `status: Draft` regardless of its true, separately evidenced disposition. STD-031, like the Engineering Work Orders and Engineering Reports whose demonstrated practice it formalizes, records its true disposition directly in its own tracked bytes — this document's `Approved` status is exactly what it appears to be, verifiable from this file alone with no separate evidence commit required.
