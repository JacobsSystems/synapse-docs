---
document_id: ER-006
title: "Bounded Actor Mailboxes — Engineering Report"
version: 0.1.0
status: Draft
author: Claude (AI-assisted, under Denver Jacobs' direction)
owner: Denver Jacobs
created: 2026-07-13
last_updated: 2026-07-13
classification: Public
related_documents:
  reports_on: EWO-006 (work-orders/EWO-006-Bounded-Actor-Mailboxes.md)
  architecture:
    - ARCH-001
    - ARCH-002 (v0.2.0 — architecture/ARCH-002-Runtime-Architecture.md)
    - ARCH-003 (v0.4.0 — architecture/ARCH-003-Runtime-Integration-Architecture.md)
  adrs:
    - ADR-0015
    - ADR-0016
    - ADR-0017
  standards:
    - STD-001
---

# ER-006 — Bounded Actor Mailboxes — Engineering Report

Registered per STD-001 §47 (Engineering Reports). Informational only — this document creates no new requirement and does not itself authorize, approve, or publish anything. Nothing described here has been committed or pushed.

## 1. Repository Baselines

Verified before this report was authored:

- `synapse-runtime`: path `/home/sudonimm/Development/SynapseOS/synapse-runtime`, branch `main`, HEAD `5ccc7f9083a71adc6ee704b2322a701935765679`, `HEAD == origin/main`, 0 ahead / 0 behind, tracked working tree shows exactly the four authorized modified files, nothing staged, no untracked files.
- `synapse-docs`: path `/home/sudonimm/Development/SynapseOS/synapse-docs`, branch `main`, HEAD `e90404baa5140ce9004839bc51921c789777e003`, `HEAD == origin/main`, tracked working tree clean. The four expected untracked files (`.ai/ARCHITECTURAL-CONTEXT.md`, `maintenance/EMO-001-ActorHost-Single-Live-Instance.md`, `work-orders/EWO-003-Message-Gateway.md`, `work-orders/EWO-006-Bounded-Actor-Mailboxes.md`) confirmed present and untouched throughout. EWO-006 independently re-verified byte-identical to its approved version (sha256 `15d2d94eb54c75658ea724b5ce34f4f411c168be712dd13d56ac46a308dce877`, unchanged) before this report was written.
- EWO-006 (v0.1.0), ARCH-002 (v0.2.0), ARCH-003 (v0.4.0), ADR-0015, ADR-0016, ADR-0017, and the complete independent EWO-006 implementation review were read in full before this report was authored. The affected source (`core/actor-host/src/{lib,internal}.rs`, `core/actor-host/README.md`, `runtime/src/lib.rs`) was re-read directly against the source tree, and the complete diff re-inspected line by line, rather than assumed to match any prior summary — including the prior implementation report, which the independent review found to contain two incorrect test-count figures (corrected below).

## 2. Governing EWO and Architecture

EWO-006 v0.1.0 (`work-orders/EWO-006-Bounded-Actor-Mailboxes.md`), independently reviewed and disposed **APPROVE AS WRITTEN**, is the sole implementation authority for this work. Governing architecture, in descending order: ARCH-001; ARCH-002 v0.2.0 (§13 — bounded, finite mailbox capacity as a mechanism-level MUST, with mandatory non-silent, audited overflow rejection; §22 — bounded mailboxes among Mandatory conformance requirements); ARCH-003 v0.4.0 (§5, §18 — the mandatory bounded-mailbox conformance gap disclosed, unchanged, since EWO-001); ADR-0015 (Audit Emitter Failure Semantics); ADR-0016 (Trusted Core Interaction Rule); ADR-0017 (Bootstrap Capability Trust Root); STD-001 §46.

## 3. Objective

EWO-006 implemented finite actor-instance mailbox capacity with deterministic overflow rejection inside Actor Host, closing the one mandatory ARCH-002 conformance gap disclosed, unchanged, since Actor Host's own first implementation — while preserving every existing public interface, Runtime's own composition-root role, and existing audit behaviour exactly as they stood before this milestone.

## 4. Implementation Summary

A single private, fixed mailbox-capacity constant (`MAILBOX_CAPACITY: usize = 64`) was added to `synapse-actor-host`'s internal implementation. `ActorHostImpl::enqueue` now checks the target mailbox's current length against this constant (`mailbox.len() >= MAILBOX_CAPACITY`) before pushing; once a mailbox reaches capacity, the check rejects with the existing `RuntimeError::Overflow` and the incoming message is never inserted. Every other property of `enqueue`'s prior behaviour is unchanged: unknown-instance handling (`Err(UnknownTarget)`) is untouched; already-admitted messages and their FIFO order are never disturbed by a rejected enqueue; capacity and overflow remain isolated per actor instance, a direct, pre-existing consequence of the unchanged per-instance `HashMap<ActorInstanceId, Vec<Message>>` storage shape; and the target actor instance's lifecycle state is entirely unaffected — Lifecycle Guardian is not called and has no awareness of mailbox capacity or overflow.

## 5. Capacity Decision

ARCH-002 §13 requires mailbox capacity be "bounded and finite" as a mechanism-level MUST, but explicitly names the specific numeric value a deployment parameter (policy), not an architectural guarantee. No numeric default exists anywhere in the published architecture, in any prior EWO, or anywhere in the prior implementation. Per EWO-006's own "Bounded Design Decision," `64` was selected as the single open engineering decision this milestone authorized: small and conservative — sufficient for the current single-process, non-distributed Minimal Runtime Profile, in which no real actor-defined handler yet exists to consume mailbox contents — while comfortably exceeding the message count (fewer than five) any pre-existing test in this workspace enqueues, so no pre-existing test incidentally depended on or was threatened by this bound. The value is a fixed, documented, crate-internal constant (`pub(crate)`, not exported): no constructor parameter, no per-actor-instance or per-actor-definition value, and no configuration mechanism of any kind was introduced, exactly as EWO-006's Architecture Constraints require. The value is implementation policy, not an architectural guarantee, and may be revisited by a future milestone without requiring an ADR, provided capacity's ownership, mechanism, and reject-only policy remain unchanged.

## 6. Runtime Behaviour

`Runtime::submit_message`'s production code was **not changed** — confirmed both by direct inspection of the diff (every line added to `runtime/src/lib.rs` falls inside `#[cfg(test)] mod tests`, confirmed by comparing the module's own opening line number against every inserted line) and by re-reading the method's own unchanged source. Its existing step 6 — `self.core.actor_host.enqueue(&instance, message.clone())`, routed unconditionally to `self.reject_message(&message, reason)` on any `Err` — already, generically, handled every possible Actor Host rejection cause before this milestone, including the new `Overflow` cause, without requiring any modification. `reject_message` emits the existing `message.rejected` event and returns the triggering error unchanged, unless that rejection-audit emission itself fails, in which case the audit failure is what the caller observes instead, exactly as ADR-0015 already, generally requires. No retry, no new handling, and no new decision point was introduced into Runtime by this milestone.

