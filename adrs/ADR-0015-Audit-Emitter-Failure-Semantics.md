---
document_id: ADR-0015
title: Audit Emitter Failure Semantics
version: 0.1.0
status: Draft
decision_owners:
  - Denver Jacobs
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Chief Architect (Class B, per GOV-010 §5), vacant — see GOV-003 §3.2; Founder (interim, until appointment)
created: 2026-07-12
last_updated: 2026-07-12
classification: Public
related_documents:
  governance:
    - GOV-003 (Approved, Act 2)
    - GOV-004 (Approved, normal-governance)
    - GOV-010 (Approved, Act 2)
  standards:
    - STD-001
    - STD-002
    - STD-004
    - STD-011
  architecture:
    - ARCH-001
    - ARCH-002
  adrs:
    - ADR-0013
    - ADR-0014
  engineering:
    - EWO-002 (work-orders/EWO-002-Actor-Host.md, current Draft — not amended by this ADR)
    - Architectural Clarification Report — Actor Host Audit Boundary (the discovery record for this ADR)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ADR-0015 — Audit Emitter Failure Semantics

> **Status notice:** This ADR is **Draft**. No approval act has occurred. This is the first implementation-driven ADR of the engineering phase — it exists because implementation of SRP-002 (Actor Host), during Architecture Review, surfaced a genuine gap in ARCH-002's own text, not because of any new feature request. See §12 (Approval Status).

## 1. Context

ARCH-002 requires Trusted Core components to emit mandatory audit events for a fixed minimum set of security-relevant occurrences (§18), and lists "audit emission for every security-relevant event" as a mandatory Conformance Requirement (§22). ARCH-002 also defines, precisely, what happens when the **downstream** Audit Pipeline — a replaceable service that consumes Audit Emitter's output for storage, indexing, retention, and redaction — fails: "Audit consumer failure — MUST NOT block or fail the triggering execution — isolated to the replaceable service" (§16); the Constitutional Execution Cycle's own audit-emission step states the same thing in different words: "Unbypassable; occurs regardless of outcome — Consumer failure MUST NOT block execution" (§11, step 17).

ARCH-002 does not define the symmetric case one layer up: what happens when the **Audit Emitter itself** — a Trusted Core component, not a replaceable service — cannot complete emission of a mandatory event. The `AuditEmitter` trait already implemented under EWO-001 models this as a real possibility (`fn emit(&mut self, event: AuditEvent) -> Result<(), RuntimeError>`), but no architectural text says what a caller must do with that `Err`.

This ADR resolves exactly that gap, and nothing else.

## 2. Problem

SRP-002 (Actor Host) requires Actor Host to cause the two audit events ARCH-002 §18 mandates for it — actor creation, actor termination — to be emitted. Before implementation could proceed, Architecture Review asked what must happen if that emission fails. No document answers this. Proceeding without an answer would mean implementation choosing, unilaterally, a policy with direct constitutional consequences (§3 below) — precisely the kind of decision GOV-004 §1 reserves to an approved architectural specification, not to implementation discretion.

## 3. Why This Gap Exists, and Why Implementation Cannot Close It

**Why downstream Pipeline failure is already defined:** Audit Pipeline is explicitly a *replaceable service* (ARCH-002 §6), and ARCH-002's general philosophy for replaceable services is containment: a service's failure must never compromise or block the trusted-core operation that produced the data it consumes (the same principle governs Scheduler, Actor Directory, and Persistence throughout §16 and §19). Given that general principle, the drafters of ARCH-002 evidently considered the Pipeline case specifically and resolved it explicitly, twice (§11 step 17; §16).

**Why Audit Emitter failure is currently undefined:** Audit Emitter is Trusted Core, not a replaceable service (ARCH-002 §5, §6) — it is one of the mechanisms whose guarantee ("supports auditability of all of the above," §5) the other three constitutional guarantees depend on. ARCH-002 §16's Failure Model table has eleven named rows; none of them is "Audit Emitter's own emission call fails." The closest candidate, "Runtime component failure (a trusted-core mechanism itself fails) — Threatens Runtime integrity — fail-stop for the affected scope," could plausibly be read to cover it, but ARCH-002 never states that a single failed `emit()` call constitutes an instance of that row, as opposed to being treated as an ordinary, locally-rejected operation (like a malformed message or an invalid capability, both of which are contained to "reject request only" in the same table). This is a genuine omission, not a considered silence: the text shows deliberate attention to the structurally analogous Pipeline case and simply does not extend that attention one layer up to the Emitter case.

