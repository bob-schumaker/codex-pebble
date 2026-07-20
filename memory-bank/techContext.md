# Technical Context

## Target platforms

- Pebble Time 2 watch app, using current supported Pebble SDK.
- Native Android companion using PebbleKit Android. The project must not ship
  `src/pkjs/index.js`, because PebbleKit JS and native PebbleKit companions are
  mutually exclusive for one watch app.
- macOS desktop bridge for MVP; Android 15+ and macOS 15+ are initial targets.

## Codex integration

- Use local `codex app-server` via its supported stdio JSONL JSON-RPC transport.
- Do not expose or depend on its experimental TCP WebSocket transport.
- App-server integration starts with initialize/initialized, then thread
  discovery/resume and event normalization.
- Watch-requestable calls are limited to `review/start` for uncommitted changes
  and `turn/interrupt`; turn steering and approval responses are prohibited.

## Key limits

- Maximum six selected threads.
- Watch-bound protocol messages are UTF-8 JSON with a 2,048-byte maximum and
  no fragmentation in MVP.
- Bridge declares state stale after 10 seconds without successful adapter
  update; supported-LAN status propagation target is p95 ≤ 2 seconds.
