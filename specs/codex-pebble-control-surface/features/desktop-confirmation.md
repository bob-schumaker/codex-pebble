# Feature: Desktop Confirmation

## Objective

Require a conscious, local desktop decision before the bridge performs an MVP
action requested from the watch.

## Requirements

- MUST present a bridge-owned local confirmation page for every consequential
  MVP action. The page MUST be reachable only on the local desktop session.
- MUST show action name, sanitized thread label, requesting paired-device name,
  request time, and expiry. It MUST not display prompts, tool output, or diffs.
- MUST default to `declined` after 60 seconds, when the desktop is locked, or
  when the confirmation page is unavailable.
- MUST bind the decision to one action ID, request ID, paired device, and
  current bridge epoch. A decision cannot be reused for another request.
- MUST publish `awaiting_desktop_confirmation` before waiting and one terminal
  action result after execution or refusal.
- MUST log request, decision, terminal result, and redacted failure code.

## Acceptance criteria

- Given a watch requests `interrupt_active_turn`, when the local owner approves
  before expiry, then the bridge calls the adapter exactly once.
- Given the owner rejects, times out, or the desktop locks, when the action is
  pending, then the bridge emits `declined` or `expired` and makes no adapter
  call.
- Given a duplicate request while confirmation is pending, when it arrives,
  then the bridge returns the same pending action state rather than showing a
  second confirmation.
