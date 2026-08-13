---
document_id: EMO-002
title: "Bounded Cross-Platform Observation Client I/O and Session Invalidation"
version: 0.1.0
status: Review
author: Denver Jacobs (AI-assisted)
owner: Denver Jacobs
reviewers:
  - Independent Reviewer: TBD
approval_authority: Founder (Denver Jacobs), after Independent Engineering Review
created: 2026-08-13
last_updated: 2026-08-13
classification: Public
implementation_status: Not Authorized
maintenance_traceability:
  ewo_maintained:
    - EWO-029.4 — Foundation-layer Observation Client
    - EWO-029.5 — connection/session/compatibility/wire realization
    - EWO-029.6 — accepted cross-platform implementation evidence
  engineering_report_related: None separately filed in the Documentation repository; accepted EWO-029.6 implementation evidence remains the governing lifecycle record
related_documents:
  architecture:
    - ARCH-016 v0.2.1 — Control Centre Foundation Architecture, Approved
    - ARCH-017 v0.2.2 — Runtime Observation Connectivity Architecture, Approved
  work_orders:
    - EWO-028 v0.1.0 Review candidate 2 — blocked on this prerequisite
    - EWO-029.3 — accepted Observation Service and Windows watchdog precedent
    - EWO-029.4 — accepted Foundation Observation Client architecture
    - EWO-029.5 v1.0.0 — accepted session/wire/compatibility architecture
    - EWO-029.6 v0.2.0 — accepted native conformance evidence
  standards:
    - STD-001 — Documentation Standards
    - STD-004 — Repository Standards (Draft guidance)
    - STD-011 — Dependency Management Standards (Draft guidance)
    - STD-031 v0.2.1 — Engineering Lifecycle Standard, Approved
  governance:
    - GOV-003 — Governance Model
    - GOV-010 — Decision Framework
    - GOV-013 — Engineering Lifecycle
verified_baseline:
  runtime_origin_main: e48d41da7c94281dcb00b57eb40f63aa1db74984
  runtime_tree: d584767ad292f84f6aa8ca995a83ac48bc26304d
  docs_origin_main: 0c768881a338df3143f1f1ad3af32acc76cb46dd
  docs_tree: 8050b3e03290637410581a3d05e9018fc9166d05
---

# EMO-002 — Bounded Cross-Platform Observation Client I/O and Session Invalidation Engineering Maintenance Order

> **Authorization boundary.** The Founder decision dated 2026-08-13 authorizes authoring and filing this narrow prerequisite candidate for review. It does not authorize implementation. No Runtime/SDK source, EWO-029 artifact, protocol, wire shape, operation, transport, authorization rule, or production behavior is modified by this filing.

## 1. Document Control

| Field | Value |
|---|---|
| Document ID | `EMO-002` |
| Title | Bounded Cross-Platform Observation Client I/O and Session Invalidation |
| Version | `0.1.0` |
| Status | **Review** — non-operative candidate for Independent Engineering Review |
| Author | Denver Jacobs (AI-assisted) |
| Owner | Denver Jacobs |
| Independent Reviewer | TBD |
| Approval Authority | Founder, after a passing Independent Engineering Review |
| Runtime baseline | `e48d41da7c94281dcb00b57eb40f63aa1db74984` |
| Documentation baseline | `0c768881a338df3143f1f1ad3af32acc76cb46dd` |
| Implementation status | **NOT AUTHORIZED** |
| Maintains | Accepted `EWO-029.4`, `EWO-029.5`, and `EWO-029.6` Observation-client implementation boundary |
| Related Engineering Report | None separately filed; accepted EWO-029.6 implementation evidence remains the governing lifecycle record |
| Downstream dependency | EWO-028 GUI implementation remains blocked until §19 is fully satisfied |

## 2. Identifier and Numbering

`STD-001` §48 requires narrowly scoped conformance maintenance of an already completed engineering milestone to use an Engineering Maintenance Order rather than rewriting its accepted EWO/implementation history. The accepted SDK client is part of completed EWO-029.4–.6, so a new `EWO-030` would be the wrong family.

