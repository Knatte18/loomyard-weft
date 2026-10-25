# Board entry `dobby-skip-done-runs`: Dobby counts no done run against max_runs

Done batten runs neither count as in flight nor stay in dobby state, so released tasks start

## Seen (2026-10-25, issue #872)

The prime's dobby started no released, ready task (`umbrella-anchor`, `max_runs` 3).
`parentRuns` (`internal/beamcli/dobbydeps.go`) lists every batten status file under the prime's `_lyx/shed/` without reading its state, and `runsInFlight` (`internal/dobbyengine/starts.go`) counts them all, so 74 records in state `done` counted as in flight and `startReady` stopped at `max_runs`.
Dobby's `state.json` also kept a `full_pass_owed` entry for every one of them.
Workaround used: `lyx batten cleanup --apply` in the prime.

## Fix (operator, 2026-10-25)

- A run whose batten state is `done` never counts against `max_runs` and is never planned a pass.
- Dobby's state record drops its entry for a done run once nothing more is owed it, so `state.json` holds only live runs.
- The board tidy's landing count still reads done runs' history as it does now; only the in-flight count and the pass planning skip them.
- A test pins it: done records beside a released ready task, and dobby starts the task.
