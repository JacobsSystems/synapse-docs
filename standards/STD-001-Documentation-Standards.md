---
document_id: STD-001
title: Documentation Standards
volume: Volume II – Standards
version: 0.3.1
status: Draft
author: Denver Jacobs
reviewers: TBD
created: 2026-07-09
last_updated: 2026-07-12
classification: Public
owner: SynapseOS Project
approval_authority: Founder / Designated Architecture Authority
related_documents: GOV-001 through GOV-010
---

# STD-001 – Documentation Standards

**SynapseOS Engineering Manual — Volume II – Standards**
**Version 0.3.1 (Draft)**
**Controlled Engineering Standard**

> This Markdown document is the canonical source-format conversion of `STD-001_Documentation_Standards_v0.1.docx`. The original Word document is retained as the source artifact pending a future controlled archival decision.

## Document Control

| Field | Value |
|---|---|
| Document ID | STD-001 |
| Title | Documentation Standards |
| Volume | Volume II – Standards |
| Version | 0.3.1 |
| Status | Draft |
| Author | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-09 |
| Last Updated | 2026-07-12 |
| Classification | Public |
| Owner | SynapseOS Project |
| Approval Authority | Founder / Designated Architecture Authority |
| Related Documents | GOV-001 through GOV-010 |

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-09 | Denver Jacobs | Initial controlled draft |
| 0.2.0 | 2026-07-11 | Denver Jacobs | MINOR amendment (§13): registered Engineering Work Orders (EWO) and Engineering Reports (ER) as controlled document families, closing the governance gap identified during review of `ENGINEERING-PROGRAM.md`. Additive only — no existing obligation removed, weakened, or reinterpreted. See new §46, §47 and amended §5, §6, §7, §10, Appendix B. |
| 0.3.0 | 2026-07-12 | Denver Jacobs | MINOR amendment: registered Engineering Maintenance Orders (EMO) and Engineering Maintenance Reports (EMR) as controlled document families, closing the governance gap identified when a post-completion implementation conformance defect was discovered in a milestone already delivered under an EWO/ER, with no existing mechanism to correct it without rewriting completed engineering history. Additive only — no existing obligation removed, weakened, or reinterpreted; EWO and ER are unchanged and EMO/EMR supplement rather than replace them. See new §48, §49 and amended §5, §6, §7, §10, Appendix B. |
| 0.3.1 | 2026-07-12 | Denver Jacobs | Final governance review refinement of the 0.3.0 EMO/EMR amendment, still Draft/unapproved. Tightened §6 and §48 to state explicitly that an EMO creates no independent engineering authorization path and derives its authority only from already-approved architecture/standards and the specific EWO it maintains, authorizing nothing outside that EWO's scope. Added explicit "Mandatory traceability" MUST-lists to §48 and §49 (EMO: EWO maintained, ER related to; EMR: EMO implemented, EWO maintained, ER supplemented), replacing §49's prior SHOULD-level phrasing. Tightened §48's MUST-NOT list to use the exact prohibited-action phrasing reviewed (authorize new engineering work; bypass or replace the ADR process) and added a summary sentence stating an EMO exists solely to maintain conformance of an already-completed deliverable. Added an explicit "EMO/EMR supplement history; they never replace it" statement to §48 and §49. Clarified in §7 that EMO/EMR numbering is independent not only of EWO/ER numbering but of any informal milestone label (e.g. "SRP-NNN"), which is not itself a controlled identifier family, and that an EMO/EMR number MUST NOT be chosen to numerically match the artifact it relates to. No new section, no new document family, no change to EWO/ER's own text, no change outside the 0.3.0 amendment's own scope. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs | Drafted | 2026-07-09 |
| Technical Review | TBD | Pending | |
| Approval Authority | TBD | Pending | |

> **Note on version state:** A content-non-mutating approval evidence commit ("STD-001 Normal-Governance Approval Evidence Record") recorded Founder approval of this document at version 0.1.0, content fingerprint `47506d13de53d393f75a23a436f2cce08ab516633df46669957711b694d1c8c0` (blob `f370ef79218658367546bf0209d66bedd23df2f8`). Per §31.5, that evidence does not modify this artifact's tracked metadata, which is why the table above still reads as drafted/pending — that is expected and correct under this document's own convention. Version 0.2.0, introduced by the amendment described in the Revision History above, is a distinct, subsequent, **not yet approved** artifact identity. The 0.1.0 evidence record disposes of 0.1.0 only and must not be read as approving 0.2.0.

