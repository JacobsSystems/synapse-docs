---
document_id: EWO-028
title: "Phase 1 Read-Only Control Centre GUI Engineering Work Order"
version: 0.1.0
status: Review
author: Denver Jacobs (AI-assisted reconstruction)
owner: Denver Jacobs
reviewers:
  - Independent Reviewer: TBD
approval_authority: Founder (Denver Jacobs), exercising Class E implementation authority under GOV-003 §3.5 and GOV-010 §4-§5
created: 2026-08-13
last_updated: 2026-08-13
classification: Public
implementation_status: Not Authorized
historical_recoverability: Previous chat-delivered draft was unfiled; its exact bytes, version, and native STOP identifiers are not recoverable
related_documents:
  architecture:
    - ARCH-016 v0.2.1 — SynapseOS Control Centre Foundation Architecture (Phase 1: Read-Only), Approved
    - ARCH-017 v0.2.2 — Runtime Observation Connectivity Architecture, Approved
  design:
    - DES-004 v0.2.1 — Control Centre Design Exploration, Founder-accepted
    - DES-005 v0.2.0 — Runtime Observation Connectivity Design Exploration, Founder-accepted
  work_orders:
    - EWO-027 v0.2.1 — downstream sequencing evidence
    - EWO-029.1 through EWO-029.6 — accepted Runtime Observation foundation
  maintenance:
    - EMO-002 v0.1.0 — bounded cross-platform Observation-client I/O prerequisite candidate; Review; implementation not authorized
  standards:
    - STD-001 — Documentation Standards
    - STD-004 — Repository Standards (Draft; consulted as repository guidance, not represented as Approved authority)
    - STD-011 — Dependency Management Standards (Draft; consulted as dependency guidance, not represented as Approved authority)
    - STD-031 v0.2.1 — Engineering Lifecycle Standard, Approved
  governance:
    - GOV-003 — Governance Model
    - GOV-010 — Decision Framework
    - GOV-013 — Engineering Lifecycle
verified_baseline:
  runtime_origin_main: e48d41da7c94281dcb00b57eb40f63aa1db74984
  docs_origin_main: 0c768881a338df3143f1f1ad3af32acc76cb46dd
---

# EWO-028 — Phase 1 Read-Only Control Centre GUI Engineering Work Order

> **Reconstruction and filing notice.** A previous `EWO-028` draft was delivered in chat and never filed in either repository. Exhaustive inspection of reachable paths and object names at the verified Documentation baseline found no recoverable `EWO-028` artifact. Its exact bytes, version, revision history, and native Engineering STOP identifiers therefore remain unknown and are not recreated or attributed here. This document is the first canonical, repository-tracked `EWO-028` identity. Version `0.1.0` identifies this reconstruction, not the missing draft. Status `Review` means it is submitted for Independent Engineering Review under `STD-001` §12 and `STD-031` §7.1; it is not Approved and authorizes no implementation.

> **Founder resumption boundary.** The Founder decision dated 2026-08-13 resumed `EWO-028` only for completion of its existing work-order lifecycle, authorized this reconstruction and the bounded GUI technology/repository decisions within it, and authorized filing for Independent Review. It explicitly did **not** authorize GUI code, repository population, Runtime or SDK production changes, new Observation operations, protocol generations, transports, authority or security models, Founder Approval, release, or deployment. Those boundaries govern this filing task and remain intact after it.

> **Founder narrow-correction boundary.** The Founder decision dated 2026-08-13 authorized correction of this exact candidate for `IER-028-100-F01` through `F03`, confirmed that the Rust core is trusted and the WebView/frontend begins the hostile boundary, and authorized authoring a separate narrow Runtime/SDK prerequisite work order for bounded cross-platform Observation-client I/O. It authorized documentation changes and normal filing only. It did **not** authorize Control Centre creation, GUI code, Runtime/SDK implementation, an eighth operation, protocol or transport expansion, remote access, mutation authority, or any `EWO-029` change.

## 1. Document Control

| Field | Value |
|---|---|
| Document ID | `EWO-028` |
| Title | Phase 1 Read-Only Control Centre GUI Engineering Work Order |
| Version | `0.1.0` — first canonical reconstructed identity; no claim about the missing chat draft's version |
| Status | **Review** — filed for Independent Engineering Review; non-operative |
| Author | Denver Jacobs (AI-assisted reconstruction) |
| Owner | Denver Jacobs |
| Independent Reviewer | TBD; no independent review is performed or claimed by this filing |
| Approval Authority | Founder, under `GOV-003` §3.5 and `GOV-010` Class E, after Independent Review |
| Runtime baseline | `e48d41da7c94281dcb00b57eb40f63aa1db74984` |
| Documentation correction baseline | `0c768881a338df3143f1f1ad3af32acc76cb46dd` |
| Implementation status | **NOT AUTHORIZED; blocked on the complete EMO-002 prerequisite lifecycle and native evidence in §6.3** |
| Intended implementation repository | Proposed dedicated `JacobsSystems/synapse-control-centre`; not created by this task |

## 2. Objective

Implement, only after a separate effective Founder Approval and publication of this EWO **and** completion of the prerequisite gate in §6.3, the smallest useful cross-platform SynapseOS Control Centre: a standalone Windows/Linux/macOS desktop process that observes one independently running local SynapseOS Runtime exclusively through a narrow Control Centre Observation adapter over the typed Foundation-layer `synapse_sdk::foundation::observation::ObservationClient`.

The resulting Phase 1 product must make Runtime state understandable without acquiring, fabricating, or bypassing Runtime authority. It must remain structurally read-only, truthfully expose incomplete or unsupported projections, and leave the Runtime fully correct and operational if the Control Centre is absent, disconnected, crashed, or uninstalled.

## 3. Engineering Question

How can the Approved `ARCH-016` read-only Control Centre be implemented over the accepted `ARCH-017` / `EWO-029` Observation foundation as a minimal, secure, maintainable desktop application without introducing a second Runtime, a hidden mutation path, a new protocol or operation, or an unsupported claim of Runtime truth?

## 4. Engineering Authority

This EWO derives its implementation scope exclusively from Approved architecture and the Founder resumption decision. It does not amend any cited source.

| Authority | Binding use in this EWO |
|---|---|
| `ARCH-016` §8-§14 | Standalone SDK-mediated projection, one local Runtime, state ownership, connection and staleness model |
| `ARCH-016` §16-§21 | Durable-only actor browsing; actor detail; capability, effect, bounded supervision and diagnostics projection; cross-cutting partial-data rule |
| `ARCH-016` §25-§31 | Existing error meaning; compatibility; security; restart/reconnection; cross-platform; presentation independence; structural read-only rule |
| `ARCH-016` §36-§41 | Invariants, Phase 1 capability matrix, risks and architecture acceptance tests |
| `ARCH-017` §13-§20 | Authorized connection/session order, local-principal boundary, seven-operation catalogue, freshness metadata and restart continuity limits |
| `ARCH-017` §21-§29 | Exact projection meanings, bounded work and client/service failure isolation |
| `ARCH-017` §31-§34 | Foundation/Experimental SDK classification, explicit Runtime opt-in, cross-platform constraints and security posture |
| `EWO-029.1`-`.6` | Accepted implementation realization of the Runtime projection, transport, service, SDK client, endpoint/wire contract and native conformance |
| `STD-001` §46 | Required EWO objective, scope, constraints, Definition of Done, validation and reporting |
| `GOV-013` §6.12 | Bounded design decisions left open by architecture, testable completion criteria and stop/escalation conditions |
| `STD-031` | Independent Review, Founder Approval, publication, implementation and later acceptance remain distinct stages |

`DES-004`, `DES-005`, Draft `STD-004`, and Draft `STD-011` are consulted as preserved design and engineering guidance. They are not represented here as Approved authority and do not override `ARCH-016`, `ARCH-017`, or the Approved lifecycle standard.

## 5. Verified Starting State

The initial reconstruction fetch on 2026-08-13 established:

- Runtime `origin/main` exactly `e48d41da7c94281dcb00b57eb40f63aa1db74984`, tree `d584767ad292f84f6aa8ca995a83ac48bc26304d`.
- Documentation `origin/main` exactly `78d09a487a9d0906ba160879bb50565b512dbc12`, tree `a7a2f650ea08cc936dd73b136ab44ce70004bdfd`.
- Both primary worktrees contained pre-existing unrelated changes and were left untouched. All evidence review and this filing were performed in clean, detached, isolated worktrees.
- No Control Centre repository existed under the verified SynapseOS repository location.
- No `EWO-028` path existed in the Documentation repository's reachable history or reachable object/path list. Only later architecture/work-order references to the missing draft and `IER-028-F01`/`F02` were recoverable.

