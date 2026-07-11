---
document_id: STD-004
title: Repository Standards
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-09
last_updated: 2026-07-11
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-004 (Approved)
    - GOV-010 (Approved, Act 2)
    - GOV-006 (Draft — legacy, unconverted)
    - GOV-008 (Draft — legacy, unconverted)
    - GOV-009 (Draft — legacy, unconverted)
  standards:
    - STD-001 (Approved)
    - STD-002 (Draft — legacy, not yet reconciled)
    - STD-003 (Draft — legacy, not yet reconciled)
  architecture:
    - ARCH-001 (Draft — constitutional foundation)
    - ARCH-002 (Draft — Runtime architecture)
  rfcs: None
  adrs:
    - ADR-0013 (Draft)
    - ADR-0014 (Draft — governs this reconciliation)
  roadmap: None
  research: None
  operational: None
  source_artifacts:
    - standards/STD-004_Repository_Standards_v0.1.docx (legacy source; disposition to be determined by a separate controlled task)
supersedes: None
superseded_by: None
ai_assistance: Conversion and drafting
---

# STD-004 — Repository Standards

*Filename pattern: `STD-004-Repository-Standards.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Conversion and drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

> **Status notice:** This document is **Draft**. This Markdown document is the candidate canonical controlled source-format conversion of `standards/STD-004_Repository_Standards_v0.1.docx`, combined with the minimum reconciliation against the constitutional architecture authorised by ADR-0014. The original Word document is retained unchanged as a legacy source artifact for provenance and conversion verification pending a separate controlled archival decision. It is not co-canonical and must not receive future substantive edits. This Markdown document remains a Draft and is not approved or operative merely because it exists, is staged, committed, or pushed.

## 1. Purpose

Define mandatory structural, governance, security and maintainability requirements for SynapseOS Git repositories.

## 2. Scope

Applies to documentation, the Runtime, SDK, plugins, modules, infrastructure, research prototypes promoted to maintained status, and other official SynapseOS repositories. As of this reconciliation, two official repositories exist: `synapse-docs` (documentation, governance, and architecture) and `synapse-runtime` (the SynapseOS Runtime, per ARCH-002).

## 3. Objectives

Repositories must be discoverable, consistently structured, secure by default, automation-ready, independently understandable and maintainable by future human and AI contributors.

## 4. Repository Philosophy

Create a repository only when a meaningful ownership, release, security or lifecycle boundary exists. Avoid both an uncontrolled monorepo and needless repository fragmentation. `synapse-docs` and `synapse-runtime` were established as separate repositories on exactly this basis — see §65–§66.

## 5. Repository Authority

Every official repository requires a named purpose, owner, status and relationship to the wider SynapseOS architecture.

## 6. Repository Classification

Core – critical platform code. The SynapseOS Runtime (`synapse-runtime`), realizing ARCH-002, is the current instance of this classification. Documentation – controlled specifications and manuals. SDK – developer-facing libraries/tooling. Plugin – independently deployable extension. Infrastructure – deployment and environment automation. Research – experimental work without production guarantees. Archive – retained but no longer actively maintained.

## 7. Naming Convention

Repository names SHALL be lowercase and hyphen-separated unless platform constraints require otherwise. Names should be concise and descriptive.

Examples:

```
synapse-docs
synapse-runtime
synapse-sdk
synapse-cli
```

## 8. Namespace Strategy

Official repositories should use a consistent `synapse-` prefix where it improves discoverability. Exceptions may apply to umbrella or organisation-level repositories.

## 9. Repository Creation Gate

Before creating a repository, define purpose, owner, classification, expected consumers, release model, security sensitivity, dependencies and reason it cannot reasonably live in an existing repository.

## 10. Minimum Root Files

Maintained repositories SHOULD include README.md, LICENSE or licence reference, .gitignore, contribution guidance where external contribution is allowed, security reporting guidance where relevant, and automation configuration appropriate to the project.

## 11. README Requirements

The root README should explain what the repository is, status, intended users, prerequisites, quick start, architecture relationship, development workflow, testing, documentation links, support expectations and licence.

## 12. Repository Status

Repositories SHOULD declare one of: Experimental, Active Development, Stable, Maintenance, Deprecated, Archived. Status must not be misleading. This is a repository-maturity vocabulary distinct from, and not to be confused with, STD-001 §12's document status lifecycle (Draft, Review, Approved, and related states), which governs individual controlled documents rather than whole repositories.

## 13. Ownership

Each repository requires an accountable owner or maintainer group. Ownership includes review, dependency health, security response, release quality and archival decisions.

## 14. CODEOWNERS

Use CODEOWNERS or equivalent when supported and useful. Critical areas may require specialist review.

## 15. Standard Directory Principles

Directories should reflect domain and responsibility. Avoid generic dumping grounds such as misc, stuff, temp or utils unless their boundaries are explicit.

## 16. Documentation Repository Structure

The `synapse-docs` repository SHOULD organise controlled material into `governance/`, `standards/`, `architecture/`, `rfcs/`, `adrs/`, `roadmap/`, `research/`, `glossary/`, `diagrams/`, `templates/`, `meeting-notes/`, `decisions/`, `assets/` and `scripts/` as applicable. This structure is already realised in the current repository.

## 17. Code Repository Structure

A code repository SHOULD clearly separate source, tests, documentation, configuration, scripts and build output. Exact structure is language-specific and should be documented. For the Runtime specifically, source separation additionally follows ARCH-002 §6's trusted-core/replaceable-service component boundary — see §18.

## 18. Source Directory

Production source code should live in a predictable location such as `src/`, `crates/`, `packages/` or a language-standard equivalent. Do not mix generated output with hand-maintained source without clear separation. `synapse-runtime` realises this as a Cargo workspace, with each trusted-core and replaceable-service crate (ARCH-002 §6) maintaining its own `src/` directory — a language-standard equivalent for a Rust workspace.

## 19. Test Directory

Tests should be discoverable and follow ecosystem conventions. Integration, end-to-end and security tests may use distinct directories where that improves execution and ownership. `synapse-runtime` follows Rust's own convention: per-crate unit tests inline in `src/lib.rs`, with a top-level `tests/` directory reserved for cross-crate integration tests.

## 20. Documentation Directory

Repository-specific documentation belongs in `docs/` when it is not part of the central controlled architecture corpus. Avoid duplicating authoritative documents — implementation repositories should cite `synapse-docs`'s controlled documents rather than restate them.

## 21. Scripts Directory

Automation scripts should be stored in `scripts/` or an ecosystem-standard equivalent, documented, reviewed and written defensively.

## 22. Configuration

Configuration files must be intentional and documented. Environment-specific secrets must not be committed.

## 23. Example Configuration

Provide safe example configuration where useful, using placeholders rather than real secrets. Example files must not be accidentally loadable as production credentials.

## 24. Ignore Rules

Each repository SHALL maintain appropriate ignore rules for build output, caches, local environments, editor state and sensitive local files.

## 25. Binary Files

Avoid committing large binaries. Where necessary, document purpose and consider artifact storage or Git LFS.

## 26. Generated Content

Generated content must be clearly distinguishable from source. Document generator, command and whether generated output is committed.

## 27. Vendored Code

Vendoring third-party source requires explicit justification, licence review, provenance and update strategy.

## 28. Dependency Manifests

Use ecosystem-standard dependency manifests and lockfiles. Dependencies must be reproducible and reviewed under STD-002 (Draft, not yet reconciled).

## 29. Licence

Every public repository must have an explicit licence decision. Absence of a licence is not an open-source strategy.

## 30. Third-Party Notices

Where licences require attribution or notices, maintain them in a predictable location and include them in relevant distributions.

## 31. Security Policy

Repositories exposed to external users SHOULD provide a SECURITY.md or central security reporting link. Do not encourage public disclosure of unpatched vulnerabilities.

## 32. Secrets Protection

Enable secret scanning or equivalent controls where available. Pre-commit checks may supplement but do not replace credential hygiene.

## 33. Branch Protection

Protected branches follow STD-003 (Draft, not yet reconciled). Main should prevent accidental deletion, uncontrolled force pushes and unreviewed integration as project maturity permits.

## 34. Access Control

Repository permissions follow least privilege. Administrative rights are limited. Machine identities and automation tokens require explicit ownership.

## 35. Automation Identity

CI/CD identities must have only required permissions. Avoid broad persistent tokens when short-lived credentials are available.

## 36. CI Directory and Files

Automation configuration should be discoverable, version-controlled and reviewed like code. Workflow changes can alter security boundaries and require scrutiny.

## 37. Required Checks

Repositories define checks appropriate to their risk: formatting, linting, build, unit tests, integration tests, security scans, documentation validation and packaging.

## 38. Reproducible Builds

Critical repositories SHOULD move toward reproducible or at least repeatable builds with pinned toolchains and documented prerequisites.

## 39. Toolchain Versioning

Pin or declare supported compiler, runtime and package-manager versions where drift could break builds.

## 40. Local Development

Provide a documented path for a new contributor to build and test locally without relying on private conversation history.

## 41. Environment Bootstrap

Bootstrap scripts MAY automate setup but must be inspectable and should avoid destructive machine-wide changes without warning.

## 42. Container Development

Development containers MAY be used where they improve reproducibility. They are not mandatory and must not conceal undocumented production assumptions.

## 43. Repository Metadata

Where hosting supports it, configure description, topics, homepage/documentation link and visibility deliberately.

## 44. Issue Tracking

Repositories should identify where defects and feature requests are tracked. Avoid fragmented duplicate trackers without clear purpose.

## 45. Issue Templates

Use templates for bugs, features and security-sensitive reports when volume justifies them. Templates should improve evidence, not create bureaucracy.

## 46. Pull Request Template

Maintained code repositories SHOULD use a PR template aligned with STD-003 (Draft, not yet reconciled).

## 47. Contribution Guide

Public contribution repositories SHOULD explain setup, standards, workflow, tests, review expectations and governance.

## 48. Code of Conduct

Public community repositories SHOULD adopt an appropriate code of conduct before broad community growth.

## 49. Architecture References

Core repositories SHOULD link to applicable ARCH, RFC and ADR documents rather than duplicating architecture prose.

## 50. API Documentation

Repositories exposing public APIs must document supported contracts and compatibility expectations.

## 51. Changelog

User-facing repositories SHOULD maintain release notes or a changelog appropriate to their release model.

## 52. Release Artifacts

Build artifacts, containers and packages must be produced through controlled processes. Do not commit release binaries to source repositories without explicit reason.

## 53. Artifact Retention

Define retention for CI artifacts according to value, cost, security and reproducibility needs.

## 54. Package Publishing

Published packages require controlled naming, ownership, provenance and recovery from compromised credentials.

## 55. Container Images

Container image repositories must use explicit tags and SHOULD avoid relying solely on mutable latest tags for production.

## 56. Software Bill of Materials

Critical releases SHOULD move toward generating an SBOM to improve dependency and incident visibility.

## 57. Provenance

The project SHOULD evaluate build provenance and artifact signing as release maturity increases.

## 58. Backup and Continuity

Critical repositories must not depend on one local computer. Remote hosting, backups or mirrors should protect project continuity.

## 59. Disaster Recovery

Repository recovery considerations include deleted branches, compromised accounts, organisation lockout, corrupted local clones and unavailable hosting.

## 60. Archiving

Archive repositories deliberately. Mark status, disable unnecessary automation, preserve history and point users to replacements.

## 61. Deprecation

Deprecated repositories should identify replacement, migration path and expected support status.

## 62. Repository Transfer

Transfers between organisations or owners require review of permissions, secrets, webhooks, package ownership, CI identities and public links.

## 63. Forks

Official forks must be clearly labelled. Do not allow stale forks to appear authoritative.

## 64. Mirrors

Mirrors may improve continuity but must define which location is authoritative and how synchronisation occurs.

## 65. Monorepo Decision

A monorepo is justified when components share lifecycle, tooling and atomic change needs. It is not chosen merely for convenience.

## 66. Multi-Repo Decision

Separate repositories are justified by independent lifecycle, security boundary, ownership, distribution or contribution model. `synapse-docs` (documentation and governance) and `synapse-runtime` (implementation) were established as separate repositories on this basis: independent lifecycle and independent tooling, consistent with §4, §65, and §66 as applied in practice.

## 67. Repository Splitting

Splitting an existing repository requires migration planning for history, issues, CI, packages, references and contributor workflow.

## 68. Repository Merging

Merging repositories requires conflict, history, ownership, build and release analysis.

## 69. AI Agent Access

Claude Code or other agents receive repository permissions proportionate to task. Agents must not be given organisation-wide administrative authority merely for convenience. This is a repository-specific application of the broader AI-authority constraints GOV-010 §15 and `.ai/ARCHITECTURAL-CONTEXT.md`'s AI Working Rule already establish; where the two appear to differ in emphasis, GOV-010 §15 is the controlling authority.

## 70. AI Agent Workspace

AI agents should operate in explicit project directories and verify repository identity before modifying files or running commands.

## 71. AI-Generated Repository Changes

AI-generated scaffolding, workflows and configuration are subject to the same review as human changes. Generated CI files require special security attention.

## 72. Destructive Operations

Agents must not delete repositories, rewrite shared history, remove branches broadly, rotate credentials or change visibility without explicit authorisation.

## 73. Public vs Private Visibility

Visibility is a deliberate governance decision. Before making a repository public, review secrets, history, personal data, internal URLs, licences and security-sensitive material.

## 74. Repository Templates

Organisation-level repository templates MAY standardise baseline files, security configuration and automation.

## 75. Standard Baseline

A maintained baseline may define README, licence, security, contribution, ignore rules, CI and ownership configuration.

## 76. Compliance Review

Critical repositories SHOULD periodically verify status, ownership, branch protection, dependencies, secrets controls, CI health, documentation and release posture. This reconciliation's own repository review of `synapse-runtime` (produced separately, not as part of this document) is the first instance of this practice.

## 77. Exceptions

Exceptions require rationale, owner, scope and review trigger proportional to risk.

## 78. Success Criteria

The standard succeeds when repositories are consistent enough to navigate quickly, secure enough to resist common mistakes, and flexible enough to support different SynapseOS components.

## 79. Open Questions

- Will SynapseOS use one GitHub organisation or multiple organisations?
- Which repositories are required for Phase 1? (`synapse-docs` and `synapse-runtime` now exist; whether further repositories are required remains open.)
- Which baseline files become mandatory immediately?
- Should repository creation require a lightweight RFC? (`synapse-runtime` was created without one; this question remains open rather than resolved by that precedent.)
- What backup or mirror strategy is affordable during the bootstrap phase?

## 80. References

Internal:

- GOV-006 – Technical Strategy (Draft, legacy)
- GOV-008 – Release Strategy (Draft, legacy)
- GOV-009 – Risk Management (Draft, legacy)
- STD-001 – Documentation Standards (Approved)
- STD-002 – Coding Standards (Draft, not yet reconciled)
- STD-003 – Git Workflow (Draft, not yet reconciled)
- ARCH-001 – Constitutional Architecture (Draft)
- ARCH-002 – Runtime Architecture (Draft)
- ADR-0013 – Architectural Evolution of SynapseOS (Draft)
- ADR-0014 – Engineering Standards Corpus Reconciliation (Draft)

## Appendix A – Minimum Repository Checklist

- [ ] Purpose defined
- [ ] Owner defined
- [ ] Classification/status defined
- [ ] README present
- [ ] Licence decision recorded
- [ ] .gitignore appropriate
- [ ] Security reporting path defined where relevant
- [ ] Branch protection considered
- [ ] CI checks defined
- [ ] Secrets reviewed
- [ ] Documentation links present
- [ ] Local build/test path documented
- [ ] Dependencies reproducible
- [ ] Backup/remote continuity confirmed

## Appendix B – Proposed synapse-docs Structure

```
synapse-docs/
├── governance/
├── standards/
├── architecture/
├── rfcs/
├── adrs/
├── roadmap/
├── research/
├── glossary/
├── diagrams/
├── templates/
├── meeting-notes/
├── decisions/
├── assets/
├── scripts/
└── README.md
```

This structure is already realised in the current repository.

## Appendix C – New Repository Proposal Template

```
Repository Name:
Purpose:
Owner:
Classification:
Status:
Consumers:
Why a separate repository is required:
Security sensitivity:
Primary language/toolchain:
Release model:
Dependencies:
Expected CI checks:
Licence:
Public/Private:
Backup/continuity plan:
Related ARCH/RFC/ADR:
```

## Appendix D – Public Release Repository Review

- [ ] Full Git history checked for secrets
- [ ] Personal/confidential data removed
- [ ] Licence confirmed
- [ ] Third-party notices confirmed
- [ ] README accurate
- [ ] Security reporting route available
- [ ] Internal endpoints removed
- [ ] CI secrets protected from untrusted forks
- [ ] Repository metadata accurate
- [ ] Maintainer ownership confirmed

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-09 | Denver Jacobs | Initial comprehensive draft (source: `standards/STD-004_Repository_Standards_v0.1.docx`). |
| 0.1.0 | 2026-07-11 | Denver Jacobs | Reconciled with the constitutional architecture under ADR-0014, converting the legacy `.docx` to canonical Markdown in the same pass. Changes: corrected the §7 naming example from the never-used "synapse-core" to the actual repository name "synapse-runtime"; §6 and §17–§18 now cite ARCH-002's trusted-core/replaceable-service component model and the Runtime's actual Cargo-workspace realisation of it; §16 confirmed accurate against the current repository structure rather than assumed; §65–§66 record the `synapse-docs`/`synapse-runtime` separate-repository decision as an applied instance of this document's own principle; §69 cross-references GOV-010 §15 and the AI Working Rule as the controlling authority for AI repository access, per ADR-0014 §8's overlap-resolution rule, without removing this document's own repository-specific wording; §80 References and the frontmatter now cite ARCH-001, ARCH-002, ADR-0013, and ADR-0014, previously absent entirely; §12 clarifies that repository-status vocabulary is distinct from STD-001 §12's document-status vocabulary. No section was rewritten for style alone; no obligation was removed, weakened, or newly imposed. This is presented as a single proposed Draft revision. No approval act has occurred. |

## Approval Status

### Document State

| Field | Value |
|-------|-------|
| Lifecycle status | Draft |
| Normal-governance disposition | Not yet approved |
| Disposition date | Not yet assigned |
| Approving actor | Not yet assigned |

### Approval Evidence (per STD-001 §31)

| Field | Value |
|-------|-------|
| Document ID | STD-004 |
| Repository path | standards/STD-004-Repository-Standards.md |
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

No field in this section may be populated until the corresponding act has genuinely occurred, evidenced per STD-001 §31. This table does not, and must not be read to, claim that any approval of STD-004 has occurred.
