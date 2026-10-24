```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check that the added row `RemovalNamedByTwoCardsIsCreditedToTheFirstOnly` fails when a removal is credited to the last or to every naming card.

No findings.

- **Added row under the correct rule.** Card 1's text is `namesB` plus the long `PATTERN-a` fence, and card 2's text is `namesB`. `slices.IndexFunc` returns index 0 (`patternplan.go:129`), so `removedByCard[0] = [PATTERN-b]`. Card 1's step swaps the short entry for the 200-byte one and drops the 150-byte `PATTERN-b` in the same step. The net change is small and stays under the line. Card 2 changes nothing. The row expects no finding and gets none, which matches the existing row `RemovalNamedByTheGrowingCardOffsetsItsGrowth`.
- **Mutant: credit the last naming card.** The removal moves to card 2, and card 1 grows by about 195 bytes from `HeadroomBytes-100` with no offset. That puts it past the line and produces a `pattern-card-headroom` finding on card 1. The row expects none, so it fails and catches this mutant.
- **Mutant: credit every naming card.** Card 1 is still offset, and the removal on card 2 is a no-op because the ID is already gone. No finding appears, so the row passes under this mutant. The mutant is behaviorally equivalent to first-only unless a later card re-adds `PATTERN-b` through a fence, and the added row has no such card. So this is not a defect. The review's remark that crediting both fails the row does not hold, but the row still pins the "first, not last" half of the rule.
- **Callers.** The change touches only the test table, so no signature or caller changed.

Tool calls: 2 reads, 2 of 6 allowed.
- `patternplan.go:100-190`: to trace `cardHeadroomFindings` crediting, which my context did not hold in changed or unchanged form.
- `gates_test.go:636-665`: to confirm the constants (`namesB`, `removableUnderLine`) the new row uses.
