```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for agreement with sibling cards, decisions and cross-references (fix-consistency).

MEDIUM — card 16 `driftVerifyEntry` seam list (line 131), against card 15's new `Deps` fields
- **Summary:** Card 15 now adds the `WeftChangedSince` func seam, which `JudgeSnapshot.Changes` calls on every judge return and resume. Card 16's entry still checks only `Checkouts` and `JudgeTable` among the judge seams, so a nil `WeftChangedSince` slips past the entry check.
- **Evidence:** card 15:61 adds `WeftChangedSince func(rev string) ([]string, error)` and card 15:62 has `battencli`'s `intentionallyNil` list "the three func seams". Card 16:131 reads "checks the seams `Config`, `Status`, `Head`, `CommitWithTrailers`, `TrailerCommit`, `FileAt`, `CaptureCurrent`, `JudgeTable`, `Checkouts`, `EndRoles` and `Compare`".
- **Twin left unchanged:** the list should gain `WeftChangedSince`, and `TestDriftEntries` should cover it.

LOW — card 10 `TestCheck` row "a record with no `awaited_at` reports `BaselineFault` saying the capture never counted"
- **Summary:** The new step 3 skips any row before Drift-Expect before the `awaited_at` test, so this row only reaches `BaselineFault` when its status is past Drift-Expect. The row does not say so, but the companion row the fix changed now pins `awaited_at` unset before Drift-Expect.
- **Evidence:** card 10 step 3 reads "A current row not past Drift-Expect … sets `Skip` … before any baseline check". The test row "a record with no `awaited_at` reports `BaselineFault`" names no status row.

Every other changed piece agrees with its siblings:
- Card 16's and card 25's Delete lists now match.
- Card 23's doc edits touch the driftengine and shedrecipe docs, not the loomrecipe doc card 16 now edits.
- Card 12's `Unguarded` seal trailer still holds under card 8's narrowed `Unguarded`.
- Card 14's "before Drift-Expect … checks nothing" matches card 10's new step 3.
- No card cites card 10's old step numbers.

Tool calls:
- grep of the plan cards for every symbol and path the fix touched: my context lacked the cards the fixer did not open (00, 23 and the others).
- read of card 23's header, plus a grep for step cross-references: my context lacked card 23's Edit list and any reference to card 10's renumbered steps.
