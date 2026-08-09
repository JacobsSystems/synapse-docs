---
document_id: ARCH-012
title: Durable DomainState Encoding Architecture
project: SynapseOS
specification: SynapseOS — the architectural principles governing the transformation between actor-defined DomainState and the durable byte representation established by ADR-0018, completing the encoding-boundary question ARCH-007 §13.9 and ARCH-011 §21 each explicitly, repeatedly defer
version: 0.2.0
status: Approved
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.2 interim Chief Architect authority (GOV-010 §5, Class B) — no Chief Architect is currently appointed, the identical basis ARCH-007, ARCH-008, and ARCH-011 each already record for themselves.
created: 2026-08-09
last_updated: 2026-08-09
classification: Public
related_documents:
  architecture:
    - ARCH-007 (v0.5.2, Approved — Persistent Actor Architecture; §13.1–§13.9, the Durable-State Contract this document extends without amendment)
    - ARCH-011 (v0.1.3, Approved — Durable Storage Mechanics; §9, §12, §13, §21, the StorageBackend boundary this document's own scope sits directly above)
  adrs:
    - ADR-0018 (v0.3.0, Approved — StorageBackend Serialization Boundary; the byte-consumption decision this document's own scope begins from)
    - ADR-0019 (Runtime Contract Definition — cited only for Cluster/Placement's own Deferred status, §10 below; not reopened)
  research:
    - RSS-003 (Durable DomainState Encoding Research — the complete research basis for this document)
  consolidation:
    - ACR-003 (Durable DomainState Encoding Architecture Consolidation — the complete consolidation basis; every recommendation below traces directly to one of ACR-003's four resolved questions)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-012 — Durable DomainState Encoding Architecture

*Filename pattern: `ARCH-012-Durable-DomainState-Encoding-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 §33. AI output is not automatically authoritative.

**Numbering — disclosed discrepancy.** The task authoring this document labeled it "ARCH-013." `synapse-docs/architecture/` was verified directly before drafting and found to contain `ARCH-000` through `ARCH-011`, with `ARCH-011-Durable-Storage-Mechanics.md` the highest-numbered filed document — this engagement's own earlier "Runtime Contract Architecture" content, produced during the same session on an unrelated topic, was itself only ever delivered as a chat research artifact and was never filed, so it does not occupy `ARCH-012`. This document is therefore filed as **ARCH-012**, the correct next-available identifier, on the same disclosed basis every prior numbering discrepancy in this engagement (`ARCH-010`→`ARCH-011`, `GOV-013`→`GOV-016`, "ER-023"→`ER-019`, "ADR-002"→`ADR-0018`) has been handled.

## Executive Summary

This document defines the architectural principles governing the transformation between actor-defined `DomainState` and the durable byte representation `ADR-0018` already requires `StorageBackend` to consume. It settles nothing `ARCH-007` §13 or `ARCH-011` §9–§13 already settled — ownership, contract association, type identity, determinism, and checksum ownership are cited, never redefined. It resolves exactly the four questions `ACR-003` consolidated: a recommended (non-mandatory) encoding pattern, one host-independence clarification extending an already-Approved prohibition, an explicit restatement of already-settled failure semantics, and an explicit deferral of everything touching distributed execution. No new Runtime service, registry, coordinator, or Persistence/StorageBackend responsibility is introduced.

## 1. Document Control and Status

| Field | Value |
|---|---|
| Document ID | ARCH-012 |
| Title | Durable DomainState Encoding Architecture |
| Version | 0.2.0 |
| Status | **Approved** — Founder Architecture Approval (`FAA-014`) recorded 2026-08-09, with one non-blocking observation formally noted (Approval Status, below); this document is the authoritative SynapseOS Durable DomainState Encoding Architecture |
| Author | Claude (AI-assisted, under Denver Jacobs' direction) |
| Approval authority | Denver Jacobs, Founder, exercising the interim Class B default (GOV-003 §3.2), identical basis to `ARCH-007`, `ARCH-008`, and `ARCH-011`'s own approval authority |
| Created | 2026-08-09 |
| Classification | Public |

This document introduces no implementation and authorizes none directly; it establishes architecture only, to be realized by a future, separately authorized Engineering Work Order, exactly as `ARCH-007` §1 and `ARCH-011` §1 already state for themselves.

## 2. Architectural Context

`ARCH-007` §13.1–§13.9 (v0.5.2, Approved) established the Durable-State Contract, its Runtime-held association, its five-way ownership split, the Durable-State Kind Identifier, actor-owned representation versioning, behavioral (not byte-exact) determinism, and six new failure categories. `ARCH-011` §9–§13 (v0.1.3, Approved) established the envelope/payload separation, atomicity guarantees, checksum requirement, and backward-compatibility SHOULD. `ADR-0018` (v0.3.0, Approved) fixed that `StorageBackend` consumes encoded bytes, never `DomainState` directly. `RSS-003` researched what remained genuinely open after those three; `ACR-003` consolidated that research into four resolvable questions. This document is the architectural resolution of those four questions, and nothing else.

## 3. Scope

**In scope:** the architectural principles governing the transformation from actor-produced `DomainState` to the durable byte representation `ADR-0018` requires — beginning after an actor has produced `DomainState` (via its own Durable-State Contract's encode operation, `ARCH-007` §13.2) and ending before those bytes reach `StorageBackend` (`ADR-0018`'s own boundary).

**Out of scope:** everything `ARCH-007`/`ARCH-011`/`ADR-0018` already settle (§2, above); everything `ACR-003` recommended deferring (§8, below); any concrete Rust type, trait signature, serialization library, or wire format (`ARCH-007` §13.9, `ARCH-011` §21 — this document extends neither deferral).

## 4. Architectural Principles

This document adds exactly three things to already-settled architecture, each minimal and directly traceable to `ACR-003`: a recommended default pattern (§5), one clarifying invariant (§6), and an explicit deferral list with reasoning (§8). Sections 6–7 restate, by citation, what is already decided, so that a reader of this document alone has the complete picture without needing to cross-reference three prior documents for context — restatement here creates no new authority and amends neither `ARCH-007` nor `ARCH-011`.

## 5. Recommended Encoding Model

**Fixed-schema, manual versioning is the recommended default pattern for a Durable-State Contract's own encode/decode operations** (`ARCH-007` §13.2). This is guidance, not a mandate — no architecture document requires, or should require, one universal encoding model across every actor, and this document does not introduce such a requirement.

**Rationale (`ACR-003` §5, Q1):** a self-describing payload's principal advantage — tolerating structural change without an explicit branch — is substantially neutralized by `ARCH-007` §13.7's already-Approved rule that decode "MUST fail explicitly... never substitute a plausible-looking approximation." Under that constraint, an actor's own decode logic must validate and fail explicitly regardless of format, so fixed-schema manual versioning delivers equivalent safety at lower implementation complexity, and is directly precedented by mature systems whose evolution discipline most resembles SynapseOS's own actor-owned model (event-sourcing "upcasting," Erlang `code_change`, Temporal version markers — `RSS-003` §4).

An actor's author remains free to choose a different pattern where its own domain justifies it — this recommendation constrains no Durable-State Contract's own implementation.

## 6. Host Independence

**Architectural invariant:** a Durable-State Contract's own durable representation MUST NOT depend on host-specific representation assumptions — including, non-exhaustively, native endianness, in-memory layout, pointer values, platform-specific integer representation, or implementation-defined text encoding.

This is additive precision on `ARCH-007` §13.2's own existing prohibition ("MUST NOT depend on any process-local, non-durable value — a memory address, an `Rc` pointer, an in-process `TypeId`, or any other value meaningless outside the process and allocation that produced it"), not a new architectural concept — the identical relationship `ARCH-011` §9's own Durable-State Kind Identifier addition already had to `ARCH-007` §13.5 ("additive precision, not a new design"). A process-local value and a host-local representation assumption are the same underlying failure mode (durability breaking outside the specific context that produced it) at two different scopes; `ARCH-007` §13.2 addressed the narrower scope, this document names the wider one it always implied.

**The concrete mechanism remains deferred** — which byte-order convention, which text encoding — exactly matching `ARCH-011` §21's precedent for the identical class of decision ("concrete envelope byte layout, concrete Rust types, trait signatures, and method names — implementation-phase").

## 7. Actor Responsibilities and Failure Semantics

Restated by citation, not redefined: an actor-authored Durable-State Contract remains solely responsible for encoding, decoding, representation evolution (schema versioning), and domain-meaning validation of its own `DomainState` (`ARCH-007` §13.2, §13.4, §13.6). A Durable-State Contract's decode operation MUST either faithfully reconstruct behaviorally-equivalent `DomainState`, or fail explicitly — silent approximation is prohibited without exception (`ARCH-007` §13.7). This document adds no new actor responsibility, removes none, and does not extend or alter the failure-semantics rule; §5's recommendation is directly motivated by it.

## 8. Deferred Architecture

Each named explicitly, with reasoning, per `ACR-003` §5 (Q3, Q4):

- **Forward compatibility between Runtime versions** — deferred. Relevant only once more than one Runtime version can be live simultaneously, a capability that does not currently exist and is not authorized.
- **Rolling upgrades** — deferred, for the identical reason.
- **Cluster migration** — deferred. Depends on the Cluster/Placement Runtime Contract, itself still Deferred (`ADR-0019`, `ACR-002`).
- **Distributed placement** — deferred, for the identical reason.
- **Cross-node negotiation** — deferred, for the identical reason; presupposes more than one node exists, which no current SynapseOS architecture provides for.
- **Wire protocols** — deferred. A wire protocol presupposes a concrete encoding format, itself explicitly deferred by `ARCH-007` §13.9 and `ARCH-011` §21 and unaltered by this document.

This document remains **intentionally independent** of Cluster/Placement — it adds no compatibility "hooks" for a not-yet-designed contract (`ACR-003` §5, Q4). Nothing in §5/§6 above precludes a future cluster capability from interpreting durable records across nodes; no explicit accommodation is needed for that to remain true.

## 9. Architecture Risks

| Risk | Mitigation |
|---|---|
| §5's recommendation being read as a mandate, foreclosing an actor author's own justified choice of a different pattern | §5's own explicit "guidance, not a mandate" statement; no enforcement mechanism is introduced anywhere in this document |
| §6's host-independence invariant being mistaken for a concrete format decision | §6's own explicit deferral of the concrete mechanism to implementation, unaltered from `ARCH-011` §21's precedent |
| §8's deferrals being read as this document overlooking distributed concerns rather than deliberately excluding them | Each deferral is named individually with its own reasoning, not silently omitted |

## 10. Architectural Invariants

Numbered independently within this document (not appended to `ARCH-007`'s own 1–21, since this is a separate document, not an amendment):

1. A Durable-State Contract's durable representation MUST NOT depend on host-specific representation assumptions (endianness, memory layout, pointer values, platform integer representation, implementation-defined text encoding) — extending `ARCH-007` §13.2's process-local-value prohibition to host-local scope (§6).
2. No encoding model is architecturally mandated; `ARCH-007` §13.2's actor-authorship freedom is unaltered (§5).
3. Decode failure semantics remain exactly as `ARCH-007` §13.7 states, unaltered by this document (§7).

## 11. Consequences

No existing ownership, Runtime responsibility, Persistence Service responsibility, or `StorageBackend` responsibility changes. A future Engineering Work Order implementing a real Durable-State Contract now has an explicit, minimal architectural basis for treating host-portability as a genuine requirement rather than an implementation afterthought (§6), and explicit guidance, not ambiguity, on which encoding pattern to reach for by default (§5). No future work is authorized or implied for forward compatibility, rolling upgrades, cluster migration, distributed placement, cross-node negotiation, or wire protocols (§8) — each remains its own, separately authorized future architectural effort.

## 12. References

`ARCH-007` (v0.5.2, Approved) §13.1–§13.9. `ARCH-011` (v0.1.3, Approved) §9, §12, §13, §21. `ADR-0018` (v0.3.0, Approved). `RSS-003` (Research Synthesis). `ACR-003` (Architecture Consolidation Review). `ADR-0019` / `ACR-002` / `RSS-002` (cited only for Cluster/Placement's own Deferred status, §8 — not reopened).

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-09 | Denver Jacobs (AI-assisted) | Initial Draft. Consolidates `RSS-003` and `ACR-003` into four resolved architectural questions: recommended (non-mandatory) fixed-schema encoding pattern; one host-independence invariant extending `ARCH-007` §13.2 without amendment; restated (not redefined) actor ownership and failure semantics; and an explicit deferral of forward compatibility, rolling upgrades, cluster migration, distributed placement, cross-node negotiation, and wire protocols pending future, separately-authorized distributed-Runtime architecture. No ownership, Runtime responsibility, or Persistence/StorageBackend boundary changed. No approval act has occurred. |
| 0.2.0 | 2026-08-09 | Denver Jacobs (Founder) | Repository Filing of `FAA-014` — **no architectural reasoning, ownership, responsibility, or Runtime semantics changed from 0.1.0**. Records the Founder's decision on the completed Independent Architecture Review (concluding `INDEPENDENT ARCHITECTURE REVIEW COMPLETE — ARCH-012 VERIFIED`, zero Critical/Major findings, one non-blocking observation `ARCH12-F01` on document-organization/instrument choice): `status` transitions from `Draft` to **`Approved`**. This is the first controlled filing of this document — it was drafted, reviewed, and approved entirely as a chat-delivered research/architecture artifact before this version; no prior version of this file was ever committed, staged, or otherwise checkpointed in `synapse-docs`. The Approval Status table is completed, recording the full verbatim Founder Declaration and `FAA-014-OBS-01`, mirroring the precedent already established by `ARCH-007`'s, `ARCH-011`'s, `EWO-016`'s, and `EWO-024`'s own dedicated-version-per-disposition convention. Not advanced to `1.0.0`, consistent with this engagement's own repeated precedent (`ARCH-011` remained at `v0.1.3` upon its own Founder approval) of not treating `STD-001` §13's "normally establishes 1.0.0" language as automatic. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-08-09 |
| Reviewer | Independent Architecture Review | `INDEPENDENT ARCHITECTURE REVIEW COMPLETE — ARCH-012 VERIFIED` — zero Critical, zero Major findings; one non-blocking observation (`ARCH12-F01`, document-organization/instrument choice, tempered by `GOV-016` §5 — the source of the relevant test — itself remaining unapproved) | 2026-08-09 |
| Approval Authority | Denver Jacobs, Founder, exercising the interim Class B default (GOV-003 §3.2) | **Approved, with non-blocking observation** — Founder Architecture Approval recorded as `FAA-014`. Denver Jacobs' own decision, recorded verbatim: "I, Denver Jacobs, acting as Founder of SynapseOS, have reviewed the completed ARCH-012 architectural record, including the architecture document, repository evidence, constitutional references, Independent Architecture Review, findings, and recommendation. I independently adopt the review's conclusion that ARCH-012 is architecturally sound and ready for approval. I approve ARCH-012 as the authoritative architecture governing the transformation between actor-defined DomainState and the durable byte representation established by ADR-0018. This approval establishes: fixed-schema, manual versioning as the recommended default architectural pattern, while preserving actor autonomy to adopt alternative approaches where justified; the architectural invariant that durable representations must be independent of host-specific memory representations; continued actor ownership of encoding, decoding, validation, and representation evolution through the Durable-State Contract; deterministic reconstruction requirements through existing constitutional authority; explicit deferral of forward compatibility, rolling upgrades, cluster placement, distributed negotiation, and wire protocols until future constitutional authority exists. This approval does not authorize Runtime implementation changes. This approval authorizes Repository Filing, controlled commit, and publication through the established SynapseOS publication process." | 2026-08-09 |

**`FAA-014-OBS-01`**, recorded formally, not requiring amendment: "While ARCH-012 introduces additive precision rather than a fundamentally new architectural subsystem, I approve its publication as a standalone architecture document. Maintaining a dedicated architecture document improves discoverability, establishes a natural location for future architectural evolution, and avoids continued expansion of ARCH-007. This observation is informational only. It requires no amendment and does not affect this approval."

This document is now **Approved**, following Founder Architecture Approval (`FAA-014`) recorded directly above, on the identical "ordinary, mutable Approval Status convention" this repository's most-recently-approved documents use throughout (demonstrated: `ARCH-007`, `ARCH-008`, `ARCH-011`, each populated in place, without requiring a version-number change to `1.0.0` as a precondition). `ARCH-012` is hereby adopted as the authoritative Durable DomainState Encoding Architecture for SynapseOS: future implementation shall conform to it; future architectural modification shall occur only through the established governance process; no implementation may intentionally diverge from it without an approved ADR or a subsequent architecture amendment. The one non-blocking observation recorded above remains an open, tracked matter — it neither blocks this approval nor requires resolution before implementation may proceed.

---

```
ARCH-012 v0.2.0 APPROVED

FOUNDER ARCHITECTURE APPROVAL RECORDED (FAA-014)

DURABLE DOMAINSTATE ENCODING ARCHITECTURE ADOPTED

READY FOR REPOSITORY FILING
```
