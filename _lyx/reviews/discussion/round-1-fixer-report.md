# Round 1 fixer report

Every finding in `round-1-review.md` is fixed in `_lyx/discussion/decision-record.md`, with ledger updates in `_lyx/discussion/support-log.md`.

## Changes

- F1 (BLOCKING): Decision 6 now says `alone_above` and `budget` both compare Decision 5 cost, limits `max_cards` and `budget` to batches of two or more cards, and bounds a card over `budget` on its own to running alone.
  An acceptance criterion pins it; ledger Q12 records the pick.
- F2: Decision 1 orders the per-fork report, then the estimator with its size source, then the calibration report, all before the cost batchifier, profiles and webster changes.
- F3: Decision 3 has the report take a named profile's coefficients.
  The Auto-mode assumption now says the template ships initial coefficients and a late card sets weights by one median-ratio scale factor; ledger Q13 records it.
- F4: Decision 4 yields a message count and a read set, and Decision 5 states how both feed the segment cost, how a package contributes, and the cost unit.
- F5: Decision 3 holds the one base-tree rule, the parent of the first commit the run's first batch made on the task branch; the Auto-mode line is deleted and Q6 points to Decision 3.
- F6: Decision 12 places the assertion; Decision 11 states the refusal for any change at or before the last begun batch's last card, relies on the fingerprint guard for out-of-rebaseline edits, and accepts changes after that point.
  Acceptance criterion extended.
- F7: Decision 8 states the bound on a mid-run load error, with an acceptance criterion and a Constraints example; ledger Q14.
- F8: Decision 10 makes a state without a partition batch with identity whatever profile is active; acceptance criterion and ledger Q15.
- F9: Decision 10's verb list is replaced by "every verb that forms batches from the plan today".
- F10: Decision 12 drops the commit hash and the sentence repeating Scope's split with `webster-delete-ordering`.

## Observed, not changed

The support log's Interview item 4 cites "Decision 12" for retiring the `Batchable?` column, which is Decision 13.
It is outside the recorded findings, so the next round's review can take it.

## Deferred

Nothing deferred.
