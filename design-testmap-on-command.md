# Coverage maps and test-suite trimming run only on command, never per landing

`testmap-on-command` · Note · enhancement, testkit, gate, dobby

## Seen

The payoff report `docs/research/speed/testmap-payoff-2026-11-15.md` (commit `055ef994`) found that the prime dobby's coverage-map upkeep pass after a landing took 7.7 to 31 minutes
and saved the gate 0 to 4.2 s on the packages each diff touched.
The operator switched upkeep off hub-wide on 2026-11-15 (`test_map_upkeep: false`).

## Design (operator, 2026-11-15)

1. Coverage maps are built only on command, never by the dobby or after a landing.
2. A built map set serves to trim the test suites: cut redundant tests and other waste, judged by what each test covers.
3. All test-suite optimisation, trimming and map building happens only on command, never automatically per landing or on any other trigger.

## What the design must settle

- Whether the gate still selects tests from the maps between two builds, when they go stale after the first landings, or always runs whole packages.
- What goes: the dobby's upkeep pass and its `test_map_upkeep` key, the upkeep-only parts of `lym gate testmap`, and the planned work that only serves upkeep.
- The command or commands the operator runs: build the maps, then report and cut redundant tests (`lym gate redundant` today).
