# Rework coverage — round 1

One finding on PR #529, one new card.

- F1 (BLOCKING), `cmd/lyx` test budget overflows after the merge with `main` — card 12.
  The row in `cmd/lyx/testdata/test-budget.yaml` goes to 75, the merged count; the merge with `main` is Finalize's own catch-up merge-in, and the card runs no git merge.
