```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F2: the search for "opens every" across `internal/shellguard/` has one hit, `rows.go:380`, the `wholeCommandRefusal` comment, which is correct. No stale sibling comment is left.
- F1: the one contained test the review named carries its marker (`rows_test.go:184`).
  Left unchecked: other contained tests in the gate's redundancy report, which is outside my context.

Calls made:
1. `grep "opens every"` in `internal/shellguard/`, to check that no stale sibling comment remains after the F2 fix.
2. `grep` for `testtiming` markers in `rows_test.go`, to confirm the F1 marker is in place.
