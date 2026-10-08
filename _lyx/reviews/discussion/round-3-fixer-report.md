# Fixer report — discussion round 3

Every finding was checked against the pair and the code, held, and was fixed.

## Fixed

- **F1 (MEDIUM, decision)** — D3's facts list now says where the output-file stat applies: to every turn end folded in that reading, the wait loop with each poll's stat, a files-only reader with its one stat.
  The `since` list gives `idle-done` one stamp per reader: the turn end's `lyx_at`, and the reading time only in the wait loop when files appear at a later poll.
  The history paragraph splits entries into signal-only (same for every reader), output-check (per reader) and call-time (per reader).
  The change-history acceptance criterion pins the `idle-done` stamp, and the support log's persisted-state-file entry names the output-check entries.
- **F2 (MEDIUM, design)** — D1 drops the one-write claim and states that a large line is flushed in chunks and can interleave, as today's two-write chain can.
  The payload is a `%s` argument, never the format string, and the stamp is inserted only after a leading `{"`, so `{}` and non-objects are appended unchanged.
  The Bound names the Stop and ask rewrite as the one change touching the wait loop's input and requires a stamped line to parse to the same event as its unstamped payload, large and with `%`, `\`, `$` and an escaped `\n`; a corrupted line is skipped as today.
  The first acceptance criterion carries that case.
- **F3 (MEDIUM, decision)** — D3 gains a precedence list: session end, then dead process, then liveness unproven (also over a finished turn end), then unreadable events file and no signal, then the signal-derived states.
  The session-end reason wins over `process-gone`.
  A new acceptance criterion pins the three cases.
- **F4 (LOW, consistency)** — D1 now says the events file, `lyx_at` and the transcript are all agent-writable, so the whole state is advisory in shadow mode, and keeps `lyx_at` out of ordering so no state depends on a clock comparison.
  D3 says the history is ordered by signal order and its `since` values are not guaranteed to increase.
- **F5 (LOW, consistency)** — D6 now says the newest signal-derived entry shows when the state last changed on a signal.
- **F6 (LOW, consistency)** — D3's facts list carries the prompt offset from `run.json`'s `promptOffset`, with offset zero for the orch watcher, and the reducer folds only signals past it; the reducer acceptance criterion gains the signals-before-offset row.

## Disputed

None.

## Deferred

None.
