```yaml
lens: fix-consistency
outcome: reported
```

Findings:

1. **MEDIUM.** Card 2's new absent-old-file bullet ("A failure of that removal is returned as a failed move is") leaves two siblings untrue.
   - That error would carry the move's wrap text, `the record stays at %s until the entry's next Discussion approval stores it under the new slug`. In this case there is no old file, and the orphan stays at the target name.
   - Card 2 says "A failed move reaches the caller as `Board.Rename`'s error after `board.json` is saved. The new slug is then flagged with no file, which `DecisionRecord` answers as not found." That is false when the target removal fails: the orphan stays, and `DecisionRecord` returns it as found.
   - Card 5's new rename-help paragraph says the error "names the record file left at the old slug's name", which does not hold for this failure.
   - Evidence: card 2's Intent bullets and its "A failed move reaches the caller" paragraph; card 5's paragraph that starts "A rename can report an error after it is saved".
   - The target-removal failure needs its own wrap text, and the two sibling sentences need to name that case.

2. **LOW.** Card 5's ImpactSummary is now stale.
   - It still reads "The help text and the board skill doc gain a line each. Other verbs and callers are untouched".
   - The fix added rename-verb help text and rename text in the skill, so the `rename` verb is no longer untouched.
   - Evidence: card 5's new paragraph "The `rename` verb's help in `Command` and the board skill's rename text say so".

The remaining fixes agree with their siblings:
- Card 7's done-entry wiring and test row match the parent's decision entry.
- Cards 8 and 9 no longer use `pastErr` anywhere.
- No other card or the overview names the dropped rows or branches.

Tool calls:
- A grep of `00-overview.md` for text the fixes touch. My context did not hold the overview.
- A grep of every card for twins of the changed text: `followDecisionRecords`, `pastErr`, the dropped error texts and `approvalErr`. My context did not hold cards 4 and 00, and I needed a check across all cards.