## 7. Audit Behaviour

The existing `message.rejected` event (ARCH-002 §18) is the sole audit consequence of an overflow rejection; no new audit event type was introduced. Exactly one `message.rejected` event fires for the overflow-rejected message — proved directly by a new Runtime test using the existing `RecordingAuditEmitter` test double, which counts exactly one `message.rejected` event among all recorded events after filling a mailbox to capacity and submitting one further message, and confirms that event's `message` field identifies the correct, overflow-rejected message. No acceptance event (`message.admitted`) occurs for the rejected message; every prior, successful submission continues to emit its own `message.admitted` as before. Audit ordering is unchanged (rejection-audit occurs before the triggering error is returned, exactly as for every other `submit_message` rejection cause). Audit failure semantics remain governed entirely by ADR-0015, unmodified.

## 8. Files Changed

| File | Change |
|---|---|
| `core/actor-host/src/internal.rs` | Added `pub(crate) const MAILBOX_CAPACITY: usize = 64` with rationale doc comment; added the capacity check inside `ActorHostImpl::enqueue`; updated the module-level and `mailboxes`-field doc comments to describe the new bound and disclose the no-dequeue limitation |
| `core/actor-host/src/lib.rs` | 8 new unit tests added within the existing `#[cfg(test)] mod tests` block (23 → 31); no change to the crate's public trait or delegating implementation |
| `core/actor-host/README.md` | Status section rewritten from a stale "no implementation yet" placeholder to accurately describe the crate's actual implemented behaviour, the new bounded-capacity mechanism, and the disclosed no-dequeue limitation |
| `runtime/src/lib.rs` | 2 new unit tests and one test-scoped `const MAILBOX_CAPACITY: usize = 64` added, entirely within the existing `#[cfg(test)] mod tests` block (52 → 54); **no production code was changed** |

