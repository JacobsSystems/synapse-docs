---
document_id: ADR-0018
title: StorageBackend Serialization Boundary
version: 0.3.0
status: Approved
decision_owners:
  - Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-30
last_updated: 2026-08-09
classification: Public
related_documents:
  governance:
    - GOV-014
    - GOV-015
  architecture:
    - ARCH-007
    - ARCH-011 (v0.1.3, Approved; FAA-011 obtained, see ARCH-011's own Approval Status table)
  adrs:
    - ADR-0016 (v0.5.0, Draft — not yet approved)
  engineering:
    - EWO-024 (Runtime Durable Storage Abstraction — filed, work-orders/EWO-024-Runtime-Durable-Storage-Abstraction.md, v0.3.0, Approved; FIA-024's genuine Founder decision is recorded within EWO-024's own Approval Status table, not as a separate filed artifact)
    - ER-024 (Independent Engineering Review of EWO-024 — Artifact-only; not independently filed)
    - EWO-024A (Engineering Amendment — Artifact-only; not independently filed)
    - ER-024 Re-Review (Artifact-only; not independently filed)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ADR-0018 — StorageBackend Serialization Boundary

> **Status notice:** This ADR is **Approved**. `FAA-013` = "APPROVE," Denver Jacobs, Founder, 2026-08-09. See §11 (Approval Status) for the full, verbatim Founder Declaration.

**Numbering — disclosed discrepancy.** The task that authorized this document labeled it "ADR-002." `synapse-docs/adrs/` was verified directly before drafting and found to contain `ADR-0001` through `ADR-0017` (four-digit sequence, per `STD-001` §7/§8's own `ADR-0001` example and filename pattern), with `ADR-0017-Bootstrap-Capability-Trust-Root.md` the highest-numbered existing document. "ADR-002" does not match this repository's own numbering scheme or its own next-available number. This document is therefore filed as **ADR-0018**, the correct next-available identifier under `STD-001`, on the same disclosed basis every prior numbering discrepancy in this engagement (`ARCH-010`→`ARCH-011`, `GOV-013`→`GOV-014`, "ER-023"→`ER-019`) has been handled: reproduced task text is honored where it constitutes required content, but a wrong label is never silently adopted as if it were correct.

## 1. Title

StorageBackend Serialization Boundary.

## 2. Status

**Approved.** `FAA-013` = "APPROVE," Denver Jacobs, Founder, 2026-08-09. See §11 (Approval Status) and the Revision History for the full lifecycle record.

## 3. Executive Summary

This ADR resolves exactly one architectural question, discovered as a disclosed scope decision during `EWO-024`: does `StorageBackend` (the seam `ARCH-011` §9 introduces beneath Persistence Service) consume an actor's `DomainState` directly, or does it consume an already-encoded persistence record (bytes)?

Repository evidence shows this is not, in fact, an open architectural choice between two live alternatives. `ARCH-011` §9 already states the backend "owns exactly, and only, raw byte-level durability for one opaque blob" and "stores and returns bytes" — never structured domain state. `ARCH-007` §10/§12 already assigns encoding and decoding exclusively to Persistence Service, not to anything beneath it. Read together, these already establish the contract this ADR is asked to decide. What is genuinely unresolved is not the architecture, but the fact that `EWO-024`'s actual, approved (`FIA-024`) implementation does not yet conform to it — its `StorageBackend` trait operates on `DomainState` directly, a disclosed, scope-limited simplification adopted only because encoding work was explicitly prohibited within that work order's own authorized scope.

**Decision: Option A.** `StorageBackend` MUST consume encoded persistence records (opaque bytes plus the envelope metadata `Persistence Service` alone owns), never `DomainState` directly. This is a clarification of already-approved architecture, not a new one. `EWO-024` remains valid on the disclosed basis it was reviewed and approved under; it is not retroactively invalidated by this ADR. `EWO-025` may not proceed unchanged if its own scope includes implementing a real, durable backend: the `StorageBackend` trait's signature must first change to accept encoded bytes, and Persistence Service must gain the encode/decode logic `ARCH-007` §12 already assigns it — genuine serialization work, requiring its own, separately authorized Engineering Work Order, not something this ADR itself performs or authorizes.

## 4. Repository Verification

| | `synapse-runtime` | `synapse-docs` |
|---|---|---|
| HEAD | `830d865085ebef434819f501886f9bc47a94b924` | `0fc690b7efa17ca39adacaad733c5e23b3528e85` |
| Branch | `main` | `main` |
| Sync with `origin/main` | 0 ahead / 0 behind | 0 ahead / 0 behind |
| Working tree | Clean — `EWO-023` and `EWO-024` (as amended by `EWO-024A`) both committed and pushed | 18 pre-existing, unrelated backlog entries; `ADR-0018` itself the only file this amendment modifies |

`ARCH-011` v0.1.3 is confirmed, by direct re-reading of its own frontmatter, to be **Approved** — `FAA-011` obtained and recorded in `ARCH-011`'s own Approval Status table, subsequent to this ADR's own initial 0.1.0 drafting. `EWO-024` (as amended by `EWO-024A`, and approved by `FIA-024`) is confirmed the current implementation baseline, now committed at `synapse-runtime` commit `830d865` and filed as `EWO-024` v0.3.0 (Approved) at `synapse-docs` commit `0fc690b` — no later engineering change to `services/persistence/` exists in `synapse-runtime`, confirmed directly by fresh source inspection, not assumed from this ADR's own earlier account of itself. Nothing was assumed from conversational summary; both repositories were verified fresh immediately before this amendment.

## 5. Current Architectural Evidence

**`ARCH-011` §9 (Storage Architecture), verbatim:** "The storage-backend abstraction owns exactly, and only, raw byte-level durability for one opaque blob at a time: write, read, and existence-check, against whatever storage technology it wraps. It owns nothing about envelopes, checksums, or version semantics." And, in the same section's own rationale for the nested boundary: "`Persistence Service` itself — not the backend — continues to own envelope construction, integrity-checksum computation and verification, and outer-version acceptability... The backend is never asked to understand what an envelope is; it stores and returns bytes."

**`ARCH-011` §13 (Storage Integrity):** "A checksum is computed over the encoded payload at write time" — presupposing an encoding step has already occurred, by the time anything reaches the checksummed/stored representation, above the backend boundary.

**`ARCH-007` §10 (Ownership Model), the single authoritative ownership table:** "Encoding a supplied domain-state representation into stored form; decoding stored data back into that representation; storage, retrieval, and durability mechanics" is owned by **Persistence Service**, not by any component beneath it. §12 restates this as a MUST: "Persistence Service... MUST perform the mechanical encoding of an actor-supplied domain-state representation into stored form, and the mechanical decoding of stored data back into that representation."

**`ARCH-011` §21 (Deferred Decisions):** "Concrete envelope byte layout, concrete Rust types, trait signatures, and method names — implementation-phase." This defers the exact spelling of the trait (method names, argument types' precise Rust form) to implementation. It does not defer *what crosses the boundary* — §9's "opaque blob," "stores and returns bytes" language is a substantive statement about the nature of that data, not a naming detail, and is not listed among §21's deferred items.

**`EWO-024`'s own disclosure**, read directly from `services/persistence/src/lib.rs`'s `StorageBackend` doc comment: "This trait operates on the existing, opaque `DomainState` representation directly, not on raw encoded bytes: converting `DomainState` into a byte-level wire format is genuine serialization work, explicitly outside this work order's own authorized scope." This is EWO-024's own contemporaneous acknowledgment that its implementation does not yet realize §9's own byte-level contract — presented, correctly, as a disclosed scope limitation, never as a claim that the architecture itself left this choice open.

**Conclusion: no genuine architectural ambiguity exists at the level of what the final contract should be.** `ARCH-011` §9 and `ARCH-007` §10/§12, read together, already and unambiguously specify Option A. The ambiguity this ADR resolves is administrative, not architectural: whether `EWO-024`'s disclosed, temporary deviation from that already-stated contract should now be formally reconciled, and what that means for engineering going forward. This ADR is the vehicle for exactly that reconciliation — a clarification, per its own stated Classification, not a new design.

## 6. Option A Assessment — Backend Receives Encoded Persistence Records

```text
Persistence Service
        │
Serializes DomainState
        ▼
StorageBackend
Receives encoded persistence records
```

- **Ownership.** Matches `ARCH-007` §10/§12 exactly: Persistence Service alone owns encoding/decoding. The backend owns none of it — consistent with `ARCH-011` §9's own "owns nothing about envelopes... stores and returns bytes."
- **Versioning.** `ARCH-011` §9/§12's envelope model (outer format-version identifier, integrity checksum, checksum-algorithm identifier) is only coherent as a wrapper around already-encoded bytes; §13's "checksum is computed over the encoded payload" presupposes exactly this ordering. Option A is the only option under which the envelope/payload split `ARCH-011` §9 establishes as its own governing principle actually means anything.
- **Portability.** Bytes are the only representation every future backend candidate (`ARCH-011` §14: SQLite, and by extension any future embeddable store) can actually store — none of them can persist an in-process `Rc<dyn Any>` (`DomainState`'s current Rust representation) across a process boundary or to disk. Option A is a precondition for any real, durable backend implementation to exist at all.
- **Backend simplicity.** The backend needs no knowledge of `DomainState`'s internal shape, no `Any`-downcasting, and no in-process coupling to actor-defined types — the narrowest possible interface, matching `EWO-024`'s own "dependency-minimal" implementation principle.
- **Testing.** A bytes-based backend can be tested against arbitrary byte fixtures, independent of any actor's own domain-state type — a genuinely backend-agnostic test surface, unlike the current `DomainState`-typed tests, which are inherently bound to in-process Rust values.
- **Architectural consistency.** Directly matches `ARCH-011` §9 and §13, and `ARCH-007` §10/§12, verbatim. No re-interpretation required.

## 7. Option B Assessment — Backend Receives DomainState, Serializes Internally

```text
Persistence Service
        ▼
StorageBackend
Receives DomainState
Serializes internally
```

- **Ownership.** Contradicts `ARCH-007` §10/§12's explicit, singular assignment of encoding/decoding to Persistence Service. Under Option B, that responsibility would be duplicated into every backend implementation — a second owner for a responsibility `ARCH-007`'s own ownership table states has exactly one.
- **Coupling.** Every future backend (SQLite, LMDB, or otherwise) would need to independently reimplement encoding, integrity-checksum computation, and envelope handling — precisely the failure mode `ARCH-011` §9's own rationale for the nested `StorageBackend` boundary already warns against one layer up ("each full Persistence Service replacement would need to reimplement envelope and integrity handling independently, with no architectural guarantee that two different implementations enforce the guarantees in §10/§13 identically"). The identical argument applies with equal force to backends under Option B.
- **Backend complexity.** Higher: the backend must either understand `DomainState`'s shape directly or be generic over serializable actor-defined types, contradicting `ARCH-011` §9's "opaque blob" characterization.
- **Portability.** `EWO-024`'s current implementation is, in effect, an even more limited form of Option B (the backend holds `DomainState` in-process, without even performing its own serialization) — and it is precisely this that makes it non-portable to any real storage technology, the concrete symptom that exposed this ADR's own question in the first place.
- **Testing.** Each backend's own private encoding logic would need separate test coverage, duplicating test surface across every backend rather than verifying it once, centrally, in Persistence Service.
- **Architectural consistency.** Contradicts `ARCH-011` §9's explicit "stores and returns bytes" language and `ARCH-007` §12's explicit, singular assignment of encoding/decoding to Persistence Service. **Rejected.**

## 8. Decision

**`StorageBackend` MUST consume encoded persistence records — an opaque byte-level payload, described by envelope metadata that Persistence Service alone owns — never `DomainState` directly.**

This restates, and makes unambiguous, what `ARCH-011` §9 and `ARCH-007` §10/§12 already establish; it introduces no new architectural concept and amends neither document. What it resolves is the administrative gap left open by `EWO-024`'s own disclosed, scope-limited deviation from that already-stated contract.

## 9. Rationale

Option A is the only option consistent with the ownership table `ARCH-007` §10 already states is "the single authoritative statement of ownership for this architecture" — encoding and decoding belong to Persistence Service, exclusively, a responsibility this ADR does not reassign, split, or duplicate. It is also the only option under which `ARCH-011` §9's envelope/payload separation, and §13's integrity-checksum guarantee ("computed over the encoded payload"), are coherent at all: both presuppose that whatever a backend stores is already-encoded, checksummed bytes, not a live, structured, in-process value. Finally, it is the only option that makes a genuinely portable, backend-independent implementation possible in the first place — no candidate backend named in `ARCH-011` §14 can durably store `DomainState`'s current Rust representation without an encoding step occurring first, above the backend boundary.

Option B was rejected because it does not describe a coherent alternative architecture; it describes duplicating a responsibility `ARCH-007` §10 already assigns to exactly one owner, for no offsetting benefit `ARCH-011` §9 identifies, and for at least one concrete cost (§9's own rationale) that document already names as the reason the nested `StorageBackend` boundary exists at all.

## 10. Engineering Impact

- **`EWO-024` remains valid.** It disclosed its own `DomainState`-based `StorageBackend` as a deliberate, temporary, scope-limited simplification, adopted only because encoding work was explicitly prohibited within its own authorized scope — never as a claim of final architectural conformance. `ER-024`, `EWO-024A`, `ER-024 Re-Review`, and `FIA-024` each reviewed and approved it on that same, explicit, disclosed basis. This ADR does not retroactively fault, invalidate, or reopen that chain.
- **No amendment to `EWO-024` itself is required by this ADR.** Nothing in this decision asks the existing implementation to be changed; it clarifies the target contract a *future* implementation must meet.
- **`EWO-025` may not proceed unchanged if its own scope includes implementing a real, durable backend** (`ARCH-011` §14: SQLite or any other candidate). No conformant real backend can be built beneath the current `StorageBackend` trait, because that trait does not yet accept encoded bytes. Before any such backend can be authorized, a separate, genuine serialization work order must first: (a) change `StorageBackend`'s signature to operate on encoded persistence records rather than `DomainState`; and (b) implement, in Persistence Service itself, the encode/decode logic `ARCH-007` §12 already assigns it. This ADR authorizes neither of those changes — consistent with its own stated purpose, it answers one architectural question and stops.
- **No additional architectural work is required beyond this ADR.** The underlying contract was already fully specified by `ARCH-011` §9 and `ARCH-007` §10/§12; this ADR closes the ambiguity `EWO-024`'s disclosed deviation exposed without amending either document.

## 11. Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs | Drafted | 2026-07-30 |
| Independent Architecture Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | Architectural substance (Option A, Decision, Rationale) confirmed sound; four citation/repository-verification findings identified (`ADR18-F01`–`F04`), none touching the architecture itself | 2026-08-08 |
| Repository Truth Amendment (v0.2.0) | Denver Jacobs (AI-assisted) | All four findings corrected — no architectural reasoning changed | 2026-08-08 |
| Final Independent Architecture Re-Review | Denver Jacobs (Founder; disclosed self-review) | All four corrections independently re-confirmed resolved; one new finding surfaced (`ADR18-F05`, stale `last_updated`) | 2026-08-08 |
| Metadata Amendment | Denver Jacobs (AI-assisted) | `last_updated` corrected; single-line diff verified | 2026-08-09 |
| Metadata Verification | Denver Jacobs (Founder; disclosed self-review) | Single-line diff independently re-confirmed; concluded `READY FOR FOUNDER ARCHITECTURE APPROVAL` — zero outstanding findings at any severity | 2026-08-09 |
| Approval Authority | Denver Jacobs, Founder, exercising Class B (Architecture) interim decision authority under `GOV-010` §5 in the absence of an appointed Chief Architect | **Approved** — `FAA-013` = "APPROVE" (verbatim Founder Declaration below) | 2026-08-09 |

**Founder Declaration (`FAA-013`), recorded verbatim, not paraphrased or regenerated:**

> "I, Denver Jacobs, acting as Founder of SynapseOS, have reviewed the completed ADR-0018 architecture lifecycle, including the architecture document, amendment history, independent reviews, metadata verification, repository evidence, and recommendations.
>
> I independently adopt the completed review chain's recommendation.
>
> I approve ADR-0018 as the authoritative architectural decision governing the Persistence Service ↔ StorageBackend serialization boundary.
>
> This decision authorizes Repository Filing, controlled commit, and publication through the established SynapseOS publication process.
>
> This decision does not itself commit, push, or publish any repository content."

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-30 | Denver Jacobs | Initial Draft. Resolves the `StorageBackend` serialization-boundary question exposed by `EWO-024`'s disclosed scope decision: whether the backend consumes `DomainState` directly or encoded persistence records. Determines that `ARCH-011` §9 and `ARCH-007` §10/§12 already establish encoded-bytes (Option A) as the architectural contract; `EWO-024` remains valid as a disclosed, temporary simplification; a future, separately authorized serialization work order is required before `EWO-025` may implement a real durable backend. No approval act has occurred. |
| 0.2.0 | 2026-08-08 | Denver Jacobs (AI-assisted) | Repository Truth Restoration amendment, correcting exactly the four findings an Independent Architecture Review identified — no architectural reasoning, Option A/B analysis, ownership, or Runtime responsibility changed. **ADR18-F01:** §4 Repository Verification cited stale HEADs (`a156972`/`8a91787`); refreshed to current (`830d865`/`0fc690b`), with working-tree state restated accurately. **ADR18-F02:** the `ADR-0016` citation incorrectly read "(Approved)"; `ADR-0016` is, in fact, `v0.5.0`, `status: Draft` — corrected to `ADR-0016 (v0.5.0, Draft — not yet approved)`, confirmed not relied upon anywhere in this ADR's own substantive reasoning (§3–§9). **ADR18-F03:** stale references refreshed — `ARCH-011` citation updated from "v0.1.1, Draft — Founder Architecture Approval Pending" to "v0.1.3, Approved; FAA-011 obtained" (approved later in this engagement, after this ADR's own 0.1.0 drafting); `EWO-024` citation updated from "not filed in synapse-docs; Artifact-only" to "filed... v0.3.0, Approved," with `FIA-024`'s genuine Founder decision noted as recorded within `EWO-024`'s own Approval Status table rather than as a separate filed artifact; the standalone `FIA-024` frontmatter line removed as redundant with that same note. **ADR18-F04:** §4's own body text previously read "`ARCH-011` v0.1.1 is confirmed... to remain the approved... architecture," directly contradicting the frontmatter's own "Draft — Approval Pending" annotation on the same document — both now consistently state `ARCH-011` v0.1.3, Approved. |
| 0.3.0 | 2026-08-09 | Denver Jacobs (Founder) | Repository Filing of `FAA-013` — **no architectural reasoning, ownership, responsibility, or Runtime semantics changed from 0.2.0**. (The intervening `last_updated` metadata correction, per its own task's explicit scope, did not itself advance the tracked `version` field — this entry is therefore 0.2.0 → 0.3.0, not 0.2.0 → 0.2.1 → 0.3.0; no `0.2.1` version was ever recorded in this Revision History.) Records the Founder's decision on the completed lifecycle (Independent Architecture Review → Repository Truth Amendment → Final Independent Architecture Re-Review → Metadata Amendment → Metadata Verification, concluding `READY FOR FOUNDER ARCHITECTURE APPROVAL` with zero outstanding findings at any severity): `status` transitions from `Draft` to **`Approved`**; the `> Status notice` callout and §2 (Status) are rewritten to reflect this; §11 (Approval Status) is expanded to record the full review chain and the verbatim `FAA-013` Founder Declaration. This version is reserved exclusively for recording that Founder disposition, mirroring the precedent already established by `ARCH-007`'s, `ARCH-011`'s, `EWO-016`'s, and `EWO-024`'s own dedicated-version-per-disposition convention — consistent with `STD-001` §31.5's own final clause, which permits (without requiring) tracked-metadata reconciliation to reflect operative approval as "a separate, later, distinct action" from the approval act itself; the two acts remain distinct here, exactly as §31.5 requires: `FAA-013` was obtained as its own genuine, prior act, and this version is solely the later, separate act of recording it. Not a MAJOR change under `STD-001` §13 (no material change to obligations, architectural intent, or compatibility — Option A, the Decision, and every citation are unchanged from 0.2.0) and not advanced to `1.0.0`, consistent with this engagement's own repeated precedent (`ARCH-011` remained at `v0.1.3` upon its own Founder approval) of not treating `STD-001` §13's "normally establishes 1.0.0" language as automatic. |
