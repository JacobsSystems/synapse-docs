---
document_id: ADR-0016
title: Trusted Core Interaction Rule
version: 0.5.0
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
  standards: None
  architecture:
    - ARCH-001
    - ARCH-002
  adrs:
    - ADR-0013
    - ADR-0014
    - ADR-0015 (Draft — not yet approved; see status notice)
  engineering:
    - EWO-002 (work-orders/EWO-002-Actor-Host.md, current Draft — not amended by this ADR)
    - Architecture Interpretation Report — Trusted Core Audit Coupling (the discovery record for this ADR)
supersedes: None
superseded_by: None
ai_assistance: Drafting
---

# ADR-0016 — Trusted Core Interaction Rule

> **Status notice:** This ADR is **Draft**. No approval act has occurred. ADR-0015, cross-referenced throughout, is also still Draft — no evidence commit exists for it. This ADR is **not** an interpretation of existing architecture. Part I establishes, from authoritative text alone, that no single interpretation is available; Part III then introduces two independent new architectural rules to close that gap, explicitly labelled as extensions, not readings. See §8 (Approval Status).

## 1. Context

Implementation of SRP-002 (Actor Host) required Actor Host to cause an audit event to reach Audit Emitter. Architecture Review, and the Architecture Interpretation Report this ADR follows, found that ARCH-002 does not determine how any two Trusted Core components connect to one another — not only for the audit case, but as a general property of the architecture. This ADR determines whether that gap is real, and if so, closes it with the smallest possible set of new rules.

## 2. Objective

Determine the minimum architectural rule required to define how Trusted Core components are permitted to interact. Nothing else. This ADR does not redesign the Runtime, Trusted Core membership, component responsibilities, scheduling, lifecycle, the capability model, the audit model, host abstraction, execution coordination, messaging, or persistence.

## 3. Part I — Establishing Necessity

**1. What interactions are already defined?** ARCH-002 §11 (Constitutional Execution Cycle) names, for each of twenty steps, the party "Responsible" and its "Trust" classification. ARCH-002 §20 (Runtime Interfaces) names, for fifteen interfaces, a Caller and a Receiver, a purpose, and a failure behaviour. Together these fully assign *logical accountability*: which component is answerable for triggering, and which for receiving, each named interaction.

**2. What responsibilities are already assigned?** Each of the seven Trusted Core components has a distinct, exclusive responsibility and an explicit "Prohibited from" boundary (§6). Each of the four constitutional guarantees is attributed to specific mechanisms (§5). Nothing about *what each component does* is missing.

**3. What is intentionally left unspecified?** ARCH-002 is deliberately silent on how a logical Caller/Receiver pairing becomes a connected interaction. This silence is consistent with its general discipline of separating guarantee-accountability from mechanism (§17: Actor Host "need not itself implement the isolation mechanism, but it must verify or enforce that the chosen mechanism actually delivers it") and separating mechanism from policy (ARCH-001 §9). But the question at issue here is not an ordinary mechanism question of the kind ARCH-002 elsewhere defers: it is whether a Trusted Core component's exclusive responsibility (§6) may be exceeded by requiring it to also know how to reach another component, and, if not, which architectural entity is accountable for connecting them at all. Neither question is addressed by any existing text.

**4. Why does interpretation fail?** §20's Caller/Receiver table uses one uniform pattern across all fifteen interfaces. Reading any single row — including the audit row — as resolving either question, while every other row uses identical language, is not textually defensible unless the same reading is applied to all fifteen. Applied uniformly, the table does not resolve either question for any row. Interpretation does not fail because the text is unclear about one case; it fails because the text is silent, uniformly, about both general questions.