## 1. Purpose

This standard defines mandatory requirements for creating, naming, structuring, reviewing, approving, versioning, storing, linking, maintaining and retiring controlled documentation within the SynapseOS project.

The objective is to ensure that documentation remains understandable to people and AI-assisted engineering tools without relying on undocumented conversation history. Documentation is a first-class engineering artifact and forms part of the project's institutional memory.

## 2. Scope

This standard applies to all official SynapseOS governance documents, standards, architecture specifications, RFCs, ADRs, roadmaps, research records, engineering playbooks, operational procedures, templates, meeting records, decision records and repository-level documentation.

Informal scratch notes are not controlled documents unless promoted into the controlled documentation system.

## 3. Normative Language

The keywords MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY and OPTIONAL indicate requirement strength.

- MUST / SHALL: mandatory.
- SHOULD: expected unless a documented reason justifies deviation.
- MAY: optional.
- TBD: unresolved and requiring future action.
- N/A: deliberately not applicable.

## 4. Documentation Principles

- Documentation is maintained close to the work it governs.
- Material decisions are traceable.
- Documents are self-contained.
- Approved documents are not silently rewritten.
- Superseded material remains historically discoverable.
- The lightest adequate documentation process is preferred.
- Documentation must not become a substitute for shipping working software.
- AI-generated content remains draft until appropriately reviewed.

## 5. Controlled Document Families

- GOV – Governance and organisational direction.
- STD – Mandatory engineering and operational standards.
- ARCH – Architecture specification and system design.
- RFC – Proposed implementation contracts and significant technical changes.
- ADR – Records of significant architectural decisions.
- ROAD – Roadmaps and milestone plans.
- RES – Research and investigation records.
- OPS – Operational procedures and runbooks, when introduced.
- TPL – Controlled templates, when a unique identifier is required.
- EWO – Engineering Work Orders. Authorize specific, scoped engineering implementation tasks within governance, architecture and standards already in force. See §46.
- ER – Engineering Reports. Record completed engineering work performed under an EWO. Informational only; creates no new requirement. See §47.
- EMO – Engineering Maintenance Orders. Authorize narrowly scoped maintenance work on a completed engineering milestone, without rewriting its original EWO or ER. See §48.
- EMR – Engineering Maintenance Reports. Record completed maintenance work performed under an EMO. Informational only; creates no new requirement. See §49.

Repository README files and local guides may remain unnumbered where they are navigational rather than authoritative. `ENGINEERING-PROGRAM.md` is one such document: it introduces EWO and ER informally, but this section is what actually registers them as controlled document families.

## 6. Document Hierarchy and Precedence

Where documents conflict, the issue must be resolved rather than ignored. As a default governance rule, approved governance constraints take precedence over subordinate standards; approved standards constrain architecture and implementation; architecture defines system intent; ADRs record specific decisions; RFCs define approved implementation contracts.

A later approved document does not automatically override an earlier higher-authority document unless the change is explicit and traceable.

An Engineering Work Order sits below Governance, Architecture and Standards in this precedence order: it is normative only for the specific engineering task it authorizes, and it may not redefine, override, or reinterpret any governance, architecture, or standards document. Where implementation performed under an EWO appears to contradict architecture, the EWO itself must require engineering to stop and escalate for architectural review rather than proceed on the EWO's authority alone. An Engineering Report carries no independent authority at all — it is a record of what happened, not a source of new obligation, and creates no requirement that did not already exist.

An Engineering Maintenance Order creates no independent engineering authorization path and carries the same authority as, and no more than, an Engineering Work Order: normative only for the narrow maintenance task it authorizes, and subject to the same prohibition on redefining, overriding, or reinterpreting governance, architecture, or standards. Its authority is derived exclusively from the already-approved architecture and standards it restores conformance to, and from the specific, completed EWO it maintains — it authorizes nothing outside that EWO's own scope, and grants engineering no authorization that does not already trace back to one of those two sources. An EMO exists only to restore implementation conformance to architecture or standards that already govern it; it is never a vehicle for a new architectural decision. Where maintenance work performed under an EMO exposes an apparent architectural deficiency, the EMO itself must require engineering to stop and route the issue through the existing ADR process rather than resolve it unilaterally — the same escalation rule §46 already states for an EWO. An Engineering Maintenance Report carries no independent authority, on the same basis as an Engineering Report.

