# Dobby counts no done run against max_runs and keeps no state for it

Fixes #872.
Dobby's run listing reads every batten status file of its worktree's runs whatever the run's state, so finished runs counted as in flight.
On the prime, 74 records in state `done` filled `max_runs` and dobby started no released, ready task; its `state.json` also held a `full_pass_owed` entry for each of them.

## What changes

- `drivePasses` in `internal/dobbyengine/passes.go` drops every listed run whose batten state is `done` before it keeps the tick's runs, records owed passes and plans passes.
  A done run therefore takes no slot in `runsInFlight`, gets no pass, and its `Runs` entry is removed from the state record by the existing drop of unlisted runs in `recordOwed`.
- A done run whose pass this dobby spawned is still in flight stays listed until that pass's exit is read, so a new start never takes the slot of a process that still runs; the next tick drops it.
- The board tidy's landing count is unchanged: it reads every run's history through its own `Landings` seam, done runs included.
- Only `done` is skipped; halted, paused and failed runs still count against `max_runs`.
- The package doc in `internal/dobbyengine/doc.go` states the rule in its passes and starts bullets.

## Design decisions

- The filter sits in the engine rather than in beamcli's `parentRuns`, because that listing also feeds the landing count, and the engine's fake-seam harness pins the behavior without real status files.

## Tests

Two subtests in `TestStartReady_RefusalsAndBackoff` (`internal/dobbyengine/starts_test.go`):

- done runs beside a released ready task under `max_runs: 1`, with a dobby start owing every run a full pass and a stale `full_pass_owed` entry seeded: the task starts, no done run gets a pass, and the state record holds no run entry;
- a run that turns done while its pass is held in flight keeps its slot until the pass exits, after which the task starts and the run's entry is gone.
