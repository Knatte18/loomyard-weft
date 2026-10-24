# Fixer-report — code review round 1

## Changes

- F1 (LOW): fixed with an own adjustment to the proposal.
  Added the row `RemovalNamedByTwoCardsIsCreditedToTheFirstNotTheLast` to `TestNewPlanGate` in `internal/loomshed/gates_test.go`, as the `fix` key described: `removesB`, `removableUnderLine`, `cardFence` of `namesB` plus `fenceOf(longEntryA)`, `secondFence` of `namesB`, expecting no finding.
  The proposal claimed crediting every naming card also fails the row.
  It does not: card one stays offset and the second removal drops an ID already gone, so that mutation is behaviourally equivalent to the current code and no row can pin it.
  The row fails under crediting the last card, which is the realistic regression, so it is named for first versus last.
  Commits: `loomshed: fix F1` (row added), then a rename commit.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the row name claimed more than it pins; fixed by renaming the row.
- fix-proposal: LOW, same point (crediting-every is not caught); fixed by the same rename.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
