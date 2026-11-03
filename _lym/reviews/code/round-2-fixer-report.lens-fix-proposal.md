```yaml
lens: fix-proposal
outcome: reported
```

One MEDIUM finding. The new test pins only one of the three `DiskSizes` call sites.

**MEDIUM — `TestExecutionBatches_SizesReadFromWorktreeRoot` (new test in `internal/webstercli/verbs_test.go`)**
- **Summary:** The test pins only the `cli.go:292` call, but the finding's fix text and the commit message claim it covers all three.
- **Evidence:**
  - The test calls `fx.CLI.executionBatches(plan, nil)` directly, and that method contains only the `cli.go:292` call.
  - `rebaseline.go:110` builds its own `batcher.DiskSizes(c.geom.WorktreeRoot)` inside the rebaseline `RunE`.
  - `runlevel.go:511` builds its own `batcher.DiskSizes(deps.Geom.WorktreeRoot)` inside the engine's run level.
  - Neither of those two goes through `executionBatches`.
  - Swapping either root to `PaneCwd` still passes the new test.
  - The review's own fix text says a swap in "cli.go executionBatches, rebaseline.go Rebaseline call, runlevel.go Run" would fail the test. That holds only for the first.
- **Fix:** Either cover the other two calls (a rebaseline verb test and a `Run` test using the probe batcher, and a subpath `PaneCwd` fixture for `runlevel_test.go`), or have the fixer-report record that only `cli.go:292` is pinned and name the other two as left unpinned.

**Not a finding:** the fixture's `PaneCwd` is the anchor (`<worktree>/backend`). I did not confirm that from the helper, though: my grep of `hubGeometryWithIndex` for `PaneCwd` returned nothing. The inference rests on the fixture's `AnchorRel: "backend"`. If it holds, a `PaneCwd`-rooted source would look for `backend/backend/README.md`, which is absent. The fixture commits `backend/README.md` only at the worktree root, so the `cli.go:292` swap does fail the test.

Tool calls:
1. grep for `DiskSizes` in `rebaseline.go` and `runlevel.go`, with context. My context showed only `cli.go:292`, so I needed to confirm those two calls sit outside `executionBatches`.
2. A second grep in the same command, for `PaneCwd` in `hubGeometryWithIndex`. My context did not show how the fixture sets `PaneCwd`. It returned nothing.