**5. Why can multiple architecture-consistent implementations exist?** A Runtime in which every Trusted Core component is free to acquire awareness of any other, and a Runtime in which no Trusted Core component ever acquires such awareness, would both satisfy every currently-stated requirement in ARCH-002 — every named interface realized, every guarantee held, every failure behaviour honoured, every Conformance Requirement in §22 met. Yet the two would differ, materially and permanently, in whether each component's §6 responsibility remains exclusive in practice. When two structurally incompatible outcomes can both claim full conformance to a boundary the architecture already treats as significant (§6's own "Prohibited from" discipline), the architecture has under-determined that boundary. That is the precise condition under which new rules — not readings — are required.

**Conclusion of Part I: new rules are objectively necessary. This ADR proceeds to Part II.**

## 4. Part II — Design Constraints

Properties any valid interaction rule must preserve, stated without reference to any mechanism:

- **Constitutional integrity** — must not alter ARCH-001's four constitutional concepts or six constitutional laws; operates strictly within the Runtime layer ARCH-001 §7 already delineates.
- **Trusted Core boundary fixedness** — must not create, imply, or require an eighth Trusted Core component; the seven named in ARCH-002 §6 remain exhaustive.
- **Guarantee attribution** — must not shift any of the four constitutional guarantees from the mechanism ARCH-002 §5 already assigns it to.
- **Preservation of exclusive responsibility** — must not require any Trusted Core component to exceed the exclusive responsibility §6 already assigns it.
- **Single architectural authority for interaction** — must not leave "who connects components" answerable by more than one architecturally accountable party at once.
- **Determinism** — must produce one answer to each question this ADR addresses, not leave either open to multiple equally valid readings.
- **Explicit ownership** — whichever party is assigned a responsibility by this ADR must already be a specific, recognized architectural entity, not a vague or newly invented one.
- **Implementation independence** — must be statable without reference to any technology; must hold regardless of what the Runtime is realized in.

## 5. Part III — Two Independent Decisions

Four distinct architectural concepts are in play here, and this ADR does not conflate them: **behavioural responsibility** (what a component is for — fixed by §6), **interaction awareness** (whether a component knows another exists), **invocation** (an actual act of one thing causing another to act), and **interaction accountability** (who is answerable, architecturally, for interactions among Trusted Core components existing and being coordinated at all). This ADR's two decisions concern interaction accountability only. Neither decision alters, transfers, absorbs, or redefines any component's behavioural responsibility as §6 already states it.

The two decisions are established in the following order: first, who holds interaction accountability; second, what follows from that assignment being exclusive.

### 5.1 Decision 1 — Who is accountable for establishing and coordinating interaction among Trusted Core components?

This is answered using only ARCH-002 §3's already-stated responsibility model for the Runtime as a whole — no implementation precedent is used.

ARCH-002 §3 states the Runtime's "single overarching responsibility": to "realize Actor, Capability, Message, and Execution Semantics as running behavior, and guarantee — regardless of what applications or Runtime services do — that non-forgery, integrity, enforcement-at-invocation, and revocation-state enforcement continue to hold." §3 further states what the Runtime already owns: "actor lifecycle mechanics, mailbox mechanics, dispatch mechanics, capability validation and binding mechanics, execution-context construction, minimal audit-event emission, the bootstrap act." Each of these named items already spans more than one Trusted Core component's own §6 responsibility — capability validation and binding mechanics span Capability Authority and Actor Host; dispatch mechanics span the Scheduler's policy and Execution Coordinator's mechanism; the bootstrap act spans all seven Trusted Core components at once (§11, step 1). Establishing and coordinating these connective mechanics is not itself listed as part of any single component's §6 responsibility — yet §3 already, and without amendment, assigns responsibility for them to the Runtime as a whole.

**Alternative 1 — Runtime.** The Runtime is accountable for establishing and coordinating interaction among Trusted Core components.

