---
document_id: STD-001
title: Documentation Standards
volume: Volume II – Standards
version: 0.1.0
status: Draft
author: Denver Jacobs
reviewers: TBD
created: 2026-07-09
last_updated: 2026-07-09
classification: Public
owner: SynapseOS Project
approval_authority: Founder / Designated Architecture Authority
related_documents: GOV-001 through GOV-010
---

# STD-001 – Documentation Standards

**SynapseOS Engineering Manual — Volume II – Standards**
**Version 0.1.0 (Draft)**
**Controlled Engineering Standard**

> This Markdown document is the canonical source-format conversion of `STD-001_Documentation_Standards_v0.1.docx`. The original Word document is retained as the source artifact pending a future controlled archival decision.

## Document Control

| Field | Value |
|---|---|
| Document ID | STD-001 |
| Title | Documentation Standards |
| Volume | Volume II – Standards |
| Version | 0.1.0 |
| Status | Draft |
| Author | Denver Jacobs |
| Reviewers | TBD |
| Created | 2026-07-09 |
| Last Updated | 2026-07-09 |
| Classification | Public |
| Owner | SynapseOS Project |
| Approval Authority | Founder / Designated Architecture Authority |
| Related Documents | GOV-001 through GOV-010 |

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-09 | Denver Jacobs | Initial controlled draft |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs | Drafted | 2026-07-09 |
| Technical Review | TBD | Pending | |
| Approval Authority | TBD | Pending | |

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

Repository README files and local guides may remain unnumbered where they are navigational rather than authoritative.

## 6. Document Hierarchy and Precedence

Where documents conflict, the issue must be resolved rather than ignored. As a default governance rule, approved governance constraints take precedence over subordinate standards; approved standards constrain architecture and implementation; architecture defines system intent; ADRs record specific decisions; RFCs define approved implementation contracts.

A later approved document does not automatically override an earlier higher-authority document unless the change is explicit and traceable.

## 7. Identifier Standard

Controlled identifiers use an uppercase family prefix, hyphen and zero-padded sequence number.

Examples:

```
GOV-001
STD-001
ARCH-000
RFC-0001
ADR-0001
```

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

The repository root `README.md` provides navigation and onboarding.

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

Status transitions must be traceable.

## 13. Versioning Standard

Controlled documents use MAJOR.MINOR.PATCH versioning.

- PATCH: editorial corrections that do not change normative meaning.
- MINOR: backward-compatible additions, clarifications or new sections.
- MAJOR: material changes to obligations, architecture intent, governance meaning or compatibility.

Draft documents may begin at 0.1.0. Initial formal approval normally establishes 1.0.0.

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
10. Approval information where formal approval applies

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

## 31. Review Comments

Review comments should identify the issue, rationale and requested disposition. Authors should resolve comments as accepted, modified, rejected with rationale, or deferred with an owner and target.

## 32. AI-Assisted Documentation

AI tools MAY draft, transform, review, compare and maintain documentation. The following controls apply:

- AI output is not automatically authoritative.
- Factual claims requiring verification must be checked appropriately.
- AI must not invent approvals, reviewers, test results or implementation status.
- AI must not silently fabricate references.
- Sensitive information must not be exposed to unapproved external systems.
- High-impact normative changes require human review.
- Prompts and outputs may be retained when they materially explain provenance or decisions.

## 33. Documentation Drift

Documentation drift occurs when implementation and controlled documentation diverge. Drift must be treated as engineering debt.

Changes that alter public interfaces, security behaviour, architecture boundaries or operational procedures SHOULD include documentation updates in the same change set where practical.

## 34. Documentation Debt

Known missing, obsolete or ambiguous documentation SHOULD be recorded and prioritised according to impact. Documentation debt that creates security, operational or implementation risk may block release.

## 35. Templates

Each major controlled family SHOULD have a maintained template. Templates define required metadata and family-specific sections but must not force irrelevant boilerplate.

Template changes are versioned and do not automatically rewrite historical documents.

## 36. Classification

Initial classifications may include:

- Public – Suitable for public distribution.
- Internal – Intended for project participants.
- Confidential – Restricted because disclosure could cause material harm.
- Restricted – Highly limited access, such as secrets or sensitive security material.

Secrets must not be stored in ordinary documentation regardless of classification.

## 37. Accessibility and Readability

Documentation SHOULD use meaningful headings, descriptive link text, sufficient contrast in exported artifacts, alt text for informative images and tables that remain understandable when printed.

Do not use colour as the only carrier of meaning.

## 38. Archiving and Supersession

Superseded documents remain available for historical traceability. The old document must identify its replacement, and the replacement should identify what it supersedes where relevant.

Do not delete historical decisions merely because the project changed direction.

## 39. Documentation Quality Gates

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

## 40. Exceptions

A deviation from this standard may be approved when compliance would create disproportionate cost or reduce clarity. The exception should record scope, rationale, owner, duration and any compensating controls.

Repeated exceptions should trigger review of the standard itself.

## 41. Compliance

New controlled documents created after approval of STD-001 SHALL comply with this standard. Existing documents SHOULD be migrated incrementally, prioritising active and high-impact material.

## 42. Success Criteria

This standard succeeds when SynapseOS documentation is consistent, searchable, traceable, reviewable, maintainable by humans and AI tools, and lightweight enough not to prevent iterative delivery.

## 43. Open Questions

- When will Markdown formally become the sole canonical source of truth?
- Which document families require signed approval artifacts?
- Should public documentation use automated linting from the first release?
- Which documentation checks should be mandatory in CI/CD?
- What retention period should apply to internal review records?

## 44. References

Internal:

- GOV-001 – Project Charter
- GOV-003 – Governance Model
- GOV-004 – Engineering Principles
- GOV-008 – Release Strategy
- GOV-009 – Risk Management
- GOV-010 – Decision Framework

External references will be added during formal review where adopted standards or authoritative guidance are relied upon.

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

## Approval

```
Author: Denver Jacobs ____________________  Date: __________

Technical Review: _______________________  Date: __________

Approved By: ___________________________  Date: __________
```
