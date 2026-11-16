```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: does the changed text agree with its sibling cards, decisions and cross-references?

Two LOW findings.

1. LOW. Card 3 `ImpactSummary`, against the new `Runs` union text in card 3's Intent.
   The summary is stale: it still says "behaviour changes only in the side order, the error a deferred or ended records side now returns, and the teardown wait".
   The fix adds a fourth behaviour change. `recordPushFailure` now unions `Runs` across failures instead of replacing the record whole, and card 3's Edit list now names `recordPushFailure` and `RecordPushFailure`.
   The Intent states it: "`recordPushFailure` unions an existing record's `Runs` with the new failure's in place of replacing them".
   That contradicts "only" in the summary.

2. LOW. Card 6, the added sentence "`KindWorktreeReset`'s doc names this keep reset beside `resetHardTo`'s hard one."
   The doc it plans would still not match the kind's real uses.
   `destroy.go:2277` already appends `KindWorktreeReset` for a third reset (`rec.Append(KindWorktreeReset, weftPath, "HEAD")`, documented at `destroy.go:2222`).
   A doc naming only the two executors stays incomplete.
   The sentence also turns the doc into a list of current consumers, which goes stale on the next new caller.
   Better: the doc describes the kind as "a worktree reset" without naming executors.

Checked and consistent:
- The `Warp-SHA` decision text in the overview agrees with card 8's commit bullet, card 12's `ReturnToPreviousTransition`, card 14's pattern line and cards 6 and 7.
- `Runs` is the only name in cards 3 and 7 and the overview; no `Run` field is left in cards 1, 9 or 13.
- `resetKeepUnderArchive` is described the same way in cards 6 and 11.
- The `updateRemoteBranch` request shape matches in cards 6 and 7.
- `OriginBattenStatus` is used only in card 6.
- Card 9's `TestPrePushCheckGuardedSeams` does not overlap the new card 8 test edit.

Tool calls:
- A grep of every plan card except card 8 for `Warp-SHA`, the trailer, `Run`, `Spawn`, readopt and `CommitFailed`. Context held cards 9, 12, 13 and 14 only in part or not at all.
- A grep of the plan for the renamed or added symbols (`OriginBattenStatus`, `FailedPushesOwedBy`, `resetKeepUnderArchive`, `KindWorktreeReset`, `RecordPushFailure`), to find twins in cards I had not read.
