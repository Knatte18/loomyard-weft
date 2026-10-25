# Decision record — dobby-skip-done-runs

## Goal

A batten run whose state is `done` takes no slot under `max_runs`, is planned no pass, and leaves no entry in dobby's `state.json`, so a beam holding many finished run records still starts its released, ready tasks (issue #872).

## Scope

- In: `internal/dobbyengine`'s pass driving and start cap, the drop of a done run's `Runs` entry from the state record, a test pinning the start beside done records, and the package doc's passes and starts bullets.
- Out: the board tidy's landing count, which keeps reading every run's history whatever its state through the `Landings` seam.
- Out: `lyx batten cleanup`, and the beamcli listing of runs, which keeps listing every batten run of the parent.

## Decisions

### Done runs are filtered in the engine, not in the listing seam

Decision: `drivePasses` drops every listed run whose state is `done` before it keeps the tick's runs, records owed passes and plans passes, unless a pass of that run is in flight.
Rationale: the engine owns what counts as in flight and what is owed, and its fake-seam harness pins the behavior without real status files; the shared beamcli listing also feeds the landing count, which must keep reading done runs.

### A done run with a pass in flight stays until the pass exits

Decision: a done run whose pass this dobby spawned has not exited stays among the tick's runs, so it keeps its slot and its state entry until the exit is read; the next tick drops both.
Rationale: the exit is the last thing owed the run; dropping it earlier would let a start take the slot of a process still running.

### The state entry goes through the existing unlisted-run drop

Decision: `recordOwed` keeps deleting the `Runs` entry of every slug not among the tick's runs, so a done run's entry, `full_pass_owed` included, is removed by that same path.
Rationale: one drop path for runs that need nothing more, with no new state field.

## Constraints

- `PATTERN-dobby-starts-runs`: dobby starts a task only from an empty status; the change only frees slots.
- `PATTERN-test-speed`: the new case is a tier0 row or subtest in the existing dobbyengine tests, on the injected clock.
- `PATTERN-cwd-resolution`: dobbyengine stays told-geometry; no new import.

## Auto-mode assumptions

- Only state `done` is skipped; `blocked`, `paused` and `failed` runs still count against `max_runs`, as the package doc says of halted runs.
- The done state string is `done`, matching `shedengine.StateDone`, kept as a dobbyengine constant beside `runStateRunning`.

## Open risks

- A run reopened from `done` by a hand pass reappears as running on the next tick with a fresh state entry; that is correct, since it is live again.

## Acceptance criteria

- With done runs listed beside a released ready task and `max_runs` at or below the number of done runs, one tick starts the task.
- A done run gets no pass and has no `Runs` entry in the state record after the tick, even when a dobby start owed every run a full pass.
- A done run whose pass is still in flight keeps its slot until that pass exits.
- The board tidy still counts a done run's landing.
