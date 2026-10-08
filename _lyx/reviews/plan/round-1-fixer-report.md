# Fixer report — plan, round 1

Every finding was validated against the code, git and the decision record, and all eight are fixed.
`lyx loom validate-plan` passes after the fixes, with only informational `create-new-unit` findings.

## Fixed

- **F1** (card 5): confirmed against git in a scratch repo: with `gc.pid` naming a live process, `git -c gc.auto=1 -c gc.autoDetach=false -c gc.worktreePruneExpire=never gc --auto --quiet` exits 0 and prints nothing.
  The review's proposed trigger, an unwritable `objects/pack`, also exits 0, so it was not used.
  `gc.bigPackThreshold=bogus` in the store's config makes `gc --auto` exit 128 on its config parse whatever the threshold, and only gc reads that key.
  The Intent now treats a non-zero exit as the failure and states that a held `gc.pid` is a silent skip; the fourth subtest writes `gc.bigPackThreshold=bogus` into one store after the pair is created and asserts the Remove succeeds with a logged warning.
- **F2** (card 2): each `parity_test.go` row builds its own repository and `Repo` and runs the freeze, CLI commit, repack and prune-packed sequence before its one read; the card says why a shared fixture would pass vacuously.
- **F3** (card 4): the constraint is now carried.
  New `gitexec.SetKillReporter` (Create) receives each kill's args, pid and group-kill error from `runCore`'s `proc.ConfigureGroupKill` callback; the default drops it.
  `cmd/lyx#main` (Edit) installs a reporter once that logs through `internal/logger`: info on a group kill, a warning when the group kill failed or on Windows.
  The Intent states the reporter's bound (it only observes a finished kill, set only by `main` and gitexec's non-parallel test), drops the false "every caller logs" claim, and the `doc.go` text and the integration test cover the reporter.
  It is not set from `cmd/lyx`'s in-process `run` seam, so parallel `cmd/lyx` tests never write it.
- **F4** (card 4): the existing `pull with phrase` row flips to true, `Operation too slow` joins the `phrases` list iterated over push and fetch, and the only new `cases` row is the `Timeout` one.
- **F5** (card 1): the fixture builds at least three old commits and deletes the oldest; the card explains that go-git's committer-time walker loads each returned commit's parents, so the two nearest old commits are read (confirmed in `commitIteratorByCTime.Next`).
- **F6** (card 4): the liveness check polls `proc.IsAlive` to a bounded deadline, as `TestRun_ExpiryKillsBackgroundedDescendant` does.
- **F7** (card 3): the `internal/verifyrun#run` target became `internal/verifyrun/run.go`, and the Intent updates its package doc's import sentence (line 6) to name `internal/proc`.
- **F8** (card 2): the Tests list updates `forcePackIndexFreeze`'s doc (`parity_test.go:342`) to name `readGoGit`.

## Disputed

None.

## Deferred

None.
