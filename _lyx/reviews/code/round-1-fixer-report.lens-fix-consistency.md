```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (the new row `RemovalNamedByTwoCardsIsCreditedToTheFirstOnly` in `internal/loomshed/gates_test.go`) for agreement with its surroundings.

**Finding 1 — LOW**
- Changed text: the row `RemovalNamedByTwoCardsIsCreditedToTheFirstOnly`. The review's `fix` key says that crediting "both" fails the row.
- Summary: the row pins "not the last card", but it cannot pin "not every naming card", so the name claims more than the row checks.
- Evidence:
  - The harness (`gates_test.go` ~772–777) copies card 0 as card 2 and sets its `Text` to `secondFence`. That text is `namesB` alone and fences no line, so card 2 adds no growth.
  - If a removal were credited to both naming cards, card 1's growth would still be offset. Card 2 has nothing to offset and no growth to flag. The row passes under crediting-both, and the review's claim that it fails is wrong.
  - The row does fail when the removal is credited to the last card only, because card 1 then stays over the line. That case is already pinned by row `RemovalNamedByALaterCardFlagsTheEarlierGrowingCard` (line 702), which differs only in having no `namesB` on card 1.
  - Pinning crediting-both would need an observable difference, such as a headroom that fits only if the removal is subtracted once. That needs a different fixture.
- Suggested handling: no code change is needed. The fixer should say in the fixer-report that the row guards crediting-first against crediting-last and not against crediting-every. Optionally rename the row to drop "OnlyFirst", or add a fixture where double subtraction flips the outcome.

No other inconsistencies found. The fixtures `removesB`, `namesB`, `removableUnderLine`, `cardOne` and `cardTwo` are reused unchanged. The row's field set matches the neighbouring rows 701 and 702. It expects no findings, consistent with row 701.

Tool calls: 1. I ran `sed -n` on `internal/loomshed/gates_test.go` (lines 640–660 and 764–800), because my context lacked the fixture definitions and how the harness applies `secondFence`.
