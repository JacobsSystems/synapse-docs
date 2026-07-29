---
document_id: ACT-003
title: Act 3 Authorization and Charter
version: 0.1.0
status: Approved
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - TBD
approval_authority: Founder (Denver Jacobs), per GOV-003 §3.1 and §5 — this document authorizes a new major engineering phase, not a Class B (Architectural) content decision; it is therefore approved at the Governance tier on the same basis as GOV-003, GOV-010, GOV-004, GOV-011, and GOV-013, not delegated to the Chief Architect (currently vacant, GOV-003 §3.2). Individual Act 3 Engineering Work Orders, once this charter is operative, are ratified as ordinary Class E (Implementation) or Class B (Architectural) dispositions per GOV-010 §4–§5, ordinarily by the Founder acting as interim Approval Authority in the absence of an identified delegate — identical basis to every EWO approved to date (EWO-014, EWO-015, EWO-016).
created: 2026-07-29
last_updated: 2026-07-29
classification: Public
related_documents:
  governance:
    - GOV-003 (Governance Model) — roles and authority this charter draws on without redefining
    - GOV-004 (Engineering Principles) — Approved (normal-governance disposition, commit `1c1aaef`)
    - GOV-010 (Decision Framework) — Approved (Act 2, commit `11cab8d`)
    - GOV-011 (Architecture Review Board Charter) — Draft, not yet approved; cited for its own explicit disclaimer that it does not authorize Act 3
    - GOV-013 (Engineering Lifecycle) — Approved (commit `3ac7cfb`); the sole authority for every lifecycle stage through Engineering Work Order authoring, and for Implementation, Independent Implementation Review, Engineering Report, and Publication as generic concepts, unchanged and unrestated by this charter
  standards:
    - STD-001 (Documentation Standards) — the sole authority for document families, identifiers, versioning, and evidence representation this charter defers to throughout; see the Filing Note below for this document's own identifier disclosure
    - STD-031 (Engineering Work Order Lifecycle Standard, v0.2.1, Approved, commit `f7a2a16`) — governs every Engineering Work Order authorized to begin under this charter; this charter does not restate, amend, or compete with it
  architecture:
    - ARCH-008 (Effect Runtime Architecture, v0.4.3, Approved) — the primary authorizing architecture for this charter's own initial roadmap
  consolidation:
    - AFR-001 (Architecture Freeze Review) — read directly; Certification Statement and Gate 4 findings are the primary evidentiary basis for this charter's own Risks section
  predecessor: SynapseOS Act 3 Engineering Baseline Report (2026-07-29, evidence-gathering task; not itself a committed or published Documentation-repository artifact — its findings are independently re-verified, not merely cited, throughout this charter)
  engineering:
    - EWO-014, EWO-015, EWO-016 (cited as the demonstrated Founder Approval / Founder Acceptance precedent this charter's own authorization act follows)
---

# ACT-003 — Act 3 Authorization and Charter

> **Status notice.** This document is **Approved**, effective 2026-07-29, per the Founder Authorization recorded in Approval Status (below). Drafting, saving, staging, committing, or pushing this document does not itself constitute its approval — the recorded Founder Authorization act does.

> **Filing note.** `ACT` is not currently a registered STD-001 §5 controlled-document family (see Appendix B). This document is filed as **ACT-003**, in `governance/`, as the narrowest existing, purpose-consistent location and identifier available — a Governance-tier authorization and charter document, structurally closest to GOV-011's own role as a charter establishing a durable governance construct. This is a disclosed, narrow convenience, not a documentation-hierarchy redesign, on the identical basis DES-001 already established for its own family-less filing (GOV-013 §6.4). Whether "ACT" should become a registered STD-001 family, with its own identifier series, is a question for a future, separately authorized STD-001 amendment; this document takes no position and makes no recommendation.
>
> **Numbering note, disclosed rather than left implicit.** This is the first dedicated charter document ever filed for any "Act" of this project's history. "Act 1" (the Runtime foundation, tagged `act1-foundation` at commit `0e4c5c9`) and "Act 2" (the governance-bootstrap period producing GOV-003, GOV-010, GOV-004, and the ADR-0011/ADR-0012 evidence model, referenced as "Act 2" throughout existing evidence commits and GOV-011 §3.1) were never themselves the subject of a dedicated authorization document — they are retrospective, informal labels this project's own history already uses, not artifacts this document is inventing. `ACT-003` is used, rather than `ACT-001`, because it names the third such phase in that already-established informal sequence; this document does not retroactively author, or claim authorship of, an `ACT-001` or `ACT-002` document that does not exist and is not created here.

## 1. Purpose

Act 3 exists to move SynapseOS from **Foundation and Governance** — establishing the governance model, the constitutional and runtime architecture, the Effect Runtime's internal mechanics, and the Engineering Work Order lifecycle standard that governs how further work proceeds — to **Production Capability**: proving that the Runtime, as architected and implemented through Act 1 and Act 2, can actually execute real work against real external systems, end to end, under the discipline STD-031 now formalizes.

This charter authorizes that transition. It does not itself perform any engineering work, does not amend architecture, and does not approve any specific Engineering Work Order. It establishes the boundary within which future Act 3 Engineering Work Orders may be authored, reviewed, approved, published, implemented, and accepted under STD-031's own lifecycle — exactly as GOV-013 §6.12's own Entry Criteria already require an Engineering Work Order to cite "the specific architecture sections that authorize the work"; this charter is the governance-tier act that opens the phase those future EWOs will operate within, on the same basis GOV-011 itself already anticipated ("A future ARB session — convened in Act 2, Act 3, or at any later point in this project's history — operates under this same charter," GOV-011 §16) without itself supplying.

## 2. Scope

**In scope for Act 3:**

- **Effect Providers.** Concrete implementations of the Provider Actor contract the Effect Runtime (ARCH-008) already defines but which no crate in the current workspace implements (verified directly: `grep -rn "trait Provider" --include="*.rs"` and a search for any `*provider*`-named source path both return empty).
- **Runtime external capability.** Any capability-authorized mechanism by which the Runtime, through a Provider, causes an effect outside its own process boundary — the specific external systems (HTTP, filesystem, a specific tool, or similar) are an Engineering Work Order's own decision, not this charter's.
- **Provider lifecycle.** Registration, idempotency (EWO-014), timeout (EWO-012), cancellation (EWO-013), and retry (EWO-015/EWO-016) behavior, already implemented generically in the Effect Coordinator, exercised for the first time against a genuine Provider rather than a test double.
- **End-to-end execution.** A demonstrated path from actor-issued Effect request through Provider execution to recorded, audited completion — proving the architecture's internal consistency (already exhaustively unit-tested, 770/0 as of this baseline) also holds under genuine execution.
- **Runtime validation.** Exercising the full mandatory validation suite (STD-031 §11) against Act 3's own work, at every stage boundary STD-031 §10 names.
- **Demonstration systems.** A minimal, narrowly scoped example or integration test proving the above — the `tests/` and `examples/` directories currently contain placeholder `README.md` files only (verified directly) and are legitimate Act 3 targets for this purpose.

**Explicitly out of scope for Act 3:**

- Distributed runtime, distributed supervision, or clustering (ARCH-002 §21/§23 — already disclosed as deferred to future scope by the architecture itself).
- A provider marketplace or plugin-distribution mechanism (ARCH-008 §33 — explicitly deferred).
- A cloud platform or hosted deployment offering.
- An intelligence layer of any kind (ARCH-000 §19, ARCH-002 §23 — no architecture exists for this yet, by design).
- A production Runtime Control API or Control Centre user interface (ARCH-008 §29–§30 — explicitly deferred; a Control Centre is not the same construct as the demonstration systems named above).
- Any future Act beyond Act 3. This charter authorizes and bounds Act 3 alone; it does not name, number, or pre-authorize an "Act 4," on the identical basis ADR-0011 §22 already establishes for its own refusal to name an "Act 3" in advance of a dedicated authorizing act — this document is that act, for Act 3 only.

## 3. Relationship to Existing Governance

This charter does not redefine, restate, or compete with any stage GOV-013 already governs, or any stage STD-031 already governs for the Engineering Work Order lifecycle specifically. GOV-013 remains the sole authority for the complete engineering lifecycle from Idea through Publication; STD-031 remains the sole authority for the Engineering Work Order lifecycle beginning after an EWO exists. This charter operates one level above both: it does not define a new lifecycle stage at all, and every Act 3 Engineering Work Order will still pass through the identical GOV-013/STD-031 sequence every EWO before it has used. What this charter adds, that neither GOV-013 nor STD-031 themselves provide, is a bounded, named scope and a set of entry/exit criteria for a *phase* of engineering work spanning many future EWOs — a construct neither document currently defines or claims to define. GOV-013 §13.1–§13.3 leave open only whether certain *stages* should become registered standards-tier practices; neither open question concerns phase-level authorization, so this charter does not resolve, reopen, or touch either.

No architecture is amended, created, or reinterpreted by this document. Every architectural citation below (ARCH-008 in particular) is read and applied exactly as currently approved.

## 4. Objectives

1. Implement and demonstrate at least one concrete Effect Provider, exercised through the existing Effect Coordinator without modification to its already-approved (ARCH-008) behavior.
2. Prove the full Effect lifecycle — dispatch, timeout, cancellation, idempotent retry — end to end against that Provider, not merely against the 770 existing unit tests' synthetic doubles.
3. Establish a minimal, repeatable demonstration (integration test or runnable example) that a future engineer, or a future Independent Engineering Review, can execute directly rather than take on faith.
4. Extend, through demonstrated practice exactly as GOV-013 §13.1 anticipates, STD-031's own EWO lifecycle to a second and third real Engineering Work Order, confirming EWO-016 was a repeatable pattern rather than a one-off.

## 5. Success Criteria

Act 3 is progressing successfully when, and only when, each claim below is independently verifiable from repository evidence, on the identical discipline STD-031 §11 already requires for every stage boundary:

- At least one Provider Actor implementation exists in the Runtime workspace, is covered by passing tests, and is exercised by at least one end-to-end demonstration.
- `cargo fmt --all -- --check`, `cargo clippy --workspace --all-targets --all-features -- -D warnings`, `cargo build --workspace --all-targets --all-features`, and `cargo test --workspace --all-targets --all-features` all pass cleanly at every Act 3 stage boundary, with test totals independently re-derived by direct summation, not asserted from a single aggregate figure.
- Every Act 3 Engineering Work Order completes the full STD-031 lifecycle — Independent Engineering Review, Founder Approval, Publication, Implementation, Independent Implementation Review, Engineering Report, Founder Acceptance, Publication, Baseline Update — exactly as EWO-016 already demonstrated once.
- No Act 3 Engineering Work Order requires modifying ARCH-008's own approved content; any genuine architectural gap discovered is escalated (STD-031 §7.4's own stop/escalate clause), not silently absorbed into an EWO's own scope.

