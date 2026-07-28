---
document_id: EWO-016
title: "ConstraintSet-Based Retry Policy"
version: 0.1.3
status: Implemented
author: Denver Jacobs
owner: Denver Jacobs
reviewers: TBD
created: 2026-07-28
last_updated: 2026-07-28
classification: Public
related_documents:
  standards:
    - STD-001
  architecture:
    - ARCH-008 (v0.4.3, Approved — architecture/ARCH-008-Effect-Runtime-Architecture.md) — §14 (capability attenuation-only model), §19.4 (retry-limit ownership, capability-ceiling/actor-narrowing precedence), §23.5 (no-new-gate reasoning this EWO reuses identically)
    - ARCH-002 (Runtime Architecture) — §9 ("constructible only by Capability Authority"; fresh-never-cached validation)
    - ARCH-001 (Constitutional Architecture) — the non-amplification rule this EWO's own narrowing-enforcement requirement directly implements: "a capability-issuing capability's own constraint set defines the maximum authority anything it mints may carry"
  adrs:
    - ADR-0017 (Approved — Bootstrap Capability Trust Root)
  predecessor: "EWO-015 / ER-016 (Retry Architecture Implementation) — the governing, closed milestone this EWO completes the one deliberately deferred input for; EWO-015's own `capability_limit: Option<u32>` parameter on `EffectCoordinator::decide_retry` already exists and is unmodified by this EWO"
  reported_by: "The next sequentially available Engineering Report identifier per STD-001 §7 at the time this EWO is implemented — not assumed in advance"
  base_state:
    runtime_head: c5959bb5a2b26b816a17ab1bdc4e77c6183a8c22
    docs_head: 1c50d55dc5574b957fd5e25ed4ded8582fd74fc7
---

# EWO-016 — ConstraintSet-Based Retry Policy

Registered per STD-001 §46 (Engineering Work Orders). This authorizes implementation work only. It does not itself constitute approval, and does not amend ARCH-008 or any other architecture, standards, or governance document.

## Document Control

| Field | Value |
|---|---|
| Document ID | EWO-016 |
| Title | ConstraintSet-Based Retry Policy |
| Version | 0.1.3 |
| Status | **Implemented** (STD-001 §12 — implementation verified; Founder-accepted; closed) |
| Author | Denver Jacobs |
| Created | 2026-07-28 |
| Architecture authority | ARCH-008 v0.4.3 (Approved) §14, §19.4, §23.5; ARCH-002 §9; ARCH-001 (non-amplification rule) |
| Dependencies | EWO-015 / ER-016 (Retry Architecture Implementation) — closed, satisfied |
| Prerequisites | None outstanding |
| Implementation repository | `synapse-runtime` |
| Review requirement | Independent Engineering Review required before Founder Approval, per this corpus's own established EWO lifecycle |

---

## 1. Purpose

EWO-015 (Retry Architecture Implementation) built a complete, deterministic Retry Decision mechanism with exactly one input deliberately left unconnected: `capability_limit: Option<u32>`, the parameter `EffectCoordinator::decide_retry` already accepts and already correctly combines with the requesting actor's own declared limit via `min(capability, actor)` — but which `Runtime::maybe_schedule_retry` currently supplies as a hardcoded `None` on every call, since no capability-declared retry constraint exists anywhere in the system yet.

This EWO completes the capability side of that path. It introduces the first concrete dimension of `ConstraintSet` — a currently genuinely empty, zero-field unit struct — carrying an immutable, capability-declared maximum-total-attempts value, together with the real, previously entirely absent narrowing-enforcement mechanism `synapse-capability-authority`'s own `issue` and `attenuate` methods require to make that declaration constitutionally meaningful. It does not touch, reinterpret, or extend EWO-015's own retry-decision logic, which already correctly consumes this value once supplied.

---

## 2. Background

**The current `ConstraintSet`** (`common/src/lib.rs:40`) is `#[derive(Debug, Clone, Default)] pub struct ConstraintSet;` — a genuine zero-field unit struct. Its own doc comment already names its intended scope: "time, usage count, budget, rate, scope, delegation depth, and similar — per the Architecture Review Board's capability-model findings. Internal representation is deferred to implementation." Nothing has yet given it a field.

**Current capability construction** (`core/capability-authority/src/internal.rs`): both `issue` and `attenuate` accept a `constraints: ConstraintSet` / `narrower: ConstraintSet` parameter respectively, and both write it into the newly minted `Capability` unconditionally, without reading or comparing it to anything. `attenuate`'s own doc comment states this explicitly: *"`narrower`'s concrete representation is deferred along with `ConstraintSet` itself... there is currently no constraint data to narrow against, so this parameter is accepted but not yet inspected."* Non-amplification is enforced today on exactly one concrete dimension — `operations`, inside `issue` only, via a subset check against the canonical issuer's own operations, returning `RuntimeError::ExceedsIssuingCeiling` on violation (`core/capability-authority/src/internal.rs:407-415`). `attenuate` enforces no equivalent check for `operations` because its own signature never lets a caller supply different ones at all — target and operations are copied verbatim from the canonical source, which is itself the reason attenuation can never amplify those two dimensions today. **No equivalent structural guarantee exists for constraints, on either path**, because there is currently no constraint data of any kind to compare.

**The existing, already-implemented retry-decision input**: `EffectCoordinator::decide_retry(effect, outcome, idempotency, capability_limit: Option<u32>) -> RetryDecision` (`services/effect-coordinator/src/internal.rs`) already implements the exact narrowing formula this EWO must supply real data to: `effective_limit = min(capability_limit, actor_declared_limit)` when both are present; whichever is present alone, when only one is; unbounded when neither is. This function's own signature is not modified by this EWO.

