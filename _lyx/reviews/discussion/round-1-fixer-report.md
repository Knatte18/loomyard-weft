# Fixer report — discussion, round 1

## Changes

All in `_lyx/discussion/decision-record.md`.

- **F1** (fixed as proposed): D5 names the span kinds as the closed set gate, held and rework, and states that the evaluations a pending entry re-runs while held record no gate span, so no two spans of one entry overlap.
  D6's gate time is the sum of the entry's gate and held spans, the rework time the sum of its rework spans.
  The shuttle acceptance criterion adds that an entry re-evaluated while held reports one held span and no overlapping gate span.
- **F2** (fixed as proposed, first option): D5's Carriers paragraph defines the carriers as every row whose producer runs a gated shuttle run, and names the Burler review row (`BurlerReviewProducer`, gated on `ReviewGateEntry`) beside the Webster row.
  The Scope In bullet and the Webster acceptance criterion name Burler's review-parse gate too.
- **F3** (fixed as proposed, first option): D7 makes the open visit the rework visit when the last history entry is a gate bounce routed to its row, drawn ` rework for <gate row>`, and a gate row's rework total that includes it is drawn with `FormatMinutes` per D1.
  The D7 acceptance criterion covers the open visit.
- **F4** (fixed as proposed): D4 adds that a child row inside a review segment takes its segment's number (`6.6 Plan-Burler-Review`), and the batten overview acceptance criterion covers it.
- **F5** (fixed as proposed): D4's examples read `7. Land` and `6.5 Plan-Write`, checked against `contracts/recipes/batten-recipe.yaml` and `contracts/recipes/loom-recipe.yaml`.
- **F6** (fixed as proposed): the third Auto-mode assumption is deleted.

## Disputed

None.

## Deferred

None.
