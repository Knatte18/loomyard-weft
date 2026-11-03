```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- `noFileReason` has one caller, `capture.go:134`. The "skipped N test(s)" strings elsewhere (`await_test.go:28`, `baseline_test.go:409`) are hand-written fixtures, so the dedup change cannot make them disagree.
- `emptyUnderSkips` still gates on raw `len(Skip) > 0`, which fits the dedup. A non-empty list has at least one distinct test, so the count is never 0.
- The `await.go:78` and `baseline.go:222` call sites are unchanged. The new `await_test.go` rows exercise the negative branch of the same gate.
- The new `capture_test.go` rows keep the `wantNamed` and `slices.Equal(rec.Skip, tt.skip)` checks consistent. The record keeps the raw `["TestA","TestA"]` while the reason counts 1, which matches the review's point that the compare dedups and the record does not.

Tool calls: 1. I ran grep over `internal/driftengine` for `noFileReason`, the "skipped N test" strings, `emptyUnderSkips` and `distinct`. My context held the edited files but not the `await.go` and `baseline.go` call sites or the other fixtures.
