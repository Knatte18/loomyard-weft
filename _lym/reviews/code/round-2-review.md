---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:10
    summary: The change description's dropped and kept counts and its kept list predate the round 1 fix commits and no longer match the diff.
    fix: In _lym/landing/summary.md recount against the final diff and update the Numbers section (dropped now 11 fewer than 198, kept now 65 not 54), and add the eleven restored or folded archive-tag tests (TestRemove_ArchivesWeftTipBeforeTeardown, TestRemove_ReusesSameTipArchiveTag, TestRemove_ForceStillArchives, TestRemove_NoOriginSkipsArchiveAndCompletes, TestAddRollback_LeavesNoArchiveTag, TestRemove_OutOfPathspecSiblingDirtRefusesBeforeAnyMutation and the other tag tests the fix commits marked) with their reasons to the kept list.
---

### [LOW] Change description counts predate the round 1 fix

**Location:** `_lym/landing/summary.md:10`

**Issue:** The summary says 198 tests were dropped and 54 were kept.
The diff over the review range adds 65 `//testtiming:keep` lines, 11 more than 54.
Fix commits `9e969b84` and `b8308134` restored or folded those tests after the summary was written.
The kept list in the summary names none of the restored archive-tag tests, such as `TestRemove_ArchivesWeftTipBeforeTeardown` and `TestRemove_ReusesSameTipArchiveTag`.
The summary is the writer's claim about the diff, and the diff no longer bears out its numbers.

## Focus checks

- No test removed over the range asserts an archive tag without a kept test pinning it.
  I scanned every removed line for the tag helpers (`tagTargetAt`, `archiveTagsAt`, `showAtTag`, `requireArchiveCovers`), `ArchiveTag`, `refs/tags`, `ls-remote` and `--tags`.
  The only removed archive-tag assertion is in `TestRemove_RemoteFalseStillPushesArchiveTag`.
  It is folded into `TestRemove_ArchivesWeftTipBeforeTeardown`, which now passes remote=false and asserts `RemoteBranchDeleted` is false next to the tag target.
- `TestAdd_ArchivedAncestorWeftLeftoverReplaced` used `requireArchiveCovers` only as setup before asserting the origin weft branch.
  The kept `TestAdd_DivergedArchivedWeftLeftoverReplaced` pins the archive tag covering the old tip.
- `TestRemove_ReturnsTheRemoteStepFailureOverAKeptDirectory` used a rejected archive tag push as a stimulus and asserted the error and `StrayPath`, not a tag.
  `SetArchiveTagPushHookForTest` is still used by `TestRemove_ArchiveOrder`.
- Every tag-pinning test the diff touches carries a `testtiming:keep` line naming the tag it pins:
  in `remove_archive_integration_test.go`, `remove_records_integration_test.go`, `remove_finish_integration_test.go`, `remove_refusal_remedy_integration_test.go`, `remove_siblingdirty_integration_test.go`, `pairbranch_archive_integration_test.go`, `cleanupremote_integration_test.go` and `add_remoteleftover_integration_test.go`.
- `archiveTagsAt` and `showAtTag` exist in `remove_archive_integration_test.go` and `remove_records_integration_test.go`, `tagTargetAt` in `archive_integration_test.go`, and `go vet ./internal/fabricengine` reports nothing.

## Scope against the record

- Every removed top-level test is in `greedy.tsv`.
- Every `greedy.tsv` test still present carries a keep line, except the four the summary says the first cut folded, `TestWireJunctions_ScratchLink`, `TestRemove_UnlinksScratchLinkBeforeWorktreeRemoval`, `TestUnwire_LeavesSiblingWorktreeUndirtied` and `TestSourceDirtyPaths`.
  The record says to skip tests the first cut already dropped or folded, and the summary records the parent confirming those four.
- `export_test.go` loses only two shims, `WorktreeDirtyTrackedForTest` and `WorktreeDirtyAllForTest`, and the diff changes no production code.
