```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling cards, decisions, test rows and cross-references they touch.

**[MEDIUM] Card 12 names `runBatch` and `Run` as the only places that set `st.Stop`, but the resume path also stops the row**
- **Changed text:** card 12, the new bullet "Each batch stop, needs-parent stop and verify stop sets `st.Stop` … `runBatch` sets it for a batch or needs-parent stop, `Run` for a verify stop."
- **Problem:** card 13, `resumeStrands` step 4, also ends the row: "a failure on the last rung, or a `needs_parent` attempt, stops the row as `runBatch` does."
  That stop is judged in `resumeStrands` from a recorded attempt, and no attempt is run there.
  The new sentence gives that stop no owner, so it leaves open the same "unowned write" F5 set out to close.
  If the resume's stop saves no `Stop`, the next call skips `reenter`.
- **Evidence:** card 12's new F5 bullet; card 13 line 40, "stops the row as `runBatch` does", which the fix did not touch.

**[MEDIUM] A crash during a verify re-entry can mark a second `Reentry` fixer and reset the lost-fixer count**
- **Changed text:** card 12, "That allowance counts the lost fixers on the rung from the latest `Reentry` fixer on … read from the fixer records."
  Also card 13, "a failure gets one fixer on the last rung, marked `Reentry`".
- **Problem:** a verify `StopRecord` is cleared only when the verify re-run passes (card 13 line 56), and a verify re-entry passes through no batch begin.
  Take a crash after a lost re-entry fixer: the next call finds `Stop` still set, runs `reenter` again, and spawns a new fixer marked `Reentry`.
  The count then restarts, giving a second same-rung retry.
  The text says the allowance is read from records precisely so a crash never resets it.
  That breaks D6's bound of one same-rung retry per attempt (decision-record.md line 113, line 128).
  Batch re-entries avoid this only because card 12's begin clears `Stop`, which no verify re-entry passes through.
- **Evidence:** card 13 line 29 ("calls `resumeStrands` and then, when `StopRecord` is set, `reenter`") and line 56; card 12, "A begin also clears any `StopRecord`".

Tool calls beyond my context:
- I grepped cards 10, 11, 14, 16 and 22 and listed the discussion directory, to find sibling text on retry, outcome, summary and stop that the fix's changes reach.
- I grepped card 10 for `checkCleanTree` and the decision record for D6, D13 and the re-entry retry wording, to check the new retry and outcome text against them.
- I read websterengine's `DirtyPaths` and card 10 lines 26–34, to check whether removing `outcome.yaml` and `summary.md` at entry trips the clean-tree check.
  It does not add a new conflict: the uncommitted state saves between a begin and `endAttempt` already put `_lym/weaver/` changes in the same position.
