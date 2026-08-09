---
document_id: EWO-025
title: "Durable Actor Recovery"
version: 0.3.0
status: Approved
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-08-09
last_updated: 2026-08-09
classification: Public
related_documents:
  standards:
    - STD-001 (§46, Engineering Work Orders)
  architecture:
    - ARCH-007 (v0.5.2, Approved; architecture/ARCH-007-Persistent-Actor-Architecture.md) — §13 (Persistent Actor Architecture: Durable-State Contract, Runtime-held ActorId-keyed contract association (§13.3), five-way ownership split (§13.4), Durable-State Kind Identifier (§13.5), Representation Version (§13.6), Encoding Determinism (§13.7), Failure Semantics (§13.8)); §12 (Persistence Service's mechanics-only boundary)
    - ARCH-011 (v0.1.3, Approved; architecture/ARCH-011-Durable-Storage-Mechanics.md) — §9 (Storage Architecture: envelope/payload separation, outer format-version, Durable-State Kind Identifier, integrity checksum, checksum-algorithm identifier); §10 (Atomicity Guarantees: write-then-publish); §11 (Startup Discovery and Validation Sequence: integrity verification precedes decode, envelope-version acceptability checked next); §13 (Storage Integrity: checksum computed at write time; envelope's own corruption independently detectable; a corrupted envelope must never produce a false "valid" read)
    - ARCH-012 (v0.2.0, Approved; architecture/ARCH-012-Durable-DomainState-Encoding-Architecture.md) — §6 (Host Independence: explicit byte order, never native/host-dependent); recommended fixed-schema manual versioning as non-mandatory default encoding pattern
  adrs:
    - ADR-0018 (v0.3.0, Approved; adrs/ADR-0018-StorageBackend-Serialization-Boundary.md) — Option A: StorageBackend MUST consume encoded bytes, never DomainState directly
  predecessor: EWO-024 (Runtime Durable Storage Abstraction) — realizes the StorageBackend seam and PersistenceHandle::with_backend composition-time substitution point this EWO builds on; this EWO also closes EWO-024's own disclosed ARCH-011 §9 byte-level deviation (§7, below) by moving StorageBackend onto encoded bytes, per ADR-0018
  reported_by: "The next sequentially available Engineering Report identifier per STD-001 §7 at the time this EWO is reported on — not assumed in advance"
  base_state:
    runtime_head: 830d865085ebef434819f501886f9bc47a94b924
    docs_head: 7063d31b27836577204512710a7b7c79c9a2ee62
---

# EWO-025 — Durable Actor Recovery

Registered per STD-001 §46 (Engineering Work Orders). This authorizes implementation work only. It does not itself constitute approval, and does not amend ARCH-007, ARCH-011, ADR-0018, ARCH-012, or any other architecture, standards, or governance document.

> **Reconstruction Notice.** As with EWO-024, this document is authored after its implementation, Independent Engineering Review, Engineering Amendment, and Independent Engineering Re-Review already occurred against the `synapse-runtime` working tree, uncommitted, at base state `830d865`. It is a retroactive reconstruction of engineering authorization from direct repository evidence and the completed review chain, not a prospective authorization of not-yet-started work. At v0.1.0 this document records the original Engineering Work Order and the implementation it authorized. At v0.2.0 it records the Engineering Amendment resolving the Independent Engineering Review's findings. At v0.3.0 (this version) it records Founder Implementation Approval (§13) and constitutes this EWO's own Repository Filing.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-025 |
| Title | Durable Actor Recovery |
| Version | 0.3.0 |
| Status | **Approved** (STD-001 §12 — Founder Implementation Approval recorded; `FIA-025` = APPROVE) |
| Author | Denver Jacobs (AI-assisted) |
| Created | 2026-08-09 |
| Architecture authority | ARCH-007 (Approved, v0.5.2) §13; ARCH-011 (Approved, v0.1.3) §9, §10, §11, §13; ADR-0018 (Approved, v0.3.0); ARCH-012 (Approved, v0.2.0) §6 |
| Dependencies | None outstanding |
| Implementation repository | `synapse-runtime` |
| Implementation location | `common/src/lib.rs`, `services/persistence/src/lib.rs`, `services/persistence/src/internal.rs`, `services/persistence/src/envelope.rs`, `runtime/src/lib.rs`, `runtime/tests/durable_actor_recovery.rs` (working tree, uncommitted at authoring time) |
| Review requirement | Independent Engineering Review required before Founder Approval, per this corpus's own established EWO lifecycle |

---

## 1. Purpose

`ARCH-007` §13, `ARCH-011`, `ADR-0018`, and `ARCH-012` together establish the complete, approved architecture for durable actor domain-state persistence and restoration — the Durable-State Contract, the envelope/payload storage format, the byte-level `StorageBackend` boundary, and the encoding-pattern guidance — but before this EWO, none of it was realized end-to-end in the Runtime. `StorageBackend` still operated on `DomainState` directly (EWO-024's own disclosed deviation), no envelope existed, no Durable-State Contract association existed on `Runtime`, and no actor had ever been durably checkpointed, terminated, and genuinely reconstructed from disk in a fresh process.

This EWO closes that gap: it implements the full, seven-step durable actor recovery path — create actor, persist, terminate, fresh process, rediscover, reconstruct `DomainState`, recreate actor, resume execution — faithfully realizing the already-approved architectural chain, with no new architecture, registry, coordinator, or ownership change of any kind.

---

## 2. Background

**Prior state**: `StorageBackend` (`services/persistence`) operated on `DomainState` directly, per EWO-024's own disclosed, scope-limited deviation from `ARCH-011` §9's byte-level contract. No envelope, no Durable-State Contract trait, no `Runtime`-held contract association, and no disk-backed `StorageBackend` implementation existed. `ADR-0018` (then Draft, now Approved v0.3.0) had already resolved the serialization-boundary question this gap exposed; `ARCH-012` (then Draft, now Approved v0.2.0) had already resolved the encoding-pattern question. Both approvals removed the last architectural blocker EWO-025's own predecessor investigation had identified.

**What this EWO's implementation actually does**, verified directly against the working tree:

- Extends `RuntimeError` (`common/src/lib.rs`) with three variants: `NoDurableStateContract`, `DurableStateKindMismatch`, `DurableStateVersionUnsupported` — realizing `ARCH-007` §13.8's Failure Semantics categories.
- Migrates `StorageBackend` (`services/persistence/src/lib.rs`, `internal.rs`) to consume and return opaque bytes (`put`/`get`), closing EWO-024's disclosed deviation per `ADR-0018` Option A.
- Introduces `DurableStateContract` (`kind_identifier`, `encode`, `decode`) as the actor-owned codec pair `ARCH-007` §13.1 requires, and `FileStorageBackend`, a genuine disk-backed `StorageBackend` implementation (one file per `ActorId`, atomic write via temp-file-then-rename, never delete-then-write, per `ARCH-011` §10).
- Introduces the envelope format (`services/persistence/src/envelope.rs`) realizing `ARCH-011` §9/§13: outer format-version, Durable-State Kind Identifier, checksum-algorithm identifier, and checksum, with FNV-1a-64 as the concrete checksum choice.
- Adds `durable_state_contracts: HashMap<ActorId, Box<dyn DurableStateContract>>` to `Runtime` (`runtime/src/lib.rs`) — an ordinary field, not a new registry, per `ARCH-007` §13.3 — plus `with_persistence`, `register_durable_state_contract`, and `known_durable_actors`; `checkpoint_actor`/`restore_actor` resolve the associated contract before delegating to Persistence, failing with `NoDurableStateContract` if none is registered.
- Adds `runtime/tests/durable_actor_recovery.rs`, proving all six required end-to-end validation scenarios (§8, below) against genuine disk-backed storage in a fresh `Runtime` instance — never the same in-process state.

---

## 3. Architectural Authority

**Why the envelope is Persistence-Service-owned, never inspected by `StorageBackend` or for domain meaning.** `ARCH-007` §12 and `ARCH-011` §9 both assign envelope construction and verification to Persistence Service's own mechanics; `StorageBackend` receives and returns fully-opaque bytes. `envelope.rs`'s functions are `pub(crate)`, unreachable from `StorageBackend` or from any actor.

**Why the Durable-State Contract association lives on `Runtime`, not a new registry.** `ARCH-007` §13.3 requires the association be "an ordinary field," reusing the composition-time-supply pattern EWO-024's own `PersistenceHandle::with_backend` already established. `durable_state_contracts: HashMap<ActorId, Box<dyn DurableStateContract>>` is exactly that — no new coordinator, scheduler, or lookup service.

**Why integrity verification precedes semantic interpretation of envelope metadata.** `ARCH-011` §11's own validation-sequence ordering — "integrity verification precedes decode... prevents attempting to decode bytes already known to be corrupted" — is realized directly in `envelope::parse`: the checksum-algorithm identifier and checksum value are read as raw, uninterpreted bytes; the checksum is verified over every subsequent byte as an opaque region; only once it verifies is the format-version, Kind Identifier length, or Kind Identifier content given any semantic interpretation.

**Why the checksum covers the format version and Kind Identifier, not only the payload.** `ARCH-011` §13 requires the envelope itself be "structured so that its own corruption is independently detectable — a corrupted envelope must never be capable of producing a false 'valid' read." The checksum covers the entire envelope except the one-byte checksum-algorithm identifier, which necessarily precedes it (a verifier cannot check a checksum without first knowing which algorithm to apply); that byte's corruption still fails closed (`Malformed`), never silently accepted, since exactly one algorithm identifier is currently defined.

**Why `FileStorageBackend` writes via temp-file-then-rename.** `ARCH-011` §10 requires write-then-publish, never delete-then-write. Each checkpoint writes to `{actor}.durable.tmp` and atomically renames it to `{actor}.durable`, so a crash mid-write never destroys a previously-valid representation.

**Why `checkpoint_actor`/`restore_actor` check contract association after authorization.** Capability enforcement is `ARCH-007`'s own constitutional concern, unrelated to and prior to the storage-mechanics question of whether a contract happens to be registered; checking authorization first, then contract presence, keeps the two concerns cleanly separated and matches this Runtime's existing ordering convention elsewhere in the dispatch path.

---

## 4. Objectives

1. Realize `ARCH-007` §13's Durable-State Contract as a concrete Rust trait (`DurableStateContract`), actor-owned encode/decode.
2. Migrate `StorageBackend` onto encoded bytes, closing EWO-024's disclosed deviation, per `ADR-0018` Option A.
3. Implement the envelope format `ARCH-011` §9/§11/§13 requires, including the validation-sequence ordering and full metadata integrity coverage.
4. Implement `FileStorageBackend`, a genuine disk-backed `StorageBackend`, proving durability survives real process termination.
5. Add the Runtime-held, `ActorId`-keyed Durable-State Contract association `ARCH-007` §13.3 requires, as an ordinary field.
6. Prove the complete seven-step recovery flow end-to-end: create, persist, terminate, fresh process, rediscover, reconstruct, recreate, resume — across six required validation scenarios (§8).
7. Introduce no new architecture, registry, coordinator, scheduler, or ownership change of any kind.

---

## 5. Non-Goals / Explicit Prohibitions

This EWO MUST NOT, and its implementation does not:

- introduce any new Runtime service, registry, coordinator, or scheduler;
- introduce any global mutable map outside the one ordinary `Runtime`-held field `ARCH-007` §13.3 itself requires;
- implement process-local persistence, a fake or simulated recovery path, a hidden cache, or any shortcut/alternate path around the approved architectural boundary;
- select a durable storage *technology* beyond the file-per-actor `FileStorageBackend` this EWO itself introduces (`ARCH-011` §14's SQLite/LMDB backend-selection question remains a separate, future milestone);
- resolve the CRC-family checksum-algorithm trade-off `ARCH-011` §13 names as unresolved — FNV-1a-64 is this EWO's own implementation choice, not a resolution of that architectural question;
- alter `ARCH-007`, `ARCH-011`, `ADR-0018`, `ARCH-012`, or any other architecture, standards, or governance document;
- change any Effect Coordinator, Capability Authority, Audit, Scheduler, Supervisor, or Provider component's own behavior.

---

## 6. Required Data Model

Realized exactly as follows:

```text
// common/src/lib.rs — RuntimeError, three new variants
NoDurableStateContract,
DurableStateKindMismatch,
DurableStateVersionUnsupported,

// services/persistence/src/lib.rs
pub trait StorageBackend {
    fn put(&mut self, actor: &ActorId, bytes: Vec<u8>) -> Result<(), RuntimeError>;
    fn get(&self, actor: &ActorId) -> Result<Vec<u8>, RuntimeError>;
    fn remove(&mut self, actor: &ActorId) -> Result<(), RuntimeError>;
    fn known_actors(&self) -> Vec<ActorId>;
}

pub trait DurableStateContract {
    fn kind_identifier(&self) -> &str;
    fn encode(&self, state: &DomainState) -> Result<Vec<u8>, RuntimeError>;
    fn decode(&self, bytes: &[u8]) -> Result<DomainState, RuntimeError>;
}

pub trait Persistence {
    fn checkpoint(&mut self, actor: &ActorId, state: DomainState, contract: &dyn DurableStateContract) -> Result<(), RuntimeError>;
    fn retrieve(&self, actor: &ActorId, contract: &dyn DurableStateContract) -> Result<DomainState, RuntimeError>;
    fn delete(&mut self, actor: &ActorId) -> Result<(), RuntimeError>;
    fn known_actors(&self) -> Vec<ActorId>;
}

pub struct FileStorageBackend { /* one file per ActorId, atomic write */ }

// services/persistence/src/envelope.rs
// [u8] checksum-algorithm identifier
// [u64] checksum (covers every byte that follows it)
// [u32] outer format-version identifier
// [u32] Durable-State Kind Identifier byte length
// [..]  Durable-State Kind Identifier bytes (UTF-8)
// [..]  payload

// runtime/src/lib.rs — Runtime, one new field
durable_state_contracts: HashMap<ActorId, Box<dyn DurableStateContract>>,

impl Runtime {
    pub fn with_persistence(&mut self, persistence: PersistenceHandle);
    pub fn register_durable_state_contract(&mut self, actor: ActorId, contract: Box<dyn DurableStateContract>);
    pub fn known_durable_actors(&self) -> Vec<ActorId>;
    // checkpoint_actor / restore_actor: resolve contract, else NoDurableStateContract
}
```

---

## 7. Required Implementation Scope

Authorized files, exactly (confirmed as the complete diff against this EWO's own base state):

```text
common/src/lib.rs
services/persistence/src/lib.rs
services/persistence/src/internal.rs
services/persistence/src/envelope.rs      (new)
runtime/src/lib.rs
runtime/tests/durable_actor_recovery.rs   (new)
```

No other file is touched by this EWO's implementation or its subsequent Engineering Amendment (§9).

---

## 8. Required Test Coverage

**Six required end-to-end validation scenarios** (`runtime/tests/durable_actor_recovery.rs`), each against a real `FileStorageBackend` on disk, a fresh `Runtime` instance representing a genuinely separate process, and no shared in-process state between checkpoint and recovery:

1–2. `persistent_actor_survives_runtime_restart_with_domain_state_intact` — create, persist, terminate, fresh process, rediscover, reconstruct `DomainState` with the original value intact.
3. `recovered_actor_receives_messages_normally` — a recovered actor resumes ordinary message handling indistinguishably from one that was never terminated.
4. `recovery_fails_explicitly_for_corrupted_durable_bytes` — bytes corrupted directly on disk, bypassing every approved Persistence/StorageBackend path, are detected and reported as `IntegrityViolation`, never silently misinterpreted.
5. `recovery_fails_explicitly_for_an_incompatible_durable_state_contract_version` — an incompatible contract version is explicitly rejected, never approximated.
6. `recovery_path_bypasses_no_approved_architectural_boundary` — recovery is proven to traverse only `Runtime → Persistence → StorageBackend`, no shortcut.

**`envelope.rs`'s own unit test module** (10 tests at Founder Approval, following the Engineering Amendment — §9): round-trip correctness, truncation/malformed-byte handling, payload corruption, format-version corruption, Kind Identifier corruption, Kind Identifier length corruption, an intact-but-unsupported format version, an unrecognized checksum-algorithm identifier, checksum differentiation, and the `EnvelopeError`→`RuntimeError` conversion.

**`internal.rs`/`lib.rs`'s own test modules**: `FileStorageBackend` construction, put/get round-trip, missing-actor behavior, `known_actors` enumeration, survival across being dropped and reconstructed at the same path; `PersistenceImpl` contract-kind-mismatch detection and corrupted-bytes detection via the envelope checksum; trait object-safety for `StorageBackend`, `DurableStateContract`, and `Persistence`.

---

## 9. Engineering Amendment (v0.2.0) — EWO25-F01, EWO25-F02, EWO25-F03

An Independent Engineering Review of the original implementation identified two Major findings and one Minor finding, all confined to `services/persistence/src/envelope.rs`:

- **EWO25-F01 (Major)** — `envelope::parse` checked `format_version` acceptability before verifying the checksum, inverting `ARCH-011` §11's required validation-sequence ordering.
- **EWO25-F02 (Major)** — the checksum covered only the payload, leaving `format_version` and the Kind Identifier unprotected; their corruption surfaced as a misleading semantic result rather than as corruption, violating `ARCH-011` §13.
- **EWO25-F03 (Minor)** — the existing corruption tests exercised only payload corruption, never envelope-metadata corruption.

**Resolution**, confined entirely to `envelope.rs`: the envelope layout was reordered so the checksum-algorithm identifier and checksum are read first, as raw bytes, and the checksum is verified over the entire remaining byte region — format version, Kind Identifier length, Kind Identifier bytes, and payload — as one opaque blob, before any of those fields is given semantic interpretation. This resolves F01 (ordering) and F02 (coverage) together, since both shared the same root cause. Four regression tests were added proving corrupted format version, corrupted Kind Identifier, corrupted Kind Identifier length (fails via checksum before semantic interpretation would otherwise be attempted), and unrecognized checksum-algorithm-identifier behavior, resolving F03. No existing test was removed; one (`parse_rejects_an_unsupported_format_version`) was rewritten to construct a legitimately-checksummed alternate version, since a naive byte-flip now correctly surfaces as `ChecksumMismatch` instead.

An Independent Engineering Re-Review independently re-derived every conclusion from source (not from the Amendment Report), confirmed all three findings genuinely resolved, confirmed zero regression anywhere in the surrounding implementation, and independently re-verified all quality gates, concluding `READY FOR FOUNDER IMPLEMENTATION APPROVAL`.

---

## 10. Risks

| Risk | Mitigation / Required Test |
|---|---|
| Checksum-algorithm identifier corruption silently accepted | Structurally excluded from checksum coverage by necessity (must be read before the checksum can be verified); currently safe because exactly one algorithm identifier is defined — corruption fails closed as `Malformed`. Non-blocking observation: would need revisiting if a second algorithm identifier is ever introduced. |
| Corrupted durable bytes silently misread as valid | `ARCH-011` §13 checksum coverage now spans the full envelope; proven by `recovery_fails_explicitly_for_corrupted_durable_bytes` and the `envelope.rs` corruption test suite |
| Recovery bypassing the approved `Runtime → Persistence → StorageBackend` path | `recovery_path_bypasses_no_approved_architectural_boundary`; envelope functions are `pub(crate)`, unreachable from outside Persistence Service |
| Incompatible durable-state version silently approximated | `recovery_fails_explicitly_for_an_incompatible_durable_state_contract_version`; `ARCH-007` §13.7's encoding-determinism requirement (decode MUST fail explicitly, never approximate) |
| Backend-technology selection (`ARCH-011` §14) mistaken for resolved by this EWO | §5's explicit non-goal; `FileStorageBackend` is a working, disk-backed implementation, not a claim that SQLite/LMDB backend selection has occurred |

---

## 11. Reporting Requirement

An Engineering Report SHALL be authored recording: objective; implementation summary; every file modified; the envelope format realized, cross-referenced against `ARCH-011` §9/§11/§13; the Durable-State Contract association realized, cross-referenced against `ARCH-007` §13.3/§13.4; compatibility confirmation; every test from §8, with outcome; the Engineering Amendment record (§9); full verification results (`cargo fmt`, `cargo clippy -D warnings`, `cargo test`, all `--workspace --all-targets --all-features`); a scope audit confirming no non-goal (§5) was implemented and no unauthorized file (§7) was touched; and an explicit statement of readiness for Independent Engineering Review. Its own document identifier SHALL be derived from repository evidence at authoring time (STD-001 §7), not assumed in advance.

---

## 12. Disposition

**Approved.** Independently reviewed: engineering substance confirmed accurate against direct source re-derivation; two Major findings (`EWO25-F01`, `EWO25-F02`) and one Minor finding (`EWO25-F03`) identified, all confined to `services/persistence/src/envelope.rs`, none elsewhere in the implementation. Corrected by Engineering Amendment (v0.2.0), §9 above. Independently re-reviewed: all three findings confirmed resolved from source, zero regression, zero new defect, concluding `READY FOR FOUNDER IMPLEMENTATION APPROVAL`.

**Founder Implementation Approval granted.** `FIA-025` = **APPROVE**, Denver Jacobs, Founder, 2026-08-09, recorded verbatim:

> "I, Denver Jacobs, acting as Founder of SynapseOS, have reviewed the completed engineering record for EWO-025, including the implementation, engineering reports, repository evidence, Independent Engineering Review, Engineering Amendment, Independent Engineering Re-Review, findings, and recommendations. I independently adopt the conclusion of the completed engineering review chain that EWO-025 faithfully implements the approved constitutional architecture established by ARCH-007, ARCH-011, ADR-0018, and ARCH-012. I approve EWO-025 as the authoritative implementation of durable actor recovery within the SynapseOS Runtime. This approval confirms that: durable actor recovery has been implemented through the approved architectural recovery path; the StorageBackend boundary remains compliant with ADR-0018; Durable-State Contracts remain solely responsible for encoding and decoding actor state; StorageBackend continues to operate exclusively on durable byte representations; envelope validation and integrity verification satisfy the approved architectural requirements; corruption is detected and reported through explicit failure rather than silent approximation; the Runtime implementation remains consistent with the approved constitutional architecture. This approval authorizes Repository Filing, controlled commit, and controlled publication through the established SynapseOS engineering lifecycle."

The implementation this EWO authorizes already exists, complete and passing all current engineering quality gates (independently re-verified 2026-08-09: `cargo test --workspace --all-targets --all-features` — 992 passed, 0 failed; `cargo clippy --workspace --all-targets --all-features -- -D warnings` — clean; `cargo fmt --all -- --check` — clean), in the `synapse-runtime` working tree at base state `830d865`, **still uncommitted** — this Founder Approval authorizes, but does not itself perform, the commit, push, or publication of that implementation, nor of this document. Each remains a distinct, separately required future act.

**EWO-025 is Approved.** No further engineering, review, or amendment is authorized under it; its remaining lifecycle steps are controlled commit and push, of both this document and the `synapse-runtime` implementation it authorizes.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-09 | Denver Jacobs (AI-assisted) | Initial Draft. Reconstructed from direct repository evidence — the `common/src/lib.rs`, `services/persistence/src/{lib,internal,envelope}.rs`, `runtime/src/lib.rs`, and `runtime/tests/durable_actor_recovery.rs` working-tree diff against base state `830d865`, `ARCH-007` (Approved, v0.5.2) §13, `ARCH-011` (Approved, v0.1.3), `ADR-0018` (Approved, v0.3.0), and `ARCH-012` (Approved, v0.2.0) — following the Engineering Work Order authorizing implementation of the durable actor recovery path once all four governing documents reached Approved status. No engineering work performed by this document; it authorizes, retroactively, exactly what the existing implementation already does. |
| 0.2.0 | 2026-08-09 | Denver Jacobs (AI-assisted) | Engineering Amendment, correcting exactly the findings an Independent Engineering Review identified — `EWO25-F01` (Major, validation-sequence ordering), `EWO25-F02` (Major, checksum coverage), `EWO25-F03` (Minor, regression-test coverage) — all confined to `services/persistence/src/envelope.rs`, recorded in full in §9. No other engineering scope, requirement, or architectural claim changed. |
| 0.3.0 | 2026-08-09 | Denver Jacobs (Founder) | Governance disposition recorded — **no engineering-scope, requirement, or exclusion changed from 0.2.0**. Records the Founder's decision on the completed Independent Engineering Re-Review (concluding `INDEPENDENT ENGINEERING RE-REVIEW COMPLETE — EWO-025 VERIFIED — READY FOR FOUNDER IMPLEMENTATION APPROVAL`, all three findings independently re-confirmed resolved from source, zero regression, zero new defect): `status` transitions from `Draft` to **`Approved`**. §12's Disposition section records the full review/amendment/re-review chain and quotes `FIA-025`'s own Founder Declaration verbatim. This version records Repository Filing of the Founder's decision only — it does not itself commit, push, or publish this document or the `synapse-runtime` implementation it authorizes; both remain distinct, separately required future acts. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-08-09 |
| Independent Engineering Review | Independent Engineering Review and Independent Engineering Re-Review | Two Major findings and one Minor finding identified and resolved (Engineering Amendment v0.2.0); zero unresolved finding at Re-Review; concluding `READY FOR FOUNDER IMPLEMENTATION APPROVAL` | 2026-08-09 |
| Approval Authority | Denver Jacobs, Founder | **Approved** — `FIA-025` = "APPROVE" (verbatim Founder Declaration recorded in §12) | 2026-08-09 |
