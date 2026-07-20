# Memory Bank

This directory stores durable context for the Codex Pebble Control Surface.

## Core files

- `projectbrief.md` — project purpose, scope, and authoritative specifications.
- `productContext.md` — intended user value and MVP behavior.
- `systemPatterns.md` — architecture and security boundaries.
- `techContext.md` — confirmed platform, protocol, and integration constraints.
- `activeContext.md` — current working horizon and pending decisions.
- `progress.md` — completed milestones and remaining work.

## Conventions

- The feature specifications under `specs/codex-pebble-control-surface/` are
  authoritative for implementation requirements. This memory bank summarizes;
  it does not override them.
- Keep current-session plans, task checklists, and restart handoffs outside this
  directory unless they become durable project facts.
- Update `activeContext.md` and `progress.md` after material project changes;
  update the other files only when their durable facts change.