## 7. Identifier Standard

Controlled identifiers use an uppercase family prefix, hyphen and zero-padded sequence number.

Examples:

```
GOV-001
STD-001
ARCH-000
RFC-0001
ADR-0001
EWO-001
ER-001
EMO-001
EMR-001
```

EWO, ER, EMO, and EMR use a three-digit zero-padded sequence number, consistent with GOV/STD/ARCH numbering, each family numbered independently starting at 001. EMO and EMR are numbered independently of the EWO/ER they relate to — an EMO's own sequence number does not encode which EWO it maintains; that relationship is carried in metadata (§48, §49), since one completed milestone may require more than one later maintenance action, and maintenance is issued in the order it arises, not in the order of the milestones it concerns. EMO and EMR numbering is likewise independent of any informal milestone label used in engineering discussion (for example, an "SRP-NNN" designation) — such labels are not a controlled identifier family under this section, and an EMO/EMR number MUST NOT be chosen to numerically match an EWO, ER, or milestone label it relates to.

Identifiers are permanent. A retired or superseded identifier must not be reassigned to a different document.

## 8. File Naming Standard

Canonical repository filenames SHALL use:

```
TYPE-NNN-Short-Descriptive-Title.md
```

Examples:

```
STD-001-Documentation-Standards.md
ARCH-000-Introduction.md
RFC-0001-Core-Lifecycle.md
ADR-0001-Core-Language-Selection.md
```

Use hyphens between words. Avoid spaces in canonical Markdown filenames. Keep titles concise but unambiguous.

## 9. Source-of-Truth Policy

The canonical engineering source of truth SHOULD be version-controlled Markdown in the synapse-docs repository.

Word and PDF documents MAY be produced as review, print, approval or publication artifacts. Binary exports must not silently become more authoritative than the controlled source unless governance explicitly designates them as signed records.

During the current project bootstrap period, Word drafts may be created first. Once approved, their authoritative content should be synchronised into canonical Markdown.

## 10. Repository Location Standard

Controlled documentation should be stored by purpose:

- `governance/` – GOV documents
- `standards/` – STD documents
- `architecture/` – ARCH documents
- `rfcs/` – RFC documents
- `adrs/` – ADR documents
- `roadmap/` – roadmap material
- `research/` – research records
- `glossary/` – shared terminology
- `diagrams/` – source diagram definitions
- `templates/` – controlled templates
- `meeting-notes/` – meeting records
- `decisions/` – non-ADR decision registers where required
- `assets/` – approved images and supporting assets
- `scripts/` – documentation automation
- `work-orders/` – EWO documents
- `engineering-reports/` – ER documents
- `maintenance/` – EMO and EMR documents

The repository root `README.md` provides navigation and onboarding. This location guidance registers where EWO, ER, EMO, and EMR documents belong once issued; it does not itself issue one, and the directories above are not created by this amendment.

## 11. Mandatory Metadata

Every controlled document MUST identify at minimum:

- Document ID
- Title
- Version
- Status
- Author or owner
- Created date
- Last updated date
- Classification
- Related documents where applicable

Documents SHOULD additionally identify reviewers, approval authority, implementation status, superseded documents and review date when relevant.

## 12. Document Status Lifecycle

- Draft – Work in progress and non-authoritative.
- Review – Submitted for formal review.
- Approved – Authoritative within its defined scope.
- Implemented – Optional status for specifications whose implementation is verified.
- Superseded – Replaced by another controlled document.
- Deprecated – Still available but scheduled for retirement.
- Archived – Retained for history and no longer maintained.
- Rejected – Formally considered but not accepted, primarily for RFCs or proposals.

Status transitions must be traceable. See §31 (Approval Evidence Representation) for how an approval disposition is evidenced.

## 13. Versioning Standard

Controlled documents use MAJOR.MINOR.PATCH versioning.

- PATCH: editorial corrections that do not change normative meaning.
- MINOR: backward-compatible additions, clarifications or new sections.
- MAJOR: material changes to obligations, architecture intent, governance meaning or compatibility.

Draft documents may begin at 0.1.0. Initial formal approval normally establishes 1.0.0.

