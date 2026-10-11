# Fixer report: discussion, round 1

Every finding was checked against `internal/shedadapters/bouncer.go` and held; all five are fixed in `_lyx/discussion/decision-record.md`.
`lyx loom validate-discussion` passes after the edits.

## Fixed

- F1 [MEDIUM, design]: confirmed that a degraded Stuck never reaches `settleUndecided`'s `EscalationBudget` branch, so repeated refusals end in Shed's budget-exhausted block.
  Kept today's route: D9's Bound now says the run blocks with Shed's budget-exhausted halt at `review_max_bounces`.
  The Open risks bullet now names the judge by role and speaks of runs blocked at the budget, and the "Repeated refusals" acceptance criterion names the same exit.
- F2 [MEDIUM, decision]: confirmed that `Call` settles an unattached on-disk verdict through `settle(ctx, n, false)` with no harvest.
  D8 now runs the key check on that at-entry settle of a CONTINUE or CIRCLING verdict too, after the entry-time probe finds no live judge, so a lossy ledger cannot become round N+1's baseline.
  Added an acceptance criterion for it.
- F3 [LOW, consistency]: confirmed that `settledGeneration` clears any unattached CONVERGED at entry.
  D7's at-entry sentence now states the bound: such a CONVERGED is cleared and re-seeded whether settled or never harvested, so the guard has no unchecked path to Done.
- F4 [LOW, consistency]: added the criterion that with `seed` and `plan_seed` both set, Plan-Bouncer's seed runs on `plan_seed` and the other segments' seeds on `seed`.
- F5 [LOW, consistency]: moved the requirement that the judge stays the only writer of its three files, with Go archiving and never rewriting them, into D9's Decision text.
  Cut the override-to-CONTINUE comparison from D9's rationale; the support log's Rejected alternatives already holds it.

## Disputed

None.

## Deferred

None.
