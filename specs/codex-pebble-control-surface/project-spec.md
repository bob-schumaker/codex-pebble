# Codex Pebble Control Surface — Project Specification

**Status:** Draft — MVP decisions resolved for implementation planning
**Goal:** Build a Pebble Time 2 wrist remote that displays Codex activity and
offers safe, compact controls for selected Codex threads.

## Product boundary

The product recreates the useful workflow of the Codex Micro—agent awareness
and rapid actions—not its physical keyboard form. The Pebble is a remote UI;
Codex runs on the user's desktop and remains the authority for execution,
approvals, and credentials.

## Monorepo components

| Feature | Responsibility | Deployment unit |
| --- | --- | --- |
| [Watch application](features/watch-application.md) | Wrist UI, input, haptics, local cached state | `.pbw` watch app |
| [Mobile companion](features/mobile-companion.md) | Pebble-to-bridge relay and connection state | Native Android PebbleKit companion |
| [Desktop bridge](features/desktop-bridge.md) | Local adapter, pairing endpoint, state aggregation | Local desktop process |
| [Codex adapter](features/codex-app-server-adapter.md) | Codex app-server lifecycle and event translation | Bridge module |
| [Shared protocol](features/shared-protocol.md) | Versioned control/status messages and fixtures | Shared package/schema |
| [Security and pairing](features/security-and-pairing.md) | Authorization boundaries and confirmation policy | Cross-cutting feature |
| [Thread selection](features/thread-selection.md) | Selected-thread ownership and lifecycle | Bridge configuration |
| [Desktop confirmation](features/desktop-confirmation.md) | Trusted action-confirmation flow | Bridge UI module |
| [Deployment and verification](features/deployment-and-verification.md) | Supported platforms, release gates, and test matrix | Cross-cutting feature |

## Primary user stories

1. As a developer with Codex work in progress, I can glance at my watch to see
   which selected threads are working, ready, failed, or waiting for desktop
   input.
2. As that developer, I can select a thread and request a small allowlisted
   action without opening the desktop UI.
3. As that developer, I receive haptic feedback when a selected task completes,
   fails, or needs my attention.
4. As a security-conscious developer, I know no watch gesture can silently
   approve a consequential action or expose my Codex credentials.

## Functional requirements

- The system MUST display the state of up to six selected Codex threads.
- The system MUST continue to show the last synchronized state while the watch
  is disconnected, clearly marked as stale.
- The system MUST use both touch and physical-button paths for core navigation
  and actions.
- The bridge MUST obtain Codex state through local `codex app-server` JSON-RPC,
  using supported stdio transport.
- The phone-to-desktop connection MUST be paired and authenticated.
- The MVP MUST be read-only by default; every action that can change code,
  execute commands, or resolve an approval MUST require desktop-side
  confirmation.

## Explicitly out of scope for MVP

- Emulating a USB/Bluetooth HID keyboard.
- A standalone Codex client on the watch.
- Sending raw terminal commands from the watch.
- Passing Codex account credentials, full prompts, transcripts, tool output, or
  file diffs to the watch by default.
- Direct use of the app-server experimental TCP WebSocket transport.
- Watch-originated approval or rejection of Codex command, file-change,
  permission, or MCP requests.
- Opening or focusing an arbitrary desktop Codex UI.

## Release increments

1. **Read-only status:** paired watch shows mock then live thread state.
2. **Notifications:** haptics and stale/disconnected indications.
3. **Safe controls:** request an uncommitted-worktree review or interruption of
   an active regular turn, both with desktop confirmation.
4. **Optional advanced controls:** desktop focus/open integration and turn
   steering, only after explicit consent UX and audit behavior are validated.

## Authoritative MVP action catalog

| Action ID | Watch label | Preconditions | Bridge effect | Desktop confirmation | Terminal results |
| --- | --- | --- | --- | --- | --- |
| `refresh_snapshot` | Refresh | Paired bridge reachable | Request fresh state snapshot; no Codex mutation | No | `succeeded`, `failed` |
| `request_review_uncommitted` | Review changes | Selected thread has a configured local worktree | Call app-server `review/start` for uncommitted changes on that thread | Required | `succeeded`, `declined`, `expired`, `failed` |
| `interrupt_active_turn` | Stop turn | Selected thread has a known in-progress regular turn | Call app-server `turn/interrupt` with its expected turn ID | Required | `succeeded`, `declined`, `expired`, `failed` |

The watch can request these actions but cannot approve app-server tool or file
change requests. The bridge must surface every server-initiated app-server
approval to the desktop owner and never auto-accept it.

## Cross-feature acceptance criteria

- Given a paired desktop with one active Codex thread, when the app-server emits
  a status change, then the watch reflects the normalized state within a
  2 seconds at p95 on the supported LAN, or displays a connection-stale
  indicator.
- Given a disconnected phone or bridge, when the user opens the watch app, then
  the last known state is visible and is labeled stale; no queued consequential
  command is executed on reconnect.
- Given an action requiring confirmation, when it is requested from the watch,
  then the desktop presents the confirmation before the bridge calls Codex.
- Given an unpaired or unauthenticated phone request, when it reaches the
  bridge, then the bridge rejects it without calling Codex or exposing state.

## Resolved MVP decisions

- The initial target is one user, one Time 2, one phone, and one desktop bridge
  on the same trusted private LAN. Public Internet exposure and overlay/VPN
  operation are post-MVP.
- The bridge has a user-configured LAN address and a TLS identity trusted during
  pairing; it must not listen on a public interface or use unauthenticated HTTP.
- Thread selection is bridge-owned and configured on the desktop; the watch can
  neither add nor remove selected threads in MVP.
- A local bridge-owned confirmation page is the only confirmation surface in
  MVP. Focus/open integration and turn steering are deferred.

## Reference

The underlying feasibility and Codex integration analysis is in
[`../references/pebble-time-2-codex-micro-emulation.md`](../references/pebble-time-2-codex-micro-emulation.md).
