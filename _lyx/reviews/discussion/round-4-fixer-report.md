# Round 4 fixer report

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D4's budget-reached test is now the `Bounces` seam's `count >= budget`, with `count` the Bouncer row's own counted `Stuck`s and `budget` Shed's effective budget for that row.
  D4 now requires the seam's routing to go through the config-applying assembly, so its budget equals `review_max_bounces`.
  An acceptance bullet pins that equality.
- F2: D4's "cannot hit Shed's block" claim is conditional on `ok` true.
  D4 names the unwired, `ok` false and errored reads as falling to Shed's block.
  The Open risk and the circling-continue acceptance bullet carry the same condition.
- F3: D8's "What bounds that" is reworded to say nothing bounds an addition's content, and that a Plan-Write addition is licensed by Plan-Review without a second reader.

## Deferred

None.
