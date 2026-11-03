```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (the `sizeProbeBatcher` double and `TestExecutionBatches_SizesReadFromWorktreeRoot` in `internal/webstercli/verbs_test.go`) for agreement with its surroundings, through the fix-consistency lens.

no findings

- The fixture's `PaneCwd` is the anchor path (`hubgeom/webstergeom.go:30`), so a `DiskSizes` rooted there resolves `backend/README.md` to `backend/backend/README.md` and reports it missing. The test's comment and the review's swap scenario hold.
- The probe path `backend/README.md` matches the fixture's commit of `layout.AnchorRel+"/README.md"`.
- The new test uses `tiergate.Require(t, 1)` and no `t.Parallel`, like its siblings.
- Both production `DiskSizes(WorktreeRoot)` sites the review names besides `executionBatches` (`rebaseline.go:110` and `runlevel.go:511`) stay untested. That follows the review's fix text, which asks for a test over `executionBatches` only.

Further tool calls:
1. Grepped for `hubGeometryWithIndex` and `PaneCwd` in `hubgeom`, because my context lacked what `PaneCwd` is in the fixture geometry.
