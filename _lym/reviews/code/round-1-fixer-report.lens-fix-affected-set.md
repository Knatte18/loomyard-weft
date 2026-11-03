```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1** (callers of `emptyUnderSkips`): the only non-test callers are `await.go:78` and `baseline.go:222`. The fix adds negative rows only to `TestDriftAwait`, which is what the finding's `fix` named. `baseline_test.go:409` still has only a positive row for the Baseline caller. The review did not ask for a negative row there, so I treat it as outside the fix.
- **F2** (sites that render the singular "1 test" wording): the only producer is `noFileReason` at `capture.go:142-152`, and it now has a `RunCapture` row. The `baseline_test.go:409` fixture still hand-writes the string, as the review said, and covers the Baseline Stuck path rather than `noFileReason`.
- **F3** (sites that count `skip`): `noFileReason` is the only site that renders a count. The other `len(skip)` uses (`verbs.go:74,76`, `config.go:196,198`, `baseline.go:172`, `capture.go:144`) are emptiness checks, so duplicates do not change them. The distinct count now covers both the singular check and the printed number, and a duplicate-skip row pins it.

Tool calls:
1. `grep` over `internal` for `len(...skip)`, `emptyUnderSkips`, `noFileReason` and "skipped N" wording. My context did not hold the non-test callers or the other count sites.