Note: recording approval evidence under §31 does not itself require or imply a version change to the approved artifact; see §31.5.

## 14. Date Standard

Dates SHALL use ISO 8601 calendar format: YYYY-MM-DD.

Example: 2026-07-09.

Ambiguous numeric formats such as 09/07/26 SHOULD NOT be used in controlled metadata.

## 15. Required Core Structure

Unless a document-family template defines otherwise, a controlled document SHOULD contain:

1. Purpose
2. Scope
3. Background or Context
4. Requirements / Design / Policy
5. Roles and Responsibilities where relevant
6. Risks and Constraints
7. Open Questions where unresolved matters exist
8. References
9. Revision History
10. Approval information where formal approval applies (see §31, Approval Evidence Representation)

Sections that do not apply may be omitted rather than filled with meaningless boilerplate.

## 16. Heading Standard

Use one H1 document title in Markdown. Main sections use H2; subsections use H3; deeper detail uses H4 only when needed.

Do not skip heading levels merely for visual appearance. Heading text should be descriptive and stable enough for linking.

## 17. Writing Style

Documentation SHALL use clear, professional and direct language. It SHOULD:

- Prefer active voice.
- Define uncommon acronyms on first use.
- Avoid unexplained jargon.
- Separate requirements from aspirations.
- Distinguish current behaviour from future intent.
- Avoid marketing claims in technical specifications.
- Avoid pretending uncertain information is settled.
- Remain understandable without access to ChatGPT or Claude conversation history.

## 18. Requirement Writing

Normative requirements should be testable where practical. Avoid vague statements such as 'the system should be secure' without defining applicable controls or references.

Good requirement: `Provider credentials MUST NOT be written to application logs.`

Weak requirement: `Handle credentials carefully.`

## 19. Lists and Procedures

Use unordered lists for collections without sequence. Use numbered lists for procedures where order matters.

A procedure SHOULD state prerequisites, steps, expected outcome, failure handling and rollback where operational risk warrants it.

## 20. Tables

Tables SHOULD be used for structured comparisons, status registers, metadata, compatibility matrices and concise requirement mappings.

Do not force long narrative content into tables when paragraphs are easier to read.

## 21. Code and Command Examples

Fenced code blocks MUST identify the language or format where known.

Examples should be minimal, valid and clearly labelled as normative or illustrative. Secrets, real credentials and sensitive production values MUST NOT appear in examples.

## 22. File Paths and Commands

Inline paths, filenames, identifiers and short commands use code formatting.

Multi-line commands use fenced code blocks. Windows and POSIX path assumptions should be stated where ambiguity matters.

## 23. Diagrams

Architecture diagrams SHOULD use text-based, version-controllable formats where practical, such as Mermaid or PlantUML. Draw.io may be used for diagrams that require richer manual layout.

Every significant diagram SHOULD include a title, purpose, legend where needed and a source file. Diagrams must not contradict the written specification.

## 24. Images and Assets

Documentation assets SHALL be stored in approved repository asset locations. Use descriptive filenames. Avoid unnecessary binary duplication.

Images containing sensitive data, credentials, personal information or confidential production details MUST NOT be committed to public documentation.

## 25. Internal References

Internal references SHOULD use stable document IDs and, where practical, relative repository links.

Example:

```
See ARCH-004 – Executive Core.
See ADR-0003 for the decision rationale.
```

Do not rely solely on phrases such as 'the document above' or 'as discussed previously'.

## 26. External References

External references SHOULD identify title, publisher or author, publication date when known, access date when relevant and destination.

References that materially support security, legal or architectural claims should favour authoritative primary sources.

## 27. Traceability

Material implementation work SHOULD be traceable through the chain:

```
Governance / Standard
→ Architecture
→ ADR where a decision is required
→ RFC where an implementation contract is required
→ Code change
→ Test evidence
→ Release
```

Not every typo or trivial refactor requires the full chain. The process must remain proportionate.

## 28. Change History

Every controlled document SHALL maintain revision history for meaningful versions. Revision entries should describe what changed, not merely state 'updated document.'

Approved documents must not have substantive history erased.

## 29. Review Requirements

Review depth is proportional to impact.

- Editorial change – author review may be sufficient.
- Minor technical clarification – peer or architecture review as appropriate.
- Normative standard change – formal review and approval.
- Security-critical change – security-focused review.
- Major architecture change – architecture review and associated ADR/RFC consideration.

