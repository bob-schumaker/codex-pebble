# System Patterns

## Component flow

```text
Pebble Time 2 app
  ↕ PebbleKit Android messages
Native Android companion
  ↕ paired TLS + authenticated private-LAN API
Desktop bridge
  ↕ local stdio JSON-RPC
codex app-server
```

## Ownership boundaries

- **Watch:** renders cached state, sends compact action requests, and provides
  button-first interaction/haptics.
- **Android companion:** transports messages, validates protocol envelopes, and
  keeps the bridge secret in Keystore-backed storage.
- **Desktop bridge:** owns pairing, selected-thread configuration, confirmation
  UI, state aggregation, ordering/recovery, and redacted auditing.
- **Codex adapter:** runs/initializes local `codex app-server`, translates its
  events, and handles every server-initiated approval only on the desktop.

## Protocol rules

- Versioned JSON envelopes carry a bridge epoch and monotonically increasing
  revision.
- A new epoch, revision gap, or reconnect requires a replacement snapshot;
  actions are never queued or replayed by the watch/companion.
- Each action has an idempotency key and a lifecycle from accepted through one
  terminal result.

## Security rules

- MVP is private-LAN only, TLS-protected, paired, and never publicly exposed.
- The watch cannot approve command/file/permission/MCP/user-input requests.
- If desktop approval UI is unavailable, the adapter declines or cancels the
  pending app-server request instead of stalling or delegating it to the watch.
