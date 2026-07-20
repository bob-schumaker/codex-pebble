# Feature: Codex App-Server Adapter

## Objective

Use local `codex app-server` JSON-RPC as the supported source of Codex state
and action execution for the desktop bridge.

## Requirements

- MUST launch or manage `codex app-server` using its stdio JSONL transport.
- MUST perform `initialize`/`initialized` handshake before other RPC calls.
- MUST use `thread/list`, `thread/read`, `thread/loaded/list`, and subscribed
  notifications to maintain selected-thread state.
- MUST translate `thread/status/changed`, `turn/started`, and
  `turn/completed` into the shared normalized status model.
- MUST treat the app-server TCP WebSocket transport as unsupported for this
  product.
- MUST expose only an explicit allowlist of action RPCs to the bridge.
- MUST handle every server-initiated approval/input request for a selected
  thread: surface it to the local desktop owner, respond `decline` or `cancel`
  if the owner cannot respond, and never forward approval choices to the watch.
- MUST use the app-server version that generated the checked-in protocol fixture
  set; an incompatible initialize/schema result puts the bridge in
  `adapter_unavailable` state.

## Lifecycle and RPC allowlist

1. Start `codex app-server --stdio`, send `initialize`, then `initialized`.
2. Obtain an initial thread list, read every saved selected thread, and publish
   a replacement snapshot before any delta.
3. Resume each selected saved thread with `thread/resume` so this bridge-owned
   app-server connection receives its turn/item notifications. A missing,
   archived, or failed resume leaves that saved slot `unavailable`; the bridge
   does not replace it automatically.
4. Translate notifications according to the shared protocol. For
   `item/commandExecution/requestApproval`,
   `item/fileChange/requestApproval`, permission, user-input, or MCP
   elicitation requests, enter `waiting_for_desktop` and use the desktop-only
   approval flow. If unavailable, send `decline`/`cancel` to app-server.
5. On app-server exit, mark every selected thread `unavailable`, restart with
   exponential backoff capped at 30 seconds, reinitialize, and publish a fresh
   snapshot under a new bridge epoch before deltas resume.

The only watch-requestable adapter calls in MVP are `review/start` for
uncommitted changes and `turn/interrupt`. `turn/steer`, approval responses,
shell commands, and config writes are prohibited.

## Acceptance criteria

- Given a valid app-server process, when the adapter initializes, then it can
  list threads and consume the notifications for subscribed threads.
- Given a `turn/completed` event with `failed` or `interrupted`, when received,
  then the adapter emits the corresponding terminal status without exposing the
  error's sensitive details to the watch.
- Given an app-server overload or disconnection, when it occurs, then the
  adapter retries safely or reports bridge-unavailable state without issuing an
  uncontrolled action retry.
- Given an app-server approval request while no desktop confirmation surface is
  available, when it arrives, then the adapter declines/cancels it and exposes
  only `waiting_for_desktop` then a redacted terminal state to the watch.

## Deferred

Support for experimental app-server APIs except where an explicitly approved
feature requires one, such as descendant-thread listing.
