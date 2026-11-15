# Report: do the coverage maps pay for themselves?

Adds `docs/research/speed/testmap-payoff-2026-11-15.md`, the measurement `testmap-maps-in-weft` waits on.
It changes no code.

## What it measured

Three landed commits that change Go code, a small, a medium and a large one (`4df9f81c`, `79f0eb4a`, `36621f0e`),
each replayed as an upkeep pass on a copy of the hub's coverage maps.
The replay is a scratch program, not committed, because `lym gate testmap` runs only in the prime worktree and writes to Bolt.
It plans with the real `testcover.Selector` and `testcover.PlanUpkeep` and measures with `testcover.Toolchain`.
It leaves out the slot and the closing audit.

## Findings

- A pass took 25.0, 7.7 and 31.3 minutes; the entries the commit really affects cost 1.5 s and 21 s of that.
  Selector misses queue whole-package rebuilds, which are the rest.
- The gate time the maps saved on the touched packages was 0 s, 4.2 s and 2.8 s.
- After one landing 54 of 211 maps need a whole rebuild; from the third landing on, 77 do.
- A gate beside a pass took 3.6 s against 3.9 s, 27.3 s against 26.0 s and 10.2 s against 10.3 s.

## Design decisions

- The maps were copied from Bolt as they stand today, not as built on 2026-11-12: the hub's upkeep passes have moved most
  of them to `343405cf`.
  The report says so.
- The replay rewinds each copied map to the commit's parent and keeps the coverage measured at `343405cf`,
  the stand-in the 2026-11-12 pilot used.
- The three commits come from 2026-11-11 and 2026-11-12; the report names their Go line counts, test files included.

## Verification

No code changed, so no test applies.
The report's numbers come from the replay's result files and the gate's own output.
