```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — per behaviour, the test that pins it, or none.

**Per-behaviour list**

1. Step never changes the gate's exit code (`run.go:200-203`, `redundancy.go:97`). The `run.go` wiring has no test. `TestRedundancyStep` calls `step.run(...)` directly and never reads `runTest`'s return code. Errors-as-lines are pinned only for "no map" and "build failure" (`broken`). **Partial.**
2. A failed `lym` build, a corrupt or unreadable map, an unreadable merge base and a failing test each become an error line. No test row covers any of them.
   - The fixture has no `tier1` declared, so `buildLym` never runs.
   - `checkMapBlocks` and the unreadable-map branch have no row.
   - `TestDropsHangs` covers only the timeout path. `testmap_integration_test.go` has a lym-build stand-in (`testMapLymBuildsEnv`), but this step's tests don't use it. **None.**
3. A failed report write is logged at Warn and the result stands (`redundancy.go:117-120`). **None.**
4. A red tier gives a report that judges nothing. `ReportsThatJudgeNothing/ARedGate` calls `step.run(true, ...)`. The `red := code != 0` computation in `run.go:192` is not tested. **Partial.**
5. No parent branch gives a note. `NoParentBranch`, using the prime location. **Pinned.**
6. A `scoper` that is not `go` gives a note. `AScoperOtherThanGo`. **Pinned.**
7. The not-in-a-hub note (`noteNoHub`). **None** (`run(false, false)` is never called). Low, since it is a code path the spec doesn't list.
8. Test selection by function text at the merge base through `CodeDirAt` and `CodeFileAt`. It is pinned by `JudgesTheDiffs...`: `TestBetaOne` is edited and taken, `TestAlphaBase` is unchanged and untaken, and `wantTaken` is checked. **Pinned.**
9. A moved or renamed file leaves its unchanged tests untaken (`moved`), and a deleted file adds none (`gone`). Both are pinned by `wantTaken`, which has no `moved` or `gone` key. **Pinned.**
10. Drops, each with its reason. `drops` pins the keep reason, `tier1` undeclared, `tier2` and the timeout. The spawn-scan drops (`OutOfProcess`, `Unresolved`) and the `reasonScanFailed` path have **no row**. The spec lists the spawn-scan drop in D7 step 2, so this is a gap. **Partial.**
11. The cap `redundancy_max_tests`, with the first N in file then name order. `zcapped` is pinned with max=7. The test count is 11 candidates, of which 6 are dropped. The ordering of kept versus past-cap is only weakly tested. The cap hits only the last one, `TestZcappedNew`, which is also last in file order. **Pinned, weakly.**
12. The slot is taken after the tiers release theirs and held to the end, with `lease` released in `close`. **None.** No test asserts slot acquisition, release, the inherited-slot path (`inherited: true` takes none) or the wait record.
13. Budget counted from slot acquisition, with no build or test started once spent (`ASpentBudgetStartsNoBuild`, using `time.Nanosecond`). The test asserts that no build starts and that `alpha`, `beta` and `broken` tests are `reasonBudgetSpent`. It does not assert the `drops` or `zcapped`/`nomap` tests, and `broken` is not judged for the budget but for the map failure. `broken` has a repaired map at that point (it was rewritten in the "Matching" subtest), so this passes. It does not cover a budget spent mid-package, with a test unrun after a successful build. The spec requires "each test it did not run" to be listed. **Partial.**
14. The skip-on-match check uses the current files, not HEAD. `ARepeatOnAMatchingTreeRunsNoBuildAndKeepsTheReport` checks `Written` unchanged and no build. It is worktree-based. **Pinned.**
15. A repeat reruns for an error line, an unmapped package or a failed-build map. `ARepeatRunsAgainWhileAPackageHasNoUsableMap` pins the package case. It does not cover the error-line path (`len(Errors) > 0`), which is only reachable with an error row and so is **none**. The budget path is pinned by `report.retries()` in `ASpentBudgetStartsNoBuild`.
16. The skip still reruns "including when the diff also changes a tier2 test". The fixture's `drops` package has `TestDropsTier2`, and the matching subtest is run with it present, but the report **keeps** (skips). The acceptance criterion is "second call runs no build and no test, including when the diff also changes a tier2 test"; that is pinned by the match subtest. **Pinned.**
17. `RedundancyReportMatches` over a commit. `TheReportMatchesACommitOfItsTreeUntilATakenTestChanges` covers true, an untouched Go file giving true, and a changed taken test giving false. A changed package file hash (non-test `.go`) giving false is **not tested**, and a report with a `Note`/`RedGate` never matching is **none**. **Partial.**
18. The map-drift line names files that differ at the merge base. `alpha` with `Drift: {"alpha_test.go"}`. **Pinned.**
19. The map commit and age on the package line. **Pinned** (`c0ffee`, `2h0m0s`).
20. Covers: the map's tests with unchanged function hash, minus the judged test itself, plus the other judged tests (`judgeRuns`).
    - Contained naming the cover: `TestAlphaNewOne` covered by `TestAlphaBase`. **Pinned.**
    - Unique: `TestAlphaNewTwo`. **Pinned.**
    - A map test whose function text changed is not counted: `TestBetaOne` is edited, and its map entry is stale, so `TestBetaNewOne`... the expected result is `Contained, Covering: [TestBetaOne]`. Because `TestBetaOne` is judged here, it is the other judged test. The stale-hash exclusion via `hashes[mapped.Name] != mapped.FuncHash` is **not isolated**, because the name is in `judged` anyway. A separate test with a changed map test that isn't judged is **none**. Likewise a map test that was deleted has no row.
    - A judged test never its own cover: `TestBetaOne` is judged and is also in the map. It is excluded by `judged[...]`, and is reported unique with no self-cover. **Pinned** (indirectly).
21. Two judged tests covering only each other's blocks are never both contained. This is pinned at the `testcover` level (`JudgeAgainst`, third row). At the gatecli level there is no row, though the logic is shared. **Pinned** (unit).
22. Greedy pass with kept tests (`JudgeAgainst`). Pinned by the three rows. `Judge` is `JudgeAgainst(runs, nil)`, so the existing `TestJudge` guards the prior behavior. **Pinned.**
23. `CoveredKeys`: double-listed block, nested package left out, key shape equal to `KeyBlocks`. `TestCoveredKeys` pins all three. **Pinned.**
24. `ParseTestFuncs`, with `File` filled. `TestScanTestFuncs` was updated for `File`. A direct row for `ParseTestFuncs` over an in-memory source is **not** present, but it is exercised through the gatecli integration test and through `ScanTestFuncs`. **Pinned** (indirectly).
25. A package that is not a module package (`testdata`, a dot or underscore directory) is passed over by `isModulePackage`. **None.**
26. `redundancyModule` errors (no `go.mod`, neither `tier0` nor `tier1` declared). **None.**
27. Config: `redundancy_max_tests` and `redundancy_budget_min` have defaults (`templateDefaults`) and a `<1` refusal. **Pinned.** An unknown report is refused with the known names: `an unknown report` is **pinned**. A tuned `webster` without `reports` gets the template's, and `reports: []` turns it off: **pinned** (one row).
28. `reports` on `parent-align`, `darn-write` and `code-burler-fix` in the template: pinned by `templateDefaults`. A tuned `parent-align` row is updated in the lints row.
29. `GateKey.Reports` cloned in `Gate()` (`tiers.go`). **None**; no test reads `Gate("webster").Reports`. Low, since the step reads `cfg.Gates` directly in `run.go:197` and not via `Gate()`.
30. `secondUnit` set in `cli.go` and `rungate.go`. **None**. A wrong zero unit would make every test time out immediately. Low-Medium, since no CLI-level test runs a gate with `reports`.
31. A test does not launch a real test binary or `go test`: the integration test uses the stand-in (`standIn` is `os.Executable`, with a `helperEnv` role). **Complies.**
32. `ReadRedundancyReport` and its tolerance of unknown keys. **None**, though it delegates to `state.ReadJSON`.

**Findings**

- **F-T1, MEDIUM, design.** `internal/gatecli/redundancy.go:97-122` and `internal/gatecli/run.go:200-203`. The never-changes-the-exit-code invariant has no test at the `runTest` seam. All tests call `step.run` directly, so a regression in the `run.go` wiring (such as `code = ...` after the step, or a step panic or error) is not caught. Evidence: `redundancy_integration_test.go:200,347`.
- **F-T2, MEDIUM, design.** `internal/gatecli/redundancy.go:486-489`, `504-525` (`buildLym`) and `redundancy.go:296,301` (`checkMapBlocks`). The acceptance criterion lists a corrupt or unreadable map and a failed `lym` build as report lines. The fixture declares no `tier1` and writes no corrupt map, so neither is exercised. The `lym`-build stand-in exists in `testmap_integration_test.go` but is not used here. Add rows for a `tier1`-declared test with a failing `lym` build (one error, tests not judged with `reasonLymFailed`, no second build) and for a map with an out-of-range block index.
- **F-T3, MEDIUM, design.** `internal/gatecli/redundancy.go:241-246` and `scanPackage` (`redundancy.go:252`). The spawn-scan drops (`OutOfProcess`, `Unresolved`) have no row; only the keep marker is covered. D7 step 2 lists them as drops with reasons. Add a row in `drops` for a test that references `os.Executable` or `exec.Command("go")`.
- **F-T4, LOW, design.** `internal/gatecli/redundancy.go:117-120`. The failed report write logged at Warn, with the result standing, has no test. Acceptance criterion: "A failed report write is logged and leaves the result too."
- **F-T5, LOW, design.** `internal/gatecli/redundancy.go:405-432` (`holdSlot`, `close`) and `redundancy.go:411`. The slot behaviours (acquire, release on `close`, the inherited path that takes none, the wait record, the failed acquisition giving `reasonNoSlot`) have no row. The budget-from-acquisition ordering is also unpinned.
- **F-T6, LOW, design.** `internal/gatecli/redundancy.go:581-584`. A map test whose function hash changed since the map was built, and which is not itself judged, must not count as a cover (acceptance criterion). The fixture's edited `TestBetaOne` is also a judged test, so `judged[...]` excludes it before the hash check runs. Deleting the hash comparison would not fail any test. Add a map entry with a stale `FuncHash` whose test is not in the diff, and expect `unique`.
- **F-T7, LOW, design.** `internal/gatecli/redundancy.go:462-499`. The budget running out mid-package, after a successful build, with the remaining tests listed as not judged for the budget, has no row. `ASpentBudgetStartsNoBuild` covers only the before-build case. Add a case where the first test runs and the rest hit the deadline.
- **F-T8, LOW, design.** `internal/gatecli/redundancyreport.go:119-141`. `RedundancyReportMatches` returning false for a changed non-test Go file's hash of a judged package, and for a report with a `Note`/`RedGate` or no parent branch, has no row. Only the taken-test change is covered.
- **F-T9, LOW, design.** `internal/gatecli/redundancy.go:160-164`, `redundancyreport.go:165-169`. The retry-on-error-line case (`len(Errors) > 0`) has no row, although the criterion lists it with the other retry triggers. A report with only an error line (such as a failed build in a package whose map is fine) should be re-run on a repeat.
- **F-T10, NIT, consistency.** `internal/gatecli/redundancytests.go:183-194`, `redundancy.go:128`, `redundancy.go:340` (`isModulePackage`, `redundancyModule`, `noteNoHub`). No row covers the testdata, dot or underscore directory pass-over, the no-`go.mod` or no-tier error, or the not-in-a-hub note.
- **F-T11, NIT, consistency.** `internal/gatecli/cli.go:103`, `rungate.go:29`. `secondUnit: time.Second` is wired with no test. A zero value would make every test time out, and only a CLI-level run with `reports` would show it.

**Test economy:** no duplicate coverage found. `TestCoveredKeys`, `TestJudgeAgainst_...` and the config rows each pin a distinct behaviour. The `TestScanTestFuncs` change only adds `File` to existing rows. No test launches a real test binary or `go test`.

**Tool calls**
1. `git diff` of `blockkey_test.go`, `verdict_test.go` and `testfuncs_test.go`: my context held only the non-test diffs and `redundancy_integration_test.go`, not these test diffs.
