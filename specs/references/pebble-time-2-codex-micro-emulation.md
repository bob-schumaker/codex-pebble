# Pebble Time 2 as a Codex Micro-Style Controller

## Conclusion

A Pebble Time 2 application can credibly emulate the *workflow* of a Codex
Micro: a glanceable dashboard for active Codex agents plus compact controls for
common actions. It cannot be a literal replacement for the Micro as a USB or
Bluetooth HID keyboard.

The best product framing is therefore a wrist-worn Codex remote and status
display, rather than a keyboard emulator.

## Intended capabilities

- Show a small, prioritized set of Codex threads/agents and their states.
- Use color, text, and haptics for states such as ready, working, waiting for
  desktop input, complete, and error.
- Select an agent/thread and request a predefined, desktop-confirmed workflow.
  Watch-initiated Codex approvals/rejections, focus integration, and free-form
  instructions are post-MVP candidates, not MVP capabilities.
- Use touch for selecting cards, swiping between pages, and optional quick
  actions; retain button equivalents for dependable one-handed use.

The actual Codex Micro has 13 mechanical keys, a joystick, a dial, RGB agent
status keys, and configurable controls. The watch should reproduce the
high-value interaction model—not attempt a one-to-one physical control layout.

## Pebble Time 2 fit

Pebble lists the Time 2 as having a 1.5-inch, 64-color e-paper touch screen at
200 x 228 pixels, four physical buttons, a microphone and speaker, RGB
backlight, vibration motor, and an estimated 30-day battery life. Its stated
interaction model is "4 buttons + touchscreen." This makes it suitable for a
small status dashboard and discrete controls, but not for dense text editing or
a keyboard-like number of shortcuts.

Touch support is confirmed by the current Time 2 product specification and by
the Pebble app ecosystem. The touchscreen should not be treated as a reason to
remove physical-button paths: Pebble has reported active Time 2 touch accuracy
and reliability issues, so every core action should remain accessible from the
buttons.

## Recommended architecture

```text
Pebble Time 2 watch app
       <AppMessage>
Pebble phone companion
       <paired, authenticated private-LAN API>
desktop bridge service
       <Codex app-server (local stdio JSON-RPC)>
Codex session(s)
```

1. The watch app renders cached agent state and sends compact commands to its
   phone companion.
2. The companion authenticates to a desktop bridge, which is responsible for
   connecting to Codex and normalizing state updates. The bridge should launch
   or own a local `codex app-server` over its default stdio JSONL transport.
3. The bridge sends a bounded, privacy-conscious status model back to the
   watch: agent id, label, state, elapsed time, and whether human input is
   required.

This companion/bridge design is required because a Pebble watch application is
not a general-purpose computer keyboard device. It also avoids putting Codex
credentials or broad computer-control permissions on the watch.

### Codex integration surface, validated against local source

The local Codex checkout confirms that `codex app-server` is the appropriate
desktop bridge boundary. It is the JSON-RPC 2.0 interface used by rich Codex
clients, including the VS Code extension. Its normal transport is stdio JSONL;
its TCP WebSocket transport is explicitly experimental and unsupported. The
bridge should therefore keep app-server local and expose its *own*, authenticated
API to the phone rather than relay the app-server socket across the network.

The app-server offers the data needed for this project:

- `thread/list` and `thread/read` return thread status; `thread/loaded/list`
  identifies active in-memory threads.
- `thread/status/changed` reports `notLoaded`, `idle`, `systemError`, or
  `active` (with activity flags).
- `turn/started`, `item/*`, and `turn/completed` provide live progress and the
  final states `completed`, `interrupted`, or `failed`.
- `turn/steer` can add a concise instruction to an in-progress regular turn;
  `turn/interrupt` requests cancellation and completes with `interrupted`.
- `thread/list` can query a root thread's spawned descendants through the
  experimental `ancestorThreadId` filter, which is useful when the dashboard is
  intended to display a multi-agent group.

This means the status dashboard does not need screen-scraping or OS keyboard
automation. For an initial release, use the public app-server protocol and map
only its coarse state to watch colors: active -> working, idle -> ready,
system error/failed -> error, and a pending app-server approval request ->
waiting for desktop. Do not infer approval state from raw tool output or offer
its approval choices on the watch.

## Scope and constraints

- Treat the watch as a remote UI, not a standalone Codex client.
- Keep payloads small and cache the last known state for disconnected use.
- Require explicit user confirmation for consequential actions such as applying
  changes, executing commands, or responding to approval prompts.
- Authenticate the companion-to-desktop connection and restrict it to a known
  device/session; do not expose an unauthenticated local-control endpoint or a
  public Internet listener in the MVP.
- Begin with read-only status, then add only the project specification's
  desktop-confirmed action catalog. Do not add write, command-execution,
  approval, or focus actions outside that catalog.
- Treat the app-server WebSocket transport as unsuitable for this product's
  external bridge because the checked-out Codex source labels it experimental
  and unsupported.

## Feasible first version

1. A four-to-six agent dashboard with colored state chips.
2. Tap/select an agent; use buttons to switch between overview and detail.
3. Haptic notification when an agent completes, fails, or needs a response.
4. Two safe actions: request an uncommitted-worktree review and interrupt an
   active regular turn; both require desktop confirmation.
5. A desktop bridge that uses only documented/supported Codex integration
   points, with a mocked status provider for early watch development.

## Sources

- Pebble Time 2 product specifications and touchscreen support:
  <https://repebble.com/watch>
- Pebble developer hardware information (Emery/Time 2 platform):
  <https://d2r0ymlem3140p.cloudfront.net/developer.pebble.com/guides/tools-and-resources/hardware-information/index.html>
- PebbleKit JS bidirectional app messaging:
  <https://developer.repebble.com/docs/pebblekit-js/Pebble/>
- Codex Micro product overview:
  <https://openai.com/es-ES/supply/co-lab/work-louder/>
- Pebble’s July 2026 Time 2 software-issue update (touch reliability caveat):
  <https://repebble.com/blog/pebble-mega-update-july-2026>
- Local Codex source checked on 2026-07-18:
  `/Users/roschuma/Repos/github/codex/codex-rs/app-server/README.md`
  (protocol, thread/turn events, and transport guidance).
