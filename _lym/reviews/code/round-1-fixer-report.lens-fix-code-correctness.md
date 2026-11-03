```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fixer's code changes (`clearMotifRun` restore, its tests, the lifecycle assertions) through the fix-code-correctness lens.

Findings:

1. **LOW** — `restoreMotifRunRecords` (`internal/battencli/cleanup.go`) restores only the status and child-recipe records, not the seed, notice-episode or settle records that `removeRunRecords` already deleted.
   - Evidence: after a failed commit, the restored run directory holds a done status with no seed, while the records HEAD still tracks the seed.
   - The re-run path still works. `RunStatus` finds the status as done, `ClearMotifRun` judges it, `removeRunRecords` removes the directory, and the commit then includes the seed deletion. `cleanup`'s `judgeRunDirectory` also reads the status as done.
   - A reader that expects a seed beside a done status, such as `lym batten status`, would see a run directory with no seed until the retry. Nothing in the diff or tests reaches that case.
   - I am reporting this as a narrow edge, not a defect.

Traces that held, one concrete input each:

- **Commit failure (`removalCommitError`):** `err` is not a `scratchEmptyError`, so `errors.Join(err, restore)` runs. With the restore succeeding, `errors.Join(err, nil)` keeps `err`'s text, and `errors.As` and `errors.Is` still see `removalCommitError` and the commit failure. `createSteps` wraps it with `%w`, so its message is unchanged.
- **Local commit made but sync failed:** the restored status is then untracked at HEAD. The re-run finds `runRecordsTracked` false, makes no commit and succeeds.
- **`RemoveAll` partial failure and `Committed` read error:** both happen after the directory delete, so the restore covers them too.
- **Callers:** the `ClearMotifRun` signature is unchanged and `internal/beamcli/deps.go` is unaffected. `createSteps` is untouched, and the `beamcli` create tests fake `ClearMotifRun`.

Each added test fails with its defect in place:

- **`RecordsCommitFailureReturnsTheErrorAndKeepsTheRun` row:** without the restore the run directory is gone, so `gone != wantCleared` fails.
- **`TestClearMotifRun_FinishesTheRemovalOfARunWhoseRecordsCommitFailed`:** without the restore the second call finds no status file and returns `cleared` false.
- **`ScratchEmptyFailureStillClearsTheRun` row:** if a scratch failure were returned as an error, the `errors.Is(err, nil)` check fails.
- **`PushSpawnFailureStillClearsTheRun` row:** it expects `spawns == 1` and `cleared` true.
- **`DoneMotifRunTheRecordsHeadDoesNotTrackIsRemovedWithNoCommit` row:** it expects zero commits and zero spawns, which holds because `runRecordsTracked` is false.
- **Lifecycle step 1 assertion:** it fails if the `WriteChildRecipe` line is removed, or if the recipe written differs from the seed's recipe.
- **Lifecycle step 2 assertion:** it fails if the record is written ahead of the seed write, since the disagreeing seed is refused after that point.

Nothing is `BLOCKING` or `MEDIUM`.

Further tool calls: none. My context already held every changed file and the review.
