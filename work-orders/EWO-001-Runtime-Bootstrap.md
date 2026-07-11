---
document_id: EWO-001
title: Runtime Bootstrap
version: 0.2.0
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-11
last_updated: 2026-07-11
classification: Public
related_documents:
  governance:
    - GOV-003
    - GOV-004
    - GOV-010
  standards:
    - STD-001
    - STD-002
    - STD-004
    - STD-011
  architecture:
    - ARCH-001
    - ARCH-002
  adrs:
    - ADR-0011
    - ADR-0012
    - ADR-0013
    - ADR-0014
  reported_by: ER-001 (see engineering-reports/ER-001-Runtime-Bootstrap.md)
---

# EWO-001 — Runtime Bootstrap

Registered per STD-001 §46 (Engineering Work Orders). Content below is the Founder's issued authorization, reproduced without substantive alteration; only the controlled-document envelope (frontmatter, Document Control, this heading) was added.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-001 |
| Title | Runtime Bootstrap |
| Version | 0.2.0 |
| Status | Draft |
| Author | Denver Jacobs |
| Created | 2026-07-11 |
| Last Updated | 2026-07-11 |
| Classification | Public |
| Reported by | ER-001 |

---

## Engineering Authority

This implementation is governed by:

### Governance

- GOV-003
- GOV-004
- GOV-010

### Standards

- STD-001
- STD-002
- STD-004
- STD-011

### Architecture Decisions

- ADR-0011
- ADR-0012
- ADR-0013
- ADR-0014

### Architecture

- ARCH-001
- ARCH-002

These documents are authoritative. This task implements them. It does not reinterpret or modify them.

---

## Objective

Implement the SynapseOS Runtime Bootstrap. Nothing else.

---

## Scope

Implement only:

- Runtime lifecycle state machine
- Runtime bootstrap sequence
- Trusted Core initialization
- Trusted Core shutdown
- Runtime bootstrap orchestration
- Startup audit emission
- Shutdown audit emission
- Runtime bootstrap unit tests

---

## Out of Scope

Do NOT implement:

- Actor execution
- Capability issuance
- Capability validation
- Capability revocation
- Message routing
- Mailboxes
- Scheduling
- Persistence
- Actor directory
- Networking
- Distributed Runtime
- External providers
- Plugins
- SDK
- Security mechanisms beyond bootstrap
- Configuration loading beyond what is strictly required to boot
- Performance optimisation

---

## Runtime Requirements

Implement the Runtime lifecycle exactly as defined by ARCH-002.

The lifecycle must include valid state transitions. Invalid transitions must return an error.

The Runtime must:

1. Construct Trusted Core components.
2. Verify successful initialization.
3. Emit Runtime Started audit event.
4. Enter Running state.
5. Accept shutdown.
6. Emit Runtime Shutdown audit event.
7. Cleanly terminate.

No additional Runtime behaviour is permitted.

---

## Trusted Core

Use the existing crate structure. Do not change Trusted Core boundaries. Do not merge responsibilities. Do not move logic between components. No Trusted Core expansion is authorized.

---

## Architecture Constraints

Do not:

- introduce new Runtime concepts;
- invent lifecycle states;
- modify constitutional concepts;
- reinterpret ARCH-001;
- reinterpret ARCH-002;
- redesign the Runtime;
- introduce new crates;
- introduce external dependencies;
- use unsafe Rust.

If implementation requires any of the above: STOP. Produce an Engineering Report explaining the issue. Do not invent a solution.

---

## Repository Constraints

Do not modify: governance documents; architecture documents; standards; repository structure. Only modify source files necessary for Runtime Bootstrap.

---

## Definition of Done

The task is complete only if all of the following are true:

- Runtime boots successfully.
- Lifecycle state machine implemented.
- Invalid transitions rejected.
- Trusted Core initialization occurs in the documented order.
- Startup audit emitted.
- Shutdown audit emitted.
- Runtime terminates cleanly.
- All bootstrap tests pass.
- No warnings.
- No unsafe.
- No external dependencies.
- Trusted Core unchanged.
- Architecture unchanged.

---

## Mandatory Validation

Execute:

```text
cargo fmt --all
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace
cargo test --workspace
cargo tree --workspace
```

All must pass.