## 30. Approval Rules

Approval authority depends on document family and project maturity. Until delegated otherwise, the Founder retains final project authority while technical approval may be assigned to designated reviewers.

An author may draft a document but SHOULD NOT represent an unreviewed draft as independently validated.

## 31. Approval Evidence Representation

This section defines how a disposition already validly made under the applicable GOV-003 and GOV-010 authority is represented against an exact, immutable artifact identity. This section does not grant, define or constrain approval authority, approval classes, approval roles, review authority, decision authority, delegation, or conflict-of-interest policy. Those remain exclusively defined by GOV-003 and GOV-010.

### 31.1 Exact Artifact Identity

Approval evidence SHALL bind to the exact, immutable identity of the artifact it disposes of:

- Document ID
- Repository path
- Version
- Artifact revision identifier (a repository-native revision identifier that uniquely and immutably identifies the committed state acted upon, for example a Git commit ID)
- Content fingerprint (a cryptographic hash computed from the exact committed bytes)
- Git blob ID, where available

Approval evidence MUST NOT rely on uncommitted content. The recorded content fingerprint MUST match the exact bytes committed at the recorded artifact revision and path.

### 31.2 Disposition

Approval evidence SHALL record:

- Disposition (Approved, Rejected, Deferred, Returned for Revision, or another disposition the applicable GOV authority expressly permits)
- Disposition type (explicitly labelled as the kind of act being represented, for example: approval, rejection, deferment)
- Approver identity
- Authority citation (the specific GOV-003 or GOV-010 provision under which the approver held authority to make this disposition)
- Effective date, in the format required by §14 (Date Standard)

This subsection requires that an authority citation be recorded. It does not itself grant, assign or interpret authority; the citation MUST reference a provision of GOV-003, GOV-010, or another duly effective controlling instrument.

### 31.3 Supporting Evidence

Approval evidence SHALL record:

- Review evidence actually available, stated plainly where limited to disclosed self-review
- Independent-review status, stated explicitly
- Self-approval or conflict-of-interest disclosure, where required by the applicable GOV authority
- Known limitations
- Unresolved issues
- Rationale

Approval evidence MUST NOT represent disclosed self-review as independent review, and MUST NOT claim review evidence that does not exist.

### 31.4 Representation Mechanism

Approval evidence representation SHALL be:

- immutable once recorded
- content-non-mutating with respect to the approved artifact
- cryptographically bound to the approved artifact's exact identity
- independently auditable without reliance on external tooling
- structured so that exact artifact identity is preserved and never silently superseded

**Reference implementation for Git repositories:** the canonical implementation is a content-non-mutating commit — a commit whose tree is identical to its parent — containing the evidence described in §31.1-§31.3. This is the reference implementation of the abstract requirements above, not a separate or additional requirement.

### 31.5 Static Lifecycle Metadata

Approval evidence SHALL NOT be interpreted as modifying the lifecycle metadata contained within the approved artifact. Recording approval evidence does not require, and MUST NOT be used to justify, rewriting the approved artifact's tracked `status`, `version`, or other metadata fields. Reconciliation of tracked metadata to reflect operative approval, if performed at all, is a separate, later, distinct action, not a precondition or consequence of valid approval evidence.

### 31.6 Exclusions

This section generalises the reusable evidence-identity model already established by ADR-0011 §14. It does not restate, replace or narrow:

- Act 1-specific provisions
- Act 2-specific provisions, including ADR-0012's Act 2 evidence schema
- Bootstrap Authority State-specific provisions

Where a disposition is made under Act 1, Act 2, or otherwise concerns Bootstrap Authority State, the applicable ADR-0011 or ADR-0012 provision governs instead of this section.

## 32. Review Comments

Review comments should identify the issue, rationale and requested disposition. Authors should resolve comments as accepted, modified, rejected with rationale, or deferred with an owner and target.

## 33. AI-Assisted Documentation

AI tools MAY draft, transform, review, compare and maintain documentation. The following controls apply:

- AI output is not automatically authoritative.
- Factual claims requiring verification must be checked appropriately.
- AI must not invent approvals, reviewers, test results or implementation status.
- AI must not silently fabricate references.
- Sensitive information must not be exposed to unapproved external systems.
- High-impact normative changes require human review.
- Prompts and outputs may be retained when they materially explain provenance or decisions.

