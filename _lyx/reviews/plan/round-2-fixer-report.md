# Round 2 fixer report — plan

## Fixed

- **F1** (MEDIUM, design): confirmed against `internal/shedadapters/bouncer_commit_test.go`.
  The first row of `TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit` settles CONVERGED over a judged ledger holding `{"beta", "open", "scope", "BLOCKING"}` and expects `Done`, which D7's ledger check refuses.
  In `_lyx/plan/05-convergence-guard.md`:
  - `**Edit:**` gains `internal/shedadapters#TestBouncer_ConvergedSettle_CallsCarryOverBeforeApproveAndCommit`.
  - `**Intent:**` states that the row's `beta` entry becomes severity `MEDIUM` in the judged ledger and the expected carry-over finding.
    The row keeps its coverage of the filter, the labels and the ledger rounds, since the carry-over keeps MEDIUM-or-worse findings.
  - `**Intent:**` records the re-check of the method across both of the guard's ledger reads: no other CONVERGED settle in `internal/shedadapters`, `internal/shedrecipe` or `internal/loomrecipe` carries an open BLOCKING ledger entry, and every round-2 or later ledger a settle reads keeps its previous round's keys.
    I verified this by grepping the three packages' tests for `BLOCKING`.
    The other hit, `circlingevidence_test.go`, exercises the circling-evidence reader and settles nothing.
    `loomrecipe`'s `writeBouncerJudge` writes `ledger: []`.
  - `**ImpactSummary:**` names this test beside the three review helpers.

## Disputed

None.

## Deferred

None.
