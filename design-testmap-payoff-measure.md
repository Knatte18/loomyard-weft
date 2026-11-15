# Measure whether coverage maps pay off: incremental upkeep time, load on other runs, gate time saved

`testmap-payoff-measure` · Task · research, gate, testkit · draft

## Goal

Measure whether coverage maps pay for themselves, before `testmap-maps-in-weft` builds their check-in.
The operator suspects the upkeep takes far too long, goes stale fast, and slows every other run by the load it puts on the machine (2026-11-15).

## Inputs

Exactly three landed commits from the last two weeks that change Go code: one small (under 100 changed lines), one medium (100 to 500) and one large (over 500).
Name the three commits and their sizes in the report.
The maps are the ones under `<bolt>/.lym/testcover/`, built on 2026-11-12.
Copy them into the task's own scratch space and work only on the copy: never write, rebuild or remove the maps under Bolt.

## Measurements, one run each, no repeats

For each of the three commits:

1. **Incremental upkeep:** plan the upkeep for the commit's diff the way `testcover.PlanUpkeep` does, re-measure the affected entries on the copy, and record the wall time and the number of tests and packages re-measured.
2. **Load on a concurrent gate:** run the tier-1 gate of one package the diff touches twice, once alone and once while step 1's re-measure runs, and record both wall times.
3. **Gate saving:** for the packages the diff touches, record the tests the maps select against the whole packages,
   and the gate wall time of the selected tests against the whole packages.

Run every gate through `lym gate test`, never a raw `go test`.
Check the system log for sleep windows over each measurement, and repeat a measurement that a sleep window crosses, once.

## Report

A report under `docs/research/speed/` with the three commits, a table of the measurements, and a short verdict:
whether an upkeep pass after a typical landing costs less than the gate time the maps save before the next landing,
and how stale the maps grow when upkeep runs only by hand.
It names facts; the decision on `testmap-maps-in-weft` is the operator's.
