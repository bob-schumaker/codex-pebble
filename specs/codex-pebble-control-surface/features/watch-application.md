# Feature: Watch Application

## Objective

Provide a fast, legible Pebble Time 2 UI for selected Codex thread state and
safe remote-action requests.

## Requirements

- MUST render up to six selected threads with a short label, normalized status,
  and freshness indicator.
- MUST treat touch and swipe as optional accelerators; physical buttons are the
  complete, reliable core path.
- MUST provide overview, selected-thread detail, and connection-state views.
- MUST use haptics for completion, failure, and waiting-for-desktop events,
  respecting the user's notification settings.
- MUST cache the most recent valid status snapshot and show its age when
  disconnected.
- MUST never store desktop credentials, Codex tokens, or full transcripts.
- MUST use this button map: up/down changes selected slot or action; select
  opens detail or submits an action request; back returns/cancels; a long back
  opens connection state. Touch tap mirrors select and swipe mirrors up/down.
- MUST haptically notify only on a selected thread's transition into `completed`,
  `failed`, or `waiting_for_desktop`, never on initial snapshot/reconnect, and
  no more than once per thread per 60 seconds.

## Acceptance criteria

- Given six selected threads, when the overview opens, then each is visible or
  reachable through a single paginated view without text overlap.
- Given an event with a newer revision than the cache, when it arrives, then
  the UI updates its matching thread only.
- Given an action request, when the user selects it, then the UI shows that it
  is pending desktop confirmation rather than implying completion.
- Given unavailable touch input, when the user uses the buttons, then they can
  select a thread, inspect details, and request each MVP action.

## Deferred

Free-form prompt composition, transcript browsing, and raw shell interaction.
