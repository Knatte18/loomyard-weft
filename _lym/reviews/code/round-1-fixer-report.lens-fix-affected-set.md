```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for F1 (measured-time bounds) and F2 (stale "100 ms" markers), and check that the fix covers every member.

**Findings:** no findings.

- F2: the only mention of the old bound was the marker on `TestBattenIntegration_Rows`. A grep for `under 100 ms`, `D2 acceptance` and `startCostBound` finds no stale copy. The marker, the constant's comment and the step's comment all agree.
- F1: in `internal/battencli`, the only measured-time comparison against a bound is `startCostBound` at `lifecycle_integration_test.go:1467`. No other `_test.go` file in the package uses `time.Since` or `elapsed`. The new `PATTERN-test-speed.md` exception covers this test, and the marker carries the `testtiming:keep` it requires.
- I did not rerun the review's repo-wide search for other measured-time tests outside `battencli`. The review rejected those as fake-clock tests.

Calls made:
1. grep for `startCostBound`, `under 100 ms` and `D2 acceptance` in `.go` and `.md` files: checks F2's set and F1's constant, which my context lacked repo-wide.
2. grep for `time.Since` and `elapsed` in the other `battencli` test files: checks F1's set of measured-time comparisons, which my context lacked beyond the one file.