## 34. Documentation Drift

Documentation drift occurs when implementation and controlled documentation diverge. Drift must be treated as engineering debt.

Changes that alter public interfaces, security behaviour, architecture boundaries or operational procedures SHOULD include documentation updates in the same change set where practical.

## 35. Documentation Debt

Known missing, obsolete or ambiguous documentation SHOULD be recorded and prioritised according to impact. Documentation debt that creates security, operational or implementation risk may block release.

## 36. Templates

Each major controlled family SHOULD have a maintained template. Templates define required metadata and family-specific sections but must not force irrelevant boilerplate.

Template changes are versioned and do not automatically rewrite historical documents.

## 37. Classification

Initial classifications may include:

- Public – Suitable for public distribution.
- Internal – Intended for project participants.
- Confidential – Restricted because disclosure could cause material harm.
- Restricted – Highly limited access, such as secrets or sensitive security material.

Secrets must not be stored in ordinary documentation regardless of classification.

## 38. Accessibility and Readability

Documentation SHOULD use meaningful headings, descriptive link text, sufficient contrast in exported artifacts, alt text for informative images and tables that remain understandable when printed.

Do not use colour as the only carrier of meaning.

## 39. Archiving and Supersession

Superseded documents remain available for historical traceability. The old document must identify its replacement, and the replacement should identify what it supersedes where relevant.

Do not delete historical decisions merely because the project changed direction.

## 40. Documentation Quality Gates

Before approval, confirm:

- [ ] Identifier and title are correct.
- [ ] Metadata is complete.
- [ ] Status and version are accurate.
- [ ] Scope is explicit.
- [ ] Normative language is consistent.
- [ ] Internal references resolve.
- [ ] External references are credible where required.
- [ ] Open questions are visible.
- [ ] Security-sensitive information is absent or appropriately controlled.
- [ ] Revision history is updated.
- [ ] Required reviewers have completed review.
- [ ] The document is understandable without conversation history.

## 41. Exceptions

A deviation from this standard may be approved when compliance would create disproportionate cost or reduce clarity. The exception should record scope, rationale, owner, duration and any compensating controls.

Repeated exceptions should trigger review of the standard itself.

## 42. Compliance

New controlled documents created after approval of STD-001 SHALL comply with this standard. Existing documents SHOULD be migrated incrementally, prioritising active and high-impact material.

## 43. Success Criteria

This standard succeeds when SynapseOS documentation is consistent, searchable, traceable, reviewable, maintainable by humans and AI tools, and lightweight enough not to prevent iterative delivery.

## 44. Open Questions

- When will Markdown formally become the sole canonical source of truth?
- Which document families require signed approval artifacts?
- Should public documentation use automated linting from the first release?
- Which documentation checks should be mandatory in CI/CD?
- What retention period should apply to internal review records?

## 45. References

Internal:

- GOV-001 – Project Charter
- GOV-003 – Governance Model
- GOV-004 – Engineering Principles
- GOV-008 – Release Strategy
- GOV-009 – Risk Management
- GOV-010 – Decision Framework

External references will be added during formal review where adopted standards or authoritative guidance are relied upon.

## 46. Engineering Work Orders (EWO)

**Purpose.** An EWO authorizes a specific, scoped item of engineering work — implementation, reconciliation, migration, or another bounded engineering task — against governance, architecture and standards already in force.

**Authority.** An EWO is normative, but only for the engineering task it authorizes. It has no authority beyond that task's stated scope.

**Constraints.** An EWO:

- MAY authorize implementation;
- MAY define implementation scope;
- MAY define validation requirements;
- MAY define completion criteria.

An EWO MUST NOT:

- redefine Governance;
- redefine Architecture;
- redefine Standards;
- approve constitutional changes (see ARCH-001 §10, Change Control);
- approve Runtime architectural changes (see ARCH-002).

If implementation performed under an EWO reveals an apparent architectural contradiction, the EWO MUST require engineering to stop and return the issue for architectural review rather than resolve it unilaterally. This mirrors the escalation rule already stated generally in §6.

**Required metadata and structure.** An EWO SHALL identify at minimum the metadata required by §11, plus: objective, scope, constraints, definition of done, validation requirements, and reporting requirements. §15's Required Core Structure applies except where an EWO-specific template defines otherwise (§36).

