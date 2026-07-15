---
document_id: ARCH-006
title: Runtime Actor Execution Architecture
project: SynapseOS
specification: SynapseOS — genuine actor execution, Runtime-owned emitted-message admission, causation, and authority resolution, and the bootstrap-grant mechanism, realizing the architecture the Runtime Actor Execution Architecture Review independently reconstructed from already-completed, already-verified implementation
version: 0.1.3
status: Draft
author: Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-15
last_updated: 2026-07-15
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-010 (Approved, Act 2)
    - GOV-004 (Engineering Principles)
  standards:
    - STD-001 (Approved)
  architecture:
    - ARCH-000 (Draft — whole-system introduction)
    - ARCH-001 (Draft — constitutional foundation)
    - ARCH-002 (Draft — Runtime architecture; §6, §7, §8, §9, §11, §13, §21, §22 directly realized by this document)
    - ARCH-003 (Draft — Runtime integration status; §5, §18 record the exact conformance gaps this document's milestone closed)
    - ARCH-004 (Draft — Local Actor Supervision Architecture; the first later milestone independently confirmed to depend on this one)
    - ARCH-005 (Draft — Temporal Runtime Architecture; the second later milestone independently confirmed to reuse this one's own admission pattern verbatim)
  rfcs: None
  adrs:
    - ADR-0015 (Approved — Audit Emitter Failure Semantics)
    - ADR-0016 (Approved — Trusted Core Interaction Rule)
    - ADR-0017 (Approved — Bootstrap Capability Trust Root)
  roadmap: None
  research: None
  operational: None
  reports:
    - ER-005 (Truthful Actor Execution-State Tracking — verified predecessor baseline)
    - ER-006 (Bounded Actor Mailboxes — verified predecessor baseline; its own final workspace test count, 232, is direct evidentiary basis for this document's historical placement, §12)
    - ER-007 (Local Actor Supervision — its own stated starting baseline, 423 tests, is the second direct evidentiary basis for this document's historical placement, §12)
    - ER-008 (Temporal Runtime — independently confirms this document's admission pattern reused verbatim, §11)
  source_artifacts:
    - synapse-runtime @ 5ccc7f9083a71adc6ee704b2322a701935765679 (working tree, including uncommitted EWO-006, this milestone's own implementation, EWO-007, and EWO-008)
  review: "SynapseOS Publication Recovery Review; SynapseOS Architecture Review — Capability-Authorized Actor-to-Actor Messaging Runtime; SynapseOS Architecture Review — Runtime Actor Execution Architecture (all three, this engineering effort; primary analytical basis for this document, restated faithfully per this document's own Historical Reconstruction Notice, §2)"
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ARCH-006 — Runtime Actor Execution Architecture

*Filename pattern: `ARCH-006-Runtime-Actor-Execution-Architecture.md` (per STD-001 §7–§8).*

> The canonical source for this document is the Markdown file stored in the official `JacobsSystems/synapse-docs` repository. Exported formats are derivative artifacts unless formally designated otherwise.

> **AI Assistance:** `Drafting` — AI-assisted content in this document remains subject to human ownership, review, validation, and approval under STD-001 (§33, AI-Assisted Documentation). AI output is not automatically authoritative.

## 1. Document Control and Status

| Field | Value |
|---|---|
| Document ID | ARCH-006 |
| Title | Runtime Actor Execution Architecture |
| Version | 0.1.3 |
| Status | **Draft** — this document's own governance status; no approval act has occurred |
| Implementation status | **Already implemented, already independently reviewed, already exercised by existing demonstrations and tests** — see §2 |
| Author | Denver Jacobs |
| Approval authority | Chief Architect (Class B, per GOV-010 §5), vacant; Founder (interim) |
| Created | 2026-07-15 |
| Classification | Public |

This document is Draft in the same governance sense every other document in this corpus is Draft: it has not completed the GOV-003/GOV-010 approval process, and nothing in it is operative or binding until it does. This is unrelated to, and must not be confused with, the *implementation status* recorded immediately above and restated throughout §2 — the architecture this document records is not proposed, pending, or hypothetical; it already exists, in full, in `synapse-runtime`'s current working tree, and has already been independently verified three separate times (Publication Recovery Review; Capability-Authorized Actor-to-Actor Messaging Runtime Architecture Review; Runtime Actor Execution Architecture Review — all cited in full, §18).

## 2. Historical Reconstruction Notice

**This is a historical reconstruction, not a prospective architecture proposal.**

- **The implementation predates this document.** Every capability this document describes — genuine `Actor::handle()` execution, the shared admission pipeline, Runtime-owned causation and authority resolution, and the bootstrap-grant mechanism — already exists in `synapse-runtime`'s current working tree (base commit `5ccc7f9083a71adc6ee704b2322a701935765679`), was implemented before this document was authored, and continues to run unmodified as this document is written.
- **The implementation has already been independently reviewed.** Three successive, independent reviews — the Publication Recovery Review, the Architecture Reconstruction Review ("Capability-Authorized Actor-to-Actor Messaging Runtime"), and the Runtime Actor Execution Architecture Review — each verified this milestone's own behaviour directly against source, tests, and runtime execution, not against any prior summary or this document itself.
- **This document records the architecture as reconstructed from that evidence.** Every architectural claim below traces to a specific, cited piece of independently-verified evidence — a source line, a test, a doc comment, an existing Engineering Report's own stated figures — never to assumption, inference from a filename, or restatement of a prior review without independent grounding.
- **This document authorizes no additional implementation.** It closes a governance gap; it does not open one. No Engineering Work Order is authorized, implied, or required by this document's own text to realize anything it describes — the described architecture is already fully realized.

Where this document's own text distinguishes reconstructed historical fact, independently verified implementation behaviour, and architectural conclusion, that distinction is preserved from its own primary sources rather than reintroduced here from scratch — this document restates the Runtime Actor Execution Architecture Review's own conclusions faithfully, per that review's own status as this document's primary architectural source, and does not reinterpret them.

## 3. Purpose

This document defines the authoritative architecture for **the Runtime's first realization of genuine actor execution**: which component invokes actor-defined logic, how the resulting emitted messages become further Runtime work, who establishes their causation and authority, and how a genuinely external caller reaches any of it. It resolves, retroactively, the architectural record for the exact gap ARCH-003 disclosed unchanged across four of its own revisions (v0.1.0 through v0.4.0): *"No claim is made anywhere in this section that the current runtime performs actor logic... no actor-defined message-handling logic exists anywhere in the workspace."*

This document does not select numeric policy, does not define implementation APIs beyond what is necessary to record already-existing ones truthfully, and does not authorize implementation — consistent with ARCH-002's own stated method (§1: "precise enough that independent engineering teams could implement recognizably conformant runtimes without identical internal code"), applied here to a document that records rather than proposes.

## 4. Scope

**In scope:** the architectural placement of genuine actor-execution invocation within the existing Execution Coordinator/Runtime relationship; the treatment of an actor's own emitted messages as fresh admission requests, never already-sent facts; Runtime-owned causation and authority resolution for those requests; the single, shared admission pipeline every message origin converges through; the bootstrap-grant mechanism and its bounded relationship to ADR-0017; the resulting truthful execution-ordering guarantees; and Runtime's own orchestration responsibility across all of the above.

**Out of scope:** supervision, restart, and failure-escalation policy (ARCH-004's own domain, realized afterward and independently confirmed to depend on this document, §15); timers, delayed execution, and time observation (ARCH-005's own domain, independently confirmed to reuse this document's admission pattern verbatim, §11, §15); persistence, durable mailboxes, workflow orchestration, effect scheduling, networking, distributed or clustered execution, service discovery, and remote execution (§9, all explicitly excluded and explained). See §9 for the complete exclusions statement and §13 for the future-compatibility boundary.

## 5. Non-Goals

This document does not define, and takes no position on:

- restart, supervision policy, or failure-escalation strategy of any kind (ARCH-004's own domain);
- timers, delayed execution, or any notion of "not yet" (ARCH-005's own domain);
- message retry, redelivery, or acknowledgement protocols;
- dead-letter queues or dead-letter storage;
- durable or persistent mailboxes, or any mailbox content transfer mechanism;
- state snapshots, event sourcing, or checkpoint recovery;
- distributed supervision, remote failure detection, node monitoring, or clustering;
- workflow compensation or generalized effect-retry frameworks;
- cluster coordination, multi-host scheduling, or remote execution of any kind;
- service discovery beyond the existing, unmodified Actor Directory contract ARCH-002 §6 already establishes;
- any Rust struct, trait, enum, method signature, function name, or field layout beyond what is necessary to record already-existing ones;
- any new Trusted Core component (ARCH-002 §5–§6 is unchanged; see §7);
- any new lifecycle state beyond ARCH-002 §15's existing set;
- any new constitutional guarantee beyond ARCH-001's four (non-forgery, integrity, enforcement-at-invocation, revocation-state enforcement).

## 6. Existing Architectural Context

This document amends no prior authority. It extends, and is bound by, the following without redefinition:

- **ARCH-000** established SynapseOS's whole-system introduction; this document inherits it without restatement.
- **ARCH-001** fixes the four constitutional guarantees. This document introduces no new guarantee; §8 records, for each invariant this milestone's implementation embodies, whether it reaffirms, extends, or newly establishes an application of those four.
- **ARCH-002** remains the sole authority for Trusted Core component responsibility (§6), the Constitutional Execution Cycle (§11), the message model's four-way content/envelope/authority/transfer distinction (§8), the mailbox model (§13), and the Minimal Runtime Profile's own conformance requirements (§21, §22). This document amends none of it; it records the first implementation to genuinely satisfy §21's "one-message actor execution" and "private actor-state transition" requirements, and the first to exercise §8's own distinction for a message an actor itself produces rather than one an external caller submits.
- **ARCH-003** recorded, across four consecutive revisions (v0.1.0–v0.4.0), that "no claim is made... that the current runtime performs actor logic" and listed "a complete actor execution flow, including actual actor-defined message-handling logic being invoked during dispatch" and "a first runnable actor" among its own Deferred Integration Work. This document records the closure of exactly those two items, evidenced independently (§8), not asserted from ARCH-003's own prior text.
- **ARCH-004** established the Supervisor precedent — a new, narrow, Runtime-composed service, reached only through Runtime — and is independently confirmed, by direct citation in its own governing Engineering Work Order, to depend on this document's own `execute_message_capturing` dispatch-failure branch as its sole point of observation (§15).
- **ARCH-005** established Temporal Runtime and is independently confirmed to reuse this document's own `resolve_emitted_message_authority` and `admit_message` functions verbatim, unmodified, for a third message origin (§11, §15).
- **ADR-0015** (Audit Emitter Failure Semantics) governs every audit-emission failure behavior this document's own admission and dispatch flow assumes (§10): a mandatory audit emission that fails causes the *reporting* operation to fail, without rollback of already-committed component-level state.
- **ADR-0016** (Trusted Core Interaction Rule) is the specific authority this document depends on most directly: Runtime alone connects every component this document's own execution model touches; no direct peer-interaction path exists anywhere in the implementation this document records (independently verified, §7).
- **ADR-0017** (Bootstrap Capability Trust Root) establishes that exactly one Bootstrap Capability is created, once, during Runtime bootstrap, never exposed through any public interface. §12 of this document depends on this directly: the bootstrap-grant mechanism this document records is the disclosed, bounded extension of ADR-0017's own anticipated bootstrap-time issuance seam, never a second, competing one.
- **ER-005, ER-006** record the verified predecessor implementation state (truthful execution-state tracking; bounded mailboxes) this document's own milestone was built directly on top of, evidenced quantitatively in §15.
- **ER-007, ER-008** record the two later milestones independently confirmed to depend on this document's own architecture, cited by name in their own governing text (§15).
- **Governance (GOV-003, GOV-010)** and **STD-001** govern this document's own review, approval, and evidentiary process; they are not architectural inputs to its content.

## 7. Architectural Principles

This document is governed by, and introduces no exception to, the following principles, each already established by the documents in §6:

- **Mechanism/policy separation** (ARCH-001 §9, applied concretely by ARCH-002 §5): every component this document's execution model touches continues to own exactly the mechanism ARCH-002 §6 already assigns it; this document transfers no responsibility between components (§7 below).
- **Sole-composer discipline** (ADR-0016 Rule 1): Runtime alone connects any two components this document's execution model spans; independently verified — no component this document touches references any other directly (§10, §11).
- **Non-forgery, extended from identity to authority claims** (ARCH-001 §5.3): an untrusted actor's own self-declared sender, causation, or authority claim carries no weight; Runtime alone, from its own already-established truth, determines each (§8, §10).
- **Fresh, never-cached validation** (ARCH-002 §9): capability validity for an emitted message is resolved at the exact moment of emission, from Capability Authority's own current registry state, never cached or assumed from any earlier point (§8, §10).
- **Truthful ordering** (ARCH-002 §11/§12, extended by ARCH-004 §18's own discipline): no audit record, and no downstream admission attempt, may occur before the fact it depends on has genuinely, truthfully occurred (§10).

## 8. Runtime Evidence Verified

Every claim below was independently re-confirmed by direct source inspection across the three governing reviews (§2), not assumed from any one review's own text alone.

| Claim | Verification | Result |
|---|---|---|
| Prior absence of genuine actor execution | ARCH-003 v0.1.0–v0.4.0, unchanged across four revisions: "no actor-defined message-handling logic exists anywhere in the workspace" | Confirmed — this document's own milestone is the first to close this gap |
| Genuine `Actor::handle()` invocation | `runtime/src/lib.rs:1697-1815`, `execute_message_capturing`: obtains `behavior_mut(&instance)` from Actor Host, passes it into `Execution Coordinator::dispatch`, which genuinely invokes it | Confirmed — no simulation, no placeholder, no bypass |
| Emitted messages treated as admission requests | `Runtime::process_emitted_messages` (`runtime/src/lib.rs:1104`) and `Runtime::resolve_emitted_message_authority` (`runtime/src/lib.rs:1033`) | Confirmed — each emitted message independently, freshly authorized and admitted |
| Single shared admission pipeline | `admit_message` (`runtime/src/lib.rs:979-1001`) called identically by `submit_message` (external origin) and `process_emitted_messages` (actor-emitted origin) | Confirmed — one function, two call sites, no divergence |
| Deterministic capability resolution | `resolve_emitted_message_authority`: exactly one structurally-matching, currently-valid, currently-bound capability required; zero rejected `EnforcementDenied`, more than one rejected `AmbiguousAuthority` (new `RuntimeError` variant, confirmed the sole new variant this milestone adds) | Confirmed — no arbitrary tie-breaking exists anywhere |
| Runtime-owned causation | `process_emitted_messages`, line 1112: `message.causation` unconditionally overwritten with the Runtime-known `triggering_message` | Confirmed — an actor's own self-declared causation claim is never trusted |
| Bootstrap grants | `BootstrapGrant`, `RuntimeBootstrapConfig`, `BootstrapGrantSet`, `Runtime::bootstrap_with_config` (`runtime/src/lib.rs:440-596`, `758-`) | Confirmed — minted once, through the existing `issue_capability` path, before `runtime.started` is audited |
| Execution ordering | `execute_message_capturing`: dispatch sequenced strictly between EWO-005's `begin_execution` and `complete`/`complete_execution`; emitted-message processing occurs strictly after `execution.completed` is already emitted | Confirmed by direct statement-order inspection |
| Reuse by Temporal Runtime | ER-008, independently authored: `process_due_timers` calls `resolve_emitted_message_authority` then `admit_message`, the identical two calls `process_emitted_messages` already makes | Confirmed — the admission pattern generalizes without modification |
| Reuse by Local Actor Supervision | EWO-007 §"Problem Statement"/"Component Boundaries": names "this session's own admission pipeline (`admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`)" as pre-existing baseline it must not disturb | Confirmed by direct citation in EWO-007's own governing text |
| Historical placement, quantitative | ER-006's own final workspace total: 232 tests. ER-007's own stated starting baseline: 423 tests. Gap of 191 tests attributable to no other governed milestone. | Confirmed — independent, cross-report arithmetic corroborates the textual-citation-based ordering |

No contradiction between the three governing reviews and the current implementation was found anywhere. This document proceeds on that basis.

## 9. Component Architecture

### 9.1 Responsibility table

| Component | Responsibility (unchanged from ARCH-002 §6) | New responsibility from this document |
|---|---|---|
| **Runtime** | Sole cross-component composer (ADR-0016 Rule 1) | Sole orchestrator of dispatch, emitted-message processing, causation, and authority resolution for this milestone's entire scope; decides no capability validity or identity itself — only sequences other components' own decisions. Gains its own primary caller-facing execution-driving surface, `step()` and `run_until_idle()` — both confirmed genuinely new to this milestone (absent from the committed predecessor state), not pre-existing methods whose signature merely remained stable (§10) |
| **Execution Coordinator** | Constructs Execution Context; performs dispatch mechanics; enforces one-owner execution | Genuinely invokes the supplied behavior — a real value to dispatch *to*, not a relocation of dispatch mechanics — and now bears the responsibility of surfacing what that invocation emits, so Runtime can treat it as further work (§10); this is a genuine widening of dispatch's own engineering surface, not merely an added parameter (implementation-level detail — the exact shape of that surface — is EWO-009's concern, not this document's, on the same basis §9.1's Scheduler treatment already establishes). Gains `fail`, the truthful counterpart to `complete` for a genuinely failed dispatch — closing the exact Execution-Coordinator-side cleanup gap EWO-005/ER-005 explicitly disclosed and left open ("Execution Coordinator today has no mechanism to cleanly cease context liveness after a dispatch failure... This EWO does not authorize adding one") |
| **Actor Host** | Actor identity, instance/mailbox/behavior ownership | Supplies the mutable behavior value (`behavior_mut`) for genuine dispatch; gains `create_instance_with_behavior`, the mechanism by which an instance is given real, genuine behavior at creation — the foundational capability genuine execution requires; gains `dequeue`, retrieving a ready instance's next message for dispatch, whose own further-work signal is what this document's own ordering guarantees (§10) depend on; `enqueue`/`live_instance` reused unmodified for the admission path |
| **Capability Authority** | Validates, binds, attenuates, delegates, revokes capabilities | Gains exactly one new public accessor, `bound_capabilities`, returning an actor's current bindings in deterministic order — the one Trusted Core crate whose own public surface this document's milestone extends with a single addition |
| **Message Gateway** | Enforces envelope integrity; validates send authority before mailbox admission | None new — `validate_envelope`/`validate_send_authority`/`admit` reused unmodified for both message origins; unaware of origin |
| **Scheduler** | Decides dispatch order among ready actors | Receives its own first genuine implementation of this already-established ARCH-002 §6 responsibility — no new architectural principle; Scheduler's own role (ready-order policy only, unaware of lifecycle or capability state) is unchanged. Implementation-level detail (the specific trait shape and internal data structure) is recorded in EWO-009, not here, on the same basis EWO-006's own bounded-mailbox implementation required no dedicated architectural treatment beyond ARCH-002 §13's already-sufficient authority |
| **Lifecycle Guardian** | Enforces legal lifecycle-state transitions | None new — EWO-005's own `begin_execution`/`complete_execution` sequence is reused unmodified, with this document's own dispatch inserted between the two calls |
| **Audit Emitter** | Emits minimal, unbypassable audit events | None new beyond a new call site — the existing `message.admitted`/`message.rejected`/`execution.completed` events are reused, unmodified, for a new message origin |

### 9.2 Non-responsibilities, explicitly confirmed

- Execution Coordinator never decides *whether* a resulting message is authorized — dispatch mechanics only.
- Actor Host never validates capability and never decides admission — identity, isolation, and mailbox mechanics only.
- Capability Authority never initiates a resolution itself — purely queried, fresh, per call, exactly as ARCH-002 §9 already requires generally.
- Message Gateway has no actor-lifecycle or message-origin awareness by design, and gains none here.
- Scheduler remains lifecycle- and origin-unaware, and gains no dependency on any other component — its architectural isolation (no path to Actor Host, Capability Authority, or Lifecycle Guardian) is unaffected by receiving its own first genuine implementation (§9.1), confirmed by its own unchanged `synapse-common`-only dependency set; the implementation's own shape is EWO-009's concern, not this document's.
- Lifecycle Guardian is never called by this document's own new code directly — only by Runtime, exactly as EWO-005 already established.

### 9.3 Runtime as sole orchestration point

Independently verified, not merely asserted: every one of `execute_message_capturing`, `process_emitted_messages`, `resolve_emitted_message_authority`, and `admit_message` is a private-or-Runtime-scoped method; no other crate's own source references any of the four. Runtime remains the sole cross-component composer this document's execution model requires (ADR-0016 Rule 1, reaffirmed without exception).

## 10. Runtime Execution Model

The complete, coherent architectural flow this document establishes, from external submission or genuine dispatch through to completion.

**Entry points.** Every genuine execution this milestone performs is ultimately driven by one of two caller-facing methods this milestone introduces: `Runtime::step()` (one bounded unit of work) and `Runtime::run_until_idle()` (repeated `step()` calls until no work remains or a caller-supplied bound is reached) — both confirmed genuinely new to this milestone (absent from the committed predecessor state entirely), not pre-existing methods whose signature happened to remain stable. Internally, each `step()` call resolves the next ready instance's next message through Actor Host's own new `dequeue` operation (§9.1); `dequeue`'s own truthful further-work signal is what determines whether Runtime re-marks that instance ready for a subsequent `step()` — the concrete mechanism realizing this section's own ordering guarantees, below. No other public method causes actor-defined logic to execute.

```text
External caller                              Actor::handle() genuinely executes
  |                                                        |
  v                                                        v
Runtime::submit_message                         Runtime::execute_message_capturing
  |                                                        |
  v                                                        v
Message Gateway: validate_envelope,             Execution Coordinator: dispatch(context,
  validate_send_authority                         message, behavior) -- GENUINE INVOCATION
  |                                                        |
  v                                                        v
Capability Authority: validate(presented)       Lifecycle Guardian: complete_execution
  |                                              (EWO-005's own sequence, unmodified)
  v                                                        |
Actor Host: live_instance                                  v
  |                                              Host Adapter: release_execution_handle
  v                                                        |
        \                                                  v
         \                                       Audit Emitter: execution.completed
          \                                                |
           \                                               v
            \                                   Runtime::process_emitted_messages
             \                                   (sequential, non-atomic, per message)
              \                                             |
               \                                            v
                \                                Runtime::resolve_emitted_message_authority
                 \                               (sender's own bound capabilities, resolved
                  \                                fresh; exactly one match required)
                   \                                        |
                    \                                       v
                     \___________________________> Runtime::admit_message
                                                    (THE SINGLE SHARED PIPELINE, §11)
                                                              |
                                                              v
                                              Message Gateway -> Capability Authority ->
                                                Actor Host -> Scheduler: mark_ready
                                                              |
                                                              v
                                              Audit Emitter: message.admitted / rejected
```

**Ordering guarantees, independently verified at the statement level:**

1. Dispatch (this document's own contribution) is sequenced strictly after EWO-005's `begin_execution` succeeds and strictly before EWO-005's `complete`/`complete_execution` runs — an actor's own handler executes only while Lifecycle Guardian's tracked state is genuinely `Executing`, never before or after.
2. Emitted-message processing occurs only after the triggering execution has fully, truthfully completed — `execution.completed` already emitted, Lifecycle Guardian already exited `Executing` — confirmed at `runtime/src/lib.rs:1801-1814`. An emitted message can never become Scheduler-eligible while its own emitting actor is still considered executing.
3. Causation is set, unconditionally, by Runtime — never by the emitting actor's own self-declared claim.
4. Authority is resolved, fresh, at the moment of emission — never cached, never inherited from registration or any earlier point.
5. Admission occurs through the single shared pipeline (§11) regardless of origin — no second path exists anywhere.
6. Audit ordering is per-message and independent — each emitted message's own `message.admitted`/`message.rejected` fires as its own outcome becomes truthfully known, never batched or deferred past that point.
7. `step()` and `run_until_idle()` are the sole caller-facing entry points into this entire model — confirmed by direct source inspection, no other public method causes `Actor::handle()` to be invoked.

## 11. Shared Admission Architecture

**Normative statement: every message, regardless of origin, converges through one identical, Runtime-owned admission pipeline (`admit_message`) before becoming eligible for scheduling. No second admission path exists, has ever existed, or may be introduced by any future extension without itself becoming a new architectural decision subject to the same rigor as this document.**

This is established here as a **constitutional Runtime property**, not merely a present implementation convenience: if a second, origin-specific admission path existed, ARCH-002 §22's own mandatory conformance requirements ("audit emission for every security-relevant event," "authority only through presented capability, never ambient") would hold only for messages that happened to take the audited path — any origin routed around it would silently escape every guarantee this architecture claims to hold universally.

**Comparison across every message origin, independently verified:**

| Origin | Envelope/authority validation | Capability resolution | Admission call | Audit |
|---|---|---|---|---|
| External submission (`submit_message`) | Message Gateway, same code | Caller-presented `Capability`, validated fresh | `admit_message` | `message.admitted`/`rejected`, same events |
| Actor-emitted (this document's own milestone) | Message Gateway, same code | `resolve_emitted_message_authority` (emitting actor's own bound set, resolved fresh) | `admit_message` — identical call, same function | same events |
| Timer-generated (ARCH-005, later) | Message Gateway, same code | `resolve_emitted_message_authority` — the identical function this document's milestone introduced, reused verbatim | `admit_message` — identical call | same events |

**This document establishes the architectural pattern ARCH-005's Temporal Runtime later reused, verbatim, without modification** — independently confirmed: ER-008 records `process_due_timers` calling `resolve_emitted_message_authority(&entry.actor, &message)` then `admit_message(&message, &capability)`, the exact two calls `process_emitted_messages` already makes. Any future milestone introducing a new message origin (a durable-timer replay, a workflow-orchestration actor's own emission, an effect-runtime consumer) inherits this same convergence requirement without needing to redesign it — the pipeline itself is already origin-agnostic by construction, not merely by accident.

## 12. Bootstrap Architecture

**Purpose.** To give a genuinely external, public-API-only caller a way to obtain a *first* capability — without which no capability-authorized Runtime operation of any kind, including the genuine execution and admission this document records, would ever be reachable from outside the crate.

**Trust boundary.** Bootstrap grants are minted through the *same* `issue_capability` path every other issued capability already uses; the only thing this mechanism changes is *who* the issuer is (the Bootstrap Capability itself, held only same-crate, per ADR-0017) and *when* issuance happens (once, during the one-time bootstrap act, before `runtime.started` is audited and before the transition to `Running`).

**Lifecycle.** Declared before `Runtime::bootstrap_with_config` runs; minted exactly once during that call; never repeatable afterward. `bootstrap_with_config` never returns a usable `Runtime` value if grant issuance itself fails, so no partially-configured Runtime is ever observable by a caller.

**Limits.** `MAX_BOOTSTRAP_GRANTS = 16` — a documented, non-configurable, fixed implementation bound, whose own doc comment explicitly cites "the same basis `core/actor-host`'s own `MAILBOX_CAPACITY` is a policy choice, not an architectural guarantee" (independent, direct textual evidence this document's milestone postdates EWO-006, §15). Duplicate grant names are rejected before any capability is minted (`IntegrityViolation`); an empty operations set is rejected at grant-construction time, before bootstrap ever runs.

**Interaction with Capability Authority.** None beyond the ordinary `issue_capability` call every other issuance already uses. Capability Authority is not itself modified, extended, or made aware that a "bootstrap grant" concept exists; from its own perspective, a bootstrap grant is indistinguishable from any other issuance.

**Relationship to ADR-0017.** Bootstrap grants are the direct, disclosed extension of the exact mechanism ADR-0017 §6 itself anticipated: *"some mechanism must exist by which the Runtime's bootstrap sequence causes the one Bootstrap Capability to come into existence, distinct from the ordinary `issue` path — the concrete form of that mechanism is an implementation decision."* Having already established a bootstrap-time issuance seam for the root itself, this document's milestone reuses the identical seam, at the identical moment, for ordinary (non-root) grants — never inventing a second seam.

**Why bootstrap grants strengthen, rather than weaken, the Runtime security model:**

1. The Bootstrap Capability itself is never returned, exposed, or reachable through this mechanism — only ordinary, ceiling-scoped capabilities it mints are ever visible to the caller.
2. The set of grants is fixed *before* any `Runtime` value exists, so no code running after bootstrap can choose what gets minted.
3. Every minted grant is subject to the identical non-amplification ceiling every other issued capability already carries — bootstrap grants earn no more authority than an ordinary `issue_capability` call could grant with the same issuer.
4. `issue_capability` remains the only *repeatable* issuance path after bootstrap — this mechanism adds no second, ongoing route to new authority.

Without this mechanism, every capability-gated demonstration of this document's own architecture would be permanently confined to same-crate test-only shortcuts, which would itself be the weaker security posture — an architecture whose guarantees are only ever exercised by code that already has privileged access to construct them.

## 13. Constitutional Runtime Invariants

| Invariant | Status |
|---|---|
| Runtime-owned execution | **First established here.** No prior milestone had Runtime cause actor-defined logic to run at all. |
| Runtime-owned causation | **Extends** ARCH-001 §5.3's existing non-forgery principle (previously applied to identity) to a new fact class (message lineage) — not a new guarantee, a new application of one. |
| Runtime-owned authority resolution | **Extends** ARCH-002 §9's "bindings are queried fresh, never cached" rule to a new query origin (an emitting actor, not merely a message presenter). |
| Shared admission pipeline | **Extends** the general Runtime-as-sole-composer discipline (ADR-0016 Rule 1) to a second, and later a third, message origin. |
| Fresh capability validation (never cached, never registration-time) | **Already existed** as a Capability Authority-level guarantee (ARCH-002 §9); this document's milestone is the first *consumer* to exercise it for actor-emitted authority. |
| Runtime as sole composer | **Already existed** (ADR-0016 Rule 1); reaffirmed, not newly established. |
| Truthful emitted-message lineage | **First established here** — no prior architecture text addressed what an actor's own emitted message's causation should be, because no prior implementation ever produced one. |
| Non-forgery (applied to authority claims, not merely identity) | **Extends** ARCH-001's existing constitutional guarantee to cover a previously-untested case: an untrusted actor's own self-asserted authority. |
| Deterministic authority resolution (exactly one match required; zero or multiple both rejected, never arbitrarily chosen) | **First established here.** `AmbiguousAuthority` is a genuinely new invariant — no prior mechanism resolved authority against a *set* of candidates, so no prior mechanism needed a tie-breaking rule at all. |
| External reachability (via bootstrap grants) | **First established here** for any capability-authorized operation beyond bare Runtime bootstrap itself — extends ADR-0017's exactly-once bootstrap-act discipline to ordinary (non-root) capability issuance occurring *within* that same act. |

No invariant recorded above overstates its own novelty: five of the ten reaffirm or extend already-existing architecture; five are newly established by this document's own milestone, each named as such explicitly.

## 14. Explicit Exclusions

Independently confirmed absent from this milestone's own implementation, each excluded for a distinct, evidenced architectural reason:

| Excluded | Why |
|---|---|
| Supervision | Requires a policy decision (restart/stop/escalate) this milestone's own scope never touches — a failing `Actor::handle()` returns `Err`, handled by Execution Coordinator's own truthful failure-cleanup mechanics (§9.1; not a pre-existing path — genuinely new to this milestone), with no restart, escalation, or policy decision applied to it; introduced afterward, cleanly, by ARCH-004 |
| Retries | Requires a redelivery concept ARCH-002 explicitly defers as policy; `process_emitted_messages`'s own rejection handling is terminal per message |
| Restart strategies | Requires the identity model (`ActorId` vs `ActorInstanceId`) this milestone never references at all |
| Timers | Time observation is a structurally separate mechanism (ARCH-001 §6's own "foundational Runtime mechanisms" list); zero clock reference anywhere in this milestone |
| Persistence | No durability contract exists anywhere in the Minimal Runtime Profile this milestone completes; nothing in `RuntimeBootstrapConfig`, `BootstrapGrantSet`, or emitted-message outcomes survives beyond the in-process `Runtime` value |
| Durable mailboxes | Orthogonal to genuine execution; mailbox contents remain exactly as EWO-006 left them — in-memory, lost on termination |
| Workflow runtime | Would require a policy layer above ordinary message admission this milestone never introduces |
| Effect runtime | No generalized effect-scheduling abstraction; this milestone's own logic is specific to "a message `Actor::handle` itself returned," nothing more general |
| Networking | Entirely in-process; no I/O anywhere in the implementation |
| Distributed runtime | `admit_message` resolves purely local `ActorInstanceId`s; no location or transport concept exists |
| Clustering | Not addressed, not partially designed, not implied by any structural choice in the implementation |
| Service discovery | Actor Directory (a separate, pre-existing replaceable service, ARCH-002 §6) is untouched; no new lookup mechanism was introduced |
| Remote execution | Dispatch remains entirely local; `behavior_mut` returns an in-process reference, never a remote handle |

Each exclusion is a scope boundary consistent with this document's own narrow purpose (§3), not an oversight — every later milestone that fills one of these gaps (ARCH-004 for supervision, ARCH-005 for timers) does so as a clean, separately-authorized addition, never requiring this document's own architecture to change.

## 15. Historical Placement

**Preceding milestone: EWO-006 (Bounded Actor Mailboxes).** Confirmed by two independent evidentiary methods:

1. **Direct textual citation.** This milestone's own `MAX_BOOTSTRAP_GRANTS` constant's doc comment cites `core/actor-host`'s own `MAILBOX_CAPACITY` as already-existing precedent — code cites a constant "on the same basis" as one that does not yet exist only if the cited one already exists.
2. **Independent test-count arithmetic across separately-authored Engineering Reports.** ER-006 states its own final workspace total explicitly: 232 tests, with `synapse-scheduler: 1` and `synapse-execution-coordinator: 15` at that exact moment. This document's own milestone's current, independently-verified state shows `synapse-scheduler: 19` and `synapse-execution-coordinator: 30` — a gain attributable to no governed milestone other than this one, since neither EWO-007 nor EWO-008 touches either crate (confirmed, their own governing text).

**Following milestones, both confirmed by direct citation in their own governing text, not by inference:**

- **EWO-007 (Local Actor Supervision) depends upon it.** EWO-007's own "Problem Statement" and "Component Boundaries" sections name, verbatim, "this session's own admission pipeline (`admit_message`, `resolve_emitted_message_authority`, `process_emitted_messages`, `runtime/src/lib.rs`)" and "this session's own `RuntimeError::AmbiguousAuthority` addition" as pre-existing baseline its own supervision-routing sequence is inserted alongside, never disturbs. ER-007's own stated starting baseline — 423 tests — is the second, independent, arithmetic confirmation of this document's own milestone's placement immediately before it.
- **EWO-008 (Temporal Runtime) reuses it.** ER-008 independently records `process_due_timers` calling `resolve_emitted_message_authority` and `admit_message`, the identical two functions this document's milestone introduced, reused without modification for a third message origin.

**Why it belongs here in SynapseOS history:** immediately after EWO-006 and immediately before EWO-007 — the single milestone at which the Runtime stopped being a provably correct mechanism for actor work it never actually performed, and became a provably correct mechanism for actor work it genuinely, verifiably performs. Every later milestone in the Runtime Integration series builds on that transition directly.

This document does not discuss Git commit history beyond what is necessary to establish this chronology; the ordering above is architectural, not a statement about repository publication state (that remains the separate, distinct concern of the Publication Recovery Review).

## 16. Risks and Deferred Decisions

- **This document's own governance chain remains incomplete beyond this Architecture document itself.** No Engineering Work Order or Engineering Report yet exists for the milestone this document records — this document restores the architectural half of the missing chain; the engineering-authorization and engineering-report halves remain a separate, not-yet-performed act, consistent with this document's own charter to author architecture only.
- **`AmbiguousAuthority` is cited by three later EWOs (EWO-006, EWO-007, EWO-008) as "the same basis this session's own... addition already was"** — a precedent retroactively relied upon by documents that, at the time they were authored, predated this document's own formal existence. This document closes that gap for future readers; it does not retroactively alter what those three EWOs' own text already says.
- **The exact Bounded Design Decisions, Explicit Exclusions, and Stop Conditions this milestone's implementation evidently observed in practice (§14) were never formally pre-declared the way every other milestone's own governing EWO declares them.** This document reconstructs what the boundaries evidently were, from evidence; a future reader cannot fully distinguish "the implementer chose this boundary deliberately" from "the implementer never considered the alternative" from this document alone — only a corresponding, separately-authored Engineering Work Order (reconstructed on the same basis as this document) could close that residual gap completely.
- **No engineering-correctness risk exists.** Every risk named above concerns the completeness of the historical/governance record, not the correctness, safety, or behavior of the implementation itself — independently confirmed sound by all three governing reviews (§2).

## 17. Normative Architecture Decisions

- Runtime MUST remain the sole cross-component composer for every operation this document describes (ADR-0016 Rule 1, reaffirmed without exception).
- Every message, regardless of origin, MUST pass through the single, identical `admit_message` pipeline before becoming eligible for scheduling (§11) — this is a constitutional Runtime invariant, not a design preference, binding on any future message origin.
- An actor's own emitted messages MUST be treated as admission requests, never as already-sent facts (§8, §10).
- Causation for an emitted message MUST be established by Runtime alone, never by the emitting actor's own self-declared claim (§10, §13).
- Authority for an emitted message MUST be resolved fresh, at the moment of emission, from Capability Authority's own current registry state, never cached or inherited (§10, §13).
- Capability resolution MUST require exactly one structurally-matching, currently-valid, currently-bound candidate — zero is denied, more than one is refused, never arbitrarily chosen (§8, §13).
- The Bootstrap Capability itself MUST NOT be exposed through the bootstrap-grant mechanism or any other means beyond ADR-0017's own existing exception (§12).
- Bootstrap grants MUST be fixed before any `Runtime` value exists and MUST NOT be mintable through any second, repeatable, post-bootstrap path (§12).
- This document MUST NOT be read as authorizing supervision, timers, persistence, distributed execution, or any of §14's other explicitly excluded concerns — each remains separately deferred to its own architecture.

## 18. References

Internal:

- GOV-003 — Governance Model
- GOV-010 — Decision Framework
- GOV-004 — Engineering Principles
- STD-001 — Documentation Standards
- ARCH-000 — Introduction
- ARCH-001 — Constitutional Architecture
- ARCH-002 — Runtime Architecture (§5, §6, §7, §8, §9, §11, §13, §15, §18, §21, §22)
- ARCH-003 — Runtime Integration Architecture (§5, §18, all four revisions v0.1.0–v0.4.0)
- ARCH-004 — Local Actor Supervision Architecture (§9, §10, §13, independently confirmed dependent on this document, §15)
- ARCH-005 — Temporal Runtime Architecture (§9, §13, §14, independently confirmed to reuse this document's own admission pattern, §11, §15)
- ADR-0015 — Audit Emitter Failure Semantics
- ADR-0016 — Trusted Core Interaction Rule
- ADR-0017 — Bootstrap Capability Trust Root
- EWO-005 — Runtime Integration: Truthful Actor Execution-State Tracking
- ER-005 — Runtime Integration: Truthful Actor Execution-State Tracking — Engineering Report
- EWO-006 — Runtime Integration: Bounded Actor Mailboxes with Deterministic Overflow Rejection
- ER-006 — Bounded Actor Mailboxes — Engineering Report (§4, §11, §12 — direct evidentiary basis for §15's historical placement)
- EWO-007 — Runtime Integration: Local Actor Supervision, Failure Escalation, and Restart Ownership (§"Problem Statement", §"Component Boundaries and Prohibited Interactions" — direct citation of this document's own milestone as pre-existing baseline)
- ER-007 — Local Actor Supervision — Engineering Report (§5, §9 — direct evidentiary basis for §15's historical placement)
- EWO-008 — Runtime Integration: Temporal Runtime, Timer Ownership, and Delayed Execution
- ER-008 — Temporal Runtime — Engineering Report (§11 — direct evidence of this document's admission pattern reused verbatim)

Source evidence (verified directly, §8):

- `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679` (working tree):
  - `runtime/src/lib.rs` (`execute_message_capturing`, `process_emitted_messages`, `resolve_emitted_message_authority`, `admit_message`, `submit_message`, `step`, `run_until_idle`, `execute_next_scheduled_message_with_outcome`, `BootstrapGrant`, `RuntimeBootstrapConfig`, `BootstrapGrantSet`, `bootstrap_with_config`, `MAX_BOOTSTRAP_GRANTS`)
  - `core/capability-authority/src/{lib,internal}.rs` (`bound_capabilities`)
  - `core/execution-coordinator/src/{lib,internal}.rs` (`dispatch`'s own genuine invocation, `fail`)
  - `core/actor-host/src/{lib,internal}.rs` (`behavior_mut`, `create_instance_with_behavior`, `dequeue`, `MAILBOX_CAPACITY`, EWO-006 precedent)
  - `services/scheduler/src/{lib,internal}.rs` (`mark_ready`, `remove`, `select_next`, `SchedulerImpl` — this milestone's own first genuine Scheduler implementation)
  - `common/src/lib.rs` (`RuntimeError::AmbiguousAuthority`)
  - `runtime/examples/{worker_pool,actor_to_actor_messaging}.rs`; `runtime/tests/{bootstrap_grant,worker_pool,actor_to_actor_messaging}.rs`
- Completed architecture reviews (this engineering effort; analytical basis for this document, independently re-verified against source per §8, not cited as a substitute for that verification):
  - "SynapseOS — Publication Recovery Architecture Review"
  - "SynapseOS Architecture Review — Capability-Authorized Actor-to-Actor Messaging Runtime"
  - "SynapseOS Architecture Review — Runtime Actor Execution Architecture"

## 19. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-15 | Denver Jacobs | Initial Draft. Historical reconstruction of the Runtime Actor Execution Architecture: the first genuine `Actor::handle()` execution, the shared Runtime-owned admission pipeline for actor-emitted messages, Runtime-owned causation and authority resolution, the bootstrap-grant mechanism, and the resulting truthful execution-ordering guarantees. Restores the architectural half of a governance chain the Publication Recovery Review found missing for an already-completed, already-independently-reviewed implementation milestone. Derived exclusively from the Runtime Actor Execution Architecture Review's own conclusions, cross-checked against ARCH-002 through ARCH-005, ADR-0015 through ADR-0017, ER-005 through ER-008, and direct source inspection of `synapse-runtime` @ `5ccc7f9083a71adc6ee704b2322a701935765679`. Authorizes no implementation — the architecture it records is already fully realized. |
| 0.1.1 | 2026-07-15 | Denver Jacobs | Corrective revision following the Independent Implementation Review of this milestone, which found (verdict: CHANGES REQUIRED) that §9.1 and §9.2 incorrectly described Scheduler as unaffected by this milestone. Direct implementation inspection showed Scheduler's own public trait was redesigned (from a single stateless `select_next(ready: &[ActorInstanceId])` method to three stateful operations, `mark_ready`/`remove`/`select_next()`) and its FIFO implementation written from scratch — confirmed by ER-006's own recorded test count for this crate immediately prior (1 test, empty placeholder struct). Corrected §9.1 (component responsibility table) and §9.2 (non-responsibilities) to accurately record this; added `services/scheduler/src/{lib,internal}.rs` to §18's source evidence list. No other section changed. Scheduler's own architectural role (ready-order policy only, ARCH-002 §6, no lifecycle or capability awareness) is unchanged by this correction — only the prior claim that this milestone left it untouched is corrected. No new architectural decision is introduced; no other claim in this document is affected. |
| 0.1.2 | 2026-07-15 | Denver Jacobs | Applied the Governance Coverage Reconciliation's approved findings. A subsequent, fully independent re-review found the 0.1.1 correction, while accurate for Scheduler, was not sufficient: three further items genuinely new to this milestone remained entirely undisclosed (`ActorHost::create_instance_with_behavior`, `ActorHost::dequeue`, `ExecutionCoordinator::fail`), and the Governance Coverage Reconciliation task subsequently surfaced two more (`Runtime::step()`, `Runtime::run_until_idle()` — both confirmed absent from the committed predecessor state, previously undisclosed anywhere in this document). Added all five to §9.1 (component responsibility table); added `step()`/`run_until_idle()` and `dequeue`'s ordering role to §10 (Runtime Execution Model), including a new ordering-guarantee bullet; updated §18's source evidence list accordingly. Per the Reconciliation's own classification, also **trimmed** §9.1's Scheduler row and §9.2's Scheduler bullet, removing method-signature-level and internal-data-structure-level detail that the Reconciliation classified as Category B (Engineering Deliverable, EWO-009's own domain) rather than Category A (Architectural) — the same treatment EWO-006's own bounded-mailbox implementation already received, requiring no dedicated architectural document beyond ARCH-002 §13's already-sufficient authority. Scheduler's own architectural role remains exactly as 0.1.1 stated it; only the level of detail carried in this document changed. No new architectural decision is introduced; no other section changed. |
| 0.1.3 | 2026-07-15 | Denver Jacobs | Applied the two remaining corrections identified by the Historical Provenance Audit (a dedicated audit, distinct from the prior Independent Reviews, that swept every historical-claim statement in this document and EWO-009 against direct source evidence). Corrected §14's Supervision row: a failing `Actor::handle()` is handled by Execution Coordinator's own truthful failure-cleanup mechanics, genuinely new to this milestone — not, as previously stated, "the pre-existing cleanup path." Corrected §9.1's Execution Coordinator row to disclose that dispatch's own engineering surface widened to include surfacing what genuine invocation emits, not merely gaining "a" parameter — implementation-level signature detail remains EWO-009's concern, per this document's own established Scheduler precedent. No architectural principle changed, no Trusted Core boundary changed, no engineering decision was reopened. |

## 20. Approval Status

| Role | Name | Status | Date |
|------|------|--------|------|
| Author | Denver Jacobs | Drafted | 2026-07-15 |
| Technical Review | TBD | Pending | |
| Approval Authority | Chief Architect (vacant); Founder (interim) | Pending | |
