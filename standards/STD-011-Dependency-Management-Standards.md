---
document_id: STD-011
title: Dependency Management Standards
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
    - GOV-004 (Approved)
    - GOV-010 (Approved, Act 2)
    - GOV-006 (Draft — legacy, unconverted)
    - GOV-008 (Draft — legacy, unconverted)
    - GOV-009 (Draft — legacy, unconverted)
  standards:
    - STD-001 (Approved)
    - STD-002 (Draft — reconciled)
    - STD-004 (Draft — reconciled)
    - STD-007 (Draft — legacy, not yet reconciled)
    - STD-008 (Draft — legacy, not yet reconciled)
    - STD-010 (Draft — legacy, not yet reconciled)
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
    - standards/STD-011_Dependency_Management_Standards_v0.1.docx (legacy source; disposition to be determined by a separate controlled task)
supersedes: None
superseded_by: None
ai_assistance: Conversion and drafting
---

# STD-011 — Dependency Management Standards

*Filename pattern: `STD-011-Dependency-Management-Standards.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Conversion and drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

> **Status notice:** This document is **Draft**. This Markdown document is the candidate canonical controlled source-format conversion of `standards/STD-011_Dependency_Management_Standards_v0.1.docx`, combined with the minimum reconciliation against the constitutional architecture authorised by ADR-0014. The original Word document is retained unchanged as a legacy source artifact for provenance and conversion verification pending a separate controlled archival decision. It is not co-canonical and must not receive future substantive edits. This Markdown document remains a Draft and is not approved or operative merely because it exists, is staged, committed, or pushed.

## 1. Purpose

Define how SynapseOS selects, approves, records, updates, verifies, secures, replaces and retires third-party dependencies across software, AI models, plugins, containers, build systems and infrastructure.

## 2. Scope

Applies to direct and transitive software packages, runtime libraries, frameworks, SDKs, CLI tools, CI actions, container images, operating-system packages, model artifacts, datasets, plugins, external APIs and hosted services where dependency risk affects SynapseOS — including `synapse-runtime`'s own Cargo dependency graph. STD-002 (Coding Standards, reconciled) and STD-004 (Repository Standards, reconciled) both defer their own dependency-management content to this document; nothing in this reconciliation duplicates them.

## 3. Objective

Reduce supply-chain, security, licensing, availability, compatibility and lock-in risks while allowing practical reuse of maintained external technology.

## 4. Dependency Philosophy

Every dependency creates capability and obligation. A dependency is not free merely because its purchase price is zero.

*Terminology note:* "capability" here is used in the ordinary sense (what a dependency enables you to do), distinct from the constitutional **Capability** (ARCH-001 §5.2) — see STD-002 §13 for the fuller disambiguation, not repeated here.

## 5. Core Principles

- Minimise unnecessary dependencies. `synapse-runtime`'s current dependency graph is the applied extreme of this principle: every crate depends on exactly `synapse-common` and nothing else — zero external dependencies (verified directly against the workspace's own `cargo tree` output).
- Prefer maintained and well-understood components.
- Record provenance.
- Pin or constrain versions appropriately.
- Use lockfiles where supported.
- Review licences.
- Scan for known vulnerabilities.
- Separate critical from convenience dependencies.
- Plan replacement for strategic dependencies.
- Treat AI models and hosted providers as dependencies.

## 6. Dependency Categories

Runtime, development, build, test, optional, plugin, infrastructure, container, operating-system, AI provider, model artifact, dataset, hosted service and transitive dependency.

*Terminology note:* "Runtime" as a dependency category (a dependency needed at execution time, as opposed to development-only) is distinct from **the SynapseOS Runtime** (capitalised, `synapse-runtime`, ARCH-002's subject matter). Both terms appear throughout this document; context disambiguates them, and no dependency category is itself part of the constitutional architecture.

## 7. Ownership

Material dependencies require an accountable owning component/team responsible for updates, compatibility and risk response.

## 8. Dependency Inventory

Maintained repositories SHOULD have machine-readable manifests and lockfiles. Critical systems SHOULD maintain broader dependency inventory or SBOM. `synapse-runtime` maintains its inventory as a Cargo workspace manifest with a committed `Cargo.lock`, per STD-004 §38–§39.

## 9. New Dependency Decision

Before adding a material dependency, assess need, alternatives, maintenance, security, licence, size, performance, portability, ecosystem maturity and replacement cost. For a dependency proposed inside a trusted-core crate (ARCH-002 §5–§6), this assessment defaults to Tier 1 review (§113, Appendix E) regardless of the dependency's own apparent size or maturity, because ARCH-002 §5 keeps the trusted core "conceptually minimal" by explicit, per-mechanism justification against the four constitutional guarantees, not by convenience, and §22 lists those guarantees as mandatory at all times.

## 10. Build vs Buy vs Reuse

Do not implement complex security or protocol functionality merely to avoid a healthy dependency. Conversely, do not add a large framework for trivial convenience.

## 11. Necessity Test

Ask whether existing standard library, approved dependency or small internal implementation can meet the requirement safely.

## 12. Maintenance Health

Consider release activity, issue responsiveness, maintainer continuity, adoption, security history and project governance.

## 13. Popularity

Popularity is a signal, not proof of safety or suitability.

## 14. Abandonment Risk

Dependencies with unclear maintenance require explicit risk consideration and possible replacement plan.

## 15. Bus Factor

Critical dependencies maintained by very few individuals may require additional continuity analysis.

## 16. Security History

Review known vulnerabilities and response quality where material. A history of vulnerabilities is not automatically disqualifying; poor response may be more significant.

## 17. Provenance

Obtain dependencies from official or trusted registries, repositories and publishers. Avoid random binary download sites.

## 18. Package Name Confusion

Guard against typosquatting, dependency confusion and malicious lookalike packages.

## 19. Publisher Verification

Verify publisher/organisation identity for critical packages where ecosystem support exists.

## 20. Direct Dependencies

Direct dependencies are explicitly selected by SynapseOS and require stronger intentional review.

## 21. Transitive Dependencies

Transitive dependencies also create risk. Use tooling to inspect and monitor them.

## 22. Version Constraints

Use ecosystem-appropriate constraints that balance reproducibility and security updates.

## 23. Lockfiles

Commit lockfiles for applications and other contexts where ecosystem guidance supports reproducible resolution. `synapse-runtime` commits `Cargo.lock`, consistent with STD-004 §38.

## 24. Floating Versions

Avoid uncontrolled floating versions in production builds.

## 25. Exact Pinning

Exact pins improve reproducibility but may delay security updates. Use with an active update process.

## 26. Semantic Versioning

Do not assume all publishers follow semantic versioning correctly. Verify behaviour and test updates.

## 27. Major Updates

Major updates require compatibility analysis, migration planning and testing.

## 28. Minor and Patch Updates

Minor/patch updates still require CI and review. Version labels do not prove low risk.

## 29. Automated Updates

Dependency bots MAY propose updates. Automated proposal does not equal automated trust.

## 30. Update Grouping

Group low-risk updates only when failures remain diagnosable. Security updates may require separate expedited handling.

## 31. Update Cadence

Repositories SHOULD establish regular dependency review cadence proportional to exposure.

## 32. Critical Vulnerabilities

Critical exploitable vulnerabilities require prompt triage, mitigation, upgrade, patch, isolation or explicit risk acceptance.

## 33. Vulnerability Databases

Use appropriate ecosystem advisories and scanning tools. Scanner output requires human/technical triage.

## 34. False Positives

Document justified suppressions with scope and review trigger.

## 35. Unreachable Vulnerabilities

A vulnerable code path may be unreachable, but this requires evidence and does not erase future exposure risk.

## 36. End-of-Life Dependencies

Unsupported dependencies require replacement, isolation or explicit time-bounded exception.

## 37. Framework Lifecycle

Track support windows for major frameworks and runtimes.

## 38. Runtime Versions

Declare supported runtime/compiler versions and update before end-of-support where practical. `synapse-runtime` does not yet pin an explicit Rust toolchain version — STD-004's compliance review (§38–§39) already recorded this as a non-blocking gap; this section does not restate that finding, only cross-references it.

## 39. Operating-System Packages

Base OS packages are dependencies and require supported sources and patch strategy.

## 40. Container Base Images

Use maintained minimal base images appropriate to need. Pin by digest where high assurance justifies it, while retaining update process.

## 41. Container Tags

Do not rely solely on mutable latest tags for controlled releases.

## 42. CI Actions

Third-party CI actions execute privileged workflow code and require dependency review. Pinning to immutable references should be considered.

## 43. Build Tools

Compilers, package managers, code generators and build plugins are supply-chain dependencies.

## 44. Generated Code

Record generator and version where generated code is committed or distributed.

## 45. SDK Dependencies

External SDKs should be isolated behind adapters when they represent replaceable providers or services. This is precisely the Provider Adapter pattern ARCH-002 already establishes: a provider adapter is modelled as an ordinary, capability-scoped Actor, never granted ambient access to the underlying SDK or service — see STD-002 §45 for the coding-level statement of the same pattern.

## 46. External APIs

Hosted APIs are dependencies. Assess authentication, quotas, pricing, availability, data handling, versioning and exit strategy.

## 47. AI Providers

AI providers are strategic dependencies. Track model availability, pricing, retention, policy, regional access, rate limits and failure modes.

## 48. Provider Neutrality

Core SynapseOS architecture SHOULD avoid unnecessary coupling to one AI provider, consistent with GOV-004 Principle 4 ("Model Agnostic").

## 49. Model Names

Do not build durable architecture around a marketing model name. Treat model identifiers as provider configuration.

## 50. Model Deprecation

Prepare for provider model retirement and forced migration.

## 51. Model Behaviour Drift

A provider may change model behaviour without source changes. Critical evaluations must be rerun after material model changes.

## 52. Local Model Dependencies

Local models introduce weights, runtime, GPU/CPU, quantisation, licence and provenance dependencies.

## 53. Model Artifact Provenance

Obtain model files from trusted sources and verify hashes/signatures where available and practical.

## 54. Model Licences

Review model licences separately from runtime code licences.

## 55. Datasets

Datasets are dependencies when used for training, evaluation or retrieval. Track provenance, licence, consent/usage constraints and version.

## 56. Embedding Models

Embedding model changes may invalidate retrieval assumptions and stored vector compatibility.

## 57. Vector Databases

Treat vector stores as replaceable infrastructure where possible. Document export and migration strategy.

## 58. Plugins

Plugins are runtime dependencies with potentially high privilege. Apply STD-007 security controls (not yet reconciled). ARCH-002 §19 already establishes the structural constraint STD-007's controls will operationalise: no extension bypasses capability enforcement, message integrity, actor isolation, execution ownership, or lifecycle controls.

## 59. Plugin Registries

Public plugin ecosystems require publisher identity, provenance, update and revocation mechanisms.

## 60. Optional Dependencies

Optional dependencies must fail clearly when unavailable and should not become hidden mandatory requirements.

## 61. Feature Dependencies

Features requiring external services must document degraded behaviour when unavailable.

## 62. Licence Review

Every material dependency requires a licence compatible with intended distribution and use.

## 63. Copyleft

Strong or network copyleft obligations require explicit review before adoption into distributed or hosted components.

## 64. Attribution

Preserve required notices and attribution.

## 65. Unknown Licence

A dependency with no clear licence is not assumed free to use.

## 66. Dual Licensing

Understand which licence option applies and record the decision.

## 67. Commercial Terms

Free tiers may change. Hosted-service dependencies require pricing and continuity consideration.

## 68. Data Residency

External services may create data-location and regulatory considerations.

## 69. Privacy

Assess what data is transmitted to third-party dependencies.

## 70. Secrets

Dependency configuration must not expose credentials in source, logs or error messages, consistent with STD-002 §17 and STD-004 §32.

## 71. Network Egress

Critical deployments SHOULD know which dependencies require outbound network access.

## 72. Offline Mode

Where offline/local operation is a goal, identify dependencies that prevent it.

## 73. Reproducible Builds

Use manifests, lockfiles, pinned toolchains and controlled sources to improve repeatability.

## 74. Hermetic Builds

Hermetic builds MAY be introduced where assurance requirements justify complexity.

## 75. Dependency Caching

Caches improve reliability and speed but must not become unverified sources of stale or compromised artifacts.

## 76. Private Mirrors

Private registries/mirrors MAY improve continuity and control as maturity grows.

## 77. Integrity Verification

Use package-manager integrity hashes, signatures or checksums where supported.

## 78. Artifact Signing

Critical third-party and internal artifacts SHOULD progressively use verifiable signing/provenance.

## 79. SBOM

Critical releases SHOULD generate an SBOM listing included software components.

## 80. SBOM Formats

Use established formats such as SPDX or CycloneDX when adopted; exact choice requires implementation decision.

## 81. VEX

Vulnerability exploitability statements MAY be used to document whether known vulnerabilities affect a product.

## 82. Dependency Graph

Critical systems SHOULD be able to inspect dependency paths to understand why a package is present. `synapse-runtime`'s dependency graph is currently trivial to inspect: every crate depends on exactly `synapse-common`, confirmed directly via `cargo tree --workspace`.

## 83. Duplicate Dependencies

Avoid unnecessary multiple versions of the same library when they increase size or risk, while respecting ecosystem resolution realities.

## 84. Dependency Cycles

Internal package dependencies should avoid cycles that weaken modularity. `synapse-runtime`'s current graph is acyclic by construction — a flat topology with `synapse-common` as the sole shared dependency and no crate depending on another component crate.

## 85. Internal Dependencies

SynapseOS-owned packages are still dependencies and require versioning, compatibility and ownership. For the Runtime specifically, dependency direction between crates follows ARCH-002 §5–§6's trusted-core/replaceable-service boundary: a trusted-core crate MUST NOT take a concrete dependency on a replaceable-service crate's implementation. Where a trusted-core mechanism must consume replaceable-service behaviour — for example, Execution Coordinator consuming a Scheduler's proposed dispatch order (ARCH-002 §11, step 10) — that consumption occurs through the service's trait interface, never a concrete implementation dependency, consistent with ARCH-002 §19's prohibition on extensions bypassing trusted-core enforcement. The exact mechanism by which trait implementations are wired together (a composition root, dependency injection, or another pattern) remains an implementation decision, not fixed by this standard.

## 86. Monorepo Dependencies

Monorepos must define boundaries rather than relying on unrestricted cross-imports. `synapse-runtime`'s Cargo workspace realises this directly: each crate's own `Cargo.toml` explicitly declares its only permitted dependency (`synapse-common`), and nothing else is reachable without a further, explicit manifest change.

## 87. API Stability

Dependencies on unstable/private APIs increase upgrade risk and require justification.

## 88. Forking Dependencies

Fork only when necessary. A fork creates long-term maintenance and security obligations.

## 89. Vendoring

Vendored code requires provenance, licence, update tracking and clear local modifications.

## 90. Patching Third-Party Code

Record patches, upstream status and reapplication strategy.

## 91. Replacement Strategy

Strategic dependencies SHOULD have an identified abstraction boundary and plausible replacement path.

## 92. Exit Plan

For hosted providers, consider data export, credential transition, API replacement and operational continuity.

## 93. Single-Provider Risk

Critical functions depending on one provider require explicit availability and lock-in risk assessment.

## 94. Multi-Provider Strategy

Multi-provider support may improve resilience but adds complexity. Adopt based on real risk, not architecture theatre.

## 95. Failure Behaviour

Define timeout, retry, circuit breaking, fallback and degraded operation for remote dependencies.

## 96. Retry Safety

Retries must consider idempotency and cost, especially for AI and financial services.

## 97. Circuit Breakers

Use circuit breakers where repeated remote failure would amplify harm.

## 98. Fallbacks

Fallbacks must preserve security and quality expectations. A weaker fallback is not automatically safe.

## 99. Dependency Observability

Track latency, failures, rate limits, version/model identity and cost for material dependencies. Detailed observability practice is STD-010's dedicated subject matter (not yet reconciled); for the Runtime specifically, ARCH-002 §18 already fixes the minimum audit-event model this tracking would report through.

## 100. Change Detection

Monitor material provider/API changes where feasible.

## 101. Deprecation Notices

Track upstream deprecation notices and deadlines.

## 102. Dependency Removal

Remove unused dependencies promptly after verification.

## 103. Dead Code

Unused code paths may keep unnecessary dependencies alive. Analyse before removal.

## 104. Cleanup

After dependency replacement, remove obsolete configuration, secrets, CI steps and documentation.

## 105. Emergency Replacement

Maintain procedures for compromised packages, revoked services or urgent provider shutdown.

## 106. Compromised Dependency

Contain affected builds, identify versions, rotate exposed credentials, replace artifacts and assess distributed releases.

## 107. Registry Compromise

Be prepared to pause updates or use trusted mirrors when ecosystem compromise is suspected.

## 108. AI Agent Dependency Changes

Claude Code and other agents may propose dependencies but must explain purpose and must not silently install broad packages.

## 109. AI Package Hallucination

AI-generated package names must be verified before installation. Models can invent plausible non-existent or maliciously occupied packages.

## 110. AI Installation Authority

Agents should not receive unrestricted system-wide package installation authority by default.

## 111. AI Lockfile Changes

Large lockfile changes require review to ensure they correspond to intended manifest changes.

## 112. AI Provider Switching

Agents must not silently change AI provider/model dependencies when this affects cost, privacy, behaviour or policy.

## 113. Dependency Approval Levels

Low-risk dev dependency – normal review. Material runtime dependency – explicit PR review. Security/identity/crypto dependency – heightened review. Strategic provider/model dependency – architecture/RFC consideration. Any dependency proposed for a trusted-core crate (ARCH-002 §5–§6) is heightened review at minimum, regardless of which category it would otherwise fall into — see §9.

## 114. Dependency Record

Material dependency decisions SHOULD record name, purpose, owner, version strategy, source, licence, risk and alternatives.

## 115. Review Triggers

Re-review on major version, ownership transfer, licence change, compromise, abandonment, material pricing change or critical vulnerability.

## 116. Metrics

Potential metrics include outdated direct dependencies, known vulnerabilities by severity, update lead time, EOL components and dependency count trends.

## 117. Prohibited Practices

- Installing packages solely because an AI suggested them without verification.
- Committing uncontrolled floating production dependencies.
- Ignoring licence status.
- Using random binary download sites.
- Hiding critical vulnerability findings.
- Treating lockfiles as optional noise where reproducibility depends on them.
- Hard-coding architecture to a single model name without justification.

## 118. Exceptions

Exceptions require owner, rationale, scope, compensating controls and review/expiry trigger.

## 119. Compliance

New maintained components SHALL comply after approval. Existing dependencies are reviewed incrementally based on risk.

## 120. Success Criteria

The standard succeeds when SynapseOS can reuse external technology while understanding provenance, security, licence, continuity and replacement risks.

## 121. Open Questions

- Which dependency scanners will be free and mandatory initially?
- Will SPDX or CycloneDX be the initial SBOM format?
- Which package ecosystems will Phase 1 use? (Resolved for the Runtime: Cargo/crates.io, per Rust's selection — see STD-002 §6.)
- What provider abstraction boundary will be adopted? (Partially addressed: the Provider Adapter actor pattern, ARCH-002 §19 — the concrete abstraction interface remains open.)
- Which dependencies require formal approval before Claude Code may add them?

## 122. References

Internal:

- GOV-006 – Technical Strategy (Draft, legacy)
- GOV-008 – Release Strategy (Draft, legacy)
- GOV-009 – Risk Management (Draft, legacy)
- STD-002 – Coding Standards (Draft, reconciled)
- STD-004 – Repository Standards (Draft, reconciled)
- STD-007 – Security Standards (Draft, legacy)
- STD-008 – Testing Standards (Draft, legacy)
- STD-010 – Observability Standards (Draft, legacy)
- ARCH-001 – Constitutional Architecture (Draft)
- ARCH-002 – Runtime Architecture (Draft)
- ADR-0013 – Architectural Evolution of SynapseOS (Draft)
- ADR-0014 – Engineering Standards Corpus Reconciliation (Draft)

## Appendix A – New Dependency Review Checklist

- [ ] Clear need defined
- [ ] Existing alternatives checked
- [ ] Official source verified
- [ ] Package/publisher identity verified
- [ ] Maintenance health reviewed
- [ ] Security history reviewed
- [ ] Licence reviewed
- [ ] Transitive dependency impact considered
- [ ] Version strategy defined
- [ ] Lockfile impact reviewed
- [ ] Runtime vs dev classification correct
- [ ] Replacement cost considered
- [ ] Data/privacy impact considered
- [ ] Owner assigned

## Appendix B – Material Dependency Record Template

```
Dependency Name:
Category:
Purpose:
Owning Component:
Owner:
Official Source:
Current Version:
Version Constraint:
Lockfile:
Licence:
Runtime Criticality:
Data Shared Externally:
Network Required:
Known Alternatives:
Replacement Strategy:
Security Notes:
Approval Reference:
Last Reviewed:
Next Review Trigger:
```

## Appendix C – AI Provider Dependency Review

- [ ] Provider legal entity identified
- [ ] Official API/docs source verified
- [ ] Data handling reviewed
- [ ] Retention/training terms reviewed
- [ ] Pricing model understood
- [ ] Rate limits understood
- [ ] Model deprecation policy considered
- [ ] Regional availability considered
- [ ] Failure modes tested
- [ ] Provider-neutral adapter planned
- [ ] Fallback decision explicit
- [ ] Cost telemetry planned
- [ ] Evaluation baseline exists
- [ ] Exit strategy considered

## Appendix D – Emergency Dependency Incident Checklist

- [ ] Identify affected dependency and versions
- [ ] Determine affected repositories/releases
- [ ] Pause unsafe builds/deployments
- [ ] Preserve evidence
- [ ] Review exploitability
- [ ] Apply patch/upgrade/mitigation
- [ ] Rotate potentially exposed credentials
- [ ] Rebuild from trusted sources
- [ ] Verify artifacts
- [ ] Communicate impact
- [ ] Add regression/detection controls
- [ ] Record lessons learned

## Appendix E – Initial Dependency Risk Tiers

| Tier | Description | Examples | Review Expectation |
|---|---|---|---|
| Tier 1 | Strategic/Critical (includes any trusted-core crate dependency, per §9) | Identity, crypto, AI provider, core database | Explicit heightened review |
| Tier 2 | Material Runtime | Frameworks, SDKs, messaging clients | Explicit PR review |
| Tier 3 | Development/Build | Linters, test tools, generators | Normal review plus CI |
| Tier 4 | Optional/Experimental | Non-production experiments | Lightweight review; clearly isolated |

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-09 | Denver Jacobs | Initial comprehensive draft (source: `standards/STD-011_Dependency_Management_Standards_v0.1.docx`). |
| 0.1.0 | 2026-07-11 | Denver Jacobs | Reconciled with the constitutional architecture under ADR-0014, converting the legacy `.docx` to canonical Markdown in the same pass. Changes: §4 and §6 add terminology-drift disambiguations (generic "capability" versus the constitutional Capability primitive; "Runtime" as a dependency category versus the SynapseOS Runtime), cross-referencing rather than repeating STD-002 §13's fuller treatment; §9 and §113/Appendix E add that any dependency proposed for a trusted-core crate defaults to Tier 1 review, per ARCH-002 §5's "conceptually minimal" trusted-core principle and §22's mandatory-guarantees conformance requirement; §45 and §58 cite the Provider Adapter actor pattern and ARCH-002 §19's extension model directly; §84–§86 record `synapse-runtime`'s actual, verified dependency graph (acyclic, `synapse-common`-only) as the applied instance of the cycle-avoidance and monorepo-boundary principles already stated; §85 adds the trusted-core/replaceable-service dependency-direction rule ARCH-002 §5–§6 and §19 together establish, without prescribing the concrete Rust-level wiring mechanism, which remains an implementation decision; §5, §23, §38, §82 record `synapse-runtime`'s current, verified dependency posture (zero external dependencies, committed lockfile, unpinned toolchain, trivially inspectable graph) as factual cross-references to STD-004's own compliance review rather than restating it; §122 References and the frontmatter now cite ARCH-001, ARCH-002, ADR-0013, ADR-0014, and reconciled STD-002/STD-004. No duplicated content was found requiring removal — STD-002 and STD-004 already correctly deferred their own dependency-management content to this document before this reconciliation began. No section was rewritten for style alone; no obligation was removed, weakened, or newly imposed. This is presented as a single proposed Draft revision. No approval act has occurred. |

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
| Document ID | STD-011 |
| Repository path | standards/STD-011-Dependency-Management-Standards.md |
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

No field in this section may be populated until the corresponding act has genuinely occurred, evidenced per STD-001 §31. This table does not, and must not be read to, claim that any approval of STD-011 has occurred.
