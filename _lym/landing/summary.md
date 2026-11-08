# Fold and trim fabricengine tier 1 tests the coverage study flagged

The coverage study in `docs/research/speed/fabric-test-redundancy-2026-11-08/` lists the `internal/fabricengine` tier 1 tests whose code blocks other tests also cover.
This change makes the folds, anchor cuts and tier 0 moves that study names, and keeps every test whose assertions coverage cannot see.

## What changes

- **Contained tests confirmed and kept.**
  The study's greedy set holds 267 unmarked tests.
  The module-wide check `go run ./cmd/testtiming -pkg ./internal/fabricengine -redundancy` confirms only five of them as contained:
  `TestFabricWarp_IsAncestorOrdersWarpCommits`, `TestChangedPathsBothSides_NamesRenamesAndRawPaths`, `TestHealthy_UnbornWeftBranchIsAVerdictNotAnAbort`, `TestUnwireJunctions_AlreadyUnwiredIsNoOp` and `TestPushWarpRebaseFreeAt_PushesAndRecordsBranchPush`.
  Each asserts an outcome its covering test reaches but never checks:
  the false direction of `IsAncestor`, raw non-ASCII and empty-range paths, a verdict instead of an error on an unborn weft branch, an empty result on a second unwire, and the `KindBranchPushed` record with no lock left behind.
  All five now carry `//testtiming:keep` with that reason, and none is dropped.
  The other 262 are not dropped, because the module-wide check does not confirm them.
- **`TestCleanup_*` fold.**
  `TestCleanup_DryRunMatchesApplyVerdict` and `TestCleanup_ForceIsReservedAndChangesNoVerdict` become `TestCleanup_VerdictIsIndependentOfDryRunAndForce` on one hub.
  Both dry runs run first on the untouched fixture.
  The apply without force follows, and the orphan branch is recreated for the forced apply.
  Every assertion of both originals stays.
- **`TestVerbCases_ArrangeSetsItsPreconditionField` fold.**
  The cross product's `assertArrangePreconditions` now takes the verb name and fails when Add, Checkout or Pull leaves its precondition field empty, where it used to skip the check.
  It runs in every cell of those verbs, so the test that built a hub per verb to check the same thing is gone.
- **`TestStates` fold.**
  `States` wraps each structural state's `Apply` so that the first apply of the state at each anchor fails when the hub manifest is unchanged.
  Every caller of `States`, `TestCrossProduct` included, gets the vacuity check.
  The check runs once per state and anchor, not in every cell, because a manifest capture hashes every file of the hub.
  The dirtiness and structural states already assert their own establishment inside `Apply`.
  `TestStates` shrinks to `TestCleanHubIsClean`, the clean-hub baseline at the nested `backend` anchor, which the cross product does not check.
- **Anchor loops.**
  `TestWireJunctions_ScratchLink`'s three anchor loops now run only at the nested `backend` anchor.
  Its sibling rows and `TestReconcile_ScratchLink` still run at the root.
  `TestCommitWeft_MachineLocalArtifactsNeverEnterWeftTreeAtAnyDepth` keeps only its `wts/some-task` row, the one that needs `hubforge.NewHub`.
  It is renamed `TestCommitWeft_MachineLocalArtifactsNeverEnterWeftTreeAtNestedAnchor`.
  Each kept row makes every assertion its loop made.
- **Tier 0 moves.**
  Each pure step is split out of the function that runs git, with no change in behaviour:
  - `entriesFromWorktrees` from `List`;
  - `parseTrackedDirtyPaths` from `trackedDirtyPaths`;
  - `excludeLinesToStrip` and `stripExcludeLines` from `unseedGitExclude`.

  New tier 0 tests pin them in `worktreelist_parse_test.go`, `dirtiness_parse_test.go` and `unwire_exclude_test.go`.
  `TestList` loses its single- and two-worktree rows, and its bare-repo row becomes `TestList_RejectsBareRepository`.
  `TestUnwire_LeavesSiblingWorktreeUndirtied` keeps only the sibling `git status` and junction checks.
  `TestSourceDirtyPaths` stays as it was: its rows assert what git reports and the refusal `Add` builds inline.

## Tier 1 time

Measured with `go run ./cmd/testtiming -resources -full -pkg ./internal/fabricengine`, which runs the package's tier 1 tests in one gate slot:

| | Wall | CPU | Git processes |
| --- | --- | --- | --- |
| Before | 45.72 s | 163.69 s | 72838 |
| After | 48.27 s | 176.19 s | 72334 |

The difference is within machine noise.
The one-minute load average rose from 2.2 to 5.9 during the runs, and the git count swings by about 1,400 between runs of the same tree.
The saving the study priced for the greedy set depended on dropping tests the module-wide check does not confirm.
The folds and cuts made here remove a few hub builds, which this measurement cannot separate from the noise.

## Design decisions

- The time is measured with `cmd/testtiming -resources` rather than in a `lym gate test` pass, because the writer's instructions bar running `lym gate test`; the tool holds a gate slot and runs the same tier 1 tests.
- A greedy-set test is dropped only when the module-wide check confirms it, as the board entry says.
  That leaves five candidates, and reading each showed an assertion its covering test lacks.
- `docs/benchmarks/test-redundancy.md` is not refreshed.
  It was written under the old `integration,tmux` tags, so a `-pkg` run refuses to splice into it, and regenerating the whole report is outside this change.
