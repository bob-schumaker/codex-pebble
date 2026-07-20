# Feature: Deployment and Verification

## Objective

Define the supported MVP environment and the evidence required before a release
is offered to a user.

## Requirements

- MUST support Pebble Time 2 with the current supported Pebble SDK, the Pebble
  mobile app, and a native Android companion built with PebbleKit Android.
  Unsupported watch, mobile, or bridge runtimes fail closed with an explanatory
  compatibility state.
- MUST support Android 15+ and macOS 15+ for MVP and package the bridge as a
  user-launched local process before adding service-manager installation.
- MUST document bridge installation, pairing, upgrade, unpair, and uninstall.
- MUST expose a local health status that distinguishes app-server unavailable,
  paired companion unavailable, and healthy.
- MUST retain redacted audit logs locally for 30 days by default and provide a
  user-initiated delete operation.
- MUST use structured logs with a correlation ID shared by an action request,
  confirmation, adapter call, and terminal result.

## Release gates

- Contract tests validate all protocol fixtures against watch, companion, and
  bridge parsers.
- Adapter integration tests cover initialization, status transitions,
  server-initiated approval decline, restart, and overload/disconnect recovery.
- Security tests cover invalid pairing, expired session, replay, duplicate
  action, and redaction behavior.
- End-to-end tests cover snapshot, stale cache, reconnection, confirmation
  timeout, and each MVP action using a controlled app-server fixture.
- The watch UI is verified in the Pebble emulator and on a physical Time 2;
  all core flows work with buttons alone.
- The Android companion is tested against Android Keystore persistence, process
  death/restart, and Pebble mobile-app connection loss.
- On the supported LAN, status propagation meets p95 <= 2 seconds and a
  six-thread snapshot fits the protocol message budget.
