# Codex Pebble Control Surface

A Pebble Time 2 wrist remote for Codex: glance at selected Codex thread state,
receive haptic updates, and request a small set of desktop-confirmed actions.

This is not a HID keyboard emulator or a standalone Codex client. Codex runs on
the desktop; the watch is a deliberately constrained remote UI.

## Status

The project is in the specification and planning stage. No runtime code or
build configuration exists yet.

## MVP

- Show up to six desktop-selected Codex threads on a Pebble Time 2.
- Render normalized state, freshness, and connection status.
- Provide button-complete navigation; touch is an optional accelerator.
- Allow status refresh, an uncommitted-worktree review request, and active-turn
  interruption. Consequential requests require local desktop confirmation.
- Use a native Android companion, a paired private-LAN connection, a local
  desktop bridge, and `codex app-server` over stdio JSON-RPC.

Watch-originated approvals, raw terminal commands, credentials, transcripts,
public Internet access, iOS support, and arbitrary desktop focus integration are
out of scope for MVP.

## Architecture

```text
Pebble Time 2 app
  ↕ PebbleKit Android messages
Native Android companion
  ↕ paired TLS + authenticated private-LAN API
Desktop bridge
  ↕ local stdio JSON-RPC
codex app-server
```

The bridge owns Codex credentials, selected-thread configuration, confirmation
UI, protocol ordering, and redacted auditing. The watch never approves a Codex
command, file change, permission, or MCP request.

## Project map

- [Project specification](specs/codex-pebble-control-surface/project-spec.md)
  — scope, action catalog, and cross-feature acceptance criteria.
- [Feature specifications](specs/codex-pebble-control-surface/features/) —
  component-level requirements.
- [Feasibility reference](specs/references/pebble-time-2-codex-micro-emulation.md)
  — Pebble and Codex integration research.
- [Memory bank](memory-bank/README.md) — durable project context for future
  work sessions.

## Next steps

1. Create an implementation plan and monorepo package layout.
2. Establish the Pebble Time 2 and Android companion toolchains.
3. Build shared protocol fixtures and a mocked desktop bridge.
4. Integrate the bridge with a local Codex app-server.

## License

No license has been selected yet.