- **Consistency with existing architecture:** §3 already commits the Runtime to "realize... as running behavior" — a responsibility that is meaningless unless something accounts for combining the individually-owned mechanics of separate components into an actual working whole. §3 already lists connective mechanics among what the Runtime owns. Assigning this accountability to the Runtime adds no new entity and states no new capability beyond what "realize... as running behavior" already requires; it makes explicit, as a normative rule, what §3's existing responsibility statement already necessarily entails.
- **Minimum-extension test:** no other named architectural entity currently holds any responsibility, under ARCH-002's existing text, that could plausibly encompass this. Every replaceable service's responsibility is explicitly non-trusted and policy-scoped (§6, §19) — incapable of accounting for interaction among trusted mechanisms without itself requiring trust it does not have. The Runtime is the only entity whose existing responsibility statement already covers this function.
- **Adopted.**

**Alternative 2 — A newly named accountable entity, distinct from the Runtime.**

- **Consistency with existing architecture:** would require assigning this new entity its own guarantee attribution and trust classification before it could safely account for interaction among trusted mechanisms — an entity capable of that is, in substance, an eighth Trusted Core component, directly violating Trusted Core boundary fixedness (Part II).
- **Not adopted.**

**Decision 1: The Runtime is the sole architectural entity accountable for establishing and coordinating Trusted Core interactions.**

"Sole" is load-bearing here, not decorative: §3's responsibility statement is singular ("its single overarching responsibility"), and the minimum-extension test above found exactly one entity capable of holding this accountability under existing text — not one entity among several equally capable candidates. Decision 1 therefore assigns *exclusive* accountability, not merely primary or default accountability.

### 5.2 Decision 2 — What follows for direct Trusted Core interaction?

Decision 2 is a consequence of Decision 1, not an independent justification. Because Decision 1 assigns interaction accountability to the Runtime exclusively, no other party can simultaneously hold that same accountability without contradicting the exclusivity Decision 1 establishes. If a Trusted Core component were to independently establish or own a direct interaction path to another Trusted Core component, that component would itself be exercising interaction accountability — the very accountability Decision 1 assigns solely to the Runtime. Two architecturally distinct entities cannot both hold sole accountability for the same thing; that is a contradiction in terms, not a policy preference.

This says nothing about, and does not touch, any component's own behavioural responsibility (§6). A component performing its own §6-defined function is not thereby exercising interaction accountability — only independently establishing or owning an interaction path to another component would be.

**Decision 2: Because interaction accountability is exclusively assigned to the Runtime, Trusted Core components must not independently establish or own direct peer interaction paths.**

## 6. Part IV — The Two Rules

This ADR introduces exactly two new rules, added because implementation demonstrated an omission, not read from prior text, in the order established in Part III:

> **Rule 1.** The Runtime is the sole architectural entity accountable for establishing and coordinating Trusted Core interactions.
>
> **Rule 2.** Because interaction accountability is exclusively assigned to the Runtime, Trusted Core components must not independently establish or own direct peer interaction paths.

