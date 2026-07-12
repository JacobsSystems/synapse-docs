---
document_id: ADR-0017
title: Bootstrap Capability Trust Root
version: 0.1.0
status: Draft
decision_owners:
  - Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment). Also touches Class C (Security/Critical) concerns — trust-boundary definition — per GOV-010 §4; no dedicated security-approval role yet exists, so Class B approval authority governs until one is established, disclosed here per GOV-010 §5's Class C provision.
created: 2026-07-12
last_updated: 2026-07-12
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-001
    - ARCH-002
  adrs:
    - ADR-0011
    - ADR-0012 (Approved)
    - ADR-0013
    - ADR-0015 (Approved)
    - ADR-0016 (Approved)
  engineering:
    - EWO-004 (Capability Authority, not yet formally issued as an EWO document at time of writing)
    - SRP-004 Independent Engineering Review (the discovery record for this ADR, synapse-runtime repository)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ADR-0017 — Bootstrap Capability Trust Root

> **Status notice:** This ADR is **Draft**. No approval act has occurred.

## 1. Title

Bootstrap Capability Trust Root.

## 2. Status

Draft. Not yet approved. See §11 (Approval Status placeholder) and the Revision History.

## 3. Context

ARCH-001 §5.2 defines Capability as requiring that "authority exists only through explicit capability derivation, traceable through an unbroken, non-widening chain to a single bootstrap root." ARCH-001 §6 restates this as a constitutional law: "Authority exists only through explicit capability derivation, traceable to a single bootstrap root through an unbroken, non-widening delegation chain."

Both statements establish that a single bootstrap root must exist. Neither states how it comes to exist, nor how it is distinguished from any other capability once it does. ARCH-002 §11 (Constitutional Execution Cycle) lists "Runtime initialization" as step 1 — "exactly one, disclosed, one-time bootstrap act" — and, separately, "Capability issuance/delegation" as step 4, assigned to Capability Authority, requiring "non-amplification, ceiling, lineage recorded." Step 4 describes ordinary issuance; it does not say whether the bootstrap root is created through step 4 or through step 1, and does not say whether step 4 requires its own issuer argument to itself already be trusted.

This gap surfaced during independent engineering review of SRP-004 (Capability Authority, `synapse-runtime` repository). The implementation, following the architecture as written, allowed `issue` and `attenuate` to accept any caller-supplied capability as an issuer or source without requiring that capability to already be validated. The review found this was not a coding defect: nothing in currently-approved architecture requires issuer or source validation, and nothing in currently-approved architecture explains how the very first capability could ever be issued if such a requirement existed unconditionally. Without resolving which capability is exempt from that requirement, and why, engineering cannot close the gap without inventing architecture. The review further found that, absent this resolution, non-amplification is not actually enforced — any caller can supply a fabricated capability as an issuer and mint a fully valid child from it — and that nothing prevents capability provenance from forming a cycle, since a capability's fitness to serve as an issuer is never itself checked against anything.

## 4. Decision

Exactly one Bootstrap Capability is created, once, as part of the Runtime's own one-time bootstrap act (ARCH-002 §11, step 1). It is not created through ordinary capability issuance (step 4) or attenuation (step 16), and it is never exposed through any public Runtime interface — no Runtime operation, at any time after bootstrap, creates or re-creates it. It is the single bootstrap root ARCH-001 §5.2 and §6 already require to exist; every other capability's provenance ultimately traces to it, through an unbroken, non-widening chain.

Ordinary capability issuance requires its issuer to already be validated. Attenuation requires its source to already be validated. The Bootstrap Capability is the sole exception to this requirement, precisely because it has no issuer to validate against — it is the terminating case, not an instance of the general rule.

Capability provenance is therefore a rooted, directed acyclic graph, by construction: every non-root capability's existence presupposes an already-valid parent, recursively, and that recursion has exactly one base case.

## 5. Rationale

**Why a distinguished, non-ordinary bootstrap act, rather than treating the first issuance as ordinary.** ARCH-002 §11 step 1 already establishes that Runtime initialization is "exactly one, disclosed, one-time bootstrap act" — a category of act already recognized as structurally prior to, and outside, the constitutional execution cycle's ordinary, repeatable steps (ARCH-001 §6: "A fixed, named set of foundational Runtime mechanisms — capability enforcement, audit emission, scheduling, time observation, transport, and bootstrap — are necessarily outside the Actor model, because each is either structurally prior to actor execution or required to remain unbypassable by it"). Locating the Bootstrap Capability's creation here is not a new category of act; it is recognizing that capability derivation's own root belongs to a category ARCH-001 already named, but had not yet connected to Capability.

**Why issuer/source validation must be mandatory for every other case.** ARCH-001 §6 already states capability derivation "may attenuate authority but never amplify it," recursively bounded by "a capability-issuing capability's own constraint set." That bound is meaningless unless the issuing capability's own standing is itself established — an unvalidated issuer supplies no genuine ceiling, only whatever shape the caller chooses to give it. Requiring validation is not a new constraint invented here; it is what "a capability-issuing capability's own constraint set defines the maximum authority anything it mints may carry" already presupposes, made explicit.

**Why exactly one root, never more.** ARCH-001 §5.2 already says "a single bootstrap root," not "one or more" or "one per some partition." This decision does not weaken or reinterpret that word; it operationalizes it.

