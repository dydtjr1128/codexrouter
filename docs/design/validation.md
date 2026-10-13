# Delivery and validation

Status: planned. None of the runtime or live-provider checks below has been executed. The repository currently contains documentation and collaboration settings only.

## Milestones

| Milestone | Independently useful result | Exit condition |
| --- | --- | --- |
| M0: connection experiment | Evidence that an unchanged Windows Codex Desktop can reach the intended local route | Catalog selection, credentials, full tool loop, continuation identity, cancellation, and native path constraints measured on an isolated test setup |
| Pre-M1 runtime decision | A justified production-language/package choice | M0 findings plus packaged CLI secret-store access, OS IPC ownership, and lifecycle probes; record the decision and supported platform subset before M1 |
| M1: minimal router and CLI | One daemon, shared control contract, bounded ingress, immutable routes, explicit failures, native forwarding | Native path, CLI-only lifecycle, protected storage, and local trust controls pass; no API-adapter support claim yet |
| M2: required API adapters | Claude and Gemini independently complete the qualified desktop workflow | Both satisfy the common acceptance matrix and provider-specific cases |
| M3: explicit subscription profiles | Separately authorized profiles can be selected for new conversations | Supported enrollment/refresh, aliases/catalog publication, desktop restart/selection, identity binding, concurrent conversations, revocation, and account isolation pass |
| Companion interface | Tray/settings control for accounts, selection, status, and recovery | Same daemon/control semantics as CLI, renderer isolation, concurrent actions, lifecycle and per-OS packaging pass |

Choose a reversible experiment language before M0; confirm the production runtime/storage afterward. M2 can have separate Claude and Gemini implementation issues after the shared contract is stable; completion of one does not satisfy the other. M3 requires the authenticated identity and binding contract, not completion of unrelated provider features. The companion depends on the working control contract and gains account actions as M3 delivers them; it need not block M2. These are delivery milestones rather than automatic issue creation or mandatory PR ordering.

## M0 experiment

Use a separately authorized test configuration and disposable workspace. Preserve the actual user's settings and login. An implementation task must define and obtain any credentials it needs without placing them in the repository. Limit capture to the minimum sanitized metadata and fixtures necessary to answer these questions:

1. Can the target desktop select truthful custom aliases and retain them through new chat, restart, and reopen? Does a native sign-in/quota gate block or substitute an external route before the router receives a request?
2. Which exact bundled client version and catalog schema are in use? Which model metadata fields control tool format, context limits, reasoning, compaction, and migration?
3. Does the user-level custom provider configuration work in both chosen credential modes over authenticated loopback? Are credentials supplied to a desktop launched outside a terminal?
4. Which paths, query parameters, headers, body fields, and content encodings does the client send for model discovery, inference, retries, compaction, and cancellation?
5. What reliable per-thread identity or qualified router-issued handle survives the tool loop and restart? Record reopen-key stability separately from forks. Can rollback/edit/regenerate be distinguished from an accidental stale replay? Can two threads sharing a root session be distinguished? If neither key nor handle works, stop M0. What proves the native principal if affinity is claimed?
6. Which tools are emitted in actual coding tasks: function, custom/freeform, grammar, parallel calls, MCP wrappers, and built-in tools? Which capability settings prevent unsupported requests?
7. Does a minimal tool loop work on each selected native/Claude/Gemini model, including a second turn carrying opaque state? Can required provider state survive a client replay and restart without abusing another provider's fields?
8. Does client cancellation close the upstream transport? Test cancellation after a completed call followed by the next user turn. Are implicit client retries and automatic model migration truly disabled?
9. Does required native state survive restart? When does compaction occur relative to pending tools and the last acknowledgment? Verify an ordinary compacted session continues; report the explicit recovery limit if the client compacts mid-loop.

Record a pass, failure, or unresolved result for every question. If a required gate fails, narrow the advertised scope or stop that milestone and revise the design. Do not replace the client, disguise a model, or broaden credential access to turn a failed experiment into a pass.

## Common acceptance matrix

Run each applicable case for native, Claude, Gemini, and later managed subscription routes. A local mock test is evidence for the translator only; a live API test is not desktop E2E evidence.

| Area | Cases and observable evidence |
| --- | --- |
| Route identity | Selected alias resolves to the exact upstream model/profile revision; unknown/retired aliases fail; catalog migrations do not bypass the router |
| Authentication | Missing/wrong local token rejected before upstream; API keys go only to their destination; native headers go only to native backend |
| Streaming | Fragmented UTF-8, text chunks, multiple items, recognized usage and absent usage, one correct terminal/item outcome, long buffered tool arguments with qualified keepalive behavior |
| Tools | Read-only tool and final answer; create/patch a disposable file through normal Codex approval, read it back, and verify exact content and one execution; two tool turns; parallel results; custom quotes/newlines; invalid/duplicate/orphan calls |
| Constraints | Unsupported schema, strictness, grammar, tool choice, image, hosted tool, and reasoning options rejected before dispatch |
| Continuation | Second user turn, full-history replay, response-ID path, exact native prefix, no duplicate history, alias conflict, acknowledged rollback/edit/regenerate, immediate and ordinary compaction, pending-tool compaction error, inherited-state fork error, partially delivered parallel calls |
| Concurrency | Two conversations on different profiles; serialized same-binding updates; conflicting resume rejected; no cross-binding payload lookup/import; optional same-installation ID metadata used only for error classification |
| Lifecycle | Restart before/after state commit, recover acknowledged vs proven unexposed vs delivery-unknown tentative turns without repeating tools, late results rejected after branch abandonment, expiry, corrupt/full storage, active-state quota pressure |
| Cancellation | Before headers, during text, during tool arguments, after completed call, simultaneous terminal event; no implicit resubmission |
| Failures | Provider 401/403/429/5xx, socket ambiguity, error after HTTP 200, truncated stream, timeout, limit, refusal/safety block |
| Retry boundary | Desktop and router retries disabled; no quota/account/model fallback; ambiguous request marked unknown rather than replayed |
| Local trust | Unexpected Host/Origin, wildcard bind, cross-host redirect, arbitrary input URL, unauthorized administration, listener squatting in both credential modes, and token replacement after suspected takeover |
| Privacy | Headers, errors, logs, state, backups, and support output contain no unintended secrets or conversation content |
| Control and platforms | One daemon; CLI/UI and external-file selection conflicts; ownership recovery; selection observation amid old-chat traffic; shared start/restore lock; no-tray/headless conditions; version mismatch blocks catalog routes including native, with update/restore recovery; packaged CLI secrets and coordinated updates per OS |

