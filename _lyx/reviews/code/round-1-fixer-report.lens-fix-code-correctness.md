```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F4 fixes (lens fix-code-correctness).

I walked one concrete input through each changed branch and found nothing wrong in the code or the added tests. One test could pass without exercising its guard, which is the only finding.

- **F1, `checkRenameLeavesTest`:**
  - Take renamed `internal/foo#Old` and listed test `internal/foo_test#TestOld`. `goPackageDir` strips `_test` from the test's unit, so both sides become `internal/foo`, the guard passes, and one finding comes out.
  - A same-package ref and an unrelated unit (`internal/bar`) behave as before.
  - A unit with a file extension comes back unchanged from `goPackageDir`, so file-unit behaviour is unchanged.
  - The new `internal/foo_test` row fails under the old `test.Unit != old.Unit` comparison, because that gave 0 findings against a wanted 1.
- **F4, `none` row:** `Validate` returns no `rename-leaves-test` findings for a `none` plan, and the row asserts 0. It would catch a guard that only excluded `csharp`.
- **F3, restructured exact-rename subtest:**
  - The first call runs with `LaterTargetsReturned` unset and must return `*LaterTargetRemovedError`. It names `WillGoAway` and not `WillMove`. If an exact rename were counted as removed, the message would name `WillMove` and the test would fail.
  - `forkRewroteReport` then lets the second call proceed to the plan-rewrite assertions.
  - The old `if _, err :=` shadowing is gone. Both new `err` uses compile, and both gate runs passed.
- **F2, FAILED-report subtest:** the only guard is `HandBackRemovedTargets: report.Status == ReportStatusOK` (`recordbatch.go:571`), checked at `recordbatch.go:991`. With the guard dropped, this fixture matches the OK hand-back fixture, so it would return the error and the test would fail.

**LOW:** `a failed report is never handed back over a removed later target` in `internal/websterengine/recordbatch_git_test.go` never asserts what `RecordBatch` returned, only that it was not a `LaterTargetRemovedError` and that the flag is unset. If a FAILED report left `RecordBatch` before the post-batch pass, the test would pass whether or not the guard exists. I did not confirm that a FAILED report reaches `postBatchChecks`; the guard's own condition suggests it does. Asserting the actual outcome (for example `ErrBatchFailed`, if that is what a FAILED report produces) would close this.

Further tool calls (1 of 6):
1. `grep` and `sed` on `internal/websterengine/recordbatch.go`, to see where `HandBackRemovedTargets` is set and consumed (lines 571, 813, 826, 985–1002). My context had only the line-570 region.