The correction-authority fresh fetch on 2026-08-13 independently established Runtime `origin/main` at the same exact commit/tree and Documentation `origin/main` at `0c768881a338df3143f1f1ad3af32acc76cb46dd`, tree `8050b3e03290637410581a3d05e9018fc9166d05`. The reviewed EWO-028 candidate at that Documentation commit had blob `e77a2bc2ca57078c948bbee23f26e1e87b213f9f` and SHA-256 `601f89d762a3858f990e2b44c8817400ae11f35faf7e685919c8e4922f33655b`. Both dirty primary worktrees were fingerprinted and preserved; correction occurred only in clean isolated worktrees.

The Runtime baseline contains a public typed `ObservationClient`, `ObservationClientConfig`, `ObservationClientError`, `ObservationSource`, the seven query methods, and their typed views. `RuntimeStatusView` currently contains `meta` and `state`; it does not contain Runtime version, build, uptime, identity/generation, or actor-count fields. `DiagnosticsView` exists, but the current Runtime truthfully returns `Unsupported` because no live queryable diagnostic state exists.

## 6. Prerequisite and Finding Treatment

### 6.1 Historical `IER-028-F01` — RESOLVED

The original blocking finding established that no genuine standalone process could observe an independently running Runtime. At Runtime commit `e48d41da7c94281dcb00b57eb40f63aa1db74984`, the following accepted path now exists:

```text
Control Centre process
    -> synapse-sdk ObservationClient
    -> local Observation endpoint and wire contract
    -> Runtime-hosted Observation Service
    -> Runtime-internal read-only projections
```

This is genuine cross-process-capable observation. It is not an in-process mock, direct Runtime embedding, CLI wrapper, filesystem scrape, or GUI-owned source of truth. The independent exact-artifact review classified historical `IER-028-F01` **RESOLVED**. The implementation acceptance gate still requires a genuine Control Centre process and a genuine independently running Runtime service process (§22.4); same-process test threads are not sufficient for that downstream proof.

### 6.2 Historical `IER-028-F02` — RESOLVED for the accepted Observation foundation

The accepted foundation enforces read-only behavior structurally:

- exactly seven named query operations and no generic dispatch;
- no mutation operation in the catalogue or public `ObservationSource` trait;
- effect payload content excluded by architecture and wire/view types;
- service absent unless explicitly enabled at Runtime bootstrap;
- authorization before any request parse or projection execution;
- Linux/macOS peer effective-UID validation plus protected endpoint ownership/mode rules;
- Windows one-SID DACL and impersonated-client SID validation;
- fail-closed unauthorized, malformed, incompatible and unavailable behavior;
- no remote transport and no network endpoint;
- native Linux/macOS conformance from Run #7 and preserved Windows security evidence from Run #4, as accepted by the Founder for `EWO-029.6`.

The independent exact-artifact review classified historical `IER-028-F02` **RESOLVED for the accepted Observation foundation**. That resolution does not mean that declaring only `synapse-sdk` proves the entire GUI structurally read-only: `synapse-sdk::foundation` also publicly re-exports Runtime-capable symbols. The Control Centre's additional least-authority structure is specified separately in §§11–12.

### 6.3 `IER-028-100-F01` — ADDRESSED by an explicit prerequisite, pending re-review

Runtime commit `e48d41da7c94281dcb00b57eb40f63aa1db74984` cannot establish bounded Windows Observation-client I/O: the SDK uses the dependency's default unbounded transport-connect path, and after connection it ignores the results of `set_recv_timeout` and `set_send_timeout`, while locked `interprocess` `2.4.3` reports Windows named-pipe I/O timeouts as unsupported. A busy endpoint or silent peer can therefore retain the synchronous SDK call indefinitely. `EMO-002 — Bounded Cross-Platform Observation Client I/O and Session Invalidation` is filed beside this correction as the separate narrow Runtime/SDK prerequisite candidate.

**GUI implementation remains blocked** until the exact EMO-002 candidate has: (1) been filed; (2) passed Independent Engineering Review; (3) received Founder Approval where required; (4) received separate implementation authorization; (5) been implemented; (6) passed Independent Implementation Review; (7) received Founder Implementation Acceptance; and (8) passed required native Windows, Linux, and macOS evidence, including silent/hung-peer behavior. EWO-028 does not select or authorize EMO-002's implementation mechanism.

For the accepted SDK produced by that lifecycle, transport establishment, complete request duration, and client response accumulation must be bounded. Timeout, transport failure, response-size overflow, malformed response, framing failure, and decode failure are **connection-fatal**. A connection-fatal failure must atomically invalidate and discard the `ObservationClient`/session; no later operation may use that stream. Prior GUI projections remain only as visibly stale. Further observation requires creation and validation of a new client/session. The GUI must never issue another operation on a potentially desynchronized stream.

### 6.4 `IER-028-100-F02` — ADDRESSED, pending re-review

The Founder has fixed the threat model: the Rust core is trusted; the WebView/frontend is the hostile/untrusted boundary. Trusted does not mean unrestricted. The Rust core remains deliberately least-authority through the narrow adapter, exact import/API policy, Tauri command manifest, capability configuration, source review, static checks, and tests in §§11–12 and §22.3. These controls provide enforceable evidence and review boundaries; this EWO does not claim mathematical impossibility or OS sandboxing of trusted Rust code.

### 6.5 `IER-028-100-F03` — ADDRESSED, pending re-review

`ObservationClient::connect` establishes transport only. `GetRuntimeStatus` is the bootstrap Observation operation used to validate usable authorization/compatibility and obtain the first truthful projection. The corrected state model is `Disconnected → Connecting → Validating → Connected` (§8.2). No eighth handshake operation exists. Any bootstrap authorization, compatibility, transport, timeout, malformed-response, framing, or decode failure discards the client/session. Unix EOF-style refusal remains `Unavailable`/connection failure unless the SDK truthfully supplies `AuthorizationFailure`; the GUI must not infer `Unauthorized` from ambiguous EOF.

### 6.6 Independent-review observations

- `IER-028-100-O01` is incorporated: Tauri 2 remains selected subject to the exact authority boundary below.
- `IER-028-100-O02` is incorporated: Stage 0 now requires clean-checkout, exact-revision SDK distribution proof and least-privilege private-repository CI access.
- `IER-028-100-O03` is incorporated: final integration evidence must use genuinely separate processes.
- `IER-028-100-O04` is preserved: this AI-assisted author correction and self-review do not replace a governance-required independent human review.

## 7. Scope

### 7.1 In scope after Founder Approval

- A standalone desktop application for Windows, Linux and macOS.
- One user-initiated connection to the well-known local Observation endpoint of one independently running Runtime.
- Connection-state presentation and explicit connect, refresh, disconnect and reconnect behavior.
- Runtime overview using only fields actually returned by `GetRuntimeStatus`.
- Durable actor browsing using only `ListDurableActors`, permanently labelled as partial actor discovery.
- Selected durable-actor detail using `GetActorDetail`.
- Per-selected-actor capabilities, effects, bounded supervision relationship and diagnostics projections.
- Truthful `AVAILABLE`, `PARTIAL`, `UNAVAILABLE`, `UNSUPPORTED`, stale and disconnected states.
- Minimal cross-platform desktop packaging and native validation evidence.
- Tests, documentation and an Engineering Report required by this EWO.

### 7.2 Explicitly out of scope

- Any Runtime, SDK, endpoint, wire, protocol, transport, authority or security-model change.
- An eighth Observation operation, generic dispatch, push/subscription, event stream, remote transport or compatibility generation.
- Any Runtime mutation or control: start, stop, terminate, restart, recover, delete, checkpoint, grant, revoke, cancel, retry or re-execute.
- Direct Runtime embedding or an in-process Runtime convenience mode.
- CLI invocation, process-provider invocation, filesystem-provider invocation, or direct filesystem inspection/mutation as an observation fallback.
- Global or transient actor enumeration, global effects/capabilities discovery, search, or a complete supervision graph.
- Persistent audit history, activity feed, telemetry expansion or invented diagnostics.
- Runtime version/build/uptime/identity/generation/session-continuity/count displays absent from the accepted SDK view.
- Authoritative Application objects, application manifests or application start/stop.
- Remote Runtime, multi-Runtime, fleet, cloud, account, billing, marketplace or Control Centre administration.
- Persistent GUI preferences, local databases, local telemetry, remote content, network APIs and automatic updating.
- Publication to an app store or public download, deployment to users, or release-channel operation.

## 8. Phase 1 Product Contract

### 8.1 Desktop application shell

“Application shell” here means only the desktop window and navigation frame; it does not mean a Runtime `Application` object.

The shell contains:

- one primary window;
- a persistent connection/status banner;
- primary navigation for **Runtime Overview** and **Durable Actors**;
- an actor-detail route opened from the durable list;
- explicit Connect/Disconnect, Refresh and Reconnect controls appropriate to current state;
- a consistent state treatment for loading, empty, error, partial, unsupported and stale content.

No Application grouping, global search, command palette, multi-window management, tray agent, background daemon or hidden service is part of Phase 1.

### 8.2 Connection state

The UI distinguishes at least:

| State | Required presentation |
|---|---|
| `Disconnected` | No current Runtime connection; any prior projections visibly stale and not presented as current |
| `Connecting` | One EMO-002-bounded transport-establishment attempt is in progress; duplicate attempts are disabled |
| `Validating` | Transport exists; `GetRuntimeStatus` is executing as the first, bootstrap Observation operation to establish usable authorization/compatibility and initial Runtime status |
| `Connected` | Bootstrap `GetRuntimeStatus` completed successfully; later queries may run |
| `Unauthorized` | Access refused; disclose the category without Runtime detail and without fallback/bypass |
| `Incompatible` | Bootstrap compatibility validation failed before server-side semantic dispatch; the failed session has been discarded |
| `Unavailable` | Endpoint absent, disabled or otherwise unreachable; no claim that Runtime itself is absent |
| `Reconnecting` | A user-requested reconnect attempt is in progress; old projections remain stale |

Phase 1 starts disconnected and performs no background endpoint scan. The user may connect to the SDK's well-known default local endpoint. Custom endpoint discovery and arbitrary endpoint entry are not required. A connection failure triggers no hidden Runtime bootstrap and no automatic transport fallback. `ObservationClient::connect` must never be represented as complete Observation validation: the client reaches `Connected` only after `GetRuntimeStatus` succeeds. An authorization, compatibility, transport, timeout, malformed-response, framing, or decode failure during validation discards the session and enters the corresponding honest disconnected/error state. No additional handshake operation may be invented.

### 8.3 Runtime overview

The overview displays only:

- current Control Centre connection state;
- the returned `RuntimeState` when `GetRuntimeStatus` succeeds;
- the projection's truth classification and observation sequence;
- a concise statement that the Runtime remains the source of truth.

Runtime version, build, uptime, stable identity, restart generation, active-actor count and failure count are explicitly labelled **UNAVAILABLE IN THE CURRENT OBSERVATION CONTRACT** or omitted behind an equally explicit explanation. A successful reconnect does not establish continuity with the previously observed Runtime.

### 8.4 Durable actor list

The list is titled **Durable Actors**, never **All Actors**. It displays returned `ActorId` values and the view's observation classification. It must always disclose:

- membership means durable-state possession at observation time;
- transient/non-durable actors are not enumerated;
- presence does not mean the actor is currently running;
- termination alone does not remove durable state;
- returned ordering carries no Runtime meaning.

The UI may apply a stable local lexical sort strictly for presentation, labelled or documented as UI-local. A `Complete` empty response means no durably persisted actors were returned by that complete query; it must never be worded as “no actors exist.” A partial empty response is not evidence that no durable actors exist.

### 8.5 Actor detail

For an actor selected from the durable list, the overview panel displays the returned actor identity and lifecycle state. `lifecycle_state: None` has the accepted specific meaning “no live instance currently exists for this actor”; it must not be relabelled unknown or unavailable.

Each subordinate panel queries and classifies independently. No cross-panel atomic snapshot is claimed.

### 8.6 Capabilities

The capability panel displays only returned descriptive fields: capability id, target, operations and optional retry maximum. It cannot grant, revoke, construct, execute or export a `Capability`. Empty-state language is conditioned on the query's truth classification; absence from a partial/unavailable view never means no authority exists.

### 8.7 Effects

The effect panel uses `ListEffects(EffectSelector::ByActor)` for the selected actor and displays only returned metadata: optional effect id, attempt id, optional status and optional provider. Effect arguments, result bodies, error-detail bodies and other payload content are absent by contract. Cancel, retry and re-execute controls do not exist.

`EffectSelector::ByEffect` remains part of the SDK's accepted operation shape but needs no Phase 1 UI entry point; omitting such an entry point does not change the operation catalogue.

### 8.8 Supervision

The supervision panel displays only `is_registered` and optional `parent` for the selected actor. It explicitly states that children, complete topology, restart count, backoff timing and failure history are unavailable. It does not synthesize a graph or infer missing relationships.

### 8.9 Diagnostics

The diagnostics panel invokes `GetDiagnostics` for the selected actor or Runtime-wide view as implemented. The currently accepted result is `UNSUPPORTED`. The panel must render that result explicitly; it must not show a misleading empty “no problems” state, construct a GUI activity log, consume `AuditEmitter::drain`, or reinterpret ordinary errors as persistent diagnostic history.

### 8.10 Loading, empty and error states

Every independently refreshed surface has a deliberate state:

| Surface state | Meaning and UI rule |
|---|---|
| Loading | A single bounded query is in flight; retain no implication of freshness |
| Available + data | Render returned fields and observation sequence |
| Available + empty | Render a domain-specific empty explanation limited to what that complete query proves |
| Partial | Render available fields plus a persistent partial-data warning |
| Unavailable | Render why the information cannot currently be obtained; stale prior data may remain only if prominently marked |
| Unsupported | Render that the operation/data is not implemented by the current Runtime contract; never render an empty-success state |
| Error | Map the existing SDK error category without inventing a competing semantic taxonomy |
| Stale | Preserve last successful data only with a dominant stale marker and no current-truth claim |

## 9. Truth and Freshness Model

The UI vocabulary deliberately maps the SDK model without changing its meaning:

| UI label | SDK/source meaning |
|---|---|
| **AVAILABLE** | `ObservationStatus::Complete`; the specific query completed, within that query's documented completeness boundary |
| **PARTIAL** | `ObservationStatus::Partial`; some projected dimensions or entries may be absent |
| **UNAVAILABLE** | `ObservationStatus::Unavailable` or the corresponding connection/query error; inability to observe is not absence of Runtime fact |
| **UNSUPPORTED** | `ObservationStatus::Unsupported` or `UnsupportedQuery`; the current implementation does not provide the requested projection |

`AVAILABLE` never means globally complete Runtime truth. It means complete only for the exact operation and projection contract invoked.

Staleness is orthogonal to all four labels. On disconnect, connection loss, timeout, response-size overflow, malformed/decode failure, failed refresh, Runtime restart, or reconnect, every previously held projection remains stale until that exact operation succeeds again. Timeout, transport failure, response-size overflow, malformed response, framing failure, and decode failure invalidate and discard the complete client/session; an individual panel error is not permission to reuse the stream. Reconnection creates a new client/session but alone refreshes nothing. Each of the seven operations carries an independent sequence; sequences must not be compared across operations to imply atomicity or a single global snapshot. If a UI-local receipt time is displayed, it must be labelled client-local receipt time, never Runtime observation time.

## 10. Observation API Usage

The Control Centre's Runtime-interaction adapter may call exactly these public SDK methods:

1. `ObservationClient::connect(ObservationClientConfig)`
2. `ObservationSource::get_runtime_status()`
3. `ObservationSource::list_durable_actors()`
4. `ObservationSource::get_actor_detail(actor_id)`
5. `ObservationSource::list_capabilities(actor_id)`
6. `ObservationSource::list_effects(EffectSelector::ByActor(actor_id))`
7. `ObservationSource::get_supervision_relationship(actor_id)`
8. `ObservationSource::get_diagnostics(actor_id)`

The numbered list contains one transport connection constructor and the exact seven `ARCH-017` query operations. `GetRuntimeStatus` is both a normal accepted query and the bootstrap operation for each newly connected client; this does not create an eighth operation. Disconnect and every connection-fatal failure are realized by discarding the client/session in the GUI process. Refresh and reconnect are GUI orchestration over those existing calls, not new Runtime operations.

The adapter must construct `ObservationClientConfig` with `request_timeout: Some(nonzero_bound)`, where the exact finite bound and permitted cancellation/scheduling tolerance are committed, tested, and reported against the Founder-accepted EMO-002 semantics. `None`, zero, or a GUI-only timer around an unbounded SDK call is forbidden in production Control Centre configuration.

No raw wire type, endpoint transport type, Runtime-internal projection, provider API, Runtime API or CLI command may appear above the dedicated Rust Observation adapter boundary. No generic “method name + JSON value” adapter is permitted.

## 11. Trusted-Core and Observation Adapter Boundary

The Founder-fixed Phase 1 threat model is explicit:

- the reviewed Rust core is trusted but deliberately least-authority;
- the WebView/frontend is the hostile/untrusted boundary;
- Phase 1 does not require the Rust core to be sandboxed from the host OS;
- dependency structure, source policy, static checks, tests, and independent source review constrain accidental authority; they do not prove mathematical impossibility.