**The current, exact integration point**: `Runtime::maybe_schedule_retry` (`runtime/src/lib.rs`) computes `let capability_limit = None; // Phase 1 — see doc comment above.` immediately before calling `decide_retry`. This is the one line this EWO's implementation replaces with a real resolution.

**The retained capability already available at exactly that point**: EWO-015 already added `Runtime::retry_dispatch_material: HashMap<EffectId, (Capability, Vec<u8>)>`, populated once at original request time and consulted by `dispatch_retry`. The same map, already keyed by the same `EffectId` `maybe_schedule_retry` already resolves from its own `attempt` parameter, already holds the exact `Capability` object this EWO needs to read a constraint from — no new retention mechanism is required.

**Why the Effect Coordinator itself requires no redesign**: `decide_retry` already treats `capability_limit` as an opaque, pre-resolved `Option<u32>` — it has no awareness of, and this EWO introduces no awareness of, where that value came from. `services/effect-coordinator`'s own crate dependencies (`synapse-common`, `synapse-timer` only) remain completely unaffected — the resolution this EWO adds happens entirely inside `runtime`, which already, legitimately depends on `synapse-capability-authority`.

---

## 3. Architectural Authority

**Why `ConstraintSet` is the correct home.** ARCH-001's own constitutional text states: "a capability-issuing capability's own constraint set defines the maximum authority anything it mints may carry," and ARCH-002 states: "issuance requires the issuer's own constraint set to bound whatever it mints, recursively." `ConstraintSet` is not a candidate this EWO selects among several — it is the object the already-approved constitutional architecture already assigns exactly this recursive-narrowing responsibility to. `ConstraintSet`'s own doc comment independently, already names "usage count... and similar" as within its intended scope — a maximum-total-attempts value is a direct instance of that already-declared intent, not a new concept requiring its own architectural justification.

**Why a nested retry constraint, not a bare field.** `ConstraintSet`'s own doc comment names five other future dimensions (time, budget, rate, scope, delegation depth) beyond the one this EWO adds. A bare top-level field for each, as they arrive, degrades `ConstraintSet` into an unstructured collection of unrelated primitives with no shared validation shape and no way to express "this dimension was never declared" distinctly from "this dimension was declared permissively." A small, purpose-built nested structure — the same pattern this corpus already uses for `IdempotencyDeclaration`, `RetryIntent`, and `RetryDecision` — preserves `ConstraintSet` as a single, coherent home indefinitely, establishing the pattern once rather than needing to retrofit it later.

**Why Capability Authority owns narrowing enforcement.** ARCH-002 §9 confirms `Capability` is "constructible only by Capability Authority" — no other component can mint or attenuate one, so no other component can be the enforcement point for a rule about what minting or attenuation may produce.

**Why Runtime resolves the value.** Runtime is already the sole component holding a retained `Capability` reference at the point this value is needed (`retry_dispatch_material`), and is already the sole cross-component orchestrator (ADR-0016 Rule 1) connecting Capability Authority's own domain to the Effect Coordinator's own domain — exactly the role it already plays for every other Effect-related sequence.

**Why Effect Coordinator remains unchanged.** `decide_retry`'s own signature already accepts `capability_limit` as an opaque `Option<u32>` (EWO-015). Nothing this EWO does requires that signature, or the Effect Coordinator's own crate dependencies, to change.

**Why Timer remains unchanged.** This EWO resolves a *ceiling on how many times* a retry may occur; it says nothing about *when*. Timer-based scheduling (EWO-015) is entirely orthogonal and untouched.

**Disclosure regarding review evidence.** This EWO's own drafting was informed by a completed architecture review ("ConstraintSet-Based Retry Policy Architecture Review," concluding `READY FOR EWO`) conducted as part of this same engineering effort. That review is cited here as supplied analytical evidence informing this EWO's own drafting decisions; it is not itself a committed repository artifact, and its conclusions are treated as authoritative only insofar as they are independently, freshly re-derivable from the repository evidence cited throughout this document — which they are, as demonstrated in §2 above.

---

## 4. Objectives

1. Give `ConstraintSet` a concrete, additive retry-policy dimension.
2. Preserve default behavior for every capability without a declared retry constraint — identical to all current, tested behavior.
3. Enforce recursive non-amplification of the declared retry constraint during both issuance and attenuation.
4. Supply the resolved capability-declared limit to the existing, unmodified `decide_retry` retry-decision mechanism.
5. Preserve deterministic behavior throughout — no wall-clock read, no mutable external lookup, no caching of a resolved value anywhere outside the single, per-call resolution point.
6. Preserve fresh, uncached capability validation, unconditionally, for every retry dispatch.
7. Preserve existing Runtime, Effect Coordinator, Capability Authority, and Timer ownership boundaries exactly as already established.
8. Maintain full compatibility with every existing caller, test, and public/internal interface not explicitly named for change by this EWO.

---

## 5. Non-Goals

This EWO MUST NOT:

- change `RetryIntent`'s own fields or semantics;
- change `RetryDecision`'s own variants;
- change `EffectCoordinator::decide_retry`'s own signature;
- change any Effect Coordinator ownership boundary, or introduce any Effect Coordinator dependency on `synapse-capability-authority`;
- introduce a global, fixed numeric Runtime retry ceiling of any kind;
- implement retry backoff, exponential backoff, jitter, absolute deadlines, total retry duration, or retry budgets;
- implement circuit breakers;
- implement provider-specific, operation-specific, or failure-class-specific retry constraints;
- implement durable persistence of any capability or retry-policy data, or claim recovery across a Runtime restart;
- implement distributed-Runtime recovery of any kind;
- design a general-purpose, multi-dimensional constraint-validation framework beyond the one retry-constraint dimension this milestone requires;
- perform any Control Centre user-interface work;
- expand the `AuditEvent` schema, unless independent implementation review identifies this as strictly required for correctness (not anticipated).

---

## 6. Required Data Model

