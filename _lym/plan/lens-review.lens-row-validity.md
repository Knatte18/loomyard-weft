```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. Checked whether each planned test row reaches its branch and would fail if the code were wrong, and whether any new test repeats an existing one.

**Findings**

1. **MEDIUM**, Card 9 tender row (`TestPauseAndResume_WriteLiftAndRefuseOverTheHub`). The step "the intent lands on the next pause once origin is restored" cannot be reached.
   - A repeated pause answers `already_paused` (`internal/tendercli/cli_test.go:140-143`), so `Record` never runs again and nothing pushes the owed intent.
   - The existing pauses are hub-scope (`cli_test.go:125`, `pause` with no slug). `commitRunIntent` only acts on a slug that has a batten record. The card names neither the slug nor the batten record the row needs.

2. **MEDIUM**, Card 9 tender row, PATTERN-test-speed "No test waits out a production duration". The offline row runs `PushRun` against a missing origin.
   - `PushRun` has no way to inject a backoff sleep.
   - `pushJob.awaitOrigin` and `settleRejection` sleep through `gitrepo.PushBackoffDelays()` with `time.Sleep` when `opts.BackoffSleep` is nil (`internal/fabricengine/pushclassified.go:443-446`).
   - The delays are 1s, 2s and 4s (`internal/gitrepo/pushretry.go:23,26`), so the row sleeps at least 7s of real time per side.
   - Fix: give `RunPush` a backoff seam, or drive the offline branch at tier0 through `runIntents` fakes.

3. **MEDIUM**, Card 8 and card 9, D1 "`lym batten cancel` acts here at once and owes its push". `commitCancelRecord` now pushes through `Push`, but no card says it treats an owed push as success.
   - Card 9's owed row covers only `PushRecords` after the lifts.
   - The unreachable-origin rows of `TestCancelRun` (`internal/battencli/cancel_test.go` ~971-1050) list `SpawnPush` in `noCalls`. They do not reach the intent commit's owed push.
   - So the "cancel offline owes its push" branch is neither planned nor tested.

4. **MEDIUM**, Card 6 `TestHandOverTips` row "a tip whose transition was given up between the read and the push is reset off origin". Without a hook between `pushOwedCodeTip`'s read and its push, the test cannot change origin's records branch at that point.
   - No hook var is declared. Compare the existing hook pattern `detachedPushHeldHook` (`internal/fabricengine/teardownlock.go:28`).
   - So the reset branch D2 requires goes unreached.

5. **MEDIUM**, Card 4 `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` new row (D1 flagged code side). The card does not name the lock holder.
   - In that test the lock is held raw (`internal/fabricengine/pushanchored_integration_test.go:371`), not by `coalesceFlagged`, so no respawn path runs whatever the code does.
   - "Origin's code branch unchanged" therefore holds even if `respawnForeignFlags` stayed.
   - To fail when the code is wrong, the holder must be a real push of the records side alone, such as `CoalescePushRequest` with a records-only request, which takes and releases the lock.

6. **LOW**, Card 4 `TestPushDeferral_PutsBackTheUnpushedFlags`, test economy and reach.
   - The new row "deferral before the records side landed puts back both flags" repeats the existing `NeitherPushed` row (`internal/fabricengine/coalesce_test.go:155`).
   - Both new rows feed `pushDeferral` a hand-built `pushed` map. They never reach the change to how `coalescePushSides` marks `pushed`, so neither fails if that change is wrong.

7. **MEDIUM**, Card 11 `Deps` gains four seams (`OwnedElsewhere`, `HandOver`, `PushRecords`, `ArchiveParentMerge`). The card states no meaning for a nil value.
   - The Finalize tier0 tests build `Deps` by hand.
   - `TestFinalize_LockFirstAndFastForward`, `TestFinalize_LandCheck`, `TestFinalize_PullRequest` and `TestFinalize_RecordLandingFailureKeepsDone` are not edited. A nil call in them would panic.
   - Fix: state the nil meaning (no owner check, no push, no archive), or list those tests.

8. **MEDIUM**, Card 12. Only `settleCodeTip` is tested.
   - Nothing tests `DobbyRuns` calling it "for a run this machine owns", or the decision to leave a run out of the tick on failure.
   - `DobbyRuns` (`internal/battencli/dobbyreads.go:151`) takes no `codeTipDeps` seam. Its only test caller is `internal/beamcli/dobbypairanchor_test.go`, which the card does not list.
   - That test will now run `hubCodeTipDeps`' real reads and fast-forward. The card neither lists nor checks it.

9. **LOW**, Card 12 `TestReturnToPreviousTransition`. No row reaches "a records branch with a single transition returns false".

10. **LOW**, Card 11 `TestArchiveParentMerge` repeats card 6's `TestHandOverTips` row "a parent branch ahead by this run's merge alone is tagged and reset level". Both reach `archiveParentMerge` on the same input. Keep one: the `ArchiveParentMerge` test, and drop the `TestHandOverTips` row.

11. **LOW**, Card 6 `TestPushRun_HandsOverOnATakeover` row "an entry naming a former name hands nothing over and lands the push". It repeats card 5's `TestRejectedRecordsPush_ReadsTheTakeover` former-name row through one more layer. The only new thing it covers is `ThisMachine` reading the machine file, which needs process-global env and so can take no `t.Parallel`. The card does not say so.

12. **LOW**, Card 7. The owner's-guard row is placed in `TestOriginCodeTip`, but it drives `PushPairAnchored` and `pushJob.resetUnnamedTip`, which is a different surface. It belongs as a row of `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`.

13. **LOW**, Card 5 row "a mark naming the entry: `ErrHandedOver`". The mark's writer, `WriteHandOverMark`, only arrives in card 6, so card 5's test has to write the JSON by hand at `HandOverMarkPath`. The card does not say this, and no helper is listed.

14. **LOW**, Card 10 `TestBoard_SetStatusPushed` "no sync child was spawned". The detached board sync is a no-op under `testing.Testing()` (`internal/boardengine#SpawnSync`, spawn.go:22-29, per the board lookup). The assertion therefore passes even if `SetStatusPushed` still spawned the sync.

