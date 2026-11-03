---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/webstercli/verbs_test.go:82
    summary: No test pins the worktree-root base of batcher.DiskSizes in a subpath-anchored hub, since the verbs fixture runs the identity batcher, which never reads a size
    fix: "In internal/webstercli/verbs_test.go add a row or assertion to the test that drives executionBatches (cli.go) over newVerbsFixture with a nil state: set fx.CLI.batcher to a small local batcher.Batcher fake whose Batch method calls the sizes argument's Lines with the fixture's Edit target as the repository-relative path (\"backend/README.md\", committed by the fixture) and records the found result; assert found is true. A DiskSizes root swapped to c.geom.PaneCwd (cli.go executionBatches, rebaseline.go Rebaseline call, runlevel.go Run) then fails the test. No production change."
---

### [MEDIUM] No test pins the worktree-root base of `batcher.DiskSizes` in a subpath-anchored hub

**Location:** `internal/webstercli/verbs_test.go:82` (`newVerbsFixture`); production reads at `internal/webstercli/cli.go:292`, `internal/webstercli/rebaseline.go:110`, `internal/websterengine/runlevel.go:511`.

**Issue:** The diff moves `Geometry.WorktreeRoot` from the anchor to the git worktree root, which silently moves the base of every `batcher.DiskSizes(geom.WorktreeRoot)` call with it.
The decision record asks each Webster use of `WorktreeRoot` to be moved to the root it means, and a test to pin both faults.
`newVerbsFixture` sets `batcher.Identity()`, whose `Batch` ignores its size source (`internal/batcher/identity.go`), so no verbs test reads a size through `DiskSizes` at all.
The only other `DiskSizes` callers in tests (`cardgate_test.go`, `runlevel_test.go`) run at an unanchored root where anchor and worktree root coincide.
Pointing those three calls at `PaneCwd` would therefore pass every test.

## Verdicts on the round's focus directive

1. `trackedRel`, `suspectBlobs`, `writtenWorktreePaths`, `failBatchInput`.
   Each is run with `PaneCwd` as a subdirectory of `WorktreeRoot`, and swapping the two roots fails each:
   - `suspect_test.go` (`TestCheckSuspectPaths`, subdirectory subtest) runs `trackedRel` and `suspectBlobs` with `paneCwd = <root>/backend` and the paths `f.txt` (expects `backend/f.txt`) and `../tracked.txt` (expects `tracked.txt`, which only a within-tree check on the worktree root accepts).
   - `runwrites_test.go` runs `writtenWorktreePaths` with the same split and expects `backend/a.go` and `top.go`, dropping `../../outside.go`.
   - `batchfail_integration_test.go` runs `failBatch` with `PaneCwd = <repo>/internal` and the relative suspect path `x.go`, and expects the blob of `internal/x.go`.
   Not a defect.
2. `newVerbsFixture`.
   It seeds a plan with `root: backend` (the anchor directory) and an Edit target `README.md` committed only at `<worktree>/backend/README.md`, plus Create targets committed under `backend/`, with no `.git` at the anchor.
   - `BeginBatch`'s dispatch check (`beginbatch.go:480`) resolved against the anchor would look at `<worktree>/backend/backend/README.md` and report `path-missing`, so `TestBeginBatchCmd_HappyPath` fails.
   - `RecordBatch`'s done checks (`recordbatch.go:933`) resolve the Create target `backend/internal/only/new.go` committed under the anchor directory; the record-batch envelope tests fail if resolved against the anchor.
   - Drift (`recordbatch.go:1067`) and the later-card Delete reference search (`recordbatch.go:136`) are exercised by `TestRecordBatchCmd_DeleteReferencedByLaterCard` and `TestRecordBatchCmd_LaterTargetRemovedEnvelope`, whose files sit at the worktree root and not under the anchor, so an anchor-based resolve fails them.
   - `DiskSizes` is not pinned; that is F1. It is also not in `recordbatch.go`; its calls are the three listed above.

The rest of the diff matches the amended entry: the git calls and plan-path resolution use `WorktreeRoot`, pane-relative resolution, gate run cwds, prompt rendering, instruction files and the session's `_lym`/`.lym` use `PaneCwd`, and `RepoRoot` is gone with its one use moved.
No anchor is joined or stripped by hand, and `hubgeom` and `standalonegeom` remain the only `Geometry` constructors.