`ConstraintSet` gains one new, optional, nested field:

```text
pub struct ConstraintSet {
    retry: Option<RetryConstraint>,
}

pub struct RetryConstraint {
    max_total_attempts: std::num::NonZeroU32,
}
```

This is an illustrative shape, not a binding syntax requirement. The implementer MUST follow current repository conventions for controlled construction (matching how `Capability` itself is "constructible only by Capability Authority," `ConstraintSet`'s own new field MUST NOT be publicly, freely mutable in a way that lets a component outside `synapse-capability-authority` fabricate or alter a declared retry constraint after the fact). A constructor and read accessor are required; a public, freely-assignable field is not, unless the implementer demonstrates it cannot be abused to bypass narrowing enforcement.

Required API surface, at minimum:

- a way to declare a `RetryConstraint` (specifically, a `max_total_attempts: std::num::NonZeroU32`) at `issue` time;
- a way to declare one at `attenuate` time (reusing `attenuate`'s own existing, currently-unused `narrower: ConstraintSet` parameter — no new parameter is required);
- a read accessor reachable from `Capability::constraints()` (already public, already exists, unmodified) down to the declared `max_total_attempts` value;
- a narrowing comparison usable internally by Capability Authority to compare a proposed child value against a canonical parent value.

`ConstraintSet` and `RetryConstraint` MUST derive whatever this crate's own existing sibling types already derive for equivalent purposes (`Debug`, `Clone`, `Default`, and `PartialEq`/`Eq` where meaningful comparison is required for narrowing checks and tests) — no speculative generic constraint trait or framework is authorized. `RetryConstraint`'s own `max_total_attempts: NonZeroU32` field already derives `Debug`, `Clone`, `Copy`, `PartialEq`, `Eq`, `PartialOrd`, `Ord`, and `Hash` from `std::num::NonZeroU32` itself; a `Default` impl for `RetryConstraint` is NOT authorized, since no zero-equivalent default retry constraint exists — the "no constraint declared" state is represented exclusively by `ConstraintSet.retry: None`, never by any value of `RetryConstraint` itself. `max_total_attempts: Option<u32>` — the shape considered during this EWO's own drafting — is deliberately rejected: it would have required a second, independent "is this `Some(0)` or `None`" distinction on top of the already-necessary `ConstraintSet.retry: Option<RetryConstraint>` distinction, for no behavioral gain (§7).

---

## 7. Required Constraint Semantics

**Meaning, stated precisely and required to be stated identically in the resulting implementation's own doc comments:**

> `max_total_attempts` is the maximum number of provider attempts permitted for one Effect, including its initial provider attempt. It is never a count of retries alone.

Worked examples, required to appear in the implementation's own doc comments verbatim in substance:

```text
1    → the initial attempt may still occur; no retry may ever be scheduled for it
3    → the initial attempt, plus at most two retries — three attempts total
None → no capability-declared total-attempt limit (no `RetryConstraint` declared at all — `ConstraintSet.retry: None`)
```

`max_total_attempts` is represented as `std::num::NonZeroU32`, not `Option<u32>` or a plain `u32` — its minimum representable value is `1`, which already, exactly expresses "the initial attempt may occur; no retry may ever be scheduled." A `u32`-based design would additionally have had to represent `0`, and would have required `0` and `1` to behave identically (since `attempt_count`, Effect Coordinator, unchanged, is incremented at the very first dispatch, before any retry question is ever asked) — two distinct representations of the same behavior, requiring ongoing implementation and test discipline to keep in sync. `NonZeroU32` removes that redundant pair entirely: there is exactly one way to express "no retry permitted" (`1`), and it is inexpressible to declare `0`. `decide_retry`'s own existing comparison (`attempt_count >= effective_limit`) already, correctly produces this behavior once a real value is supplied via `.map(NonZeroU32::get)`, requiring no change to that comparison or to `decide_retry`'s own `Option<u32>` signature.

**This value MUST NOT gate the initial Effect request or dispatch in any way.** It is consulted for the first time only inside the existing, unmodified `Runtime::maybe_schedule_retry`, itself only reached after an attempt has already, genuinely occurred and reached a retry-eligible terminal outcome. A capability declaring a `RetryConstraint` of `1` (the minimum representable value) still fully authorizes the initial dispatch under its own existing `operations` grant — this EWO introduces no new admission-time check of any kind.

**Actor/capability composition, unchanged from EWO-015, restated for completeness:**

```text
actor limit only       → actor limit governs
capability limit only  → capability limit governs
both present            → minimum(actor limit, capability limit) governs
neither present         → no total-attempt limit is enforced
```

**Attenuation narrowing table, required exactly:**

```text
parent None,      child None       → valid
parent None,      child Some(N)    → valid narrowing
parent Some(P),   child Some(C), C <= P → valid narrowing
parent Some(P),   child Some(C), C > P  → invalid widening — rejected
parent Some(P),   child None            → invalid widening — rejected
```

**`narrower` is the child's complete resulting constraint set, not a delta.** `attenuate`'s own `narrower: ConstraintSet` parameter is not merged with, or overlaid onto, the parent's own constraint set — it entirely replaces it, subject only to the narrowing check above. There is no "leave unspecified, inherit the parent's value" mode: a caller wishing the child to retain the parent's own declared `max_total_attempts` value MUST re-declare that identical value explicitly in `narrower`. This is why the table's own last row (`parent Some(P), child None → invalid widening — rejected`) is correct as stated, not an edge-case oversight: omitting the field does not mean "inherit `P`," it means "declare no limit at all," which is strictly wider than any finite `P` and is therefore rejected. This mirrors the existing, unmodified `operations` field's own already-established behavior — `attenuate` does not merge operation sets either; it copies the canonical source's own operations verbatim (or, in a fuller future model, would be subject to the identical narrowing check), never treating an absent value as "inherit."

**Rejection, not clamping.** An invalid-widening attempt MUST be rejected outright, reusing the existing `RuntimeError::ExceedsIssuingCeiling` variant (`common/src/lib.rs`), already returned by `issue`'s own existing operation-set ceiling check for the identical underlying concern (a caller attempting to obtain more authority than its own issuer/source possesses). No new error variant is authorized unless independent implementation review demonstrates `ExceedsIssuingCeiling` is not semantically applicable — this EWO's own drafting evidence found no such reason. Silent clamping is explicitly rejected: it would produce a capability whose own declared value differs from what the caller believed they were requesting, an outcome this corpus's own truthfulness discipline (ADR-0015's precedent, applied here by analogy) does not permit for any authorization-adjacent fact.

**This applies to both `issue` and `attenuate`**, since both accept a caller-supplied constraint value today (`constraints`/`narrower` respectively) and neither currently validates it. `issue`'s own existing operations-ceiling check (against the canonical issuer) is the direct precedent for the equivalent retry-constraint check this EWO adds there; `attenuate`'s own currently-unused `narrower` parameter is the direct, already-present integration point for the equivalent check this EWO adds there.

The specification explicitly forbids:

- silently dropping a parent's own declared constraint when minting a child;
- replacing a constrained parent with an unconstrained child capability;
- any path by which a minted capability's own effective `max_total_attempts` could exceed its issuer's or source's own;
- bypassing the existing, unmodified `operations` narrowing check as a side effect of adding this one;
- any behavioral difference between the `issue`-time check and the `attenuate`-time check beyond what each method's own existing signature already, structurally requires.

---

## 8. Required Runtime Integration

`Runtime::maybe_schedule_retry` MUST be extended to:

1. Retrieve the retained `(Capability, Vec<u8>)` tuple for the Effect from the existing, unmodified `retry_dispatch_material` map, using the same `effect` value the method already resolves from its own `attempt` parameter.
2. Read that `Capability`'s own `constraints()` accessor (already public, already exists) down to its declared `max_total_attempts`, if any.
3. Resolve this into the exact `capability_limit: Option<u32>` value already passed to `decide_retry` — replacing the current hardcoded `let capability_limit = None;` with this real resolution, and nothing else in that method's own structure.
4. Preserve the method's own existing failure handling exactly: if `retry_dispatch_material` contains no entry for the Effect (a case already handled, structurally unreachable in practice but defensively guarded, per EWO-015's own established pattern), the existing `RuntimeError::UnknownTarget` path MUST remain unchanged — this EWO adds a read, not a new failure mode.
5. Preserve fresh capability validation exactly as EWO-015 already established it: `dispatch_retry`'s own call to `admit_message` remains completely unmodified by this EWO, and continues to perform full, unconditional, uncached revocation/expiry/integrity/domain validation on every retry dispatch, independent of and unaffected by this EWO's own retry-limit resolution step.
6. Introduce no second, duplicate storage of the resolved value anywhere — not on `EffectRecord`, not on any new Runtime field, not anywhere outside the single per-call resolution inside `maybe_schedule_retry` itself.

