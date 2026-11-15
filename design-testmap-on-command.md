# Coverage maps and test-suite trimming run only on command, never per landing

`testmap-on-command` · Task · enhancement, testkit, gate, dobby · running · Darn-Write

Depends on: [`testmap-miss-causes`](design-testmap-miss-causes.md)

## Seen

The payoff report `docs/research/speed/testmap-payoff-2026-11-15.md` (commit `055ef994`) found that the prime dobby's coverage-map upkeep pass after a landing took 7.7 to 31 minutes
and saved the gate 0 to 4.2 s on the packages each diff touched.
The operator switched upkeep off hub-wide on 2026-11-15 (`test_map_upkeep: false`).

The study `docs/research/speed/testmap-smart-build-2026-11-15/report.md` (commit `b1fc64b4`) reaches the same stance:
build on command, run no pass per landing, and leave the gate's selection off until its saving on importing packages is measured.
It finds the cost is `lym`'s bookkeeping, not the method: 205 of 239 s of the measure loop on `internal/loomcli` went to naming covered functions per linked package.
It estimates a lean build of the full set at about 16 CPU-minutes, about 4 minutes at four tests side by side, against the 44 minutes of the first build.
Those figures are estimates scaled from one package.

## Design (operator, 2026-11-15)

1. Coverage maps are built only on command, never by the dobby or after a landing.
2. A built map set serves to trim the test suites: cut redundant tests and other waste, judged by what each test covers.
3. All test-suite optimisation, trimming and map building happens only on command, never automatically per landing or on any other trigger.
4. The gate stops selecting tests from the maps and always runs whole packages, since maps no upkeep keeps go stale after the first landings.
5. The first use of the on-command build is the redundant-test cut of `redundancy-sweep-existing`.
6. The on-command build takes the smart-build report's lean measure loop, keeping one process per test and Go's block coverage:
   read each test's profile once, run the queued tests side by side, write once per package, and build the test binaries ahead.
7. The report of `testmap-miss-causes`, which this task waits on, says what sends a build into whole-package rebuilds and what following names would save.
   Take what it recommends into the build where it lowers the number of packages a build re-measures.

## What the run settles

- What goes: the dobby's upkeep pass and its `test_map_upkeep` key, the gate's map-based selection, the upkeep-only parts of `lym gate testmap`, and the planned work that only serves upkeep.
- The command or commands the operator runs: build the maps, then report and cut redundant tests (`lym gate redundant` today).
- Time one full build with the lean loop once, on this machine, and state it beside the report's estimate.
