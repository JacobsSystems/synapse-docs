---
document_id: ADR-0020
title: Disposition of the Unrecoverable ARCH-013 Draft
version: 0.1.0
status: Approved
decision_owners:
  - Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-08-10
last_updated: 2026-08-10
classification: Public
related_documents:
  governance:
    - GOV-013 (Draft — Engineering Lifecycle; §6.4-§6.8, the Design Exploration / Design Approval Review / Architecture Authoring lifecycle this decision preserves unmodified)
    - GOV-018 (v0.2.0, Approved — SynapseOS Platform Vision and Constitution)
    - ACT-005 (v0.1.0, Approved — Developer Platform Era)
  architecture:
    - ARCH-014 (v0.7.0, Approved — Synapse SDK Architecture; the sole Approved artifact currently carrying certain ARCH-013-derived principles, cited throughout this decision; not modified by it)
  standards:
    - STD-001 (Draft; §5 Controlled Document Families — ADR fits precisely as "Records of significant architectural decisions"; §12 Document Status Lifecycle — a genuine vocabulary gap identified, not resolved, by this decision, §6 below)
  roadmap:
    - ROAD-001 (v0.1.0, Approved — SynapseOS Platform Strategy and Roadmap)
  research:
    - DES-002 (v0.2.1, Draft, Founder-accepted, filed — Documentation Platform and SDK Documentation: Design Exploration; the dependent workstream this decision's own §7 pause directly affects)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ADR-0020 — Disposition of the Unrecoverable ARCH-013 Draft

> **Numbering Note.** `ADR-0019` is already reserved, cited three times in the Approved `ARCH-012` ("Runtime Contract Definition," tied to the still-Deferred Cluster/Placement architecture) — verified directly before drafting, not assumed available. This decision is filed as `ADR-0020`, the next identifier confirmed genuinely clean.

## 1. Context

`ARCH-013 — Developer Platform Architecture` existed as chat-delivered Draft material (v0.2.0) at an earlier stage of this engagement. It was never filed to `synapse-docs` and never received Founder Architecture Approval. Its own Independent Architecture Re-Review left one finding, F05, unresolved. `ARCH-014` (Synapse SDK Architecture, Approved v0.7.0) cites `ARCH-013` extensively as its own direct constitutional predecessor, for specific, quoted principles (Runtime Boundary §7, SDK/Developer-Platform ownership boundary §8, CLI boundary §9, the Hello World/constitutional acceptance test §11, the developer-mental-model error-reporting principle §12, and the Public/Internal API distinction §13 — the last of these already refined by `ARCH-014` §8 into its own four-tier model).

An exhaustive recovery search — `synapse-docs` full git history (pickaxe search across every commit ever made), reflog, stash, all branches, `synapse-runtime`, the filesystem, and this engagement's own persistent memory system — found **no original `ARCH-013` file anywhere, at any point.** Only the fragments `ARCH-014` and `EWO-026.1` quote from it, and downstream summaries derived from those same fragments, survive. Reconstructing a complete document from these fragments would require new architectural judgment, not recovery, and has not been attempted.

## 2. Decision

1. `ARCH-013` is recorded as having historically existed as an unfiled Draft (v0.2.0, "Developer Platform Architecture"); its complete original text is confirmed unrecoverable.
2. `ARCH-013`'s own identifier remains permanently, exclusively associated with this history and will never be reassigned to a different document (`STD-001` §7).
3. No missing `ARCH-013` content will be fabricated. No inferred reconstruction will be represented as original `ARCH-013` text.
4. No `STD-001` §12 status value is force-fitted to `ARCH-013` — none of the eight existing terms (Draft, Review, Approved, Implemented, Superseded, Deprecated, Archived, Rejected) accurately describes a document that existed, was never filed, and is now unrecoverable (§6, below).
5. Finding F05 remains permanently, historically unresolved: severity Minor, character disclosure-only, exact content unrecoverable. It is not inferred, not administratively closed, and not transferred to any future document's own review record.
6. Every `ARCH-013`-derived principle currently relied upon by any Approved artifact derives its operative authority solely from that Approved artifact (`ARCH-014`), never independently from `ARCH-013` itself (§4, below).
7. A narrower replacement Developer Platform Architecture — scoped to the specific cross-cutting principles known to have originated in `ARCH-013` that remain unowned by any current Approved artifact (the CLI boundary, the general form of Progressive Disclosure, the general form of the SDK/CLI/platform generalization test) — may be prepared later, under a freshly, independently verified identifier, through the unmodified `GOV-013` lifecycle. It is explicitly not authorized to become an umbrella architecture covering every `ACT-005` Developer Platform domain.
8. `ARCH-014` remains Approved and is not substantively amended by this decision. A narrow provenance amendment — recording only that `ARCH-013`'s source is now confirmed unrecoverable and that `ARCH-014`'s own reliance on it now rests on `ARCH-014`'s own Approved status directly — is authorized in direction only, as its own future, separately governed amendment engagement.
9. Documentation Platform + SDK Documentation Architecture Authoring (`DES-002` v0.2.1, Founder-accepted, filed) remains paused pending the narrower replacement architecture identified in item 7 — an architectural dependency (specifically, `DES-002`'s own `R02` CLI-independence and `R21` universal concept-tiering requirements), not a scheduling preference.

## 3. Rationale

Reconstructing `ARCH-013` from its own surviving fragments alone would require inventing content for every section not directly quoted — a different act from recovery, and one this decision declines to perform without further, explicit Founder direction. Treating the fragments as sufficient to proceed with new, dependent architecture work (the Documentation Platform + SDK Documentation Architecture) risks exactly the redundancy or silent contradiction this engagement's own discipline exists to prevent — the specific trigger for this decision.

## 4. Surviving Principle Authority

| Principle | Carrying Approved Artifact | Status |
|---|---|---|
| Runtime Boundary | `ARCH-014` §7 | Explicit, fully carried |
| SDK/Developer-Platform ownership boundary | `ARCH-014` §8 | Explicit, fully carried for SDK |
| CLI architectural boundary | None | **Orphaned** — no current Approved artifact carries this |
| Hello World Principle / constitutional acceptance test | `ARCH-014` §11 | Explicit, carried for SDK's own instantiation |
| Developer-mental-model error-reporting principle | `ARCH-014` §12 | Explicit, carried for SDK errors |
| Public/Internal API / `TrustedCore` precedent | `ARCH-014` §8 | Explicit, already superseded in practical effect by `ARCH-014`'s own four-tier model — no further replacement work needed |
| Runtime Authority (general, cross-domain form) | `ARCH-014` (SDK-specific form only) | Partially orphaned |
| Progressive Disclosure (general, cross-domain form) | None | **Orphaned** |
| SDK/CLI generalization test (general, cross-domain form) | `ARCH-014` (SDK half only) | Partially orphaned |

No principle above is granted independent authority by its appearance in this table — each derives authority only from the Approved artifact named, or is explicitly recorded as currently unowned.

## 5. Replacement Architecture Direction

Recommended scope, not authorized for authoring by this decision: the orphaned cross-cutting principles in §4 (CLI boundary, general Progressive Disclosure, general generalization test) — narrower than a full Developer Platform umbrella, sized to what `ARCH-013` is actually known to have established and what remains genuinely unowned. Lifecycle entry point (direct Architecture Authoring vs. a preceding Design Exploration) and the eventual identifier are both determined at the time that work is separately authorized, verified fresh against the repository as it exists then — `ARCH-015` was confirmed evidence-supported as next-available during this decision's own preparation, but is not allocated or reserved here.

## 6. Disclosed Governance Gap

`STD-001` §12's Document Status Lifecycle has no term accurately describing "existed, never filed, now unrecoverable." This is recorded as a candidate for a future, separate `STD-001` amendment — not created, not required, and not blocking this decision.

## 7. Consequences

Documentation Platform + SDK Documentation Architecture Authoring remains paused. `ARCH-014` requires no substantive change. A future, separate engagement may pursue the `ARCH-014` provenance amendment (§2 item 8); a further future, separate engagement may pursue the replacement architecture (§2 item 7, §5). Neither is begun by this decision.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-10 | Denver Jacobs (AI-assisted) | Initial decision record, drafted following exhaustive recovery search confirming `ARCH-013`'s complete text unrecoverable. Records historical disposition, F05 treatment, surviving-principle authority attribution, replacement-architecture direction, and the Documentation Platform Architecture pause. Founder-approved in substance prior to this filing. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-08-10 |
| Approval Authority | Denver Jacobs, Founder (interim, per `GOV-003` §3.2 vacancy) | **Approved** | 2026-08-10 |

**Founder Approval, recorded as declared:** *"Approved in substance. The proposed ADR-0019 [renumbered ADR-0020 per the Founder's own subsequent identifier-conflict resolution] — Disposition of the Unrecoverable ARCH-013 Draft — is accepted as the correct architectural decision mechanism for recording the historical and architectural consequences of ARCH-013's loss."* Full declaration, covering every item in §2 above, preserved in this engagement's own conversational record. No retroactive approval of `ARCH-013` itself is granted by this Approval.
