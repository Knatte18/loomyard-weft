```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `runTest` loop, `haltExitCode`, `printFailedRuns`, `runEnd` and `FailedTier` in the merged tree.

**Cases run**
- `runTest` loop, all runs pass: `failures` is empty, `code` is 0, `printFailedRuns` prints nothing, exit 0. Correct.
- `runTest` loop, tier0 exits 3 and tier1 exits 5: both runs execute, `failures` holds both, `code` and `firstFailed` stay at tier0's 3. The closing line names both and the exit is 3. Correct.
- `runTest` loop, guard run after a failed tier: it is reached, since the loop is not broken on failure. Correct.
- `runTest` loop, refusal in a later run (`runGuard` setup error, or `report` with `result.refusal`): `ran` is false and `halt` is true, so the loop returns through `haltExitCode` and exits 1. This is the documented "refusal exits 1". Correct.
- `runTest` loop, busy hub with no earlier failure: exits 75. Correct.
- `runTest` loop, busy hub after an earlier failure of 3: `haltExitCode` returns 3, because 75 would read as no failure. Correct.
- `runTest` loop, cancelled context during acquire (`acquireSlot` returns 1, `ran` false): exits 1, or 128 plus the signal when a signal was caught. Correct.
- `runTest` loop, halting run that ran and exited 0 with no earlier failure: prints the "cancelled" line and exits 1, so a halt is never green. Correct.
- `runTest` loop, halting run that ran and failed with code 7: it joins `failures`, and `code` is set to 7 if it is the first failure. `haltExitCode` returns 7. Correct.
- `haltExitCode`: for `end.ran && firstFailureCode == 0` it returns 1. For `end.ran` with a prior failure it returns that failure's code. For `end.code == SlotBusyExit && firstFailureCode != 0` it returns the failure's code, and otherwise `end.code`. All branches correct. The switch precedence (`&&` binds tighter than `||`) reads as intended.
- `printFailedRuns`: empty list prints nothing. `halted` adds the "later runs did not run" suffix. Correct.
- `runGreenAware`: a failed or halted run is never passed to `recordPass`, since it requires `code == 0` and `!halt`. So a failed tier never carries. Correct.
- `reportCarried`: it returns `ran: true` with code 0 and does not fail the call. Correct.
- Timed-out tier or empty-tier failure: `report` returns `ran: true` with a nonzero code, so it counts as failed and does not halt. Correct.
- `FailedTier`: summary lines are printed in plan order, so the first non-zero line is the first failing run. The closing "failed runs:" line does not match `tierSummaryPattern`, so it is ignored. Correct.

**Findings**
- LOW, `design`, `internal/gatecli/run.go:166-173`, `internal/gateslot/runlog.go:190`: when a refusal or a non-busy acquire failure halts the call after an earlier failed run, the exit code is the halting run's 1, not the first failure's code. `FailedTier` still names the first failing tier from the summary lines. The two disagree in that one case: the exit is 1 while `FailedTier` names the earlier tier, whose code was different. This follows the decision that a refusal ends the call "as before", and the closing line is printed, so it is arguably intended. I found no defect.

No other defects.

Tool calls:
1. Read `tierSummaryPattern` and `TierSummaryLine` in `internal/gateslot/runlog.go`, to check that the closing line cannot match `FailedTier`'s pattern.
