```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1 and F2 test changes.

**LOW — `internal/loomcli/redundancynote_test.go`, new `TestRedundancyReportNote_ReadsTheWorktreeReport`**
- The new test is tier 1 (`tiergate.Require(t, 1)`, hubforge hub). It sits in a tier-0 file.
- Every other tier-1 test in `internal/loomcli` lives in a `*_integration_test.go` file, for example `discussionrecord_integration_test.go`, `landingdeps_integration_test.go` and `gatefixer_integration_test.go`. `redundancynote_test.go` is the only exception in that grep.
- The file header (`redundancynote_test.go:1`) still says the file "covers how a code review's redundancy_report note renders the gate's redundancy report". It no longer describes the new read-and-match-against-HEAD test.
- A fix would move the test to `redundancynote_integration_test.go` with its own header and imports. Alternatively, widen the header and accept the mixed tiers.

**No other findings.** The F2 row in `rubric_test.go` matches its neighbours in field style and wanted text. The `//testtiming:keep` comment on `TestReadRubric` still holds.

Tool calls made:
1. A grep for the `_integration_test` naming convention and for `tiergate.Require(t, 1)` in `internal/loomcli`. My context had no listing of which test files hold tier-1 tests.
2. `git show --stat HEAD~1`, to confirm which file the F1 commit touched.
