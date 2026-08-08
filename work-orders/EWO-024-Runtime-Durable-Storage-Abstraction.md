---
document_id: EWO-024
title: "Runtime Durable Storage Abstraction"
version: 0.3.0
status: Approved
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-08-08
last_updated: 2026-08-08
classification: Public
related_documents:
  standards:
    - STD-001 (§46, Engineering Work Orders)
  architecture:
    - ARCH-011 (v0.1.3, Approved; architecture/ARCH-011-Durable-Storage-Mechanics.md) — §8 (Ownership Model: storage-backend selection and substitution assigned to embedding code at Runtime composition time), §9 (Storage Architecture: the nested `StorageBackend` seam this EWO realizes — "owns exactly, and only, raw byte-level durability for one opaque blob... stores and returns bytes")
    - ARCH-007 (v0.5.2, Approved — architecture/ARCH-007-Persistent-Actor-Architecture.md) — §8, §10, §12, §16, §20 (Persistence Service's existing mechanics-only boundary, unmodified by this EWO); §13.3 (this EWO's own `PersistenceHandle::with_backend` substitution point is the mechanism ARCH-007 v0.5.2 §13.3 later builds on for Durable-State Contract resolution)
    - ARCH-002 (Runtime Architecture) — §6, §22 (the general replaceable-service substitution pattern this EWO applies to Persistence Service's own internal backend, identical in kind to Scheduler/Supervisor/Temporal Runtime)
  adrs:
    - ADR-0016 (Draft — not yet approved) — Rule 2 (reachability constraint: a storage backend is reached only through Persistence Service, itself reached only through Runtime — never directly, never through the Effect-dispatch pipeline)
    - ADR-0018 (v0.1.0, Draft — not yet approved) — resolves the `StorageBackend` serialization-boundary question this EWO's own implementation exposes and discloses (§7, below); does not amend or invalidate this EWO
  predecessor: None — independent of EWO-023 (Effect Idempotency Metadata), a contemporaneous but unrelated milestone
  reported_by: "The next sequentially available Engineering Report identifier per STD-001 §7 at the time this EWO is reported on — not assumed in advance"
  base_state:
    runtime_head: d541f434a203d4c011c7ffa5ae3776f9b03e7d87
    docs_head: 0d839cf95b6b5e8bbb204836fc532063653ce610
---

# EWO-024 — Runtime Durable Storage Abstraction

Registered per STD-001 §46 (Engineering Work Orders). This authorizes implementation work only. It does not itself constitute approval, and does not amend ARCH-011, ARCH-007, or any other architecture, standards, or governance document.

> **Reconstruction Notice.** Unlike every prior EWO in this repository, this document is authored *after* its implementation already exists in the `synapse-runtime` working tree, uncommitted, at base state `d541f43`. It is a retroactive reconstruction of engineering authorization from direct repository evidence (the working-tree diff and its own doc comments), not a prospective authorization of not-yet-started work. Every requirement stated below is verified as already satisfied by the existing implementation; none is invented. At v0.1.0, this document did not itself constitute Independent Engineering Review, Engineering Amendment, Founder Implementation Approval, or Repository Filing of the implementation — each was, at that time, a separate, later lifecycle stage. Each has since occurred and is recorded in this document's own later revisions: Engineering Review and Amendment at v0.2.0; Founder Implementation Approval and this Repository Filing at v0.3.0 (§13).

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-024 |
| Title | Runtime Durable Storage Abstraction |
| Version | 0.3.0 |
| Status | **Approved** (STD-001 §12 — Founder Implementation Approval recorded; `FIA-024` = APPROVE) |
| Author | Denver Jacobs (AI-assisted) |
| Created | 2026-08-08 |
| Architecture authority | ARCH-011 (Approved, v0.1.3) §8, §9; ARCH-007 (Approved) §8, §10, §12, §16, §20; ARCH-002 §6, §22 |
| Dependencies | None outstanding |
| Implementation repository | `synapse-runtime` |
| Implementation location | `services/persistence/src/lib.rs`, `services/persistence/src/internal.rs` (working tree, uncommitted at authoring time) |
| Review requirement | Independent Engineering Review required before Founder Approval, per this corpus's own established EWO lifecycle |

---

## 1. Purpose

`ARCH-011` §9 requires Persistence Service's internal storage mechanics to sit behind a swappable `StorageBackend` abstraction, substitutable only at Runtime composition time by embedding code (`ARCH-011` §8), on the identical basis every other replaceable Runtime service already uses (`ARCH-002` §6/§22). Before this EWO, `services/persistence`'s own `PersistenceImpl` held its `HashMap<ActorId, DomainState>` store directly, with no such seam — `ARCH-011` §9's own requirement was architecturally approved but not yet realized in code.

This EWO introduces exactly that seam: a `StorageBackend` trait, an `InMemoryStorageBackend` realizing it (the crate's own pre-existing in-memory behavior, unchanged), and a `PersistenceHandle::with_backend` constructor through which embedding code may supply an alternative backend. It does not implement a real, durable storage technology of any kind.

---

## 2. Background

**Prior state** (`services/persistence/src/internal.rs`, pre-EWO-024): `PersistenceImpl` owned a `HashMap<ActorId, DomainState>` directly and implemented the public `Persistence` trait (`checkpoint`/`retrieve`/`delete`) against it inline. No substitution point existed; the in-memory store was the only possible backend, hardcoded.

**Governing architecture**: `ARCH-011` §9, added on that document's own Independent Architecture Review (Finding 4), establishes a nested boundary distinct from `ARCH-007` §22's own coarser, whole-Persistence-Service-level replaceability: "the storage-backend abstraction owns exactly, and only, raw byte-level durability for one opaque blob at a time: write, read, and existence-check... it stores and returns bytes." `ARCH-011` §8 assigns backend selection and substitution to embedding code at Runtime composition time.

**What this EWO's own implementation actually does, verified directly against the working tree**: introduces the seam `ARCH-011` §9 requires, but operates the seam on the existing, already-opaque `DomainState` representation directly, not on encoded bytes. This is a disclosed, scope-limited simplification — genuine serialization (converting `DomainState` to a byte-level wire format) is explicitly out of this EWO's own authorized scope (§6, below) — not a claim that `ARCH-011` §9's byte-level contract is already fully realized. §7 states this disclosure in full.

---

## 3. Architectural Authority

**Why a `StorageBackend` trait, not a generic parameter or enum.** `ARCH-011` §9 itself specifies a trait-shaped seam ("the storage-backend abstraction"), substitutable at Runtime composition time — a `Box<dyn StorageBackend>` field on `PersistenceImpl` is the direct, minimal realization of that requirement, mirroring how Scheduler, Supervisor, and Temporal Runtime are already substitutable (`ARCH-002` §6).

**Why `InMemoryStorageBackend` is the trait's sole implementation.** The crate's own pre-existing in-memory behavior is not removed or redesigned — it is re-expressed as the trait's first, and currently only, implementation, with `PersistenceHandle::new()`/`Default` continuing to construct it exactly as before. `ARCH-011` §9's own text explicitly reserves multiple future backends (SQLite named directly, §14) without requiring this EWO to build more than one.

**Why `PersistenceHandle::with_backend` is additive, not a replacement for `new()`.** `ARCH-011` §8 assigns backend substitution to embedding code, not to Persistence Service's own internal default. Adding a second constructor, rather than parameterizing the existing one, preserves every existing call site (`PersistenceHandle::new()`, `Default::default()`) with zero required change — the narrowest public surface satisfying `ADR-0016`'s requirement that Persistence Service's real behavior be invocable by the Runtime, with no additional public type, factory, or generic abstraction introduced beyond it.

**Why the backend is invoked exclusively by `Persistence Service`'s own logic.** `ADR-0016` Rule 2 and `ARCH-011` §9's own "Interaction with Persistence Service" clause require the backend to be unreachable except through `PersistenceImpl` itself — confirmed directly: no file outside `services/persistence/` references `StorageBackend` or `with_backend` anywhere in the current workspace (verified by workspace-wide `grep`).

**Disclosed non-conformance to `ARCH-011` §9's byte-level contract.** `ARCH-011` §9 states a backend "stores and returns bytes," never structured domain state. This EWO's own `StorageBackend` trait operates on `DomainState` directly. This is named explicitly, not concealed, in this EWO's own §6 (Non-Goals) and §7 (Disclosed Deviation) — resolving it is `ADR-0018`'s own subject, itself still Draft and unapproved, and is not resolved by this EWO.

---

## 4. Objectives

1. Introduce the `StorageBackend` trait `ARCH-011` §9 requires, as Persistence Service's own internal substitution seam.
2. Preserve the crate's existing in-memory behavior exactly, re-expressed as that trait's sole implementation (`InMemoryStorageBackend`).
3. Introduce `PersistenceHandle::with_backend`, the Runtime-composition-time substitution point `ARCH-011` §8 assigns to embedding code — purely additive; `PersistenceHandle::new()`/`Default` unaffected.
4. Preserve the existing public `Persistence` trait (`checkpoint`/`retrieve`/`delete`) unchanged — `PersistenceImpl` delegates to the backend internally; no external caller observes any difference in that trait's own shape.
5. Introduce no new external dependency, no real durable storage technology, and no encoding/serialization mechanism.

---

## 5. Non-Goals

This EWO MUST NOT, and its existing implementation does not:

- implement a real, durable storage backend of any kind (SQLite or otherwise — `ARCH-011` §14, a future, separately authorized milestone);
- implement `DomainState`-to-bytes serialization, or change `StorageBackend` to operate on encoded bytes rather than `DomainState` directly — genuine serialization work, explicitly out of scope (§7);
- implement the envelope (version tag, Durable-State Kind Identifier, integrity checksum) `ARCH-011` §9/§13 assigns to Persistence Service itself;
- wire `PersistenceHandle::with_backend` into `RuntimeCore`'s own composition (confirmed: no caller outside `services/persistence/` exists yet) — a distinct, future milestone;
- change the public `Persistence` trait's own signature;
- change any Effect Coordinator, Capability Authority, Audit, Scheduler, or Provider component (confirmed: each carries a zero diff against this EWO's own base state);
- claim cross-restart identifier or state uniqueness of any kind.

---

## 6. Required Data Model

Realized exactly as follows (`services/persistence/src/lib.rs`, `services/persistence/src/internal.rs`):

```text
pub trait StorageBackend {
    fn put(&mut self, actor: &ActorId, state: DomainState) -> Result<(), RuntimeError>;
    fn get(&self, actor: &ActorId) -> Result<DomainState, RuntimeError>;
    fn remove(&mut self, actor: &ActorId) -> Result<(), RuntimeError>;
    fn contains(&self, actor: &ActorId) -> bool;
    fn known_actors(&self) -> Vec<ActorId>;
}

pub(crate) struct InMemoryStorageBackend {
    store: HashMap<ActorId, DomainState>,
}

pub struct PersistenceHandle(internal::PersistenceImpl);

impl PersistenceHandle {
    pub fn new() -> Self { /* unchanged: in-memory backend */ }
    pub fn with_backend(backend: Box<dyn StorageBackend>) -> Self { /* new */ }
}

pub(crate) struct PersistenceImpl {
    backend: Box<dyn StorageBackend>,
}
```

`known_actors` realizes the mechanics-only, backend-level startup-discovery capability `ARCH-011` §11 requires ("`Persistence Service` MUST be able to enumerate the set of `ActorId`s for which a stored representation currently exists"). Its own existence does not trigger, imply, or perform restoration — `ARCH-011` §11's restoration-*policy* question (always explicit, never automatic) is unaffected, and no caller within this EWO's own scope invokes it for that purpose.

---

## 7. Disclosed Deviation — `DomainState`, Not Bytes

`ARCH-011` §9 states a storage backend "stores and returns bytes," never structured domain state. This EWO's `StorageBackend` trait operates directly on the existing, opaque `DomainState` representation instead. This is a deliberate, disclosed, scope-limited simplification — converting `DomainState` to a byte-level wire format is genuine serialization work, and encoding was explicitly excluded from this EWO's own authorized scope (§5). It is not a claim that `ARCH-011` §9's byte-level contract is already satisfied.

`ADR-0018` (Draft, unapproved) independently confirms: `ARCH-011` §9 and `ARCH-007` §10/§12, read together, already establish that a backend must consume encoded bytes (its own "Option A"); this EWO's `DomainState`-based implementation remains valid as a disclosed, temporary simplification under that ADR's own conclusion; no amendment to this EWO is required by it; and any future work implementing a real, durable backend (`ARCH-011` §14) may not proceed unchanged under the current `StorageBackend` signature — a separately authorized serialization work order must first change it to accept encoded bytes. This EWO does not perform, and does not authorize, that future change.

---

## 8. Required Compatibility Behaviour

Verified directly against the working tree:

- `PersistenceHandle::new()` and `Default::default()` continue to construct a handle backed by `InMemoryStorageBackend`, behaviorally identical to every version of this crate before this EWO.
- The public `Persistence` trait's own signature (`checkpoint`/`retrieve`/`delete`) is unchanged; `PersistenceImpl` now delegates each method to its held `Box<dyn StorageBackend>` instead of a direct `HashMap`, with no externally observable difference.
- No `Cargo.toml`/`Cargo.lock` change — confirmed by empty diff.
- No file outside `services/persistence/` is touched by this EWO's own implementation.

---

## 9. Required Implementation Scope

Authorized files, exactly (confirmed as the complete diff against this EWO's own base state):

```text
services/persistence/src/lib.rs
services/persistence/src/internal.rs
```

This EWO does NOT authorize changes to any other crate. `ARCH-007` v0.5.2 §13.3's own future use of `PersistenceHandle::with_backend` for Durable-State Contract resolution (`runtime/src/lib.rs`'s `RuntimeCore`) is not implemented by this EWO and requires its own, separately authorized work.

---

## 10. Required Test Coverage — This EWO's Own Scope

The following, present in the current working tree, constitute this EWO's own base test coverage:

**`internal.rs`** — `InMemoryStorageBackend`: default-constructible; `put`-then-`get` round-trips; `get` fails for an unstored actor; `put` overwrites a prior representation; `remove` removes a stored representation and fails for an unstored actor; `contains` reflects presence/absence truthfully; `known_actors` reports exactly the currently-stored set; distinct actors have independent storage. `PersistenceImpl`: `default()` is backed by the in-memory backend and behaves as before; `with_backend` delegates every operation to the supplied backend.

**`lib.rs`** — `StorageBackend` is object-safe (`&dyn StorageBackend`); a handle is constructible via `with_backend`; a handle constructed via `with_backend` (with a fresh, empty backend) behaves identically to `new()` across checkpoint/retrieve/delete.

> **Disclosed exclusion.** Two further tests already present in the working tree — `a_handle_constructed_with_a_pre_populated_backend_observes_its_existing_state` and `a_handle_constructed_with_a_distinguishable_backend_genuinely_delegates_every_operation` (the latter via a `RecordingStorageBackend` test double) — are explicitly labeled in their own code comments as **EWO-024A**, resolving a Major finding from an Independent Engineering Review (**ER-024**) of this EWO's own original test coverage: the tests above construct a fresh, empty backend on every path and therefore cannot distinguish genuine backend substitution from a silently substituted default. Those two tests, the finding they resolve, and the review that found it are **not** part of this document's own authorized scope — they belong to `EWO-024A`, a separate amendment document this task does not author (see Repository Discipline). They are named here only so this EWO's own scope is not confused with work already layered on top of it in the same working tree.

---

## 11. Risks

| Risk | Mitigation / Required Test |
|---|---|
| A future backend substitution being silently ignored in favor of the default | Identified as this EWO's own coverage gap (§10, disclosed exclusion) — resolved by `EWO-024A`, not by this document |
| `StorageBackend` operating on `DomainState` rather than encoded bytes being mistaken for full `ARCH-011` §9 conformance | §7's explicit disclosure; `ADR-0018` as the dedicated resolution vehicle |
| Premature Runtime-composition wiring (`RuntimeCore` calling `with_backend`) being introduced under this EWO's own authority | §5's explicit non-goal; confirmed by workspace-wide `grep` that no such caller currently exists |
| Backend reachable other than through `Persistence Service` | `ADR-0016` Rule 2; `ARCH-011` §9's own "Interaction with Persistence Service" clause; no other crate references `StorageBackend` (confirmed) |

---

## 12. Reporting Requirement

An Engineering Report SHALL be authored recording: objective; implementation summary; both files modified; the exact `StorageBackend` seam realized, cross-referenced against `ARCH-011` §8/§9; the disclosed `DomainState`-vs-bytes deviation (§7) and its relationship to `ADR-0018`; compatibility confirmation (§8); every test from §10, with outcome; full verification results (`cargo fmt`, `cargo clippy -D warnings`, `cargo build`, `cargo test`, all `--workspace --all-targets --all-features`); a scope audit confirming no non-goal (§5) was implemented and no unauthorized file (§9) was touched; and an explicit statement of readiness for Independent Engineering Review. Its own document identifier SHALL be derived from repository evidence at authoring time (STD-001 §7), not assumed in advance.

---

## 13. Disposition

**Approved.** Independently reviewed (`ER-024`): engineering substance confirmed accurate, all quality gates clean; two Minor findings identified, both in this document's own citations, none in the Runtime implementation. Corrected by Engineering Amendment (v0.2.0). Independently re-reviewed (`ER-024` Re-Review): both findings confirmed resolved from source, zero regression, zero new defect, concluding `READY FOR FOUNDER IMPLEMENTATION APPROVAL`.

**Founder Implementation Approval granted.** `FIA-024` = **APPROVE**, Denver Jacobs, Founder, 2026-08-08, recorded verbatim:

> "I, Denver Jacobs, acting as Founder of SynapseOS, have reviewed the complete EWO-024 implementation lifecycle, including the Engineering Work Order, Engineering Review, Engineering Amendment, Engineering Re-Review, repository evidence, and implementation status. I adopt the recommendation of the completed engineering review chain as my own Founder decision. I approve EWO-024 as the authoritative SynapseOS implementation for the StorageBackend abstraction and PersistenceHandle composition-time backend injection. This decision authorizes repository filing, controlled commit, and publication through the established SynapseOS engineering publication process. This decision does not itself commit, push, or publish repository content."

The implementation this EWO authorizes already exists, complete and passing all current engineering quality gates (independently re-verified 2026-08-08: `cargo test --workspace --all-targets --all-features` — 977 passed, 0 failed; `cargo clippy --workspace --all-targets --all-features -- -D warnings` — clean), in the `synapse-runtime` working tree at base state `d541f43`, **still uncommitted** — this Founder Approval authorizes, but does not itself perform, the commit, push, or publication of that implementation, nor of this document. Each remains a distinct, separately required future act, consistent with `FIA-024`'s own explicit final sentence.

**EWO-024 is Approved.** No further engineering, review, or amendment is authorized under it; its remaining lifecycle steps are controlled commit and push, of both this document and the `synapse-runtime` implementation it authorizes.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-08 | Denver Jacobs (AI-assisted) | Initial Draft. Reconstructed from direct repository evidence — the `services/persistence/src/{lib,internal}.rs` working-tree diff against base state `d541f43`, that diff's own doc comments (which already, contemporaneously disclose the `StorageBackend`/`EWO-024` relationship and the `DomainState`-vs-bytes scope limitation), `ARCH-011` §8/§9, `ARCH-007` (Approved, v0.5.2), `ARCH-002` §6/§22, `ADR-0016` Rule 2, and `ADR-0018` (Draft) — following the Engineering Lifecycle Reconstruction of 2026-08-08, which found the implementation complete, unauthorized by any filed instrument, and not invalidated by any later repository change. No engineering work performed by this document; it authorizes, retroactively, exactly what the existing implementation already does, and discloses, exactly as the implementation itself already does, what remains deferred (§10's excluded tests, belonging to a future `EWO-024A`; real serialization and a real backend, belonging to future, separately authorized milestones). |
| 0.2.0 | 2026-08-08 | Denver Jacobs (AI-assisted) | Engineering Amendment, correcting exactly the two Minor findings an Independent Engineering Review (`ER-024`) identified — no engineering scope, requirement, implementation description, or architectural claim changed beyond these two citations. **ER-024-F01:** the related-documents citation of `ADR-0016` incorrectly read "(Approved)"; `ADR-0016` is, in fact, `v0.5.0`, `status: Draft` — "No approval act has occurred" at every revision through 0.5.0. Corrected to `ADR-0016 (Draft — not yet approved)` (frontmatter). **ER-024-F02:** §9 (Required Implementation Scope) cited `ARCH-007` v0.5.2 §17 for the Durable-State Contract association `PersistenceHandle::with_backend` supports — §17 is, in fact, "Deletion and Retention Architecture," unrelated content; the document's own frontmatter already, correctly cited §13.3 ("Contract Association — Runtime-Held, Composition-Time Supply, Not a New Registry") for the identical claim. Corrected §9's body citation from §17 to §13.3, resolving the internal self-contradiction. `ER-024`'s one Observation (`ER-024-OBS-01`, no explicit Definition-of-Done/Acceptance-Criteria section) is explicitly not addressed by this amendment, per `ER-024`'s own classification of it as non-blocking and this amendment's own narrow authorization. |
| 0.3.0 | 2026-08-08 | Denver Jacobs (Founder) | Governance disposition recorded — **no engineering-scope, requirement, or exclusion changed from 0.2.0**. Records the Founder's decision on the completed Independent Engineering Re-Review of `ER-024`'s Engineering Amendment (concluding `INDEPENDENT ENGINEERING RE-REVIEW COMPLETE — EWO-024 VERIFIED — NO ENGINEERING REGRESSION DETECTED — READY FOR FOUNDER IMPLEMENTATION APPROVAL`, both Minor findings independently re-confirmed resolved, zero new defect): `status` transitions from `Draft` to **`Approved`**. §13's Disposition section is rewritten to record the full review/amendment/re-review chain and quote `FIA-024`'s own Founder Declaration verbatim. The Approval Status table is completed (Independent Engineering Review and Approval Authority rows recorded), mirroring the precedent already established for `EWO-016`'s own 0.1.2 disposition. This version records Repository Filing of the Founder's decision only — it does not itself commit, push, or publish this document or the `synapse-runtime` implementation it authorizes; both remain distinct, separately required future acts, exactly as `FIA-024`'s own final sentence states. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-08-08 |
| Independent Engineering Review | `ER-024` and `ER-024` Re-Review | Two Minor findings identified and resolved (Engineering Amendment v0.2.0); zero Critical, zero Major, zero unresolved finding at Re-Review; concluding `READY FOR FOUNDER IMPLEMENTATION APPROVAL` | 2026-08-08 |
| Approval Authority | Denver Jacobs, Founder | **Approved** — `FIA-024` = "APPROVE" (verbatim Founder Declaration recorded in §13) | 2026-08-08 |
