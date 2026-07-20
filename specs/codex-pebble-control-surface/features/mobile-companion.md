# Feature: Mobile Companion

## Objective

Relay compact state and command messages between the Pebble watch app and the
paired desktop bridge. The MVP is a native Android companion using PebbleKit
Android, not PebbleKit JS.

## Requirements

- MUST register the watch application's UUID with PebbleKit Android and carry
  watch messages through its supported app messaging mechanism. The packaged
  watch app MUST not include `src/pkjs/index.js`, because PebbleKit JS and a
  native PebbleKit companion cannot be used together.
- MUST maintain pairing/connection state and report it to the watch.
- MUST validate message shape, protocol version, and size before forwarding.
- MUST reconnect safely without replaying consequential commands.
- MUST transfer only the normalized status and allowlisted command model.
- MUST request a replacement snapshot on reconnect, new bridge epoch, revision
  gap, or unsupported cached protocol version.
- MUST treat touch-triggered commands as requests only; it must not queue an
  action while offline or replay one after reconnect.
- MUST store the paired bridge secret only in Android Keystore-backed storage;
  it MUST not place the secret in the watch, SharedPreferences, logs, or a
  Pebble app message.

## Acceptance criteria

- Given a valid paired bridge response, when it contains a status snapshot,
  then the companion forwards it to the watch in the shared protocol format.
- Given an expired pairing or malformed message, when received, then the
  companion rejects it and reports a non-sensitive error state.
- Given a temporary network interruption, when connectivity returns, then the
  companion requests a fresh snapshot rather than replaying old actions.

## Deferred

Background remote networking beyond the chosen local-network or private-overlay
design.

iOS support and a PebbleKit JS-only companion are separate post-MVP work; they
require their own credential-storage and lifecycle design.
