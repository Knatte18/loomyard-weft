---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/gatecli/redundancyreport.go:165
    origin: lens:code-correctness
    summary: a step cancelled mid-run lists its unrun tests as not judged for the cancelled call, yet retries() ignores that reason, so a repeat call on the unchanged tree keeps the report and never judges them
    fix: in retries() in redundancyreport.go, also return true when a test line has Reason reasonCancelled (add it beside the reasonBudgetSpent check), so a later call reruns what the cancelled call left unrun; add a case to the retry-on-repeat subtest of TestRedundancyStep
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/redundancy_integration_test.go:208
    origin: lens:code-tests
    summary: the acceptance criterion that a map test whose function text changed is not counted as a cover has no test that fails if the hash comparison is deleted
    fix: in TestRedundancyStep, add a map test for a package whose map holds an entry with a stale FuncHash for a test that is not in the diff and covers the block a new test covers, and expect the new test unique; the existing edited TestBetaOne row is excluded by the judged-name check before the hash check runs, so it does not pin it
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/gatecli/redundancy_integration_test.go:197
    origin: lens:code-tests
    summary: no test reaches the step through runTest, so the wiring that runs it only for a key listing reports and never changes the exit code, and the secondUnit values set in cli.go and rungate.go, are unpinned
    fix: add a tier1 case that runs runTest or RunGate with a hub gate key listing reports [redundancy] over a red tier and a green tier, and asserts the exit code equals the tiers' own and that redundancy.json is written; this also pins that a zero secondUnit would not time out every test
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/gatecli/redundancy.go:486
    origin: lens:code-tests
    summary: a failed lym build and a corrupt or unreadable coverage map are listed by the decision record as report lines, but no test row reaches buildLym or checkMapBlocks
    fix: in redundancy_integration_test.go, declare tier1 in a case of the step test and use the testmap lym-build stand-in (testMapLymBuildsEnv) to fail the lym build, asserting one error line, the tier1 tests not judged with reasonLymFailed and no second build; add a map whose test names a block index outside its table and assert the package line and error line
  - id: F5
    severity: LOW
    class: design
    location: internal/gatecli/redundancy.go:241
    origin: lens:code-tests
    summary: the spawn-scan drops (out of process, unresolved) and the failed-scan drop have no test row, only the keep marker is covered
    fix: add a test to the drops package of the step test that references os.Executable or exec.Command with go, and expect it not judged with testcover.ReasonOutOfProcess
  - id: F6
    severity: LOW
    class: design
    location: internal/gatecli/redundancy.go:462
    origin: lens:code-tests
    summary: a budget spent after a successful build, with the remaining tests of the package listed as not judged for it, has no test; only a budget spent before any build is covered
    fix: add a case to the spent-budget subtest where the first test of a package runs and the rest hit the deadline, asserting the rest are not judged with reasonBudgetSpent
  - id: F7
    severity: LOW
    class: design
    location: internal/gatecli/redundancy.go:160
    origin: lens:code-tests
    summary: the retry rules of a report holding only an error line, and RedundancyReportMatches returning false for a changed non-test Go file of a judged package or a noted or red report, have no rows
    fix: add a repeat case where the report holds only an error line and the step runs again, and add rows to the RedundancyReportMatches subtest for a changed non-test Go file and for a report with a note or red gate
  - id: F8
    severity: LOW
    class: design
    location: internal/gatecli/redundancy.go:117
    origin: lens:code-tests
    summary: the failed report write logged at Warn with the result standing, and the slot hold and release including the inherited-slot path, have no test
    fix: add a case that makes the report path unwritable and asserts the step returns, the Warn is logged and no report appears; add a case with inherited set that asserts no lease is taken
---

### [MEDIUM] A cancelled step is kept as complete on the next call

**Location:** `internal/gatecli/redundancyreport.go:165`
**Issue:** When the call is cancelled during the step, `stopReason` returns `reasonCancelled` and each unrun test is listed as not judged with it, with no error line. `retries()` checks only `reasonBudgetSpent`. The next call on an unchanged tree matches the report and skips the step, so those tests are never judged. D7 says a later call retries what failed or ran short.

### [MEDIUM] No test pins the stale-hash cover exclusion

**Location:** `internal/gatecli/redundancy_integration_test.go:208`
**Issue:** An acceptance criterion says a covering map test whose function text changed since the map is not counted. In the fixture the only changed map test, `TestBetaOne`, is also a judged test, so the judged-name check excludes it first. Removing the `hashes[mapped.Name] != mapped.FuncHash` comparison in `judgeRuns` fails no test.

### [MEDIUM] The step's wiring in runTest is untested

**Location:** `internal/gatecli/redundancy_integration_test.go:197`
**Issue:** Every test calls `step.run` directly. Nothing runs a gate key with `reports` through `runTest`, so the `Contains` check, the `red := code != 0` value, the guarantee that the exit code stays as the tiers decided, and the `secondUnit` values in `cli.go` and `rungate.go` are unpinned. A zero `secondUnit` would time out every test with all tests green.

### [MEDIUM] A failed lym build and a corrupt map have no test

**Location:** `internal/gatecli/redundancy.go:486`
**Issue:** The decision record names both as report lines that leave the gate's result alone. The fixture declares no `tier1`, so `buildLym` never runs, and no map has an out-of-range block index, so `checkMapBlocks` never fails.

### [LOW] Spawn-scan drops have no test

**Location:** `internal/gatecli/redundancy.go:241`
**Issue:** D7 step 2 drops tests the spawn scan marks as possibly running module code in another process. Only the keep marker and the tier drops are pinned.

### [LOW] A budget spent mid-package has no test

**Location:** `internal/gatecli/redundancy.go:462`
**Issue:** The spent-budget subtest covers a budget spent before any build. The branch where a test runs and the next finds the deadline passed is unpinned.

### [LOW] Retry on an error line and the match's false cases have no rows

**Location:** `internal/gatecli/redundancy.go:160`
**Issue:** The criterion lists an error line among the triggers that rerun the step, but only the unmapped-package and spent-budget triggers have rows. `RedundancyReportMatches` is tested for a taken test changing, not for a judged package's non-test Go file changing or for a noted or red report.

### [LOW] Failed report write and slot hold have no test

**Location:** `internal/gatecli/redundancy.go:117`
**Issue:** The acceptance criterion that a failed write is logged and leaves the result has no case. The slot acquire, release in `close`, the wait record and the inherited-slot path that takes none are also unpinned.

## Rejected

- Skip-on-match ignores `redundancy_max_tests` and a rebuilt map (code-correctness, LOW): D7 defines the match as D8's, over test and file hashes, and the diff does exactly that.
- `Gate().Reports` clone untested (code-tests): the step reads `cfg.Gates` directly, and the clone is a one-line copy beside the lints clone.
- The not-in-a-hub note, `isModulePackage` pass-overs and `redundancyModule` errors have no row (code-tests NIT): the cases are not in the acceptance criteria and each is a short branch.
- The `secondUnit` wiring as its own NIT (code-tests): merged into the runTest wiring finding.
- No finding from the code-affected-set lens: it found every key, doc and test of the changed symbols updated, and no declared PATTERN change or `Pattern-Departure` trailer is owed.
