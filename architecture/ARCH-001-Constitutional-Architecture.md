---
document_id: ARCH-001
title: Constitutional Architecture
project: SynapseOS
specification: SynapseOS — constitutional architecture governing all future ARCH documents
version: 0.1.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-11
last_updated: 2026-07-11
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
  standards:
    - STD-001 (Approved)
  architecture:
    - ARCH-000 (Draft — historical record, unmodified; identifier permanently bound per STD-001 §7)
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

# ARCH-001 — Constitutional Architecture

*Filename pattern: `ARCH-001-Constitutional-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Purpose

This document defines the constitutional architecture of SynapseOS: the fixed set of architectural concepts and laws that every subsystem, every future ARCH document, and every implementation must preserve. Implementation is expected to change; the constitutional architecture is not, except through demonstrated architectural contradiction or significant architectural improvement — never through novelty alone.

## 2. Scope

This document governs: the constitutional concepts (Actor, Capability, Message, Execution Semantics); the constitutional laws binding them; the layering separating constitutional, Runtime, and Infrastructure responsibility; the boundary between the SynapseOS Runtime and the underlying host platform; and the mechanism-versus-policy separation as it applies at the constitutional level.

This document does not govern: implementation algorithms, specific runtime technologies, or subsystem-level design (scheduler internals, storage engine design, transport protocols) — these belong to ARCH-002 and subsequent ARCH documents; the historical reasoning behind SynapseOS's architectural identity — this belongs to ADR-0013; and governance authority, review, or approval process — these belong to GOV-003, GOV-010, and STD-001.

## 3. Relationship to ARCH-000 and ADR-0013

This document does not erase, rename, invalidate, or rewrite `ARCH-000-Introduction.md`. That document's identifier is permanently bound to it, per STD-001 §7, and it remains, unmodified, as historical architectural evidence of the project's earlier application-platform-era thinking.

This document supersedes ARCH-000 only in role — as the current normative constitutional reference — precisely where ADR-0013 has already recorded that ARCH-000's earlier architectural summaries (its §7–§13) no longer serve that role. This document does not itself perform that supersession; it reflects a determination ADR-0013 already made.

This document becomes the normative foundation inherited by ARCH-002 and all subsequent ARCH documents.

## 4. SynapseOS Identity

SynapseOS is an Intelligence Operating System: it provides operating-system-level abstractions, mechanisms, and services for intelligent systems, operating above conventional operating systems and applying operating-system architectural principles to intelligent computation rather than directly managing hardware resources. The historical reasoning behind this identity is recorded in ADR-0013 and is not repeated here.

## 5. Constitutional Concepts

Four concepts are constitutional. Each owns exactly one responsibility, is definable without reference to implementation, and must not be redefined by any subsequent ARCH document.

### 5.1 Actor

**What it is:** the unit of execution.

**What it owns:** its own private state, its own behavior, and its own mailbox.

**What it must never become:** an authority holder in its own right; a container of shared, externally mutable state; a substitute for Capability. Actor identity grants no authority (§6). Actor state may be observed or modified only through messages the actor itself chooses to process.

### 5.2 Capability

**What it is:** the unit of authority — a structured, unforgeable, immutable object binding a target, a bounded set of permitted operations, a constraint set, a provenance record, and a revocation handle.

**What it owns:** its own authority structure, and nothing else.

**What it must never become:** a role, a permission string, an access-control-list entry, an API key, a bearer token conveying unscoped authority, a bare actor address, a service identifier, a policy decision, or an identity claim. Authority exists only through explicit capability derivation, traceable through an unbroken, non-widening chain to a single bootstrap root.

### 5.3 Message

**What it is:** the unit of interaction — an immutable, uniquely identified, strongly typed request for work, addressed to an actor, optionally carrying a capability reference for delegation.

**What it owns:** its own content, identity, causation, and correlation.

**What it must never become:** a source of authority in itself. A message requests work; it never confers authority. The authorization to send a given message type, and any capability the message carries for the recipient's use, are distinct concerns and must not be conflated.

### 5.4 Execution Semantics

**What it is:** the constitutional rule-set governing when execution begins, ends, suspends, and resumes; what determinism is required; and what must remain observable throughout.

**What it owns:** the temporal and behavioral rules of execution.

**What it must never become:** a runtime data structure. Execution Context — the record of an actor's identity, active capability bindings, and causation metadata at a given moment — is not a constitutional concept. It is the Runtime-layer realization of Execution Semantics, owning nothing independently of the constitutional concepts it composes.

## 6. Constitutional Laws

- Actor identity grants no authority.
- Actor state is private; it may be observed or modified only through messages the actor itself chooses to process.
- Authority exists only through explicit capability derivation, traceable to a single bootstrap root through an unbroken, non-widening delegation chain.
- Messages request work but never confer authority.
- Capabilities are immutable once issued; attenuation always produces a new, distinct capability, never a modification of an existing one.
- Capability derivation may attenuate authority but never amplify it. This applies recursively: a capability-issuing capability's own constraint set defines the maximum authority anything it mints may carry.
- Every execution has exactly one owning actor.
- Execution follows the constitutional Execution Semantics: suspension does not extend the validity of held capabilities; resumption revalidates authority rather than assuming its continuation.
- Messages are immutable once created and carry a unique, immutable identity.
- The Runtime realizes the constitutional architecture; it does not redefine it.
- A fixed, named set of foundational Runtime mechanisms — capability enforcement, audit emission, scheduling, time observation, transport, and bootstrap — are necessarily outside the Actor model, because each is either structurally prior to actor execution or required to remain unbypassable by it. Runtime services that are not Actors must never perform application execution.

## 7. Architectural Layering

**Constitutional Layer** — Actor, Capability, Message, Execution Semantics. Fixed. Defines what must remain true regardless of implementation.

**Runtime Layer** — realizes the constitutional layer, in two parts. A fixed set of foundational, privileged mechanisms sits outside the Actor model by structural necessity: scheduling, time observation, capability enforcement (non-forgery, integrity, enforcement at invocation, revocation-state enforcement), audit emission, transport, and the one-time bootstrap act. Everything else that realizes constitutional behavior — Execution Context, lifecycle mechanics, isolation mechanics, resource accounting, and all application-level services (knowledge and storage mediators, provider adapters, network-endpoint coordinators, audit processing, security-response logic) — is ordinary Actor World, built from the constitutional primitives, not exempt from them.

**Infrastructure Layer** — the underlying conventional operating system and hosting substrate SynapseOS runs above. Illustrative, not exhaustive or closed.

## 8. Runtime Boundary

The SynapseOS Runtime is responsible for: logical actor scheduling and dispatch, capability enforcement, message delivery semantics, audit emission, the bootstrap act, and realization of Execution Semantics.

The underlying host platform remains responsible for: physical process and thread scheduling on real CPUs, virtual memory management, physical device drivers, the physical network stack beneath the message-transport abstraction, and durable storage primitives that storage-mediating actors use rather than reimplement. Host-level process supervision (restarting a crashed SynapseOS Runtime process) is the host platform's responsibility; the Runtime cannot supervise its own crash, for the same structural reason no Runtime mechanism can be scheduled by itself.

SynapseOS does not reimplement host-level hardware management. The host platform is infrastructure; it holds no special status, privilege, or capability within the constitutional authority model.

## 9. Mechanism versus Policy

The trusted core enforces exactly four mechanisms: capability non-forgery, capability integrity (covering target binding, constraint integrity, and provenance lineage as one guarantee), enforcement at invocation, and revocation-state enforcement. Everything else — who should receive authority, risk thresholds, provider-selection rules, budget-allocation strategy, scheduling strategy, audit storage and retention policy — is replaceable policy.

The governing test: does implementing a given concern outside the trusted core still allow the core's guarantees to hold? Does it enforce one non-negotiable invariant, or choose among several valid strategies? Enforcement of an invariant is mechanism; selection among valid strategies is policy.

## 10. Change Control

The constitutional architecture defined in §5 and §6 is frozen. Constitutional concepts and laws evolve only through demonstrated architectural contradiction or significant architectural improvement. Architectural novelty is never sufficient justification for constitutional change.

No future ARCH document may redefine a constitutional concept or law. A future ARCH document that appears to require such a redefinition has surfaced a potential architectural contradiction, not license for a local override — that contradiction must be raised and resolved through the ADR process, in the same manner ADR-0013 resolved the contradiction that led to this document.

## 11. Relationship to Future ARCH Documents

ARCH-002 (Runtime Architecture) and all subsequent ARCH documents inherit this constitutional architecture without exception. They refine Runtime-layer and Infrastructure-layer design — the mechanics of scheduling, isolation, persistence, transport, and subsystem behavior. They do not redefine Actor, Capability, Message, or Execution Semantics, and they do not restate the constitutional laws in §6; they cite this document instead.