**Location and identifier.** `work-orders/EWO-NNN-Short-Descriptive-Title.md`, per §7 and §8.

**Status lifecycle.** EWOs use the status values defined in §12 (typically Draft → Approved → Implemented). Approval evidence, where formally required, follows §31.

## 47. Engineering Reports (ER)

**Purpose.** An ER records engineering work completed under a specific EWO.

**Authority.** An ER is informational only. It creates no new engineering requirement, and does not itself authorize, approve, or retroactively expand the scope of the EWO it reports against.

**Required content.** An ER records, at minimum:

- objective;
- implementation summary;
- validation performed;
- deviations from the authorizing EWO, if any;
- architectural conformance;
- recommendations.

**Location and identifier.** `engineering-reports/ER-NNN-Short-Descriptive-Title.md`, per §7 and §8. An ER SHOULD identify the EWO it reports against in its metadata (§11, Related Documents).

**Relationship to architecture review.** Where an ER records an architectural conformance concern or a deviation reflecting a possible architectural contradiction, that finding is escalated for architectural review under §6 and does not resolve itself merely by being recorded.

## 48. Engineering Maintenance Orders (EMO)

**Purpose.** An EMO authorizes narrowly scoped maintenance work on a completed engineering milestone, where architecture and standards already in force determine the correct behaviour and the milestone's original EWO and ER are not to be rewritten. Maintenance includes: implementation conformance fixes; implementation defects; standards conformance updates; implementation-level refinements; and other narrowly scoped engineering maintenance of already-completed work.

**Authority.** An EMO creates no independent engineering authorization path. Its authority is entirely derived, not original: from the already-approved architecture and standards the maintenance restores conformance to, and from the specific, completed EWO it maintains. An EMO is normative only for the maintenance task it authorizes, has no authority beyond that task's stated scope, no greater authority than an EWO (§46), and MUST NOT authorize any work outside the scope of the specific EWO it maintains.

**Constraints.** An EMO:

- MAY authorize a conformance fix to a completed milestone's implementation;
- MAY define the corrective scope and validation requirements;
- MAY define completion criteria.

An EMO MUST NOT:

- authorize new engineering work;
- broaden the original milestone's scope;
- introduce future-milestone work;
- reinterpret architecture;
- introduce new architecture;
- bypass the ADR process, or replace it;
- authorize unrelated engineering;
- redefine Governance;
- redefine Architecture;
- redefine Standards;
- approve constitutional changes (see ARCH-001 §10, Change Control);
- approve Runtime architectural changes (see ARCH-002).

An EMO exists solely to maintain conformance of an already-completed engineering deliverable to architecture or standards that already govern it — never to make a new architectural decision, and never to authorize engineering work beyond that narrow purpose.

If maintenance work exposes an apparent architectural deficiency — a gap, contradiction, or genuinely undetermined question architecture does not already resolve — the EMO MUST require engineering to stop and route the issue through the existing ADR process (ADR family, §5; §6) rather than resolve it unilaterally. This mirrors the escalation rule §46 already states for an EWO, and an EMO MUST NOT resolve such an issue itself.

**Mandatory traceability.** Every EMO MUST identify, in its metadata (§11, Related Documents):

- the EWO it maintains;
- the ER it relates to, if one exists.

An EMO SHALL identify at minimum the metadata required by §11, plus the two items above, and: the identifier of any earlier EMO or EMR it supersedes, where applicable; objective; corrective scope; constraints; definition of done; validation requirements; and reporting requirements. §15's Required Core Structure applies except where an EMO-specific template defines otherwise (§36).

**Location and identifier.** `maintenance/EMO-NNN-Short-Descriptive-Title.md`, per §7 and §8.

**Status lifecycle.** EMOs use the status values defined in §12 (typically Draft → Approved → Implemented). Approval evidence, where formally required, follows §31. Review depth follows §29; a normative maintenance change follows the "Normative standard change" review tier at minimum, since it changes what a completed milestone's implementation is required to do.

**Historical integrity.** Issuing an EMO never modifies the EWO or ER it maintains. Completed EWOs and ERs remain immutable; engineering history is preserved. An EMO supplements the historical record; it never replaces it, and maintenance is always recorded through new documents, never by rewriting completed engineering records (§4, §28).

## 49. Engineering Maintenance Reports (EMR)

**Purpose.** An EMR records the implementation and verification of maintenance authorized by a specific EMO.