No file outside these four was touched. `common/src/lib.rs`, `core/message-gateway/`, `core/lifecycle-guardian/`, `core/execution-coordinator/`, `core/capability-authority/`, `core/audit-emitter/`, `core/host-adapter/`, every Cargo manifest, and `Cargo.lock` are all confirmed unmodified.

## 9. Public-Interface and Dependency Assessment

The `ActorHost` trait and `enqueue`'s own signature (`fn enqueue(&mut self, instance: &ActorInstanceId, message: Message) -> Result<(), RuntimeError>`) are byte-for-byte unchanged — confirmed directly: every line touched in `core/actor-host/src/lib.rs` falls within the test module. No new public method, type, parameter, or constructor exists anywhere in the diff. No capacity getter or other test-only production API was introduced. No new `Runtime` constructor parameter, no new configuration field, and no new `synapse-common` type or `RuntimeError` variant were introduced — `RuntimeError::Overflow` was already defined and is reused unmodified. No new Cargo dependency was added in any crate; `synapse-actor-host`'s dependency set remains exactly `synapse-common`, confirmed by `cargo tree -p synapse-actor-host`. No new Trusted Core interaction was introduced: Actor Host still has no path to any other Trusted Core component, and no other component gained a path to Actor Host's mailbox state.

## 10. Actor Host Tests

`core/actor-host/src/lib.rs`: **23 → 31 tests (8 new)** — independently re-confirmed against the committed pre-implementation source (`git show HEAD:core/actor-host/src/lib.rs | grep -c '#\[test\]'` → 23) and the current source (31). The original implementation report understated the true pre-existing baseline by one test; the delta of 8 new tests and the resulting total of 31 were always correct.

The 8 new tests:

- `enqueue_succeeds_below_capacity`
- `enqueue_succeeds_exactly_at_capacity`
- `enqueue_rejects_with_overflow_once_mailbox_is_at_capacity`
- `rejected_overflow_message_is_not_stored`
- `fifo_order_of_admitted_messages_is_unchanged_by_a_rejected_overflow_enqueue`
- `mailbox_capacity_and_overflow_are_isolated_per_actor_instance`
- `enqueue_against_an_unknown_instance_still_fails_with_unknown_target_regardless_of_capacity`
- `a_newly_created_instance_starts_with_an_empty_mailbox_and_full_capacity_available`

