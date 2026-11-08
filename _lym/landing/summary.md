# Cut and fold redundant fabricengine tier 1 tests

The coverage study in `docs/research/speed/fabric-test-redundancy-2026-11-08/report.md` found that many `internal/fabricengine` tier 1 tests cover no block another test does not.
This change cuts and folds those tests, following the report's "For the later task" section.
Block coverage does not see assertions, so a contained test stays when it guards data the coverage tool cannot see.

## Test changes

- Folded the two `TestCleanup_*` tests onto one hub.
- Folded `TestVerbCases_ArrangeSetsItsPreconditionField` into the shared check it supports, which now fails on an empty field.
- Moved `TestStates`' vacuity check into the fixtures' own apply path.
- Cut the anchor loops of `TestWireJunctions_ScratchLink`, `TestStates` and the weft machine-local-artifacts test to the rows a nested anchor needs.
  `scratchlink_integration_test.go` now runs each row once at the subpath anchor `backend`; the repository-root anchor is covered by the other rows.
- Dropped the tests the module-wide `-redundancy` check confirmed as contained and that assert nothing coverage cannot see.
- Marked kept contained tests `//testtiming:keep` with the reason, for example:
  `Healthy` returning a healthy verdict on an unborn weft branch;
  a second `UnwireJunctions` returning an empty result;
  `ChangedPathsBothSides` returning a non-ASCII path raw;
  `PushWarpRebaseFreeAt` recording `KindBranchPushed` and leaving no push-lock file.

## Pure logic moved to tier 0

Three tests asserted pure logic through git.
Each now has a tier 0 test over a small production seam, and the git test stays only where git is the point.

- `TestList`: `worktreelist.go` splits `entriesFromWorktrees` out of `List`; tests in `worktreelist_parse_test.go`.
- `TestSourceDirtyPaths`: `dirtiness.go` splits `parseTrackedDirtyPaths` out of `trackedDirtyPaths`; tests in `dirtiness_parse_test.go`.
- `TestUnwire_LeavesSiblingWorktreeUndirtied`: `junction.go` splits `excludeLinesToStrip` and `stripExcludeLines` out of `unseedGitExclude`; tests in `unwire_exclude_test.go`.
  The git test remains and asserts the unwired pair's anchored exclude line is stripped.

These splits are the only production changes.
They move code without changing behaviour.

## To check by hand

- Tier 1 time for `internal/fabricengine`, before and after.
  The task asks for one gate pass each, stated in this description; no measurement was recorded in the run, so run both and compare.
- Run `go run ./cmd/testtiming -pkg ./internal/fabricengine -redundancy` and confirm that no dropped test left a block uncovered.
- Review the `//testtiming:keep` reasons for contained tests that guard archive tags, origin deletes, user content, mutation records or machine-local files.
