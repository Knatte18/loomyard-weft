# Measure every package tier0 and tier1 test, build and wall time

`test-time-measure` · Task · research, gate, testkit · done

## Aim (operator, 2026-11-27)

Measure where gate time goes, package by package, so the test-speed project picks which packages to cut or restructure.

## What to measure

For every package under `internal/`, `cmd/lym` and `tools/` that holds tests, on the current main:

- tier0 and tier1 test time, each once, through `lym gate test --tier <n> <pkg> -- -count=1 -v`, the package's own time from go's ok line;
- the same call's wall time, so build and link time shows as wall minus test time;
- the slowest tests in each package from the -v output.

Run one gate call per package and tier, so 2 calls per package and no repeats, except a second run of the 10 slowest packages by wall time to see the spread.
Note the other runs and gate slots in use at each call, from `lym gate status`, since a loaded machine inflates times.

## Report

A report under `docs/research/speed/` with one table of every package sorted by wall time: tier0 test, tier1 test, wall, build share, and the slowest tests.
Close it with the packages worth a cut (test time of 15 s or more), the ones where build or link dominates, and for each what kind of change would help: cut tests, share fixtures, drop sleeps, or a restructure like fabric-core.
No test or production change.
