```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the ended-run slot and teardown diff.

**Per-case list**

- `entryEnded`, done entry: true. Correct.
- `entryEnded`, entry gone from the local board: true. Correct for a removed entry, with one caveat (finding F1).
- `entryEnded`, entry with a run status or no status: false. Correct.
- `DobbyRuns`, `PairPresent`: `markerExists` on the worktree path. A missing path gives false. Correct.
- `slotsTaken` and `runsInFlight` with `EntryEnded`: an ended run frees its slot whatever its state. Correct.
- `strandedEnded`: failed, halted and paused-short-of-parent runs give true. Running, paused-with-work-on-parent and done give false. A run with a pass in flight, a held driver lock, a cancel marker, a covering pause or an owned-elsewhere mark gives false. Correct.
- `tearDownEnded` (daemon), nil dependency: returns nil. Correct.
- `tearDownEnded` (daemon), result not ready: sets `busy` and returns. Correct.
- `tearDownEnded` (daemon), result ready: reads it, clears `d.teardown`, then re-evaluates the stranded set. Correct.
- `tearDownEnded` (daemon), no due slugs: starts nothing. Correct.
- `tearDownEnded` (daemon), a slug no longer stranded: its wait is deleted. Correct.
- `readEndedTeardown`, removed pair: logs and waits one probe interval. Correct.
- `readEndedTeardown`, first failure: logs once, waits one probe. Correct.
- `readEndedTeardown`, later failures: the wait doubles up to an hour, with no repeat log. Correct.
- `readEndedTeardown`, nil map from a closed channel: every slug fails as unreported. Correct.
- `loop.go` binary-check guard: `CheckBinary` is skipped while `d.teardown != nil`. Correct.
- `loop.go` guard when `tearDownEnded` is not reached in a tick: the pending result waits for the next tick that reaches it. Correct, because the result channel is buffered and the goroutine does not leak.
- `beamcli.tearDownEnded`, `pairteardown.New` failure: every slug carries the error. Correct.
- `beamcli.tearDownEnded`, reconcile error: every slug carries the error. Correct.
- `endedTeardownResult`, unreachable origin: the error names it. Correct.
- `endedTeardownResult`, entry removed, busy or errored: maps to nil, busy error or failure error. Correct.
- `endedTeardownResult`, slug absent from the report or not ended on origin: returns the "does not show ended" error. Correct.
- `reconcileEnded` guards the hub-wide scope: it tears down only runs with a batten run record that this machine's own landing has not reached, reports a live driver as `Busy`, and skips the scope's own run. Not a defect.

**Findings**

- **F1, MEDIUM, design.** `internal/battencli/dobbyreads.go:156-165` and `entryEnded` at about line 237.
  - `entryOf` is built from `boardengine.ReadTasks(geom.BoardDir)`, the local board. Absence from it is read as "gone", so `EntryEnded` is true.
  - If a run's slug is missing from the local board because the board is stale (a task created or adopted from another machine, or a Bolt not yet pulled), the run reads as ended.
  - The result is that the run frees its `max_runs` slot and counts as stranded when it is not running.
  - The slot is freed on the local board's word alone, while the teardown path judges from origin. That can start more runs than `max_runs` allows.
  - The teardown itself is protected, because the reconcile judges from origin's board. The slot check has no such guard.
  - Evidence: `dobbyreads.go:152-165`, `slotsTaken` in `internal/dobbyengine/starts.go`.
- **F2, LOW, design.** `internal/dobbyengine/ended.go` `readEndedTeardown`.
  - On a nil result ("pair removed") the next attempt waits one probe, and the log line repeats each probe while `PairPresent` stays true.
  - That happens only when the pair stays on disk after the reconcile reported it removed, for example a stray directory at the worktree path. The run is then logged "pair removed" forever, with no failure episode or backoff growth.
  - Evidence: `ended.go` `readEndedTeardown`, nil branch resets `interval` and `failing`.

**Tool calls**
1. Read `internal/battencli/dobbyreads.go:128-168`: to see how `entryOf` is built; the diff showed only part of it.
2. Read `internal/fabricengine/originreconcileended.go:90-210`: to see which runs the hub-wide reconcile tears down and how busy and error outcomes are reported.
