```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the changed code of F2, F3 and F4 through the correctness lens.

**MEDIUM — `dobbydeps_integration_test.go`, the `CodeReviewCaps` assertion in `TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig`.**
- The test builds its expectation with `loomengine.LoadConfig(fabricengine.ConfigDirs(h.Location), "loom").Caps(loomengine.PerchCode)`. That is the same call chain the seam makes (`deps.go` `CodeReviewCaps`), so it can only fail on a difference in dirs or perch.
- The hub's loom config is not customized, so the defaults are probably equal across perches.
  - If the seam read `PerchPlan` instead of `PerchCode`, the test would still pass.
  - If the seam read the wrong config dirs, it would still pass, because the dirs are the same ones the test passes.
- The review asked for an assertion on "the caps the hub's loom config sets for the code perch". The test needs a hub config that sets a distinctive code-perch cap, such as `code_max_bounces` of 7 and `code_hard_max_bounces` of 9, and must assert those values.
- The `FileTask` half of the test does fail with the defect in place. With the `SetStatus(…, nil)` call dropped, the new task keeps the `draft` status. That makes `filed.Status` non-nil, and the test fails.

**LOW — `deps.go`, the new `SetStatus` error in `FileTask`.**
- The message says "release it, then retry". A retry after a manual release is refused by `openMergeTask`, which finds the open `-merge-in` task and prints "stands open … `lym batten cancel --remove`". That is the same misleading way forward F4 set out to remove, now reached by one path.
- `mergeParent` also appends "fix the cause it names, then run … again". Only the "remove it" branch of the message works with a retry. Dropping "or release it, then retry" would remove the contradiction.
- `lym board` is not a runnable command on its own; it needs a verb.
- The F4 path has no test. The review did not ask for one, and the earlier "Rejected" bullet covers plain error wraps.

**F3 — no findings.**
- `boardengine.IsDoneStatus(task.Status)` is equivalent to the removed inline check. The `done := boardengine.StatusDone` in `mergeparent_test.go` still gives a `*string` through `&done`.
- The gate passed.

No further tool calls were made; my context held everything this lens needed.
