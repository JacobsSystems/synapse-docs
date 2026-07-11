---
document_id: STD-002
title: Coding Standards
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
    - GOV-009 (Draft — legacy, unconverted)
  standards:
    - STD-001 (Approved)
    - STD-004 (Draft — reconciled)
    - STD-009 (Draft — legacy, not yet reconciled)
    - STD-010 (Draft — legacy, not yet reconciled)
    - STD-011 (Draft — reconciled)
    - STD-012 (Draft — legacy, not yet reconciled)
    - STD-013 (Draft — legacy, not yet reconciled)
    - STD-029 (Draft — legacy, not yet reconciled)
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
    - standards/STD-002_Coding_Standards_v0.1.docx (legacy source; disposition to be determined by a separate controlled task)
supersedes: None
superseded_by: None
ai_assistance: Conversion and drafting
---

# STD-002 — Coding Standards

*Filename pattern: `STD-002-Coding-Standards.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Conversion and drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

> **Status notice:** This document is **Draft**. This Markdown document is the candidate canonical controlled source-format conversion of `standards/STD-002_Coding_Standards_v0.1.docx`, combined with the minimum reconciliation against the constitutional architecture authorised by ADR-0014. The original Word document is retained unchanged as a legacy source artifact for provenance and conversion verification pending a separate controlled archival decision. It is not co-canonical and must not receive future substantive edits. This Markdown document remains a Draft and is not approved or operative merely because it exists, is staged, committed, or pushed.

## 1. Purpose

Define mandatory coding requirements for SynapseOS source code so that human developers and AI coding agents produce secure, maintainable, testable, observable and replaceable software.

## 2. Scope

Applies to production code, libraries, services, CLI tools, SDKs, plugins, automation scripts, infrastructure code and generated code committed to SynapseOS repositories, including the Runtime (`synapse-runtime`, per ARCH-002). Language-specific standards may extend but must not silently weaken this standard.

## 3. Engineering Intent

Code is an implementation of approved system intent, not the primary location for undocumented architecture. Prefer clarity over cleverness and explicit contracts over hidden behaviour. This is a coding-level application of `.ai/ARCHITECTURAL-CONTEXT.md`'s "architecture before implementation" principle and ARCH-001 §1.

## 4. Core Rules

- Keep modules cohesive and loosely coupled.
- Depend on abstractions at architectural boundaries — for the Runtime specifically, this is the trusted-core/replaceable-service boundary ARCH-002 §5–§6 defines.
- Avoid direct AI-vendor coupling outside provider adapters.
- Make failure explicit.
- Validate untrusted input.
- Never hard-code secrets.
- Keep changes small and reviewable.
- Test behaviour, especially critical paths.
- Preserve traceability to approved work.

## 5. Architecture Compliance

Code MUST conform to approved architecture, standards, ADRs and RFCs applicable to the change. If implementation reveals that the specification is wrong, update the controlled design through the governance process rather than silently diverging. For constitutional concepts specifically (ARCH-001 §5–§6), this means raising an apparent contradiction through the ADR process per ARCH-001 §10, not resolving it locally in code.

## 6. Language Strategy

SynapseOS is polyglot only where justified. Each additional language increases operational and maintenance cost. Language choice must consider safety, ecosystem maturity, deployment, performance, contributor accessibility and long-term support. Rust has been selected for the Runtime (`synapse-runtime`), consistent with these criteria; this does not itself justify Rust for every future SynapseOS repository, each of which makes its own justified choice under this section.

## 7. Repository Hygiene

Repository-level hygiene — build artifacts, local caches, credentials, private keys, machine-specific configuration, temporary debug files, ignore rules, and generated-content handling — is STD-004's dedicated subject matter (STD-004 §22–§26, §32). This section is retained only as a coding-level pointer: code must not assume or reintroduce anything STD-004 prohibits at the repository level.

## 8. Formatting

Use automated formatters supported by the selected language ecosystem. Formatting rules SHOULD be enforced consistently in CI. Do not spend review time debating style that can be automated.

## 9. Static Analysis and Linting

Production repositories SHOULD enable appropriate compiler warnings, linters and static analysis. New high-severity findings must not be ignored without documented rationale.

## 10. Naming

Names must reveal intent. Avoid unexplained abbreviations and misleading generic names such as data, thing, manager or helper where a precise domain name exists. Public APIs require especially stable and descriptive naming. Detailed naming conventions are STD-006's dedicated subject matter (not yet reconciled); this section states the coding-level principle only.

## 11. Functions and Methods

Functions SHOULD have one clear responsibility, explicit inputs and understandable outputs. Avoid excessive parameter lists, hidden global dependencies and unexpected side effects.

## 12. Classes, Types and Modules

Types and modules SHOULD represent coherent responsibilities. God objects, circular dependencies and modules that accumulate unrelated behaviour are prohibited design smells requiring review. ARCH-002 §6 applies this same discipline at the Runtime component level: avoid both a monolithic component that silently owns everything and excessive micro-decomposition.

## 13. Interface Design

Interfaces at subsystem boundaries must be minimal, explicit and versionable — the coding-level application of ARCH-001's "interfaces are architecture" principle and ARCH-002 §20's Runtime interface contracts. Do not expose implementation details unnecessarily. Provider abstractions must describe capabilities rather than vendor-specific product terminology.

*Terminology note:* "capabilities" in the preceding sentence is used in its ordinary sense — functional abilities a provider offers — and is distinct from the constitutional **Capability** (ARCH-001 §5.2): the unit of authority binding a target, permitted operations, constraints, provenance, and a revocation handle. Neither usage governs the other; this note exists solely to prevent the two from being conflated, per ADR-0014 §9.

## 14. Dependency Direction

Core domain logic SHOULD NOT depend directly on UI frameworks, cloud vendors, AI vendors or infrastructure details. Use ports, adapters or equivalent separation where the architectural boundary warrants it. The Runtime's own dependency graph — every crate depending only on `synapse-common`, zero external dependencies (ARCH-002 §5–§6) — is the current, strictest applied instance of this principle.

## 15. Dependency Management

Dependency assessment (necessity, maturity, maintenance, licence, security history, transitive dependencies, exit strategy) is STD-011's dedicated subject matter. Every third-party dependency introduces supply-chain, licensing and maintenance risk; apply STD-011's assessment before adding one.

## 16. Version Pinning

Reproducible version constraints and lockfile practice are STD-011's dedicated subject matter. Lockfiles SHOULD be committed when standard practice for the ecosystem — `synapse-runtime` commits `Cargo.lock`, consistent with this.

## 17. Secrets

Secrets MUST NOT appear in source code, examples, logs, test fixtures or committed configuration. Use approved secret-management mechanisms and environment-specific injection. This is a source-code-level application of a principle STD-004 §32 also addresses at the repository-tooling level (secret scanning) — the two are complementary, not duplicative, per ADR-0014 §12.

## 18. Input Validation

Treat external input as untrusted. Validate type, format, size, range, encoding and authorisation context as appropriate. Validation must occur at trust boundaries, not only in user interfaces.

## 19. Output Encoding

Output destined for HTML, shells, SQL, logs, URLs or other interpreters must be encoded or parameterised appropriately. Avoid string concatenation for security-sensitive interpreter contexts.

## 20. Authentication and Authorisation

Authentication establishes identity; authorisation determines permission. Code must not confuse the two. Sensitive operations require server-side or trusted-boundary enforcement. This is precisely ARCH-001 §6's constitutional identity/authority separation ("Actor identity grants no authority... Authority exists only through explicit capability derivation"), applied at the coding level: authority is proven by presenting a valid Capability, never inferred from identity.

## 21. Least Privilege

Services, plugins, agents and tools SHOULD receive only the permissions required for their task. Autonomous execution capabilities require explicit capability boundaries. For the Runtime, this is realised precisely by the constitutional Capability model (ARCH-001 §5.2): least privilege is not a coding convention layered on top of the Runtime, it is what the Capability model exists to enforce structurally.

## 22. AI and Agent Safety

AI-generated instructions are untrusted input. Tool execution must enforce permissions independently of model output. High-impact actions should support approval gates, dry-run modes, idempotency and auditability where appropriate. This is consistent with, and does not restate, GOV-010 §15's constraints on AI-assisted decision-making.

## 23. Prompt Injection Resistance

Content retrieved from users, websites, files, email or tools must not automatically become trusted system instruction. Boundaries between instructions, data and tool authority must be explicit.

## 24. Error Handling

Errors must be handled intentionally. Do not swallow exceptions silently. Error messages should provide actionable context without leaking secrets or sensitive internals.

## 25. Error Taxonomy

Where appropriate, distinguish validation errors, authentication failures, authorisation failures, dependency failures, transient errors, permanent errors, conflicts, timeouts and internal defects. ARCH-002's own `RuntimeError` representation is deliberately minimal, with per-component error taxonomies deferred as an implementation-phase refinement (ARCH-002 §23); this section's taxonomy is a reasonable starting point for that future refinement, not a conflicting one.

## 26. Retries

Retries require bounded attempts, backoff and awareness of idempotency. Never retry destructive or non-idempotent operations blindly.

## 27. Timeouts

External calls SHOULD have explicit timeouts. Infinite waits are prohibited for production dependencies unless formally justified.

## 28. Cancellation

Long-running operations SHOULD support cancellation or cooperative shutdown where the platform and language permit. ARCH-002 §10 already reserves an Execution Context field for cancellation state; this section's guidance applies directly to code built on it.

## 29. Resource Management

Files, sockets, database connections, locks and other resources must be released deterministically where practical. Use language-native safe resource patterns.

## 30. Concurrency

Shared mutable state requires explicit design. Race conditions, deadlocks and ordering assumptions must be considered and tested. Prefer simpler concurrency models when performance does not justify complexity. ARCH-002 §12 has already made this Runtime's concurrency decision: at most one message processed per actor instance at a time, with no shared mutable state between actors by construction — code within an actor's own message-handling logic should not need to defend against the concurrency hazards this section warns about, and any apparent need to do so is worth questioning against ARCH-002 §12 before being treated as a coding-level problem to solve locally.

## 31. Data Integrity

Changes affecting durable state SHOULD define transaction boundaries, consistency expectations, failure behaviour and recovery. Never assume a distributed workflow is atomic.

## 32. Idempotency

Operations likely to be retried, replayed or delivered more than once SHOULD be idempotent or use explicit deduplication mechanisms. ARCH-002 §8 reserves replay-protection information in the message envelope for exactly this purpose.

## 33. Logging

Structured logging practice, log levels, and required context are STD-010's dedicated subject matter (not yet reconciled). Use structured logging where practical; logs must not expose secrets, access tokens, or unnecessary personal data.

## 34. Observability

Health, metrics, and trace-context requirements are STD-010's dedicated subject matter (not yet reconciled). For the Runtime specifically, ARCH-002 §18 already fixes the minimum audit-event model and the security-audit/operational-tracing/debugging/billing/provenance distinction — Runtime-adjacent code should conform to ARCH-002 §18 directly rather than to a separate, independently-derived observability scheme.

## 35. Comments

Comments explain why, constraints or non-obvious consequences. Do not restate obvious code. Stale comments are defects.

## 36. Documentation Comments

Public APIs and extension points SHOULD document purpose, parameters, outputs, errors, side effects, security expectations and stability where relevant.

## 37. Magic Values

Unexplained magic numbers and strings SHOULD be replaced with named constants, configuration or domain types when doing so improves clarity.

## 38. Configuration

Environment-specific behaviour belongs in controlled configuration, not source edits. Configuration must be validated at startup or before use. Detailed configuration management practice is STD-013's dedicated subject matter (not yet reconciled).

## 39. Feature Flags

Feature flags require ownership and removal plans. Permanent forgotten flags create hidden complexity and must be treated as technical debt.

## 40. Backward Compatibility

Public contracts must not be broken casually. Breaking changes require explicit versioning, migration guidance and approval according to release strategy. Detailed release and versioning practice is STD-012's dedicated subject matter (not yet reconciled).

## 41. Deprecation

Deprecated APIs should identify replacement guidance and expected removal path. Do not leave consumers without a migration route when one is feasible.

## 42. Database Code

Use parameterised queries or safe ORM mechanisms. Schema changes require migration and rollback/forward-recovery consideration. Production data assumptions must be explicit. This anticipates ARCH-002's deferred Storage Architecture (ARCH-002 §23); no storage technology is prescribed by this section or by that deferral.

## 43. API Code

Validate requests, enforce authorisation, use stable error contracts, define timeouts and protect against unbounded payloads. Detailed API design practice is STD-009's dedicated subject matter (not yet reconciled); STD-009's REST/HTTP assumptions have not yet been validated against the Runtime's actor/capability/message model and should not be treated as settled for the Runtime's own external surface until that reconciliation occurs.

## 44. Plugin Code

Plugins must not receive unrestricted platform authority by default. Plugin lifecycle, capabilities, compatibility and failure isolation must follow approved architecture. This is directly realised by ARCH-002 §19's extension model: no plugin or extension bypasses capability enforcement, message integrity, actor isolation, execution ownership, or lifecycle controls, and each extension category receives only the scoped interface its role requires. Detailed SDK/plugin/extension practice is STD-029's dedicated subject matter (not yet reconciled, but a high-value future reconciliation target given this alignment).

## 45. Provider Adapter Code

AI provider-specific SDKs, request formats and product names SHOULD remain inside provider adapters. Core modules consume provider-neutral contracts. The Runtime realises this directly: a provider adapter is modelled as an ordinary, capability-scoped Actor — never granted ambient access to the underlying provider — consistent with RFC-0014 §28's "provider output is untrusted; models cannot grant authority."

## 46. Generated Code

Generated code must identify its generator or provenance where practical. Generated output is not exempt from security, licensing, testing or review requirements.

## 47. AI-Generated Code

Code produced by Claude Code, ChatGPT or other AI tools is treated as untrusted draft implementation until reviewed and tested. AI must not invent passing tests, completed security reviews or unsupported APIs. Consistent with STD-001 §33 and `.ai/ARCHITECTURAL-CONTEXT.md`'s AI Working Rule.

## 48. Testability

Design dependencies so that critical behaviour can be tested without uncontrolled external services. Avoid architecture that makes deterministic testing unnecessarily difficult.

## 49. Unit Tests

Unit tests SHOULD be fast, isolated and focused on observable behaviour. Avoid tests that merely duplicate implementation details.

## 50. Integration Tests

Use integration tests for boundaries such as databases, event buses, provider adapters, file systems and service contracts.

## 51. Security Tests

Security-sensitive code requires tests for denied paths, invalid input and abuse cases, not only successful behaviour.

## 52. Determinism

Tests SHOULD avoid uncontrolled time, randomness, network access and global state. Inject clocks, random sources and external clients where justified. Consistent with ARCH-002 §12's "determinism where practical" execution principle.

## 53. Performance

Do not optimise without evidence. Performance-sensitive code should define measurable targets and use representative benchmarks. This is the coding-level application of `.ai/ARCHITECTURAL-CONTEXT.md` §2's "correctness before optimisation" principle — not a separate or weaker rule. Detailed performance and resource-governance practice is STD-024's dedicated subject matter (not yet reconciled).

## 54. Complexity

Complexity must earn its cost. Prefer standard library capabilities and straightforward designs over custom frameworks unless requirements justify them. Consistent with `.ai/ARCHITECTURAL-CONTEXT.md`'s "avoid unnecessary abstraction" principle.

## 55. Duplication

Avoid harmful duplication, but do not create premature abstractions merely to remove a few repeated lines. Abstract when concepts are genuinely shared.

## 56. Dead Code

Remove dead code rather than commenting it out. Version control preserves history.

## 57. TODO Markers

TODO/FIXME markers SHOULD include sufficient context and, where practical, an issue or owner. Security-critical TODOs must not be forgotten in production. This states a minimum bar; specific repositories or components MAY adopt a stricter zero-TODO discipline where practical, as `synapse-runtime`'s initial scaffold did.

## 58. Commit Scope

A code change SHOULD have a coherent purpose. Avoid mixing unrelated refactors, formatting and feature behaviour when separation improves review.

## 59. Code Review Readiness

Before requesting review: format code, run relevant tests, inspect the diff, remove debug artifacts, update documentation and identify known risks.

## 60. Definition of Done

A change is complete only when applicable implementation, tests, documentation, review, security considerations and traceability obligations are satisfied. Consistent with GOV-010 §5: review and approval remain distinct acts, and satisfying review-related obligations here does not itself constitute approval.

## 61. Prohibited Practices

- Hard-coded production secrets.
- Silent exception swallowing.
- Disabling security checks to make tests pass.
- Fabricating test results.
- Direct vendor coupling across core modules without approval.
- Unbounded retries.
- Logging credentials or tokens.
- Executing model-generated commands without independent controls.
- Committing knowingly vulnerable dependencies without explicit risk acceptance.
- For Runtime code specifically, the prohibitions ARCH-002 §22 already fixes: ambient authority, direct shared mutable actor state, mutable messages, mutable capability authority, implicit capability inheritance, actor communication without validated send authority, execution without a single owning actor, and spontaneous authority creation.

## 62. Exceptions

Exceptions require documented rationale proportional to impact. Repeated exceptions indicate that the standard or architecture may need revision.

## 63. Compliance

New production code SHALL comply after approval of STD-002. Existing code SHOULD be migrated incrementally, prioritising security-critical and actively modified areas.

## 64. Success Criteria

The standard succeeds when SynapseOS code remains understandable, secure, testable, replaceable and maintainable even as AI models, contributors and technologies change.

## 65. Open Questions

- Which language will implement SynapseOS Core? (Resolved for the Runtime: Rust — see §6. Other repositories remain open.)
- Which linters and formatters become mandatory per language? (For Rust: `cargo fmt` and `cargo clippy` are already in use in `synapse-runtime`; not yet formally mandated here.)
- What minimum coverage or mutation-testing thresholds are appropriate?
- Which security scanners become CI quality gates?
- How will AI-generated code provenance be recorded?

## 66. References

Internal:

- GOV-004 – Engineering Principles (Approved)
- GOV-006 – Technical Strategy (Draft, legacy)
- GOV-009 – Risk Management (Draft, legacy)
- GOV-010 – Decision Framework (Approved)
- STD-001 – Documentation Standards (Approved)
- STD-004 – Repository Standards (Draft, reconciled)
- ARCH-001 – Constitutional Architecture (Draft)
- ARCH-002 – Runtime Architecture (Draft)
- ADR-0013 – Architectural Evolution of SynapseOS (Draft)
- ADR-0014 – Engineering Standards Corpus Reconciliation (Draft)

External authoritative references may be added during formal review and language selection.

## Appendix A – Pre-Commit Checklist

- [ ] Change has a coherent purpose
- [ ] Code formatted with approved tooling
- [ ] Relevant lint/static checks pass
- [ ] Relevant tests pass
- [ ] No secrets or debug artifacts included
- [ ] Error paths considered
- [ ] Security-sensitive input validated
- [ ] Documentation updated where behaviour changed
- [ ] Dependencies justified and reviewed
- [ ] Diff inspected by the author/implementer

## Appendix B – AI-Generated Code Review Checklist

- [ ] Referenced APIs and library versions actually exist
- [ ] Code matches approved architecture/RFC
- [ ] No fabricated tests or claims
- [ ] Authentication and authorisation paths reviewed
- [ ] Input boundaries validated
- [ ] Secrets handling reviewed
- [ ] Failure, timeout and retry behaviour reviewed
- [ ] Concurrency assumptions reviewed where applicable
- [ ] Tests include negative and abuse cases
- [ ] Human reviewer understands the change before approval

## Appendix C – Pull Request Coding Gate

| Gate | Required | Evidence |
|---|---|---|
| Formatting | Yes | Automated formatter |
| Lint/Static Analysis | Yes where configured | CI result |
| Tests | Yes | CI/local evidence |
| Documentation | When behaviour/contracts change | Updated controlled docs |
| Security Review | Risk-based | Reviewer evidence |
| Traceability | For governed work | ARCH/RFC/ADR references |

## Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-09 | Denver Jacobs | Initial comprehensive draft (source: `standards/STD-002_Coding_Standards_v0.1.docx`). |
| 0.1.0 | 2026-07-11 | Denver Jacobs | Reconciled with the constitutional architecture under ADR-0014, converting the legacy `.docx` to canonical Markdown in the same pass. Changes: §7 (Repository Hygiene), §15–§16 (Dependency Management, Version Pinning) trimmed to cross-reference STD-004 and STD-011 respectively rather than restate their dedicated subject matter, per ADR-0014 §8; §33–§34 (Logging, Observability) trimmed to cross-reference STD-010 and ARCH-002 §18; §13 adds an explicit terminology-drift disambiguation between the generic "capabilities" (functional abilities) and the constitutional Capability primitive (ARCH-001 §5.2), per ADR-0014 §9; §20, §21, §30, §44, §45 add substantive citations to ARCH-001/ARCH-002 where the existing guidance already anticipated the identity/authority separation, the capability model, the concurrency model, the extension model, or the provider-adapter pattern, per ADR-0014 §11; §61 (Prohibited Practices) extends the existing list with ARCH-002 §22's Runtime-specific prohibitions; §6 and §65 record that Rust has been selected for the Runtime, resolving one of two originally open questions; §66 References and the frontmatter now cite ARCH-001, ARCH-002, ADR-0013, ADR-0014, and reconciled STD-004, previously absent. No section was rewritten for style alone; no obligation was removed, weakened, or newly imposed beyond the ARCH-002 §22 additions to §61, which state prohibitions already constitutionally settled elsewhere. This is presented as a single proposed Draft revision. No approval act has occurred. |
| 0.1.0 | 2026-07-11 | Denver Jacobs | Minor cross-reference correction made as a direct consequence of the STD-011 reconciliation performed in the same session: §15, §16 and the frontmatter's `related_documents` entry for STD-011 removed the now-stale "(not yet reconciled)" qualifier, since STD-011 was reconciled immediately after this document. No other content changed; this is not a re-reconciliation of STD-002. |

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
| Document ID | STD-002 |
| Repository path | standards/STD-002-Coding-Standards.md |
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

No field in this section may be populated until the corresponding act has genuinely occurred, evidenced per STD-001 §31. This table does not, and must not be read to, claim that any approval of STD-002 has occurred.
