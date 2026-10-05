# Round 3 fixer report

## Fixed

- F1: D4 now defines "reached" as the counted `Stuck`s before this round's `Stuck`, seed included, being `>=` `review_max_bounces`, which is Shed's comparison; the Nth judged round of N escalates, matching the acceptance criterion.
- F2: D4's circling-continue claim holds when the `Bounces` seam reads; D4 states every review-segment Bouncer is wired with the seam; Open risks names the failed-read fall-through to Shed's block.
- F3: the Goal drops the PR-review clause; D8's Bound states the entry lands in `## Decisions`, where Plan-Review reads it, and that the marker and diff serve a human reader only.

## Deferred

None.
