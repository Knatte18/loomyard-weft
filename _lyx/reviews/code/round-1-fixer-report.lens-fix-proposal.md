```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal (the new row `RemovalNamedByTwoCardsIsCreditedToTheFirstOnly`).

**LOW — `gates_test.go`, the row `RemovalNamedByTwoCardsIsCreditedToTheFirstOnly`: the row does not fail when a removal is credited to every naming card, which the review's `fix` claims it does.**
- The review says crediting "the second naming card, or both" fails the row because card one's growth is not offset.
  That holds for the second card alone and fails for "both".
- Crediting the second card alone leaves `removedByCard[0]` empty.
  Card one then grows past the line and is flagged, so the row fails.
- Crediting every naming card puts `PATTERN-b` in both `removedByCard[0]` and `removedByCard[1]` (`patternplan.go:129-134`).
  Card one's growth is still offset, because it is credited there.
  Card two's step then removes an entry that is already gone.
  I did not check `pattern.ProjectOverview`, but if it ignores an absent ID the projection is unchanged and the row still passes.
- That mutation looks behaviourally equivalent to first-only crediting, so no row could tell them apart.
  The row pins "first rather than last", which was the realistic regression, so the gap is small and the test is sound.
- The commit message and any report text should say the row pins first-versus-last, not first-versus-every.

No other findings.

Tool calls (2 of 6):
- `grep` in `internal/loomshed`: located `cardHeadroomFindings`. My context held only the test file, not the production code.
- `Read` of `patternplan.go:118-168`: to trace the crediting logic for the mutation cases.
