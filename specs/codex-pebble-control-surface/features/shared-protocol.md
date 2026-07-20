# Feature: Shared Protocol and Fixtures

## Objective

Define a small, versioned contract shared by the watch, companion, bridge, and
tests so component releases cannot silently disagree about state or actions.

## Requirements

- MUST define versioned status snapshots, incremental updates, action requests,
  action results, and typed error envelopes.
- MUST use opaque thread identifiers and short display labels; no prompt,
  transcript, command, diff, or credential fields belong in the MVP status
  payload.
- MUST include a monotonically increasing revision or sequence number for
  ordering and stale-update rejection.
- MUST define an idempotency key for every action request.
- MUST supply fixtures for working, ready, failed, waiting-for-desktop, stale,
  and disconnected states.

## Version 1 contract

All messages are UTF-8 JSON objects. Each envelope contains `version: 1`,
`type`, `bridge_epoch` (a random identifier generated at bridge start), and a
`sent_at_ms` bridge timestamp. The bridge is the source of ordering; watch and
companion clocks do not determine freshness.

The MVP does not fragment messages. A serialized watch-bound message MUST be
at most 2,048 bytes. The bridge omits optional fields first; if a full snapshot
still exceeds the limit, it sends a typed `payload_too_large` error and the
companion requests a compact replacement snapshot. A six-thread snapshot is a
release-gate fixture and MUST fit the budget.

### Snapshot and delta

```json
{
  "version": 1,
  "type": "status_snapshot",
  "bridge_epoch": "opaque-random-id",
  "revision": 41,
  "sent_at_ms": 0,
  "connection_state": "connected",
  "threads": []
}
```

`revision` is a strictly increasing integer within one `bridge_epoch` and
applies to the complete bridge state. A `status_delta` carries `revision` and
the changed complete thread record. Consumers accept a delta only when its
epoch matches and its revision is greater than their cached revision. A new
epoch, a revision gap, or reconnection requires a replacement snapshot; the
snapshot discards all earlier cached state.

`connection_state` is `connected`, `stale`, or `disconnected`. The bridge marks
state `stale` after 10 seconds without a successful adapter update and
`disconnected` when the companion or bridge link is unavailable. The watch
calculates displayed cache age from `sent_at_ms` only as presentation metadata.

### Canonical thread status

Each selected slot has `slot` (1–6), opaque `thread_id`, `label`,
`thread_status`, and `last_changed_at_ms`. The only `thread_status` values are:

| Status | Meaning | Codex mapping / precedence |
| --- | --- | --- |
| `working` | A regular turn is in progress | `turn/started` until terminal event; wins over raw `active` |
| `waiting_for_desktop` | Codex asked the bridge client for approval/input | Pending server request; never watch-approvable |
| `ready` | Loaded and not executing | app-server `idle` |
| `completed` | Latest observed turn completed successfully | `turn/completed: completed`; retained for 10 minutes then `ready` |
| `failed` | Latest observed turn/system operation failed | `turn/completed: failed` or `systemError` |
| `interrupted` | Latest observed turn was interrupted | `turn/completed: interrupted`; retained for 10 minutes then `ready` |
| `unavailable` | Saved selection cannot be read | `notLoaded`, missing, archived, or adapter unavailable |

The adapter maps an app-server approval request to `waiting_for_desktop`; it
returns to the item/turn-derived state only after `serverRequest/resolved` and
the authoritative `item/completed` or `turn/completed` event. `connection_state`
is separate from this status and always takes precedence in watch presentation.

### Actions

```json
{
  "version": 1,
  "type": "action_request",
  "bridge_epoch": "opaque-random-id",
  "request_id": "UUID",
  "action": "interrupt_active_turn",
  "thread_id": "opaque-id",
  "expected_turn_id": "opaque-id"
}
```

Action state is one of `accepted`, `awaiting_desktop_confirmation`,
`executing`, `succeeded`, `declined`, `expired`, or `failed`. Every action
result includes the original `request_id`, action, and a typed error code when
not successful. The bridge retains an idempotency record for 10 minutes; a
duplicate request returns its current or terminal action state and never invokes
the adapter twice.

Stable error codes are `unauthenticated`, `unsupported_version`,
`invalid_request`, `unsupported_action`, `thread_unavailable`,
`turn_not_active`, `bridge_unavailable`, `adapter_unavailable`,
`confirmation_unavailable`, `payload_too_large`, and `internal_error`. Error
messages sent to the watch are fixed, non-sensitive display text.

## Acceptance criteria

- Given an update with an older revision than cached state, when received, then
  a consumer ignores it.
- Given a fixture, when every component's contract test parses it, then each
  either accepts it or reports a version-compatible validation error.
- Given a new required field, when a protocol version changes, then older
  consumers fail closed rather than assume unsafe defaults.
- Given a bridge restart or missing revision, when a client receives an update,
  then it requests and replaces its state with a new snapshot before rendering
  later deltas.

## Deferred

General public API compatibility or third-party bridge clients.