Both are new rules within the Runtime architecture (ARCH-002's layer), not new constitutional rules within ARCH-001 — they govern Runtime-internal structure, not the constitutional concepts or laws ARCH-001 defines, and they do not alter ARCH-001's layering (§7) in any respect. "The Runtime" here is exactly the entity ARCH-002 §3 already names and already assigns an overarching, connective responsibility to — not a new concept. Rule 2 is not an independent decision; it is what Rule 1's exclusivity necessarily entails, stated explicitly so it cannot be mistaken for a separately-justified prohibition.

**Responsibility ownership is preserved, explicitly.** Rule 1 assigns the Runtime interaction accountability only. It does not assign the Runtime, and the Runtime does not acquire, any Trusted Core component's own behavioural responsibility. Every one of the seven Trusted Core components continues to own exactly the responsibility ARCH-002 §6 already defines for it, in exactly the terms §6 already states — capability validation and binding remain Capability Authority's; actor identity and isolation remain Actor Host's; and so for each of the other five. The Runtime does not absorb, duplicate, or redefine any of them. Interaction accountability (who is answerable for Trusted Core components being connected at all) and behavioural responsibility (what each component itself does) are distinct architectural concepts throughout this ADR, and Rule 1 speaks only to the former.

**Scope of Rule 1, stated explicitly.** Rule 1 assigns the Runtime accountability for the *architectural interaction model* only — that Trusted Core components connect through it and not directly through one another. It defines architectural ownership only. It does not prescribe, and must not be read to prescribe: application-programming interfaces; interfaces of any other kind; call paths; dependency graphs; constructors; dependency injection; traits or any other language-level abstraction; package or crate dependencies; execution techniques; or any other implementation mechanism by which this accountability is realized. All of these remain entirely open, to be decided when the relevant engineering work is authorized. This ADR remains fully implementation-independent.

Neither rule expands, and this ADR does not otherwise expand, the set of interactions ARCH-002 §11 and §20 already catalogue. A future need to connect two Trusted Core components in a way not already named in either place still requires its own architectural recognition before it may proceed.

**Whether the selected architectural rule changed:** no. Runtime accountability for Trusted Core interaction, with direct peer interaction excluded, remains the selected outcome. What changed is the logical chain: Runtime accountability is now established first, directly from §3's existing responsibility statement; the prohibition on direct interaction is now derived as its necessary consequence, not argued independently; and no responsibility-transfer or "unstated eighth responsibility" argument remains anywhere in the reasoning.

## 7. Part V — Consequences

- No Trusted Core component's own responsibility, as stated by §6, is altered, absorbed, duplicated, or redefined by this ADR.
- The Runtime, and only the Runtime, is accountable for establishing and coordinating Trusted Core interaction — an explicit statement of a responsibility §3 already implied but never stated as a rule.
- Trusted Core components must not independently establish or own direct peer interaction paths — a necessary consequence of Rule 1's exclusivity, not a separate policy choice.
- Rule 1 concerns accountability for the interaction model only; it prescribes no mechanism, call path, API, dependency structure, or execution technique, and engineering retains full discretion over how the Runtime's accountability is realized.
- The audit-coupling question the Architecture Interpretation Report identified is resolved by these rules exactly as every other Trusted-Core-to-Trusted-Core interaction is: connection is the Runtime's accountability, not any component's own.
- ADR-0015's decision (an operation fails if its mandatory audit emission fails) is unaffected and remains fully independent: that ADR determines the consequence of a failed audit call; this ADR determines only who is accountable for the call being connected in the first place.

## 8. Part VI — Follow-Up

| Document | Amendment required | Why | Normative or clarifying |
|---|---|---|---|
| ARCH-002 §6 (Runtime Component Model) | Yes | Should record Rule 1 alongside the exclusive-responsibility model it preserves | Normative |
| ARCH-002 §3 (Runtime Identity and Responsibility) | Yes | Should record Rule 2 as an explicit statement of the connective accountability §3's existing responsibility language already implies | Normative |
| ARCH-002 §20 (Runtime Interfaces) | Recommended | Would make explicit, at its own source, that the table's entries are connected via the Runtime per Rules 1–2, reducing the risk of future misreading | Clarifying |
| EWO-002 (work-orders/EWO-002-Actor-Host.md) | Yes | Its "Audit Obligations" section left the coupling-accountability question open pending exactly this kind of resolution; the question is now answered and must be incorporated before implementation resumes | Normative |
| ADR-0015 | No | Its own decision is already stated as independent of the coupling mechanism; an optional cross-reference would aid a reader but changes no content | Clarifying, optional |
| Architecture Interpretation Report (Trusted Core Audit Coupling) | No | A discovery record, not a controlled document under STD-001 §5 | Not applicable |

No amendment is performed by this ADR.

## 9. Approval Status

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
| Document ID | ADR-0016 |
| Repository path | adrs/ADR-0016-Trusted-Core-Interaction-Model.md |
| Version | 0.5.0 |
| Artifact revision identifier | Not yet created |
| Content fingerprint | Not yet created |
| Git blob ID | Not yet created |
| Disposition | Not yet approved |
| Approver identity | Not yet assigned |
| Authority citation | Not yet assigned |
| Effective date | Not yet assigned |

No field in this section may be populated until the corresponding act has genuinely occurred, evidenced per STD-001 §31.

## 10. Change History

| Version | Date | Author | Description |
|---------|------|--------|-------------|
| 0.1.0 | 2026-07-12 | Denver Jacobs | Initial Draft, framed as an interpretation of §11/§20's existing structure. No approval act occurred. |
| 0.2.0 | 2026-07-12 | Denver Jacobs | Rewritten as an explicit architectural extension (Parts I–VI), with the prohibition on direct interaction and Runtime ownership stated as a single combined decision, justified partly by relationship-count and topology reasoning. No approval act occurred. |
| 0.3.0 | 2026-07-12 | Denver Jacobs | Returned for final Architecture Review revision. Separated the single combined decision into two independent rules (§5.1 Decision 1: no direct Trusted Core peer interaction, justified by §6's exclusive-responsibility model; §5.2 Decision 2: Runtime accountability for connection, justified independently as the minimum extension of §3's already-stated "realize... as running behavior" responsibility and its enumerated ownership of mechanics already spanning multiple components) with an explicit statement of why Decision 1 does not automatically imply Decision 2. Removed all relationship-count and topology (mesh/star) reasoning, replacing it with reasoning grounded in ownership, exclusive responsibility, Trusted Core boundary preservation, and single architectural authority. Removed all reference to implementation precedent, including EWO-001, from the justification. Added an explicit "Scope of Rule 2" statement confirming Runtime accountability covers the architectural interaction model only, not any implementation mechanism, call path, API, dependency structure, or execution technique. The demonstrated necessity for new rules (Part I), the distinction between interpretation and architectural extension, the ADR's implementation-independence, and the statement that the rules govern only connection topology and never expand recognized interactions are all unchanged from 0.2.0. The selected outcome (Runtime accountability) did not change. No approval act has occurred. |
| 0.4.0 | 2026-07-12 | Denver Jacobs | Returned for one final Architecture Review revision, addressing the logical justification only — the architectural decision itself unchanged. Removed the responsibility-transfer / "unstated eighth responsibility" argument entirely. Explicitly separated four distinct concepts (responsibility ownership, interaction awareness, invocation, interaction accountability) at the start of §5 to prevent them being conflated. Reordered the logical chain: §5.1 now establishes Runtime accountability first, directly from §3's existing "single overarching responsibility" statement (unchanged reasoning from 0.3.0, now presented first rather than second); §5.2 now derives the prohibition on direct Trusted Core interaction as a necessary logical consequence of that accountability being exclusive ("sole"), not as an independently justified rule. Part IV's Rule 1 and Rule 2 reordered to match. Added an explicit "Responsibility ownership is preserved, explicitly" statement confirming the Runtime does not absorb, duplicate, or redefine any Trusted Core component's own §6 responsibility, and that interaction accountability and behavioural responsibility remain distinct throughout. Implementation independence (no call paths, APIs, dependency structures, mechanisms, or execution techniques prescribed) is restated unchanged. The demonstrated necessity for new rules (Part I), the distinction between interpretation and architectural extension, the selected architectural rule, and the follow-up amendment list (Part VI) are all unchanged from 0.3.0. No approval act has occurred. |
| 0.5.0 | 2026-07-12 | Denver Jacobs | Final precision pass before approval, at Architecture Review's direction — no change to the architectural decision, the logical chain's structure, the necessity demonstration, the interpretation/extension distinction, or the follow-up amendment list, all confirmed unchanged from 0.4.0. Aligned terminology throughout §5 and Part IV to the reviewer's own exact vocabulary ("behavioural responsibility" and "interaction accountability," replacing "responsibility ownership" and "connection accountability"). Expanded the "Scope of Rule 1" statement to name each excluded category explicitly — application-programming interfaces, interfaces of any other kind, call paths, dependency graphs, constructors, dependency injection, traits or other language-level abstractions, package or crate dependencies, and execution techniques — rather than relying on the prior draft's generic "implementation mechanism" phrasing, removing any ambiguity for approval. No approval act has occurred. |
