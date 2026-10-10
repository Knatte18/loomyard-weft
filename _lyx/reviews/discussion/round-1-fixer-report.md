# Fixer report: discussion, round 1

I checked every finding against the code it cites, and every premise held.
All ten findings are fixed in `_lyx/discussion/decision-record.md`.
The support log records the round, under `## Review rounds` and in the Interview and Question ledger.
`lyx loom validate-discussion` exits 0.

## Fixed

- **F1 (BLOCKING):** D11 no longer keys the skip on the recording command's `session-start` event, which is written whatever the verb does.
  After a successful render, `lyx orch resume-context` writes a delivery mark in the orch dir, beside the cycle request.
  The watcher skips its pointer only on a mark dated at or after the boundary whose text equals `PendingResume`.
  A failed render writes no mark, so the typed fallback runs.
  The Bound now states that the pointer can arrive twice, which is harmless.
  It also states that a dropped output with a mark written skips the pointer.
  The draft-case and fallback criteria prove delivery by the mark.
- **F2 (BLOCKING):** D11 now picks the verb's pointer from the whole persisted state:
  - `PendingResume` in `resuming`, which covers a watcher tick that enters `resuming` before the hook runs;
  - the resume prompt in `compacting`;
  - the reload prompt otherwise.
  D11 also states what a compaction in `handoff-requested` or `clearing` gets.
  A mismatching mark makes the watcher type its own pointer, so the handoff note is never lost.
  Verified against `tickCompacting`, `reloadAfterCompaction`, `startReload` and `startAutoReload` in `internal/orchengine/watcher.go`.
- **F3 (BLOCKING):** Verified that `SpawnWatchdog` is reached only from `reedcli`'s `up`/`resume`/`attach` and from `loomcli/start.go`.
  `orchcli`, `loomcli/arm.go`, `battencli`, `burlercli`, `webstercli` and a strand add's pre-flight all boot through `reedengine` without it.
  D5 now has `reedengine`'s cold session boot touch the signal, at a path the engine is told on its geometry.
  The `told-geometry` constraint, the acceptance criterion, the plan-writer note, the support log's Interview turn 4 and Q3 follow.
  The assumption that understated the gap is dropped.
- **F4 (MEDIUM):** D5 gains a Bound.
  Any operator process can touch the file, but a signal wake runs no faster than the 5 s base, so the worst case is today's cadence.
  lyx's own writer touches the signal once per booted session.
  A session created by hand is found within 60 s.
- **F5 (MEDIUM):** D7 releases an unmatched turn start as `turnStartHold` does in `internal/shuttleengine/send.go`: by a later turn end, by a reported interrupt, or by the 10 s idle override.
  The watcher keeps one hold across its ticks.
  D7's Bound covers the override, and a new criterion covers the release.
- **F6 (MEDIUM):** D7 reads the folded state, not the newest signal, and covers every state name in `internal/shuttleengine/sessionstate.go`:
  - `idle-stalled` of any cause is ready;
  - `dead` holds;
  - `unknown` of any cause falls back to the pane probe;
  - a `session-start` signal leaves the reading unchanged.
  The readiness criterion mirrors this.
- **F7 (LOW):** D5 now stats first: only a file found there is consumed and runs a cycle, so the loop's own remove event runs nothing.
- **F8 (LOW):** D7 now keeps `IdleProbe` with a new `Reason` field, and the plan-writer note agrees.
- **F9 (LOW):** The fsnotify v1.9.0 pin moved into D3, and the assumption line is gone.
- **F10 (NIT):** Open risks are cut to the residual risks: watch the first smoke run for a dialog, and read the first live compaction's transcript.
  The orphan-reap risk and the `/color` risk restated D5 and D7, so they are dropped.
  Neither carried a requirement the Decisions lack.

## Disputed

None.

## Deferred

None.
