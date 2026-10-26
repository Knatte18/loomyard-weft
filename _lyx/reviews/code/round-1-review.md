---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/planparser/validate.go:checkRenameLeavesTest
    origin: lens:code-correctness
    summary: rename-leaves-test misses a test in the external test package, whose pseudo-unit is <dir>_test and never equals the renamed symbol's unit
    fix: In checkRenameLeavesTest, replace the test.Unit != old.Unit comparison with one that also accepts the package's other Go package, using GoPackageCounterpart or the trimmed-suffix helper in internal/planparser/targetdirs.go (a unit equals old.Unit, or old.Unit plus "_test"). Add a row to TestValidate_RenameLeavesTest where the renamed symbol is in unit u and the test ref is in u_test, expecting one finding.
  - id: F2
    severity: LOW
    class: design
    location: internal/websterengine/recordbatch_git_test.go:TestRecordBatch_PostBatchChecks
    origin: lens:code-tests
    summary: no test pins that a FAILED report is never handed back, so dropping the report.Status == ReportStatusOK condition at recordbatch.go:570 breaks nothing
    fix: Add a subtest to TestRecordBatch_PostBatchChecks that records a FAILED fork report whose batch removed a symbol a later card targets, and asserts the call returns no LaterTargetRemovedError and the flag stays unset.
  - id: F3
    severity: LOW
    class: design
    location: internal/websterengine/recordbatch_git_test.go:383
    origin: lens:code-tests
    summary: the acceptance criterion that an exact quarry rename draws no refusal has no engine-level test, since the mixed rename/delete subtest sets LaterTargetsReturned before it runs
    fix: In the subtest at recordbatch_git_test.go:383, leave LaterTargetsReturned unset and assert the first record-batch hands back only the plain deletion and not the exact-renamed symbol, or add a separate unflagged row for an exact rename.
  - id: F4
    severity: NIT
    class: design
    location: internal/planparser/validate_test.go:TestValidate_RenameLeavesTest
    origin: lens:code-tests
    summary: the decision record says rename-leaves-test is silent under none as well as csharp, but only the csharp row exists
    fix: Add a row to TestValidate_RenameLeavesTest with the plan language none, same cards as the flagged row, expecting no findings.
---

### [MEDIUM] rename-leaves-test misses external-package tests

**Location:** internal/planparser/validate.go:checkRenameLeavesTest
**Issue:** The check requires `test.Unit == old.Unit`.
quarry names an external test package's unit `<dir>_test` (`internal/planparser/targetdirs.go:327`), and the repo's Go testing convention makes `package foo_test` the default.
A `foo_test#TestX` ref therefore never matches a renamed `foo#X`, and the check stays silent for the tests most likely to be written that way.
The decision record says "same unit", which the diff follows literally; the gap is in how the check meets the incident class, not in a departure from the record.

### [LOW] FAILED report hand-back exclusion is untested

**Location:** internal/websterengine/recordbatch_git_test.go:TestRecordBatch_PostBatchChecks
**Issue:** The record bounds the refusal as never firing on a FAILED or NEEDS_PARENT report.
The only guard is `HandBackRemovedTargets: report.Status == ReportStatusOK` at `internal/websterengine/recordbatch.go:570`, and no test fails if it is removed.

### [LOW] Exact-rename exemption not pinned in the engine

**Location:** internal/websterengine/recordbatch_git_test.go:383
**Issue:** The subtest that mixes a rename and a deletion sets `LaterTargetsReturned` first, so it cannot show that an exact rename draws no hand-back.
The record's acceptance criteria list this case.
The unit-level `TestRemovedTargets` covers the reported-rename and declared-removal exemptions, but quarry dropping an exact rename from `Deleted` is a property of the answer fed to it, not of the function.

### [NIT] No `none`-language row for rename-leaves-test

**Location:** internal/planparser/validate_test.go:TestValidate_RenameLeavesTest
**Issue:** `GoToolchain()` is true for an absent language, so `none` is the case most easily broken by a wrong guard; only `csharp` is covered.

## Rejected

- code-correctness LOW, `RemovedTargets` indexing a pair `New` side or Create target through `targetCards`: a Rename `New` side is a `plan:` handle, not a glyph, so it cannot equal a deleted symbol's glyph; a Create of a name the batch just deleted is a speculative plan the lens itself calls unlikely.
- code-tests LOW, the real `batchDelta.RemovedTargets` wrapper is not exercised: it is one line delegating to `RemovedTargets` and `PendingPlan`, both covered, and a per-helper test the public surface already covers is itself a test-economy finding.
- code-affected-set and code-pattern-fit: no findings reported; the handler's own holistic read of the diff against the board entry and decision record found none either.