---

## 9. Required Compatibility Behaviour

- `ConstraintSet::default()` MUST continue to construct a value with no declared retry constraint (`retry: None`), identical in every observable respect to today's zero-field unit value.
- **Every existing `ConstraintSet` construction site across the workspace requires mechanical modification once the field is added — this is a compile-breaking change, not a zero-modification one.** `ConstraintSet` is today a genuine zero-field unit struct, constructed everywhere via the bare literal `ConstraintSet`. Direct `grep` search across the workspace (`grep -rl "ConstraintSet" --include="*.rs" .`, cross-checked against `grep -rn "ConstraintSet::default()\|Default::default()"`, which returns zero matches anywhere) confirms 131 construction occurrences across exactly 13 files: `common/src/lib.rs`; `core/capability-authority/src/{internal.rs,lib.rs}`; `core/message-gateway/src/lib.rs`; `runtime/src/lib.rs`; `runtime/examples/{actor_to_actor_messaging.rs,worker_pool.rs}`; `runtime/tests/{actor_supervision.rs,actor_to_actor_messaging.rs,bootstrap_grant.rs,bootstrap.rs,timer.rs,worker_pool.rs}` — none of them uses `ConstraintSet::default()`. Once `ConstraintSet` gains a field, Rust no longer permits the bare unit-struct literal for a struct with fields; every one of these 13 files requires the mechanical, purely syntactic replacement of the literal `ConstraintSet` with `ConstraintSet::default()` (or an explicit `ConstraintSet { retry: None }`), with no change in behavior or intent at any site. This replacement is REQUIRED for the workspace to compile at all, and MUST be performed as part of this EWO's own implementation, across all 13 files named in §12 — not merely the 4 files authorized there for semantic change. This EWO's own earlier drafting claim of "31+ call sites... continue to compile and pass without modification" was independently verified false during this document's own Independent Engineering Review and is corrected here.
- All existing EWO-015 tests (744 total at this EWO's own baseline, independently re-confirmed during this Independent Engineering Review by a fresh, full `cargo test --workspace --all-targets --all-features` run: 744 passed, 0 failed) MUST continue to pass, in substance, unmodified — only their `ConstraintSet` construction syntax, where present, requires the mechanical update described above.
- Every existing Effect and capability without a declared retry constraint MUST behave exactly as today: `capability_limit: None`, actor's own `max_attempts` alone governing, identical to every currently passing test.
- No serialization migration is required or authorized: no durable, serialized `ConstraintSet` representation exists anywhere in the current repository (confirmed: no `capability_authority` interaction anywhere in `services/persistence`), so no migration path can exist to define.

---

## 10. Determinism Requirements

Given identical:

- `Capability` constraint data (immutable once issued, per existing, unmodified `Capability` semantics);
- requesting-actor `RetryIntent`;
- Effect attempt count;
- attempt-level failure outcome;
- idempotency classification;

the resulting Retry Decision MUST be identical, every time, exactly as EWO-015 already established and independently verified for the actor-only case. Policy resolution itself MUST be deterministic: it MUST NOT depend on wall-clock state, mutable external configuration, mutable shared actor or capability metadata read from anywhere other than the retained, immutable `Capability` object itself, network state, or any cache independent of that single retained object. The retained, immutable `Capability` is the sole source of truth for this value at every resolution — reading it twice, at two different times, for the same Effect, MUST yield the identical result, since nothing about it can change without minting an entirely new, distinct `Capability` object.

---

## 11. Security and Abuse Resistance

Proportionate to this milestone, the implementation MUST ensure:

- no attenuation path may produce a capability whose own effective `max_total_attempts` exceeds its source's own;
- no issuance path may produce a capability whose own effective `max_total_attempts` exceeds its issuer's own (mirroring the existing `operations`-ceiling check exactly);
- no unconstrained descendant may be minted from a constrained parent;
- fresh capability validation before retry dispatch (EWO-015, unmodified) is never bypassed, weakened, or made conditional on this EWO's own retry-limit resolution;
- no new capability operation string or authorization gate is introduced for registering or reading a retry constraint (identical "no new gate" reasoning to ARCH-008 §23.5, applied here);
- no new retry-scheduling path, queue, or timer mechanism is introduced;
- a capability's own declared retry constraint cannot be mutated after issuance — obtaining a different value requires issuing or attenuating a genuinely new, distinct capability, never altering an existing one;
- no actor-controlled input can alter a `Capability` object's own constraint data once retained by Runtime for a pending Effect.

This EWO explicitly does not attempt, and must not be read as attempting, a complete rate-limiting or denial-of-service architecture; it resolves exactly the one constraint dimension named in §6.

---

## 12. Required Implementation Scope

Authorized files, for semantic implementation (the retry-constraint data model, narrowing enforcement, and Runtime resolution):

```text
common/src/lib.rs
core/capability-authority/src/lib.rs
core/capability-authority/src/internal.rs
runtime/src/lib.rs
```

Additionally authorized, for the mechanical, purely syntactic `ConstraintSet` → `ConstraintSet::default()` (or equivalent explicit-field literal) migration required at every existing construction site once the field is added (§9) — no semantic change of any kind is authorized in these files beyond that single mechanical substitution:

```text
core/message-gateway/src/lib.rs
runtime/examples/actor_to_actor_messaging.rs
runtime/examples/worker_pool.rs
runtime/tests/actor_supervision.rs
runtime/tests/actor_to_actor_messaging.rs
runtime/tests/bootstrap_grant.rs
runtime/tests/bootstrap.rs
runtime/tests/timer.rs
runtime/tests/worker_pool.rs
```

This two-tier list (13 files total) was independently derived by direct `grep` search across the workspace during this EWO's own Independent Engineering Review, superseding this document's own earlier, incomplete 4-file-only list and the false "31+ call sites... without modification" claim it accompanied (§9, corrected).

Test modules colocated within the semantic-implementation files above, or within existing crate test locations already used by their own respective existing test suites, are authorized.

This EWO does NOT authorize changes to:

```text
services/effect-coordinator
services/timer
services/persistence
```

unless a future Independent Implementation Review identifies an unavoidable, architecture-supported requirement — none is anticipated, since `decide_retry`'s own signature already accepts exactly the value this EWO resolves.

No `Cargo.toml` change is authorized unless independent implementation review demonstrates one is unavoidable — none is anticipated, since `runtime` already depends on `synapse-capability-authority` and no other crate's dependency graph requires alteration.

---

## 13. Required Test Coverage

**Constraint data model:** a default `ConstraintSet` carries no retry constraint; an explicitly declared `RetryConstraint` is retained and readable via `Capability::constraints()`; a `Capability`'s own constraint data cannot be mutated after construction (no `pub` setter of any kind exists on `Capability` itself, confirmed by the existing, unmodified struct).

**Issuance:** unconstrained direct issuance (no ancestor to violate) succeeds; issuance declaring the minimum representable value (`1`) succeeds; issuance declaring a larger representable value succeeds; the issued capability's own `constraints()` exposes exactly the declared value. (A `RetryConstraint` of `0` is inexpressible by construction of `NonZeroU32` itself — no runtime test is required or possible for this case.)

**Attenuation:** unconstrained parent → unconstrained child succeeds; unconstrained parent → constrained child succeeds (narrowing from "no limit" to "some limit" is always valid); constrained parent → equal-value child succeeds; constrained parent → smaller-value child succeeds; constrained parent → larger-value child fails with `RuntimeError::ExceedsIssuingCeiling`; constrained parent → unconstrained child fails with the identical error; the existing, unmodified `operations` narrowing check continues to be enforced independently and simultaneously with the new retry-constraint check; a failed attenuation attempt leaves the parent capability's own canonical record completely unmodified (mirroring the existing, unmodified behavior already proven for a rejected `operations` widening attempt).

**Runtime resolution:** a retained capability with no declared retry constraint supplies `capability_limit: None` to `decide_retry`, identical to current behavior; a retained capability with a declared constraint supplies exactly that value; an actor-only limit continues to behave exactly as EWO-015 already tests; a capability-only limit governs correctly; both present together correctly resolve to their minimum; a declared value of `1` (the minimum representable value) correctly denies every retry after the initial attempt; a declared value of `3` permits no more than three total attempts; `Runtime` reads the constraint from the correct Effect's own retained capability — proven by two Effects, each holding a distinct capability with a distinct declared value, resolving completely independently of one another; the existing, unmodified failure handling for missing dispatch material remains exactly as it is today.

**Capability revocation:** a retry may be legitimately scheduled under a currently-valid, constraint-bearing capability; revoking that capability before the scheduled retry timer fires still causes the existing, unmodified admission path (`dispatch_retry` → `admit_message` → `Capability Authority::validate`) to deny the retry dispatch — proving the new retry-constraint resolution never substitutes for, races against, or bypasses revocation enforcement.

**Determinism:** repeated resolution of the identical, immutable retained `Capability`'s own constraint produces the identical `capability_limit` value on every call; identical full retry-decision inputs (state plus this newly-real value) produce identical `RetryDecision` outcomes, extending the existing `decide_retry_is_deterministic_for_identical_state_and_inputs` test pattern (EWO-015) to a genuinely non-`None` capability limit for the first time.

**Regression:** the full existing EWO-015 retry test suite continues to pass unmodified; the full existing Capability Authority test suite continues to pass unmodified; the full existing Runtime test suite continues to pass unmodified; full workspace verification (`cargo fmt`, `cargo clippy -D warnings`, `cargo build`, `cargo test`, all `--workspace --all-targets --all-features`) passes cleanly from a forced-clean state.

---

## 14. Error Handling

- An attenuation attempt that would widen the retry constraint MUST return `Err(RuntimeError::ExceedsIssuingCeiling)`, identical to the existing, unmodified behavior for an `operations`-widening attempt, and MUST NOT mutate the source/parent capability's own canonical record.
- An issuance attempt whose declared retry constraint would exceed the issuer's own MUST return the identical error, on the identical basis.
- `Runtime::maybe_schedule_retry` finding no entry in `retry_dispatch_material` for the Effect MUST continue to produce the existing, unmodified `RuntimeError::UnknownTarget` — this EWO introduces no new failure mode at that point.
- A capability validation failure at retry-dispatch time (revocation, expiry, integrity, domain mismatch) MUST continue to be handled exactly as EWO-015 already, fully establishes — completely independent of, and unaffected by, this EWO's own retry-constraint resolution.
- The absence of a declared retry constraint (on either the capability or the requesting actor's own `RetryIntent`) MUST continue to mean "no total-attempt limit is enforced," exactly as today — this is not an error condition of any kind.
- No malformed numeric state requires handling beyond what Rust's own type system already precludes by construction: `NonZeroU32` makes a zero-valued `max_total_attempts` inexpressible, and `Option<RetryConstraint>` on `ConstraintSet` is the sole representation of "no constraint declared." No silent fallback from an invalid, rejected widening attempt to an unconstrained result is authorized under any circumstance.

---

## 15. Implementation Procedure

The future implementation task MUST follow this project's own established procedure:

```text
Repository verification
→ specification-to-code trace
→ baseline verification
→ minimal implementation
→ focused tests
→ full verification
→ scope audit
→ one Runtime commit
→ push
→ implementation report
→ independent implementation review
```

This EWO itself authorizes no implementation and performs none.

---

## 16. Verification Requirements

The future implementation MUST run, from a forced-clean state, and report exact results for:

```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo build --workspace --all-targets --all-features
cargo test --workspace --all-targets --all-features
git diff --check
```

Focused tests for every directly modified crate (`synapse-common`, `synapse-capability-authority`, `synapse-runtime`) MUST also be run and reported. The baseline to compare against is 744 passed, 0 failed (this EWO's own base state, per §Document Control); the final count MUST be independently re-derived by direct summation, not asserted from a single aggregate figure, consistent with this engineering effort's own established validation discipline.

---

## 17. Acceptance Criteria

Implementation under this EWO may be accepted only when:

1. `ConstraintSet` carries a concrete, additive, nested retry-policy dimension.
2. A `ConstraintSet` with no declared retry constraint preserves every currently-tested behavior exactly.
3. Capability Authority enforces retry-constraint narrowing during both `issue` and `attenuate`.
4. Every widening attempt, at either path, fails explicitly with `RuntimeError::ExceedsIssuingCeiling` and mutates nothing.
5. `Runtime::maybe_schedule_retry` resolves the capability-declared limit from the already-retained, immutable `Capability` in `retry_dispatch_material` — never from a new cache, never re-resolved at capability-issuance time and duplicated elsewhere.
6. That resolved value is supplied to the existing, entirely unmodified `EffectCoordinator::decide_retry` call.
7. `services/effect-coordinator` gains no new dependency on `synapse-capability-authority`, and its own public API surface is unchanged by this EWO.
8. Timer remains the sole retry-scheduling mechanism, untouched.
9. Fresh, uncached capability validation remains unconditionally mandatory for every retry dispatch, unmodified from EWO-015.
10. No global, fixed numeric Runtime retry ceiling is introduced anywhere.
11. No non-goal named in §5 is implemented.
12. Every test named in §13 exists and passes; full workspace verification (§16) passes cleanly.
13. The Documentation repository remains unchanged throughout Runtime implementation.
14. An Independent Implementation Review finds no CRITICAL or MAJOR defect.

---

## 18. Deferred Work

Recorded here as explicitly, deliberately out of scope — none of the following is implemented, specified, or begun by this EWO:

- retry backoff, exponential backoff, jitter, absolute deadlines, total retry duration, retry budgets, circuit breakers;
- provider-specific, operation-specific, or failure-class-specific retry constraints;
- policy-source audit enrichment (recording, in the retry audit events, whether the actor or the capability supplied the effective limit) — a genuine, low-risk future enhancement, not required for this milestone's own correctness;
- durable persistence of capability or retry-policy data, and restart/recovery behavior of any kind;
- distributed-Runtime Effect recovery;
- a generalized, multi-dimensional constraint-validation framework extending beyond the one retry-constraint dimension this milestone requires — the other five dimensions `ConstraintSet`'s own doc comment already names (time, usage count generally, budget, rate, scope, delegation depth) remain their own, separately authorized future milestones;
- SynapseOS Control Centre policy editing or visualization of any kind.

---

## 19. Risks

| Risk | Mitigation / Required Test |
|---|---|
| Introducing the first genuinely real narrowing-enforcement logic into a previously entirely inert `ConstraintSet` mechanism, with no prior precedent in this codebase to follow exactly | §13's full Attenuation and Issuance test categories; direct reuse of the already-proven `operations`-ceiling pattern rather than an invented one |
| Accidentally widening capability authority through an incorrectly ordered or incorrectly directioned comparison | Explicit narrowing table (§7), required exact test coverage for every row of it (§13) |
| Conflating "total attempts" with "retries" in either the implementation's own documentation or its test names | §7's own mandatory, verbatim worked examples |
| Treating a declared minimum value (`1`) as denial of the initial Effect invocation itself | §7's explicit prohibition, restated in §Non-Goals-adjacent language; a dedicated test proving the initial dispatch is unaffected |
| Duplicating capability-declared policy inside Effect Coordinator state | §8's explicit "no second, duplicate storage" requirement; `decide_retry`'s own signature is the only place this EWO is authorized to supply the value |
| Stale, cached policy resolution diverging from the capability's own current, true state | §10's explicit determinism requirement; the retained object itself is the sole source of truth, read fresh at every resolution |
| Breaking every existing `ConstraintSet` construction site across the workspace once a field is added — 13 files, confirmed by direct `grep`, all requiring the mechanical `ConstraintSet` → `ConstraintSet::default()` substitution | §9's corrected compatibility requirement; §12's explicit two-tier (semantic / mechanical-migration) file-scope list; full regression suite (§13, §16) |
| Silently altering EWO-015's own already-accepted, closed semantics | This EWO touches no file EWO-015 itself introduced beyond the single named line in `maybe_schedule_retry`; `decide_retry`'s own signature and logic are explicitly, repeatedly named as unmodified throughout this document |
| Introducing an unjustified, invented global numeric ceiling to "simplify" the design | §5's explicit non-goal; §Acceptance-Criteria's own explicit prohibition |
| Scope creep into a general-purpose constraint or policy framework | §5's and §12's explicit file-scope and non-goal boundaries |

---

## 20. Reporting Requirement

An Engineering Report SHALL be authored recording: objective; implementation summary; every file modified; the exact narrowing semantics implemented, cross-referenced against §7's own table; the exact Runtime integration point changed (§8); compatibility confirmation (§9); every test from §13, with its outcome; exact verification results (§16, with independently re-derived totals); a scope audit confirming no non-goal (§5) was implemented and no unauthorized file (§12) was touched; commit hash and push confirmation; deferred work (§18), restated factually; and an explicit statement of readiness for Independent Implementation Review. Its own document identifier SHALL be derived from repository evidence at authoring time (STD-001 §7), not assumed in advance.

---

## 21. Disposition

**Implemented. Accepted. Closed.**

Independently reviewed pre-implementation (IER-016): 0 CRITICAL, 0 unresolved MAJOR (one MAJOR finding, IER-016-F03, corrected within the same review pass), two MINOR findings corrected (IER-016-F01, IER-016-F02), one non-blocking OBSERVATION recorded (IER-016-OBS-01).

Founder-approved, v0.1.2, 2026-07-28.

Published.

Implemented at Runtime commit `29ea55ced6348490f90bd7baeb08d3d4705f19ab` (`runtime: enforce capability-declared retry constraints (EWO-016)`).

Independently reviewed post-implementation (IIR-016): 0 CRITICAL, 0 MAJOR, 1 MINOR (a focused-test-count reporting error in the implementation report, corrected; no bearing on implementation correctness), 0 OBSERVATION. Concluded `IIR-016 COMPLETE — READY FOR ENGINEERING REPORT (ER-017)`.

Recorded in `engineering-reports/ER-017-ConstraintSet-Based-Retry-Policy.md`.

Founder-accepted, v0.1.3, 2026-07-28. No unresolved CRITICAL or MAJOR finding, at either review stage, remains outstanding.

**EWO-016 is closed.** No further implementation, correction, or review is authorised under this EWO; any future work on retry constraints, `ConstraintSet`, or its deferred dimensions (§18) requires its own, separately authorized Engineering Work Order.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-28 | Denver Jacobs (AI-assisted) | Initial Draft. Authored from ARCH-008 v0.4.3 §14/§19.4/§23.5, ARCH-001's non-amplification rule, ARCH-002 §9, the closed EWO-015/ER-016 milestone, and direct inspection of the current, entirely inert `ConstraintSet`/`issue`/`attenuate` implementation (confirmed: the existing `RuntimeError::ExceedsIssuingCeiling` variant already used for `operations`-ceiling violations is directly reusable, unmodified, for retry-constraint widening violations too). Specifies a nested `RetryConstraint` dimension on `ConstraintSet` (`max_total_attempts: Option<u32>`, counting the initial attempt); real narrowing enforcement at both `issue` and `attenuate`; Runtime-side resolution from the already-retained capability in `retry_dispatch_material`, replacing exactly the one hardcoded `None` line in `maybe_schedule_retry`; and an unmodified `decide_retry` signature. Informed by, but not itself constituting, the completed "ConstraintSet-Based Retry Policy Architecture Review" (`READY FOR EWO`), whose conclusions are treated as authoritative only insofar as independently re-derivable from the repository evidence this document itself cites. |
| 0.1.1 | 2026-07-28 | Denver Jacobs (AI-assisted) | Corrected per Independent Engineering Review IER-016 (findings below). §9/§12: corrected the false "31+ call sites... continue to compile... without modification" compatibility claim and the incomplete 4-file authorized-scope list — direct `grep` evidence (zero `ConstraintSet::default()` uses anywhere; 131 bare-literal `ConstraintSet` construction occurrences across 13 files) proves every one of those 13 files requires a mechanical `ConstraintSet` → `ConstraintSet::default()` migration once the field is added; §12 now authorizes those 9 additional files for mechanical-only change. §6/§7: changed `RetryConstraint.max_total_attempts` from `Option<u32>` to non-optional `std::num::NonZeroU32`, eliminating the redundant double-"no constraint" representation (`ConstraintSet.retry: None` vs. `Some(0)`) and the resulting `0`/`1`-equivalence complexity, with zero impact on `decide_retry`'s existing `Option<u32>` signature (resolved via `.map(NonZeroU32::get)`); updated worked examples, test coverage (§13), and error-handling language (§14) to match. §7: added an explicit clarification that `attenuate`'s `narrower: ConstraintSet` parameter represents the child's complete resulting constraint set, not a delta merged against the parent — resolving a latent ambiguity in the narrowing table's own last row. §19: updated the corresponding risk-table row to reflect the corrected file count and scope split. Status remains Draft; not Founder-approved; not published; not authorized for implementation. |
| 0.1.2 | 2026-07-28 | Denver Jacobs (Founder) | Governance disposition recorded — **no engineering-scope, requirement, or exclusion changed from 0.1.1**. Records the Founder's decision on the completed Independent Engineering Review IER-016 (concluding `EWO-016 INDEPENDENT ENGINEERING REVIEW COMPLETE — READY FOR FOUNDER APPROVAL`, one MAJOR finding corrected within the review pass — IER-016-F03 — two MINOR findings corrected — IER-016-F01, IER-016-F02 — one non-blocking OBSERVATION recorded, no CRITICAL or unresolved MAJOR finding): `status` transitions from `Draft` to **`Approved`**; §19's stale `0`/`1`-equivalence risk-mitigation language (superseded by 0.1.1's own `NonZeroU32` correction but not yet cleaned up at that time) is corrected for internal coherence; §21's Disposition section is rewritten to reflect the true, current reviewed-and-approved state, replacing the placeholder "not yet independently reviewed" language 0.1.0 originally carried; the Approval Status table is completed (Approval Authority recorded against Denver Jacobs, Founder, exercising Class E implementation-decision authority under GOV-010 §4–§5, in the absence of an identified delegate, on the identical basis EWO-014's and EWO-015's own approval dispositions already established). This EWO reserves its own dedicated version for this pure governance disposition, mirroring EWO-014's and EWO-015's own precedent (each used a final version exclusively for the Founder's disposition, with no engineering-scope change) — the only difference being that EWO-016's engineering corrections were recorded at 0.1.1, a version earlier than the corresponding corrections were recorded at in EWO-014/015 (which folded their own review-correction narratives into 0.1.0 directly), so the equivalent "pure disposition" version here is 0.1.2 rather than 0.1.1. |
| 0.1.3 | 2026-07-28 | Denver Jacobs (Founder) | Founder Acceptance recorded — **no engineering-scope, requirement, or exclusion changed from 0.1.2**. Records the Founder's acceptance of the completed Runtime implementation (commit `29ea55ced6348490f90bd7baeb08d3d4705f19ab`, `runtime: enforce capability-declared retry constraints (EWO-016)`), following an Independent Implementation Review (IIR-016: 0 CRITICAL, 0 MAJOR, 1 MINOR — a focused `synapse-common` test-count reporting error, independently corrected, with no bearing on the Runtime implementation's own correctness — 0 OBSERVATION; concluding `IIR-016 COMPLETE — READY FOR ENGINEERING REPORT (ER-017)`) and publication of the permanent Engineering Report (`engineering-reports/ER-017-ConstraintSet-Based-Retry-Policy.md`, commit `9ea9fa63481f9b69bd4be56cb019254043937a6a`): `status` transitions from `Approved` to **`Implemented`** (STD-001 §12 — "optional status for specifications whose implementation is verified"), the first use of this status value in this repository, applied rather than an invented equivalent (e.g. "Accepted"/"Closed") since STD-001 already defines it precisely for this purpose. §21's Disposition section is rewritten to record implementation, post-implementation review, Engineering Report publication, Founder Acceptance, and closure. The Approval Status table gains two rows (Independent Implementation Review; Founder Acceptance), completing the full pre- and post-implementation review/approval lineage this EWO's own lifecycle required. **EWO-016 is closed** by this disposition; no further work is authorised under it. |

## Approval Status

| Role | Name | Status | Date |
|---|---|---|---|
| Author | Denver Jacobs (AI-assisted) | Drafted | 2026-07-28 |
| Independent Engineering Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `EWO-016 INDEPENDENT ENGINEERING REVIEW COMPLETE — READY FOR FOUNDER APPROVAL` (one MAJOR finding — IER-016-F03 — and two MINOR findings — IER-016-F01, IER-016-F02 — corrected within the same review pass; one non-blocking OBSERVATION recorded; 0 CRITICAL, 0 unresolved MAJOR remaining) | 2026-07-28 |
| Approval Authority | Denver Jacobs, Founder, exercising Class E (Implementation) decision authority under GOV-010 §4–§5 in the absence of an identified delegate | **Approved** | 2026-07-28 |
| Independent Implementation Review | Denver Jacobs (Founder; disclosed self-review — Chief Architect role vacant, no independent third-party reviewer available) | `IIR-016 COMPLETE — READY FOR ENGINEERING REPORT (ER-017)` (0 CRITICAL, 0 MAJOR, 1 MINOR — reporting-accuracy correction only, no bearing on implementation correctness — 0 OBSERVATION) | 2026-07-28 |
| Founder Acceptance | Denver Jacobs, Founder, exercising Class E (Implementation) decision authority under GOV-010 §4–§5 in the absence of an identified delegate | **Accepted — EWO-016 Closed** | 2026-07-28 |