**Authority.** An EMR is informational only, on the same basis as an ER (§47). It creates no new engineering requirement, and does not itself authorize, approve, or retroactively expand the scope of the EMO it reports against, or of the original EWO.

**Mandatory traceability.** Every EMR MUST identify, in its metadata (§11, Related Documents):

- the EMO it implements;
- the EWO maintained;
- the ER whose historical record it supplements, if one exists.

**Required content.** An EMR records, at minimum:

- scope completed;
- files changed;
- testing performed;
- deviations from the authorizing EMO, if any;
- engineering assessment;
- relationship to the original EWO and ER, including which of their statements, if any, the recorded maintenance work supersedes as a matter of historical record — without editing the original documents' own text.

**Location and identifier.** `maintenance/EMR-NNN-Short-Descriptive-Title.md`, per §7 and §8.

**Relationship to architecture review.** Where an EMR records an architectural conformance concern or a deviation reflecting a possible architectural contradiction, that finding is escalated for architectural review under §6 and does not resolve itself merely by being recorded — on the same basis as an ER (§47).

**Historical integrity.** The original EWO and ER remain unchanged by an EMR. An EMR supplements the historical record; it never replaces it, and does not retroactively alter what the original EWO authorized or what the original ER recorded — it documents subsequent maintenance as a distinct, separate, cross-referenced record.

## Appendix A – Controlled Document Header Template

```
Document ID:
Title:
Version:
Status:
Author:
Reviewers:
Created:
Last Updated:
Classification:
Owner:
Related Documents:
Related RFCs:
Related ADRs:
```

## Appendix B – Document Family Register

| Prefix | Family | Primary Purpose | Example |
|---|---|---|---|
| GOV | Governance | Project authority and direction | GOV-010 |
| STD | Standard | Mandatory rules and conventions | STD-001 |
| ARCH | Architecture | System intent and structure | ARCH-000 |
| RFC | Request for Comments | Implementation proposal/contract | RFC-0001 |
| ADR | Architecture Decision Record | Decision rationale and consequences | ADR-0001 |
| ROAD | Roadmap | Milestones and future planning | ROAD-001 |
| RES | Research | Investigation and evidence | RES-001 |
| EWO | Engineering Work Order | Authorize scoped engineering work | EWO-NNN (none issued yet) |
| ER | Engineering Report | Record completed engineering work | ER-NNN (none issued yet) |
| EMO | Engineering Maintenance Order | Authorize scoped maintenance on a completed milestone, without rewriting its EWO/ER | EMO-NNN (none issued yet) |
| EMR | Engineering Maintenance Report | Record completed maintenance work | EMR-NNN (none issued yet) |

## Appendix C – Pre-Approval Checklist

- [ ] Correct document family and permanent ID assigned
- [ ] Filename follows standard
- [ ] Metadata complete
- [ ] Purpose and scope explicit
- [ ] Requirements are clear and proportionate
- [ ] Facts, assumptions and future intent distinguished
- [ ] Related documents linked
- [ ] Security-sensitive information reviewed
- [ ] Open questions recorded
- [ ] Revision history updated
- [ ] Review completed
- [ ] Approval recorded

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-09 | Denver Jacobs | Initial comprehensive draft |
| 0.2.0 | 2026-07-11 | Denver Jacobs | Registered Engineering Work Orders (EWO) and Engineering Reports (ER) as controlled document families. See §46, §47. |
| 0.3.0 | 2026-07-12 | Denver Jacobs | Registered Engineering Maintenance Orders (EMO) and Engineering Maintenance Reports (EMR) as controlled document families, supplementing EWO/ER to allow narrowly scoped post-completion conformance maintenance without rewriting completed engineering history. See §48, §49. |
| 0.3.1 | 2026-07-12 | Denver Jacobs | Final governance review refinement of the 0.3.0 EMO/EMR amendment: explicit authority-inheritance statement, mandatory (MUST-level) traceability lists for both EMO and EMR, tightened scope-restriction wording, explicit supplement-not-replace historical-integrity statements, and explicit independence of EMO/EMR numbering from EWO/ER numbers and informal milestone labels. See §6, §7, §48, §49. |

## Approval

```
Author: Denver Jacobs ____________________  Date: __________

Technical Review: _______________________  Date: __________

Approved By: ___________________________  Date: __________
```