15. **LOW**, Card 13. `NewDobbyRunCells` will default its verdict seam to the real `pairteardown.ReadRun`.
    - Other tier0 tests construct it at `internal/battencli/overview_test.go:432`, `:464` and `:503`. Only `TestDobbyRunCells_ReadsTheRunsFromDisk` is listed.
    - Those tests would run the real reader at tier0, which may spawn git once a board opens. Either list them and set the seam in each, or state that the default is safe at tier0.

16. **LOW**, Card 8. The card states no meaning for a nil `Admit`, `Owed` or `Sleep` in `statuscommit.Deps`, or for a nil `Push` in battencli's `commitStatusDeps`. `TestCommitCancelRecord` and `TestCommitRunRecords` build these by hand.

**Tool calls**
- Read `landingdeps_markdone_test.go` and the `DobbyRunCells`, `TestStatus_JSONRoundTrip` and `TestCommitRecordsVerb_ErrorEnvelopes` sources: needed their fixtures and tiers, which the context lacked.
- Read the markdone fixture and the `SetOwners`/`Bolt.CommitWritten` skip paths: needed to know whether the tier0 markdone test would spawn git.
- Read the push backoff and the `realSeamFixture` and tender test origins: needed the backoff timing for the offline rows.
- Read the backoff constants, `TestPauseAndResume_WriteLiftAndRefuseOverTheHub`'s opening and `realSeamFixture`: needed the delay values and the tender row's pause shape.
- Read the `NewDobbyRunCells` test callers, the `DobbyRuns` signature and test, and `TestPushDeferral`'s rows: needed to check the seam callers and whether the deferral rows repeat existing ones.