## 6. Entry Criteria

Confirmed directly, not assumed, immediately before this charter's own authorization:

- **Runtime baseline:** `29ea55ced6348490f90bd7baeb08d3d4705f19ab`, branch `main`, clean, `0/0` against `origin/main`.
- **Documentation baseline:** `f7a2a1666fcbeff7a26b42131856d5508c1d2612`, branch `main`, `0/0` against `origin/main`, only previously disclosed, unrelated drift present (`STD-001` modification pre-dating 2026-07-13; eleven untracked files, none touched by this task).
- **Approved standards:** STD-001 (v0.4.0, approval evidence located for v0.1.0/v0.2.0 only — see §8); STD-031 (v0.2.1, cleanly Approved, commit `f7a2a16`).
- **Approved lifecycle:** GOV-013 (v0.1.0, Approved, commit `3ac7cfb`).
- **Approved engineering process:** STD-031's own EWO lifecycle, demonstrated complete exactly once (EWO-016, Founder Acceptance commit `a316874`).
- **Baseline evidence:** the SynapseOS Act 3 Engineering Baseline Report (2026-07-29), independently re-verified in the drafting of this charter rather than cited as self-validating.

All entry criteria are satisfied. No entry criterion above is blocked by any risk disclosed in §8.

## 7. Initial Engineering Roadmap

