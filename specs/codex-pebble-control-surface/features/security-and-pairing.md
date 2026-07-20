# Feature: Security and Pairing

## Objective

Ensure the wrist remote remains a narrowly authorized control surface rather
than a path to unaudited desktop or Codex control.

## Requirements

- MUST operate only on a configured private LAN for MVP. The bridge MUST not
  bind a public interface, provide a port-forwarding guide, or accept plaintext
  HTTP.
- MUST pair a companion to a specific local bridge using a user-visible,
  one-time pairing ceremony. The desktop owner opens the bridge's local setup
  page, which displays a 128-bit one-time pairing code and bridge certificate
  fingerprint; the phone's companion settings page accepts both values.
- MUST use TLS for companion-to-bridge traffic and pin the bridge certificate
  fingerprint established at pairing.
- MUST generate a random per-device secret at pairing, retain it only in the
  Android companion's Keystore-backed storage and the bridge's protected local
  store, and authenticate every request with device ID, expiry, nonce, and MAC
  derived from that secret.
- MUST accept a session for at most 24 hours before renewal and reject a nonce
  already used within its 10-minute replay window.
- MUST bind requests to a paired device and reject replay, expiry, and unknown
  protocol versions.
- MUST require desktop confirmation for consequential actions in the MVP.
- MUST not store Codex credentials on the watch or transmit them through the
  companion protocol.
- MUST redact sensitive text from watch-facing errors and audit records.
- MUST allow the desktop owner to revoke a device immediately; revocation
  deletes its secret, invalidates its sessions, and clears its watch cache on
  the next connection.
- MUST log only action/request IDs, timestamps, device aliases, sanitized
  labels, decisions, and error codes. Logs remain local for 30 days unless
  deleted earlier.

## Threat model and confirmation boundary

The MVP defends against an unpaired phone, LAN observer/replay, a stale watch,
and accidental watch input. It does not defend against a compromised paired
phone or unlocked desktop; those are explicit post-MVP hardening concerns.

The watch is never an app-server approval client. When app-server requests a
command, file-change, permission, MCP, or user-input decision, the bridge
shows it only to the desktop owner. If that UI is unavailable, it responds with
`decline` or `cancel`; it must not leave the turn stalled or ask the watch to
decide.

## Acceptance criteria

- Given a request without valid pairing state, when it reaches the bridge, then
  it is rejected before any Codex adapter action.
- Given a captured valid request, when replayed after completion, then it does
  not trigger a second action.
- Given a command-changing request, when desktop confirmation is denied or
  times out, then the bridge reports a declined result and leaves Codex state
  unchanged.
- Given a revoked device or expired session, when it calls the bridge, then the
  bridge rejects it before returning a snapshot or touching Codex.

## Deferred

Enterprise device management, shared-device tenancy, and remote Internet
exposure.
