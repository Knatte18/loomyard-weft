```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test that would fail if it broke.

**Behaviours and the tests that pin them**

1. **A darn run's verify-budget notice now opens with `loom <slug>: `** (`internal/loomcli/darnwiring.go:273`, the `Notify` closure in `treeScopeSettings`).
   - Pinned by `TestVerifyBudgetNotice_NamesTheRun` (`internal/loomcli/notifystamp_integration_test.go:41-67`, tier 1).
   - The test gets the budget from `treeScopeSettings` on a real pair, calls `Notify`, and compares the posted creel subjects exactly with `slices.Equal` against the literal `"loom budget-task: Darn-Write gate: ..."` (lines 63-66).
   - The expected value is a literal, not something computed the way the code computes it, so the test is not tautological.
   - It fails if the prefix is dropped, misworded, or takes another value than the pair's worktree name. It also fails if `Notify` stops posting to the parent orch, because the posted subjects would then be empty.
   - The commit message and summary say the test fails on the merge base. Its assertion bears that out: the base posts the line without the prefix.
2. **Doc comment on `treeScopeSettings`, line 254.** This is prose only and needs no test.
3. **File header of `notifystamp_integration_test.go`.** This is prose only and needs no test.

**Unchanged and outside this lens:** the hardener branch, which has no budget, and `verifyBudgetNotice`'s own rendering. The diff touches neither.

**Advisory reports:**
- The mutation report's "1 uncovered changed blocks" and the reach report only list tier-0 `TestWire_*` tests reaching `treeScopeSettings`. That fits a tier-0 map: the new tier-1 test is the one that runs the `Notify` closure. It is a candidate only, and the test above covers it, so I raise no finding.
- The redundancy report puts the new test under "not judged". No existing test asserts the budget notice's posted subject. `TestHaltNoticeSeams_CarryTheRunsAutonomy` asserts only the halt seams' autonomy stamp, so the new test is not redundant.
- The test runs without `t.Parallel`, and its doc comment gives the reason: the strand environment variable it sets with `t.Setenv` is process-global. That follows `PATTERN-test-speed`.

**Findings:** none.

Tool calls: none. My inherited context already held the diff, the test helpers, `treeScopeSettings`, and the reach, mutation and redundancy reports.
