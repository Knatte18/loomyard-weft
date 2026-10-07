# Round 2 fixer report

Every finding in `round-2-review.md` is fixed in `_lyx/discussion/decision-record.md`, with ledger updates in `_lyx/discussion/support-log.md`.
Round 1's fixes show no regression; its observed-not-changed item is F11 here.

## Changes

- F1 (BLOCKING): Decision 5 gains a per-fork message count, paid once per batch, as the saving that merging buys against the accumulated context it costs; it joins the profile coefficients.
  An acceptance criterion pins that identical read sets cost less merged and a card adding much context costs more merged; an Open risk says the term cannot be fitted from one-card forks.
  Ledger Q16 records the pick and asks the operator to confirm the term matches the accepted model.
- F2: Decision 1 places the coefficients type and its `batcher.yaml` parse with the estimator, before the calibration report; Decision 8 keeps the registry, `active:` resolution and load errors.
  Ledger Q17.
- F3: Decision 4 now defines the estimator's inputs (cards, size source, coefficients) and both outputs once; Decision 5 only sums over them.
- F4: Decision 5 fixes the cost unit as tokencount's weight unit; the Auto-mode rule names the three context coefficients the scale factor multiplies and has the calibration card set `alone_above` and `budget` in that unit.
  Ledger Q18.
- F5: Decision 11 keeps identity for the tail and records no partition on a state without one; acceptance criterion extended.
  Ledger Q21.
- F6: the Auto-mode assumption defines a poor fit as a ratio spread above 3, and the card then commits nothing; ledger Q19.
- F7: Decision 3 sums the weight of a card named by several forks and skips a card no fork names; the rationale drops "every"; acceptance criterion extended; ledger Q20.
  The Decision 3 rationale also moves to its own line.
- F8: Decision 8 defines an invalid parameter and states that no threshold value makes the split infeasible; acceptance criterion extended; ledger Q22.
- F9: Decision 11 says each changed plan file is named with `--card NN` as the changed-file guard requires.
- F10: Decision 4 drops the signals Decision 5 never used (test-file line counts, the package directory listing, the created/edited/deleted split) and defines the package and test entries, one pair per distinct target directory.
- F11: the support log's Interview item 4 now cites Decision 13.

## Deferred

Nothing deferred.
The thresholds in F6 (spread above 3) and the per-fork term in F1 are self-picks under Auto-mode, recorded in the ledger for the operator to confirm.
