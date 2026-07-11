---
document_id: ADR-0012
title: Content-Non-Mutating Act 2 Approval Evidence
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: A fresh, explicit, disclosed founding act invoking ADR-0011 §3's source-of-authority theory — distinct from and not a reuse of Act 1; not yet exercised
created: 2026-07-11
last_updated: 2026-07-11
classification: Public
related_documents:
  governance:
    - GOV-003 (Draft, published, not approved)
    - GOV-010 (Draft, not yet published)
  standards:
    - STD-001 (Draft — read as contextual evidence only; not affected by this ADR)
  architecture: None
  rfcs: None
  adrs:
    - ADR-0011 (Draft, Act 1 effective — corrected only as to Act 2 evidence mechanics; see §14)
  roadmap: None
  research: None
  operational: None
  source_artifacts: None
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ADR-0012 — Content-Non-Mutating Act 2 Approval Evidence

> **Status notice:** This ADR is **Draft**. No approval act described in this document has occurred. Drafting, staging, committing, or pushing this ADR does not itself constitute its approval, by the same principle ADR-0011 §7 establishes for Act 1. See §20 (Approval Status).

## 1. Context

ADR-0011 established a bounded, two-act bootstrap mechanism to move GOV-003 and GOV-010 out of Draft status into genuine, operative effect. Its own Act 1 evidence (approving ADR-0011 itself) is recorded through a separate, content-non-mutating Git commit, explicitly because ADR-0011 would otherwise have to record its own approval evidence inside the very artifact being approved. ADR-0011 §13 and §21 state that GOV-003, GOV-010, and GOV-004 are "not self-referential in this way" and therefore may use "the ordinary, mutable Approval Status convention" for Act 2 and validation evidence — meaning approval evidence is written directly into the target document's own tracked Approval Status table.

## 2. Problem

