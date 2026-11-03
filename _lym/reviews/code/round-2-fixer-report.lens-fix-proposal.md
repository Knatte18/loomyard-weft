```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal, which deletes `TestDriftRequests_JoinOnTheWorktreeRootAndStartInTheAnchor` and the two `export_test.go` re-exports.

**Findings**

- **LOW** — deleted test `TestDriftRequests_JoinOnTheWorktreeRootAndStartInTheAnchor`. The review says the test "asserts nothing the public-surface test does not", and that is not true for `Timeout`.
  - **What the test pinned:** `Timeout: time.Duration(step.TimeoutMin) * time.Minute` (`internal/loomcli/driftwiring.go:207` and `:222`). The deleted test asserted `capture.Timeout == 3*time.Minute` and the same for the compare.
  - **Why the integration test does not cover it:** `TestDriftDeps_RunWhereTheStepsDirResolves` sets `timeout_min: 1` and runs commands that finish at once. A wrong unit, such as `time.Second` or `time.Hour`, passes every row.
  - **Where else it might be caught:** a grep of `internal/loomcli/*_test.go` and `internal/driftengine/*_test.go` finds no assertion on this conversion. Engine tests set `CommandStep.TimeoutMin` directly (`baseline_test.go:89`). I did not check whether the engine tests catch a zero timeout.
  - **Case where the proposal is wrong:** change `time.Minute` to `time.Second` in `driftCaptureRequest`. Every remaining test stays green, and a 30-minute capture would be killed after 30 seconds.
  - **Scope of the gap:** the review's stated reason for the deletion covers only `WorktreeDir` and `AnchorDir`, which the integration rows do pin. It does not cover this conversion. The deleted test was not redundant for `Timeout`.
  - **A fix that stays within the review's intent:** keep the deletion and the exports removed. Add a `TimeoutMin: 3` step and a timeout assertion to an existing seam test, or restore a minimal test of the `Timeout` field only. Under the `testing` rule on coverage gaps, that is a row or assertion in an existing test, not a new test.

**Checked, no finding**

- `WorktreeDir` and `AnchorDir` are pinned through `driftDeps` by the integration rows, for both seams, over an anchor of `backend`.
  - **Swap, no-dir row:** the command runs in the worktree root, not `root/backend`, so the row fails.
  - **Swap, `dir: backend` row:** `Join(anchor, "backend")` is `root/backend/backend`, which does not exist, so the run is refused.
  - **Drop:** an empty dir resolves to the process cwd, which is not the anchor.
- `export_test.go` still builds, and no other file uses the removed exports. `go vet` and the tier-0 gate passed in the parent's context.

**Tool calls**
1. Read `driftwiring_integration_test.go`: my context lacked it, and I needed to know which rows pin which fields.
2. Grep for `driftCaptureRequest` and `driftCompareRequest` in `internal/loomcli/*.go`: my context lacked the helper bodies, and I needed to see which fields they set.
3. Grep for `TimeoutMin` and `time.Minute` in `internal/loomcli/*_test.go` and `internal/driftengine/*_test.go`: I needed to know whether another test pins the minutes conversion.