Together these prove: enqueue succeeds below and exactly at capacity; the next enqueue is rejected with `RuntimeError::Overflow`; the rejected message is never stored (using the existing `mailbox_message_ids` same-crate test seam); already-admitted messages remain in unchanged FIFO order after a rejection; capacity and overflow are isolated per actor instance (filling instance A's mailbox does not affect instance B); unknown-instance rejection remains `UnknownTarget`, never `Overflow`; and a newly created instance starts with an empty mailbox and full capacity available. All pre-existing Actor Host tests continue to pass unmodified in outcome.

## 11. Runtime Tests

`runtime/src/lib.rs`: **52 → 54 unit tests (2 new)** — independently re-confirmed against the committed pre-implementation source (`git show HEAD:runtime/src/lib.rs | grep -c '#\[test\]'` → 52) and the current source (54), correcting the same class of understatement the original implementation report made for Actor Host (it correctly stated this delta as "52 → 54," which independent re-verification confirms was accurate).

The 2 new tests:

- `submit_message_returns_overflow_once_the_mailbox_is_filled_to_capacity_through_the_public_api` — fills a mailbox to capacity through ordinary, repeated `Runtime::submit_message` calls via the existing public API alone (no test-only seam), then confirms the next submission returns exactly `Err(RuntimeError::Overflow)`.
- `submit_message_emits_the_existing_message_rejected_event_on_overflow` — using the existing `RecordingAuditEmitter` test double, confirms exactly one `message.rejected` event is emitted for the overflow-rejected message specifically, with the correct message identity, and that every prior, successful submission's own `message.admitted` emission is unaffected.

`runtime/tests/bootstrap.rs` (integration tests): **16, unchanged** — no modification, confirmed by the diff containing no changes to that file. Existing `submit_message` tests (envelope, send-authority, capability-validity, existence rejections; successful admission) all continue to pass unmodified in outcome.

## 12. Workspace Test Results

**Total workspace tests: 232 passed, 0 failed** — independently computed by summing every `test result:` line from a fresh `cargo test --workspace` run in this task (1 + 31 + 1 + 3 + 1 + 48 + 3 + 15 + 11 + 32 + 14 + 1 + 54 + 0 + 16 + 1 = 232). This corrects the original implementation report's stated total, which undercounted the true figure by 17. Every individual crate's own reported total is independently reconciled below:

| Crate/target | Tests | Result |
|---|---|---|
| `synapse-actor-directory` | 1 | Pass |
| `synapse-actor-host` | 31 (23 pre-existing + 8 new) | Pass |
| `synapse-api` | 1 | Pass |
| `synapse-audit-emitter` | 3 | Pass |
| `synapse-audit-pipeline` | 1 | Pass |
| `synapse-capability-authority` | 48 | Pass |
| `synapse-common` | 3 | Pass |
| `synapse-execution-coordinator` | 15 | Pass |
| `synapse-host-adapter` | 11 | Pass |
| `synapse-lifecycle-guardian` | 32 | Pass |
| `synapse-message-gateway` | 14 | Pass |
| `synapse-persistence` | 1 | Pass |
| `synapse-runtime` unit tests | 54 (52 pre-existing + 2 new) | Pass |
| `synapse-runtime` integration tests (`bootstrap.rs`) | 16 (unchanged) | Pass |
| `synapse-scheduler` | 1 | Pass |
| Doc-tests (all crates) | 0 | Pass (none exist) |
| **Total** | **232** | **All pass, 0 failures** |

## 13. No-Dequeue Limitation

No per-message dequeue or mailbox-drain mechanism exists anywhere in the current workspace — nothing removes an individual message from a mailbox once admitted; the only removal is whole-mailbox deletion, performed by `terminate_instance`. Consequently, once a mailbox reaches its 64-message capacity, it remains full — permanently rejecting further `enqueue` calls with `Overflow` — until its owning actor instance is terminated, which removes the mailbox entirely. This is a pre-existing, honest consequence of the current Minimal Runtime Profile not yet invoking real actor-defined handler logic (ARCH-002 §21; ARCH-003 §5, §10): nothing currently consumes mailbox contents, so nothing currently frees capacity short of termination. `Runtime::execute_message` does not read from, or otherwise consume, any actor's mailbox — Execution Coordinator's own construct/dispatch/complete sequence operates entirely independently of mailbox contents, unaffected by this milestone. EWO-006 did not introduce this limitation and was not scoped to resolve it; this is disclosed, unchanged, and unresolved by this milestone, exactly as EWO-006 itself required.

## 14. Architecture Compliance

- ARCH-002 §13: mailbox capacity is now bounded and finite (mechanism-level MUST, satisfied); overflow produces a defined, non-silent, audited response (reject-with-audit-event, satisfied via the existing `message.rejected` path); FIFO ordering of accepted messages is preserved; no priority reordering or backpressure was added — both remain explicitly deferred policy, per ARCH-002 §13's own text.
- ARCH-002 §22: the mandatory bounded-mailbox conformance requirement, disclosed as an open gap since EWO-001, is now implemented.
- ARCH-003 v0.4.0: no new Trusted Core interaction was introduced; Actor Host still has no path to any other Trusted Core component; Runtime remains the sole composition root, with zero production-code change required to realize this milestone; Message Gateway's own `admit` behaviour, and its documented disclaimer of capacity responsibility, are both unaffected. No claim of real actor execution is made anywhere in this report or the underlying implementation.
- No ADR or architecture revision was required or performed before or during implementation, consistent with EWO-006's own governance assessment.

Per ARCH-003 §20's own standing conformance requirement, ARCH-003 itself should be updated in a separate, subsequent conformance-update task (mirroring the pattern already used for EWO-004 and EWO-005) to reflect that the "bounded mailbox capacity and audited overflow handling" item is no longer deferred.

## 15. Stop-Condition Assessment

Independently re-assessed against the actual diff; none of EWO-006's 11 stop conditions was triggered:

1. No public interface (`ActorHost`, `Runtime`, `MessageGateway`, or any `synapse-common` type) required any change.
2. No new `RuntimeError` variant was required — `Overflow` proved sufficient.
3. No new audit event was required — `message.rejected` proved sufficient.
4. `Runtime::submit_message` required no code change of any kind.
5. Capacity required no per-actor-instance or per-actor-definition differentiation.
6. No configuration mechanism was found architecturally required.
7. Isolating capacity per actor instance required nothing beyond the existing per-instance `HashMap` keying already in place.
8. No Trusted Core component other than Actor Host required modification.
9. No direct dependency between `synapse-actor-host` and any other Trusted Core crate was required.
10. No ADR guarantee or ARCH-002/ARCH-003 architectural boundary required change.
11. The disclosed no-dequeue limitation was not found to require resolution to satisfy this EWO's own Definition of Done.

## 16. Validation Results

All Mandatory Validation gates independently re-run and confirmed passing in this task:

| Gate | Result |
|---|---|
| `cargo fmt --all -- --check` | Clean |
| `cargo clippy --workspace --all-targets --all-features -- -D warnings` | Clean, zero warnings |
| `cargo build --workspace` | Clean |
| `cargo test --workspace` | All 232 tests pass, 0 failures |
| `git diff --check` | Clean, no whitespace errors |
| `cargo tree -p synapse-actor-host` | Unchanged: exactly `synapse-common` |

`git diff --stat`: `core/actor-host/README.md` (+26/−2), `core/actor-host/src/internal.rs` (+42/−7 net across hunks), `core/actor-host/src/lib.rs` (+154), `runtime/src/lib.rs` (+75) — 4 files changed, 290 insertions(+), 7 deletions(-) combined. `git diff --name-only`: exactly the four authorized files.

## 17. Known Limitations

- No per-message dequeue or mailbox-drain mechanism exists (§13) — a mailbox that reaches capacity remains full until its instance is terminated.
- No real actor-defined handler execution exists or is claimed.
- No scheduler work of any kind was introduced.
- No backpressure of any kind was introduced — overflow is reject-only.
- No retry of any kind was introduced.
- No persistence of any kind was introduced.
- Mailbox capacity is not configurable at this milestone — no constructor parameter, no per-instance/per-definition value, no configuration mechanism.
- The capacity value `64` remains implementation policy, not an architectural guarantee, and may be revisited by a future milestone without requiring an ADR, provided capacity's ownership, mechanism, and reject-only policy remain unchanged.

## 18. Repository State

`synapse-runtime` (`git status --short`):

```
 M core/actor-host/README.md
 M core/actor-host/src/internal.rs
 M core/actor-host/src/lib.rs
 M runtime/src/lib.rs
```

Nothing staged. HEAD remains `5ccc7f9083a71adc6ee704b2322a701935765679`; `origin/main` unchanged.

`synapse-docs` (`git status --short`):

```
?? .ai/
?? engineering-reports/ER-006-Bounded-Actor-Mailboxes.md
?? maintenance/
?? work-orders/EWO-003-Message-Gateway.md
?? work-orders/EWO-006-Bounded-Actor-Mailboxes.md
```

Nothing staged. HEAD remains `e90404baa5140ce9004839bc51921c789777e003`. EWO-006 was not modified by this report — confirmed byte-identical (sha256 unchanged) before and after this task. This report (ER-006) is the only newly created file in `synapse-docs`. The three originally-protected untracked files remain present and untouched.

## Final Disposition

Implementation and this report are ready for independent review. This report does not claim final approval, completion publication, commit, push, or architecture-conformance publication — those remain separate, subsequent, explicitly authorized acts, consistent with STD-001 §47's own informational-only status for Engineering Reports.

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-07-13 | Claude (AI-assisted) | Initial report following EWO-006 v0.1.0 implementation, incorporating the corrected test-count figures (Actor Host 23→31, workspace total 232) established by the independent EWO-006 implementation review, which found the original implementation report's baseline and total test-count figures inaccurate. |