The implementation boundary is:

```text
WebView/frontend (hostile boundary)
    -> exact typed Tauri command allow-list
    -> Control Centre application/core
    -> narrow Observation adapter crate/module
    -> synapse-sdk foundation::observation API
    -> local Observation Service
```

The Observation adapter must be a separate Rust workspace crate and is the only production crate that may directly depend on or import `synapse-sdk`. The Tauri application crate and every other Control Centre production component depend only on the adapter's inert owned view models and narrow connection/query results. The adapter exposes connection lifecycle and the exact seven Observation projections only. It must not expose or return SDK objects, `Runtime`, Runtime handles, bootstrap/construction, mutation APIs, capability execution, providers, endpoint/wire primitives, raw transport access, or generic request/dispatch values.

The adapter's manifest pins `synapse-sdk` to the exact full Founder-accepted EMO-002 Runtime revision; a branch, floating tag, local path, copied source, or developer-local Runtime checkout is forbidden. Production Control Centre source must prohibit, unless separately reviewed and authorized:

- wildcard `synapse-sdk` imports and SDK prelude imports;
- `synapse_sdk::foundation::Runtime`, Runtime construction/bootstrap, and Runtime handles;
- direct `synapse-runtime`, provider, `synapse-observation-wire`, or `synapse-observation-endpoint` dependencies/imports;
- unsafe Rust, `extern`/FFI, sidecars, shell/process execution;
- unrestricted filesystem access or unrestricted network/HTTP access;
- raw alternate IPC, generic command dispatch, generic `invoke`, or arbitrary SDK objects.

Repository checks must parse manifests and production source against explicit allowlists/denylists and must be supplemented by independent manual source review. Any future mutating feature requires new architecture and a separate EWO; it cannot be added as an “extra command.”

## 12. Security Boundary

The Control Centre inherits, does not replace, the accepted Observation security boundary:

- local machine only;
- same local OS principal as the Runtime owner unless the Runtime is explicitly configured otherwise;
- authorization before request parsing and projection execution;
- Unix/macOS endpoint-owner/effective-UID and mode protection;
- Windows DACL/SID and impersonation checks;
- fail closed without endpoint repair or alternate transport;
- effect payload minimization;
- bounded request/response/session behavior.

The GUI adds these controls:

- exactly one main window/WebView and an explicitly listed production capability set; capability auto-discovery/default exposure is not accepted as the evidence boundary;
- only the bundled local application origin may access the explicitly allow-listed commands;
- a restrictive production Content Security Policy with `default-src 'self'`, no remote `connect-src`, no CDN, and no unsafe remote navigation;
- no secrets in frontend state and no credential store—the OS principal is the credential boundary;
- no raw Runtime error body exposed where the SDK intentionally returns only an authorization category;
- no retry storm: one connection attempt and one refresh batch at a time, with a bounded request timeout;
- developer tools disabled in release builds;
- dependency inventory, lockfiles, licence review and vulnerability review before acceptance.

The exact Phase 1 application command allow-list is:

1. `cc_connect`
2. `cc_disconnect`
3. `cc_reconnect`
4. `cc_refresh_all`
5. `cc_refresh_runtime_overview`
6. `cc_refresh_durable_actors`
7. `cc_select_actor`
8. `cc_refresh_actor_detail`
9. `cc_refresh_actor_capabilities`
10. `cc_refresh_actor_effects`
11. `cc_refresh_actor_supervision`
12. `cc_refresh_actor_diagnostics`

`cc_connect` accepts no endpoint supplied by the frontend and uses the approved default local endpoint. `cc_select_actor` accepts one closed `ActorIdInput` schema and validates it against the current durable-actor projection before updating backend-owned selection. Actor-panel refresh commands accept no actor identifier from the frontend and operate only on that validated backend-owned selection. Every command returns an inert typed response/state delta; Phase 1 defines no application backend-to-frontend event channel.

No generic dispatcher, arbitrary frontend-supplied command name, raw IPC request, generic event channel, or alternate bridge may bypass this list. No `invoke` wrapper may accept a variable command name. No command may accept raw JSON/value maps, paths, URLs, shell strings, wire values, or SDK objects. No shell, filesystem, HTTP, SQL, process, global-shortcut, updater, or remote-content plugin; sidecar; remote asset; arbitrary navigation; or unrestricted host API is permitted.

Implementation acceptance artifacts must include the exact Tauri capability configuration, application command manifest/list, IPC schemas, CSP, plugin list, dependency manifests, and evidence that one window/WebView/capability set is present. Independent Implementation Review must compare those artifacts byte-for-byte and semantically against this EWO. Tauri capabilities constrain the hostile WebView's access to trusted core commands; they do not make Rust code safe or sandbox the core from the OS.

## 13. GUI Technology Evaluation

### 13.1 Decision method

The evaluation is intentionally ordinal rather than a false-precision benchmark. Each option was checked against Windows/Linux/macOS support, Rust/SDK integration, performance/resource posture, security boundary, packaging/update consequences, maintainability, UI capability, accessibility, long-term risk and architectural complexity.

| Candidate | Cross-platform / Rust fit | Resource and security posture | UI, accessibility and packaging | Decision |
|---|---|---|---|---|
| **Tauri 2** | First-class desktop targets; Rust core can depend directly on `synapse-sdk`; web frontend remains presentation-only | Uses system WebView rather than bundling a browser; capability/permission model and CSP support a narrow bridge; different WebViews require native testing | Mature HTML/CSS UI capability and semantic accessibility path; official platform installer/signing tooling | **SELECTED** |
| Slint | Direct Rust integration and native compilation; all three desktop targets | Lightweight, narrow native surface | Strong declarative UI; desktop accessibility/packaging maturity and licensing model add risk relative to this product's needs | Rejected for Phase 1 |
| `eframe`/`egui` | Direct Rust integration and all three desktop targets | Efficient for immediate-mode tooling | Project documentation describes APIs as still in flux and native accessibility support as Windows/macOS-focused; weaker fit for a polished, accessible Linux desktop inspector | Rejected for Phase 1 |
| Electron | All three desktop targets; Rust SDK requires a native bridge/sidecar | Bundles Chromium and Node.js; larger runtime/update and bridge attack surface | Excellent web UI/accessibility ecosystem and mature packaging | Rejected: unnecessary runtime and bridge complexity |
| Flutter | All three desktop targets; Rust SDK requires FFI/plugin or sidecar boundary | Dedicated engine and Dart toolchain | Strong UI capability; good desktop support and packaging, but adds a second language/runtime and native binding layer | Rejected: unnecessary integration complexity |
| Qt / Avalonia / three native UIs | Broad desktop capability | Potentially strong native behavior | C++/.NET interop, licensing/toolchain burden, or three divergent implementations; multiplies ownership and test surfaces | Rejected: disproportionate complexity |
| Browser-only web app | Broad rendering reach | Browser sandbox is strong | Cannot directly use the local Rust SDK or satisfy the standalone desktop/local-IPC contract without adding a separate service | Rejected: fails the direct SDK boundary |

### 13.2 Selected Phase 1 technology

**Tauri 2 with a bundled, framework-free semantic HTML/CSS/TypeScript frontend.** React, Vue, Svelte and other component frameworks are not selected. Phase 1 is small enough that adding one would create a second application framework and dependency lifecycle without a demonstrated need.

The Rust core owns the `ObservationClient`, connection/session state, query scheduling and projection-to-view-model conversion. The WebView owns presentation and ephemeral navigation state only. Typed, narrow Tauri commands pass inert owned view models across the bridge; the frontend never receives SDK handles, endpoint handles, capabilities, transport objects or generic wire values.

### 13.3 Selection rationale

- **Windows/Linux/macOS:** Tauri's WRY layer supports the three desktop targets. Current Tauri documentation identifies WebView2 on Windows, WKWebView on macOS and WebKitGTK on Linux.
- **Rust and SDK:** the application core is ordinary Rust, allowing a direct, typed Cargo dependency on `synapse-sdk` without FFI, Node native modules, sidecars or a second local service.
- **Performance and memory:** system WebViews avoid shipping a browser engine with each app and provide an appropriate resource posture for a mostly textual inspector. No numeric memory claim is made without measurement.
- **Security:** Tauri capabilities, command permissions and CSP enable a small, inspectable WebView-to-Rust surface. Phase 1 further removes plugins, remote content and generic commands.
- **Packaging:** Tauri provides platform bundle/installer tooling and signing integration for Windows, Linux and macOS.
- **Maintainability:** one Rust core plus standards-based presentation keeps platform semantics out of the GUI framework and avoids a bespoke UI renderer.
- **UI capability:** semantic HTML/CSS is sufficient for lists, status banners, details and responsive panels without adding a component framework.
- **Accessibility:** semantic elements, keyboard behavior and browser accessibility trees provide the most mature common route among the evaluated Rust-compatible options. This is a test obligation, not an automatic compliance claim.
- **Long-term risk:** Tauri and each system WebView remain material dependencies. Exact versions are locked, updates reviewed, and the presentation/domain boundary makes a future replacement possible without redefining Runtime authority.

