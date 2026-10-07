# Round 3 fixer report

Every finding in `round-3-review.md` is fixed; the edits are in `decision-record.md` and `support-log.md`.

## Changed

- F1 (threshold unit and scaling rule): Decision 5 now says cost is a sum of per-message context reads, equal to context tokens under the initial coefficients and rescaled to weight by calibration, and that `alone_above` and `budget` move with their coefficients.
  Auto-mode assumptions state that the three context coefficients are the only ones carrying a context size, so every cost scales by the factor, and that the tuning card multiplies both thresholds by the same factor.
  Q18 in the support log matches.
- F2 (estimate definition): Decision 5 names the segment-cost function as the one function both callers use, defines a card's estimate as its one-card batch cost, and says which Decision applies which coefficient.
  Decision 3 and the acceptance criterion say "segment-cost function"; Decision 1 places that function with the estimator.
  Q25 added.
- F3 (card order): Decision 1 lists the full order, with the coefficient parse rejecting a missing or negative coefficient, the calibration report tested on a fixture, the profiles card with the template's initial coefficients after it, and the tuning card after both.
  Its rationale is now the inactive cost profile and the identity default.
  Decision 8 says which load errors the estimator's parse already raises.
  Q24 added.
- F4 (acceptance wording): the rebaseline criterion now says the tail stays on the identity batchifier and no partition is recorded.
- F5 (per-batch profile name): Scope, Decisions 8, 10 and 11 and the acceptance criteria record the profile name and estimate per batch, and a begun batch keeps both at a rebaseline.
  Q23 added.
- F6 (rationale attribution): Decision 5's rationale names the per-fork term as the one addition to the operator's context-read model and points at the open risk.
- F7 (call-site list): the verb list in the notes for the plan writer is removed.

## Deferred

Nothing deferred.
