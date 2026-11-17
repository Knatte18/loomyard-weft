```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the change set agrees with the sibling cards, rows and cross-references it touches.

1. **MEDIUM.** Card 3's new row `ARebaseThatConflictsIsAbortedLeavesTheBranchAsItWasAndNamesTheRun` disagrees with step 5 of the same card.
   - **The conflict:** the row asserts "origin holds no takeover entry", but step 5 says "a records branch whose push landed is left as it is".
   - **Evidence:** step 4 writes the entry on both the pair's records branch and the parent's. The hook commits only on the parent's branch, at attempt 1. So the push of the pair's own records branch can land, and origin then holds the entry on that branch.
   - **What to do:** either the row asserts that origin's parent branch holds no entry, or steps 4 and 5 state that the parent's branch is pushed before the pair's.

2. **LOW.** In card 8 step 3, the new "first pulls Bolt and reads the `Hold` seam once more" leaves the failure of that pull unworded.
   - **Evidence:** step 1 words its own pull failure ("returns its error naming the re-run once origin answers"). The new pull in step 3 has no such line, so the green end has no stated outcome when the pull fails.

3. **LOW.** Card 9's new text "drops every entry cards 2, 3, 5 and 7 added" disagrees with card 8.
   - **Evidence:** card 8 already drops `FixHold`, `ClearHold` and `HoldRefusal.Error`, three of card 2's entries, so card 9 can only drop the remaining ones.
   - **Same problem in the overview:** its new Shared Decision says "the card that reaches it removes the entry". But card 9 removes card 5's `internal/adoptrecord` entries because it deletes the package, not because it reaches those functions.

Further tool calls: none.
