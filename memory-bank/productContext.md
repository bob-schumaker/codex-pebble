# Product Context

## User and problem

The initial user is one developer running Codex on one desktop. They need a
low-distraction way to see selected work in progress and react safely without
opening the desktop UI for every status check.

## User experience

- The overview shows up to six ordered, privacy-safe thread labels.
- Touch is an optional accelerator; all core navigation works with Pebble's
  buttons because Time 2 touch reliability is not assumed.
- Status colors/haptics communicate `working`, `waiting_for_desktop`, `ready`,
  `completed`, `failed`, `interrupted`, and `unavailable`.
- A watch action is only a request. The desktop owner confirms consequential
  actions locally before Codex is called.

## Privacy posture

Watch payloads contain opaque IDs, safe labels, coarse state, and timestamps.
Prompts, transcripts, command output, diffs, credentials, and detailed errors
stay off the watch and out of normal audit records.
