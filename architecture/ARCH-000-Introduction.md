---
document_id: ARCH-000
title: Introduction
project: SynapseOS
specification: SynapseOS — whole-system introduction and shared architectural context
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: TBD
created: 2026-07-09
last_updated: 2026-07-10
classification: Public
related_documents:
  governance:
    - GOV-001 (Draft)
    - GOV-003 (Draft)
    - GOV-004 (Draft)
    - GOV-006 (Draft)
    - GOV-010 (Draft)
  standards:
    - STD-001 (Draft)
  architecture: None
  rfcs:
    - RFC-0001 (Draft)
    - RFC-0002 (Draft)
    - RFC-0003 (Draft)
    - RFC-0010 (Draft)
    - RFC-0014 (Draft)
  adrs:
    - ADR-0001 (Proposed)
    - ADR-0002 (Proposed)
    - ADR-0003 (Proposed)
  roadmap: None
  research: None
  operational: None
  source_artifacts:
    - architecture/00-Introduction.md (renamed to this file via git mv; same continuing ARCH-000 artifact, not a second source document)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# Introduction

*Filename: `architecture/ARCH-000-Introduction.md` (per STD-001 §7–§8 identifier and naming rules).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§32, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Purpose

This document is Chapter 0 of the SynapseOS Architecture Book. It introduces the Architecture Book itself, establishes the shared vocabulary and structural context that later architecture chapters will build on, and records — without adopting or approving — the technical direction currently expressed across SynapseOS's governance, ADR, and RFC corpus. Nothing in this document constitutes formal approval of any decision it describes. Its own status is Draft, and it becomes authoritative only for what it is: an introduction, not a specification.

## 2. Scope

This Introduction covers, at a conceptual level: why SynapseOS's architecture is organized the way current Draft material proposes, the system's architectural planes and component categories as currently drafted in RFC-0001, and the Phase 1 direction recorded in ADR-0001, ADR-0002, and ADR-0003. It does not select final technologies, does not accept any Proposed ADR, and does not define the detailed internals of any subsystem — those belong to later architecture chapters, RFCs, and ADRs, none of which are assigned an identifier or created here. Where this document is silent on a topic, that silence should be read as "not yet addressed," not as "resolved elsewhere."

## 3. Background

SynapseOS's Draft governance and RFC material frames the project as a platform for coordinating multiple AI models, tools, and intelligence-related services rather than a single application built around one model or vendor. RFC-0001 (Draft) states this as a goal: to "create a local-first, modular, observable, secure, replaceable and evolvable intelligence operating system with explicit control over identity, capability, execution, memory and knowledge" (§3, Goals).

The problem this framing responds to, per GOV-001 (Draft, "Guiding Principles" and "Objectives"), is that AI capabilities and providers are changing quickly, and a project built tightly around any one of them risks obsolescence. Current Draft governance direction (GOV-004, Principle 4 "Model Agnostic"; GOV-006, "AI Provider Strategy") and Draft RFC direction (RFC-0014, §2 Goals) both converge on the same response: treat models and providers as replaceable components behind stable contracts, not as the architecture itself.

## 4. Problem Statement

RFC-0001 (Draft) states the problem directly: "SynapseOS requires a stable system contract that prevents models, agents, plugins, storage technologies and deployment choices from becoming the architecture itself" (§2, Problem Statement). In other words, the coordination problem SynapseOS's architecture is meant to solve is how to let heterogeneous, fast-changing intelligence capabilities — different models, different tools, different storage and deployment technologies — plug into a stable system without each new capability forcing a redesign of the whole. RFC-0001's five architectural planes (§5) and its Technology Neutrality rule (§19: "Architecture contracts define semantics first. Technology choices require ADRs and implementation RFCs.") are both direct responses to this problem as currently drafted.

## 5. Design Goals

The following goals are drawn from current Draft governance principles and the Draft RFC-0001 goal statement; none has been formally ratified, but all currently point in the same direction across the reviewed corpus:

- **Modularity and replaceability** — "Every component should be replaceable through well-defined interfaces" (GOV-004, Principle 3); RFC-0001 §13 Dependency Rule constrains core platform components from depending on domain plugins.
- **Model and provider neutrality** — "No module may depend directly on a specific AI vendor. Providers must be replaceable without architectural redesign" (GOV-006, AI Provider Strategy); RFC-0014 (Draft) exists specifically to define provider-neutral model access.
- **Observability** — RFC-0001 defines a dedicated Observability Plane (§10); GOV-004 Principle 7 expects "every critical service exposes logs, metrics and health information."
- **Security by design** — GOV-006 Security Strategy calls for embedding authentication, authorization, secrets management, and auditing "from the beginning rather than adding them later"; RFC-0001 §25 and §30 set deny-by-default and untrusted-input expectations at the RFC level.
- **A local-first starting point, where supported** — RFC-0001 §17 states the reference bootstrap "SHALL run on a single developer workstation or affordable server without requiring a hyperscale cloud," while §18 keeps a path open to later distribution.
- **Human governance** — GOV-004 Principle 5: "Humans retain authority over strategic decisions and production releases"; GOV-010 §15 requires that AI systems not "silently approve their own high-impact actions."
- **Specification-led evolvability** — GOV-004 Principle 1 and GOV-003's "documentation-first, architecture-driven governance model" both require an approved specification, and where required an RFC, before production code; RFC-0001 §19 formalizes this as a rule that technology choices require ADRs.
- **Explicit contracts over implicit coupling** — a theme repeated across RFC-0001 (§13, §26), RFC-0002 (§10, §19), and RFC-0010 (§13), all of which favor declared interfaces over direct, informal dependencies between components.

These are current Draft design goals, not verified system properties.

## 6. Non-Goals

This Introduction does not:

- select final implementation technologies — ADR-0001, ADR-0002, and ADR-0003 remain Proposed, not accepted, and this document does not change that;
- claim that any part of the architecture it describes has been implemented — no source code exists in this repository at the time of writing;
- define every subsystem in detail — component-, interface-, and protocol-level specification is the job of later architecture chapters, RFCs, and ADRs;
- resolve the open governance gaps identified during the evidence review that preceded this chapter (for example, named approval authorities, or a memory- and tool-governance framework) — these remain open;
- replace or reduce the authority of the RFCs, ADRs, standards, or future implementation specifications it references. Where this document paraphrases a Draft RFC or a Proposed ADR, the source document remains the primary reference.

## 7. Architectural Context

RFC-0001 (Draft) organizes the system into five architectural planes (§5–§10):

- **Control Plane** — owns configuration, identity, capability policy, lifecycle, registry, approvals, and administrative control.
- **Execution Plane** — runs bounded tasks, processes, tools, workflows, and agent actions, and "SHALL not invent authority" of its own.
- **Intelligence Plane** — hosts model abstraction, reasoning, planning, reflection, and cognitive modules; RFC-0001 is explicit that "intelligence outputs are proposals until authorised execution."
- **Data Plane** — owns durable state, memory, knowledge, event persistence, artifacts, and derived indexes.
- **Observability Plane** — collects logs, metrics, traces, audit events, and operational evidence across the other four planes.

These planes are currently Draft architecture-contract definitions in RFC-0001 — a way of dividing responsibility conceptually — not evidence that five corresponding subsystems have been built. Cross-plane interaction is required to use "explicit contracts" (RFC-0001 §5), a rule that recurs throughout the more detailed RFCs (RFC-0002, RFC-0003, RFC-0010, RFC-0014) reviewed for this chapter.

## 8. Architecture

At the level this Introduction operates on, the current Draft direction describes a system coordinated by a small kernel, with most behavior implemented through explicit, versioned contracts rather than direct coupling:

- **Kernel coordination.** RFC-0002 (Draft) defines the kernel as coordinating "trusted platform mechanics" — lifecycle, component registry, service resolution, execution-context propagation, cancellation, health, and policy-enforcement integration (§2–§3) — and explicitly states it "is not an LLM, chatbot, business workflow engine or domain application" (§2). RFC-0001 §11 adds that the kernel "SHALL remain smaller than the total platform and avoid domain-specific intelligence."
- **Modular boundaries.** ADR-0003 (Proposed) recommends implementing Phase 1 core control and execution services "in one deployable .NET host with strict modules," with cross-module access mediated by interfaces rather than direct data access — a decision that, per the evidence reviewed, is consistent with RFC-0001 §31's own Draft direction to "begin as modular monolith where economical."
- **Execution lifecycle.** RFC-0003 (Draft) defines a Process → Run → Task → Step hierarchy with an explicit state machine (Created, Queued, Admitted, Running, Waiting, Suspended, Cancelling, Cancelled, Succeeded, Failed, TimedOut), budgets, deadlines, and cancellation propagation.
- **Provider abstraction.** RFC-0014 (Draft) defines provider-neutral access to language, reasoning, embedding, reranking, and multimodal models, with the explicit goal of preventing "provider lock-in" while preserving "exact model identity" (§2).
- **Extension and plugin boundaries.** RFC-0010 (Draft) defines how the system discovers, validates, loads, isolates, and manages extensions, including an explicit distinction between in-process trusted plugins and out-of-process, higher-risk plugins (§10).
- **Data and state ownership.** RFC-0001 §23 requires that "each authoritative state domain has one declared owner and write contract."
- **Observability.** Addressed structurally by RFC-0001's Observability Plane and, at the component level, by RFC-0002 §14's requirement that kernel lifecycle events use stable schemas.

This is a conceptual sketch drawn from Draft RFC content and a Proposed ADR; it intentionally stops short of the level of detail that later, dedicated architecture chapters are expected to provide.

## 9. Components

RFC-0001 §12 (Draft) enumerates the following core component categories: Kernel, Scheduler, Event Bus, Identity Service, Capability Service, Tool Gateway, Configuration Service, Plugin Runtime, Memory Service, Knowledge Service, Model Abstraction Layer, and Observability. This is presented in the source RFC as a Draft enumeration of intended responsibilities, not as an inventory of components that currently exist in code — this repository contains no implementation of any of them at the time of writing.

## 10. Interfaces

Current Draft and Proposed direction consistently favors contract-first interaction over direct, informal coupling:

- RFC-0002 §25 defines a set of language-neutral reference interfaces for the kernel (`IKernel`, `IComponent`, `IComponentRegistry`, `IExecutionContext`, `IHealthContributor`).
- RFC-0010 §12 requires that out-of-process plugins communicate through "a versioned RPC/message interface with identity, timeout and resource controls."
- RFC-0014 defines a request/response envelope model for model invocation (§6–§7), independent of any specific provider's native API shape.
- ADR-0001 (Proposed) states that cross-language communication (between the primary implementation language and any Python adapters) "uses explicit RPC/message/tool contracts," and that there should be "no direct Python imports inside kernel."

None of the sources reviewed for this chapter commits to a specific wire protocol or serialization technology for these interfaces — that choice is left open for a future RFC or ADR, and this document does not invent one.

## 11. Data and State

RFC-0001 §23 (Draft) establishes that "each authoritative state domain has one declared owner and write contract." ADR-0003 (Proposed) adds, at the Phase 1 implementation level, that there should be "no arbitrary cross-module database access," with modules instead depending on interfaces and contracts.

Memory is named as a planned system capability — it appears as "Memory Service" in RFC-0001 §12's component list, and as part of the Data Plane in RFC-0001 §9. However, **formal memory governance — rules for memory access, retention, and data ownership specific to memory as a subsystem — was not found in the governance and ADR material reviewed for this chapter, and remains an open question** (see §19, Open Questions). This document does not attempt to resolve it.

## 12. Security Considerations

The Draft security direction currently expressed across the reviewed sources includes:

- **Deny by default and least authority.** RFC-0001 §30 calls for enforcing "deny-by-default" for consequential actions, and §22 states that components "SHALL fail without silently broadening authority."
- **Untrusted model and plugin output.** RFC-0001 §30 requires treating "all model/retrieval/plugin content as untrusted"; RFC-0014 §28 states "provider output is untrusted; models cannot grant authority."
- **Explicit trust classes.** RFC-0010 §10–§11 distinguishes in-process ("effectively highly trusted") plugins from out-of-process, higher-risk plugins, each with different review expectations.
- **Identity and capability enforcement at boundaries, not in prompts.** RFC-0001 §25: "Identity and capability checks occur at enforcement boundaries; prompts are not access control."
- **Human review for high-impact actions.** GOV-010 §15 (Draft, "AI-Assisted Decision-Making") states that "AI must not silently approve its own high-impact actions or fabricate evidence," and names autonomous execution as a security-critical decision category (§4, Class C) requiring explicit risk review.

None of these controls should be read as implemented. They are the security direction currently recorded in Draft RFC and governance material; no code or deployed system exists in this repository to enforce them yet.

## 13. Reliability and Failure Handling

RFC-0001 §22 (Draft) requires that "components SHALL fail without silently broadening authority," with control-plane failure defaulting to safe denial for consequential actions. RFC-0003 (Draft) defines the supporting mechanics at the execution level: explicit budgets and deadlines (§8–§9), cancellation that propagates from parent to child tasks unless a detached task is specifically approved (§10), retries that require explicit attempt records and idempotency (§12), and a requirement (§17) that "partial failure" be represented accurately rather than "collapsed into false success." ADR-0003 (Proposed) contributes an isolation strategy at the Phase 1 level: higher-risk or less-trusted work — Python adapters, sandboxed tools, higher-risk plugins — runs in separate processes rather than inside the primary host, so that failure in that work is less likely to affect the core system.

## 14. Observability

RFC-0001 defines a dedicated Observability Plane (§10, Draft) responsible for collecting "logs, metrics, traces, audit events and operational evidence" across the other planes. RFC-0002 §14 requires that kernel lifecycle events use stable, well-defined schemas so they can be consumed consistently. No specific observability backend or toolchain is committed to by any source in this chapter's primary evidence set, and no observability tooling of any kind is deployed in this repository at the time of writing.

## 15. Performance Considerations

