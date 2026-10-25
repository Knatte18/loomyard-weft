# Dobby counts no done run against max_runs and keeps no state for it

A beam holding many finished batten runs no longer stalls its released, ready tasks.
Before, `drivePasses` in `internal/dobbyengine` took every run the listing returned, so each run in state `done` counted as in flight under `max_runs` and kept a `full_pass_owed` entry in dobby's `state.json`.
In the observed case 74 done records held the prime's dobby at `max_runs` 3, and `umbrella-anchor` never started until `lyx batten cleanup --apply` removed them.

Closes #872

## What changes

- A run whose batten state is `done` takes no slot under `max_runs` and is planned no pass.
- Its entry leaves the state record on the tick that drops it, `full_pass_owed` included, so `state.json` holds only live runs.
- `doc.go` states the rule in its passes and starts bullets.
- `starts_test.go` pins the start: done records beside a released ready task, with `max_runs` at or below their number, and the tick starts the task.
  `passes_test.go` is adjusted to match.

## Design decisions

- The filter sits in the engine, not in the beamcli run listing.
  The engine owns what counts as in flight and what is owed, and its fake-seam tests cover it without real status files.
  The listing also feeds the board tidy's landing count, which must keep reading done runs' history, so it is unchanged, as is `lyx batten cleanup`.
- `drivePasses` drops done runs from the tick's list before `recordOwed` runs, so the existing path that deletes the `Runs` entry of any unlisted slug removes the state entry.
  No new state field.
- A done run whose pass this dobby spawned and has not yet exited stays in the list.
  It keeps its slot and its entry until the exit is read, and the next tick drops both, so a start never takes the slot of a process still running.
- Only `done` is skipped, as a `runStateDone` constant beside `runStateRunning`.
  Blocked, paused and failed runs still count against `max_runs`, as the package doc already says of halted runs.
- A run reopened from `done` by a hand pass shows as running on the next tick and gets a fresh state entry.

## To check by hand

Not run: dobby against a real prime with accumulated done records.
