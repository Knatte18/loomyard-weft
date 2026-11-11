---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/pull.go:199
    summary: The pull path's upstream-anchor stop resolves its verb directory from f.warpPath and has no anchored-hub row, though the board entry asks a row for each such call.
    fix: Add a row to the "A marker change on either side stops the pull before warp or the weft moves" table in internal/fabricengine/pull_integration_test.go, or a new test beside it, that builds the diverged-weft fixture in a hub anchored at a subdirectory such as "backend" (the fixture at line 38 uses Anchor "."). Assert that the stop's way forward names "run `lym fabric anchor <slug> --set <marker>` in <warpPath>/backend, which integrates the other marker". The row must fail if lymVerbDir(f.warpPath) fell back to the worktree root.
---

### [MEDIUM] Pull-path upstream-anchor stop has no anchored-hub row

**Location:** internal/fabricengine/pull.go:199, internal/fabricengine/anchorstop.go:171, internal/fabricengine/pull_integration_test.go:38
**Issue:** `upstreamAnchorStop` resolves its way-forward directory with `lymVerbDir(codeWorktree)`, and it has two call sites.
The push path, `pushclassified.go:647` with `filepath.Join(hubPath, slug)`, is now pinned in a hub anchored at `backend` by the changed anchor-change row.
The pull path, `pull.go:199` with `f.warpPath`, is exercised only by `TestPull_WeftDivergedWarpAdvancesCleanly` in a hub made with `Anchor: "."`.
There `lymVerbDir(f.warpPath)` equals the root, and the assertions are substring checks on `lym fabric anchor` and `--set <marker>`, so a regression to the root directory goes unseen.
The board entry's note `anchor-stops-untested` asks for a test row for the upstream-anchor stop of the steps not yet converted, and `verbdir-calls-untested` asks a row for each call that resolves the lym verb directory.
The pull-path call is one of them, so the diff leaves it uncovered.

## Rejected

- Lens `code-tests` LOW: the new weft-conflict row pins only `WayForward`, not Branch, Paths or Err. The extraction is a mechanical move of the removed literal. The existing `TestPull_WeftDivergedWarpAdvancesCleanly` conflict row already asserts `ErrReplayConflict` and the way-forward text through `Pull`, and the weft-dirty row beside it pins only `WayForward` the same way.
- The production extraction `weftConflictStop` is not licensed by an answer key that asks only for tests: rejected. It builds the same four `WeftStopError` fields as the removed literal. It is the minimal seam that lets a tier 0 row reach the anchored way forward, and it mirrors the existing `weftDirtyStop` and `WeftDirtyStopForTest` pair.
- The hand-built `want` string in the new row duplicates production wording: rejected. The neighbouring `WeftDirtyStopCommitsInTheAnchoredDir` and the `wayForwardRows` do the same. The existing pull test builds its expectation through `recordsConflictWayForward` in an unanchored hub, so it cannot pin the anchored directory, which this row does.
- `newFailureFixtureAt` and `newPairFixtureAt` change other callers' behavior: rejected. `newFailureFixture` and `newPairFixture` delegate with `"."`, the previous anchor, and the other two anchor-change rows keep `"."`.
- `anchoredTask`, `anchoredWeft` and `anchoredDir` out of scope for the new row: rejected. They are defined at `failedpush_test.go:253-255` in `TestFailedPushRecord`, which holds the row.
