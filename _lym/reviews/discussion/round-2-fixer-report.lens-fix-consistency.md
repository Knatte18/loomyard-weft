```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: I checked whether the changed text still agrees with the sibling decisions, criteria and cross-references it touches.

1. **LOW.** Changed text: D5's new sentence "gets a map recording its Go file hashes and no tests, with the build failure as its cause where one occurred". The D4 record it depends on was not updated to match. D4's "It holds:" list names the commit and build time, the Go file hashes, the block key table and the per-test entries. It has no field for a build-failure cause. Evidence: D4's bullets in `_lym/discussion/decision-record.md` are unchanged, and the D5 criterion "the latter naming the failure" relies on that field.

2. **LOW.** Changed text: D7 step 1, which now reads "the package's `_test.go` files at the merge base" for each package holding a changed test file. The twin cost sentence in D7's Rationale was not updated. It still says "A repeat call on a tree its report matches pays only the diff read, the merge-base reads of the changed test files and the hashes". After the F5 fix, that read covers every `_test.go` file of each touched package, not only the changed files.

3. **LOW.** Changed text: support-log Q7, which now adds "`redundancy_build_timeout_min` 10 per build (Round 2)". Q7 is the refresh's limits, but the key also bounds the gate step: D7 step 4 says "Each coverage build and the `lym` build run under `redundancy_build_timeout_min` (D5)". Q6, the step's cost limits, still lists only `redundancy_max_tests`, `redundancy_test_timeout_sec` and `redundancy_budget_min`.

Further tool calls: none.