No source reviewed for this chapter establishes concrete performance targets, and none are invented here. Latency, throughput, resource use (particularly under a local-first deployment model), model-invocation cost, and the constraints of running on developer-workstation-class hardware (per RFC-0001 §17's local-first bootstrap requirement) are identified as future measurable concerns that later architecture and RFC work will need to quantify. No benchmark, SLA, or numeric target is established by this document.

## 16. Scalability Considerations

Current Draft direction favors starting simple and distributing later, rather than the reverse. RFC-0001 §18 (Draft, "Scale-Out Model") requires that "contracts SHALL permit later distribution of scheduler, bus, storage and services without rewriting application semantics," while §31 ("Migration and Evolution") directs implementations to "begin as modular monolith where economical" and "split components only when measured scaling, isolation or operational needs justify it." ADR-0003 (Proposed) reflects this directly for Phase 1: a single-host modular monolith now, with extraction to separate processes or services deferred until a measured need arises (ADR-0003, §9, "Extraction Strategy" — extract "when measured scaling, security isolation, independent release or reliability requirements justify it"). This is a stated intention, not a demonstrated scaling result.

## 17. Constraints

Constraints currently in view come from several distinct kinds of source and should not be read as a single, uniformly binding list:

**Draft governance constraints** (GOV-006, "Technology Philosophy"): prefer "stable, widely supported technologies that encourage modularity, portability and long-term maintainability," and avoid "unnecessary complexity and vendor lock-in."

**Draft RFC constraints** (RFC-0001): the kernel "SHALL remain smaller than the total platform" (§11); "core platform components MUST NOT depend on domain plugins" (§13); "technology choices require ADRs and implementation RFCs" (§19).

**Proposed ADR constraints**, applicable only if and when the corresponding ADR is accepted: ADR-0001 — no direct Python imports inside the kernel, cross-language communication only through explicit contracts; ADR-0002 — a single canonical repository with preserved top-level boundaries; ADR-0003 — no arbitrary cross-module database access, domain plugins must not reference kernel internals.

**Repository-observed constraints**: at the time of writing, this `synapse-docs` repository contains the controlled documentation corpus (governance, standards, architecture, RFC, and ADR documents) and does not contain the `sdk/`, `kernel/`, `plugins/`, `modules/`, or `tests/` directories that RFC-0001 §28 and ADR-0002 describe as part of a single canonical monorepo. This is a factual observation about the current repository, not a statement about whether or when that structure will be realized, or whether it will be realized in this repository or a separate one.

## 18. Risks

The following risks are drawn from patterns identified during the evidence review that preceded this chapter, grounded in the reviewed sources:

- **Confusing proposed architecture with implemented reality.** Every technical decision described in this chapter is Draft or Proposed; no corresponding implementation exists. This is the risk this chapter has been written most carefully to avoid, and the reason for its repeated status attribution.
- **Premature distribution.** RFC-0001 §31 and ADR-0003 both explicitly warn against splitting components before scaling, isolation, or operational needs justify it.
- **Provider lock-in.** The explicit reason RFC-0014 and GOV-006's "AI Provider Strategy" exist as Draft direction is to prevent this outcome; it remains a risk until that direction is implemented and verified.
- **Architectural drift.** GOV-004 Principle 1 and RFC-0001 §19 both require technology and architecture decisions to be captured as ADRs/RFCs rather than embedded silently in code; this risk is the failure mode those rules exist to prevent.
- **Undocumented decisions.** The same rules above exist because decisions made outside the documented ADR/RFC process are, by definition, untraceable.
- **Unsafe autonomous execution.** GOV-010 names autonomous execution as a security-critical decision class (§4, Class C) requiring explicit risk review, but — per the evidence reviewed — concrete controls for it have not yet been defined anywhere in the corpus.
- **Insufficient isolation.** RFC-0010 §11 itself notes that in-process plugins are "effectively highly trusted code and require stronger review" — meaning the isolation boundary chosen for any given extension is itself a risk decision, not a solved problem.
- **Source-of-truth fragmentation.** STD-001 §9 establishes Markdown as this repository's canonical documentation format, with Word/PDF as derivative artifacts; this project has already had to resolve one live instance of two documents claiming the same identifier (STD-001 itself), which illustrates the risk concretely rather than hypothetically.

No numeric risk score is assigned; none of the sources reviewed for this chapter provide one specific to these items.

## 19. Open Questions

The following are open as of this writing and are not answered by this document:

- Whether ADR-0001 (primary implementation language) will be accepted, rejected, or revised.
- Whether ADR-0002 (repository strategy) will be accepted, rejected, or revised.
- Whether ADR-0003 (Phase 1 runtime architecture) will be accepted, rejected, or revised.
- What concrete controls govern autonomous execution (named as security-critical in GOV-010 but not yet defined).
- What formal memory-governance rules apply, beyond memory being named as a planned capability.
- What tool- and capability-governance rules apply beyond the brief risk mention in the governance material reviewed.
- Who holds named architecture-approval authority — no document reviewed names a specific individual or role beyond an undefined "Chief Architect" (GOV-003) and unnamed "Architecture Review" signoff lines (RFC and ADR Approval blocks).
- What performance targets the system must meet.
- What the final deployment topology will be, beyond the Phase 1 direction ADR-0003 proposes.
- How the current, documentation-only `synapse-docs` repository relates to the single canonical monorepo (including `sdk/`, `kernel/`, `plugins/`, `modules/`, `tests/`) that RFC-0001 §28 and ADR-0002 describe — whether that structure will be built inside this repository or a separate one has not been established by any source reviewed.

## 20. Future Evolution

Current Draft direction anticipates evolution along several lines, none of them committed to a timeline by any source reviewed: distribution of the scheduler, event bus, storage, and services away from the Phase 1 modular monolith once measured scaling, isolation, or operational needs justify it (RFC-0001 §18, §31); providers and models remaining replaceable as the model landscape changes (RFC-0014's stated goal, GOV-006's "AI Provider Strategy"); richer intelligence-plane capabilities such as reasoning, planning, and reflection modules building on top of the model-abstraction and execution foundations described here (RFC-0001 §8, §12); and governed autonomy — expanded autonomous execution capability that remains subject to the human-oversight and decision-authority framework GOV-010 describes, rather than autonomy without those controls. This document does not describe a timeline, a committed roadmap, or a guarantee that any of these directions will be pursued.

## 21. Architecture Decisions

The following ADRs are recorded here because they are directly relevant to this chapter's subject matter — not because they have been accepted:

| ADR | Title | Status |
|---|---|---|
| ADR-0001 | Primary Implementation Language | **Proposed** |
| ADR-0002 | Repository Strategy – Monorepo vs Multi-Repo | **Proposed** |
| ADR-0003 | Phase 1 Runtime Architecture – Modular Monolith | **Proposed** |

Inclusion in this list records relevance to the architecture this chapter introduces. It does not record or imply acceptance. Each ADR's own status field is the authoritative record of whether it has been accepted; as of this writing, all three remain Proposed.

## 22. References

| Document ID | Title | Status | Path |
|---|---|---|---|
| GOV-001 | Project Charter | Draft | `governance/GOV-001_Project_Charter_v0.1.docx` |
| GOV-003 | Governance Model | Draft | `governance/GOV-003_Governance_Model_v0.1.docx` |
| GOV-004 | Engineering Principles | Draft | `governance/GOV-004_Engineering_Principles_v0.1.docx` |
| GOV-006 | Technical Strategy | Draft | `governance/GOV-006_Technical_Strategy_v0.1.docx` |
| GOV-010 | Decision Framework | Draft | `governance/GOV-010_Decision_Framework_v0.1.docx` |
| STD-001 | Documentation Standards | Draft | `standards/STD-001-Documentation-Standards.md` |
| RFC-0001 | SynapseOS System Architecture | Draft | `rfcs/RFC-0001_SynapseOS_System_Architecture_v0.1.docx` |
| RFC-0002 | Kernel Architecture | Draft | `rfcs/RFC-0002_Kernel_Architecture_v0.1.docx` |
| RFC-0003 | Process & Execution Model | Draft | `rfcs/RFC-0003_Process_and_Execution_Model_v0.1.docx` |
| RFC-0010 | Plugin Runtime & Extension Model | Draft | `rfcs/RFC-0010_Plugin_Runtime_and_Extension_Model_v0.1.docx` |
| RFC-0014 | Model Abstraction Layer | Draft | `rfcs/RFC-0014_Model_Abstraction_Layer_v0.1.docx` |
| ADR-0001 | Primary Implementation Language | Proposed | `adrs/ADR-0001_Primary_Implementation_Language_v0.1.docx` |
| ADR-0002 | Repository Strategy – Monorepo vs Multi-Repo | Proposed | `adrs/ADR-0002_Repository_Strategy__Monorepo_vs_Multi-Repo_v0.1.docx` |
| ADR-0003 | Phase 1 Runtime Architecture – Modular Monolith | Proposed | `adrs/ADR-0003_Phase_1_Runtime_Architecture__Modular_Monolith_v0.1.docx` |

No external web references are used in this document.

## 23. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-10 | Denver Jacobs | Substantive rewrite of the initial ARCH-000 placeholder into a template-compliant Draft introduction, preserving document identity (renamed from `architecture/00-Introduction.md` to `architecture/ARCH-000-Introduction.md`). |

## 24. Approval Status

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-10 |
| Technical Review | TBD | Pending | |
| Approval Authority | TBD | Pending | |