---

## Definition of Failure

Stop immediately if:

- a constitutional contradiction is discovered;
- ARCH-002 cannot be implemented as written;
- Trusted Core expansion becomes necessary;
- a new Runtime abstraction appears necessary;
- an architectural decision is required.

Do not resolve these issues yourself. Report them.

---

## Engineering Decision Log

Record:

- implementation decisions;
- repository decisions;
- deferred decisions;
- architectural decisions (expected: None);
- constitutional decisions (expected: None);
- future work enabled;
- future work deferred.

---

## Completion Report

Provide:

1. Files modified.
2. Files created.
3. Runtime behaviour implemented.
4. Lifecycle states implemented.
5. Tests added.
6. Validation results.
7. Dependency changes (expected: none).
8. Trusted Core changes (expected: none).
9. Architecture changes (expected: none).
10. Engineering Decision Log.
11. Any issues requiring architectural review.

Stop after Runtime Bootstrap. Do not begin the next engineering milestone.

---

## Authorized Implementation Clarification (2026-07-11)

This clarification resolves the gap ER-001 identified (composition-root placement undesignated). It does not change this EWO's original Objective, Scope, Out of Scope, Runtime Requirements, Trusted Core constraints, Architecture Constraints, Repository Constraints, or Definition of Done — all remain exactly as issued above.

**Decision:** exactly one new workspace member is authorized: an executable composition-root crate, package name `synapse-runtime`, directory `runtime/`, providing the executable entry point, constructing the seven trusted-core components, wiring their defined interfaces, owning Runtime bootstrap orchestration, and initiating orderly shutdown. This crate is not an eighth Trusted Core component, not a replaceable service, not a new constitutional concept, and not a new Runtime architectural abstraction — it is an implementation assembly boundary only. No ARCH amendment was required or made.

**Component construction:** each of the seven trusted-core crates was given the narrowest necessary public construction interface — a public constructor returning either an intentionally opaque public handle (six crates, where the underlying trait's behavior remains out of scope) or the crate's public trait abstraction directly via `Box<dyn Trait>` (Audit Emitter only, since emission is explicitly in scope). No internal implementation field was made public. No `pub(crate)` type was broadly widened.

**Dependency direction:** the composition root depends on `synapse-common` and all seven trusted-core crates. No replaceable-service crate is depended on — none is required by ARCH-002 §21 for Runtime initialization or shutdown. No trusted-core or replaceable-service crate depends on the composition root. No dependency cycle exists (verified via `cargo tree --workspace`).

This EWO is now **resumed** under its original scope. See ER-001 for full implementation and validation results.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-11 | Denver Jacobs | Initial EWO, registered per STD-001 §46. |
| 0.2.0 | 2026-07-11 | Denver Jacobs | Added Authorized Implementation Clarification resolving the composition-root gap ER-001 identified. Original objective, scope, and constraints unchanged. |
| 0.2.0 | 2026-07-11 | Denver Jacobs | Repository reconciliation correction (no version bump — Draft-phase correction, consistent with prior corpus precedent): fixed the Document Control table's `Version` field, which still read 0.1.0 after the frontmatter was bumped to 0.2.0; added the Disposition section's explicit distinction between "Resumed and completed" (engineering-record disposition) and the still-outstanding STD-001 §31 approval-evidence act; updated the Disposition text to reflect that the Runtime implementation is now committed locally in `synapse-runtime`. |

## Disposition

Disposed by ER-001: **Resumed and completed.** The composition-root gap identified on 2026-07-11 was resolved the same day by the Founder's Authorized Implementation Clarification above. Runtime Bootstrap was then implemented, committed locally in `synapse-runtime`, and validated within this EWO's original scope and constraints. See `engineering-reports/ER-001-Runtime-Bootstrap.md` for the full record, including the original halt as historical evidence and the committed-artifact identity.

**Distinct from formal approval.** "Resumed and completed" is a work-product and engineering-record disposition — it records that the authorized engineering task was carried out and reported. It is not, and must not be read as, the STD-001 §31 approval-evidence act. No content-non-mutating evidence commit exists for EWO-001, ER-001, or the Runtime implementation as of this revision. Recording that evidence, including approver identity and authority citation, remains a separate, later, not-yet-performed act.
