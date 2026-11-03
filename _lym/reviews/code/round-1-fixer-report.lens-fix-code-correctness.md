```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I traced each changed function, branch and added test by hand against the code in context. Two LOW findings.

**Traced and holding**
- **`captureReading`:**
  - An unset dir with the anchor at `root/dotnet` renders `` `dotnet` ``, dir `.` renders `` `.` ``, and a relative dir renders as written.
  - An absolute dir outside the root falls back to the whole path.
  - Printing `step.Dir` raw, the old behaviour, would fail every new `seats_test.go` row.
- **`missingDirRefusal`:** the compare site keeps `lym loom resume`, the capture site keeps `lym drift capture and lym loom resume`, and the side site now gets `lym drift capture --side`. The old `Name == stepCapture` branch would have put `lym loom resume` in the side refusal, so the new side row fails with the defect in place.
- **`RunCompare` and `RunSideCapture` rows:** with the anchor at `root/dotnet` and `Dir` set to `dotnet`, swapping or dropping `WorktreeDir` makes `runStep` refuse a missing dir, so both rows fail with the defect.
- **`TestCheckDirs` compare row:** removing the compare entry from `CheckDirs` makes it pass instead of erroring, so the row fails with the defect.
- **Regular-file row:** dropping the `info.IsDir()` check in `StatCommandDir` makes `afile` resolve, so the row fails with the defect.
- **configcli rows:**
  - Row 1 builds the refusal from `errHubFileLoad` wrapping the dir error, so it names `capture.dir` and the path.
  - Row 2 passes because the dir error text is identical before and after the save.
  - Dropping `CheckDirs` from the editor path, or the same-error allowance, fails the respective row.
- **Request helpers:** the new helpers in `driftcli/wiring.go` and `loomcli/driftwiring.go` reproduce every field the old hand-built requests set (`Timeout`, `Now`, `Pool`, `WaitDir`). The call sites still set `Skip`, `Check` and `LockRetry` as before. Behaviour is unchanged.
- **`AnchorDir` in `BaselineInputs`:** the only constructors are `loomcli`'s and the test's, and both set it.
- **Lifecycle step:** after `ReanchorPair`, `PairDir` returning `AnchorPath()` would differ from `WorktreePath()` and fail. The added precondition guards against a fixture that does not differ.

**Findings**

1. **LOW**, `internal/battencli/dobbynotice_test.go`, `TestReadNoticeChild` assertion `child.PairDir != taskLocation.WorktreePath()`.
   - **What:** the assertion passes if the seam reverts to `AnchorPath()`.
   - **Evidence:** `noticeTaskWorktree` builds a bare `.git` directory with no anchor marker, so `taskLocation.AnchorRel` is `.` and `AnchorPath()` equals `WorktreePath()`.
   - **Impact:** the reverted-seam case is pinned only by the lifecycle step. This is not a defect in the fix: the assertion now names the right accessor and the lifecycle step covers the revert.

2. **LOW**, `internal/loomcli/driftwiring_test.go`, `TestDriftRequests_JoinOnTheWorktreeRootAndStartInTheAnchor`.
   - **What:** it pins `driftCaptureRequest` and `driftCompareRequest` but not that `driftDeps` calls them.
   - **Evidence:** the `RunCapture` and `Compare` closures in `driftDeps` hand-set `Skip`, `Check` and `LockRetry` after the helper. A closure that bypassed the helper would not fail this test.
   - **Impact:** the review's wiring gap is closed for the construction. The closure-to-helper link has no test, and that matches the `driftcli` side.

Further tool calls: none. My context already held every changed file.
