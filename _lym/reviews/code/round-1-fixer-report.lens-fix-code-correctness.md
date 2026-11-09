```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check commit 57fe9421 through the correctness lens, running each added or changed branch and test against a concrete input.

Findings:

1. **LOW**, `ABudgetSpentMidPackageListsTheRestOfThePackageAsNotJudged` (`redundancy_integration_test.go`).
   - The test can pass without reaching the branch it targets.
   - It asserts only that `TestDropsLate` is "not judged" for `reasonBudgetSpent`.
   - That outcome also comes from the budget check at the top of `judgePackage`, which fires when the step reaches `drops` after the 3s deadline.
   - The test pins the in-loop `stopReason` check only on a machine fast enough to reach `drops` in under 3s. Deleting that check passes under load.
   - It never fails wrongly. Asserting `TestDropsHangs` was "timed out after 5s" would prove the build ran first, but that assertion is flaky under load.

2. **LOW**, `TestRedundancyReportMatchesNothingWhenItJudgedNothing`, row `ARedGate`.
   - The row sets `RedGate: true` together with `Note: noteRedGate`.
   - The `report.Note != ""` half of the guard in `RedundancyReportMatches` returns false for this row on its own.
   - Deleting `|| report.RedGate` therefore fails nothing.
   - The row should set `RedGate: true` with an empty `Note` to pin that half.

Branches I traced:

- **`retries()`**: a report with a `reasonCancelled` test returns true. The new unit row `ATestACancelledCallLeftUnrun` fails without the added clause.
- **Stale hash (`judgeRuns`)**: `TestStaleOld` is excluded because `hashes["TestStaleOld"]` differs from the map's value. `TestStaleNew` then comes out unique. With the hash comparison deleted, `TestStaleOld` would be kept and `TestStaleNew` would be contained, so the row fails.
- **Lym build failure**:
  - The stand-in records the build, then fails.
  - `buildLym` caches the error, so `TestDropsTier1` and `TestDropsTier1Again` both get `reasonLymFailed` from one build.
  - The one-line builds-file assertion fails if the build were retried.
  - The cap of 9 keeps all four `drops` tests in range.
- **Corrupt map**: block index 9 against a table of 2 makes `checkMapBlocks` error. The alpha package line is unjudged with "its coverage map is unreadable", and the error line has the `coverage map of alpha: ` prefix.
- **Inherited slot**: with `inherited` ignored, `Acquire` would block on the held single slot until the 30s context ends, and the step would list an error. The test fails in that case.
- **Failed report write**: the `Mkdir` at the report path makes the write fail. Dropping the `return` after the Warn would print a summary to stderr, so the `stderr.Len() != 0` check catches it. The Warn itself is not asserted.
- **Non-test Go file change**: the `alpha/alpha.go` commit changes the file hashes, so `matches` returns false. The restore commit returns it to true, so the later test-edit `wantMatch(false)` is not masked.
- **`runTest` wiring**: the red case (exit 7) requires `RedGate` true and the same exit code. The green case requires `RedGate` false and exit 0. Both require the summary line on stdout and a report with the right gate key. Removing the `Contains` check or the `red` value fails a row.
- **Callers of the changed `writeGate` signature**: all three call sites were updated and the package vet is clean.

Not covered: `secondUnit` literals in `cli.go` and `rungate.go` remain unpinned, which the fixer presumably reports as partial for F3.

Tool calls used: none; the context already held everything needed.
