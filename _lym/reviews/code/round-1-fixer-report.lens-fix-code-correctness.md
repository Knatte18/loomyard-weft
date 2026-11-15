```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code changed by the fixer's round.

- **MEDIUM** — `internal/fabricengine/snapshot_integration_test.go`, `TestRebuildIndex_AgreesWithTheIncrementalIndexOverABackDatedMerge`.
  - The test was retargeted at `RebuildIndex`, but it can no longer fail on the ordering defect it was named for.
  - Index entries are keyed by warp SHA, and this history holds two distinct warp SHAs. A date-ordered scan in `scanWarpSHATrailers` (dropping `--topo-order`) would therefore yield the same two entries.
  - The only assertion that distinguished topological from date order was `SnapshotWarpSHAForTest` returning the side baseline. The fix removed it.
  - The `--topo-order` behaviour of the production scan is now pinned by no test. F7 asked for a retarget only where `RebuildIndex` or `WeftSHAForWarpSHA` tests cover that behaviour, and this one does not.
  - A real retarget would give both commits the same warp SHA and assert `WeftSHAForWarpSHA` resolves to the topologically newest commit.

- **MEDIUM** — `internal/fabricengine/index.go:185`, the "does not have any commits yet" tolerance. The fix deleted `TestSnapshotWarpSHA_UnbornWeftHEAD`.
  - My grep over `internal/fabricengine` finds no other reference to that message text. I did not grep for tests that rebuild the index on an unborn weft, so another test may still cover the branch.
  - If none does, the tolerance in `scanWarpSHATrailers` is now untested, and the fixer chose to delete this test rather than retarget it at `RebuildIndex`.

- **LOW** — The pre-existing `GOOS=windows go vet ./internal/reedengine` failure (`mustTmuxOutput` undefined in the integration tests) is not caused by the fix; only `proctree_windows.go` changed in that package. The `filepath.Base` change is correct on Windows, which splits on both separators.

Checked and found correct:
- `TestReplayRequest_RefusesABoardWorktreeOfNoHub` reproduces the deleted test's layout, and the gate ran it green.
- The `PushAnchored` retargets pass tier 1.
- `TerminalPhases` is a clone, and its allowlist entry satisfied the guard.
- The two-buffer row in `TestExitCode_UnknownModule` asserts the envelope on stdout and not on stderr.

Tool calls:
1. `grep -rn "any commits yet" internal/fabricengine` plus the `scanWarpSHATrailers` listing: to see whether any test still covers the unborn-HEAD tolerance. My context lacked the post-edit test inventory.
