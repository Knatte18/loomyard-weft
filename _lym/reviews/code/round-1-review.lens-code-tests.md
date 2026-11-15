```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
1. **A passing `vstest` invocation that reported no test prints the line to stderr and the log.**
   - Pinned by `TestRunTierNoTestNotice` in `internal/gatecli/tierrun_test.go`. Rows `ANarrowedProjectThatReportsNoTestIsNamedWithItsClassFilter` and `AWholeRunThatReportsNoTestIsNamed` fail if the line is dropped or reworded.
   - The test checks both stderr (exact lines) and the log.
2. **Per-invocation discovered delta.**
   - Pinned by the narrowed row. Its first invocation reports a test and its second reports none, so a counter compared without the `discoveredBefore` subtraction would print no line and fail.
3. **No line when the project reported a test.** Pinned by `AProjectThatReportsATestPrintsNoLine`.
4. **No line for a non-`vstest` counter.** Pinned by `AGoJSONRunPrintsNoLine`.
5. **Class filter terms in the line.** Pinned by the narrowed row, which has two terms joined by `|`. The whole-run row pins the no-terms form.
6. **The line names the run directory when the invocation has no targets** (`project = dir` in `noTestNotice`). **None.** Every row sets `targets`, so this branch is unpinned.
7. **No line when the invocation fails.** The notice sits after the `exit != 0 || ctx.Err() != nil || capLine != ""` return. **None.** No row has a failing `vstest` command, so a regression that prints the line for a failed or capped run is undetected.
8. **The `runInvocations` signature gains `notice`.** `TestRunInvocationsTestTargets` in `internal/gatecli/testtargets_test.go` passes a no-op, and the `runTier` wiring is covered by item 1.
9. **`fabricengine.CodeLocation` maps a pair's records sibling to its code worktree, and the testmap refusal names the prime's code worktree.** Pinned by the `FromAPairsRecordsSibling` row in `TestGateTestMap`.
10. **`CodeLocation` maps the prime's records sibling to the prime, so the verb acts there.** Pinned by `FromThePrimesRecordsSiblingActsInThePrime`.
11. **`CodeLocation` leaves a location that is not a weft sibling unchanged.** Covered implicitly by the existing `OutsideThePrime` and gate rows, which resolve code worktrees.
12. **`CodeLocation` leaves a location unchanged when the code worktree directory is absent.** **None.** No test builds a weft sibling with no paired directory.
13. **The `resolveHub` mapping reaches the other gate verbs.** The summary claims `lym gate redundant` is fixed. **None.** No test runs `redundant`, or any verb other than `testmap`, from a records worktree. Only `testmap` is pinned.
14. **The `doc.go` and stencil text changes.** Static prose, exempt.
15. **The detached-run subtest no longer races the lease against the child's start.** The edited subtest is itself the test. It fails if the line is not ended after `expireLease`, or if the tier is not killed and the slots not freed (`requireEnded`, `requireFree`).
16. **The redundancy broken-build subtest no longer depends on a per-test timeout.** The edited subtest is itself the test, and its `TestBetaNewOne` contained assertion is unchanged.

**Findings**
- **LOW, class `design`.** Location: `internal/gatecli/tierrun_test.go:231-291`, over `internal/gatecli/tierrun.go:350-355`.
  - Summary: no row covers a failing `vstest` invocation, which must print no no-test line, or an invocation with no targets, whose line names the run directory.
  - Fix: add two rows to the existing `TestRunTierNoTestNotice` table, not new tests.
  - Evidence: no table row sets an empty `targets` or a command that exits non-zero.
- **LOW, class `design`.** Location: `internal/fabricengine/weftwiring.go:62-71`.
  - Summary: the "absent code worktree directory returns the location unchanged" branch of `CodeLocation` has no test row. It is the safe-failure path for a records worktree with no pair.
  - Fix: add one row to `TestGateTestMap`'s refusal table, or one case to a `fabricengine` test.
  - Evidence: the only new rows are `FromAPairsRecordsSibling` and `FromThePrimesRecordsSiblingActsInThePrime`, both with a paired code directory.
- **LOW, class `design`.** Location: `internal/gatecli/run.go:670-675`, `internal/gatecli/testmap_integration_test.go:728-737`.
  - Summary: `resolveHub` now serves every gate verb, and the summary claims `lym gate redundant` is fixed, but only `testmap` is exercised from a records worktree.
  - Fix: an added row on an existing verb table, not a new test.
- **Test economy:** no duplicate or per-helper test found. `TestRunTierNoTestNotice` and the new `testmap` rows each cover a behaviour no existing test covers, which agrees with the redundancy report's "unique".

**Tool calls (1 of 6)**
1. `grep` over `internal/gatecli/tierrun_test.go` for tier markers and test names. My context did not show whether the sibling tests spawn a shell under `tiergate.Require(t, 0)`. `TestRunTierProcessCap` also calls `runTier` at tier 0, so the new test follows the file's own convention.