**Why the DAG guarantee follows without inventing a new mechanism.** Once every derivation (other than the one root) requires its issuer or source to already be valid, an inductive argument — not a new enforcement mechanism — establishes acyclicity: a capability can only be used to derive another once it has itself passed validation, which in turn required its own parent to have already passed validation, and so on; this chain cannot revisit a capability not yet validated, and therefore cannot close a loop. The guarantee is a consequence of the validation requirement, not a separate invariant requiring its own implementation.

## 6. Consequences

- Capability Authority's `issue` and `attenuate` operations must validate their issuer/source argument before minting a derived capability — a required behavioural correction to the current SRP-004 implementation, to be carried out as separate, later engineering work (EMO or equivalent), not performed by this ADR.
- Some mechanism must exist by which the Runtime's bootstrap sequence causes the one Bootstrap Capability to come into existence, distinct from the ordinary `issue` path — the concrete form of that mechanism is an implementation decision for that same later engineering work, not fixed here.
- Capability provenance becomes provably a rooted DAG once the above is implemented; the cycle risk the SRP-004 review identified is closed at the architectural level by this decision, though it remains open in the current implementation until the corresponding engineering correction lands.
- No Trusted Core component changes. No new component is introduced. No responsibility moves between components: Capability Authority still owns capability lifecycle and current validity in full; Runtime still owns only construction, sequencing, and bootstrap orchestration, exactly as ARCH-002 §3 already states.
- No Runtime sequencing changes: this decision clarifies what already happens once, during the already-defined bootstrap step; it does not add a new step to the Constitutional Execution Cycle or reorder any existing one.

## 7. Alternatives Considered

**A — No distinction; every issuance, including the first, accepts an unvalidated issuer.** This is the status quo the SRP-004 review examined. Rejected: it makes the non-amplification law unenforceable, since any caller can fabricate an issuer with arbitrary authority, and it provides no basis for the DAG guarantee ARCH-001's "single bootstrap root" language requires.

**B — Multiple, independent bootstrap roots (for example, one per Runtime instance or one per some other partition).** Rejected: ARCH-001 §5.2 already says "a single bootstrap root," not a bounded or unbounded plurality of roots. Adopting multiple roots would not be a clarification of existing text; it would contradict it, requiring a constitutional change under ARCH-001 §10's Change Control, which this ADR does not undertake.

**C — Require re-verification of the entire lineage chain at every use, rather than requiring only that the immediate issuer/source already be validated.** Rejected as unnecessary and as reaching into implementation: the inductive argument in §5 already establishes the DAG and non-amplification guarantees from immediate-parent validation alone; requiring full-chain re-verification at every step would be prescribing a concrete verification mechanism, which is Capability Authority's own implementation concern (ARCH-002 §9 already states this mechanism is deferred), not an architectural decision.

## 8. Conformance

A conforming Runtime:

- creates exactly one Bootstrap Capability, once, during its one-time bootstrap act, and never again;
- never exposes Bootstrap Capability creation through any public, repeatable Runtime interface;
- rejects any ordinary issuance or attenuation whose issuer or source has not itself already been validated;
- can, for any capability it currently considers valid, demonstrate that its provenance traces, through validated intermediates, to the one Bootstrap Capability.

A conforming Runtime is not required to implement any particular data structure, cryptographic scheme, or verification algorithm to satisfy the above — those remain deferred, per ARCH-002 §9.

## 9. Relationship to Existing Architecture

This ADR clarifies ARCH-001 §5.2 and §6; it does not redefine either. It introduces no new constitutional concept, no new Trusted Core component, and no new Runtime responsibility category. It does not amend ARCH-002: ARCH-002 §11 step 1 ("exactly one, disclosed, one-time bootstrap act") and step 4 ("Capability issuance/delegation... non-amplification, ceiling, lineage recorded") are silent on this specific point, not contradictory to it — step 4's own requirement of "non-amplification" and "ceiling" already presupposes an issuer with real standing to bound against, which only an already-validated issuer can supply, so step 4, properly read, was already implicitly describing ordinary issuance, with the bootstrap case implicitly belonging to step 1's already-distinct category. Per ARCH-001 §6, "The Runtime realizes the constitutional architecture; it does not redefine it" — ARCH-002 inherits this clarification without requiring its own edit.

## 10. Engineering Impact (Informational Only)

This ADR authorizes no engineering work. The corrections to `synapse-runtime`'s Capability Authority implementation this decision implies — issuer/source validation in `issue`/`attenuate`, and a bootstrap-time mechanism for creating the one Bootstrap Capability — remain subject to STD-001's ordinary EWO/EMO process, applied against this ADR and the corresponding ARCH-001 amendment once approved.

## 11. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs | Drafted | 2026-07-12 |
| Technical Review | TBD | Pending | |
| Approval Authority | TBD | Pending | |

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-12 | Denver Jacobs | Initial Draft, resolving the bootstrap-trust-root ambiguity discovered during SRP-004 (Capability Authority) independent engineering review. Establishes that exactly one Bootstrap Capability is created once, during Runtime's already-defined one-time bootstrap act, never through ordinary issuance and never exposed through any public interface; that ordinary issuance and attenuation require an already-validated issuer or source, with the Bootstrap Capability as the sole, structurally necessary exception; and that capability provenance is therefore a rooted, acyclic structure by construction. No approval act has occurred. |
