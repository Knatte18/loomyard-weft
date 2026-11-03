```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Task: check the F1 change (`sizeProbeBatcher` and `TestExecutionBatches_SizesReadFromWorktreeRoot` in `internal/webstercli/verbs_test.go`) through the fix-code-correctness lens.

Walkthrough with the fixture's inputs:
- **Test with the fix in place:** `ParsePlan` reads the fixture plan. `executionBatches(plan, nil)` computes `MerriamBase`, then calls `ExecutionBatches` with `DiskSizes(c.geom.WorktreeRoot)`. A nil state goes to `formBatches` with `c.batcher`, which is the probe. The probe calls `sizes.Lines("backend/README.md")`, which reads `<worktree>/backend/README.md`. The fixture committed that file, so `found` is true and the test passes.
- **Test with the defect in place:** `webstergeom.go:30` sets `PaneCwd` to the anchor path, `<worktree>/backend`. A `DiskSizes` rooted there reads `<worktree>/backend/backend/README.md`. `diskSizes.Lines` maps `ErrNotExist` to `(0, false, nil)` (`estimate.go:46`), so `found` is false and the test fails. It therefore catches the swap F1 describes.
- **Failure paths:**
  - If `Lines` errors, the probe returns the error and `executionBatches` fails, so the test fails loudly.
  - If `formBatches` never calls the batcher, `found` stays false and the test fails. It cannot pass vacuously.
- **Callers:** no production signature changed, and the new types are test-only. The package compiled and the tier-1 gate passed, with the new test in the log.
- **Residual point, not a finding:** the test pins only the `cli.go` `executionBatches` call, not the `rebaseline.go:110` and `runlevel.go:511` `DiskSizes` calls. That is the scope the review's `fix` key set.

Tool calls beyond my context:
1. Read `internal/batcher/estimate.go:25-65`, because my context did not show how `diskSizes.Lines` handles a missing file.
2. Grepped `PaneCwd` and `WorktreeRoot` in `internal/hubgeom/*.go`, because my context did not show what `PaneCwd` is in the webster geometry the fixture uses.
3. Grepped `hubGeometryWithIndex` in the webstercli tests, to confirm the fixture builds its geometry from `hubgeom.WebsterGeometry`.
