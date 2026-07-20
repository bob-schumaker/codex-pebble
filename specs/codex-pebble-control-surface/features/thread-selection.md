# Feature: Thread Selection and Configuration

## Objective

Make the bridge the authoritative owner of the up-to-six Codex threads shown on
the watch.

## Requirements

- MUST provide a desktop-local configuration view listing discoverable Codex
  threads and the current ordered selection.
- MUST persist selected opaque thread IDs, an optional user-provided display
  label, and a configured worktree only on the desktop bridge.
- MUST limit the selection to six threads. A seventh selection attempt MUST be
  rejected until one is removed.
- MUST prefer a user-provided label; otherwise it MUST show a privacy-safe
  bridge-generated label such as `Thread 1`, never a prompt-derived title.
- MUST preserve a missing, archived, or unloaded selection as `unavailable`
  until the desktop owner removes it; it MUST not silently select a replacement.
- MUST include a descendant only when the desktop owner explicitly selects it.

## Acceptance criteria

- Given no saved selection, when the watch opens, then it shows an explicit
  empty-state message and no Codex thread data.
- Given six selected threads, when the desktop owner adds another, then the
  bridge rejects it and identifies the limit without changing the selection.
- Given a selected thread disappears from `thread/list`, when a snapshot is
  published, then its slot remains present with status `unavailable`.
- Given a user changes the order or label, when the next snapshot arrives, then
  the watch uses the saved order and sanitized label.
