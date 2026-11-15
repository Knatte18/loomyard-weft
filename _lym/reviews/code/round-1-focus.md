---
round: 1
focus:
  - "Scope against the board entry in `_lym/discussion/decision-record.md` `## Goal`: it asks that a finished run's directory and its task branch be removed after Land and Worktree-Teardown. Settle whether the diff removes the task branch, or only the records and scratch through `battencli.ClearDoneRun`, and whether the entry's expectation is met either way."
  - "`internal/dobbyengine/donerun.go` `clearDoneRuns` gating: `d.landed` is set only by `countLandings` in `internal/dobbyengine/tidy.go`. Settle whether it can be stale or nil on a tick where the landing count did not run (nil `Deps.Landings`, an anchor change, a tidy agent in flight), so that a run is cleared before its landing is counted or its board-tidy count is lost."
  - "`clearDoneRuns` against `current.LandNotices` and `CountedLandings`: settle whether a done run is ever cleared while its landing notice is undecided, and whether deleting its `CountedLandings` entry can make the same landing count twice if the board or prime still reports it."
  - "`internal/battencli/cleanup.go` `clearDoneRun` with `admitsAnyRun`: settle whether it removes only a run whose status reads done under both locks, and whether a done run with no child-recipe record (`recipe == \"\"`) takes the same removal, commit message and `restoreDoneRunRecords` path correctly when the removal fails."
  - "`ClearDoneRun` called from `internal/beamcli/dobbydeps.go` with the prime location: settle whether `ResolveBattenPrime` is right for every dobby that fills `Deps.ClearDoneRun`, including a non-prime parent worktree's dobby, and that no other `Deps` construction is left with a nil seam it needed."
  - "Comment and doc compliance in the diff against the repository's constraints document: the `internal/dobbyengine/doc.go` done-run clear section, the `cleanup.go` and `cli.go` help text, and the `Deps.ClearDoneRun` comment. Settle whether each claim matches the code, e.g. the stated tick placement, the log line and the 'tried again at the next tick' behavior."
  - "Test economy and coverage in `internal/dobbyengine/donerun_test.go` and `internal/battencli/cleanup_test.go`: the redundancy report lists all three new tests as unique, so settle only behaviors the diff adds with no coverage (the in-flight, driver-lock, landing-notice-memory and uncounted-landing skips, the failed-clear episode logging, the `CountedLandings` update failure)."
---

The decision record holds only the imported board entry, so the measuring stick is that entry's `## Goal` text.
The commit message claims records and scratch removal only, so the task-branch half of the entry is the first scope question.