ADR-0011's own §14 establishes exact-byte artifact identity (SHA-256 of exact committed bytes) as the authoritative approval-evidence fingerprint, and §15.2 states that any content-byte change to a bootstrap-Approved document creates a new content identity that "is not automatically bootstrap-Approved." Editing GOV-003's (or GOV-010's) own Approval Status table to record Act 2 approval necessarily produces a new Git blob, a new SHA-256, and a new commit identity (Y), distinct from the exact artifact identity that was actually approved (X). No provision of ADR-0011 permits Y to inherit approval granted to X. The prescribed Act 2 evidence mechanism is therefore self-defeating: following it as written produces a record that, by ADR-0011's own §15.2, does not validly approve the document it is written into. This defect blocks safe execution of any GOV-003 or GOV-010 Act 2 approval under the mechanism as currently described. It does not invalidate Act 1, the existence of bounded Act 2 authority, or the substantive content of GOV-003 or GOV-010.

## 3. Decision Drivers

- ADR-0011's exact-byte identity model (§14) must be preserved without exception, exactly as it already is for Act 1.
- The correction must not touch ADR-0011's tracked bytes, which are — by ADR-0011's own §13 — never modified after that Draft was finalized, for any reason.
- The correction must not expand Act 2's document scope, reopen GOV-004 validation criteria, alter reactivation rules, or create new Founder authority.
- The correction must apply the same content-non-mutating principle already proven sound for Act 1's own evidence, rather than inventing a new mechanism.
- The correction must not silently assume the defect is resolved by interpretation alone, given the express, repeated contradictory text in ADR-0011 §13/§21.

## 4. Decision

For Act 2 approval evidence targeting GOV-003 or GOV-010, this ADR establishes the following binding rule:

All Act 2 approvals of GOV-003 and GOV-010 must use separate, subsequent, content-non-mutating Git evidence commits. The approved target document must not be modified to insert approval evidence. Each evidence commit must be empty and tree-identical to its parent for the bootstrap sequence. Approval becomes effective only when the exact compliant evidence commit is pushed to `origin/main`. This rule applies separately to GOV-003 and to GOV-010; each document receives its own evidence commit.

## 5. Scope

This ADR corrects only the Act 2 approval-evidence *mechanism* described in ADR-0011 §12.2, §13, §14, and §21, as applied to GOV-003 and GOV-010. It applies prospectively to any Act 2 approval evidence not yet created as of this ADR's own effective date.

## 6. Non-Scope

This ADR does not:

- alter the scope of Act 2 (ADR-0011 §8) — still exactly GOV-003 and GOV-010, no other document;
- approve GOV-003;
- approve GOV-010;
- modify GOV-003 or GOV-010's tracked content;
- approve STD-001;
- modify GOV-004 or its validation criteria (ADR-0011 §18–§19);
- begin GOV-004 validation;
- establish permanent or general Founder override authority;
- modify ADR-0011's Act 1 decision, Act 1 evidence, or Act 1 evidence commit;
- reopen ADR-0011's reactivation rules (ADR-0011 §20–§22) beyond inheriting the same evidence-location correction;
- authorise release, deployment, security-risk acceptance, legal authority, or financial authority;
- authorise ARCH-001 or any future architecture document.

## 7. Exact-Byte Identity

This ADR reaffirms, without exception, ADR-0011 §14's principle that SHA-256 of exact committed file bytes is the authoritative, tool-independent fingerprint of any approved identity, and ADR-0011 §15.2's principle that any content-byte change creates a new content identity that does not automatically inherit prior approval. This ADR resolves the tension identified in §2 by ensuring Act 2 evidence-recording never changes the target document's bytes at all, eliminating the X-to-Y question entirely rather than attempting to justify an inheritance rule.

## 8. Evidence Commit Requirements

For each Act 2 target document (GOV-003, GOV-010), the approval evidence commit must:

1. be created only after the exact target artifact (document ID, path, version, commit, blob, SHA-256, byte size, line count) has been finalized, committed, and pushed;
2. leave the target document's tracked bytes completely unchanged;
3. be tree-identical to its parent commit (an empty commit for the bootstrap sequence);
4. carry, in its commit message, the complete evidence record specified in §9;
5. use the exact subject convention specified in §10;
6. become effective only once both created and pushed to `origin/main` (§11).

## 9. Evidence Fields

Every Act 2 evidence commit under this ADR must record, at minimum:

1. document ID
2. repository path
3. version
4. artifact commit
5. Git blob ID
6. SHA-256
7. byte size
8. line count
9. approver
10. approver capacity
11. approval-authority source
12. approval type
13. disposition
14. review evidence
15. independent-review status
16. conflict or self-approval disclosure
17. rationale
18. known limitations
19. unresolved issues
20. relationship to Bootstrap Authority State
21. relationship to the other Act 2 target document
22. relationship to Act 2 suspension
23. relationship to GOV-004 validation
24. effective date
25. evidence-publication requirement
26. relevant evidence

The evidence commit's own hash is established externally, by Git, after the commit is created, and is never predicted or embedded within its own message.

## 10. Evidence Subjects

```text
GOV-003 Act 2 Approval Evidence Record
```

and

```text
GOV-010 Act 2 Approval Evidence Record
```

Each subject is distinct from `ADR-0011 Act 1 Evidence Record`, identifies its exact target document, identifies Act 2, identifies approval evidence, and does not by itself imply approval of the other target document or Act 2 suspension.

## 11. Effectiveness

**GOV-003 evidence publication.** After the valid GOV-003 evidence commit is pushed:

```text
GOV-003 approval: Effective
GOV-003: Operative
Bootstrap Authority State: Active
Act 2: Active, limited to completing GOV-010
GOV-010: Not approved
GOV-004 validation: Not permitted to begin
```

**GOV-010 evidence publication** (after GOV-003 approval is already effective). After the valid GOV-010 evidence commit is pushed:

```text
GOV-010 approval: Effective
GOV-010: Operative
Act 2: Suspended
Bootstrap Authority State: Suspended
GOV-004 validation: Eligible to begin only through a later, separately authorised normal-governance action
```

GOV-004 validation does not begin automatically upon Act 2 suspension.

## 12. State Transitions

| Trigger | Bootstrap Authority State | Act 2 | GOV-003 | GOV-010 |
|---|---|---|---|---|
| Before any Act 2 evidence | Active | Active | Draft, not approved | Draft, not approved |
| GOV-003 evidence commit pushed | Active | Active, limited to completing GOV-010 | Operative | Draft, not approved |
| GOV-010 evidence commit pushed (GOV-003 already effective) | Suspended | Suspended | Operative | Operative |
| GOV-004 validation succeeds (separate action, §18–§19 of ADR-0011, unaffected by this ADR) | Terminated | Terminated | Operative | Operative |

## 13. Static Metadata

GOV-003 and GOV-010's tracked bytes remain unchanged after their Act 2 approval becomes effective. Their tracked `status: Draft` values and Approval Status placeholders remain static during the bootstrap period. Operative approval state is derived from immutable, external Git evidence (the pushed evidence commits), never from the target documents' own tracked bytes — exactly as ADR-0011 §11/§13 already establish for ADR-0011 itself. No X-to-Y inheritance applies, because no mutation of the target document ever occurs. Later reconciliation of tracked metadata (e.g., updating a `status:` field to reflect operative approval) is a separate, later, normal-governance action, not required for approval to be effective. This is a bootstrap-specific exception required by exact-byte identity preservation and does not establish a permanent repository-wide lifecycle model; static Draft metadata does not negate externally evidenced bootstrap approval.

## 14. Relationship to ADR-0011

This ADR preserves ADR-0011's Act 1 decision and evidence unchanged; preserves Act 2's exact two-document scope (ADR-0011 §8) unchanged; preserves GOV-004 validation requirements (ADR-0011 §18–§19) unchanged; preserves reactivation rules (ADR-0011 §20–§22) unchanged except for inheriting the same evidence-location correction where reactivated Act 2 records its own corrective approvals; and preserves all Founder-authority limitations (ADR-0011 §23) unchanged. It corrects only the Act 2 approval-evidence mechanism.

```text
For Act 2 approval-evidence mechanics only, this ADR supersedes any ADR-0011 provision requiring approval evidence to be written into the tracked bytes of GOV-003 or GOV-010.

All other ADR-0011 provisions remain unchanged.
```

ADR-0011's own tracked bytes are not modified by this ADR, consistent with ADR-0011 §13's own rule that its content is never modified after its Draft was finalized, for any reason.

## 15. Duplicate and Error Handling

Only the first valid, identity-specific evidence commit pushed to `origin/main` for a given target-document artifact identity controls, by the same first-valid-record principle ADR-0011 §19 already applies to GOV-004 dispositions. A duplicate evidence commit for the same artifact identity is invalid and has no effect on state unless explicitly marked, in its own message, as a non-substantive duplicate acknowledging the first record. Conflicting evidence requires a separately governed correction. An erroneous evidence commit must never be amended or force-pushed; correction occurs only through a later, append-only Git commit that explicitly supersedes the erroneous record, mirroring ADR-0011 §15.4's correction model. A content-byte change to the target document after approval creates a new artifact identity requiring its own new approval evidence, per ADR-0011 §15.2. This mechanism introduces no new tag, branch, external system, or evidence file.

## 16. Consequences

### 16.1 Positive Consequences

- Resolves the identified exact-byte identity contradiction without touching ADR-0011's tracked bytes.
- Preserves Act 1's evidence integrity completely.
- Extends a mechanism already proven sound (Act 1's Model B) rather than inventing a new one.
- Restores a safe path to GOV-003 and GOV-010 approval.

### 16.2 Negative Consequences

- Requires a fresh, explicit, disclosed founding act to become effective, which this ADR does not itself perform.
- Adds a second foundational document to track alongside ADR-0011.
- Requires disciplined manual record-keeping, exactly as ADR-0011 already requires for its own evidence.

## 17. Risks

- **This ADR itself facing the same bootstrap-approval problem it is meant to solve** — mitigated by explicitly disclosing, rather than concealing, the approval-authority gap in §19, and by requiring its own approval evidence to use the same content-non-mutating model, applied reflexively.
- **Confusion between this ADR's evidence markers and ADR-0011's Act 1 marker** — mitigated by the distinct, explicit subject conventions in §10.
- **Scope creep beyond the Act 2 evidence mechanism** — mitigated by the explicit Non-Scope list in §6.

## 18. Alternatives

Considered and rejected: Option A (interpretive confirmation only, without a corrective document) — rejected as insufficient because an interpretive statement cannot safely displace ADR-0011's express, repeated, contradictory normative text. Option B (direct narrow amendment of ADR-0011's tracked bytes) — rejected because ADR-0011 §13 currently prohibits its own tracked-content modification "for any reason," and amending it would itself require resolving the same authority question this ADR exists to resolve, while also placing the existing Act 1 evidence commit's SHA-256 reference at risk of referring to a superseded identity. This ADR (Option C — a separate, non-mutating corrective document) was selected as the only option preserving both ADR-0011's byte integrity and the already-effective Act 1 evidence.

## 19. Authority Basis

Drafting this ADR uses ordinary authorship/contribution capacity — the same basis used to draft ADR-0011, STD-001's conversion, and GOV-003/GOV-010's conversion. **This ADR's own approval is not yet authorized and is not performed by drafting, committing, or pushing it**, by the same principle ADR-0011 §7 establishes for Act 1. No existing document in this repository currently grants approval authority over a new ADR: GOV-003 and GOV-010 remain unapproved Drafts, STD-001 remains Draft and excluded from any authority role, and ADR-0011's Act 1 is exhausted and non-reusable, applying only to ADR-0011 itself. The only textually defensible basis for this ADR's approval is a **fresh, explicit, disclosed founding act**, invoking the same standing source-of-authority fact that grounded Act 1 (ADR-0011 §3 — Denver Jacobs's Founder status as a binding project fact, independent of any document), but performed as a **distinct act, not a reuse of Act 1**, scoped solely to this ADR. This act has not occurred as of this Draft.

