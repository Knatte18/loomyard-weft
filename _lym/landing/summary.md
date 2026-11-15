# dobby: clear a finished run's records and task branch

A done batten run's record stayed in the prime after Land and Worktree-Teardown.
A second task filed under the same slug, such as a second `lym beam merge-parent` merge-in, was then refused by batten.
Its start found the done record and a leftover task branch.

The parent worktree's dobby now removes a finished run itself.
The slug is free again without `lym batten cleanup --apply`.

Closes #128

## What changes

- `internal/dobbyengine/donerun.go` adds the done-run clear, `clearDoneRuns`.
  It runs each scheduling tick, after the landing count and the landing notices.
  Through the new `Deps.ClearDoneRun` seam, it removes the records, scratch and task branch of each listed run whose batten status reads done.
- A run is cleared only when all of these hold:
  - no pass of it is in flight, and its driver lock is free;
  - the landing-notice memory holds nothing of it;
  - a landing it holds is counted.

  So neither its landing notice nor its board-tidy count is lost.
- A cleared run logs `run <slug>: done run cleared` and leaves the counted landings.
  A later run under the slug then counts its own landing.
  Each tick also drops the counted landings of a slug that is neither landed nor listed.
- `internal/battencli/cleanup.go` adds `ClearDoneRun`, wired in `internal/beamcli/dobbydeps.go`.
  It generalises the motif-run clear into `clearDoneRun`, which takes an admit function and an optional branch discard.
  `ClearMotifRun` keeps its behaviour: it admits only motif and arras runs and discards no branch.
  `ClearDoneRun` admits any recipe, including a run with no child-recipe record.
- `ClearDoneRun` archives the task branch under a tag and deletes it locally and on origin, when either holds it.
  The discard runs before any record is removed, so a failure leaves the run readable and the next tick retries.
  It reads origin again afterwards, because the origin half can be handed to another process and fail unreported.
- The `lym batten cleanup` help, `lym batten run` help, `lym beam create` help, `docs/overview.md` and the `dobbyengine` and `beamcli` package docs now say that the dobby removes a done run.
  A done record still refuses a start until the dobby has removed it.

## Design decisions

- The fix sits in dobby's teardown, not in merge-parent's slug choice, following the operator's comment on the issue.
  Every recipe benefits, and merge-parent keeps its slug.
- The clear waits for the landing count and notice, so removing the record cannot drop a landing the board tidy still has to count.
- A failed clear is logged once per episode and never fails the tick.
  The run is finished either way, so the next tick tries again.
- Bound: while a board-tidy agent is in flight nothing is counted, so a run that landed meanwhile stays until the agent is settled.

## To check by hand

Nothing was run by hand, and no regression check ran for this task.

- `docs-describe-old-done-run-refusal`: confirm that no doc or help text still says a done run is refused until `lym batten cleanup --apply` is run.
  The edits are in `docs/overview.md`, `internal/battencli/cli.go`, `internal/battencli/cleanup.go`, `internal/beamcli/cli.go`, `internal/beamcli/create.go` and `internal/beamcli/doc.go`.
- `done-run-clear-leaves-task-branch`: confirm that the clear in `internal/battencli/cleanup.go` leaves no task branch, locally or on origin, once the run's records are gone.
  Run a real merge-in twice under one slug and check that the second starts.
