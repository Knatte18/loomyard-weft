# Measure what sends coverage-map upkeep into whole-package rebuilds, and what a name-following selector would save

`testmap-miss-causes` · Task · research, testkit, gate · running · Seed-Child

## Seen

The payoff report `docs/research/speed/testmap-payoff-2026-11-15.md` (commit `055ef994`) found that a coverage-map upkeep pass after a landing took 7.7 to 31 minutes,
while the tests the commit affects cost 1.5 s and 21 s of it.
The rest is whole-package rebuilds that a selector miss queues.
The operator switched upkeep off hub-wide on 2026-11-15 (`test_map_upkeep: false`) and wants to know whether the maps can be made fast enough to pay.

## What to measure (operator, 2026-11-15)

On the same three commits the payoff report used, small `4df9f81c`, medium `79f0eb4a` and large `36621f0e`, one run each:

1. **What triggers the misses.** For each package an upkeep pass rebuilds whole, name the change that sent it there,
   by the kind of declaration changed: function body, signature, type, method set, constant, variable, embedded file, build tag, test file, or other.
   Give the count of packages and the rebuild seconds for each kind.
2. **What the note `selector-non-body-miss` would save.** That note proposes following the names a changed declaration is referenced by, instead of rebuilding the whole package.
   Estimate, without building it, how many of the whole-package rebuilds it would turn into a targeted re-measure, and the seconds that saves per commit.
   Read `internal/testcover` and `internal/gatecli` for how the selector decides today.
3. **What else would cut the pass.** Name any other cause of time in the pass that the two answers above leave, such as compile time, serial work or package count, with its seconds.

## Bounds

- Three commits, one measurement run each; no repeats unless a run fails.
- Work on a copy of the maps, as the payoff report did; never write, rebuild or remove anything under Bolt.
- Run no agent and change no code; the result is a report under `docs/research/speed/`.
- Measure while the machine is otherwise quiet, and record the other load beside each timing.
- The report states facts and estimates with their basis; the decision on the maps is the operator's.
