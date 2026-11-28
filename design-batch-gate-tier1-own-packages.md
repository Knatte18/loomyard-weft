# The batch gate runs tier1 over the batch own packages, not their dependents

`batch-gate-tier1-own-packages` · Task · enhancement, gate, webster

## Seen (`docs/research/webster/gate-time-2026-11-28.md`, lever A)

Since the card gate scope grew in `a69580709`, the batch gate takes 24.1 min per row against 6.6 before, mostly its tier1 run over the dependent packages.
That tier1 listed run takes 18.9 min per row.
None of the batch gate's tier1 failures after the change was in a dependent package, and the plan-level verify still runs tier1 over everything.

## Change

The batch gate's tier1 runs over the packages of the batch's own cards only, not their dependents.
Tier0 and the card gate keep their scope, and so does the plan-level verify.

## Expected

About 14.6 min saved per Webster or Weaver row; the report's estimate is 4.3 min for the narrower run against 18.9 min today.
