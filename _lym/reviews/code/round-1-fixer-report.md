# Round 1 fixer-report

## Changes

### F1 — fixed (as proposed, with one fold)

Restored from f2ac2bf7, each with a `testtiming:keep` line naming the tag it pins.
All edits are in `internal/fabricengine`.

- `remove_archive_integration_test.go`: `TestRemove_ArchivesWeftTipBeforeTeardown`, `TestRemove_ReusesSameTipArchiveTag`, `TestRemove_ForceStillArchives`, `TestAddRollback_LeavesNoArchiveTag`; the header sentences for these were restored.
- `remove_siblingdirty_integration_test.go`: recreated with `TestRemove_UntrackedDriveReportIsCommittedAndArchived` only.
- `remove_records_integration_test.go`: `TestRemove_OutOfPathspecSiblingDirtRefusesBeforeAnyMutation`.
- Folded, not restored separately: `TestRemove_RemoteFalseStillPushesArchiveTag` is now the `RemoteBranchDeleted == false` assertion in `TestRemove_ArchivesWeftTipBeforeTeardown`.
  Both made the same `Remove(l, slug, false, false)` call on the same hub and asserted the same origin tag, so the happy-path test already covered it (the fix-proposal lens's finding).

Also restored, as members of the same set found by the self-check:

- `TestRemove_DirtyRefusalLeavesPairIntact` and its helper `assertPairIntact` (no tag after a refused Remove), in `remove_refusal_remedy_integration_test.go`.
- `TestCleanup_PrefixSetOverBareSlugOrphan` (tag name under `archive/<slug>/`), in `cleanupremote_integration_test.go`.
- `TestRemove_NoOriginSkipsArchiveAndCompletes` and `TestCleanup_NoOriginDeletesAndReportsArchiveSkip` (no tag plus a reported skip reason).

Gate repair: the header sentence in `remove_archive_integration_test.go` reached 120 characters, so it is now split into two sentences.

Commits: 9e969b84 (the seven), b8308134 (the fold and the four additions), then a commit for the gate repair.
`go vet ./internal/fabricengine` is clean, and `lym gate test --tier 1 ./internal/fabricengine` passed with 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: MEDIUM, redundant `RemoteFalse` test — fixed as proposed (folded).
- fix-affected-set: MEDIUM `TestRemove_DirtyRefusalLeavesPairIntact` — fixed (restored).
- fix-affected-set: MEDIUM `TestCleanup_PrefixSetOverBareSlugOrphan` — fixed (restored).
- fix-affected-set: LOW `TestRemove_NoOriginSkipsArchiveAndCompletes` and `TestCleanup_NoOriginDeletesAndReportsArchiveSkip` — fixed (restored).
- fix-affected-set: LOW `TestRemove_ReturnsTheRemoteStepFailureOverAKeptDirectory` — not fixed, not disputed: its removed lines only inject a tag-push failure and assert the returned error and `StrayPath`, with no tag assertion, so it is outside the tag-assertion set (cited from the removed text of that test).

## Disputed

None.

## Deferred

None.

## Rulings

None.
