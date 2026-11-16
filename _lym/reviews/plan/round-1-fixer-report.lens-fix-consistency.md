```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 fix set agrees with the sibling cards, decisions and rows it touches (fix-consistency lens).

**MEDIUM: card 8's new `ErrPushLockBusy` sentence contradicts card 4.**
- **Changed text:** card 8 (`08-transition-sync-push.md`), the sentence added for F15: "…and the holder lands the flagged sides on release."
- **Evidence:** card 4 retires the foreign-flag respawn. Its Intent says "`coalesceFlagged` no longer respawns a flag of a side the holder does not owe. Such a flag stays on disk for its worktree's next push". Card 4's `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` row has a records-only holder leave `pending.code` unpushed, with "origin's code branch stays unchanged".
- **Summary:** a holder lands only the sides it owes. The flagged code side of the busy seam's pair stays on disk until the seam's own retry, so it is not landed on release.

**LOW: card 1's `PushOwed` doc still names two causes of an owed push.**
- **Changed text:** the overview's `owed-is-a-failed-push-record` decision and cards 7 and 8 now count class `deferred` as owed (F2).
- **Evidence:** card 1's Intent (`01-runfsm-push-owed.md:18`) still says `Inputs.PushOwed` is "true while a push of the run is owed, for an unreachable origin or a held teardown lock alike (D1)". Its doc comment "says so".
- **Summary:** the deferred records side, which ReadRun now reads as owed, is missing from the field's stated meaning.

**LOW: card 12's new bullet does not say whether it applies inside the grace.**
- **Changed text:** card 12 (`12-code-tip-grace.md`), the bullet "still missing and the local code branch holds the tip: it pushes the tip through `PushRun`…".
- **Evidence:** the bullet comes after "still missing and the transition younger than `runfsm.CodeTipGrace`… nothing; the run waits". The bullet names no grace condition, and its `TestSettleCodeTip` row says "past the grace".
- **Summary:** a reader cannot tell whether a tip this machine holds is pushed during the grace or waits like any other.

Tool calls:
- One grep over `_lym/plan/0*.md` and `1*.md` for the changed terms (`transport`, `teardown_lock`, `OriginRunStatus`, `machinename.Names`, `endRunPhase`, `deferred`, `code_tip`, `PushRecords`). My context lacked cards 10, 13 and 14 and needed every twin site of those terms.