### 13.4 Known Tauri tradeoffs

- WebView2, WKWebView and WebKitGTK are not identical; layout, focus and accessibility behavior must be natively tested.
- Linux distribution support depends on a compatible WebKitGTK baseline and packaging host.
- The WebView is a second security surface; local-only assets, strict CSP and the minimal command allow-list are mandatory.
- Tauri's updater is deliberately not enabled. Update signing, hosting, rollback and release authority require a later, separate decision.
- Exact Tauri, Rust, frontend toolchain and transitive dependency versions remain an implementation-time dependency record, locked after the approved baseline check; this EWO selects the technology family, not a floating version.

## 14. GUI Repository Decision

### 14.1 Selected structure

Use a new dedicated repository: **`JacobsSystems/synapse-control-centre`**.

The repository is not created or populated by this correction task. Repository creation is permitted only after this EWO receives effective Founder Approval/publication, separate GUI implementation authorization, and the complete EMO-002 prerequisite gate in §6.3 has passed.

The recommended Linux VM checkout location is:

```text
/home/sudonimm/Development/SynapseOS/synapse-control-centre
```

That path is a local workspace convention, not repository identity. The project must not be created on `E:\` and must not be placed inside the `synapse-runtime` working tree.

### 14.2 Rationale

- The Control Centre is a distributable desktop product with a release, installer, dependency, security and lifecycle boundary independent of the Runtime library/workspace.
- Keeping it out of `synapse-runtime` prevents Tauri/WebView/frontend dependencies and platform installer tooling from entering the Runtime's trusted workspace or lockfile.
- The Runtime must remain independently buildable and operable without the GUI; separate repositories make that invariant visible.
- The GUI can pin and consume an accepted SDK revision while Runtime and GUI evolve on separate cadences.
- Independent repository permissions and CI reduce the blast radius of GUI build and release tooling.

### 14.3 Repository creation gate

Before the first source commit, the implementation stage must record purpose, owner, Active Development status, visibility, licence treatment, release model, security reporting route, consumers and dependency on `synapse-sdk`. Recommended initial visibility is private until a separate publication/licensing review. No public release follows automatically from EWO approval.

## 15. Internal Implementation Boundaries

The following boundaries are required but do not prescribe excessive layering:

```text
Bundled semantic UI
    -> narrow typed Tauri command allow-list
    -> UI view-model/application state
    -> Observation adapter (only production component directly depending on synapse-sdk)
    -> ObservationClient
    -> independently running local Runtime Observation Service