**Why implementation cannot legitimately invent this:** Audit is one of the four guarantees the entire Trusted Core exists to uphold (ARCH-001; ARCH-002 §5). The available choices for what happens on emission failure — discard the failure, fail the triggering operation, or fail the Runtime — are not interchangeable implementation conveniences; each implies a different, materially distinct evidentiary and availability posture for the constitutional guarantee itself. This is a Class B (Architectural) decision under GOV-010 §4 ("core protocols... platform-wide technical choices"), and GOV-004 §1 (already approved, per prior normal-governance disposition) states plainly: "Production code MUST NOT be written without an approved architectural specification." No such specification currently exists for this case.

**Why this surfaced only during SRP-002, not EWO-001:** EWO-001's own audit calls (Runtime Started, Runtime Shutdown) were made against the same `AuditEmitter` trait, but its current, minimal implementation stores events in an in-process `Vec` — a `push` that cannot, in practice, fail. EWO-001 propagated the `Result` mechanically (`?`) without the question ever becoming operationally material, because nothing in that implementation could actually produce the `Err` case. SRP-002 is the first milestone to examine, generally, how a *repeatable* pattern for Trusted-Core-to-Audit-Emitter interaction should work — and in doing so, Architecture Review necessarily asked what the contract requires when emission genuinely fails, a question EWO-001's trivial implementation had never forced into the open.

## 4. Decision Drivers

- Preserve the audit guarantee's actual value: an audit trail with silent, undetectable gaps is worse than no formal guarantee at all.
- Preserve the containment philosophy ARCH-002 §16 applies everywhere else: failures should be scoped to the smallest defensible blast radius, not the largest.
- Avoid amending ARCH-002's Runtime-level lifecycle state machine (§15) unless the decision genuinely requires it.
- Avoid inventing a new Trusted Core mechanism, component, or cross-cutting failure-propagation channel — the Trusted Core must remain minimal (ARCH-002 §5).
- Resolve the question once, generally, for all seven Trusted Core components — not narrowly for Actor Host alone, since the gap in ARCH-002 §16 is general, not Actor-Host-specific.

## 5. Considered Options

### Option A — Audit emission failure fails only the triggering operation

Failure of `AuditEmitter::emit()` causes the triggering operation to fail before that operation may be reported as successful. This applies only to operations for which ARCH-002 already defines audit emission as mandatory; it does not require Runtime failure, rollback, or a transaction model, and produces no other effect — no other operation, actor, or the Runtime as a whole is affected.

- **Constitutional consistency:** Strong. Gives ARCH-002 §22's existing "audit emission for every security-relevant event" requirement an observable consequence: an operation with a mandatory audit obligation is not reported successful unless that obligation was met.
- **Trusted Core invariants:** Consistent with the containment philosophy visible throughout §16, where most rows scope failure to "current execution/operation only," not wider.
- **Audit integrity:** No unaudited mandatory-event success can occur, and the caller is told, deterministically, that it did not occur.
- **Failure containment:** Minimal blast radius — exactly the one operation.
- **Operational consequences:** If Audit Emitter becomes broadly unable to emit, every mandatory-audited operation across the Runtime begins failing — a fail-safe posture (halt rather than operate un-audited), not a fail-open one.
- **Implementation complexity:** Low. Uses the same per-operation `Result` pattern every Trusted Core interface already uses; requires no new Runtime-wide mechanism.
- This option is the narrow reading of §16's existing "Runtime component failure... fail-stop for the affected scope" row, where "affected scope" is read as the triggering operation. It is a clarification of an ambiguous existing row, not an invention of a new one.

### Option B — Audit emission failure causes the Runtime to enter `Failed`

Any failed mandatory audit emission, anywhere, transitions the entire Runtime to the `Failed` state.

