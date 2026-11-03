```yaml
lens: fix-proposal
outcome: reported
```

**LOW — `noFileReason` in `internal/driftengine/capture.go`: the F3 dedupe never changes the count in production.**
- Every production `CaptureRequest` gets its skip list from `driftengine.SkipList(scope)`: `internal/driftcli/cli.go:243` and `:297`, and `internal/loomcli/driftwiring.go:190`. `SkipList` already removes duplicates (`compare.go:114-128`).
- So `len(skip)` equals the distinct count before `noFileReason` sees it. F3's premise, that a repeated `--skip` reaches the reason as a raw count, does not hold on those paths.
- The added `slices.Compact(slices.Sorted(slices.Values(skip)))` is therefore a defensive no-op outside tests. The new "counts a repeated skip once" row only passes because it feeds `RunCapture` a list that production never builds. It guards no real behavior.
- Separately, `SkipList` also appends accepted tests, so the reason's "scope partial skipped N tests" can count accepted tests too. That text predates this fix and F3 does not touch it.
- Suggested handling: keep the change, since it is harmless, or revert it and drop the duplicate row. Both options are valid.

**F1 and F2: no findings.**
- F1: dropping the `WroteNoFile` condition from `emptyUnderSkips` makes the first new row fail, because `Skip` is set and `--scope` appears. Dropping the `len(Skip) > 0` condition makes the second new row fail. Both rows would catch the defects they target. The pre-change record case, skips with no `wrote_no_file` key, is the same shape as the first row.
- F2: the singular row asserts "skipped 1 test,", so the trailing comma rules out "tests". `wantNamed` is non-empty, so the "no skips named" check is skipped for that row, as intended.

Tool calls:
1. grep for `Skip:` in non-test Go files under `internal`: I needed to know what `RunCapture` callers pass as `Skip`. My context lacked it.
2. grep for `CaptureRequest{` and `SkipList(` in non-test Go files under `internal` and `cmd`: I needed to confirm that every production caller routes through `SkipList`. My context lacked it.
