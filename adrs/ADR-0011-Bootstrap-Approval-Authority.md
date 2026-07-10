---
document_id: ADR-0011
title: Bootstrap Approval Authority
version: 0.1.0
status: Draft
decision_owners:
  - Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Denver Jacobs, Founder of SynapseOS — proposed Act 1 Founding Act; not yet exercised
created: 2026-07-10
last_updated: 2026-07-10
classification: Public
related_documents:
  governance:
    - GOV-003 (Draft)
    - GOV-010 (Draft)
  standards:
    - STD-001 (Draft — explicitly excluded from this ADR's scope; see §9)
  architecture: None
  rfcs: None
  adrs: None
  roadmap: None
  research: None
  operational: None
  source_artifacts: None
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# Bootstrap Approval Authority

*Filename pattern: `ADR-0011-Bootstrap-Approval-Authority.md` (four-digit sequence number, per STD-001 §7 and §8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§32, AI-Assisted Documentation). AI output is not automatically authoritative.

> **Status notice:** This ADR is **Draft**. No approval act described in this document has occurred. Drafting, saving, staging, committing, pushing, merging, or publishing this ADR does not itself constitute Act 1, Act 2, or any approval of any document. Document lifecycle status (this field) and Bootstrap Authority State (§11–§13) are two separate dimensions — see §11. As of this revision, Bootstrap Authority State is `Not Established`. See §35 (Approval Status).

## 1. Context

SynapseOS's governance corpus currently has no ratified internal approval authority. GOV-003 (Governance Model) and GOV-010 (Decision Framework) are the documents intended to define who may make and approve which decisions, but both remain Status: Draft — by STD-001's own definition, "work in progress and non-authoritative." No document anywhere in this repository has reached Approved status. No reviewer is named on any document. No document explicitly and validly grants anyone the authority to change that state.

This creates a bootstrap paradox: the documents needed to establish normal governance cannot themselves be approved without some approval authority already existing, and no such authority currently exists in ratified form.

## 2. Problem

The project needs a way to move GOV-003 and GOV-010 out of Draft status and into genuine, operative effect, so that normal governance can begin — without pretending that any currently Draft document is already binding, without inventing authority that isn't there, and without creating a mechanism broad or permanent enough to function as general Founder override power. This ADR exists solely to solve that narrow problem.

## 3. Source-of-Authority Theory

No ratified internal approval authority currently exists within this repository. Draft GOV-010 is not binding authority. Draft STD-001 is not binding authority. Git commit authorship proves contribution activity only — it does not prove governance authority, and Git contribution history does not prove ownership.

Denver Jacobs's Founder status is treated in this ADR as a binding project fact from the project continuity record, not as something inferred from Git history or derived from any Draft document's content.

> No ratified internal approval authority currently exists within this repository. Denver Jacobs, as Founder of SynapseOS, may perform a narrowly scoped, explicitly disclosed, temporary Founding Act solely to establish this ADR as effective. This mechanism does not derive from Draft GOV-010, Draft STD-001, Git authorship, contribution history, or inferred repository ownership.

This ADR does not assert any legal claim (ownership, corporate authority, or otherwise) beyond the factual, disclosed act described here. It creates no permanent or general Founder authority.

## 4. Decision Drivers

- No ratified approval authority currently exists anywhere in the repository.
- GOV-003 and GOV-010 cannot honestly be treated as binding while Draft.
- A mechanism is needed that is traceable, disclosed, and does not overstate its own authority.
- The mechanism must be narrow enough that it cannot be mistaken for, or misused as, general Founder override power.
- The mechanism must define its own end: it must not become a permanent alternative to normal governance.
- The mechanism's operational state must be traceable from repository evidence, not only from prose interpretation.

## 5. Considered Options

1. Do nothing; rely on implicit, undocumented Founder authority.
2. Treat Draft GOV-010 as though it were already binding authority.
3. Treat Draft STD-001 as though it were already binding authority.
4. Bootstrap-approve the full, previously-considered fifteen-document foundational set (GOV-001/003/004/006/010, STD-001, ADR-0001/0002/0003, RFC-0001/0002/0003/0010/0014, ARCH-000).
5. Bootstrap-approve STD-001 first, ahead of GOV-003/GOV-010.
6. Widen this ADR's authority to also cover Draft-to-Review metadata transitions, not only Approval.
7. Establish permanent, general Founder override authority.
8. Wait indefinitely for an independent human reviewer to be appointed before taking any action.
9. Narrow this ADR's scope to exactly GOV-003 and GOV-010, and no other document. **(selected)**

## 6. Decision

Establish a narrowly scoped, temporary, two-act bootstrap mechanism, defined and bounded entirely by this ADR, with an explicit, separately-tracked **Bootstrap Authority State** (§11–§13) distinct from this document's own lifecycle status:

- **Act 1 (Founding Act):** a one-time, non-reusable act, performed by Denver Jacobs as Founder of SynapseOS, applying solely to this document, establishing this ADR as effective.
- **Act 2 (Bounded Bootstrap Approval Authority):** an authority defined and bounded by this ADR, exercisable only after Act 1, applying to exactly two documents — GOV-003 and GOV-010 — and to no other document, decision, or act.

This ADR does not select or imply a broader authority model. It does not bootstrap-approve STD-001 or any document other than GOV-003 and GOV-010. It defines, in advance, an objective, non-discretionary mechanism by which it terminates once normal governance is demonstrated operative — where "demonstrated operative" means the normal-governance mechanism can issue a legitimate, authorized, traceable disposition, not that any specific document must be approved (§18).

## 7. Act 1 — Founding Act

Act 1 applies only to this document (ADR-0011). It is the act, if and when performed, by which Denver Jacobs, as Founder of SynapseOS, establishes this ADR as effective. Act 1:

- applies only to ADR-0011 and to no other document;
- is antecedent to Act 2 — Act 2 cannot exist or be exercised until Act 1 has occurred;
- does not derive its authority from ADR-0011, because ADR-0011 is not yet effective at the moment Act 1 occurs — Act 1's authority derives solely from the source-of-authority theory in §3;
- is one-time and non-reusable;
- MUST NOT approve GOV-003;
- MUST NOT approve GOV-010;
- MUST NOT approve any other document;
- MUST NOT authorize a production release;
- MUST NOT accept security risk;
- MUST NOT approve architecture or implementation;
- MUST NOT be cited, by this document or any future document, as general or permanent Founder override authority.

**None of the following constitutes Act 1:** drafting ADR-0011; saving ADR-0011; staging ADR-0011; committing ADR-0011; pushing ADR-0011; merging ADR-0011; publishing ADR-0011. Act 1 requires its own explicit, disclosed approval act and evidence record (§13), distinct from any of these ordinary Git or editorial operations. As of this Draft, Act 1 has not been exercised, and Bootstrap Authority State remains `Not Established` (§12).

## 8. Act 2 — Bounded Bootstrap Approval Authority

Act 2 exists only after ADR-0011 becomes effective through Act 1, and only while Bootstrap Authority State is `Active` (§12). It is defined and bounded entirely by this ADR's own text.

**Act 2 applies to exactly two documents, and no others:**

- `GOV-003` (Governance Model)
- `GOV-010` (Decision Framework)

**Act 2 MUST NOT apply to, and confers no authority whatsoever over:**

- STD-001 (see §9 for the explicit reason)
- GOV-001
- GOV-004
- GOV-006
- ADR-0001
- ADR-0002
- ADR-0003
- RFC-0001
- RFC-0002
- RFC-0003
- RFC-0010
- RFC-0014
- ARCH-000
- ARCH-001 or any future architecture document
- any future ADR
- any future RFC
- any future GOV document
- any future standard
- source code of any kind
- implementation-conformance claims
- production release authorization
- deployment authorization
- security-risk acceptance
- financial authority
- legal authority of any kind
- authority to change any document's status from Draft to Review (see §10)
- authority to edit or author document content (see §22.3)

No phrase in this document such as "foundational documents" or "in-scope documents" may be read as expanding this list. The two-document enumeration above is exhaustive and exact.

## 9. STD-001 Exclusion

STD-001 is deliberately and explicitly excluded from Act 2. This is based on direct inspection of STD-001's own text:

- STD-001 §2 (Scope) states it applies to "all official SynapseOS governance documents, standards, architecture specifications, RFCs, ADRs, roadmaps, research records, engineering playbooks, operational procedures, templates, meeting records, decision records and repository-level documentation" — an explicit, repository-wide scope.
- STD-001 §30 (Approval Rules) states: "Approval authority depends on document family and project maturity. Until delegated otherwise, the Founder retains final project authority while technical approval may be assigned to designated reviewers."
- STD-001 §6 (Document Hierarchy and Precedence) states that "approved standards constrain architecture and implementation," placing approved Standards above ADRs in precedence.

Read together, if STD-001 were bootstrap-approved under this ADR, §30's Founder-authority default would — by STD-001's own repository-wide scope declaration — appear to activate immediately across every controlled document family in this repository, not merely GOV-003 and GOV-010. Because STD-001, once Approved, would sit above this ADR in STD-001's own declared precedence order, this ADR could not validly narrow or subordinate that effect after the fact.

Therefore:

- STD-001 remains outside Act 2's scope, now and permanently under this ADR.
- Draft STD-001, including §30, is treated in this document as **contextual design evidence only** — it informs why a bootstrap mechanism of this shape is reasonable, but it is not treated as current binding authority.
- STD-001 is **not** bootstrap-approved by this ADR, in whole or in part.
- STD-001 should be reviewed and, if appropriate, approved later, under normal governance, once GOV-003 and GOV-010 are genuinely operative — at which point a real, structured authority model will already exist to interpret and apply §30 sensibly, rather than §30 being the only authority in force.

This ADR does not claim to override an Approved STD-001. STD-001 is not Approved, and this ADR does not make it so.

## 10. Review and Evidence Distinctions

This ADR distinguishes several concepts that must not be conflated:

- **Informal critique** (reading, commenting, architectural criticism, AI-assisted consistency checking) requires no bootstrap authority and may occur at any time, on any document.
- **This ADR does not grant Review-transition authority.** Who may change a document's status field from Draft to Review remains an open, unresolved question in this repository (no text anywhere explicitly grants it), and this ADR does not attempt to resolve it. That gap is recorded as an open question (§32.1) and as an explicit Non-Goal (§32.2), and is deliberately left outside this ADR's scope.
- Formal `Review` status is not assumed merely because review activity occurred on a document.
- **Bootstrap Approval** (an act performed under Act 2) is not the same as, and does not equal:
  - independent human review — no independent reviewer currently exists anywhere in this repository;
  - technical validation of a document's content — a document may be well-validated without being formally approved, or vice versa;
  - implementation readiness — an Approved RFC or ADR does not mean corresponding code exists or is ready.
- **Normal-governance disposition** is a distinct concept from bootstrap Approval: it is an act performed under GOV-003/GOV-010's own, eventually-operative provisions, and cannot occur for anyone until those provisions are themselves operative. A normal-governance disposition **may be Approval, Rejection, Deferral, Return for Revision, or another lifecycle-compatible outcome the operative governance text permits** — it is not limited to Approval (see §18).
- ADR Approval does not automatically validate the underlying technology choices it records; it records that a governance act occurred, not that the technical content has been independently verified correct.

This ADR does not invent an independent reviewer, and does not represent AI-generated critique as independent human review.

## 11. Document Lifecycle Status vs. Bootstrap Authority State

**ADR-0011's document lifecycle status (the `status:` field, governed by STD-001 §12) and this ADR's Bootstrap Authority State (defined in §12, governed entirely by this ADR) are two separate dimensions and MUST NOT be conflated.**

This document's `status:` field uses only STD-001's eight repository-wide lifecycle terms (Draft, Review, Approved, Implemented, Superseded, Deprecated, Archived, Rejected) and reflects ADR-0011's own document lifecycle exactly as it would for any other controlled document — it does not, and structurally cannot, represent whether Act 1 has occurred or what state Act 2 is in.

Bootstrap Authority State is a separate, ADR-0011-specific operational state machine (§12), **derived** (§13) — not separately stored inside this document — from evidence recorded in GOV-003's, GOV-010's, GOV-004's, and this ADR's own Approval Status tables, that tracks specifically whether the mechanism this ADR defines is dormant, active, suspended, reactivated, or permanently terminated. A reader who wants to know whether Act 1 or Act 2 has occurred must apply §13's derivation rule to those Approval Status tables, not consult this document's `status:` field.

## 12. Bootstrap Authority State Machine

Bootstrap Authority State takes exactly one of five values at any time: `Not Established`, `Active`, `Suspended`, `Reactivated`, `Terminated`.

### 12.1 State: Not Established

Applies while ADR-0011 is Draft and Act 1 has not occurred. Act 2 does not exist. No bootstrap approval authority exists in any form. **This is the current state as of this revision.**

### 12.2 Transition: Act 1

A valid, one-time Founding Act concerning ADR-0011 occurs, evidenced per §13 as a separate, subsequent, content-non-mutating repository commit — never as an edit to ADR-0011's own tracked content — created and pushed to the canonical remote. Consequence: ADR-0011 becomes effective according to the documented Act 1 evidence; Act 1 becomes permanently exhausted; Bootstrap Authority State becomes `Active`. Drafting, saving, staging, committing, pushing, merging, or publishing this document does not, by itself, satisfy this transition (§7); nor does any commit that lacks the evidence fields required by §14 or the message convention required by §13.

### 12.3 State: Active

Act 2 may act only on GOV-003 and GOV-010, only under this ADR's constraints (§8).

### 12.4 Transition: Both Valid Act 2 Approvals Exist

When valid bootstrap Approval records (§14–§15) exist for both GOV-003 and GOV-010, Bootstrap Authority State becomes `Suspended`, non-discretionarily, effective immediately upon the later of the two records becoming effective (§17).

### 12.5 State: Suspended

No new Act 2 approval may occur. Normal governance is primary. A GOV-004 validation disposition (§18) is attempted.

### 12.6 Transition: Successful Validation Disposition

A validation disposition satisfying §19's success criteria occurs. Bootstrap Authority State becomes `Terminated`, permanently.

### 12.7 Transition: Complete Structural-Failure Predicate

Only if the exact structural-failure predicate (§20) is fully satisfied and traceably documented (§21): Bootstrap Authority State becomes `Reactivated`, as a non-discretionary consequence of this ADR's predeclared rule — not as a new discretionary Founder decision.

### 12.8 State: Reactivated

Authority is narrower than the original `Active` state. It may operate only to approve corrected, committed content for GOV-003, GOV-010, or both, only where the immutable failure evidence demonstrates that the specific document contributes to the documented structural defect (§22). It does not grant authority to author or edit content (§22.3).

### 12.9 Transition: Corrective Approval Complete

After a valid corrective bootstrap Approval is recorded: Bootstrap Authority State returns to `Suspended`, and the validation act (§18) is retried against the corrected mechanism.

### 12.10 State: Terminated

Permanent. Act 2 cannot reactivate from `Terminated`. Act 1 cannot recur. ADR-0011 cannot be cited to recreate bootstrap authority of any kind. Normal governance is the sole approval path, for the two documents this ADR ever touched and for every other document in the repository.

## 13. Authoritative State Determination

Bootstrap Authority State is **derived, not stored**, and **ADR-0011's own tracked content is never modified after this Draft is finalized, for any reason, including recording Act 1.** This preserves this repository's exact-byte content-identity model (§14) without exception, rather than excepting any section of this document from it — a semantic label (such as calling a section "non-normative") does not and cannot make cryptographically different content identical, and no canonicalization or exclusion-zone hashing scheme is defined or invented here.

**Act 1 evidence model:** Act 1 approves ADR-0011's content exactly as committed at the time of the act (the "approved identity"). Evidence of Act 1, if it occurs, is recorded in a **separate, subsequent repository commit that does not modify ADR-0011's tracked file at all** — an empty commit (a standard, documented Git operation; this repository has no hook, policy, or convention prohibiting it) whose commit message contains, at minimum, every field required by §14: exact document ID, exact path, exact document version, the exact commit hash and SHA-256 content fingerprint of the approved identity, approver identity, approval type, self-approval disclosure, rationale, known limitations, and effective date. Because the approved identity's commit hash and content fingerprint already exist and are fixed before this evidence commit is made, no field predicts a future value, and no self-reference occurs.

This ADR's own Approval Status table (§35) contains, for its "Act 1" row, a **static statement that is true both before and after Act 1 occurs** — not a row that is filled in or otherwise mutated. It directs a reader to this repository's commit history, searchable by a fixed, defined convention (a commit message containing the literal string `ADR-0011 Act 1 Evidence Record`), rather than claiming within ADR-0011's own bytes that Act 1 has or has not occurred. This is a deliberate, narrow, disclosed departure from the otherwise-standard mutable Approval Status row convention used elsewhere in this corpus, justified specifically because ADR-0011 is the one document in this mechanism whose own approval evidence would otherwise have to be recorded inside the very artifact being approved. GOV-003, GOV-010, and GOV-004 are not self-referential in this way; their own Approval Status tables continue to use the ordinary, mutable row convention for Act 2 and validation evidence.

**Act 1 effectiveness moment:** Act 1 is effective only once its evidence commit both (a) exists and (b) has been pushed to this repository's canonical remote — a purely local, unpushed commit is not yet repository-resolvable and does not make Act 1 effective. Bootstrap Authority State remains `Not Established`, and Act 2 cannot become `Active`, until both conditions hold.

Bootstrap Authority State at any time is the deterministic result of the following rule, evaluated top to bottom against: (i) this repository's pushed commit history, searched for a valid Act 1 evidence commit per the convention above; and (ii) the Approval Status tables of GOV-003, GOV-010, and GOV-004 (each following the ordinary, mutable Approval Status convention already used throughout this repository):

| Condition (first match applies) | Bootstrap Authority State |
|---|---|
| No valid, evidenced, pushed Act 1 evidence commit exists | `Not Established` |
| A valid, pushed Act 1 evidence commit exists, and fewer than two of {GOV-003, GOV-010} have a valid Act 2 bootstrap-Approval entry in their own Approval Status table | `Active` |
| Both GOV-003 and GOV-010 have a valid Act 2 bootstrap-Approval entry, and no validation disposition of GOV-004 (§18) satisfying §19 has been recorded, and no unresolved structural failure (§20) has been recorded | `Suspended` |
| Both GOV-003 and GOV-010 have a valid Act 2 bootstrap-Approval entry, and the *first* validation disposition of GOV-004 satisfying §19 has been recorded (§19's first-valid-disposition rule) | `Terminated` |
| Both GOV-003 and GOV-010 have a valid Act 2 bootstrap-Approval entry, a structural failure per §20 has been evidenced, and no successful corrective approval or subsequent qualifying validation disposition has since been recorded | `Reactivated` |

A "valid, evidenced" entry or commit means one satisfying §14's Approval Evidence Identity Model and traceable to a specific, pushed repository commit — never an assertion made only in prose.

This mechanism requires no external system, no new repository file, and no new controlled-document family: it reuses this repository's existing commit mechanism (already used throughout this engagement) and the Approval Status table convention already present in every document in this corpus — adapted, for ADR-0011's Act 1 row only, to a static, non-mutating form for the specific, disclosed reason above. **ADR-0011's own tracked content — including its normative rules and its Approval Status table — is never modified after this Draft is finalized, for any reason.** A future citation of "ADR-0011" always references this single, fixed, forever-unchanging content identity.

**As of this revision, no Act 1 evidence commit exists in this repository. Bootstrap Authority State is therefore `Not Established`.**

## 14. Approval Evidence Identity Model

Every Act 1 act, Act 2 act, validation disposition, or Bootstrap Authority State transition performed under this ADR MUST be recorded with immutable evidence including at minimum:

- exact document ID
- exact repository-relative path
- exact document version
- exact repository commit hash
- exact content fingerprint: **SHA-256 computed from the exact committed file bytes**, as stored in the Git blob for the specific commit and path referenced — UTF-8 as committed, line endings exactly as committed, with no normalization applied before hashing. The fingerprint is never computed from working-tree content that is not yet committed, and never computed from rendered or exported output (rendered Markdown, HTML, PDF, or any other transformation). Where useful, a Git blob identifier may additionally be recorded, but the SHA-256 of committed source bytes remains the authoritative, tool-independent fingerprint.
- review evidence actually available, stated plainly if that evidence is limited to disclosed self-review
- approver or disposition-actor identity
- approval or disposition type (Act 1, Act 2, or normal-governance disposition, explicitly labelled, and if a disposition, which one — Approved, Rejected, Deferred, Returned for Revision, or other)
- explicit self-approval disclosure where applicable
- rationale
- known limitations
- unresolved issues
- effective date
- relationship to Bootstrap Authority State at the time of the act (§12–§13)

No approval or disposition record under this ADR may claim review evidence that does not exist, or represent a disclosed self-review as independent review.

**Committed content requirement:** no Act 1 approval, Act 2 approval, validation disposition, or Bootstrap Authority State transition may rely solely on uncommitted working-tree content. Before any such act is effective: the exact relevant document content MUST exist in a repository commit; the record MUST identify that commit; the file MUST exist at the recorded path in that commit; the recorded SHA-256 MUST match the exact committed bytes; the recorded document version MUST match the version stated in that commit. This ADR does not require the entire repository working tree to be clean globally — only that the specific content being acted upon has exact, verifiable committed identity.

## 15. Change and Invalidation Rules

### 15.1 Change after review, before approval or disposition

If a document's content changes after review evidence was produced for it, that review evidence does not automatically apply to the changed content. Material changes require renewed or explicitly updated review evidence. Because this repository does not currently define who holds authority to determine materiality (an open question, §32.1), the safer bootstrap rule applies: **any content-byte change requires renewed review evidence**, without exception, until materiality authority is defined elsewhere.

### 15.2 Change after bootstrap Approval

Any content-byte change to bootstrap-Approved GOV-003 or GOV-010 creates a new content identity. The previous bootstrap Approval record remains valid historical evidence for the *old* content identity only — it does not extend to the changed content. Changed content is not automatically bootstrap-Approved. A new valid approval act is required for the new content identity, if bootstrap authority is still available for that document (i.e., Bootstrap Authority State is `Active` or, for a specific defect, `Reactivated`). If Bootstrap Authority State is `Suspended` or `Terminated` at the time content changes, the changed content MUST NOT be silently treated as approved — it requires disposition under normal governance instead.

### 15.3 Path rename

The same content appearing at a different path is a changed approval identity, because path is an explicit part of the evidence model (§14). The previous record remains historical evidence for the old path. A new path requires its own explicit evidence entry cross-referencing the prior one, consistent with how this repository has handled prior renames (e.g., ARCH-000).

### 15.4 Invalid approval discovery

If an Act 2 approval record for GOV-003 or GOV-010 is later shown to have been invalid (e.g., incomplete evidence, or fraudulent): it does not satisfy the suspension predicate (§12.4). Because Bootstrap Authority State is derived (§13), not stored, the correction is made at the source: the invalid entry in GOV-003's or GOV-010's own Approval Status table MUST be corrected traceably — by appending a correcting entry, never by silently deleting or rewriting a prior one — after which §13's derivation rule automatically reflects the corrected evidence. No invalid approval may be treated as valid merely because a later state transition already occurred in reliance on it; downstream acts must be reassessed against valid authority evidence.

## 16. Self-Approval Disclosure

If exercised, Act 1 is a disclosed founding self-approval act: the same person (Denver Jacobs) both authors this ADR and performs the act that establishes it as effective. Act 2 approvals, if exercised, may likewise involve the same person as both drafter and approver of GOV-003 and/or GOV-010, for as long as no independent reviewer exists.

This is an explicit governance limitation, not a hidden default. It MUST be disclosed in every approval or disposition record made under this ADR. It must never be represented as independent review. AI-generated critique, however extensive, is not independent human review and does not satisfy this requirement. Independent human review remains preferred wherever it becomes available during the period this ADR is in effect — but reviewer availability alone, without more, does not by itself terminate this ADR's authority (see §5, Considered Options, option 8).

## 17. Suspension — Trigger and Effective Moment

When valid bootstrap Approval records exist for both GOV-003 and GOV-010 (§14), Bootstrap Authority State becomes `Suspended` (§12.5) for any new Act 2 approval act. This suspension is non-discretionary — a direct, automatic consequence of both records existing, not a choice made by anyone at that time.

**Exact effective moment:** suspension becomes effective immediately upon the later of the two approval records' recorded effective dates and commits — i.e., whichever of the GOV-003 or GOV-010 approval records was completed second. No new Act 2 approval act may begin after that moment. An approval act already underway but not yet effective at that moment does not receive automatic grandfathering; it must independently establish valid authority as of its own effective moment — which, once suspension has occurred, it structurally cannot do, because Act 2 no longer has authority to act once suspended.

## 18. Validation Act

While Bootstrap Authority State is `Suspended`, exactly one validation act is defined to test whether normal governance, as newly established by the (now bootstrap-approved) GOV-003 and GOV-010, is genuinely operative:

**Target:** `GOV-004` (Engineering Principles).

**Act:** a genuine, real, non-simulated **normal-governance disposition** of GOV-004, performed entirely under the authority mechanism GOV-003 and GOV-010 now define — not under Act 1 or Act 2.

The following are explicitly stated:

- GOV-004 is outside Act 2's scope (§8); Act 2 cannot approve it, has never been able to approve it, and this ADR does not change that.
- GOV-004 is not pre-approved by this ADR or by any prior act.
- **Validation does not require GOV-004 to become Approved.** The authorized actor may approve, reject, defer, return GOV-004 for revision, or issue any other disposition explicitly permitted by the operative GOV-003/GOV-010 governance text.
- Substantive rejection of GOV-004 is a valid governance outcome. Deferral is a valid governance outcome. Return for revision is a valid governance outcome.
- Refusal to approve GOV-004 is not, by itself, structural failure (§20).
- This validation act tests whether normal governance authority and process are operative and can act legitimately — it does not test whether GOV-004 specifically deserves Approval.
- The validation act must not invoke Act 1 or Act 2 authority in any part of its performance.
- Required review evidence for whatever disposition is actually made must actually exist, not be assumed.
- The actor performing the validation act must be explicitly, textually authorized by GOV-003/GOV-010's own approved text for the disposition made — not inferred, not self-declared.
- GOV-004 must not be approved merely to make this validation succeed; if the authorized actor's genuine assessment is that GOV-004 should be rejected, deferred, or returned for revision, that outcome is itself a legitimate and sufficient demonstration of operability.

## 19. Validation Success Criteria

The validation act succeeds only if all of the following are true:

1. GOV-003 has valid Act 2 bootstrap Approval (§14).
2. GOV-010 has valid Act 2 bootstrap Approval (§14).
3. Bootstrap Authority State is `Suspended` (§12.5).
4. The validation actor is explicitly identifiable from the operative, approved GOV-003/GOV-010 text.
5. That actor has executable authority, under the approved text, to issue the disposition actually made.
6. Required review evidence for the disposition actually made exists.
7. Required metadata on GOV-004 is complete, or the disposition itself (e.g., Return for Revision) explicitly addresses metadata incompleteness as its own basis.
8. The disposition made is one explicitly permitted by the operative GOV-003/GOV-010 governance text.
9. The disposition is traceably recorded per §14.
10. Commit hash, path, version, and content-fingerprint evidence exist for the act, per §14.
11. No Act 1 authority is invoked at any point in the act.
12. No Act 2 authority is invoked at any point in the act.
13. No undefined role or authority is invented to complete the act.
14. No ad hoc interpretation of an unresolved authority question is required to complete the act.

**The disposition outcome itself (Approved, Rejected, Deferred, Returned for Revision, or another permitted outcome) is not predetermined by this ADR and does not affect whether validation succeeds**, provided conditions 1–14 hold.

**First-valid-disposition precedence rule:** GOV-004 may receive more than one normal-governance disposition over time (for example, an initial Return for Revision followed later by a substantive Approval or Rejection). Only the **first** disposition of GOV-004 that satisfies all fourteen conditions above counts as the validation act defined in this section. That first qualifying disposition, and only that one, terminates Act 2 (§12.10) and Bootstrap Authority State. Every disposition of GOV-004 after that point — including any that revisits, reverses, or supersedes the first — is ordinary normal-governance activity under GOV-003/GOV-010, entirely outside this ADR's scope, and has no effect on Bootstrap Authority State.

If all fourteen conditions hold for a disposition, and no earlier disposition already satisfied them: Bootstrap Authority State becomes `Terminated` (§12.10), permanently, per §13's recording requirement. Act 1 remains exhausted and non-reusable, as it has been throughout. From this point forward, normal governance is the sole approval path for every document in this repository, including the twelve documents Act 2 never had authority over.

## 20. Structural Failure Criteria

A structural failure of the validation act exists only if all of the following hold:

1. GOV-003 and GOV-010 have both received valid Act 2 bootstrap Approval.
2. Bootstrap Authority State is `Suspended`.
3. The validation act defined in §18 was genuinely attempted (not skipped, not simulated).

**AND** at least one of:

4. The validation actor cannot be identified from the operative, approved GOV-003/GOV-010 text.
5. An actor is identified, but the approved text does not grant that actor executable authority to issue any permitted disposition.
6. The required act cannot be completed without inventing an undefined role or authority.
7. GOV-003 and GOV-010, as bootstrap-Approved, conflict with one another regarding normal governance authority in at least one of the following enumerated, objective ways, each requiring specific citation to the conflicting text of both documents as evidence:
   - (a) the two documents assign final decision authority for the same governance act to different, mutually exclusive actors or roles, with no stated precedence between them;
   - (b) one document permits an act that the other document, on its face, prohibits;
   - (c) the two documents define incompatible authority for the same document lifecycle transition (e.g., both claim exclusive authority to move a document from Review to Approved, or neither grants that authority);
   - (d) the two documents state incompatible precedence rules for resolving conflicts between themselves or with other governance documents (e.g., each claims priority over the other);
   - (e) an actor or role identifiable as authorized under one document's approved text is, under the other document's approved text, explicitly unauthorized for the same act.

   **This condition is an explicit, objective, enumerated blocker category, not a discretionary or open-ended interpretation** — a conflict not falling within (a)–(e), or asserted without citation to the specific conflicting text of both documents, does not satisfy this condition. If a conflict within (a)–(e) exists and is cited, normal governance is not treated as demonstrably operative, and the validation act must not proceed until the conflict is corrected (§22).

**AND all of:**

8. The failure is documented with immutable evidence (per §14's evidence model).
9. The failure is not merely human error.
10. The failure is not merely missing review evidence.
11. The failure is not merely incomplete metadata.
12. The failure is not merely a content defect in GOV-004.
13. The failure is not merely disagreement with the substantive content of GOV-004.
14. The failure is not merely refusal by an authorized actor to approve GOV-004 (rejection, deferral, and return-for-revision are valid dispositions, not failures — §18).
15. The failure is not merely temporary unavailability of an otherwise correctly identified and authorized actor.
16. The failure is not merely an authorized actor declining, recusing, or delaying due to a disclosed conflict of interest.
17. The failure is not merely a scheduling delay.

None of conditions 9–17 constitute the approved governance text lacking an identifiable, executable authority mechanism — they are ordinary process circumstances to be worked through under normal governance itself, not evidence that the mechanism is structurally unusable. Only conditions 4, 5, 6, or 7 — each concerning whether the mechanism itself can function at all — constitute structural failure under this ADR.

## 21. Predicate Evaluation and Evidence Recording

This ADR distinguishes two separate things:

- **Predicate satisfaction** — a factual claim that every predeclared condition in §20 is met.
- **Evidence recording** — the act of recording that evidence in the relevant document's own Approval Status table, from which the resulting Bootstrap Authority State transition is then derived (§13). The sole exception is Act 1 evidence, which is recorded in a separate, non-mutating repository commit rather than in any Approval Status table, per §13's Act 1 evidence model — ADR-0011's own Approval Status table (§35) is never used for this purpose and is never edited to record it.

The person who records this evidence (the "recorder") does not thereby gain authority to redefine, waive, or add predicates; does not gain authority to reactivate Act 2 at their own discretion; and does not gain any authority beyond attesting that the predeclared, objective conditions in §20 are evidenced. This repository does not currently define a dedicated "recorder" role, and this ADR does not invent one; the recorder is simply whoever performs the mechanical, evidence-based check, which may be AI-assisted (per the standing rule that AI may assist with validation and evidence preparation) but must be human-attested and transparently recorded per §13 and §14, never occurring silently.

**If a reactivation determination is later shown to be false** (i.e., the predicate in §20 was not actually satisfied): the reactivation is invalid from that point forward. Any act relying solely on that invalid reactivation lacks valid bootstrap authority. Because Bootstrap Authority State is derived (§13), correction occurs by appending a correcting entry to the specific Approval Status table(s) whose evidence was misrecorded — never by silently deleting or rewriting a prior entry — after which §13's derivation rule automatically reflects the corrected evidence. This correction mechanism itself does not create a new discretionary Founder decision point; it is the same evidence-attestation function described above, applied to identify and record an error.

## 22. Non-Discretionary Reactivation

If, and only if, the complete structural-failure predicate in §20 is satisfied and traceably documented per §21, Bootstrap Authority State becomes `Reactivated` (§12.8) as a non-discretionary consequence of this ADR's predeclared rule. No new discretionary Founder authority event occurs. No Act 3 exists anywhere in this document. No person — including Denver Jacobs — receives general or ongoing discretion to decide when reactivation occurs; it either follows automatically from the documented, objective predicate, or it does not occur at all.

### 22.1 Scope of reactivated authority

Reactivated Act 2 is narrower than the original `Active` state. It may act only to approve corrected, committed content addressing the specific governance defect that caused the documented structural failure, in GOV-003, GOV-010, or both — only where the immutable failure evidence demonstrates that the specific document contributes to the documented defect. It may not approve any unrelated change, and it may not be used to approve any document outside §8's exact two-document list.

### 22.2 GOV-003/GOV-010 conflict as structural defect

Where the structural failure arises from §20 condition 7 (GOV-003 and GOV-010 materially conflicting), the reactivated authority's narrow purpose is limited to approving a corrected, committed version of whichever document (or both) resolves the specific identified conflict — not to approving unrelated changes to either document.

### 22.3 Reactivation does not grant editing authority

Act 2 — in its original `Active` state and in its `Reactivated` state alike — is approval authority, not general content-editing authority. Reactivation does not grant, and must never be read as granting, authority to author or edit GOV-003 or GOV-010's content. Document amendment must occur through whatever authorship or editing process is otherwise legitimately available in this repository (the same process by which these documents were originally drafted); reactivated Act 2 may only approve the resulting corrected, committed content once it exists. Approval authority and authorship authority are not conflated by this ADR.

### 22.4 Reapproval identity

Every corrected content identity requires its own approval evidence under §14 — reapproval applies to the entire exact committed document identity, not to an abstract "delta" or partial change. Previous approval records remain valid historical evidence for the prior content identity only.

### 22.5 Repeat cycles

After the specific defect is corrected and validly approved: Bootstrap Authority State returns to `Suspended` (§12.9), and the validation act (§18) is retried. Success terminates Act 2 permanently per §19. A further structural failure may trigger reactivation again, but only if the full §20 predicate is independently satisfied and documented anew each time (§21).

This ADR does not impose an arbitrary numeric cycle cap, because no specific number is evidence-justified. Instead: **a second consecutive structural-failure determination targeting the same document, within the overall period this ADR remains in effect, triggers mandatory reconsideration under Revisit Conditions (§31) before any further reactivation may be recorded for that document.** This reconsideration requirement does not itself grant, waive, or create new authority — it pauses the mechanism, pending review of whether this ADR's own design remains adequate, rather than allowing indefinite silent cycling.

**Repeated reactivation cycles do not create general Founder authority.** Each cycle remains bounded to the specific defect that triggered it, under the same non-discretionary rule defined here.

## 23. Precedence and Subordination

This ADR is exceptional, temporary, non-precedential, and narrowly scoped. It is not a general governance model. It is not reusable by analogy to any other situation. It is not evidence that Founder status ordinarily grants unilateral approval power — it grants authority for exactly the two acts and the exact document scope defined in this document, and nothing beyond that.

This ADR is subordinate to, and yields to, later operative normal governance once the handover defined in §§17–22 succeeds. It is permanently exhausted after Bootstrap Authority State reaches `Terminated` (§12.10).

Future documents must not cite this ADR as authority for: unrelated self-approval; permanent Founder override; emergency powers; release authority; security-risk acceptance; bypassing review; or bypassing governance of any kind.

## 24. Rationale

The two-act structure (§§7–8) separates founding this document from exercising the authority it defines, avoiding the circularity of a document deriving authority from itself. Limiting Act 2's scope to exactly GOV-003 and GOV-010 (§8), rather than the full corpus previously considered, minimizes the exceptional-authority footprint to precisely what is needed to escape the bootstrap paradox and nothing more. Excluding STD-001 (§9) prevents an unintended, much broader authority leak that direct inspection of STD-001's own text revealed would otherwise occur.

Separating document lifecycle status from Bootstrap Authority State (§11–§13) makes the mechanism's operational state traceable from repository evidence rather than dependent on prose interpretation, without overloading STD-001's own repository-wide lifecycle vocabulary with ADR-0011-specific meanings. Defining validation as testing operability rather than requiring a specific disposition (§18–§19) prevents the mechanism from pressuring an approval outcome. Requiring committed-content identity and a specific, tool-independent fingerprint method (§14) closes a category of ambiguity this project has already encountered directly. Distinguishing predicate satisfaction from evidence recording (§21), and approval authority from editing authority (§22.3), prevents the fallback and correction paths from becoming disguised, renewable forms of general override authority.

## 25. Consequences

### 25.1 Positive Consequences

- Resolves the governance bootstrap paradox without pretending any Draft document is already binding.
- Minimizes exceptional authority to exactly two documents, not the full corpus.
- Creates traceable, immutable approval and disposition evidence for every act performed under this ADR.
- Establishes an explicit, objective path to normal governance rather than an open-ended exception.
- Avoids the STD-001 §30 repository-wide authority leak identified during design review.
- Avoids bootstrap-approving the previously-considered fifteen-document set.
- Separates document lifecycle status from operational Bootstrap Authority State, making the mechanism's actual state traceable from repository evidence.
- Does not pressure a predetermined approval outcome for the validation target.
- Specifies a tool-independent, committed-content evidence identity model, closing a category of risk already encountered in this project.

### 25.2 Negative Consequences

- Disclosed self-approval remains a real governance limitation for as long as no independent reviewer exists.
- The mechanism is materially more complex than an ordinary document approval, by design, to remain safe.
- Handover requires a genuine, real validation act — it cannot be assumed or shortcut.
- Authority over Draft-to-Review metadata transitions remains unresolved and is not addressed by this ADR.
- The quality and completeness of GOV-003 and GOV-010, once approved, remains critical — this ADR cannot compensate for defects in their content.
- The structural-failure predicate requires careful, disciplined evidence-gathering to apply correctly.
- No independent human reviewer currently exists to check any of this work.
- The Approval Status table entries this derivation relies on require disciplined, manual (AI-assisted) maintenance, since no repository tooling currently enforces or automatically derives Bootstrap Authority State.
- Materiality authority for content changes is unresolved, so this ADR defaults to requiring renewed review evidence for any content-byte change, which may be more conservative than eventually necessary.

## 26. Risks

- **Confusing bootstrap Approval with normal-governance Approval or with technical validation** — mitigated by the explicit distinctions in §10 and the mandatory labelling requirement in §14.
- **Scope creep beyond the two enumerated documents** — mitigated by the exhaustive exclusion list in §8.
- **Reactivation becoming a disguised, repeatable override mechanism** — mitigated by the objective, non-discretionary predicate in §20–§22 and the explicit "does not create general Founder authority" statement.
- **Self-approval being mistaken for independent review** — mitigated by the mandatory disclosure requirement in §16.
- **This ADR itself being cited beyond its scope in the future** — mitigated by the explicit precedence and subordination statement in §23.
- **Validation outcome bias toward Approval** — mitigated by explicitly defining validation success in terms of any authorized, evidence-backed disposition, not Approval specifically (§18–§19).
- **Operational state becoming untraceable** — mitigated by the separate Bootstrap Authority State machine and its explicit, evidence-based derivation rule (§11–§13).
- **Self-referential content fingerprinting** — mitigated by never modifying ADR-0011's own tracked content after this Draft is finalized, for any reason, and by recording Act 1 evidence exclusively in a separate, subsequent, content-non-mutating commit rather than inside ADR-0011 itself, so no field of this document ever needs to reference its own post-edit hash (§13).
- **Evidence fingerprint ambiguity (working tree vs. committed content)** — mitigated by the explicit committed-bytes, tool-independent fingerprint specification (§14).
- **Silent or undiscoverable erroneous reactivation** — mitigated by the challenge/correction mechanism in §21.
- **Reactivation being read as editing authority** — mitigated by the explicit authorship/approval distinction in §22.3.

## 27. Security Impact

This ADR concerns documentation-governance process, not a technical security boundary. Its principal security-relevant property is the discipline it imposes on who may claim approval authority and under what evidence — reducing the risk of undocumented, unverifiable, or silently-expanded authority claims elsewhere in the corpus. It does not grant, and explicitly excludes, any authority over security-risk acceptance (§8).

## 28. Operational Impact

Operationally, this ADR requires that any Act 1 act, Act 2 act, or validation disposition be recorded per the evidence model (§14) in the relevant document's own Approval Status table before it is treated as having occurred, from which Bootstrap Authority State is then derived (§13), and that the suspension and validation steps (§17–§19) be executed in sequence rather than skipped. No tooling or automation currently exists to enforce this; compliance depends on disciplined manual (and AI-assisted) record-keeping until normal governance is operative.

## 29. Migration Impact

Once Bootstrap Authority State reaches `Terminated` (§12.10), all future approval activity — for the two documents this ADR ever touched, and for every other document in the repository — migrates entirely to normal governance under GOV-003/GOV-010. This ADR itself does not need to be revised, deleted, or amended for this migration to occur; it simply becomes permanently exhausted and historical, per §23.

## 30. Validation

The substantive validation mechanism for this ADR's own design is the handover sequence defined in §17–§22: suspension with an exact effective moment, a real disposition-based validation act (§18), objective success criteria independent of outcome (§19), and objective structural-failure criteria with non-discretionary, narrowly-scoped reactivation (§20–§22). No separate validation mechanism is defined for this section beyond a pointer to those.

## 31. Revisit Conditions

This ADR should be revisited if: the structural-failure predicate (§20) is triggered a second consecutive time against the same document (mandatory per §22.5); an independent human reviewer becomes available and has not yet been incorporated into the process; materiality authority for content changes (§15.1) becomes defined elsewhere in a way that could relax this ADR's conservative default; or normal governance, once operative, identifies a defect in this ADR's own design that was not anticipated here.

## 32. Open Questions and Non-Goals

### 32.1 Open Questions

- When, if ever, will an independent human reviewer be appointed?
- Who may transition a document's status from Draft to Review — unresolved corpus-wide, and deliberately not addressed by this ADR.
- Who holds authority to determine materiality of a content change (§15.1) — unresolved; this ADR defaults to the conservative "any change requires renewed review evidence" rule until this is defined elsewhere.
- Should STD-001 itself be amended, later, under normal governance, to narrow §30's repository-wide reach?
- GI-01 (ADR lifecycle vocabulary mismatch between this corpus's legacy `.docx` ADRs and the current, STD-001-aligned Markdown template) remains open.
- GI-11 (Founder/Product Owner/Chief Architect terminology inconsistency across the governance corpus) remains open.

### 32.2 Explicit Non-Goals

This ADR does not, and is not intended to:

- authorize ARCH-001 or any future architecture document;
- authorize, validate, or imply readiness of any source-code implementation;
- authorize a production release;
- authorize production deployment;
- accept security risk on behalf of the project;
- approve the previously-considered fifteen-document foundational set;
- grant authority over Draft-to-Review lifecycle status transitions, for any document;
- technically validate the correctness of any ADR's underlying technology choices;
- establish or imply implementation readiness for any RFC or ADR;
- establish permanent or general Founder override authority of any kind;
- grant content-editing or authorship authority under Act 2 or its reactivation (§22.3).

## 33. References

| Document ID | Title | Status | Path |
|---|---|---|---|
| GOV-003 | Governance Model | Draft | `governance/GOV-003_Governance_Model_v0.1.docx` |
| GOV-010 | Decision Framework | Draft | `governance/GOV-010_Decision_Framework_v0.1.docx` |
| STD-001 | Documentation Standards | Draft — contextual evidence only, explicitly excluded from this ADR's scope (§9) | `standards/STD-001-Documentation-Standards.md` |
| GOV-004 | Engineering Principles | Draft — validation-act target only, outside Act 2 scope (§18) | `governance/GOV-004_Engineering_Principles_v0.1.docx` |

No external web references are used in this document.

## 34. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-10 | Denver Jacobs | Initial Draft, establishing a narrowly-scoped, two-act bootstrap approval mechanism for GOV-003 and GOV-010, with an explicit, non-discretionary handover to normal governance and a derivation-based Bootstrap Authority State model (§11–§13) separate from this document's own lifecycle status, including an evidence architecture in which Act 1 is recorded exclusively via a separate, non-mutating, pushed commit rather than any edit to this document, so that ADR-0011's own tracked content — including this table — is never modified after this Draft is finalized. This is presented as a single initial Draft: the document was iterated on during drafting, incorporating architectural review findings before this Draft was ever committed, staged, or seen outside its own continuous authoring process — no prior version of this content was ever established as a distinct, checkpointed revision, so no separate revision entry is recorded for that iteration. No approval act has occurred. |

## 35. Approval Status

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-10 |
| Act 1 (Founding Act) | *This row is intentionally static and is never edited, including after Act 1 occurs (§13).* Act 1 evidence, if it exists, is recorded exclusively in a separate, pushed repository commit whose message contains the literal string `ADR-0011 Act 1 Evidence Record`, never in this table. | *Not determinable from this table by design* | *N/A — see §13* |
| Technical Review | TBD | Pending | |

**Bootstrap Authority State:** `Not Established` (§12.1), derived per §13 by searching this repository's pushed commit history for a valid Act 1 evidence commit per the convention above — not by inspecting this table's Act 1 row, which never changes. As of this revision, no such commit exists.
