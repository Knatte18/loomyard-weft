---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/gatecli/redundancy.go:530
    summary: a test killed in flight by a cancelled call is recorded as a failed test, not as cancelled, so retries() keeps the report and the test is never rerun
    fix: In redundancyRun.runTest, right after runner.RunAlone returns and before the err and TimedOut switch, add a check that returns testcover.Run{} and reasonCancelled when s.ctx.Err() != nil. Then a test the cancel killed is listed as not judged for the cancelled call, and retries() reruns the step.
  - id: F2
    severity: LOW
    class: design
    location: internal/gatecli/redundancy_integration_test.go:351
    summary: no test makes a call cancelled mid-run produce reasonCancelled, so the stopReason context branch and F1's path are uncovered
    fix: Add a subtest (or a row in an existing subtest) to TestRedundancyStep that cancels the step's ctx once the first test has started, for example through a cancel the stand-in signals by creating a file the test polls for. It asserts that the tests not yet run are not judged with reasonCancelled, that the test in flight is too, and that report.retries() is true. Drop the retries-only row's reliance on a hand-made report as the sole cover of the cancelled reason.
  - id: F3
    severity: LOW
    class: design
    location: internal/gatecli/redundancy.go:577
    summary: the judged-name exclusion in judgeRuns (a judged test is never its own cover) is pinned by no test, because every judged test with a map entry also has a stale function hash
    fix: In TestRedundancyStep add a case where a judged test has a map entry whose FuncHash equals its current hash, such as a map built after the test was added on the branch, with a second judged test covering the same block. Assert the first is not listed among its own covering tests and the other is reported contained by the map-only cover or unique. Deleting the judged[mapped.Name] clause must then fail the test.
  - id: F4
    severity: LOW
    class: design
    location: internal/gatecli/redundancy_integration_test.go:418
    summary: no test asserts that the step holds a gate slot while it runs, so removing the Acquire in holdSlot fails nothing
    fix: Add a subtest that runs the step with a stand-in test that blocks, or a stand-in that records the slot directory's holder files, and asserts that a holder record with Site redundancySite exists for the worktree during the run and is gone after it. PATTERN-gate-slot requires a gate run to hold a slot, so this behavior needs coverage.
---

### [MEDIUM] A test killed in flight by a cancelled call is not listed as cancelled

**Location:** internal/gatecli/redundancy.go:530
**Issue:** `stopReason` lists tests the loop has not started as `reasonCancelled`, and `retries()` reruns the step on that reason, which settles focus question 1 for the unrun tests.
The test running when the cancel arrives is not covered.
`RunAlone` kills the binary's process group on the context's end.
Reading `internal/testcover/runner.go:96`, a killed binary yields an `ExitError`, no `DeadlineExceeded` and no test result line, so it returns `ActionFail` and a nil error.
`runTest` then returns that run with no reason, and `judgeRuns` records the test as not judged with the eligibility reason for a failed test.
No error line and no `reasonBudgetSpent` or `reasonCancelled` line goes into the report when the killed test is the last one to run.
`retries()` is then false, and the next call on the unchanged tree keeps the report, so the test is never judged.
I derived this from the code and did not run it.

### [LOW] No test produces reasonCancelled

**Location:** internal/gatecli/redundancy_integration_test.go:351
**Issue:** `reasonCancelled` appears in tests only inside the hand-built report of the `retries()` table.
No test cancels a step's context, so the `s.ctx.Err()` branch of `stopReason` and the in-flight case of F1 are unpinned.
The `retries()` row checks that a reason is honored, not that the step ever writes it.

### [LOW] The judged-name exclusion is not pinned

**Location:** internal/gatecli/redundancy.go:577
**Issue:** Focus question 2 is settled for the `FuncHash` comparison.
`TestStaleOld` is not in the diff, its map entry holds a different hash, and `TestStaleNew` covers the block it covers.
Deleting `hashes[mapped.Name] != mapped.FuncHash` keeps `TestStaleOld` as a cover and turns `TestStaleNew` from unique to contained, so `JudgesTheDiffsNewAndChangedTestsAgainstTheMaps` fails.
That is not a defect.
The `judged[mapped.Name]` half of the same condition is a different case.
The only judged test with a map entry is `TestBetaOne`, whose edit also changes its hash, so the hash half alone drops it and no test needs the name half.
The acceptance criterion that a judged test is never its own cover is then unpinned for a map entry that matches the current text.

### [LOW] The step's slot hold is not asserted

**Location:** internal/gatecli/redundancy_integration_test.go:418
**Issue:** `AnInheritedSlotIsHeldByTheCallerSoTheStepTakesNone` pins the inherited case.
Every other subtest runs the step with its own slot, and none of them checks that the slot is held during the run or released after it.
Removing the `Acquire` in `holdSlot` leaves all of them passing, though `PATTERN-gate-slot` requires the step to run tests only in held slots.

## Rejected

- Focus 3, the `runTest` wiring: settled, not a defect. `TestGateTest_Scenario` has a subtest in `test_integration_test.go` that runs `--gate reported` with `reports: [redundancy]` through the CLI, expects the tier's own exit code 0 and 7, and reads `redundancy.json` with the gate key and `RedGate`.
- Focus 4, the failed `lym` build and the corrupt map: settled, not a defect. `ABrokenLymBuildAndACorruptMapAreErrorLinesThatLeaveTheOtherTestsJudged` asserts one `lym build:` line, one `coverage map of alpha:` line, a single build for both tier1 tests and `TestBetaNewOne` still contained. The gate result is unchanged by construction, since `redundancyStep.run` returns nothing and the wiring subtest pins the exit code.
- The retry on an error-only report, the match rows, the spawn-scan drop, the mid-package budget and the failed report write each have a test in `redundancyreport_test.go` or `redundancy_integration_test.go`.