This section establishes a planned opening sequence only. **No specific Engineering Work Order is authorized by this charter.** Each item below still requires its own EWO, independently drafted, cited against ARCH-008, and passed through the complete STD-031 lifecycle before any implementation may begin.

```text
EWO-017 (planned identifier, not yet drafted)
First Concrete Effect Provider

  |
  v

End-to-End Runtime Demonstration

  |
  v

Additional Providers

  |
  v

Runtime Capability Expansion
```

This ordering follows directly from the dependency analysis in the Act 3 Engineering Baseline Report §12: every mechanism a first Provider needs (timeout, cancellation, idempotency, retry) is already implemented and closed (EWO-012–EWO-016); no further architecture is required before it; and it is the one capability every deferred item in ARCH-008 §33 has in common as a prerequisite.

## 8. Risks

**Governance risks:**

- **ADR-0017 / ARCH-001 v0.2.0 evidence-commit anomaly (AFR-001 Gate 4, Finding 2 — still open as of 2026-07-20, independently reverified in this task's own reading of AFR-001 directly).** Both evidence commits (`efac343`, `84e1168`) cite an Architecture Review Board review dated five days before GOV-011, the Board's own charter, was drafted, and both claim independent review completed, contradicting the explicit "no independent human reviewer exists" disclosure every other evidence commit in this repository makes. AFR-001 states this cannot be adjudicated by an audit and requires a governance-authority-level process outside any single EWO's, AFR's, or this charter's own authority.

  **Determination: does not block Act 3; accepted as deferred governance work, explicitly, not silently.** Three reasons. First, the anomaly concerns the *evidence commit's own narrative integrity*, not a challenge to ARCH-001's substantive content — no defect in the constitutional architecture itself has been identified anywhere in this repository's history. Second, Act 3's own initial roadmap (§7) cites ARCH-008 — cleanly, unambiguously Approved, with two undisputed evidence commits — as its authorizing architecture, not ARCH-001 directly; Act 3's own engineering work does not newly depend on resolving this question. Third, this condition is not new to Act 3: every Engineering Work Order from EWO-001 through EWO-016 has already been conducted under this exact same uncertified-baseline condition, disclosed by AFR-001 since 2026-07-20; Act 3 inherits, rather than introduces, this risk. This charter records it as a standing, named action item for whoever eventually convenes an Architecture Review Board or equivalent governance-authority process — not as work Act 3 itself is authorized or expected to resolve.

- **Disputed Bootstrap Authority State (GOV-011 §3.1, independently reverified against ADR-0011 §12.6/§19 and the GOV-004 evidence commit `1c1aaef` directly in this task).** ADR-0011 §12.6 states a successful validation disposition terminates Bootstrap Authority State as a non-discretionary consequence; the GOV-004 evidence commit's own closing line states "this act does not reactivate, further suspend, or terminate Bootstrap Authority State." The two texts disagree, and no governance adjudication of the question has occurred.

  **Determination: does not block Act 3; deferred.** Both readings agree GOV-003 and GOV-010 are, at minimum, Operative — the decision-making authority Act 3's own Engineering Work Orders will rely on (GOV-010 §4–§5, Class E approval) is unaffected by which reading eventually prevails.

- **GOV-011 and GOV-012 remain formally unapproved** (self-disclosed, "Not yet approved," in both documents' own Approval Status tables). Act 3 does not depend on a convened Architecture Review Board — the STD-031 lifecycle's own Independent Engineering Review, realized as disclosed Founder self-review, is the operative review mechanism, exactly as EWO-014/015/016 already demonstrated. **Deferred, not blocking.**

- **STD-001's currently operative content (v0.3.0–v0.4.0, registering the EMO/EMR/RSS/ACR/AFR document families this charter itself cites) has no independently located approval evidence distinct from the earlier v0.1.0/v0.2.0 evidence commits** (§6 above). STD-031 itself already depends on STD-001 for identifier and versioning rules; this charter does the same. **Deferred, not blocking** — no defect in STD-001's actual content has been identified, only an evidentiary gap in its own approval trail, and every EWO to date has already relied on the same content without treating this as a blocker.

- **No independent human reviewer exists anywhere in this repository** (structural, disclosed consistently at every review and approval to date, including this one). Not unique to Act 3; **accepted as an ongoing, disclosed condition**, not a defect this charter can cure.

**Engineering risks:**

- **Zero concrete Effect Providers and zero integration-test coverage exist as of this baseline** — the entire Effect Runtime (2,888 LOC, the workspace's largest crate) is unproven end to end. This is not a risk Act 3 defers; it is the risk Act 3 exists to retire (§1, §4).
- `services/actor-directory` and `services/audit-pipeline` remain interface-only stubs. Not in Act 3's initial roadmap scope (§2); may become in-scope for a later Act 3 Engineering Work Order if a Provider's own design requires either, but no such requirement is assumed here.

**Architectural risks:**

- **Architecture Baseline is not certified as Version 1.0** (AFR-001 §9, Certification Statement). Gates 1–3 passed; Gate 4 fails on the ADR-0017/ARCH-001 finding alone (Finding 1, the EWO-010 dangling citation, is resolved). This is the same condition as the governance risk above, viewed from the architecture side — **deferred, not blocking**, for the identical reasons.
- ARCH-002 through ARCH-007 (excepting ARCH-001, disputed, and ARCH-008, clean) remain formally `Pending` approval in their own Approval Status tables, with no separate evidence commit located for any of them. Every EWO to date has proceeded on this same basis. **Deferred, consistent with established project practice**, not a new condition Act 3 introduces.

**Not treated as a risk, but disclosed for completeness:** `EWO-003-Message-Gateway.md` exists on disk, untracked, corresponding to Runtime functionality already implemented in Act 1 history, with no `ER-003`. This predates Act 2 and Act 3 alike and is not an Act 3 engineering or governance risk; it is named here only so it is not mistaken for something this charter overlooked. Its correction, if pursued, is Act 1/Act 2 housekeeping, not an Act 3 Engineering Work Order.

## 9. Exit Criteria

Act 3 may be declared complete only when all of the following are independently verifiable from repository evidence:

1. Every Objective in §4 is satisfied, with evidence, not merely asserted.
2. Every item in §5's Success Criteria holds simultaneously, verified fresh (not relied upon from an earlier, potentially stale, check).
3. At least one full EWO → IER → Founder Approval → Publication → Implementation → IIR → Engineering Report → Publication → Founder Acceptance → Publication → Baseline Update cycle (STD-031 §6) has completed for a genuine Effect Provider — not merely for EWO-016's own retry-policy work, which predates and motivated this charter but does not itself satisfy it.
4. A dedicated Engineering Report (or a closing summary meeting STD-001 §47's own requirements) records, honestly, which of Act 3's initial roadmap items (§7) were completed, which were deferred, and why — on the identical "truthful engineering history" discipline this entire corpus already practices.
5. No new, undisclosed duplicate governance or competing lifecycle definition has been introduced by any Act 3 artifact — verified by the same discipline IER-STD-031 and IER-STD-031-R2 already applied to STD-031 itself.

Closing Act 3 does not require resolving the governance risks disclosed in §8 as deferred; it does require that this charter's own disclosure of them remains accurate and undisputed at closing time, or is updated if their status has changed.

## 10. Authority

This is a Governance-tier decision, approved on the same basis as GOV-003, GOV-004, GOV-010, GOV-011, and GOV-013 — authorizing a new phase of engineering work is not a Class B (Architectural) content decision within already-approved architecture, and is therefore not delegable to the Chief Architect role (currently vacant, GOV-003 §3.2). The Founder (Denver Jacobs), exercising GOV-003 §3.1's retained final authority, is the Approval Authority for this charter. Every individual Act 3 Engineering Work Order remains a separate, later decision, ordinarily Class E (Implementation) within this charter's and ARCH-008's already-approved scope, ratified per GOV-010 §4–§5 exactly as EWO-014, EWO-015, and EWO-016 already were.

## References

Internal:

- GOV-003 — Governance Model
- GOV-004 — Engineering Principles (Approved, commit `1c1aaef`)
- GOV-010 — Decision Framework (Approved, Act 2, commit `11cab8d`)
- GOV-011 — Architecture Review Board Charter (Draft; §16 and ACR-001 §16 both explicitly disclaim authorizing Act 3, which this document supplies)
- GOV-013 — Engineering Lifecycle (Approved, commit `3ac7cfb`)
- STD-001 — Documentation Standards (§5, §7, Appendix B — controlled-document-family registry; this charter's own filing note)
- STD-031 — Engineering Work Order Lifecycle Standard (v0.2.1, Approved, commit `f7a2a16`)
- ARCH-008 — Effect Runtime Architecture (v0.4.3, Approved)
- AFR-001 — Architecture Freeze Review (read directly for §5 Gate 4, §6 Outstanding Future Work Register, §7 Outstanding Risks Register, §9 Certification Statement)
- ADR-0011 — Bootstrap Approval Authority (§12.6, §19, §22 — "No Act 3 exists anywhere in this document")
- EWO-014, EWO-015, EWO-016 and ER-015, ER-016, ER-017 (cited as demonstrated Founder Approval / Founder Acceptance precedent)

Source evidence (independently re-verified during this charter's own preparation, not restated from memory):

- Runtime repository, `git log`, `git status`, `git fsck`, HEAD `29ea55c`, and direct source-tree inspection (`grep -rn "trait Provider"`, `find -iname "*provider*"`, `tests/`, `examples/` contents) confirming no concrete Effect Provider exists.
- Documentation repository, `git log --all --oneline | grep -i "approv\|evidence\|accept"`, cross-checked against each cited document's own Approval Status table.
- `adrs/ADR-0011-Bootstrap-Approval-Authority.md` §12.6, §19, §22, read directly.
- `governance/GOV-011-Architecture-Review-Board-Charter.md` §3.1, §16, read directly.
- `consolidation/AFR-001-Architecture-Freeze-Review.md` §5 (Gate 4), §6, §7, §8, §9, read directly.
- `governance/GOV-004-Engineering-Principles.md` evidence commit `1c1aaef`, read directly via `git show`.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-29 | Denver Jacobs (AI-assisted) | Initial authorization and charter. Establishes Act 3 (Production Capability), scoped to Effect Providers, Runtime external capability, Provider lifecycle, end-to-end execution, Runtime validation, and demonstration systems; explicitly excludes distributed runtime, marketplace, cloud platform, intelligence layer, production Control Centre, and any future Act. Records Entry Criteria (Runtime/Documentation baselines, approved STD-031/GOV-013, demonstrated EWO-016 precedent), an initial, non-binding roadmap (first concrete Effect Provider through Runtime capability expansion), and Exit Criteria. Discloses and dispositions, without concealment, four governance/architectural risks inherited from prior Acts (the ADR-0017/ARCH-001 evidence anomaly, the disputed Bootstrap Authority State, GOV-011/GOV-012's own unapproved status, and ARCH-002–ARCH-007's pending approval) — each determined to be deferred, non-blocking, and consistent with established project practice, not silently carried forward. No architecture amended. No Engineering Work Order authorized by identifier. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-29 |
| Founder Authorization | Denver Jacobs, Founder, exercising GOV-003 §3.1's retained final authority for a Governance-tier decision (§10 above) | **Approved** — Act 3 formally authorized; charter effective on publication of the evidencing commit | 2026-07-29 |

This charter is genuinely **Approved** on the ordinary, mutable Approval Status convention this repository's engineering-tier and most-recently-approved documents use (demonstrated: EWO-014, EWO-015, EWO-016, and STD-031 v0.2.1's own Approval Status tables, each populated in place). It is not a constitutional-tier document under ADR-0011's exact-byte-identity convention; its `Approved` status is exactly what it appears to be, verifiable from this file alone.