```

- **Observation adapter:** owns the client and exact SDK calls; its narrow interfaces expose no SDK objects or presentation concepts.
- **Application state:** owns ephemeral connection state, loading state, selected actor and stale snapshots; no Runtime authority.
- **View-model mapping:** preserves all `Option` and `ObservationStatus` meaning; no invented defaults.
- **Presentation:** renders inert view models and user intent; cannot perform direct system access.

The synchronous, non-`Sync` Observation Client is owned by one dedicated Rust worker. SDK calls never block the WebView/UI event loop. The work queue is bounded and coalesces duplicate refresh intent; Phase 1 permits at most one connection attempt and one active refresh batch per connection. A connection-fatal result invalidates the worker's complete client/session, cancels or rejects queued work for that session, and causes all prior projections to become stale. The worker must not retain or retry the failed stream. No periodic high-frequency polling is required. Expensive actor-panel data is fetched on selection or explicit refresh, not eagerly for every actor in the list.

## 16. Refresh, Disconnect and Reconnection

- **Connect:** one explicit user action to the default local endpoint; `ObservationClient::connect` transitions to `Validating`, and bootstrap `GetRuntimeStatus` success alone transitions to `Connected`.
- **Refresh all:** while connected, query Runtime status and durable actors, then refresh the currently selected actor's panels. Each result commits independently and retains its own sequence/status.
- **Panel refresh:** individual panel retry is allowed after a non-connection-fatal semantic error without refreshing unrelated panels. A timeout, transport, response-size overflow, malformed-response, framing, or decode failure invalidates the complete session and permits no panel retry on that stream.
- **Disconnect:** drop the client locally; preserve prior projections only as visibly stale.
- **Connection-fatal failure:** discard the client/session, transition to `Disconnected` plus the truthful error classification, mark every projection stale, stop or reject queued work for that session, and issue no further operation on the failed stream.
- **Reconnect:** one explicit attempt creates a new client/session and repeats `Connecting → Validating`. Successful validation does not clear stale state except for the bootstrap Runtime-status projection that actually succeeded. Every other surface must be re-observed before it becomes current.
- **No automatic retry loop:** exponential backoff, background reconnect and launch-time scanning are unnecessary in Phase 1 and are excluded unless later authorized.

## 17. Cross-Platform Strategy

- One shared Rust core and one shared semantic frontend.
- Platform-specific code is limited to Tauri/build configuration that cannot be shared; no platform-specific Runtime semantics.
- Native builds and native smoke execution are required on Windows, Linux and macOS. Cross-compilation alone is insufficient.
- The same product contract and status vocabulary applies on all platforms.
- Platform differences in WebView, fonts, focus, scaling, menus and installer behavior are documented and tested rather than hidden behind “cross-platform” as an assumption.
- The GUI does not redesign the accepted endpoint/security implementation. It depends on Founder-accepted EMO-002 native bounded-I/O evidence and must additionally exercise its own real SDK connection, silent-peer response, session invalidation, and refusal/error presentation on applicable native paths.

## 18. Accessibility Strategy

No specific formal accessibility compliance standard is selected here because `DES-004` `CC-46` / `ARCH-016` explicitly defer that decision. Phase 1 nevertheless must satisfy the already-preserved accessibility attributes:

- complete keyboard reachability with logical focus order and visible focus;
- semantic headings, landmarks, lists, tables, buttons and status regions;
- accessible names and descriptions for every interactive control and status badge;
- no color-only distinction among available/partial/unavailable/unsupported/stale states;
- sufficient contrast and support for OS text scaling/zoom;
- status and error changes exposed through appropriate live-region semantics without excessive announcements;
- screen-reader smoke tests on Windows, macOS and Linux using platform-appropriate tooling;
- no pointer-only interaction and no animation required to understand state.

Any failure that prevents a usable keyboard/screen-reader path on a target platform and cannot be corrected within Tauri's existing presentation boundary triggers `EWO-028-CAN-STOP-05`.

## 19. Packaging and Update Strategy

Implementation produces native, unsigned test bundles first and release-candidate bundles only after all native gates pass:

| Platform | Minimum packaging evidence |
|---|---|
| Windows | Native `tauri build`; per-user NSIS installer or equivalent Tauri-supported installer; WebView2 prerequisite behavior documented; signing readiness recorded |
| macOS | Native `.app` plus `.dmg`; signing/notarization procedure documented; no distribution without the required credentials and release authority |
| Linux | Native AppImage plus one package appropriate to the supported baseline (initially Debian package); WebKitGTK/runtime prerequisites documented |

Build each artifact on its target OS or a target-native CI runner. Record toolchain versions, hashes and SBOM/dependency inventory appropriate to the repository. Packaging validation does not authorize publication, installation on user systems, app-store submission or deployment.

The Tauri updater plugin is excluded from Phase 1. Users update by obtaining a later separately released signed installer. Any automatic-update mechanism requires a separate decision covering signing keys, update metadata hosting, rollback, channel policy and operational authority.

## 20. Dependency and Supply-Chain Requirements

- Pin the exact `synapse-sdk` Git revision that receives Founder Implementation Acceptance under EMO-002; no branch dependency and no local path dependency in committed manifests.
- Pin exact compatible Tauri 2 and frontend tool versions through `Cargo.lock` and the selected frontend lockfile.
- Use vanilla TypeScript and the smallest practical build toolchain; no component framework or general state-management package.
- Add no plugin or package without a recorded necessity, licence, maintenance, security, portability and replacement assessment.
- Generate a direct/transitive dependency inventory before Independent Implementation Review.
- Record the Tauri system prerequisites for each platform and the minimum Linux distribution/WebKitGTK baseline.
- Treat major Tauri/WebView integration updates as explicit re-review triggers, not automatic maintenance.
- A vulnerability, licence conflict, abandoned dependency or unavailable target support that cannot be mitigated without changing the selected technology triggers `EWO-028-CAN-STOP-07`.

## 21. Controlled Implementation Stages

No stage begins until this exact EWO candidate has passed Independent Review, received effective Founder Approval, been published, and received separate GUI implementation authorization; and until EMO-002 has completed every prerequisite lifecycle/evidence step in §6.3. Documentation review or approval of EMO-002 alone is insufficient.

### Stage 0 — repository and baseline realization

- Fresh-fetch Runtime and Docs; verify the approved/published EWO and the exact Founder-accepted EMO-002 Runtime/SDK revision.
- Create the dedicated repository only under the approved organization/location and repository policy.
- Record ownership, visibility, licence treatment, security contact and release boundary.
- From a clean checkout containing no developer-local Runtime tree, prove that Cargo fetches and builds the named `synapse-sdk` crate at one full immutable Git revision, with `Cargo.lock` committed and no floating branch/tag or path override.
- Record the private repository access strategy and least-privilege CI credential path without committing credentials. Reproduce the proof in clean CI, not only on a developer machine.
- Stop if any clean-checkout/revision/authentication proof fails; do not solve it by moving the GUI into `synapse-runtime`, copying SDK source, or depending on a local checkout.

### Stage 1 — minimal Tauri shell and security posture

- Scaffold Tauri 2 plus vanilla TypeScript.
- Establish one window/WebView, local bundled assets, strict CSP, explicitly listed capability configuration, an explicit application-command manifest, typed IPC schemas, and no prohibited plugins.
- Add static architecture tests for manifests, exact SDK import allowlist, forbidden host APIs, commands, capabilities, events/alternate IPC and remote-content absence.
- Produce an empty native window on all three platforms before product features are added.

### Stage 2 — typed Observation adapter and deterministic state model

- Add the single-client worker, bounded queue and typed adapter over the seven SDK methods; no other production component directly imports the SDK.
- Define connection, validation, connection-fatal invalidation, loading, status, error and stale state transitions as pure testable logic.
- Use a fake `ObservationSource` for deterministic unit tests; no fake is used as production data.
- Preserve every optional field and status without default fabrication.

### Stage 3 — connection shell and Runtime overview

- Implement explicit connect/disconnect/reconnect and the `Connecting → Validating → Connected` bootstrap model using `GetRuntimeStatus`.
- Implement Runtime Overview from `RuntimeStatusView` only.
- Add explicit unavailability treatment for version/build/uptime/identity/count fields.
- Validate that bootstrap authorization/compatibility failure causes no semantic operation dispatch after compatibility rejection, no fallback, and immediate client/session disposal; do not invent a handshake operation or infer `Unauthorized` from ambiguous Unix EOF.

### Stage 4 — durable browse and actor detail panels

- Implement the Durable Actors list and its permanent partial-discovery disclosure.
- Implement actor overview, capabilities, effects-by-actor, bounded supervision and diagnostics panels.
- Fetch selected-actor data intentionally; do not prefetch every panel for every actor.
- Add loading/empty/partial/unavailable/unsupported/error/stale states for each panel.

### Stage 5 — truthfulness, accessibility and resource hardening

- Complete keyboard, semantic, scaling, contrast and screen-reader behavior.
- Verify stale/reconnect behavior and independent sequence handling.
- Measure cold start, idle memory, refresh latency and query volume on documented fixtures; report measurements without inventing an architecture-level target.
- Confirm no UI-thread SDK call, unbounded queue, polling storm or eager all-actor detail fetch.

### Stage 6 — native conformance, packaging and report

- Run the complete validation suite on genuine Windows, Linux and macOS environments.
- Build and smoke-test the native package formats in §19.
- Complete a changed-file scope audit, dependency/licence/security inventory and structural read-only audit.
- Author the Engineering Report and submit the implementation for Independent Implementation Review. Do not publish or deploy the application under this stage.

Each stage is a small, reviewable commit or commit series. A later stage may not conceal a failed earlier gate.

## 22. Required Tests

### 22.1 Rust/application-core tests

- All connection-state transitions, including `Disconnected → Connecting → Validating → Connected`, unauthorized, incompatible, unavailable, timeout, disconnect, connection-fatal invalidation, and reconnect.
- `ObservationClient::connect` is transport-only; `GetRuntimeStatus` is the bootstrap operation; no eighth handshake operation exists.
- Exact mapping from `ObservationStatus` to the four UI truth labels.
- Timeout, transport failure, response-size overflow, malformed response, framing failure, and decode failure each discard the client/session, reject later operations on that stream, and mark every prior projection stale.
- Reconnect creates and validates a new client/session; it never reuses the invalidated stream and only the successful bootstrap Runtime-status projection becomes current automatically.
- Per-operation sequence isolation; no cross-operation atomicity inference.
- `lifecycle_state: None` renders “no live instance,” not unknown.
- Complete-empty versus partial-empty semantics for every collection.
- Diagnostics `Unsupported` renders unsupported, never “no issues.”
- Effect optional identifiers/status/providers remain optional and payload content cannot enter the view model.
- One bounded connection/query owner and coalesced duplicate refresh behavior, backed by the accepted EMO-002 SDK semantics rather than a GUI-only deadline around an unbounded SDK thread.

### 22.2 Frontend tests

- Every product surface renders loading, available, partial, unavailable, unsupported, error and stale cases.
- Durable-only language is present in non-empty, empty, partial and stale list states.
- No prohibited controls or action verbs exist.
- Keyboard order, visible focus, semantic names and status announcements.
- No remote requests or remote asset references in production output.

### 22.3 Structural security tests

- Only the Observation adapter manifest directly depends on `synapse-sdk`, at one full immutable Git revision; all other production components depend only on the adapter's narrow owned types.
- Parsed production-source checks allow only explicitly named `synapse_sdk::foundation::observation` symbols and reject wildcard/prelude SDK imports, `synapse_sdk::foundation::Runtime`, Runtime construction/bootstrap/handles, arbitrary SDK objects, and generic request/dispatch values.
- Manifests and source reject direct Runtime, provider, wire and endpoint dependencies/imports; unsafe Rust; `extern`/FFI; sidecars; shell/process execution; unrestricted filesystem/network/HTTP access; and raw alternate IPC.
- Exact Tauri capability configuration, application-command manifest/list and typed IPC schemas match the independently reviewed allow-list; one window/WebView/capability set exists.
- No arbitrary frontend-supplied command names, generic dispatcher/invoke wrapper, raw IPC request, generic event channel, or alternate bridge exists. Backend command parameters are typed and validated.
- No shell/filesystem/HTTP/SQL/process/global-shortcut/updater/remote-content plugin, remote origin, arbitrary navigation, wildcard command, or remote asset exists.
- Production CSP is restrictive and release WebView developer tools are disabled.
- Independent manual source review confirms the trusted Rust core remains least-authority; static checks are not represented as proof of mathematical impossibility.
- Authorization failure surfaces no fallback and no Runtime detail beyond the SDK category.

### 22.4 Integration and native tests

- Real Control Centre process connects through the pinned SDK to a separately running Runtime with Observation explicitly enabled.
- The Control Centre and Runtime service are genuine separate OS processes; a single test process using server/client threads does not satisfy this gate.
- Each of the seven query operations is exercised through the application adapter.
- Connection refusal when the endpoint is absent/disabled and truthful `Unavailable` presentation.
- Existing Runtime native authorization behavior is consumed unchanged; the GUI does not weaken or bypass it.
- Runtime termination while connected; reconnect to a restarted Runtime; all previous views remain stale until individually refreshed.
- Genuine native Windows, Linux and macOS silent/hung peers accept the connection/request and then withhold a response; the SDK call returns within the accepted bound, the client/session is invalidated, no later operation is sent on that stream, prior projections remain stale, and reconnect uses a new client/session.
- Genuine native peers continuously send bytes without a JSON Lines delimiter beyond the accepted client response ceiling; accumulation remains bounded, the session is invalidated, and no stream reuse occurs.
- Native transport failure and malformed/decode-response cases establish the same session-fatal/no-reuse semantics.
- Genuine Windows, Linux and macOS window launch, connect, browse, detail and package-install smoke paths.
- Native keyboard and screen-reader smoke evidence per §18.

### 22.5 Quality and dependency gates

At minimum, using the then-approved exact toolchain and lockfiles:

```text
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo build --workspace --all-targets --all-features --locked
cargo test --workspace --all-targets --all-features --locked
npm ci
npm run check
npm test
npm run build
cargo tauri build
git diff --check
```

Equivalent commands may replace frontend command names only where the repository's committed scripts provide the same objectively documented gate. Results and independently re-derived test counts must be recorded in the Engineering Report.

## 23. Acceptance Gates and Definition of Done

Implementation is complete only when all statements are true:

1. EMO-002 has completed all eight lifecycle/evidence prerequisites in §6.3; the exact Founder-accepted Runtime/SDK commit is recorded.
2. The dedicated repository exists under the approved identity and contains no Runtime production source copy or submodule.
3. The GUI builds from clean checkout with locked dependencies on Windows, Linux and macOS.
4. Only the Observation adapter directly depends on `synapse-sdk`, at the exact EMO-002-accepted revision; every other production component consumes only the narrow adapter interface.
5. The app is a separate process and connects only through the real `ObservationClient`.
6. `GetRuntimeStatus` is the bootstrap operation; no eighth operation or separate handshake exists.
7. All seven Observation queries are reachable through typed adapter paths and no other Runtime operation exists.
8. Timeout, transport, response-size overflow, malformed-response, framing, and decode failures are session-fatal; no failed stream is reused and native busy-endpoint/silent-peer/unterminated-response gates pass.
9. Runtime Overview displays no unsupported version/build/uptime/identity/count claim.
10. Durable actor browsing is permanently and correctly disclosed as partial discovery.
11. Actor detail, capabilities, effects, supervision and diagnostics conform exactly to §§8-10.
12. Available, partial, unavailable, unsupported and stale states pass deterministic tests.
13. Disconnect/reconnect never establishes false continuity and never silently freshens prior data.
14. Static and manual audits confirm the exact adapter/import policy, no Runtime mutation path, no direct Runtime embedding, no CLI/provider/filesystem fallback, no alternate IPC/event bridge, no remote content, and no prohibited Tauri plugin/host capability.
15. Review artifacts contain the exact capability configuration, command manifest/list, IPC schemas, CSP, plugin list and dependency manifests, all matching this EWO.
16. UI work cannot block the UI event loop or generate unbounded/query-storm load.
17. Accessibility attributes and native smoke paths in §18 pass, with limitations disclosed.
18. Native package artifacts build and smoke-test on all three targets; no release/deployment is inferred.
19. Format, lint, build, test, dependency, licence, vulnerability, scope and diff gates pass or have a Founder-approved documented exception.
20. Zero unresolved Critical or Major Independent Implementation Review findings remain.
21. An Engineering Report records exact commits, files, dependency inventory, native evidence, package hashes, test counts, deviations, findings and remaining limitations.

## 24. Engineering STOPs

Because the historical draft's native STOP identifiers are unrecoverable, the following are newly assigned **canonical reconstruction identifiers**. They are not represented as recovered historical identifiers.

- **`EWO-028-CAN-STOP-01` — Observation contract expansion.** Stop if implementation appears to require an eighth operation, new field, protocol generation, wire shape, transport, push/subscription or remote endpoint.
- **`EWO-028-CAN-STOP-02` — Runtime/SDK prerequisite or production change.** Stop before any GUI implementation unless EMO-002 has completed every lifecycle/evidence prerequisite in §6.3. During GUI work, stop before editing `synapse-runtime` or changing SDK production behavior and return the dependency to its own lifecycle.
- **`EWO-028-CAN-STOP-03` — Mutation, authority, or IPC bypass.** Stop if any Runtime mutation, administrative endpoint, direct embedding, CLI/provider/filesystem path, raw wire access, alternate source of Runtime truth, generic Tauri dispatcher/IPC/event bridge, or forbidden host capability appears necessary.
- **`EWO-028-CAN-STOP-04` — Standalone exact-revision SDK unrealizability.** Stop if a clean dedicated-repository checkout cannot reproducibly consume the exact accepted `synapse-sdk` revision with `Cargo.lock` and least-privilege CI authentication, without a local path, copied source, Runtime modification, direct forbidden dependency, or boundary bypass.
- **`EWO-028-CAN-STOP-05` — Cross-platform/accessibility failure.** Stop if Tauri cannot provide the required Windows/Linux/macOS or usable keyboard/screen-reader path without changing the selected technology or architecture.
- **`EWO-028-CAN-STOP-06` — Scope expansion.** Stop if global discovery, Application authority, persistent audit, remote/multi-Runtime, restart identity/continuity or richer diagnostics become necessary to continue.
- **`EWO-028-CAN-STOP-07` — Dependency/security/licence failure.** Stop on an unmitigable critical dependency risk, incompatible licence, abandoned required component, need for a forbidden direct dependency/plugin/host capability, or security configuration that must be weakened to function.
- **`EWO-028-CAN-STOP-08` — Authorization absent.** Stop if the exact EWO under implementation lacks completed Independent Review, effective Founder Approval and publication evidence.
- **`EWO-028-CAN-STOP-09` — Baseline divergence.** Stop on unexplained Runtime/Documentation/repository baseline mismatch; investigate rather than reset or overwrite.
- **`EWO-028-CAN-STOP-10` — Release authority.** Stop before public distribution, deployment, app-store submission, signing-key operation or automatic-update enablement without separately verified release authority.

A STOP returns evidence and the smallest concrete decision needed to the Founder. It does not authorize unilateral redesign.

## 25. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| WebView compromise reaches local system APIs | Local assets only; strict CSP; minimal capability/command allow-list; no plugins or generic dispatch |
| GUI accidentally acquires Runtime authority | Narrow adapter as the only direct SDK consumer; explicit Observation-symbol allowlist; forbidden dependency/API checks plus independent source review |
| Partial durable list appears complete | Permanent Durable Actors label and partial-discovery explanation in every state |
| Old data appears current after reconnect | Orthogonal stale flag; per-operation re-observation required |
| Synchronous SDK or silent peer blocks progress | EMO-002 prerequisite; dedicated single owner worker and bounded queue; native silent-peer/session-invalidation evidence |
| Query load harms Runtime | Manual/on-navigation refresh, no eager all-actor details, no polling storm, recorded measurements |
| Platform WebView differences break usability | Genuine native UI/accessibility/package tests on all three targets |
| Separate repo drifts from SDK | Exact Git revision pin, lockfile, compatibility tests and explicit update review |
| Tauri/frontend supply-chain growth | Framework-free frontend, no plugins by default, dependency inventory and necessity review |
| Packaging mistaken for release authority | Build/smoke only; explicit release STOP and updater exclusion |

## 26. Reporting Requirements

The Engineering Report must record:

- exact Runtime, Documentation, EWO publication and Control Centre repository commits;
- exact EMO-002 candidate, approval, implementation, independent implementation review, Founder acceptance and native-evidence identities;
- repository creation decision evidence, visibility/licence treatment and owner;
- complete changed-file and staged-scope audit;
- selected exact Tauri/Rust/Node/toolchain/dependency versions and lockfile identities;
- exact Tauri capability configuration, command manifest/list, IPC schemas, CSP and plugin list;
- direct/transitive dependency, licence and vulnerability review;
- mapping of every §23 gate to source/tests/evidence;
- exact commands and independently re-derived test counts;
- native Windows/Linux/macOS run identifiers and outcomes;
- package names, formats, SHA-256 hashes and signing/notarization state;
- performance/resource measurements with environment and fixture disclosed;
- accessibility checks and limitations per platform;
- all Engineering STOPs encountered and their dispositions;
- all Independent Implementation Review findings by severity and disposition;
- confirmation that no Runtime/SDK source, Observation contract or release/deployment state changed outside authorization.

## 27. Self-Review for Filing

This is an AI-assisted author-side hostile self-review only. It is not Independent Engineering Review, does not satisfy `STD-031` §7.1, and does not replace an independent human reviewer where governance requires human independence.

### 27.1 Method

The complete candidate was checked against:

- `ARCH-016` projection, partiality, staleness, security, read-only and future-extension invariants;
- `ARCH-017` connection order, trust boundary, seven-operation closure, truth metadata and failure-isolation requirements;
- exact public SDK types and methods at Runtime `e48d41da7c94281dcb00b57eb40f63aa1db74984`;
- accepted `EWO-029.1`-`.6` implementation and native evidence boundaries;
- `IER-028-100-F01`–`F03`, `IER-028-100-O01`–`O04`, the Founder-fixed trusted-core/WebView threat model, and the separate EMO-002 prerequisite candidate;
- `STD-001`, `GOV-013` and `STD-031` EWO/lifecycle requirements;
- architecture contradiction, accidental mutation authority, unsupported Runtime claims, technology/repository assumptions, security regression, protocol expansion, scope creep, cross-platform risk, stale historical claims and missing review obligations.

### 27.2 Findings

| Severity | Count | Disposition |
|---|---:|---|
| Critical | 0 | None found |
| Major | 0 | None found |
| Minor | 0 | None found |
| Observation | 4 | Recorded below; non-blocking |

- **`SR-028-O01` — historical artifact identity unavailable.** Exact prior bytes/version/STOP identifiers are unrecoverable. Disposition: disclosed in frontmatter, Filing Notice, version rationale and newly namespaced STOP identifiers.
- **`SR-028-O02` — system WebView divergence.** Tauri uses different engines across targets. Disposition: genuine native UI/accessibility/package gates are mandatory; no identical-rendering claim is made.
- **`SR-028-O03` — SDK distribution boundary.** `synapse-sdk` is not published at the accepted Runtime baseline and is workspace-owned. Disposition: a full-revision Git dependency proof is Stage 0; local paths, source copying and moving the GUI into Runtime are prohibited; failure triggers `EWO-028-CAN-STOP-04`.
- **`SR-028-O04` — AI-assisted correction independence.** This author-side correction and self-review are AI-assisted and cannot self-certify independent human review. Disposition: explicitly disclosed; the next stage remains Independent Engineering Re-Review.

### 27.3 Self-review verdict

**PASS FOR FILING TO INDEPENDENT ENGINEERING RE-REVIEW.** No Critical, Major or Minor author-side self-review finding remains. Historical `IER-028-F01` and `IER-028-F02` remain resolved as classified by the prior review. `IER-028-100-F01`–`F03` are addressed, not self-resolved. This verdict approves nothing and authorizes no implementation.

## 28. References

### 28.1 Repository authorities and evidence

- `architecture/ARCH-016-Control-Centre-Foundation-Architecture.md` v0.2.1, Approved; blob `62f6c325eaadcd61d999cae1ee4c1a2e30204de2`; SHA-256 `c462aa0e2ef57d97c86e9dd02eede4c6c968cf8a8426a5f2a2cc8a220d55ba49`.
- `architecture/ARCH-017-Runtime-Observation-Connectivity-Architecture.md` v0.2.2, Approved; blob `3689aa0c00358c4135c7092f31c66fc44f36cf2a`; SHA-256 `4fd10286622fe69dc1f263bdee714580bb9c5b25eef29ab68e0434130c981235`.
- `consolidation/DES-004-Control-Centre-Design-Exploration.md` v0.2.1, Founder-accepted.
- `consolidation/DES-005-Runtime-Observation-Connectivity-Design-Exploration.md` v0.2.0, Founder-accepted.
- `work-orders/EWO-027-Audit-Pipeline-Downstream-Audit-Event-Consumption.md` §42 sequencing evidence.
- `work-orders/EWO-029.1-Evidence-and-Internal-Projection-Foundation.md` through `work-orders/EWO-029.6-Cross-Platform-Observation-Conformance.md`.
- `maintenance/EMO-002-Bounded-Cross-Platform-Observation-Client-IO-and-Session-Invalidation.md` v0.1.0, Review candidate; implementation not authorized.
- `standards/STD-001-Documentation-Standards.md`; `standards/STD-031-Engineering-Lifecycle-Standard.md`.
- `governance/GOV-003-Governance-Model.md`; `governance/GOV-010-Decision-Framework.md`; `governance/GOV-013-Engineering-Lifecycle.md`.
- Runtime `sdk/src/foundation/observation.rs` at `e48d41da7c94281dcb00b57eb40f63aa1db74984`.
- Locked `interprocess` 2.4.3 Windows local-socket stream implementation and Runtime `runtime/src/observation_service/windows.rs` at the same Runtime commit.
- Founder `EWO-029.6` Implementation Acceptance Decision, `EWO-028` Unblocking/Resumption Decision, and `EWO-028 v0.1.0` Narrow Correction Authorization, supplied directly in the governing engagement record.

### 28.2 GUI technology primary sources

Reviewed 2026-08-13:

- Tauri overview and system-WebView/resource posture: <https://v2.tauri.app/start/>
- Tauri process/WebView model: <https://v2.tauri.app/concept/process-model/>
- Tauri platform prerequisites: <https://v2.tauri.app/start/prerequisites/>
- Tauri capabilities and permission boundary: <https://v2.tauri.app/security/capabilities/>
- Tauri Content Security Policy: <https://v2.tauri.app/security/csp/>
- Tauri distribution, installers and signing: <https://v2.tauri.app/distribute/>
- Electron architecture/resource comparison: <https://www.electronjs.org/docs/latest/>
- Electron process/security maintenance model: <https://www.electronjs.org/docs/latest/tutorial/process-model> and <https://www.electronjs.org/docs/latest/tutorial/sandbox>
- Slint Rust/native desktop positioning and licence: <https://github.com/slint-ui/slint>
- `egui`/`eframe` platform, API-stability and accessibility disclosures: <https://github.com/emilk/egui>
- Flutter desktop and native-code binding model: <https://docs.flutter.dev/platform-integration/desktop> and <https://docs.flutter.dev/platform-integration/bind-native-code>

## 29. Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 0.1.0 | 2026-08-13 | Denver Jacobs (AI-assisted reconstruction) | First canonical repository-tracked reconstruction of the previously chat-delivered, unfiled `EWO-028`; exact historical bytes/version/STOP identifiers explicitly unrecoverable. Incorporates the accepted `ARCH-017`/`EWO-029` Observation foundation; treats `IER-028-F01` and `F02` as ADDRESSED pending Independent Review; selects Tauri 2 with a framework-free semantic frontend; recommends a dedicated `synapse-control-centre` repository; defines the smallest truthful read-only product contract, structural guarantee, staged implementation, tests, gates and newly identified canonical STOPs. Author-side self-review: 0 Critical, 0 Major, 0 Minor, 3 Observations. Filed with status `Review`; no Founder Approval or implementation authorization. |
| 0.1.0 (Review candidate 2) | 2026-08-13 | Denver Jacobs (AI-assisted correction) | Founder-authorized narrow material correction after Independent Exact-Artifact Review returned `MATERIAL CORRECTION REQUIRED`. Addresses `IER-028-100-F01` by making EMO-002's complete lifecycle/native evidence a hard GUI prerequisite and defining connection-fatal/no-reuse semantics; addresses `F02` by recording the trusted-Rust-core/hostile-WebView threat model, narrow adapter, explicit source policy and exact Tauri authority artifacts; addresses `F03` with transport-only connect and `GetRuntimeStatus` bootstrap validation. Incorporates `O01`–`O03`, preserves `O04`, strengthens Stage 0, native/process tests, gates and the existing ten canonical STOPs. Historical `IER-028-F01`/`F02` resolution is preserved. Review status; no approval or implementation authorization. |

## 30. Approval Status

| Role / Stage | Holder | Status | Date |
|---|---|---|---|
| Founder Resumption Decision | Denver Jacobs, Founder | **EWO-028 RESUMED for correction/reconstruction and review preparation only** | 2026-08-13 |
| Author | Denver Jacobs (AI-assisted) | Reconstructed and filed v0.1.0 | 2026-08-13 |
| Independent Engineering Review of candidate 1 | AI-performed technical reviewer; human independence not claimed | **MATERIAL CORRECTION REQUIRED** — 0 Critical, 2 Major, 1 Minor, 4 Observations | 2026-08-13 |
| Founder Narrow Correction Authorization | Denver Jacobs, Founder | **AUTHORIZED for documentation correction and EMO-002 authoring only** | 2026-08-13 |
| Author self-review of candidate 2 | Denver Jacobs (AI-assisted; not independent review) | Pass for re-review: 0 Critical, 0 Major, 0 Minor, 4 Observations | 2026-08-13 |
| Independent Engineering Re-Review | TBD | **NOT YET PERFORMED** | — |
| Founder Approval | Denver Jacobs | **NOT GRANTED** | — |
| Publication of Approved EWO | — | **NOT YET PERFORMED; requires prior Founder Approval** | — |
| Implementation | — | **NOT AUTHORIZED** | — |

## 31. Disposition

**EWO-028 v0.1.0 REVIEW CANDIDATE 2 — NARROWLY CORRECTED AND FILED FOR INDEPENDENT ENGINEERING RE-REVIEW.**

Historical `IER-028-F01` and `IER-028-F02` remain **RESOLVED**. `IER-028-100-F01`–`F03` are **ADDRESSED, pending independent re-review**. Tauri 2 and the dedicated `synapse-control-centre` repository remain bounded recommendations under the corrected least-authority design. EMO-002 v0.1.0 is the separate prerequisite candidate; neither its documentation filing nor any EWO-028 disposition authorizes Runtime/SDK implementation.

No approval or implementation is implied by authorship, self-review, filing, commit or push. GUI implementation remains blocked even if this corrected EWO later passes review, until §6.3's complete EMO-002 prerequisite lifecycle/evidence and separate GUI authorization exist. The exact next lifecycle stage is **Independent Engineering Review of corrected EWO-028 candidate 2 and EMO-002 v0.1.0**.

**STOP — return control to the Founder.**
