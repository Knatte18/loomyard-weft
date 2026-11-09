## Changes

All eight findings were fixed in commit `57fe9421`.
The self-check findings were fixed in the commit after it.
`lym gate test ./internal/gatecli --tier 0 --tier 1` is green after both.
Each finding was fixed as proposed unless marked below.

- F1: `retries()` in `redundancyreport.go` also returns true for a test with reason `reasonCancelled`.
  The new `TestRedundancyReportRetries` in `redundancyreport_test.go` holds the cancelled row, and `doc.go` names the cancelled case.
- F2: a `stale` package whose map holds `TestStaleOld` with a stale `FuncHash` and covers the block `TestStaleNew` covers.
  The step test expects `TestStaleNew` unique.
  Raised `redundancy_max_tests` from 7 to 9 for the added tests.
- F3: a scenario subtest in `test_integration_test.go` runs a gate key with `reports: [redundancy]` over a green and a red tier through `runTest`.
  It asserts the exit code is the tier's own, `redundancy.json` is written with the gate key and the `RedGate` value, and the summary line appears.
  The `secondUnit` literals are not pinned; see Deferred.
- F4: the stand-in `go build` of `lym` fails under the new `testMapFailLymBuildEnv`.
  A step subtest declares a `prebuilt_lym` tier1, fails the build, and asserts one `lym build` error line, both tier1 tests not judged with `reasonLymFailed`, and one build.
  The same subtest corrupts alpha's map with block index 9 and asserts the `coverage map of alpha` error line, the package line and the not-judged tests.
- F5: `TestDropsSpawns`, in a `//go:build tier1` file that calls `os.Executable` and has no `Require`, is expected not judged with `testcover.ReasonOutOfProcess`.
  The failed-scan drop has no row; see Deferred.
- F6: `TestDropsLate` follows `TestDropsHangs` in `drops`.
  A subtest with a 3 second budget and a 5 second test timeout asserts `TestDropsLate` is not judged with `reasonBudgetSpent`.
- F7: unit rows for an error-only report, a note, a red gate and a retry-free report.
  The step test adds a committed change to `alpha/alpha.go` that must break the match, and its restore, which must match again.
- F8: a subtest makes the report path a directory and asserts the step returns, the path is unchanged and nothing is printed.
  A second subtest holds the only slot with `inherited` set and asserts the step judges without waiting.
  The Warn line is not asserted; the logger is process-global and the subtests share a parallel test.

## Disputed

- F6 self-check finding on `ABudgetSpentMidPackageListsTheRestOfThePackageAsNotJudged` (correctness lens, LOW), ground `every fix worse`:
  the lens notes the test reaches the in-loop `stopReason` check only on a machine fast enough to reach `drops` before the budget.
  Asserting `TestDropsHangs` timed out would prove the build ran, but it fails when the machine is slow, and the lens agrees.
  The test never fails wrongly, so it stays.

## Deferred

- F3, the `secondUnit` literals in `cli.go` and `rungate.go`.
  A test that pins them needs a green call that runs a real redundancy test, which needs a Go module hub with maps and a stand-in `go`.
  The gate helper process sets `helperEnv` to the stand-in `tier` role, which clashes with the testmap `go` role, so a new helper role would be needed.
  I left this to the operator.
- F5, the failed-scan drop.
  The fix key asks only for the spawn row.
  A failed scan adds an error line to every report of the hub's fixture, so it needs its own package and run.

## Rulings

None.

## Self-check

- fix-code-correctness, LOW: mid-package test can pass without reaching the in-loop check. Disputed, `every fix worse`.
- fix-code-correctness, LOW: the `ARedGate` row set a note beside `RedGate`. Fixed; the row sets `RedGate` alone.
- fix-consistency, LOW: `doc.go` names only the budget case. Fixed.
- fix-proposal, LOW: the `cmd` cleanup could delete tracked files. Fixed; it removes `cmd/lym`, then `cmd` only if empty.
- fix-proposal, LOW: a subtest name claimed a log it does not assert. Fixed; renamed.
- fix-proposal, LOW: the mid-package subtest relied on the budget an earlier subtest left. Fixed; it writes the gate.
- fix-proposal, LOW: the `secondUnit` literals are unpinned. Deferred, as above.
- fix-affected-set: no findings.
