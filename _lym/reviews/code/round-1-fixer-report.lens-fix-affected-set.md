```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1 (every site of the owed-push way forward):** The search covers all `PushOwedWayForward` and `PushOwedWorktree` sites in `internal/` and `cmd/`. The only non-`runfsm` producer is `internal/pairteardown/runread.go:238-239`, which now sets both fields, and its test row at `runread_test.go:318-319` asserts both. No other code builds these `Inputs`. The set is covered.
- **F2 (tests that lost `t.Parallel`):**
  - I compared the `t.Parallel()` counts per modified test file between `7346d6a8` and HEAD.
  - Seven files lost calls: `battencli/commitstatus_test.go`, `fabriccli/cli_test.go`, `fabricengine/parenttips_integration_test.go`, `hubforge/hub_test.go`, `lymcwd/lymcwd_test.go` and `statuscommit/statuscommit_test.go`, plus the flattened `cli_test.go`.
  - I traced `battencli/commitstatus_test.go`, `lymcwd/lymcwd_test.go`, `statuscommit/statuscommit_test.go` and `parenttips_integration_test.go` to deleted tests. The removed calls sit inside `TestNewCommitStatusSeam_LayoutWiring`, `TestResolveWithin_OneRepository`, `TestNew_MergeOperationWaits` and `TestChangedCodePaths_LeavesOutTheRecordsPaths`.
  - `cli_test.go` dropped two calls (top level plus row) for one restored (top level) because the test was flattened to a single case. It is now correct.
  - I did not trace `hubforge/hub_test.go` (−4). The diff stat shows only deletions there, consistent with the removed one-repository hub tests, but I did not read the removed lines.

Tool calls:
1. Grep of `PushOwedWayForward` and `PushOwedWorktree` over `internal` and `cmd`, for F1's site set. Context held only the edited files.
2. Per-file `t.Parallel()` count at base versus HEAD plus the diff's removed `t.Parallel` lines, for F2's set. Context held only `cli_test.go`.
3. The `parenttips` diff and the removed `Test` and `t.Run` lines in three files, to attribute the dropped calls to deleted tests.
