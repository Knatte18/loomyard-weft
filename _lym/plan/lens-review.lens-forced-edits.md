```yaml
lens: forced-edits
outcome: reported
```

Forced-edits review of the 12-card plan for part 2 of the fabric restructure.

1. BLOCKING — Card 4 (`origin-reads-seams`): re-signing `originStatusAtTip` breaks three callers that no card lists.
   - Card 4 re-signs `internal/fabricengine#originStatusAtTip` to return `[]byte`. It lists only `OriginRunStatus` and `readCodeTipAt` as callers.
   - `internal/fabricengine/handover.go:200` (`OriginParentRunStatus`) and `handover.go:215` (`OriginBattenStatus`) return the result as `shedengine.Status`.
   - `internal/fabricengine/takeover.go:95` (`ReadUpstreamTakeover`) reads `status.Takeover` from the result.
   - The build fails, and `caller-uncovered` fires.
   - These callers belong to part 4's concern (D9). The fix is either to list them on card 4, or to leave `originStatusAtTip` intact and add a new bytes reader for this part.
   - Swapping its `shedrun.TreeRunFileRels` for the `RunRecordPaths` seam also puts the unset-seam case on those part-4 verbs. Card 4 refuses that case only in `OriginRunStatus` and `OriginCodeTip`.

2. MEDIUM — Card 12 (`board-test-moves`): the moved and rewritten tests depend on helpers that only some cards list.
   - The moved boardengine tests rest on `internal/fabricengine_test` helpers that the external package `boardengine_test` cannot reach:
     - `originHub`, `newOriginHub`, `originPair`, `writeStatus`, `publishBoard`, `originBoard`, `writePairStatus`, `endedTemplate`, `arrangeEnded`, `endedHub`, `movesTemplate`, `entryOf`, `requireEntry`, `kindCount`, `callLog`;
     - `newFailureFixture`/`newFailureFixtureAt`, `pairFixture`, and the `recordsFixture` methods `boardBranch` and `endEntryOnOrigin`;
     - `boardEntry` and `withBoardStatus` in `replay_integration_test.go`;
     - the `export_test.go` shims, such as `CodeDivergedWayForwardForTest`.
   - Card 12 declares no `Create` handle for their boardengine copies.
   - In fabric, it lists only `movesPair` among the helpers that hold the imports it says go:
     - `writeStatus` uses `shedrun.StatusFile`;
     - `newFailureFixture` calls `boardkit.InstallReplay`;
     - `endedTemplate`, `originBoard` and the `replay_integration_test.go` helpers use `boardengine` types.
   - So "this concern's fabric test files lose their board and shed imports" cannot hold as listed. Those helpers need `Edit` or `Delete` glyphs on card 12.

3. LOW — Card 9 / card 11: an allowlist reason goes stale and is listed only under `Uses`.
   - `cmd/lym/remoterefslock_test.go:22` gives the reason for `fetchMovedHeads` as "runs inside judgeAgainstOrigin, whose caller holds the repository's remote-refs lock".
   - Card 9 moves the fetch into `reconcileReader`, under `runRule`'s phase lock, so that reason becomes false.
   - Card 11 claims the entry "stays true" and lists `cmd/lym#remoteRefsLockFetchAllowlist` under `Uses` only. Card 9 should list it under `Edit` and reword the reason.

4. LOW — Card 7 (`push-side-core`): a source scan pins `goneEntryEnded` to its file.
   - `internal/fabricengine/boltreplay_test.go:178` requires `goneEntryEnded` to be in `pushclassified.go`.
   - Card 7 deletes its only caller, `awaitOrigin` (`pushclassified.go:812`), and moves the origin read into `pushJob.read`. It does not say that `goneEntryEnded` stays in that file.
   - It also does not list `internal/fabricengine#TestBoltReplay_FabricNamesNoBoardFile`. If the reader moves to another file, that scan's readers map is stale.

5. LOW — Card 10 lists `internal/fabricengine_test#TestReconcileOrigin_Ended` under `Uses`, and the later card 12 deletes it. That is `uses-later-target`, which the gate flags; the fix is to drop the `Uses` entry.

6. LOW — Card 7: the class order changes for one error, and `TestRunRule` is not checked against it.
   - Moving `rejectionRetryKind`'s order into `outcomeClassOf` (`internal/fabricengine/executor.go:168`) reclassifies `gitrepo.ErrPushRetriesExhausted` from failed to transport.
   - `runRule` records a read failure for the rule only when it is a transport failure, so a read wrapping that error would now be recorded instead of returned.
   - `TestRunRule`'s read-error rows (`executor_test.go`) are not named for a re-check.

Tool calls:
- A grep of `internal/fabricengine` for callers of the deleted and re-signed members. My context had only the callers outside fabric and the concern files themselves.
- `lym quarry resolve` over every existing glyph the cards list. My context had not confirmed their spellings; all of them resolved.
- The hubforge entry points, the `handover.go` and `takeover.go` caller bodies, `pushlockholder_test.go`'s holder use and `remoterefslock_test.go`'s scan shape. I needed them to judge the forced edits.