EMO and EMR sequences are independent. `EMO-001` is already reserved by repository records and by `EMR-002`'s explicit numbering disclosure. No `EMO-002` file or reservation exists. The next available governed maintenance identifier is therefore **`EMO-002`**, filed at `maintenance/EMO-002-Bounded-Cross-Platform-Observation-Client-IO-and-Session-Invalidation.md`. Existing `EMR-002` does not collide because it belongs to the separate EMR family.

## 3. Objective

Specify the smallest architecture-preserving Runtime/SDK maintenance required to make transport establishment and every synchronous `ObservationClient` request genuinely bounded on Windows, Linux, and macOS and to ensure that timeout, transport, malformed-response, framing, or decode failure can never leave a reusable, potentially desynchronized session.

Implementation, if separately authorized after review and approval, must preserve the accepted seven-operation local Observation architecture and produce genuine native evidence suitable for EWO-028's prerequisite gate.

## 4. Engineering Question

How can the existing synchronous `ObservationClient` guarantee finite transport-connect and request bounds plus irreversible session invalidation after connection-fatal failure on all three native platforms—especially Windows named pipes—without changing Observation operations, wire/protocol semantics, authorization, local-only transport, or Runtime authority?

## 5. Authority and Lifecycle

This maintenance order derives all authority from already accepted architecture and the specific completed EWO-029.4–.6 implementation boundary it maintains:

- `ARCH-017` §§13–20, 27–34: fixed connection/session ordering, seven operations, synchronous request/response, local-only authorization, error categories and cross-platform conformance;
- accepted `EWO-029.3`–`.6`: exact implemented service/client/wire/endpoint architecture and native security evidence;
- the Founder EWO-028 correction authorization: bounded/cancellable Windows requests, silent-peer handling, session invalidation and native evidence are mandatory prerequisites;
- `STD-031`: authoring, independent review, Founder approval, implementation authorization, implementation, independent implementation review and Founder acceptance are separate stages.

Under `STD-001` §48, this EMO may restore implementation conformance and define validation, but may not introduce new architecture or independent engineering scope. If native investigation exposes an architectural deficiency rather than an implementation-level correction, this EMO must stop and route the issue through the existing ADR/architecture lifecycle. This candidate requests Independent Engineering Review only. It performs and authorizes no production change.

## 6. Exact Current Limitation

At Runtime commit `e48d41da7c94281dcb00b57eb40f63aa1db74984`:

1. `sdk/src/foundation/observation.rs` lines 294–309 opens a local stream through `LocalSocketStream::connect` and, when `request_timeout` is present, calls `set_recv_timeout` and `set_send_timeout` only after connection.
2. `LocalSocketStream::connect` uses `ConnectOptions`' default unbounded wait. In locked Windows source, `local_socket::Stream::from_options` calls `DuplexPipeStream::connect_by_path`, whose documented/default wait mode is unbounded; the wrapper does not apply `ConnectOptions`' configurable wait mode. A busy/exhausted pipe can therefore retain transport establishment outside `request_timeout`.
3. SDK lines 300–302 discard both timeout-setter `Result` values, so a client can be returned even when neither request bound was installed.
4. Locked dependency `interprocess` `2.4.3`, `src/os/windows/named_pipe/local_socket/stream.rs` lines 25–26 and 53–55, explicitly returns `io::ErrorKind::Unsupported` because its Windows local-socket named-pipe wrapper does not support those timeout setters.
5. `ObservationClient::exchange` writes a framed request and blocks in `read_line`. A Windows peer that accepts the request and remains silent can retain that call indefinitely.
6. The same `read_line` appends until delimiter/EOF without enforcing a client-side byte ceiling. A faulty peer that continuously supplies non-delimited bytes can grow client memory and evade an inactivity-style timeout while progress continues.
7. Accepted EWO-029.5 defines a configurable **server-side** `max_response_bytes` with a 512-byte minimum, but fixes no universal generation-1 maximum known to the client and the current public client config contains no response-size field. A client-side ceiling is therefore required but its exact authority/value is not presently settled.
8. The client state contains the buffered stream and an `established` flag but no invalid/poisoned state. Write, EOF, read, framing, or decode failure returns an error while the same client object retains the stream.
9. A late response after a consumer-side timeout can become the next read's input if the stream is reused, violating one-request/one-response correlation without any wire request ID.

This is a client-side realizability and lifecycle defect. It does not invalidate the accepted server authorization, wire schema, seven-operation catalogue, or native evidence for what EWO-029.6 actually executed.

