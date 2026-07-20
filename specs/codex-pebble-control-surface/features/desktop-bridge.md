# Feature: Desktop Bridge

## Objective

Operate locally on the developer's computer as the authenticated boundary
between the phone companion and Codex.

## Requirements

- MUST expose a minimal authenticated API for snapshots, event updates, and
  action requests.
- MUST own pairing and reject unknown clients before accessing Codex.
- MUST normalize Codex events into the shared protocol and retain only the
  least state needed to serve a reconnecting companion.
- MUST rate-limit and deduplicate action requests.
- MUST present desktop-side confirmation before any consequential action.
- MUST log non-sensitive audit records for pairing, requests, confirmations,
  and failures.
- MUST own selected-thread configuration and the desktop confirmation surface.
- MUST bind only to the configured private-LAN interface, reject public-network
  interfaces, and require TLS plus the paired-device session on every request.
- MUST coalesce outbound status updates when the companion is slow and require
  a fresh snapshot after reconnect or bridge restart.

## Acceptance criteria

- Given an authenticated companion, when Codex emits a relevant status event,
  then the bridge publishes one monotonically newer normalized update.
- Given an unauthenticated client, when it requests a snapshot or action, then
  the bridge returns no Codex state and makes no app-server call.
- Given a duplicate request identifier, when it is repeated, then the bridge
  returns the original terminal result without executing it twice.

## Deferred

Multi-user hosting, Internet exposure, and cloud relay infrastructure.
