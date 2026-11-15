# Report: what sends coverage-map upkeep into whole-package rebuilds

Adds `docs/research/speed/testmap-miss-causes-2026-11-15.md`, a measurement report that follows the payoff report.
It changes no code.

## What the report answers

- **Triggers.**
  On the payoff report's three commits, every selector miss is an embedded file (small, 54 packages), an added import (medium, 7),
  or a changed type (large, 119, first miss `configengine.Spec`).
  Function body and signature changes are selected, not missed.
  The large commit's first miss hides more: 54 of its packages also see a changed embedded file and an added method.
- **What `selector-non-body-miss` would save.**
  Following the names a changed type is referenced by would narrow 0, 0 and 64 of the whole rebuilds.
  On the large commit that is about 527 s of a 2255 s pass.
  The estimate reads syntax, not types, and the report says where it over- and under-counts.
- **What else costs time.**
  The bookkeeping after each test round takes 76 to 81 percent of a pass:
  a status read, keying every covered block and rewriting the whole map file, 1.2 s a round on average.
  Builds and test runs are the rest.

## Design decisions

- The measurement ran a copy of `internal/gatecli`'s pass in a scratch module, not committed, against a copy of the maps.
  Nothing under Bolt was written, and no agent ran.
- The builds and test declarations came from the worktree at `055ef994`, later than the commits, which queued a few extra jobs.
  The report counts them apart from the miss rows.
- The machine was not quiet during the runs; the report gives the load per run beside each timing and says the medium run is the most affected.

## Testing

No code or test changed.
The report's plan counts match the payoff report's (54, 7 and 119 misses).
