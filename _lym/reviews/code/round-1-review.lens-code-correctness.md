```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. Each function or branch the diff changes is listed with the case I ran.

**Per function and case**

1. `goDriver.run`, the yield early-return (`internal/sheddriver/drive.go:204-210`).
   - Input: a step prints a yield envelope (OK, running, outcome `yield`).
   - Result: it returns before `SaveEnvelope` and before the `arbiterAttemptRefused` check. The `resumed` reset and `d.rules`/`d.attempts` reset are skipped too. `d.exit` is `ExitYield`, as before. Correct.
   - The `resumed=true` path (`resumeOpenAttempt`) never reaches the early-return, because it sits inside `if !resumed`. A resumed yield still takes the old path through `settleStop`, so the early-return's no-envelope guarantee does not cover it.
   - I did not find that `resumeOpenAttempt` can produce a yield stop, so there is no finding.
   - Gap: before the diff, a yield carrying `arbiter_attempt` (a step refusing an arbiter attempt while yielding) was saved and then ran through `arbiterAttemptRefused`. Only a non-yield stop gets that check now, which is consistent with "a yield decides nothing". Not a defect.

2. `commitRunRecords` (`internal/battencli/commitstatus.go`), six cases.
   - Merge-state read error: wrapped and returned. Correct.
   - Merge active: logs and returns nil. Correct.
   - Commit error: returned, and `Spawn` is not called. Correct.
   - `committed=false`: returns nil (`err` is nil there), and `Spawn` is not called. Correct.
   - Spawn error: only warned, returns nil. Correct.
   - Commit succeeds: spawns the push. Correct.

3. The Park seam (`internal/battencli/drive.go:276`).
   - Input: `commitRunRecords` fails.
   - Result: it warns and the park still proceeds. Correct.
   - Order: `handBackRun` writes the report (`drivehandback.go:161`) and only then calls `Park` (`:167`), so the commit includes the stop report. Correct.
   - The `DoneFrictionFailed` hand-back returns before `Park`. Its report is written after `finishDone`'s `CommitRecords`, so it is not committed until the next commit of the run's records. This is minor and consistent with the "next commit carries them" design. It is not a defect, and the run is done in any case.

4. The `CommitRecords` seam (`internal/battencli/drive.go:327`) via `finishDone` (`drive.go:832-838`).
   - Input: `commitRunRecords` fails.
   - Result: the error is returned and the driver fails, so the run is not parked and `Friction: failed` is not handed back. This is the existing contract for `CommitRecords` errors, and it is deliberately stricter than Park.
   - A transient git failure at the done stop therefore fails the driver, where the same failure at a hand-back only warns. The asymmetry is by design and not a defect.
   - Timing: the done step's envelope is saved at `drive.go:211` before `settleStop` and `finishDone`, so the commit includes it. Correct.

5. `battenRunCommitPaths` (`internal/battencli/commitstatus.go:85`).
   - Input: no driver directory holds a file.
   - Result: `DriverRecordsRel` returns a nil slice, `append(paths, nil...)` leaves `paths` unchanged, and the status file is still staged. Correct.
   - With files present, the order is drive-reports, arbiter, tackler, matching the `want` list in the test.

6. `runRecordsDeps` (`internal/battencli/drive.go:334-340`).
   - Input: `recordsDeps` is nil.
   - Result: it passes `c.slug` as `runID`. The existing `readopt.go:89` does the same, and `driverDepsFor` uses `ResolveRunID(location, slug)`, which only differs for the literal `self` run id. For a slug that is not `self` it is identical. No finding.

7. `DriverRecordsRel` and `dirHoldsFile` (`internal/shedrun/paths.go`).
   - Case 1: a missing directory. `WalkDir` calls the callback with an error, it is returned, and `found` stays false. Correct.
   - Case 2: an empty directory. No file is found, so it is omitted. Correct.
   - Case 3: an unreadable directory, or a file inside it that cannot be listed. The walk error is swallowed, so `found` is false when no file was seen before the error. The directory then drops out of the pathspec and its files stay untracked, with only a debug-level trace, if any.
     - The old inline `holdsFile` (`internal/loomcli/wiring.go:172`) swallowed the same errors, so this is not a regression.
     - It is a silent drop for an unreadable `_lym` directory. LOW at most.
   - Case 4: `dir` is a regular file. The old `holdsFile` had an `os.Stat` `IsDir()` guard and returned false for it. `WalkDir` on a regular file calls the callback once with `d` being the file, so `!d.IsDir()` is true and `dirHoldsFile` returns true.
     - Result: `DriverRecordsRel` would stage a path that is a file. The old code omitted it.
     - It only matters if one of the three driver paths is a plain file. `start_driver_test.go:414` deliberately plants a plain file at a drive-reports path to test report-write failure.
     - A pathspec naming a file is not an invalid pathspec, because git stages it. There is no commit failure. This is a behaviour difference from the three inlined checks, but harmless in practice.
     - I list it as LOW, `consistency`.
   - Emptiness rule and order match the old three checks (drive-reports, arbiter, tackler).

8. `statusCommitPathspec` (`internal/loomcli/wiring.go:102-115`).
   - The order changed. The reviews and durable directories come first, then the driver records, then the settle records. This is the same relative order as before. Correct.
   - Its doc comment now reads "then `DriverRecordsRel` ...". It matches the code.

**Findings**

- LOW / consistency, `internal/shedrun/paths.go` `dirHoldsFile`: the `os.Stat(...).IsDir()` guard that `holdsFile` in `internal/loomcli/wiring.go:172-176` carries was dropped. A driver path that is a regular file now counts as holding a file and is staged. The three checks `DriverRecordsRel` replaced had returned false for it. Evidence: `wiring.go:172-176` against `paths.go` `dirHoldsFile`.
- LOW / design, `internal/shedrun/paths.go` `dirHoldsFile`: a walk error on an unreadable directory is swallowed, so the directory is silently left out and its files stay untracked, with no log line. This is unchanged from the old inline check. Evidence: `paths.go` `_ = filepath.WalkDir`.

No tool call failed or was out of scope.

**Tool calls**
1. Read `internal/sheddriver/drive.go:825-864`: `finishDone`, which was not in my context.
2. Read `internal/battencli/commitstatus.go:108-167`: `battenCommitStatusDeps`, to confirm the commit scope.
3. Grep for `Park(` in sheddriver: to find where `Park` is called.
4. Grep for `battenCommitStatusDeps(` and `ResolveRunID`: to check the slug-versus-runID argument.
5. Read `internal/sheddriver/drivehandback.go:100-180`: to confirm the report is written before `Park`.
