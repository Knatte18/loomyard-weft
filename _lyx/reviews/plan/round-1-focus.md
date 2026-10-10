---
round: 1
focus:
  - "Card 12 against the live generation's `_lyx/loom/rework/round-1/findings.md` F1: does the card carry F1's fix, the row set to the merged count of 75 with nothing else changed, and does its `Intent` license no scope beyond it?"
  - "Card 12 `**Intent:**` against F1's instruction to merge `main` into the branch: the card defers the merge to Finalize's catch-up merge-in; settle whether that leaves F1 covered or leaves the merged-tree guard unreached by this plan."
  - "`## verify:` in `00-overview.md`: does a command that runs the tests of `cmd/lyx`, the package whose budget guard the card targets, appear in the plan's template order?"
  - "Card 12 `**Edit:**` and `**Uses:**` against the file `cmd/lyx/testdata/test-budget.yaml` and the guard `cmd/lyx/testbudget_test.go`: do the stated numbers (74 before, 75 after, 74 tests on the branch) match what those files hold?"
---

The plan holds one card, so granularity, card order and the attack-surface question are expected to be quiet; the focus goes where the one card can go wrong, which is fidelity to the PR-Gate finding and the merge-timing claim.
