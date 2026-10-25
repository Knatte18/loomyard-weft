```yaml
lens: fix-consistency
outcome: reported
```

Fork lens: fix-consistency. I checked the changed text against sibling cards, the decision record and the criteria it touches.

1. **MEDIUM.** Card 13's new sentence ("On card 12's handled-false paths … the other cause's escalation brief points the parent at the round's review") does not cover every case card 12 lists.
   - Card 12's Routing lists, among the handled-false cases, "the round's decision file carries a cause other than `decision` (an automatic `budget` accept or continue, …)".
   - An automatic `budget` decision file comes from an auto-accept or a budget continue. That path writes no escalation brief to the parent; card 12 row 17 says it "settles as today's continue, with no `decision` escalation written".
   - On that path no brief points the parent at the review. The new bound holds only for an escalation record or a parent-recorded decision of another cause.

2. **LOW.** Card 10's new carry-on step for a writer with no parent ("on finding one, write its answer into the record as for a parent answer and carry on") then reaches the next bullet: "Once the record is committed, ask your parent to record the same answer with `lyx loom decision add`".
   - A writer with no parent has no one to ask, so the answer stays named as an unrecorded parent correction with no step that records it.
   - The card does not say to skip or replace that bullet on the no-parent path. Decision record D10 (`_lyx/discussion/decision-record.md`, around line 156) also has only the parent form.

3. **LOW.** Card 10's added phrase "or a resume reaches you" contradicts the card's own Bound line, which is unchanged: "the parent's reply wakes the writer, and otherwise `writer_timeout_min` ends the session; nothing respawns it."
   - The new text assumes a resume re-runs the writer, while the Bound says nothing respawns it.

Calls:
- grep of `00-overview.md` and `decision-record.md` for twin terms (HandsBack, tackler, Open risks, Decision for, standsAsRecorded and others): my context did not hold either file.
- `sed -n` of decision-record D8 (lines 115–140) and D10 (lines 154–162): needed the exact bound sentences to compare against the changed cards.
