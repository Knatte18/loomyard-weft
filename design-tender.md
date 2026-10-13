# A tender module for hub-wide pause, resume and revive

`tender` · Note · enhancement, shuttle, reed, batten

## Idea (operator, 2026-10-13)

`lyx pause` and `lyx resume` are two top-level verbs from one package (`internal/pausecli`), so `lyx --help` lists them as if each were a module.
They should live in a module of their own, named `tender` (a loom tender keeps the looms running: stops them, restarts them, watches over them), with `lyx pause` and `lyx resume` kept as shortcuts (one line each in `internal/shortcutcli/shortcuts.yaml`).

Names ruled out (operator, 2026-10-13): `hub` (the whole LYXHUB folder), `treadle` (taken by `internal/treadleengine`), `weaver` (planned as a sister module of Webster), `brake` and `hold` (fit pausing only, not restarting sessions).

## What else could live in `tender`

Hub-wide control over everything that runs, much of it done by the orch by hand today:

- `pause` and `resume`, for the hub or one pair, as today.
- Revive after a network outage: find every session with `cause: api-error` and `state: idle-stalled` (today `lyx shuttle state` in each pair) and give each one line to carry on (today `lyx shuttle send` per session).
- Drain: let every step in flight finish, then stop, unlike pause, which interrupts at once; for a deploy or a hibernation.
- Stale binaries: list long-lived processes running a deleted binary and restart them (overlaps note `stale-binary-daemons`).
- Status: one view of every live session in the hub, who is working, waiting or stuck.
- A cap on concurrent runs (an open point in `orch-split`).

## Scope to discuss

- Which of these belong in `tender`, and which stay with their modules (`shuttle`, `reed`, `batten`).
- Verb names under `tender`, and which get shortcuts.
