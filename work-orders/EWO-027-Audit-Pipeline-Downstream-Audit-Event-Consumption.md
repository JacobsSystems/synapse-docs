---
document_id: EWO-027
title: "Audit Pipeline: Downstream Audit Event Consumption"
version: 0.3.0
status: Implemented
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
created: 2026-08-12
last_updated: 2026-08-12
classification: Public
related_documents:
  architecture:
    - ARCH-001 (Constitutional Architecture, Approved) — §6/§7/§8 (audit emission named as one of the foundational Runtime mechanisms sitting outside the Actor model, not one of §9's four irreducible mechanisms); §9 (the four irreducible mechanisms: capability non-forgery, capability integrity, enforcement at invocation, revocation-state enforcement — audit is not among them, corrected per IAR-027-F03)
    - ARCH-002 (Runtime Architecture, Approved) — §5 (Audit Emitter, Trusted Core, "supports auditability of all of the above"), §6 (Audit Pipeline, replaceable service, binding contract), §11 step 17, §13 (bounded-mailbox precedent this document's own overflow policy follows), §16 (Failure Model), §18, §19, §20, §22
  adrs:
    - ADR-0015 (Draft, v0.1.0, unapproved) — adjacent, distinct failure domain (Emitter's own emission failure); not this document's own authority
    - ADR-0016 (integrated into ARCH-002 per EWO-002 v0.3.0) — Trusted Core interaction accountability belongs to the Runtime alone
  work-orders:
    - EWO-001 — origin of AuditEmitter's own pre-existing in-process buffer and the deferred "real Audit Pipeline consumer"
    - EWO-002 (Draft v0.3.0) — independent restatement of this document's own governing failure-domain-separation rule
    - EWO-025 (Approved, Founder Implementation Approval v0.3.0) — the actual, verified source of the `FileStorageBackend` direct-file-I/O-outside-capability-mediation precedent this document relies on (ARCH-011 itself names no concrete backend implementation)
    - EWO-029.3 (Approved) — independent precedent for this document's own `runtime → services/audit-pipeline` dependency direction
  design-explorations:
    - DES-004 — independently analyzes the preserved implementation as "write-only," scopes audit-history/query architecture out as separate future work
  engineering-reports:
    - ER-001, ER-002 — origin of the "Pipeline failure remains independent" deferred test this document's own §26 makes non-vacuous
  governing_reviews:
    - EWO-027 — Independent Architecture Review (2026-08-12) — verdict MAJOR CORRECTION REQUIRED; findings IAR-027-F01 through F05 and observations O01 through O04, all treated (§39 Correction Matrix)
    - EWO-027 — Independent Architecture Re-Review (2026-08-12) — verdict NARROW CORRECTION REQUIRED; IAR-027-F01–F05 confirmed RESOLVED; IAR-027-O02–O04 confirmed INCORPORATED; IAR-027-O01 found NOT actually incorporated despite v0.2.0's own Correction Matrix claiming otherwise (NRR-027-F01); v0.2.1 corrects exactly that one item
  founder_decisions:
    - Delivery Semantics (2026-08-12) — best-effort/at-most-once RETAINED, explicitly as EWO-027's own new architectural decision, not as something ARCH-002 §6/§16 requires (§13)
    - Bounded Audit Buffer (2026-08-12) — AuditEmitter buffer MUST be bounded; DROP-OLDEST overflow policy; separate dropped-event counter (§14)
    - Founder Architecture Approval (2026-08-12) — APPROVE WITH NON-BLOCKING OBSERVATIONS, obtained via direct, explicit Founder response (see §40)
  predecessor: EWO-027 v0.2.0 and v0.1.0 (Draft, unfiled, chat-delivered — this filed version's own direct predecessors)
  base_state:
    runtime_head: ba0af14 (local, uncommitted candidate implementation preserved on top of this base; NOT authorized for implementation by this document — see §31, §41)
    docs_head: a9c9c76
---

# EWO-027 — Audit Pipeline: Downstream Audit Event Consumption

> **Filing Notice.** This document is filed at v0.2.1, the version at which Founder Architecture Approval was granted (2026-08-12) — no automatic version bump on approval, per `STD-001` §13 ("recording approval evidence... does not itself require or imply a version change to the approved artifact") and direct precedent (`EWO-029.4` filed/approved at v0.2.0; `EWO-029.5` filed/approved at v0.3.0 — in both cases the approved version is whatever version the draft had reached, with only `status` changing). This document was authored, reviewed, corrected, re-reviewed, corrected again, and approved entirely in chat/session work before this Filing — the identical "reconstruction" pattern `EWO-024`, `EWO-025`, and every `EWO-026.x`/`EWO-029.x` predecessor already established for themselves.
>
> **This document authorizes architecture only. It does not authorize implementation, commit, or push of any `synapse-runtime` code.** See §40 (Approval Record) and §41 (Implementation Authorization Status).

## 1. Purpose

To define the architecture for the **Audit Pipeline** — the replaceable service `ARCH-002` §6 already names and binds ("Storage, indexing, retention, redaction of emitted events... Consumes Audit Emitter output only; failure here MUST NOT affect the triggering execution") but which no work order had ever implemented. `Audit Emitter` has emitted events into an unread, pre-existing in-process buffer since `EWO-001`; this document defines how those events are read out, bounded, and delivered to a real, external, replaceable consumer — without altering Audit Emitter's own Trusted Core status.

Audit emission supports auditability; it is not itself one of `ARCH-001` §9's four irreducible mechanisms (§2 below states the accurate relationship).

## 2. Authoring Hierarchy

Approved governance → approved architecture → approved/filed work orders → accepted implementation invariants → historical repository evidence → preserved unpublished implementation → inference, in that order, never reversed.

## 3. Background / Recovered Context

`ARCH-002` §5–§6 already establish two, deliberately separate things: **Audit Emitter**, Trusted Core, "Minimal audit-event emission (not storage or processing)"; and **Audit Pipeline**, a *replaceable service*, "Storage, indexing, retention, redaction of emitted events," bound by the rule that it "Consumes Audit Emitter output only; failure here MUST NOT affect the triggering execution." `EWO-001` implemented Audit Emitter as a trivial in-process `Vec<AuditEvent>` buffer — sufficient to make `emit()` calls type-check, insufficient to make audit evidentiary beyond a single process's own memory, discarded on every restart. `ER-001` §Deferred Decisions names this explicitly: "a real Audit Pipeline consumer for emitted events" was deferred, not forgotten.

No work order was ever filed to close that deferral. Instead, the identifier `EWO-027` began appearing, unfilled, as a placeholder other work orders route around: `EWO-026.4`, `EWO-026.5`, and `EWO-029.1` each independently, explicitly disclose that real, uncommitted `EWO-027` (Audit Pipeline) work sits in the working tree, alongside their own commits, and was deliberately excluded from each. `EWO-029.1` records, as a lesson learned, that this exact drift was once accidentally staged into an unrelated commit and had to be corrected via `git reset --soft` before push — direct evidence this is neither abandoned nor accidental, but actively, carefully preserved.

## 4. Governing Evidence

| Requirement | Governing Document | Section | Implication for EWO-027 |
|---|---|---|---|
| Audit Pipeline is a replaceable service consuming Audit Emitter output; its failure must not affect the triggering execution | ARCH-002 (Approved) | §6 | This work order's central failure-containment obligation |
| Audit-event emission step is unbypassable; "Consumer failure MUST NOT block execution" | ARCH-002 | §11 step 17 | Same obligation, restated at the execution-cycle level |
| §11 step 17's tolerance governs only downstream *consumer* (Pipeline) failure, not Emitter's own emission failure (that is ADR-0015's separate, still-Draft domain) | ARCH-002 | §11 (explanatory note) | This work order must not conflate the two failure domains; §16 draws this boundary explicitly |
| "Audit consumer failure — MUST NOT block or fail the triggering execution — isolated to the replaceable service" | ARCH-002 | §16 (Failure Model) | Canonical failure-tier text this work order's own Failure Isolation section restates |
| Audit Emitter is Trusted Core; "not storage or processing" | ARCH-002 | §5 | Audit Pipeline must never be folded into Audit Emitter or treated as Trusted Core |
| Mailbox capacity MUST be bounded and finite; the specific number is a deployment parameter (policy) | ARCH-002 | §13 | Direct precedent this document's own bounded-buffer requirement (§14) follows |
| Security audit, operational tracing, debugging, metrics, billing, and provenance are distinct consumer classes — "not one undifferentiated log" | ARCH-002 | §18 | Bounds this work order against becoming a general observability/telemetry framework |
| No unrestricted plugin surface for replaceable services | ARCH-002 | §19 | Audit Pipeline's own interface stays narrowly scoped to consumption |
| Audit-event emission "Never blocks on downstream consumption" | ARCH-002 | §20 (Runtime Interfaces) | Restates the non-blocking obligation a third time |
| "audit emission for every security-relevant event... not reported successful unless that emission succeeded" | ARCH-002 | §22 | Governs Emitter's own emission (ADR-0015's domain), not Pipeline consumption |
| No Trusted Core component interacts directly with another; the Runtime alone owns interaction accountability | ADR-0016 (integrated) | via EWO-002 v0.3.0 | Audit Pipeline must be Runtime-mediated |
| "Downstream Audit Pipeline failure remains governed exactly as ARCH-002 §11 step 17 and §16 already state... a distinct failure domain from Audit Emitter's own emission failing" | EWO-002 (Draft v0.3.0) | Audit Failure Semantics | Independent restatement of this document's own central rule |
| `runtime` depends on `services/audit-pipeline` (never the reverse) to reference the externally-defined `AuditPipeline` trait via `with_audit_pipeline` | EWO-029.3 (Approved) | §5, §17 | Direct, already-approved precedent for this document's own dependency direction |
| `FileStorageBackend` (direct file I/O, composition-root-configured, outside capability/filesystem-provider mediation) | EWO-025 (Approved) | Full document | The verified source of this precedent — `ARCH-011` supplies only the generic storage-backend abstraction (§9) and names no concrete implementation |
| The trusted core enforces exactly four irreducible mechanisms: capability non-forgery, capability integrity, enforcement at invocation, revocation-state enforcement | ARCH-001 (Approved) | §9 | Audit is **not** among these four; `ARCH-002` §5's "Supports auditability of all of the above" is precise |
| Audit emission is one of the "fixed, named set of foundational Runtime mechanisms" sitting outside the Actor model | ARCH-001 (Approved) | §6, §7, §8 | Audit's actual governing status |
| Current `AuditEmitter::drain`/`AuditPipeline::consume` mechanism is real, "write-only"; a stable, queryable audit-history store is separate, future, out-of-scope architecture | DES-004 | §19, §42 | Confirms this work order must not attempt history/query/read-back |

## 5. Problem Statement

`Audit Emitter` has satisfied `ARCH-002` §22's mandatory emission requirement since `EWO-001`, but the events it emits have never left process memory. `ARCH-002` §6 already defines the missing piece (Audit Pipeline) and its failure contract; no work order had ever built it before this one.

## 6. Scope

**In scope:**
- A `drain()`-style, destructive, order-preserving read-out mechanism on `AuditEmitter`.
- An `AuditPipeline` trait: `consume(&mut self, event: AuditEvent)`, no `Result` return.
- Exactly one concrete `AuditPipeline` implementation: an append-only, local-file text sink (`FileAuditPipeline`).
- Optional Runtime-level composition: zero or one `AuditPipeline` per `Runtime`.
- An explicit, caller-triggered delivery operation (`flush_audit_events`).
- A bounded `AuditEmitter` buffer with a DROP-OLDEST overflow policy and a dedicated `dropped_due_to_capacity` counter (§14).
- Placement in `services/audit-pipeline`, depended on by `runtime` (never the reverse).

**Explicitly out of scope:** audit history storage, querying, or any read-back API; log rotation, retention policy, compression, encryption; any remote or networked audit sink; any new wire protocol or serialization dependency; retry, redelivery, or exactly-once/at-least-once delivery guarantees; automatic or background flush triggering; general telemetry, metrics, distributed tracing; SDK-level audit API, Control Centre integration; any new capability type or authority-granting mechanism; `EWO-029.x`; `DX-001`, SDK publishing metadata, `LICENSE-*` files; authoring or inventing `OPS-001`.

## 7. Architectural Invariants

Audit Emitter remains Trusted Core, emission-only. Audit Pipeline remains a replaceable service, never promoted to Trusted Core. All Emitter↔Pipeline interaction is Runtime-mediated. Pipeline consumption failure must never block or fail the Runtime operation that produced the audited event — structurally distinct from, and never conflated with, Audit Emitter's own emission-failure domain (`ADR-0015`, still Draft, not this document's concern).

## 8. AuditEmitter Boundary

`AuditEmitter` gains one new trait method beyond its existing `emit`: a destructive, order-preserving read-out (`drain`) that empties its buffer and returns every event emitted since the last drain. The buffer itself is not new — it has existed, unbounded, since `EWO-001`. This document changes two things about it, both new: (1) the destructive read-out (`drain`); (2) a capacity bound with DROP-OLDEST overflow policy (§14). The buffer's mere existence is out of this document's own scope; its unboundedness is not, and is corrected here.

**This is EWO-027's own architecture choice, requiring Independent Implementation Review's own confirmation, not pre-authorized by `ARCH-002` itself.** `ARCH-002` §6 establishes *that* Audit Pipeline consumes Audit Emitter's output; it does not specify *how* — push versus pull. `ADR-0015` §7 independently discloses a directly analogous open question ("which coupling mechanism connects a given Trusted Core component to Audit Emitter") as still unresolved on the Component→Emitter side; the pull-based `drain()` is this document's own answer to the structurally identical question on the Emitter→Pipeline side.

## 9. AuditPipeline Abstraction

A single-method trait: `consume(&mut self, event: AuditEvent)`, returning nothing. The absence of a `Result` return type structurally guarantees that **ordinary, `Result`-shaped failure** cannot propagate through `consume`'s own return channel — a real, load-bearing, type-system-enforced property, sufficient on its own to satisfy `ARCH-002` §6/§11 step 17/§16/§20's repeated non-blocking requirement for *that* failure class.

It does **not** guarantee anything about a `consume` implementation that panics — nothing in the trait signature prevents a panic from unwinding through the caller. This is a truthful architectural limitation, not resolved by inventing a containment mechanism here: whether a future implementation stage adds panic containment (e.g. `catch_unwind` at the `Runtime::flush_audit_events` call site) is an **implementation acceptance obligation**, not something this architecture document mandates. Every future `AuditPipeline` implementation is responsible for its own panic safety.

The no-`Result` interface also intentionally forecloses any implementation from reporting ordinary sink failure upward through `consume` itself for retry or alerting purposes — acceptable given retry/redelivery/error-routing are explicitly out of scope (§6); a future retry-capable implementation would require this interface to evolve, which this document does not authorize or design for now.

## 10. Runtime Integration

A `Runtime` optionally owns one `AuditPipeline` (`Option<Box<dyn AuditPipeline>>`), on the identical composition-time-optional basis already established for `PersistenceHandle` and the Observation Service. Delivery is exclusively caller-triggered — **this is EWO-027's own architecture choice**, not implied by `ARCH-002` itself, satisfying the non-blocking requirement (§16) by construction since `emit`'s own call path never touches the Pipeline at all.

This architecture supports **exactly zero or one** configured `AuditPipeline` per `Runtime`. This is an explicit, intentional **initial simplification** for this architecture stage, not a constitutional limit.

## 11. FileAuditPipeline Status

**Classification: the first concrete `AuditPipeline` implementation** — not the architecture-mandated sole sink, not merely an example. This mirrors the direct precedent of `services/persistence`'s `StorageBackend` trait plus `EWO-025`'s own first concrete `FileStorageBackend` implementation: a narrow trait boundary, with exactly one first concrete implementation, substitutable later without an architecture change. Its current line-based, human-readable text output format is an implementation detail, not a protocol — `DES-004` §19 already independently classifies the mechanism as "write-only" with no read-back contract to be compatible with.

## 12. Event Ordering

Among `AuditEvent`s that remain in the buffer and are subsequently drained, relative emission order MUST be preserved. **No claim is made that every emitted event survives to be drained** — an event discarded by overflow (§14) or lost to downstream Pipeline failure (§13) never reaches the Pipeline at all, and its absence is not an ordering violation, since ordering is a property only of the events that do survive.

## 13. Delivery Semantics

EWO-027 retains **best-effort, at-most-once** audit delivery. **This is EWO-027's own deliberate, new architectural decision for this stage — it is not something `ARCH-002` §6 or §16 requires.** Those sections require only that Pipeline failure not affect the triggering execution; they say nothing about whether the audit event itself must survive that failure. A retain-and-retry alternative was considered and explicitly not selected, per Founder Decision, in favor of the simpler model.

Full disclosed consequence: an `AuditEvent` may (1) be successfully emitted; (2) enter the `AuditEmitter` buffer; (3) be removed during delivery (drain); (4) fail downstream consumption; (5) never reach durable or external storage. **That event is not retried by EWO-027.** No retry infrastructure, acknowledgement, redelivery, durable pending queue, exactly-once, or at-least-once semantics exists or is authorized.

**Two distinct loss channels, never conflated:**

**(A) Pipeline-consumption loss** — an event survives the buffer and is handed to `consume()`, but the sink fails. Counted by `FileAuditPipeline::write_failures` (§16).

**(B) AuditEmitter-buffer-overflow loss** — an event is discarded before ever being drained, because the bounded buffer was full when it was emitted. Counted by `AuditEmitter::dropped_due_to_capacity` (§14).

## 14. Bounded Audit Buffer / Overflow Policy

`AuditEmitter`'s buffer **MUST be bounded**. Unbounded Runtime-owned audit-event accumulation is prohibited — directly consistent with `ARCH-002` §13's own, already-Approved bounded-mailbox precedent.

**Overflow policy: DROP-OLDEST.** When the buffer is at capacity and a new event is emitted: the oldest buffered event MUST be discarded; the newest event MUST be admitted; `AuditEmitter` MUST maintain a monotonically increasing, in-memory `dropped_due_to_capacity` counter, incremented once per discarded event; dropping an event MUST NOT itself emit another `AuditEvent`; overflow MUST NOT block or fail the triggering Runtime operation.

**Rationale:** bounds Runtime memory; preferentially retains newer evidence; keeps the triggering execution fully isolated; introduces no backpressure; avoids recursive audit generation; makes loss measurable rather than entirely silent.

**Capacity:** no existing architecture establishes an applicable concrete capacity value — `ARCH-002` §13 itself defers the mailbox's own specific bound to "a deployment parameter (policy)," and this document follows the identical precedent. Capacity **MUST be finite** and **MUST be either explicitly configured or have an implementation-defined bounded default**; the exact default is an implementation-stage decision. Founder Implementation Acceptance must verify the selected default is documented and tested.

## 15. AuditEmitter Trusted-Core Boundary

Neither the pre-existing buffer nor its new bound/overflow policy constitutes "storage or processing" in the sense `ARCH-002` §5 prohibits Audit Emitter from performing — transient, in-process bookkeeping incidental to emission, not a storage service; no indexing, retention policy, or query capability is introduced.

## 16. Write-Failure Accounting

`FileAuditPipeline::write_failures` counts **only** downstream write/flush failures at the Pipeline sink. It explicitly **excludes**: `AuditEmitter` buffer-overflow drops (a wholly separate counter); Pipeline construction failures; serialization/formatting failures; shutdown loss; successfully delivered events. `AuditEmitter::dropped_due_to_capacity` and `FileAuditPipeline::write_failures` are **two distinct diagnostic indicators**, never merged, never generalized into a metrics framework, not exposed through any SDK/Control Centre/query API.

## 17. Flush Semantics

Explicit, caller-triggered, delta-since-last-flush, safe with no Pipeline configured. Because the buffer is now bounded (§14), a `Runtime` that never flushes **no longer risks unbounded memory growth** — but it **does** risk audit-completeness degradation: old events will eventually be silently discarded under DROP-OLDEST as new ones arrive. Forgetting to flush is therefore no longer a resource-safety problem; it remains an audit-completeness problem.

## 18. Shutdown/Lifecycle Semantics

No Runtime lifecycle machinery is invented. Events remaining buffered when `Runtime` is destroyed **may be lost**; this document does **not** guarantee automatic shutdown flushing; this is part of the same, already-disclosed, accepted best-effort model as §13/§14, not a separate gap.

## 19. Durability Claims and Non-Claims

**Claimed:** `FileAuditPipeline` calls `File::flush()` after every write. **Not claimed:** no `fsync`/`sync_all` exists — data reaching physical storage, surviving an OS crash or power loss, is not guaranteed. No rotation, retention, compression, or encryption exists or is claimed.

## 20. Security / Authority Boundary

**Audit data flow introduces no Runtime authority.** `consume` receives an owned `AuditEvent` and returns nothing — no reference to `Runtime`, no capability, no command channel. Nothing in this document's scope can control actors, mutate Runtime state, grant capabilities, execute effects, or change supervision or persistence.

## 21. Filesystem/Path Security Decision

`FileAuditPipeline::new(path)` performs direct `std::fs::File` I/O against an operator-supplied path — not mediated through `services/filesystem-provider`'s capability-scoped actor pathway. This is not a novel deviation: `EWO-025`'s own `FileStorageBackend` (Approved) establishes the identical precedent — composition-root-time, operator-configured direct file I/O, entirely outside actor/capability mediation, since path selection here is a deployment-configuration concern. No new capability model is introduced.

## 22. Dependency Direction

`runtime` → `services/audit-pipeline` → `synapse-common`, never the reverse — independently affirmed by `EWO-029.3` §5/§17 (Approved). `core/audit-emitter` gains no new dependency. No cycle exists.

## 23. Placement

`services/audit-pipeline` is architecturally consistent with `ARCH-002` §6's classification of Audit Pipeline as a replaceable service, structurally parallel to `services/persistence`.

## 24. Configuration

Path (and buffer capacity, §14) are supplied entirely by the embedding application at `Runtime` construction time. No configuration file format, environment variable convention, or discovery mechanism is in scope.

## 25. Error Model

`FileAuditPipeline::new` returns `Result<Self, RuntimeError>`, mapping I/O failure at construction time to `RuntimeError::IntegrityViolation` — the identical convention `EWO-025`'s `FileStorageBackend::new` establishes. Post-construction, `consume` never returns an error — the only observable post-construction failure signals are the two loss counters (§13, §16).

## 26. Testing Architecture / Acceptance Requirements

Founder Implementation Acceptance must require tests proving, at minimum: (1) bounded capacity is enforced; (2) capacity cannot grow without bound; (3) overflow drops the oldest event; (4) the newest event is retained; (5) `dropped_due_to_capacity` increments correctly; (6) overflow does not fail the triggering Runtime operation; (7) overflow does not recursively emit an `AuditEvent`; (8) surviving events retain relative order; (9) repeated flushes deliver only the current delta; (10) no-Pipeline configuration remains safe; (11) a deliberately failing `AuditPipeline` cannot fail the triggering Runtime operation through ordinary (`Result`-shaped) failure paths — makes `ER-002`'s own "Pipeline failure remains independent" test non-vacuous for the first time; (12) `write_failures` and `dropped_due_to_capacity` accounting remain separate; (13) shutdown with unflushed events exhibits the documented best-effort behavior; (14) high-volume operation remains memory-bounded.

## 27. Implementation Sequencing

Not authorized by this document (§41).

## 28. Implementation Recovery / Porting Strategy

The preserved candidate implementation sits on `synapse-runtime` local `main` at `ba0af14`, seven commits behind current `origin/main` (`4d8d32d`), independently confirmed non-overlapping with every one of those seven intervening commits.

Reconciling a generic implementation-sequence template against this repository's actual, real, twice-repeated lifecycle pattern (`GOV-013`, **Draft, v0.1.0, unapproved**, cited by `EWO-029.1`; the same pattern `EWO-025`/`EWO-026.x` independently followed even while `GOV-013` itself remained Draft) yields two distinct stages, not one flat list:

**Architecture stage** (this document): Draft v0.1.0 → Independent Architecture Review (MAJOR CORRECTION REQUIRED) → Narrow Architecture Correction (v0.2.0) → Independent Architecture Re-Review (NARROW CORRECTION REQUIRED) → Narrow correction (v0.2.1) → Founder Architecture Approval (§40) → this Filing. **Complete.**

**Implementation stage** (separate, later, not authorized by this document): treat the preserved code as a submission against this now-approved architecture → Independent Implementation Review → Narrow Implementation Correction → Re-Review → synchronize `synapse-runtime` to current `origin/main` → mechanically port only EWO-027-owned hunks → regenerate `Cargo.lock` → targeted tests → workspace tests → clippy/fmt → Founder Implementation/Acceptance Approval → Repository Filing → controlled, narrowly-staged commit → push. **Not begun.**

## 29. OPS-001 Disposition

No `OPS-001` artifact exists anywhere in either repository, independently re-confirmed multiple times across this document's own review history. Disposition: reconstructable from `ARCH-001`/`ARCH-002` directly; not created by this document. The preserved code's own citations of `OPS-001` should be corrected during Independent Implementation Review, not before.

## 30. Preserved Implementation — Status

Historical/evidentiary material only. The preserved candidate implementation predates this document's own corrections and does **not** currently implement the bounded-buffer/DROP-OLDEST policy (§14) or the corrected delivery-semantics framing (§13) — expected and disclosed, not a defect; these are new obligations for the next implementation stage to satisfy.

## 31. EWO-029.x Isolation

Confirmed orthogonal — zero shared types, zero shared files beyond incidental proximity. `EWO-029.3` independently cites the `runtime → services/audit-pipeline` boundary as a clean precedent for its own, unrelated placement.

## 32. Compatibility Considerations

None identified against any other approved architecture. The `AuditEmitter` trait change (§8) is additive, not a breaking change to `emit`'s own existing signature or callers.

## 33. Risks / Trade-offs

Best-effort delivery loss (§13) and buffer-overflow loss (§14) are both explicitly named, Founder-decided, disclosed risks. No fsync/durable-to-media guarantee (§19). DROP-OLDEST's recency-priority rationale trades off against the causal/investigative value of older events in some security-investigation scenarios — noted for future awareness, not reopened (`NRR-027-O02`, carried forward, §39).

## 34. Engineering Stops

None triggered across this document's own architecture-stage lifecycle (recovery, review, correction, re-review, narrow correction, approval, filing).

## 35. Acceptance Criteria (architecture stage)

Satisfied — Founder Architecture Approval granted (§40).

## 36. Founder Decisions Required (at architecture stage)

None remaining for architecture. Implementation-stage authorization remains a separate, future Founder decision (§41). Unrelated, still outstanding: `windows.rs` archive/restore; `DX-001`/SDK-metadata drift disposition.

## 37. Change History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-12 | Denver Jacobs (AI-assisted recovery) | Initial Draft, recovered from repository evidence. |
| 0.2.0 | 2026-08-12 | Denver Jacobs (AI-assisted correction) | Targeted correction applying the five findings and four observations of the Independent Architecture Review of v0.1.0. New §14 (Bounded Audit Buffer / Overflow Policy). §13 rewritten. ARCH-001/EWO-025 citations corrected. §9 panic-safety distinction added. |
| 0.2.1 | 2026-08-12 | Denver Jacobs (AI-assisted correction) | Narrow, PATCH-scale correction applying exactly `NRR-027-F01`, the sole finding of the Independent Architecture Re-Review of v0.2.0. §28's `GOV-013` citation corrected to disclose "(Draft, v0.1.0, unapproved)." §39 Correction Matrix corrected to truthfully attribute `IAR-027-O01`'s closure to this version. No architectural decision altered. **Founder Architecture Approval granted at this version** (§40) — filed as-is, no version bump on approval, per `STD-001` §13 and direct `EWO-029.4`/`EWO-029.5` precedent. |
| 0.3.0 | 2026-08-12 | Denver Jacobs (Founder) | Governance disposition recorded — no architectural decision, obligation, or scope changed from 0.2.1. Records Final Founder Implementation Acceptance (§41) for the complete implementation, chat-delivered-and-reviewed across four working-management stages (informally labeled "Units 1–4" for session convenience only — a label this architecture itself never defines) plus one Narrow Implementation Correction, through `synapse-runtime` commit `9fa5cbb459a6aec921b4b32fc71b8105964087e2`. `status` transitions from `Approved` to **`Implemented`**, per `STD-001` §12/§46. MINOR-scale per `STD-001` §13 (a clarifying addition, not a change to obligations or architecture intent), directly following `EWO-025` §12/Revision-History precedent (0.2.0 → 0.3.0 to record a Founder implementation disposition). This version records Repository Filing of the Founder's decision only — it does not itself alter §1–§37 above. |

## 38. Approval Status

### Document State

| Field | Value |
|---|---|
| Lifecycle status | Implemented |
| Normal-governance disposition | Approved — Architecture, with non-blocking observations; Implemented — complete implementation Founder-accepted (§41) |
| Disposition date | 2026-08-12 (architecture); 2026-08-12 (implementation) |
| Approving actor | Denver Jacobs (Founder) |

## 39. Correction Matrix

| Review Item | Correction | Status |
|---|---|---|
| IAR-027-F01 | §13 rewritten: best-effort/at-most-once framed as EWO-027's own Founder-decided architecture choice, not ARCH-002-required | RESOLVED |
| IAR-027-F02 | §14 (new): bounded buffer, DROP-OLDEST policy, `dropped_due_to_capacity` counter | RESOLVED |
| IAR-027-F03 | §1/§4 corrected: audit's relationship to ARCH-001 §9 stated accurately | RESOLVED |
| IAR-027-F04 | §4/§11/§21 corrected: precedent is EWO-025, not ARCH-011/EWO-024 | RESOLVED |
| IAR-027-F05 | §9 corrected: no-`Result` guarantees isolation only from ordinary failure, not panics | RESOLVED |
| IAR-027-O01 | Not actually incorporated by v0.2.0 despite that version's own matrix claiming otherwise (`NRR-027-F01`). Genuinely incorporated by v0.2.1: §28's `GOV-013` citation now explicitly reads "(Draft, v0.1.0, unapproved)" | INCORPORATED (v0.2.1) |
| IAR-027-O02 | §10: zero-or-one-Pipeline-per-Runtime disclosed as intentional initial simplification | INCORPORATED |
| IAR-027-O03 | §8: pre-existing buffer vs. newly-introduced drain/bounding distinguished | INCORPORATED |
| IAR-027-O04 | §9: no-`Result` interface's forward constraint on future retry-capable implementations disclosed | INCORPORATED |
| NRR-027-F01 | §28 GOV-013 citation corrected; see IAR-027-O01 row | ADDRESSED |
| NRR-027-O01 | Constitutional-guarantee-vs-completeness distinction | NON-BLOCKING OBSERVATION, carried forward (§33); not incorporated into normative text |
| NRR-027-O02 | DROP-OLDEST recency-vs-causal-value tension | NON-BLOCKING OBSERVATION, carried forward (§33) |
| NRR-027-O03 | O04's forward-looking "interface evolution" phrasing | NON-BLOCKING OBSERVATION; substance present via §9, not separately elaborated |

## 40. Approval Record

**Founder Architecture Approval: GRANTED WITH NON-BLOCKING OBSERVATIONS.**

**Founder:** Denver Jacobs
**Date:** 2026-08-12
**Decision, verbatim (obtained via direct, explicit response, not inferred):** "Grant approval, explicitly noting NRR-027-O01/O02/O03 (and any other observations) as acknowledged but non-blocking, carried forward rather than resolved."

**Carried-forward observations, explicitly not resolved by this approval:**
- `NRR-027-O01` — the constitutional-guarantee-vs-audit-completeness distinction could be stated more explicitly in a future revision.
- `NRR-027-O02` — DROP-OLDEST's recency-priority rationale trades off against the causal/investigative value of older events in some scenarios.
- `NRR-027-O03` — a future retry-capable `AuditPipeline` implementation would require the `consume` interface to evolve; not designed for here.

This approval covers the architecture defined in §1–§37 above in full, including the bounded-buffer/DROP-OLDEST policy, best-effort/at-most-once delivery semantics, and all other sections. It does not resolve, and does not need to resolve, the three carried-forward observations above — they remain open for a future revision at the Founder's own discretion, not blocking this approval.

## 41. Implementation Authorization Status

### 41.1 At Filing (v0.2.1, 2026-08-12) — Historical

**NOT GRANTED.** Founder Architecture Approval (§40) authorized the architecture only. It did not authorize:

- modifying `synapse-runtime` implementation code;
- modifying `AuditEmitter`, `AuditPipeline`, or `FileAuditPipeline`;
- modifying Cargo manifests or `Cargo.lock`;
- modifying or adding tests;
- reconciling, committing, or pushing the preserved candidate implementation already sitting uncommitted in `synapse-runtime`;
- beginning Independent Implementation Review;
- any other implementation-stage activity.

The preserved candidate implementation predated this document's own corrections (§30) and remained untouched, uncommitted, and unauthorized for commit. Implementation authorization was recorded as a separate, future Founder decision — **EWO-027 — Implementation Lifecycle Initiation / Authorization** — not granted by the v0.2.1 Filing.

### 41.2 Final Founder Implementation Acceptance (v0.3.0, 2026-08-12) — GRANTED

Implementation authorization was subsequently granted (session-level, not itself separately filed) and the implementation was carried out in an isolated `synapse-runtime` worktree off fresh `origin/main` — never against the preserved candidate implementation named in §41.1, which remains untouched, uncommitted, and still unauthorized for commit. The complete implementation proceeded through four working-management stages, informally labeled "Unit 1" (`AuditEmitter` bounded buffer), "Unit 2" (`AuditPipeline`/`FileAuditPipeline`), "Unit 3" (Runtime integration), and "Unit 4" (remaining §26 acceptance-test closure) purely for session/engineering-management convenience — **this architecture does not define, and has never defined, "Units"; that label carries no architectural authority** — plus one Narrow Implementation Correction (§26 item 12), each independently reviewed, and each individually Founder-accepted before the next began:

| Commit | Description |
|---|---|
| `3472eb7792b863eb05009376fc8c314f94b6ce83` | `AuditEmitter` bounded buffer, DROP-OLDEST, `dropped_due_to_capacity` (Founder-accepted) |
| `383f8841d50fc994015b208484397109e9ba7ff0` | `AuditPipeline`/`FileAuditPipeline` service (Founder-accepted) |
| `47da4ecfb619ffa0bf242c4a9146bc543bc0dd7a` | Runtime integration (`with_audit_pipeline`, `flush_audit_events`) (Founder-accepted) |
| `2f2eda85bee72c86372f6f34487a4e63ccc66bd0` | §26 acceptance-test closure, items 11–13 |
| `9fa5cbb459a6aec921b4b32fc71b8105964087e2` | §26 item 12 Narrow Implementation Correction — final commit |

**§26 Acceptance Status: 14 of 14 items satisfied**, independently re-derived across the complete current test suite (not assumed from the individual stage acceptances above), confirmed by the Independent Implementation Re-Review of `9fa5cbb459a6aec921b4b32fc71b8105964087e2`: PASS, zero open Critical/Major/Minor finding, zero unresolved observation.

**Final Founder Implementation Acceptance — GRANTED.** Denver Jacobs, Founder, 2026-08-12, recorded verbatim:

> "ACCEPT — Grant Final Founder Implementation Acceptance for EWO-027 v0.2.1 implementation."

This acceptance covers the complete implementation through `9fa5cbb459a6aec921b4b32fc71b8105964087e2`. No additional EWO-027 implementation unit is pending or authorized. The architecture (§1–§37) remains Approved and unaltered by this acceptance.

**EWO-027 is CLOSED / COMPLETE.**

## 42. Exact Next Lifecycle Stage

**EWO-027 itself has no further lifecycle stage — it is closed (§41.2).** This EWO is unrelated to, and does not gate or unblock, any other work order.

The next mandatory SynapseOS lifecycle stage, independent of EWO-027, per current repository evidence (`ARCH-017`, `EWO-029.5`, `EWO-029.5` §36/§37):

1. **`EWO-029.5` implementation completion** — the `synapse-observation-wire` crate (`synapse-runtime` commits `8bd0aab`, `0cb1a24`, `4d8d32d`) is committed but not yet integrated into `runtime`/`sdk`, and has no recorded Founder Implementation Acceptance or Engineering Report.
2. **`EWO-029.6`** — formal cross-platform (Windows/Unix/macOS) conformance testing, named but not yet started (`EWO-029.5` §33).
3. Only once `ARCH-017`'s own implementation (items 1–2) is complete does **`EWO-028`** (Draft, unfiled — Phase 1 Read-Only Control Centre GUI Engineering Work Order) unblock (`EWO-029.5` §36; `ARCH-017` frontmatter), and proceed through its own remaining lifecycle (Correction of `IER-028-F01`/`F02` → Re-Review → Founder Approval → Implementation).

`ARCH-015` (Developer Platform Boundary), `ARCH-016` (Control Centre Foundation, Phase 1 read-only), and `ACT-005` (Developer Platform Era) are all already Approved and are not blockers. GUI technology selection and Control Centre repository creation remain open but are not separately-gated prerequisite work — they fall within `EWO-028`'s own lifecycle.