## 7. Root Cause

The generic `interprocess::local_socket::Stream` API presents timeout setters on every platform, but its locked Windows implementation cannot realize them. The SDK treats the setters as best-effort even though downstream EWO-028 requires a hard bound. Separately, the client models protocol establishment but not terminal session invalidity, so it lacks the internal state transition required after a partial write, failed read, timeout, EOF, or malformed response.

The root cause is not Tauri, the WebView, GUI scheduling, server dispatch, or a missing eighth operation. A GUI-side timer around a still-blocked SDK connect/exchange thread cannot correct it.

## 8. Required Behavioral Contract

When `ObservationClientConfig.request_timeout` is `Some(duration)`:

- transport establishment must either complete or return within a documented finite bound derived from `duration` plus a small measured scheduling tolerance;
- the complete request exchange—write, delimiter, response wait, bounded response read, and decode—must either finish or return within a documented finite bound derived from `duration` plus a small measured cancellation/scheduling tolerance;
- response accumulation must stop at a finite independently reviewed client ceiling even when bytes continue arriving without a delimiter; overflow is connection-fatal and must not allocate/read indefinitely;
- the client must not be returned from `connect` with a silently ineffective configured bound;
- Windows, Linux, and macOS must expose the same public semantic result even if the private OS mechanism differs;
- a timeout is `ObservationClientError::Timeout`, preserving the existing closed public error taxonomy;
- transport write/read/EOF failure remains `ConnectionFailure` unless an already-accepted more specific mapping applies;
- no new public mutation or generic cancellation authority is introduced.

This maintenance order does not require changing the existing `request_timeout: Option<Duration>` public shape. Any proposed public API change is a review-time exception and triggers `EMO-002-STOP-07`.

## 9. Connection-Fatal Session Semantics

The following are connection-fatal:

- local or wire-reported timeout;
- partial/failed request write, response EOF, read failure, or other transport failure;
- oversized/missing delimiter or malformed response framing;
- response accumulation exceeding the accepted client ceiling;
- response decode/schema failure;
- a valid `MalformedRequest` response, because the accepted typed client should never generate one and continuing would conceal client/protocol divergence;
- compatibility failure during the first operation.

Before returning any connection-fatal error, the SDK must atomically mark the session invalid and begin cancellation/closure. The stream must not be reused. Every subsequent Observation method on the same `ObservationClient` must fail locally without writing bytes. A caller must construct a new client and repeat first-operation compatibility validation before further observation.

`UnsupportedQuery`, `UnavailableInformation`, and a successfully decoded `RuntimeSideFailure` are operation-semantic results and need not invalidate an otherwise synchronized session. `AuthorizationFailure` normally prevents creation of a usable session; if received after transport creation, the client is discarded.

No continuity is inferred between the invalidated and replacement sessions.

## 10. Architecture-Preserving Candidate Mechanism

Direct source inspection found a relevant accepted precedent in `runtime/src/observation_service/windows.rs`: the Windows server uses an independent clone of the named-pipe handle, a bounded watchdog and `CancelIoEx` to interrupt stalled overlapped I/O. The locked `interprocess` implementation ultimately performs overlapped Windows I/O, and Microsoft documents `CancelIoEx` as marking outstanding I/O for the supplied file handle for cancellation.

The leading candidate for Independent Review is therefore:

1. retain `interprocess` and the existing named-pipe endpoint;
2. establish a genuinely bounded connect path on every platform; independently verify whether the dependency's lower-level Windows timed-connect path can be used safely or whether a narrowly reviewed dependency correction is required;
3. add a private Windows client I/O cancellation guard/watchdog, armed before the first potentially blocking write and retained through response decode;
4. cancel outstanding pipe I/O at the deadline through a safely owned/duplicated handle;
5. invalidate and close the client session after cancellation or any other connection-fatal failure;
6. on Unix/macOS, use a bounded connect path, continue using native socket exchange timeouts, propagate timeout-configuration failure instead of ignoring it, and apply the identical invalidation state machine;
7. bound and deterministically terminate every helper resource; no detached permanently blocked thread is acceptable.

