# Project Brief

## Purpose

Build a Pebble Time 2 wrist remote for Codex. It recreates the useful workflow
of Codex Micro—glanceable agent/thread state and compact controls—without
emulating a physical keyboard.

## Product boundary

The watch is a remote UI. A local desktop bridge owns Codex integration,
credentials, thread selection, security decisions, and any consequential action.
The watch is never a standalone Codex client or HID keyboard.

## Authoritative artifacts

- `specs/codex-pebble-control-surface/project-spec.md`
- `specs/codex-pebble-control-surface/features/`
- `specs/references/pebble-time-2-codex-micro-emulation.md`

## MVP scope

- Display up to six bridge-selected Codex threads.
- Show normalized status, cache freshness, connection state, and haptic
  transitions.
- Allow only snapshot refresh, uncommitted-worktree review request, and active
  regular-turn interruption; consequential actions require desktop confirmation.

## Explicit exclusions

- Watch-originated Codex approvals, raw terminal commands, transcripts, diffs,
  credentials, free-form prompts, and HID emulation.
- Public Internet exposure, iOS support, PebbleKit JS companion support, and
  arbitrary desktop focus/open integration in MVP.
