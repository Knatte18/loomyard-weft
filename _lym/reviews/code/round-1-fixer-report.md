# Fixer-report, code review round 1

Gate: `lym gate test` tier 0 over `./internal/battencli`, `./internal/shedrun` and `./internal/beamcli` passed, and tier 1 over `./internal/battencli` passed, before the self-check.
Tier 0 over `./internal/battencli` and `./internal/beamcli` passed again after the help-text change.
Commits are grouped where findings share a file; nothing was pushed.

## Changes

- **F1, fixed with my own change** (commit `battencli: fix F1, F3, F6`).
  Proposed: keep the removal retryable either by restoring the run directory or by making `createSteps` call `ClearMotifRun` when the status file is absent.
  Made: `clearMotifRun` writes the status and child-recipe records back (`restoreMotifRunRecords` in `internal/battencli/cleanup.go`) whenever `removeRunRecords` fails with anything but a scratch-empty failure.
  Not made: the `createSteps` change.
  Evidence: every `TestCreate` row with a fresh slug lists its calls and none lists `clear` (`internal/beamcli/create_test.go`, rows around lines 255-300), so the change would also call the clearing on every first create, taking locks and a records HEAD read each time.
  Retry check: `TestClearMotifRun_FinishesTheRemovalOfARunWhoseRecordsCommitFailed` fails the commit, then re-runs and asserts the commit and the removal finish.
  The `ClearMotifRun` doc states the restore.
- **F2, fixed with my own change** (commit `battencli: fix F2`).
  Proposed: a row in `wire_test.go` beside the seam's refusal test.
  Made: assertions in the two existing Tier 1 integration steps in `internal/battencli/lifecycle_integration_test.go`, one asserting `shedrun.ReadChildRecipe` equals the child seed's recipe after Seed-Child, one asserting no record after a refused disagreeing seed.
  Evidence: the `wire_test.go` test runs with no task worktree, so the seam stops at the absent-pair refusal before the record write; only a real hub reaches it.
- **F3, fixed as proposed** (commit `battencli: fix F1, F3, F6`).
  Rows added to `TestClearMotifRun_RemovesOnlyADoneMotifRun`: records commit failure, push spawn failure, scratch-empty failure and a run the records HEAD does not track.
  `cleanupStubs` gained a `commitErr` field.
- **F4, fixed as proposed**, plus the self-check addition (commits `docs: fix F4` and `beamcli: fix F4 (self-check)`).
  `docs/overview.md` names the clearing in the beam entry and the batten cleanup paragraph.
  The `lym beam create` and `lym batten cleanup` help text in `internal/beamcli/cli.go` and `internal/battencli/cleanup.go` now name it too.
- **F6, fixed as proposed** (commit `battencli: fix F1, F3, F6`).
  The `clearMotifRun` doc states its locking on its own terms, and the `ReadChildRecipe` doc lost its history sentence.

`decision`-class finding F5 is left out of this report, as the fix step says.

## Self-check

- fix-code-correctness, LOW: `restoreMotifRunRecords` restores the status and child-recipe records but not the seed, notice-episode or settle records. Disputed, every fix worse (see Disputed).
- fix-affected-set, LOW: `internal/battencli/cancelcmd.go:294` also calls `removeRunRecords`. Deferred (see Deferred).
- fix-affected-set, LOW: the `lym beam create` help lacked the clearing step, and the `lym batten cleanup` help lacked the create clause. Fixed as proposed, in the last commit.
- fix-consistency: no findings.
- fix-proposal: no findings.

## Disputed

- **Self-check, fix-code-correctness LOW, every fix worse.**
  A full restore of the run directory means reading and rewriting every record before the removal, each step with its own failure modes, in a path that already failed.
  The retry needs none of it: `judgeRunDirectory` and `clearMotifRun` read only the status and the child-recipe record, and the retry's `removeRunRecords` commit takes the whole run directory as its pathspec, so it commits the deletions of the unrestored seed, notice-episode and settle records (`internal/battencli/cleanup.go`, `removeRunRecords`).

## Deferred

- **Self-check, fix-affected-set LOW: `lym batten cancel` has the same deletion-before-commit order at `internal/battencli/cancelcmd.go:294`.**
  It is outside the reviewed change and outside what F1 names, and whether a failed cancel removal should restore its records is a call for the run's parent.
