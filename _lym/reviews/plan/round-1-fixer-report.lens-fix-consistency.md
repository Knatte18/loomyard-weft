```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the round-1 fixer's change set agrees with the cards, rows and cross-references around it.

**MEDIUM: the new `Kept` condition has no parameter to reach `movedRun` or `takeOverRuns`.**
- Changed text: card 9's Intent, step 2: "and, when `WriteAdopt` kept this machine's hold (`Kept`), the hold's `FixTask` are not moved … Under a take-hold rewrite (`Kept` false…) the fix task's run is judged as any run".
- What's wrong: the not-moved judgement belongs to `movedRun`, and `takeOverRuns` calls it. The Create signatures of both were left as they were. `movedRun(hold, registry, identity, subject, slug, takeover)` and `takeOverRuns(deps, hold, owners, identity, adopted, rootAdopted, outcomes)` have no `Kept` or `AdoptWrite` parameter, so neither can tell a kept hold from a take-hold rewrite.
- Evidence: card 9's Create lines 6–7, unchanged beside the new Intent sentence.

**LOW: card 9 says the adopt names a machine even when no machine holds the hold.**
- Changed text: card 9, steps 7 and 8: "A result with `HoldLost` (another machine took the hold during the check, or it cleared) ends the adopt naming that machine".
- What's wrong: card 8's step 3 sets `HoldLost` also when `ClearHold` reports false, and then its line says "the hold cleared meanwhile". In that case there is no machine to name. Card 2's no-hold `FixHold` refusal is the same case.
- Evidence: card 8 step 3, "or saying the hold cleared meanwhile"; card 8 step 4, "which names the other machine when one took the hold".

**LOW: card 8's new `failedEnd` signature uses a plan type that card 8's Uses list omits.**
- Changed text: card 8's Edit entry `failedEnd` -&gt; `(fixTask string, lines []string, lost *boardengine.HoldRefusal)`.
- What's wrong: card 8's Uses names `plan:internal/boardengine#FixHold` and `#ClearHold` but not `plan:internal/boardengine#HoldRefusal`. Card 8's new step 3 and step 4 text and the `Fix` seam also rely on that type. Card 9 lists the type in its Uses, but card 8 lands first. `lym loom validate-plan` passed, so this is a gap in the dependency record only.

**LOW: `failedEnd`'s new third return has no `TestFailedEnd` assertion.**
- Changed text: card 8, "`failedEnd` … returns … and the `*HoldRefusal` `FixHold` returned", and the `TestFailedEnd` sentence.
- What's wrong: the new rows in `TestRunHoldCheck` reach the refusal only through a fake `Fix` seam. `TestFailedEnd` was edited in this round but gained no assertion that a refusing `FixHold` comes back as `lost`, so the real path that passes the refusal on is untested.

**LOW: card 3's fetch step was not widened when the archive step was.**
- Changed text: card 3, step 2, which now archives "over the moved pair and the parent's pair".
- What's wrong: `ArchiveOwnerTips` archives both the code tip and the records tip of every pair it is given. Step 1 still fetches only "the pair's code and records branches and the parent's records branch". The parent's code tip is therefore judged against a remote-tracking ref that was never fetched. It can be stale, so a tip origin already holds may be read as missing and given a needless archive tag.
- Evidence: the `ArchiveOwnerTips` doc, "A pair's tips are the branches its code worktree and its records worktree have checked out"; card 3 step 1.

I made no further tool calls; everything I checked was already in my context.