An execution test must use a harmless disposable fixture and the client's normal approval path. Never use a real repository mutation just to establish that an adapter emits a tool call.

## Provider-specific cases

### Claude

- Replay the complete assistant content array with tool outputs, including hidden/redacted and empty-text thinking blocks.
- Preserve signature deltas through stream completion; refuse incomplete native state.
- Change the instruction prefix, tool schema, model, and account in separate negative cases; verify the router blocks incompatible continuation before sending it.
- Verify partial JSON arguments, parallel tool grouping, maximum-token termination, and errors after HTTP 200.

### Gemini

- Pin the Interactions API version and verify exact event/delta types against recorded responses, including conflicting documentation examples.
- Assert `store=false` and foreground execution; verify explicit step replay without `previous_interaction_id`.
- Preserve thought/signature-only steps and original function-call/result associations; repeat request-scoped tool/instruction settings.
- Verify `interaction.completed` carrying `requires_action`, `completed`, and `incomplete`, plus failure/error paths; assert tool handoff versus final answer, safety blocking, parallel grouping, and schema limitations.

### Native and managed subscription routes

- Verify the distinction between native Codex backend credentials and separately authorized plan API credentials; never substitute one token type for the other.
- Confirm supported models and entitlements by a scoped execution test, not merely catalog presence.
- Qualify a trustworthy account identity and detect independent desktop login changes before allowing identity-sensitive resume.
- Enroll profiles A and B deliberately; new-chat route selection affects only new conversations, and loss of A never reroutes an A conversation to B.
- Exercise one concurrent refresh per managed profile, rotating refresh tokens, cancellation/revocation during refresh, and crashes around credential persistence.
- Verify desktop account, workspace, plugins, and MCP connections are not represented as having switched with inference profile selection.

## Evidence format and release gate

Store a small sanitized qualification record with client and bundled runtime versions, operating system, router revision, exact upstream model/API versions, credential mode, catalog/capability revision, case results, and known limits. Include no real account identifiers, keys, authorization headers, private prompts, or opaque provider state. Use synthetic fixtures and privately authorized live credentials.

Do not call a route supported until its required cases pass in the actual desktop. The initial product release requires native routing plus both Claude and Gemini; a partial result can be described as an experiment. Requalify affected cases after client, model, API, tool-format, catalog, authentication, or continuation changes. Do not promise compatibility with every future desktop update.

Neither credential mode is qualified for mutually untrusted local users until server authentication or an OS-isolated transport protects the endpoint against port squatting. A deployment that cannot establish that boundary must keep routing unavailable in that environment; a local client token alone is insufficient. A changed desktop/catalog version requires visible requalification or explicit prior-connection recovery rather than stale silent routing.

For this documentation change, validation consists of consistency review, local-link and Mermaid syntax checks, English/provenance checks, and Git whitespace/scope checks. Runtime, provider calls, login, model selection, and E2E tests are not applicable yet and must be reported as unrun.

## Decisions still requiring evidence

| Decision | Evidence needed | Default until resolved |
| --- | --- | --- |
| Runtime and packaging | Node/TypeScript experiment, clean-machine CLI package, measured lifecycle/resources and OS secret storage; compare a focused alternative only when needed | Recommended experiment stack only; no runtime/dependencies installed |
| Desktop discovery/configuration | Actual catalog, alias visibility, native gates, launch environment | No installation command published |
| Conversation identity | Stable key/handle and fork/reopen behavior | No profile-switching or stateful compatibility claim |
| Native principal binding | Supported authenticated identity that survives refresh | No native account-pinned resume claim |
| Managed subscription enrollment | Supported registration, scopes, eligible accounts, token/model contracts | No auth-file import or borrowed OAuth client |
| Physical state store and limits | Minimal required native state, atomic crash recovery, measured size/lifetime | No database or unbounded retention commitment |
| Reasoning and compaction | Recognizable compaction boundary, no unresolved calls, safe new epoch, and successful continued coding | Stop ambiguous transitions; no silent signature removal |
| Optional capabilities | Actual client demand and independent acceptance evidence | Explicitly unsupported |
| Companion and OS coverage | CLI contract, tray/window lifecycle, renderer isolation, independent secret owner, target-client availability | Electron is the leading UI candidate; all platform support remains unverified |
