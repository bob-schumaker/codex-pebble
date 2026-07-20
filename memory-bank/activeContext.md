# Active Context

## Current status

The project is specification-complete for implementation planning. No runtime
code, build configuration, or test harness exists yet.

## Immediate next work

1. Convert the approved feature specs into an implementation plan and monorepo
   package layout.
2. Establish the Pebble Time 2/Android companion development toolchain.
3. Build protocol fixtures and a mocked desktop bridge before connecting to a
   live Codex app-server.

## Decisions already made

- Android-only native companion for MVP.
- One user/watch/phone/desktop bridge on a private LAN.
- Desktop-only confirmation and no watch-originated Codex approvals.

## Deferred decisions

- Post-MVP iOS companion and secure credential design.
- Private-overlay/VPN or remote connectivity.
- Desktop Codex UI focus/open integration and turn steering.
