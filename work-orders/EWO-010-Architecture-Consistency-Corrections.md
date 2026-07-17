---
document_id: EWO-010
title: "Architecture Consistency Corrections"
version: 0.1.1
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-17
last_updated: 2026-07-17
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-002 (v0.2.1 — architecture/ARCH-002-Runtime-Architecture.md) — corrected by this EWO
    - ARCH-003 (v0.5.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md) — corrected by this EWO
    - ARCH-006 (v0.1.4 — architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md) — corrected by this EWO
  adrs:
    - ADR-0013 (v0.1.1 — adrs/ADR-0013-Architectural-Evolution-of-SynapseOS.md) — corrected by this EWO
    - ADR-0014 (v0.1.1 — adrs/ADR-0014-Engineering-Standards-Corpus-Reconciliation.md) — corrected by this EWO
  consolidation:
    - ACR-001 (v0.2.0 — consolidation/ACR-001-Architecture-Consolidation-Review.md) — originating source of the Architecture Amendment Register findings this EWO's corrections implement
    - AFR-001 (v0.1.0 — consolidation/AFR-001-Architecture-Freeze-Review.md) — independently re-verified this EWO's corrections and identified this document's own absence as a Gate 4 defect
  governance:
    - GOV-012 (v0.1.0 — governance/GOV-012-Architecture-Review-Board-Session-001.md) — originating source of GOV-012-ISS-010, GOV-012-ISS-012, and GOV-012-ISS-013, the three findings this EWO's corrections implement; GOV-012-ISS-011 was investigated but not implemented (§ Scope, § Out of Scope)
  reported_by: Not applicable — no Engineering Report exists or is required for a documentation-only correction task (§46, this EWO's own "Constraints")
  predecessor: EWO-009 (work-orders/EWO-009-Runtime-Actor-Execution.md) — the milestone whose completion is the specific fact this EWO's ARCH-003 correction records
  base_state:
    docs_head: a993ab1083731598a83ccab40a87028f1229c614
    runtime_head: 0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7
---

# EWO-010 — Architecture Consistency Corrections

Registered per STD-001 §46 (Engineering Work Orders — "authorizes a specific, scoped item of engineering work — implementation, reconciliation, migration, or another bounded engineering task"; **reconciliation** is this EWO's own basis). **This document is a historical reconstruction — see "Historical Reconstruction Notice," immediately below — filed to restore the missing controlled-document record for a documentation-reconciliation task whose corrections already exist, are already applied to five documents, and have already been independently re-verified by a subsequent review (AFR-001).**

## Historical Reconstruction Notice

**This Engineering Work Order is a historical reconstruction, not a prospective authorization.**

- **The execution predates this document.** Every correction this EWO records — the ARCH-003 implementation-baseline currency fix, the ARCH-002 §8 count-label fix, and the STD-001 citation-precision fix applied to ADR-0013, ADR-0014, and ARCH-006 — was already carried out, in full, before this document was authored. This document did not exist at the time that work was performed; the five affected documents' own Change History entries cite "EWO-010" only because this filing restores, after the fact, the controlled record their citations already assumed.
- **Execution date and reconstruction date are distinct, and are stated separately below, not conflated.** The correction work was executed on **2026-07-17**, during a task presented as "EWO-010 — Architecture Consistency Corrections," conducted directly against the repository with no separate work-order document filed at the time. This document was authored — reconstructed — later the same day, **2026-07-17**, after an independent Architecture Certification Recovery Review found the absence of a filed `work-orders/EWO-010-*.md` document to be a genuine, dangling-citation defect (a Gate 4 finding in AFR-001, confirmed independently by that Review). Where this document's `created`/`last_updated` fields both read 2026-07-17, that reflects this document's own true authoring moment — it does not, and must not be read to, claim this file existed earlier.
- **The corrections have already been independently re-verified**, twice: once by their own executing task (direct diff inspection, table-consistency checks, frontmatter validation, and a full `git status` confirmation that only the five intended files changed), and again by AFR-001 (Architecture Baseline Certification), which independently re-confirmed the corrected text is present in the current tracked files before citing this EWO's own absence as a separate, narrower defect.
- **This document authorizes no additional work.** No correction remains to be performed under this EWO. Its "Scope," "Objectives," and "Completion Criteria" sections record what was already, verifiably, done — they are not a forward work list. Where this document uses the present-authorizing tense common to this repository's own EWO structure, each such section is understood, throughout, as a retrospective record of what was done and verified, never as an unmet requirement.
- **This document does not imply governance adjudication has occurred for the separate, unrelated finding the same Certification Recovery Review identified** — the ADR-0017/ARCH-001 v0.2.0 evidence-provenance anomaly, and the wider "Architecture Review Board" citation pattern across ADR-0013, ARCH-002, and EWO-004. That finding is expressly out of this EWO's own scope (§ Out of Scope, below) and remains unresolved, exactly as the Certification Recovery Review left it.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-010 |
| Title | Architecture Consistency Corrections |
| Version | 0.1.1 |
| Status | **Draft** — this document's own governance status; the correction work it records is already complete |
| Author | Denver Jacobs |
| Owner | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-17 |
| Last Updated | 2026-07-17 |
| Classification | Public |
| Applicable repository | `synapse-docs` (documentation only — no runtime change) |
| Target branch | `main` |
| Documentation base HEAD | `a993ab1083731598a83ccab40a87028f1229c614` |
| Runtime reference HEAD | `0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7` (consulted read-only, to verify ARCH-003's correction; not modified) |
| Reported by | Not applicable (§ Engineering Authority) |

## Engineering Authority

This EWO derives its authority from STD-001 §46 (reconciliation as an authorized EWO purpose) and from the specific findings it implements:

1. **GOV-012-ISS-013** (Architecture Amendment Required) — ARCH-003's stale "no actor-defined message-handling logic exists" statement, superseded by EWO-009/ER-009.
2. **GOV-012-ISS-010** (Accepted with Clarification) — ARCH-002 §8's "four distinct concerns" undercounting its own five-element enumeration.
3. **GOV-012-ISS-012** (Accepted with Clarification, STD-001 version-precision) — ADR-0013, ADR-0014, and ARCH-003's flat "STD-001 (Approved)" citation, not distinguishing STD-001's genuinely evidenced v0.1.0/v0.2.0 approval from its unapproved current v0.4.0 content. GOV-012-ISS-012 names ADR-0013, ADR-0014, and ARCH-003 as the documents requiring this correction; it does not name ARCH-006. The corresponding ARCH-006 instance of the same defect was discovered independently during this EWO's own execution, not originally identified by GOV-012-ISS-012 — see § Scope (as implemented) and § Out of Scope for the distinction, drawn precisely there rather than folded silently into this citation.

This EWO does not itself introduce any of these three findings — each originates in GOV-012 (Architecture Review Board Session 001) and, for the underlying evidentiary basis, ACR-001. GOV-011 §5 item 6 and §8 item 3 each state that an "Architecture Amendment Required" decision "creates an obligation" to open a new ADR, not an amendment in itself — the authority basis for this EWO's implementation of GOV-012-ISS-013. GOV-011 does not use this quoted phrase for "Accepted with Clarification" decisions; this EWO's authority to implement GOV-012-ISS-010 and GOV-012-ISS-012 rests instead directly on GOV-012's own recorded Required Action for each finding, per GOV-011 §8 item 2's separate definition of that classification.

No Engineering Report (ER) is filed for this EWO, and none is required: STD-001 §47 registers ER as the record of "completed engineering work performed under an EWO," and this repository's own established convention (confirmed by inspection of EWO-001 through EWO-009) files a dedicated ER only for runtime-code milestones. A documentation-only reconciliation task's own completion record is this EWO document itself, per §46's own required-content list, which does not name an ER as a precondition.

## Purpose

Reconcile the published SynapseOS architecture with architectural decisions already approved and implemented, correcting three specific classes of documentation drift GOV-012 (Architecture Review Board Session 001) identified: a stale implementation-status statement, an internal count-label error, and an imprecise approval-status citation — each affecting one or more of five already-published documents.

## Background

GOV-012, the first formal Architecture Review Board session, evaluated the completed Act 2 evidence base and produced twenty-two decided findings. Four of them — GOV-012-ISS-010, GOV-012-ISS-011, GOV-012-ISS-012, and GOV-012-ISS-013, using GOV-012's own issue-identifier numbering directly, not a separate numbering this document invents — required a documentation correction rather than new research, a new ADR, or an architectural amendment in substance. This EWO implements three of those four (GOV-012-ISS-010, GOV-012-ISS-012, GOV-012-ISS-013, per § Engineering Authority above); the fourth, GOV-012-ISS-011, was independently re-investigated during this EWO's own execution and found to require no correction for two of its three affected citations (ADR-0015 and ADR-0016, already accurate) and, for the third (ADR-0017), a governance adjudication this EWO has no authority to perform, rather than a documentation correction — see § Scope (as implemented) and § Out of Scope.

## Historical Reconstruction Notice

See above. This section intentionally repeats no further content, per this document's own precedent (EWO-009 uses the identical structure).

## Problem Statement

At the time GOV-012 was conducted, five published documents contained text that no longer accurately reflected repository evidence:

1. `architecture/ARCH-003-Runtime-Integration-Architecture.md` (then v0.4.0) stated, in five separate locations (§5, §10 twice, §18, §21 diagram), that no actor-defined message-handling logic existed anywhere in the workspace — a statement independently re-confirmed still present, verbatim, in the tracked file text, despite EWO-009/ER-009 (2026-07-15, two days after ARCH-003's own last revision) having closed exactly this gap.
2. `architecture/ARCH-002-Runtime-Architecture.md` (then v0.2.0) §8 enumerated five distinct elements (content, envelope, send authority, deliberately transferred capabilities, operational trace information) while labeling them "four distinct concerns" — an internal arithmetic mismatch, present since the section was first authored, deliberately preserved without correction through RES-002, RSS-001, and ACR-001 in turn, each lacking authority to edit ARCH-002 directly.
3. `adrs/ADR-0013-Architectural-Evolution-of-SynapseOS.md`, `adrs/ADR-0014-Engineering-Standards-Corpus-Reconciliation.md`, and `architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md` each cited STD-001 flatly as "(Approved)" in their own frontmatter. Independent verification (SHA-256 recomputation against two separate, pushed normal-governance evidence commits) confirmed STD-001 versions 0.1.0 and 0.2.0 were each genuinely, evidentially approved — but the current, operative version (0.4.0, which registers the RSS/ACR/AFR document families this corpus now depends on) has never been separately approved. The unqualified citation did not distinguish the two.

## Objectives

1. Correct ARCH-003's stale implementation-baseline statement to accurately reflect EWO-009/ER-009's completion, across every section that referenced it.
2. Correct ARCH-002 §8's internal count-label error.
3. Correct the STD-001 approval-status citation in ADR-0013, ADR-0014, and ARCH-006 to distinguish evidenced-approved versions from unapproved current content.
4. Perform no correction, in any of the above, that alters an architectural principle, ownership boundary, interaction model, Runtime sequencing rule, Trusted Core boundary, design rationale, or ADR decision.

## Scope (as implemented)

- `architecture/ARCH-003-Runtime-Integration-Architecture.md`: frontmatter (version 0.4.0→0.5.0, `last_updated`, STD-001 citation, added ARCH-006 to `related_documents.architecture`, evidentiary commit hash updated from `5ccc7f9083a71adc6ee704b2322a701935765679` to `0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7`); §1 Document Control table (version); §4 (existing-implementation commit hash); §5 (replaced the stale bullet with a description of the closed gap, citing EWO-009/ER-009/ARCH-006); §10 (the "not required by the Minimal Runtime Profile" and "no claim is made" statements updated; real host-level execution correctly retained as still deferred); §18 (removed the now-completed deferred-work item); §21 diagram (replaced the "DEFERRED... does not exist yet" callout); §22 References (added ARCH-006, EWO-009, ER-009; updated commit hash); §23 Change History (added a 0.5.0 entry).
- `architecture/ARCH-002-Runtime-Architecture.md`: §8 ("four" corrected to "five"); frontmatter (version 0.2.0→0.2.1, `last_updated`); §26 Change History (added a 0.2.1 entry).
- `architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md`: frontmatter STD-001 citation corrected; version 0.1.3→0.1.4, `last_updated`; §19 Change History (added a 0.1.4 entry). ADR-0015/ADR-0016/ADR-0017 citations already present in this document's frontmatter were independently re-verified during this task (each backed by a hash-matched normal-governance evidence commit) and found accurate for ADR-0015 and ADR-0016; ADR-0017's own evidence commit was found to contain content inconsistent with this repository's established evidentiary pattern and was **not** corrected either direction, pending the separate governance adjudication the Certification Recovery Review later identified as required (§ Out of Scope).
- `adrs/ADR-0013-Architectural-Evolution-of-SynapseOS.md`: frontmatter STD-001 citation corrected; version 0.1.0→0.1.1, `last_updated`; §16 Change History (added a 0.1.1 entry).
- `adrs/ADR-0014-Engineering-Standards-Corpus-Reconciliation.md`: frontmatter STD-001 citation corrected; version 0.1.0→0.1.1, `last_updated`; §24 Change History (added a 0.1.1 entry).

## Out of Scope

- The GOV-003/GOV-010 "(Approved, Act 2)" citations already present in ADR-0013, ADR-0014, GOV-004, and ARCH-003's own frontmatter — independently re-verified accurate (hash-matched Act 2 evidence commits) and left unchanged.
- RSS-001's own Evidence Map completeness gap (GOV-012-ISS-009) — RSS-001 is not an architecture document; correcting it is outside this EWO's own "Architecture Consistency Corrections" purpose.
- ARCH-003 §18's remaining, unverified items (deterministic cleanup after partial failure, first-runnable-actor demonstration, end-to-end construction-through-shutdown demonstration) — deliberately left unexamined; verifying their current status was outside this task's own precisely bounded scope.
- **The ADR-0017 and ARCH-001 v0.2.0 approval-evidence anomaly**, and the wider "Architecture Review Board" citation pattern subsequently found in ADR-0013, ARCH-002, and EWO-004 by the Architecture Certification Recovery Review — explicitly out of scope for a documentation-reconciliation EWO; STD-001 §46 itself requires that where implementation or documentation work reveals an apparent architectural or governance contradiction, "the EWO itself must require engineering to stop and return the issue for architectural review rather than resolve it unilaterally." That escalation occurred; adjudication remains pending, under separate, future governance work.
- Filing an Engineering Report for this EWO (§ Engineering Authority explains why none is required).
- Any correction to GOV-011, GOV-012, AFR-001, RSS-001, ACR-001, or any RES document.
- Any runtime code change of any kind.

## Repository Constraints

- Only the documentation repository (`synapse-docs`) may change; `synapse-runtime` is read-only reference material, consulted only to independently verify ARCH-003's corrected claim (confirmed: `fn handle` implementations and actor-to-actor messaging tests directly present at `0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7`).
- No staging, committing, pushing, tagging, or merging occurred as part of the corrections this EWO records, nor as part of filing this EWO itself.
- No Git history was altered, rewritten, or rebased.
- **A pre-existing, unrelated modification to `standards/STD-001-Documentation-Standards.md`** was already present in the working tree before this EWO's corrections began, is unconnected to the creation of this EWO or its three corrections, and was not touched by this EWO at any point — neither during the original correction work nor during this document's own retrospective filing. A `git status` or `git diff` taken against this EWO's own `base_state.docs_head` will show `STD-001` as modified; this note discloses, rather than leaves unexplained, why that file appears alongside the five files this EWO actually corrected. No claim is made here about the origin of that unrelated modification beyond this fact.

## Evidence Reviewed

STD-001 §46; ARCH-002, ARCH-003, ARCH-006 (pre- and post-correction text, directly diffed); ADR-0013, ADR-0014 (pre- and post-correction text); GOV-012's own decision records for the four implemented findings; ACR-001 §7.11 (GAP-002), §9 (GAP-005, GAP-008, GAP-009), and §13 (Cross-Architecture Findings) as the originating evidentiary basis GOV-012 itself relied on; the STD-001 Approval Status section and its two cited normal-governance evidence commits (`7b0fdc3`, `252a368`), independently re-verified by direct SHA-256 recomputation; the `synapse-runtime` working tree at `0e4c5c9e8d27be6bfb00fb26ef19995258ad7ed7`, consulted directly for the ARCH-003 correction; and AFR-001's own independent re-confirmation that all five corrections are present and accurate in the current tracked files.

## Corrections Performed

See Scope (as implemented), above, for the complete, file-by-file record. In summary: three distinct correction classes (stale implementation status; internal count-label error; imprecise approval-status citation), applied across five files, each traced to a specific GOV-012 finding, each independently re-verified twice (by this task's own validation pass, and subsequently by AFR-001).

## Files Affected

| File | Before | After |
|---|---|---|
| `architecture/ARCH-003-Runtime-Integration-Architecture.md` | v0.4.0 | v0.5.0 |
| `architecture/ARCH-002-Runtime-Architecture.md` | v0.2.0 | v0.2.1 |
| `architecture/ARCH-006-Runtime-Actor-Execution-Architecture.md` | v0.1.3 | v0.1.4 |
| `adrs/ADR-0013-Architectural-Evolution-of-SynapseOS.md` | v0.1.0 | v0.1.1 |
| `adrs/ADR-0014-Engineering-Standards-Corpus-Reconciliation.md` | v0.1.0 | v0.1.1 |

No file beyond these five was modified by the corrections this EWO records. This document's own filing modifies no other file.

## Verification Requirements

Already satisfied, per the corrections' own execution: frontmatter YAML validity (all five files); Markdown table pipe-count consistency (all five files); absence of residual stale phrasing outside quoted historical Change History text; `git status` confirmation that exactly the five intended files, and no others, carried uncommitted modifications at the time of correction. Independently re-confirmed a second time by AFR-001.

## Completion Criteria

All five files carry the corrected text, an incremented version, an updated `last_updated` date, and a Change History entry describing the correction, citing this EWO by name. All five conditions are satisfied as of this document's own authoring — confirmed by direct inspection of each file's current tracked content immediately before this EWO was drafted.

## Stop Conditions

None were triggered during the original correction work. One was triggered during a *subsequent*, separate task (Architecture Certification Recovery Review) and is recorded here for traceability only, not resolved by this EWO: the ADR-0017/ARCH-001 v0.2.0 evidence-provenance anomaly required this EWO's own execution to stop short of correcting or endorsing those two citations, and to record the matter as requiring governance adjudication instead (§ Out of Scope).

## Expected Deliverables

The five corrected files themselves (already delivered, prior to this document's own authoring) and this EWO document (delivered now, as a retrospective record).

## Final Verification Requirements

A future independent review of this EWO should confirm: (a) this document's own `created`/`last_updated` dates are not earlier than the actual corrections' own execution date; (b) no file other than this new EWO document was modified by this filing; (c) the five files' own Change History entries citing "EWO-010" now resolve to a real, filed document at the expected path; (d) this document makes no claim about the separate, unresolved ADR-0017/ARCH-001/ADR-0013/ARCH-002/EWO-004 provenance question beyond acknowledging it exists and is out of scope.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-17 | Denver Jacobs | Initial Draft. Historical reconstruction of the Architecture Consistency Corrections milestone: the first genuine controlled-document record of a documentation-reconciliation task executed the same day, implementing three GOV-012 findings — GOV-012-ISS-013 (ARCH-003 currency), GOV-012-ISS-010 (ARCH-002 §8 count label), and GOV-012-ISS-012 (STD-001 citation precision in ADR-0013/ADR-0014/ARCH-003, with an equivalent ARCH-006 instance discovered independently during execution) — across five files, all independently re-verified twice before this filing. A fourth investigated finding, GOV-012-ISS-011, required no correction for its ADR-0015/ADR-0016 portion and, for its ADR-0017 portion, a governance adjudication outside this EWO's own authority, rather than a documentation correction. Restores the governance record an Architecture Certification Recovery Review found missing (a Gate 4 defect in AFR-001), without altering any of the five already-corrected files, any Git history, or any evidence commit. Explicitly excludes the separate, unresolved ADR-0017/ARCH-001 v0.2.0 provenance anomaly from its own scope. |
| 0.1.1 | 2026-07-17 | Denver Jacobs (AI-assisted) | Corrected per the Independent Technical Review's own findings (verdict: CHANGES REQUIRED): swapped the GOV-012-ISS-010/GOV-012-ISS-013 misattribution (the ARCH-003 fix is GOV-012-ISS-013; the ARCH-002 §8 fix is GOV-012-ISS-010); removed the erroneous "GOV-012-ISS-013" co-citation from the STD-001 citation-precision fix, leaving GOV-012-ISS-012 alone; disclosed, precisely, that GOV-012-ISS-012 names only ADR-0013, ADR-0014, and ARCH-003, and that the ARCH-006 instance of the same defect was an execution-discovered finding, not one GOV-012-ISS-012 itself identified; corrected the GOV-011 citation from an unverifiable "§4.3" to the two locations (§5 item 6, §8 item 3) where the quoted "creates an obligation" language actually appears, and clarified that GOV-011 does not use that phrase for "Accepted with Clarification" decisions; and disclosed the pre-existing, unrelated STD-001 modification a base-state diff would otherwise leave unexplained. No correction to any of the five already-completed documentation fixes was made or implied; this revision corrects only this EWO's own internal citations and disclosures. |

## Disposition

Not yet reviewed. Not yet approved. This EWO's own filing restores a historical record; it does not itself require, and has not received, a fresh disposition act distinct from the corrections it records, which were already independently re-verified twice before this document existed.