This is a **candidate mechanism submitted for review**, not implementation authorization. The review must verify the connect-wait path, whether the dependency's client handle/clone semantics make cancellation safe, whether cancellation covers both write and read, and how a client response ceiling is established without contradicting accepted server configuration. Implementation must not copy the server pattern mechanically or invent a ceiling without those proofs and the decision in §11.

## 11. Alternatives and Escalation Boundary

The following alternatives may be investigated under a later implementation authorization, but none is selected by this filing:

| Alternative | Assessment boundary |
|---|---|
| Private Windows overlapped I/O with a bounded wait and cancellation | Potentially removes a polling watchdog but expands direct Windows API code; requires review before selection |
| Dependency lower-level timed-connect path or narrow connect wrapper | Potentially preserves the dependency while bounding connection; must prove behavior against busy/exhausted pipes and avoid remote names |
| Upgrade or narrowly patch `interprocess` | Permitted only if dependency, compatibility, licence and native behavior are reviewed; must preserve endpoint and wire semantics |
| Replace the IPC library | Material architecture/dependency decision; STOP and route through Founder-directed ADR/architecture review before implementation |
| Detached worker plus caller-side timer | Rejected as insufficient: it does not cancel I/O, bound resource retention, or prevent later stream desynchronization |
| New wire cancellation message or request ID | Out of scope; protocol/operation expansion is prohibited |

**Response-ceiling decision required.** EWO-029.5 fixes a server-side configurable response bound and a minimum, not a universal maximum communicated to or fixed for clients. Before implementation authorization, Independent Review and the Founder must determine whether an exact private SDK safety ceiling is a compatible implementation-level realization, whether `ObservationClientConfig` requires a reviewed public response-bound field, or whether EWO-029.5/architecture requires a separate amendment establishing a protocol maximum. This EMO selects none of those paths. If the decision changes the public API, accepted response compatibility, or EWO-029 semantics, `EMO-002-STOP-07` applies and the ADR/architecture lifecycle must resolve it first.

If native research disproves the leading candidate or requires library replacement, protocol change, unbounded helper threads, new public cancellation primitives, or materially different architecture, implementation must stop and return exact evidence. An architectural deficiency must enter the ADR/architecture lifecycle; this EMO cannot resolve it.

## 12. Public and Internal API Boundary

- Preserve the exact seven `ObservationSource` operations and `ObservationClient::connect`.
- Preserve the eight accepted `ObservationClientError` variants unless a separately reviewed architecture amendment authorizes otherwise.
- Keep cancellation and validity state private to SDK transport/session internals.
- Do not expose raw handles, transports, wire DTOs, cancellation tokens, generic dispatch, or mutable Runtime objects.
- A local “session invalidated” diagnostic may use the existing `ConnectionFailure(String)` category; exact diagnostic text is non-normative.
- `ObservationClient` may remain synchronous and non-`Sync`; this work order does not add async, pooling, multiplexing, retry, or background reconnection.

## 13. Wire and Protocol Preservation

Implementation must preserve byte-for-byte:

- protocol generation and compatibility probe semantics;
- JSON/JSON Lines framing and maximum-size rules;
- first-request versus established-request shapes;
- all request/response DTOs and discriminators;
- the exact seven operation identifiers and inputs;
- response sequence/freshness/partiality meaning;
- the closed wire error catalogue.

Cancellation is a local client transport/session action. No cancellation frame, request ID, heartbeat, ping, eighth operation, or server semantic change is permitted.

## 14. Security Preservation

Implementation must preserve:

- local-only Unix-domain socket and Windows `\\.\pipe\` transport;
- Unix/macOS endpoint ownership, mode and peer-effective-UID checks;
- Windows one-SID DACL, impersonation and SID comparison behavior;
- authorization before version/operation semantic processing;
- fail-closed invalid endpoint/root behavior;
- no remote pipe/server address, TCP, HTTP, WebSocket, or network discovery;
- no Runtime mutation, capability execution, provider access, or generic cancellation authority.

A duplicated Windows cancellation handle must refer only to the already-authorized local connection and must not broaden access. Cancellation failure must fail closed: invalidate/drop the client and report that native boundedness was not established; never continue as though the timeout succeeded.

## 15. Dependency Implications

The preferred result retains locked `interprocess` `2.4.3`. A target-specific SDK dependency on existing workspace `windows-sys` features may be necessary if the client mirrors the accepted `CancelIoEx` boundary; that addition requires exact feature, licence, version and lockfile review.

Any `interprocess` version change, local patch, fork, or replacement requires:

- a written necessity and alternatives assessment;
- exact revision/version pinning and `Cargo.lock` update;
- API/source review of named-pipe creation, overlapped I/O, handle cloning and cancellation;
- licence and supply-chain review;
- regression evidence for endpoint naming, ACL/SID behavior and all native platforms;
- Founder direction and, where architecture is implicated, ADR/architecture review if it crosses `EMO-002-STOP-07`.

No dependency may introduce remote transport, generic RPC, an async runtime, or GUI technology merely to solve this prerequisite.

## 16. Controlled Implementation Stages

No stage begins without a passing Independent Engineering Review, effective Founder Approval where required, and separate implementation authorization for the exact committed candidate.

### Stage 0 — native mechanism proof

- Reconfirm exact Runtime/dependency source and baselines.
- Build a minimal native Windows proof against the exact locked dependency demonstrating a bounded busy/exhausted-pipe connection attempt and whether safely owned handle cancellation bounds both blocked write and blocked read.
- Measure cancellation latency, helper termination and handle/thread counts.
- Stop and return evidence if the candidate mechanism is disproved or requires a material alternative.

### Stage 1 — session invalidation model

- Add private valid/invalid session state with one-way invalidation.
- Centralize connection-fatal classification and ensure invalidation occurs before error return.
- Prove subsequent calls fail locally without writing.

### Stage 2 — platform I/O realization

- Implement the independently reviewed Windows bounded-I/O mechanism.
- Propagate Unix/macOS timeout-configuration failures.
- Apply one platform-independent semantic result and session state machine.

### Stage 3 — deterministic and integration tests

- Add silent peer, stalled reader/writer, transport failure, malformed response, invalid-session and new-session tests.
- Add continuously streaming unterminated-response tests that prove the accepted client byte ceiling and memory/resource bound.
- Confirm operation-semantic nonfatal results remain usable where specified.
- Confirm all seven existing operations and wire fixtures remain unchanged.

### Stage 4 — native conformance and reporting

- Run the full locked Runtime/SDK test suite.
- Run genuine native Windows, Linux and macOS evidence workflows.
- Produce the Engineering Maintenance Report under the then-next available EMR identifier, changed-scope audit, dependency review and implementation evidence for Independent Implementation Review.

## 17. Required Tests

### 17.1 Deterministic SDK tests

1. Configured timeout cannot be silently ignored.
2. Busy/exhausted endpoint connection returns within the documented bound on every native platform.
3. Silent peer produces `Timeout` within the documented bound/tolerance.
4. Peer that does not read bounds the request-write path.
5. A peer continuously sending bytes without a delimiter cannot exceed the accepted client response ceiling or grow memory without bound.
6. Timeout and response-size overflow atomically invalidate the session.
7. Transport write failure, EOF and read failure each invalidate the session.
8. Malformed framing and decode failure each invalidate the session.
9. A valid `MalformedRequest` response invalidates the session.
10. No operation writes any byte after invalidation.
11. A new `connect` creates a distinct usable session and repeats first-operation compatibility validation.
12. `UnsupportedQuery`, `UnavailableInformation`, and decoded `RuntimeSideFailure` remain nonfatal as specified.
13. No eighth operation, cancellation frame, request ID, heartbeat or wire fixture appears.
14. Every helper thread/handle terminates or closes within a documented bound; repeated timeouts/overflows do not grow resources without bound.

### 17.2 Genuine native tests

- **Windows:** a busy/exhausted named-pipe connect returns in bound; separately, genuine peers cover a silent response and continuous non-delimited response beyond the client ceiling; calls return in bound, pending I/O is actually cancelled where applicable, accumulation is bounded, the session is unusable, no later bytes arrive on it, helper/handle counts settle, and a fresh client succeeds.
- **Linux:** an equivalent genuine Unix-domain peer produces the same semantic result and invalidation/no-reuse proof.
- **macOS:** the equivalent native Unix-domain test runs on genuine macOS infrastructure and proves the same semantics.
- Each platform also covers mid-response truncation/malformed response and transport termination.
- Evidence records wall-clock bound, configured timeout, tolerance, platform/toolchain, process identities, exact commit, test name/count and outcome.

Cross-compilation, source inspection, mocks, or one test process alone do not replace required native silent-peer evidence. Deterministic unit tests may use threads, but the native integration proof must use separate client and peer processes.

## 18. Regression and Quality Gates

At minimum:

```text
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo build --workspace --all-targets --all-features --locked
cargo test --workspace --all-targets --all-features --locked
cargo test -p synapse-sdk --locked
git diff --check
```

Required native workflows must run on genuine Windows, Linux, and macOS. Existing EWO-029.6 Observation conformance and Windows cross-principal refusal workflows must remain successful or be rerun where the reviewed implementation touches their dependency surface.

## 19. Acceptance Criteria and EWO-028 Release Gate

This maintenance order is implementation-complete only when:

1. the selected implementation mechanism matches an independently reviewed and Founder-authorized architecture;
2. configured bounded transport establishment, request I/O, and response accumulation are effective on Windows, Linux, and macOS and cannot fail silently;
3. the transport-connect path and complete write/read/decode exchange are time-bounded and byte-bounded, not merely a GUI wait;
4. every connection-fatal failure irreversibly invalidates the session before error return;
5. no later byte is sent or accepted through the invalid session;
6. a new client/session is required and revalidates compatibility;
7. no helper resource remains indefinitely blocked or grows without bound under repeated failures;
8. all seven operations, wire bytes, compatibility, authorization and local-only transport remain unchanged;
9. Windows ACL/SID and Unix/macOS ownership/peer-principal evidence remains successful;
10. genuine native silent-peer and malformed/transport-failure evidence passes on all three platforms;
11. the complete locked workspace quality suite passes;
12. Independent Implementation Review has zero unresolved Critical or Major findings;
13. Founder Implementation Acceptance is granted for the exact implementation commit and evidence.

EWO-028 GUI implementation remains blocked until **all thirteen** criteria are satisfied. Filing, review, approval, or implementation authorization for this maintenance order alone does not release that block.

## 20. Engineering STOPs

- **`EMO-002-STOP-01` — Baseline divergence.** Stop if Runtime `origin/main` or accepted EWO-029 architecture differs from the reviewed baseline in a way affecting this defect.
- **`EMO-002-STOP-02` — Wire/operation expansion.** Stop if a new operation, request ID, cancellation frame, protocol generation, framing or DTO change appears necessary.
- **`EMO-002-STOP-03` — Security/transport expansion.** Stop if authorization, ACL/SID, endpoint ownership, local-only transport or fail-closed behavior must weaken or if remote transport is proposed.
- **`EMO-002-STOP-04` — Runtime authority expansion.** Stop if mutation, provider/capability execution, Runtime lifecycle authority or generic cancellation authority appears necessary.
- **`EMO-002-STOP-05` — False boundedness.** Stop if the solution only times out a caller while leaving I/O/helper work indefinitely blocked, leaks resources, or cannot prove cancellation.
- **`EMO-002-STOP-06` — Session reuse/desynchronization.** Stop if any connection-fatal path can retain/reuse the stream or a later response can satisfy a different request.
- **`EMO-002-STOP-07` — Material mechanism/dependency/API/architecture expansion.** Stop before replacing/forking the IPC library, adding an async runtime, changing public SDK shape/error taxonomy/configuration, selecting a client response ceiling that alters accepted compatibility, or adopting materially different threads/tasks/cancellation architecture. Return exact evidence to the Founder; if architecture is deficient, route through the ADR/architecture lifecycle rather than resolving it in this EMO.
- **`EMO-002-STOP-08` — Cross-platform semantic divergence.** Stop if Windows, Linux and macOS cannot provide the same public timeout/invalidation contract.
- **`EMO-002-STOP-09` — Scope collision.** Stop if GUI/Tauri code, EWO-028 implementation, EWO-029 amendment or unrelated cleanup becomes necessary.
- **`EMO-002-STOP-10` — Native evidence unavailable.** Stop before acceptance if genuine Windows, Linux and macOS silent-peer evidence cannot be produced.
- **`EMO-002-STOP-11` — Authorization absent.** Stop before production modification unless review, Founder disposition and implementation authorization bind the exact candidate.
- **`EMO-002-STOP-12` — Regression.** Stop on unexplained failure in Observation conformance, cross-principal security, wire fixtures, SDK tests or workspace gates.

A STOP records exact evidence and returns the smallest necessary decision to the Founder. It never authorizes unilateral redesign.

## 21. Explicit Exclusions

- GUI/Tauri/Control Centre implementation or repository creation.
- Any EWO-028 product redesign.
- Any amendment to EWO-029.1–.6.
- New Observation operations, DTOs, fields, protocol generation, framing or serialization.
- Remote transport, discovery, multi-Runtime support, subscription/push, pooling, multiplexing or automatic reconnect.
- Runtime mutation, control, repair, lifecycle, capability or provider authority.
- General-purpose SDK cancellation API or async API.
- Unrelated Runtime/SDK refactoring, dependency upgrades or cleanup.
- Release, deployment or public distribution.

## 22. Engineering Maintenance Report Requirements

The later EMR must record:

- exact candidate approval and implementation-authorization evidence;
- Runtime/Docs base and implementation commits, tree/blob identities and SHA-256 fingerprints;
- selected mechanism and rejected alternatives with native evidence;
- every changed file and staged-scope audit;
- dependency/lockfile/licence/security changes;
- timeout contract, tolerance and measured worst-case behavior;
- exact connection-fatal classification and invalidation state machine;
- helper thread/handle lifecycle and repeated-failure resource measurements;
- wire/operation/security preservation proofs;
- commands, independently derived test counts and native run identifiers;
- all STOPs and review findings with dispositions;
- confirmation that no GUI, EWO-029, protocol, remote transport or mutation work occurred.

## 23. Risks and Mitigations

| Risk | Required mitigation |
|---|---|
| `CancelIoEx` assumption is wrong for the client handle | Stage 0 native proof; STOP rather than infer from server similarity |
| Transport connect blocks before a client exists | Use only an independently reviewed bounded connect path and test a busy/exhausted endpoint natively |
| Write blocks before response wait | Arm the reviewed bound before the first blocking write and test a non-reading peer |
| Continuous non-delimited response grows memory | Enforce an independently accepted client byte ceiling and test streaming overflow natively |
| Cancellation races natural completion | Treat documented benign race explicitly; invalidate session regardless after deadline |
| Dropping a handle races pending I/O | Define single ownership/clone lifetime and prove helper completion before teardown claims |
| GUI timeout hides a leaked SDK thread | Reject caller-only deadline as false boundedness |
| Late response desynchronizes next request | One-way invalidation and no-reuse test |
| Dependency change weakens endpoint security | Exact source/dependency review plus native ACL/SID and endpoint regression evidence |
| Platform implementations drift semantically | Shared state/error contract plus native three-platform conformance |

## 24. Author-Side Hostile Self-Review

This AI-assisted self-review is not Independent Engineering Review and does not replace an independent human reviewer where governance requires one.

Verified:

- exact Windows unbounded-connect path, timeout limitation, and ignored setter errors are cited from committed source;
- no implementation mechanism is represented as already authorized or proven;
- the leading candidate and material alternatives are disclosed;
- boundedness covers write, read, cancellation latency and resources;
- session-fatal/no-reuse semantics are objective;
- native Windows/Linux/macOS silent-peer evidence is mandatory;
- seven operations, wire/protocol, authorization and local-only transport are preserved;
- no Runtime mutation, generic cancellation, remote transport, GUI work or EWO-029 expansion is authorized;
- EWO-028 remains blocked through Founder Implementation Acceptance;
- STOPs capture material design, dependency, security, native evidence and scope escalation.

Self-review finding register: 0 Critical, 0 Major, 0 Minor, 4 Observations.

- **`SR-EMO-002-O01` — Windows mechanism remains review-dependent.** The server precedent makes `CancelIoEx` credible but does not itself prove client write/read cancellation. Disposition: Stage 0 native proof and STOP-05/07.
- **`SR-EMO-002-O02` — Optional timeout remains public.** `request_timeout: None` remains possible. Disposition: EWO-028 must configure `Some`; changing the public API is not required by this candidate.
- **`SR-EMO-002-O03` — AI-assisted review independence.** Author-side analysis cannot self-satisfy independent human review. Disposition: disclosed; next stage is Independent Engineering Review.
- **`SR-EMO-002-O04` — client response ceiling is not fixed by accepted architecture.** The current server-side response bound has a minimum but no universal client-known maximum, while `read_line` is unbounded. Disposition: §11 records the exact decision alternatives and `EMO-002-STOP-07` requires ADR/architecture escalation if this is not a compatible implementation-level decision.

**Self-review verdict: PASS FOR FILING TO INDEPENDENT ENGINEERING REVIEW.** This verdict approves and authorizes nothing.

## 25. References

- Runtime `sdk/src/foundation/observation.rs`, `runtime/src/observation_service/windows.rs`, `runtime/src/observation_service.rs`, `sdk/tests/observation.rs`, `Cargo.lock`, and relevant manifests at `e48d41da7c94281dcb00b57eb40f63aa1db74984`.
- Locked `interprocess` 2.4.3 Windows local-socket/named-pipe source, especially unsupported timeout setters and raw-handle access.
- `architecture/ARCH-016-Control-Centre-Foundation-Architecture.md` v0.2.1.
- `architecture/ARCH-017-Runtime-Observation-Connectivity-Architecture.md` v0.2.2.
- `work-orders/EWO-028-Phase-1-Read-Only-Control-Centre-GUI.md` v0.1.0 Review candidate 2.
- `work-orders/EWO-029.3-Observation-Service.md` through `work-orders/EWO-029.6-Cross-Platform-Observation-Conformance.md`.
- Microsoft `CancelIoEx`: <https://learn.microsoft.com/en-us/windows/win32/fileio/cancelioex-func>.
- Microsoft named-pipe operations: <https://learn.microsoft.com/en-us/windows/win32/ipc/named-pipe-operations>.
- Microsoft synchronous/overlapped pipe I/O: <https://learn.microsoft.com/en-us/windows/win32/ipc/synchronous-and-overlapped-input-and-output>.
- Founder EWO-028 v0.1.0 Narrow Correction Authorization, supplied directly in the governing engagement record.

## 26. Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-13 | Denver Jacobs (AI-assisted) | Initial narrow Runtime/SDK prerequisite candidate. Records the Windows unbounded-connect path, unsupported named-pipe timeout setter and session-reuse/desynchronization defect; defines bounded transport-establishment/full-exchange and connection-fatal invalidation semantics; identifies bounded dependency connect plus the existing server `CancelIoEx` watchdog as review-dependent leading candidates; preserves seven operations, wire/protocol, authorization and local-only transport; mandates genuine native three-platform busy-endpoint/silent-peer evidence; and blocks EWO-028 until full implementation acceptance. Review status; implementation not authorized. |

## 27. Approval Status

| Role / Stage | Holder | Status | Date |
|---|---|---|---|
| Founder authoring authorization | Denver Jacobs, Founder | **AUTHORIZED TO PREPARE AND FILE FOR REVIEW ONLY** | 2026-08-13 |
| Author | Denver Jacobs (AI-assisted) | Candidate authored | 2026-08-13 |
| Author self-review | Denver Jacobs (AI-assisted; not independent) | Pass for filing: 0 Critical, 0 Major, 0 Minor, 4 Observations | 2026-08-13 |
| Independent Engineering Review | TBD | **NOT YET PERFORMED** | — |
| Founder Approval | Denver Jacobs | **NOT GRANTED** | — |
| Implementation Authorization | Denver Jacobs | **NOT GRANTED** | — |
| Implementation | — | **NOT AUTHORIZED / NOT STARTED** | — |
| Independent Implementation Review | — | **NOT STARTED** | — |
| Founder Implementation Acceptance | Denver Jacobs | **NOT GRANTED** | — |

## 28. Disposition

**EMO-002 v0.1.0 — FILED AS A REVIEW CANDIDATE ONLY.**

The exact next lifecycle stage is Independent Engineering Review of this committed candidate alongside corrected EWO-028 candidate 2. Runtime/SDK implementation is not authorized. GUI implementation remains blocked. EWO-029 remains accepted and unamended.

Founder decisions required after review: accept, correct, or reject the prerequisite architecture; and, only after an eligible candidate exists, separately decide Founder Approval and implementation authorization.

**STOP — return control to the Founder.**