## 20. Approval Status

### 20.1 Document State

| Field | Value |
|-------|-------|
| Lifecycle status | Draft |
| Approval disposition | Not yet approved |
| Disposition date | Not yet assigned |
| Approving actor | Not yet assigned |

### 20.2 Immutable Approval Evidence

| Field | Value |
|-------|-------|
| Document ID | ADR-0012 |
| Repository path | adrs/ADR-0012-Content-Non-Mutating-Act-2-Approval-Evidence.md |
| Version | 0.1.0 |
| Artifact commit | Not yet created |
| Git blob ID | Not yet created |
| SHA-256 | Not yet created |
| Approver | Not yet assigned |
| Approver capacity | Not yet assigned |
| Approval type | Not yet assigned |
| Review evidence | Not yet created |
| Self-approval / conflict-of-interest disclosure | Not yet created |
| Rationale | Not yet created |
| Known limitations | Not yet created |
| Unresolved issues | Not yet created |
| Disposition | Not yet approved |
| Effective date | Not yet assigned |
| Evidence commit | Not yet created |
| Evidence publication state | Not yet published |
| Relevant evidence | Not yet created |

This document is not operative. No reviewer is currently assigned. No evidence commit currently exists. This table does not, and must not be read to, claim that this ADR has been approved.

## 21. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-11 | Denver Jacobs | Initial proposed Draft, correcting the Act 2 approval-evidence mechanism identified as defective in ADR-0011 §13/§21 (target-document-mutation prescription conflicting with §14/§15.2's exact-byte identity model), by extending ADR-0011's own proven Act 1 evidence model (separate, content-non-mutating Git commit) to Act 2 approvals of GOV-003 and GOV-010. No approval act has occurred. This ADR does not itself resolve its own approval-authority gap (§19), which requires a separate, explicit, disclosed founding act not yet performed. |
