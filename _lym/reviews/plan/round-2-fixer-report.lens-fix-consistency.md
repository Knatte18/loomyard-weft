```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text still agrees with the sibling cards, the shared decisions and the cross-references.

**LOW: card 6's ImpactSummary still calls the wrapper a re-export**
- Changed text: card 6's Callers paragraph (`_lym/plan/06-remove-archive-order.md:60`) and its re-signed Edit entry at line 22. Both now say `ArchiveWeftTipForTest` "becomes a wrapper function in `export_test.go` in place of today's re-export var".
- Stale sibling: the same card's ImpactSummary (line 92) still lists "the test re-export and its four tests" among the callers of the re-signed archive helpers.
- Evidence: line 60 says "wrapper function … in place of today's re-export var"; line 92 says "callers (`removePair`, `Cleanup`, `archiveDiscardedTips`, the test re-export and its four tests)".

**Twins checked and found consistent:**
- **Card 10:** no other card names `StartFirstPass`, `Command` or `TestStartFirstPass` as setting `readopt`. Card 10's ImpactSummary was updated to name `armAt`. Card 10's tier1 note now credits the standing-records-worktree and beam-anchor criteria to card 4's `TestAdd_AdoptOnly` rows, and both rows exist (card 4, lines 67 and 75).
- **Card 13 and the overview:** the D3 decision now lists `judgeOrigin` and `stoppedReason` as card 13's Intent and Edit list do. The adopter-first exception still holds, because `adopterFirstRepair` is a separate stop from `stoppedReason`.
- **Card 13's re-signed helpers:** the landingshed tests call none of `reportVerdict`, `stoppedReason` or `reconcileVerdict` directly. So "re-signing three private verdict helpers whose callers are on this card" holds, with `Finalize.ownerChecks` and `ParentAlign.Call` listed.
- **Card 4 and card 3:** card 4's new drop bullet agrees with card 3's line 29, which defers the adopted-warp case to card 4, and with card 4's last test row.
- **Card 8:** the new `cleanupAfterEnumerate` field and `SetCleanupAfterEnumerateForTest` sit beside card 6's `archiveSleep` and `SetArchiveSleepForTest` with no conflict. Card 8's ImpactSummary was updated.
- **Card 5:** the parallel wording and the new `TestRunCLI_PushSkippedWhileTeardownLockHeld` match its Create entry and the `push_skipped` bullet.

Further calls:
- A grep of `_lym/plan/*.md` for the changed symbols and phrases, to find twins in cards my context had not fully read (cards 1, 2, 7, 9, 11, 12, 14).
- A grep of the landingshed tests for direct calls to the re-signed helpers, and of the plan for the new hook and field names, to check card 13's "callers are on this card" claim and card 8's field against card 6's. My context had neither.
