---
document_id: ADR-0014
title: Engineering Standards Corpus Reconciliation
version: 0.1.1
status: Draft
decision_owners:
  - Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-11
last_updated: 2026-07-17
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
  standards:
    - STD-001 (v0.1.0 and v0.2.0 Approved via normal-governance disposition; current v0.4.0 content not separately approved — see STD-001's own Approval Status section)
    - STD-002 through STD-030 (Draft — legacy corpus this ADR governs the reconciliation of; none amended by this ADR)
  architecture:
    - ARCH-000 (Draft — historical record, unmodified)
    - ARCH-001 (Draft — constitutional foundation)
    - ARCH-002 (Draft — Runtime architecture)
  rfcs: None
  adrs:
    - ADR-0011 (Draft, Act 1 effective)
    - ADR-0012 (Approved)
    - ADR-0013 (Draft — architectural evolution basis for this document)
  roadmap: None
  research: None
  operational: None
  source_artifacts: None
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ADR-0014 — Engineering Standards Corpus Reconciliation

*Filename pattern: `ADR-0014-Engineering-Standards-Corpus-Reconciliation.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Context

A read-only audit of `standards/STD-002` through `STD-030` — 29 Draft, `.docx`-format documents predating the constitutional architecture — found a substantial, largely sound engineering standards corpus that has never been reconciled with ADR-0013's Intelligence Operating System identity or ARCH-001/ARCH-002's constitutional model. The audit classified each document (still valid, partially valid, requiring reconciliation, or conflicting), found one duplicate file, one genuine terminology collision (STD-017's "Approval Classes" versus GOV-010 §4's "Decision Classes"), and several high-value reconciliation candidates whose existing content already anticipates concepts ARCH-002 later formalized.

This ADR does not perform that reconciliation. It exists because reconciling 29 documents without first agreeing on precedence, identifier permanence, amendment-versus-replacement policy, and duplication rules would produce 29 independently-reasoned, potentially inconsistent outcomes — exactly the kind of drift this project's governance discipline exists to prevent.

## 2. Problem

Three questions must be answered once, consistently, before any individual standard is touched: what governs what when a legacy standard's content and the now-frozen constitutional architecture appear to disagree; whether reconciliation should amend documents in place or replace them; and how overlap, terminology drift, and duplication — already observed within the corpus — should be resolved without turning every reconciliation into a bespoke judgment call.

## 3. Decision Drivers

- STD-001 §7's identifier-permanence rule already governs this repository and must be applied consistently, not re-litigated per document.
- ARCH-001 §10 freezes constitutional concepts against redefinition "through demonstrated architectural contradiction or significant architectural improvement" only, via the ADR process — any reconciliation rule that let a Standard silently override a constitutional concept would violate this directly.
- GOV-003 §5's document hierarchy ("Governance → Standards → Architecture → ADRs → RFCs → Implementation → Testing → Release") is stated as a bare ordering with no elaboration of what "precedence" means when a constitutional freeze is in effect — this ADR is the first place that ambiguity has needed to be resolved, and resolving it once, explicitly, avoids 29 separate ad hoc interpretations.
- The GOV-003/GOV-010/GOV-004 amendment-strategy-then-correction-then-validation pattern, and STD-001 §31's "generalize, don't duplicate" resolution, are both already-proven, working precedents this reconciliation should reuse rather than reinvent.
- The audit found zero documents whose core purpose is invalidated by the constitutional architecture — the corrective work is reconciliation, not replacement, in every case examined.

## 4. Decision

The Standards corpus (STD-002 through STD-030) will be reconciled with the constitutional architecture through targeted amendment of individual documents, one at a time, under the rules this ADR establishes. No document is retired, replaced, renumbered, or merged as a consequence of this ADR. Precedence, identifier permanence, amendment policy, overlap resolution, terminology-drift resolution, cross-reference policy, and reconciliation order are fixed here; applying them to any specific document remains a separate, later, individually-authorized task.

## 5. Precedence: Governance, Architecture, Standards, and Implementation

GOV-003 §5's hierarchy is a workflow ordering — the sequence in which authority, design, and delivery activities occur — not an override rule. It must be read together with, not in place of, ARCH-001 §10's constitutional freeze. The two are reconciled as follows:

- For **constitutional matters** — the concepts ARCH-001 §5 fixes (Actor, Capability, Message, Execution Semantics) and the laws in ARCH-001 §6 — ARCH-001 is unconditionally authoritative. No Standard may override, contradict, weaken, or silently redefine a constitutional concept. A Standard that appears to require this has surfaced a potential architectural contradiction, to be raised through the ADR process exactly as ARCH-001 §10 already requires — not resolved by amending the Standard alone.
- For **Runtime-architectural matters** — anything ARCH-002 fixes (the trusted-core boundary, the component model, the constitutional execution cycle) — ARCH-002 is authoritative in the same sense, subject to its own deferred-architecture contracts (ARCH-002 §23), which Standards may legitimately fill in.
- For **engineering practice matters** — coding style, git workflow, testing discipline, dependency policy, naming, and the other genuine subject matter of the STD-002–030 corpus — Standards remain authoritative *within* the bounds Architecture sets. This is what GOV-003 §5's "approved standards constrain architecture and implementation" means in practice: Standards govern *how* implementation and architecture documentation are carried out, not *what* the constitutional architecture is.
- Where a Standard's practice guidance and an ARCH document's structural requirement appear to conflict, the ARCH document controls, and the Standard is amended to conform — the same rule already applied when GOV-004 was reconciled against GOV-003/GOV-010, and when STD-001 itself was extended rather than contradicted.

## 6. Identifier Permanence

STD-001 §7 already establishes that identifiers are permanent and a retired or superseded identifier must not be reassigned. This ADR adds no new rule; it states explicitly that this applies without exception to the entire legacy corpus. STD-002 remains, and will always remain, "Coding Standards." No future task may mint a document under an already-occupied STD identifier, mirroring the resolution already reached for `ARCH-000-Introduction.md`.

## 7. Amendment Versus Replacement

Amendment is preferred over replacement for every document the audit classified as still valid, partially valid, or requiring reconciliation — which was every document examined. Replacement (a new document under a new identifier, with the old one formally superseded per STD-001 §12) is reserved for the case where a document's core purpose, not merely its wording, is invalidated by the constitutional architecture. The audit found no such case. Most reconciliation work is expected to be MINOR under STD-001 §13's versioning standard — added cross-references, corrected terminology, clarified precedence — not a rewrite, consistent with how GOV-003, GOV-010, and GOV-004 were each reconciled through targeted amendment rather than replacement.

## 8. Overlap Resolution

Where two or more Standards address the same topic at different depths (for example, STD-002 §14–16 on dependencies versus the dedicated STD-011), the more specific, dedicated document is authoritative for that topic. The more general document is amended to cross-reference the dedicated one rather than restate its content. This is the same discipline STD-001 §31 already applied when generalizing ADR-0012's evidence model rather than duplicating it.

## 9. Terminology Drift Resolution

A term that now carries a precise constitutional meaning (Actor, Capability, Message, Execution Semantics, Trusted Core, Runtime) must, where a legacy Standard uses it with a different or looser meaning, be either corrected to the constitutional meaning or explicitly disambiguated as an intentionally distinct, non-conflicting usage. Not every match requires a rename: the audit found STD-010's `actorId` log-field name uses "actor" in an ordinary, compatible, non-conflicting sense and needs no correction, whereas STD-029's references to "the kernel" use the superseded Phase 1/2 RFC-0002 sense and should be updated to reference the Trusted Core or Runtime as ARCH-002 now defines them. This is a judgment applied per instance during individual reconciliation, not a blanket search-and-replace mandate.

## 10. Obsolete Architectural Reference Resolution

Any reference to an architectural artifact, mechanism, or plan that ARCH-001 or ARCH-002 has superseded or rendered moot — for example STD-013's reference to a future "kernel RFC" that never materialized in that form — must be updated to cite the actual, current authoritative document instead. This is corrective, not interpretive: the current authoritative reference already exists and only needs to be substituted.

## 11. Introducing Constitutional Terminology into Legacy Standards

Reconciliation may introduce explicit citations to ARCH-001 or ARCH-002 where a Standard's existing content already anticipates a constitutional concept it predates — for example STD-007 §19's "capability-based control" recommendation, which can now cite ARCH-001 §5.2 and ARCH-002 §9 directly instead of speaking generically. This strengthens the Standard's authority and removes an independent, duplicate definition of the same idea. Reconciliation must not, in the course of adding such citations, expand a Standard's own prescriptive detail beyond its domain — a Standard should cite the constitutional model, not restate it, consistent with `.ai/ARCHITECTURAL-CONTEXT.md`'s "avoid unnecessary abstraction" principle and STD-001 §31's own generalize-rather-than-duplicate precedent.

## 12. Cross-Reference Versus Duplication Policy

Cross-reference is required, not merely preferred, wherever a single authoritative source now exists for a constitutional or architectural concept — no Standard should carry its own competing definition of what a capability, an actor, or the trusted core is. This is distinct from cross-cutting engineering practices, such as secrets handling, which the audit found restated across twelve documents: STD-001 §4 already establishes that controlled documents are self-contained, and a domain-appropriate restatement of a general practice (STD-026's "do not persist secrets in storage backups" versus STD-029's "do not expose secrets to plugin code") is not true duplication and does not need to be collapsed into a single cross-referenced source. The test is whether two passages assert the *same* thing about the *same* concept (true duplication, to be resolved by cross-reference) or apply a *shared principle* to *different* domains (acceptable restatement, left alone).

## 13. Retirement Strategy

Retirement (STD-001 §12's Superseded, Deprecated, or Archived lifecycle states) remains available as a mechanism but is not indicated for any document in the audited corpus — every document was classified as still valid, partially valid, or requiring reconciliation, none as obsolete in purpose. Retirement should be considered only if a future reconciliation pass finds a document's entire subject matter has been fully absorbed elsewhere, and even then only as the outcome of that document's own dedicated reconciliation task, never pre-emptively or in bulk.

## 14. Future Lifecycle of Engineering Standards

Reconciled Standards follow STD-001 §12's existing lifecycle (Draft → Review → Approved) and STD-001 §31's Approval Evidence Representation model for recording their eventual approval — the same mechanism already used for GOV-004 and STD-001 itself, requiring no new evidence scheme. Approval authority for Standards remains GOV-003 §3.2's Chief Architect role (Class B per GOV-010 §5), currently vacant with the interim Founder default, the same authority chain already exercised throughout this engagement. Once reconciled and approved, a Standard's citations of constitutional concepts inherit ARCH-001 §10's change-control discipline: a future amendment to a reconciled Standard that would contradict ARCH-001 or ARCH-002 must be raised through the ADR process, not resolved locally within the Standard.

## 15. Recommended Reconciliation Order

1. Hygiene with no decision required: the STD-013 duplicate file.
2. The STD-017 / GOV-010 approval-class terminology collision — the single finding with the most active potential for confusion if left unresolved.
3. The audit's highest-value reconciliation candidates, whose existing content already anticipates ARCH-002: STD-025 (Distributed Systems & Messaging), STD-010 (Observability), STD-022 (Tool & Action Execution), STD-023 (Resilience), STD-029 (SDK/Plugin), STD-026 (Storage).
4. The foundational engineering-practice documents most immediately load-bearing for continued Runtime implementation: STD-002 (Coding), STD-004 (Repository), STD-011 (Dependency Management) — STD-004 in particular is time-sensitive, since `synapse-runtime` already exists and was scaffolded without reference to it.
5. Remaining still-valid documents requiring only minor extension (STD-006, STD-007, STD-008, STD-009, STD-013, STD-016, STD-020, STD-024).
6. A scope discussion, not a rewrite, for STD-018 and STD-030, whose character reads closer to governance/policy than engineering-implementation standard.

Each step remains a separate, individually-authorized task; this ordering is a recommendation, not an authorization to proceed.

## 16. Scope

This ADR: establishes precedence between Governance, Architecture, Standards, and Implementation for the purpose of reconciliation; reaffirms identifier permanence for the legacy corpus; sets amendment-over-replacement policy; and fixes rules for overlap, terminology drift, obsolete references, constitutional-terminology introduction, cross-referencing, retirement, and future lifecycle.

## 17. Non-Scope

This ADR does not amend, rewrite, rename, renumber, merge, retire, or convert any STD document. It does not resolve the STD-017/GOV-010 terminology collision, remove the STD-013 duplicate file, or reconcile any individual Standard's content — each remains a separate, later, individually-authorized task following the order in §15. It does not introduce a new engineering standard. It does not change any constitutional concept, redefine Runtime architecture, or alter GOV-003, GOV-010, STD-001, ADR-0011, ADR-0012, ADR-0013, ARCH-001, or ARCH-002.

## 18. Rationale

Fixing precedence, identifier permanence, and the amendment/overlap/terminology/cross-reference rules once, in a single ADR, is what allows the 29 individual reconciliation tasks §15 orders to proceed as narrow, mechanical applications of an already-settled policy rather than as 29 separate occasions to re-derive the same governance questions — precisely the discipline that made the GOV-003/GOV-010/GOV-004 and STD-001 §31 reconciliations tractable rather than open-ended.

## 19. Consequences

- Every future individual Standard reconciliation task can cite this ADR for precedence, identifier permanence, and duplication policy instead of re-litigating them.
- The STD-017/GOV-010 collision and the STD-013 duplicate file remain open until their own dedicated tasks address them; this ADR does not close either.
- `synapse-runtime`'s existing scaffold is not retroactively judged against STD-004 by this ADR; that comparison is deferred to STD-004's own reconciliation task, prioritized in §15.
- STD-018 and STD-030's family placement (STD versus GOV) remains an open question this ADR flags but does not resolve.

## 20. Risks

The precedence rule in §5 is the most consequential clarification in this ADR and rests on an interpretation of GOV-003 §5's bare, unelaborated ordering — a reasonable, carefully-reasoned interpretation, but an interpretation nonetheless, since GOV-003 §5 itself does not spell out what "precedence" means under a constitutional freeze. If this interpretation later proves wrong, correcting it requires amending this ADR, not silently reinterpreting it during an individual Standard's reconciliation. The "restatement versus duplication" test in §12 requires judgment per instance and could be applied inconsistently across 29 separate future tasks without care.

## 21. Relationship to Previous Documents

This ADR builds directly on the read-only audit's findings, treating them as accepted input, not re-derived here. It preserves ADR-0013's architectural-evolution decision and ARCH-001/ARCH-002's constitutional and Runtime architecture unchanged, and extends the same governance discipline already applied to GOV-003, GOV-010, GOV-004, and STD-001 to a corpus that had, until the audit, never been examined against any of them.

## 22. Validation

This ADR's precedence interpretation (§5) was checked directly against GOV-003 §5 and ARCH-001 §10's exact text, not recalled from memory, given how much of this ADR's usefulness depends on getting that reconciliation right.

## 23. References

- `standards/STD-002_Coding_Standards_v0.1.docx` through `STD-030_Research_Experimentation_and_Future_Capability_Standards_v0.1.docx` — the audited corpus, unmodified by this ADR.
- `governance/GOV-003-Governance-Model.md` §5, §8 — document hierarchy, precedence.
- `governance/GOV-010-Decision-Framework.md` §4, §5 — Decision Classes, Decision Authority.
- `standards/STD-001-Documentation-Standards.md` §4, §7, §12, §13, §31 — self-containment, identifier permanence, status lifecycle, versioning, approval evidence representation.
- `architecture/ARCH-001-Constitutional-Architecture.md` §5, §6, §10 — constitutional concepts, laws, change control.
- `architecture/ARCH-002-Runtime-Architecture.md` §23 — deferred architecture and their contracts.
- `adrs/ADR-0013-Architectural-Evolution-of-SynapseOS.md` — architectural identity this reconciliation extends to the Standards corpus.
- The legacy-standards-corpus read-only audit (conducted in this engagement; not itself a controlled document).

## 24. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-11 | Denver Jacobs | Initial Draft, establishing the governance approach for reconciling the legacy engineering standards corpus (STD-002 through STD-030) with the constitutional architecture: precedence between Governance, Architecture, Standards, and Implementation; identifier permanence; amendment-over-replacement policy; overlap, terminology-drift, obsolete-reference, and cross-reference resolution rules; retirement strategy; future Standards lifecycle; and a recommended reconciliation order. No individual Standard is amended by this ADR. No approval act has occurred. |
| 0.1.1 | 2026-07-17 | Denver Jacobs (AI-assisted, EWO-010) | Editorial correction only, per EWO-010 (Architecture Consistency Corrections): corrected the frontmatter citation of STD-001 from a flat "(Approved)" to distinguish its genuinely evidenced approval at versions 0.1.0 and 0.2.0 (independently verified: two separate normal-governance evidence commits, each hash-matched against the content approved) from its current, operative version (0.4.0), which has never been separately approved. GOV-003 and GOV-010's own "(Approved, Act 2)" citations were independently re-verified during the same task and found accurate — no correction was needed or made to those two. No decision recorded in this ADR was reopened, and no other content changed. |

## 25. Approval Status

### 25.1 Document State

| Field | Value |
|-------|-------|
| Lifecycle status | Draft |
| Normal-governance disposition | Not yet approved |
| Disposition date | Not yet assigned |
| Approving actor | Not yet assigned |

### 25.2 Approval Evidence (per STD-001 §31)

| Field | Value |
|-------|-------|
| Document ID | ADR-0014 |
| Repository path | adrs/ADR-0014-Engineering-Standards-Corpus-Reconciliation.md |
| Version | 0.1.0 |
| Artifact revision identifier | Not yet created |
| Content fingerprint | Not yet created |
| Git blob ID | Not yet created |
| Disposition | Not yet approved |
| Disposition type | Not yet assigned |
| Approver identity | Not yet assigned |
| Authority citation | Not yet assigned |
| Effective date | Not yet assigned |
| Review evidence | Not yet created |
| Independent-review status | Not yet created |
| Self-approval or conflict disclosure | Not yet created |
| Known limitations | Not yet created |
| Unresolved issues | Not yet created |

No field in this section may be populated until the corresponding act has genuinely occurred, evidenced per STD-001 §31. This table does not, and must not be read to, claim that any approval of ADR-0014 has occurred.
