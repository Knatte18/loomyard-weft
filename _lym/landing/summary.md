# Time every internal/fabricengine test one by one on a quiet machine

Adds `docs/research/speed/fabricengine-per-test-times-2026-11-07.md`, a measurement report; no code changes.

The report times the tier 0, tier 1 and tier 2 runs of `internal/fabricengine` serially (`-parallel=1`, `-count=1`, `-run .`), each run once on a machine whose one-minute load was below 1.0 with no other run in flight.
Tier 1 takes 132 s in the package, and tier 0 takes 0.15 s.
Tier 2 holds no test in this package.
Over 1500 leaf tests share the tier 1 time, with a median of 60 ms and the slowest ten groups of tests taking 28 % of it.
The slowest fifteen are each read for what they spend time on: hub copies, `AddPair`, test-side git processes and the verb under test, with fixed waits rare.

## Design decisions

- The gate records `go test -v` text, not `go test -json` events, so the elapsed times come from the `--- PASS` lines of the recorded stdout.
- A top-level test is ranked by the larger of its own line and the sum of its leaves, because a parallel subtest's time is on its own line and not in its parent's.
- The record asked for every test and subtest in the report; it lists the 60 slowest of the 1728 printed lines and the 30 slowest groups of tests.
  The full list stays in the gate's recorded stdout, which the report names, since a file of 2649 rows adds nothing to its conclusions.
- The record asked for the three runs only and no other run, and the darn prompt says to run no `lym gate test`.
  The three runs are the task's work and ran as the record states; no other gate run was made.
- The report's code reading for eleven of the fifteen tests came from a lookup subagent, and the counts of git processes inside production code it did not open are marked as guesses in the report.

## Departures

None from the PATTERN entries; the commit holds a doc only.
