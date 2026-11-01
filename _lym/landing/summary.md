# Shuttle Wait reads a strand its own Stop removed as the stop

A `Wait` on a run handle whose own `Stop` already removed the strand now ends `OutcomeDied`, and `finalize` records the run as stopped unless the output files show it done.
Before, the liveness check in `internal/shuttleengine` found the strand missing from reed's status and returned the `errStrandNotTracked` mechanism failure.
That failure reads as "reed's strand table no longer holds this run's strand", which is how the Parts-Plan-Write chair wait failed in the `plan-split-bench` study.
When the handle itself removed the strand, the strand's absence is the stop, not reed's bookkeeping going away.

## Design decisions

- `checkLivenessTick` asks the new `Run.markedStopped` (read under the record lock) only after the satisfied-files check, so finished work stays `OutcomeDone` behind a stopped handle.
- The general rule is unchanged: a strand reed stops tracking, with no `Stop` from this handle, is still a mechanism failure and never `died`.
  `docs/overview.md`, the `doc.go` and `wait.go` package comments and the `OutcomeDied` comment state the new exception beside that rule.
- `completionsignal_enforcement_test.go` pins the `checkLivenessTick [OutcomeDied]` return count at 3, each guarded by `allOutputFilesExist`.
- `TestStop_WaitThatSeesTheStoppedStrandGoEndsDied` covers both cases: files missing ends `died` and records `stopped`, files present ends `done`.

## Scope limit

This change covers the case where the handle's own `Stop` removed the strand.
It does not find what else could remove a live chair strand from reed's table while its part writers run, which the board entry also asks for.
The report's cause stays open: no `Stop`, reed remove or down was seen in the `psb-rv-split` worktree, and a `fabric add --local` pair or a hand-seeded run may be a factor.

## To check by hand

- The `stale-never-died-claims` finding: look for remaining comments or docs that say a strand reed no longer tracks is never `died`, without the stop-marked exception.
  The ones in `engine.go`, `wait.go`, `doc.go` and `docs/overview.md` are updated; search the rest of `internal/shuttleengine` and `docs/` for others.
- Run `lym gate test ./internal/shuttleengine` (not run for this description).
- Re-run the `run-verbs-shared` split arm of the `plan-split-bench` study to see whether the chair failure still occurs.