- **Constitutional consistency:** Defensible in spirit (treats an inability to audit as a threat to Runtime integrity), but disproportionate: it applies the *widest* possible reading of "Runtime component failure... fail-stop for the affected scope," where "affected scope" becomes the entire Runtime rather than the operation, for what may be a single, transient, per-call failure.
- **Trusted Core invariants:** Poor. Every other actor's unrelated, otherwise-legitimate operation is halted by one operation's unrelated audit failure — this is the option that most departs from ARCH-002 §16's containment philosophy (isolating actor failure from Runtime; isolating consumer failure from triggering execution), not the closest to it.
- **Audit integrity:** Equal to Option A (no unaudited success occurs) but achieved at materially higher operational cost.
- **Failure containment:** Worst of the three real options — maximizes rather than minimizes blast radius.
- **Operational consequences:** Severe — a single emission hiccup becomes a total-Runtime outage, a availability/single-point-of-failure risk disproportionate to the guarantee being protected.
- **Implementation complexity:** Higher, and with a concrete architectural obstacle: ARCH-002 §15 currently states the Runtime lifecycle as `Initializing → Running → Stopping → {Stopped | Failed}`. Read literally, `Failed` is reachable only following `Stopping` (or, per EWO-001's own implemented transition set, directly from `Initializing`) — **there is no existing `Running → Failed` transition anywhere in ARCH-002 or in the Runtime's current implementation.** Adopting Option B would require either amending ARCH-002 §15 to add a new transition, or routing every single per-operation audit failure through a full orderly-shutdown sequence before reaching `Failed` — a mechanism disproportionate to a single actor's audit hiccup and inconsistent with treating shutdown as the deliberate, one-time act ARCH-002 §11 step 1/step 20 describes it as.
- **Rejected**: requires either an unplanned amendment to the already-defined Runtime lifecycle transition set, or a disproportionate operational mechanism; violates the containment philosophy the existing architecture otherwise applies consistently.

### Option C — Audit emission failure is ignored

The triggering operation proceeds and reports success regardless of whether its mandatory audit event was actually emitted.

- **Constitutional consistency:** Directly and immediately contradicts ARCH-002 §22's own, already-approved mandatory Conformance Requirement — "audit emission for every security-relevant event." This is not a close call.
- **Trusted Core invariants:** Violates the audit guarantee's entire purpose. Worse than having no formal audit requirement at all, since it creates the *appearance* of a complete audit trail while silently permitting gaps no consumer of that trail can detect.
- **Audit integrity:** Directly contradicted.
- **Failure containment:** Nominally perfect (nothing else is ever affected) only because the failure is erased rather than contained.
- **Operational consequences:** Maximally convenient, at the cost of the exact guarantee the Trusted Core's seven-component design exists to provide.
- **Implementation complexity:** Lowest — but the Decision Criteria explicitly exclude optimizing for implementation convenience.
- **Rejected**: contradicts an already-approved mandatory Conformance Requirement outright.

### Option D — Another architecture-consistent option

No materially distinct fourth option was found. The only additional consideration surfaced during analysis is the "affected scope" framing already used to distinguish Options A and B above: both are readings of the same existing §16 row, differing only in how narrowly "affected scope" is construed. That framing does not yield an independent fourth option; it explains why Option A is the conservative, textually-grounded choice and Option B the expansive one.

## 6. Decision

**Option A is adopted.** Where the architecture defines audit emission as mandatory for an operation, failure to emit the required audit event causes that operation to fail. This defines the behaviour only of operations that already possess a mandatory audit obligation under ARCH-002 §18 — it does not redefine Trusted Core operations generally. The rule applies uniformly to every Trusted Core component with such an obligation, not to Actor Host alone. It does not alter, and is fully independent of, the existing downstream Audit Pipeline failure-containment rule (§11 step 17; §16), which remains exactly as already defined.

Per the Decision Criteria: this is the only option preserving constitutional correctness (upholds §22's mandatory requirement) without expanding Trusted Core integrity risk (Option B), without violating evidence integrity (Option C), without requiring a new mechanism (Trusted Core remains minimal — no new component, no new cross-cutting channel), with fully deterministic behaviour (a uniform, uniform-everywhere rule), and without amending the existing Runtime architecture (ARCH-002 §15's lifecycle transition set is untouched).

## 7. Consequences

- An operation for which the architecture requires mandatory audit emission shall not be reported as successful unless the required audit event has been emitted successfully. This states an observable architectural contract only; it does not prescribe rollback, transaction semantics, implementation ordering, persistence, or internal state transitions, all of which remain implementation freedom.
- This applies only to the events §18 already designates mandatory — it does not create any new mandatory-audit obligation beyond what §18 already lists, and it does not redefine the success criteria of any Trusted Core operation that has no mandatory audit obligation.
- No change to Runtime-level lifecycle semantics (ARCH-002 §15). Runtime-level `Failed` remains reachable only via the transitions already established (`Initializing → Failed`, `Stopping → Failed`); a per-operation audit-emission failure never, by this decision, directly causes a Runtime-wide state transition.
- The existing Audit Pipeline (downstream consumer) failure-containment rule is untouched and remains textually and architecturally distinct from this decision. The two failure domains — Emitter emission failure (this ADR) and Pipeline consumption failure (already defined) — remain governed by different rules and must not be conflated.
- Every Trusted Core component that causes a mandatory audit event to be emitted inherits this same failure-propagation rule, regardless of which coupling mechanism ultimately connects it to Audit Emitter. This ADR resolves the failure-semantics question generally; it does not resolve, and is independent of, the separate open question (raised in the Actor Host Audit Boundary Architectural Clarification Report) of which coupling mechanism connects a given Trusted Core component to Audit Emitter. Whichever mechanism is eventually chosen for that separate question must implement this ADR's failure rule.

## 8. Required Updates (not performed by this ADR)

Reconsidered against the narrower decision in §6, which deliberately avoids any change to Runtime-level failure or lifecycle semantics: only sections whose absence would leave a genuine gap or a genuine misreading risk are listed. None of these amendments is made here.

- **ARCH-002 §15 (Runtime Lifecycle) — does not require amendment.** The decision in §6 never touches Runtime-level state; the existing transition set (`Initializing → Running → Stopping → {Stopped | Failed}`) is unaffected and needs no change.
- **ARCH-002 §16 (Failure Model) — does not require amendment.** On reconsideration, the decision in §6 is an operation-level reporting contract, not a new failure-taxonomy classification; it does not require reinterpreting or adding a row to the existing table, and no existing row in §16 becomes inaccurate as a result of this decision.
- **ARCH-002 §22 (Conformance Requirements)** — a cross-reference to this ADR alongside the existing "audit emission for every security-relevant event" bullet would help a reader find the operation-failure consequence, but nothing in §22's current text is inaccurate or contradicted without it. Recommended, not required.
- **ARCH-002 §11 (Constitutional Execution Cycle), step 17** — its existing text ("Consumer failure MUST NOT block execution") addresses only downstream Pipeline tolerance and does not itself misstate anything about Emitter failure; a short clarifying cross-reference would reduce the risk of a reader inferring the same tolerance applies to Emitter failure, which it does not. Recommended, not required.
- **EWO-002 (work-orders/EWO-002-Actor-Host.md)** — genuinely requires a follow-up revision. Its "Audit Obligations" section explicitly left audit-emission failure behaviour unresolved pending this ADR; that section should incorporate this decision once approved. This ADR does not itself amend EWO-002.
- **STD-002 (Coding Standards)** — MAY warrant a cross-reference noting that operations with mandatory audit obligations must propagate emission failure as their own failure, for engineering-level consistency; optional, not required, and not a defect in its current text.

## 9. Risks

- A future, more capable Audit Emitter implementation (e.g., one backed by a real I/O channel) may fail more often, and more meaningfully, than the current trivial in-memory implementation — this decision means such failures will visibly and correctly propagate as operation failures, which is the intended, disclosed trade-off, not an unforeseen one.
- If Audit Emitter's failure rate is ever non-negligible in a real deployment, this decision means Trusted Core operations become correspondingly less available — an intentional fail-safe posture, disclosed here rather than discovered later.

## 10. Security Impact

This decision strengthens, rather than weakens, the audit guarantee's security value: it removes the possibility of a silent, undetectable audit gap for any mandatory-audited operation. It introduces no new attack surface and grants no new authority to any actor or component.

## 11. Validation

No implementation validates this ADR directly; validation occurs when EWO-002 (or its successor) is revised to incorporate this decision and its own validation gates (per EWO-002's Mandatory Validation) confirm the resulting behaviour.

## 12. Approval Status

### Document State

| Field | Value |
|-------|-------|
| Lifecycle status | Draft |
| Normal-governance disposition | Not yet approved |
| Disposition date | Not yet assigned |
| Approving actor | Not yet assigned |

### Approval Evidence (per STD-001 §31)

| Field | Value |
|-------|-------|
| Document ID | ADR-0015 |
| Repository path | adrs/ADR-0015-Audit-Emitter-Failure-Semantics.md |
| Version | 0.1.0 |
| Artifact revision identifier | Not yet created |
| Content fingerprint | Not yet created |
| Git blob ID | Not yet created |
| Disposition | Not yet approved |
| Approver identity | Not yet assigned |
| Authority citation | Not yet assigned |
| Effective date | Not yet assigned |

No field in this section may be populated until the corresponding act has genuinely occurred, evidenced per STD-001 §31.

## 13. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-12 | Denver Jacobs | Initial Draft. Resolves the Audit Emitter failure-semantics gap Architecture Review identified during SRP-002 (Actor Host), documented in the Actor Host Audit Boundary Architectural Clarification Report. Scope strictly limited to this one question; no other document redesigned. No approval act has occurred. |
| 0.1.0 | 2026-07-12 | Denver Jacobs | Returned for revision following Architecture Review (no version bump — Draft-phase correction). Narrowed §6's decision language to apply only to operations that already possess a mandatory audit obligation under ARCH-002 §18, rather than restating it as a general Trusted Core rule; removed "constitutionally complete" throughout as an unnecessary elevation of the decision into a broader constitutional invariant; replaced "success and mandatory-audit emission become atomically linked" with an observable-contract statement that does not prescribe rollback, transaction semantics, implementation ordering, persistence, or internal state transitions; refined Option A's own definition to the same narrower framing; re-evaluated §8 (Required Updates) and concluded, on reconsideration, that ARCH-002 §15 and §16 do not genuinely require amendment under the narrowed decision, downgrading §22 and §11 step 17 from required to recommended, and retaining EWO-002 as the one genuinely required follow-up. The ambiguity statement, option analysis, rejection of Options B and C, consequences substance, and implementation independence are unchanged. |
