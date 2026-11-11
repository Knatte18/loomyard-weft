```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether each part of the change set still agrees with the sibling decisions, criteria and cross-references it touches or depends on.

1. **MEDIUM.** The D4 rationale still says "a change outside them, on either side of the build, is a miss per D5". "Them" now means function declarations, after "inside function declarations" was swapped in.
   - Summary: that sentence still makes every change outside function declarations a miss. The narrowed D5 bullet now exempts an added const, a var with no initializer and a type with no methods.
   - Evidence: D5's fifth bullet now reads "other than comments, formatting, a reordered or regrouped import block, and an added declaration no unchanged code can reach". The D5 criterion now says "an added const in A … is no miss".

2. **LOW.** The D3 criterion "an importing package's guard test, out-of-process test and test absent from its map run even when D4 selects nothing" was left as it was.
   - Summary: it contradicts the new D3 text, which keeps only the tier's tests in the union and drops tests the watch skip names. It also contradicts the new criterion, where an unwatched watch-marked guard test runs no test.
   - Evidence: the new D3 paragraph reads "The union holds only tests of the tier and drops the package's tests the diff's watch skip names". The new criterion reads "whose only union member is an unwatched watch-marked guard test runs no test". The old criterion should say "of the tier" and "not named by the watch skip".

3. **LOW.** Support-log Q5 still reads "Selection diffs every covered package from the map's own `Commit` (D4)".
   - Summary: D4 and Q6 now say "linked package", so this ledger entry still uses the term the round replaced. A reader would take it as a narrower set.
   - Evidence: D4 now reads "the scope compares every linked package its map records". Q6 was updated to "linked-package", but Q5 was not.

Further tool calls: none.
