# Measure the coverage maps: build cost, redundant runs, crosscheck

Adds a research report under `docs/research/speed/coverage-map-measurement-2026-11-18/` with the raw `lym gate redundant` answers beside it.
No code and no test changed.

## What the report finds

- A full `lym gate test --map --all` took 12 min 49.7 s over 210 packages and 4911 tests,
  with a 2.12 GB largest process and no memory cap or kill.
  Available memory on the shared machine never fell below 15.9 GB, so the 2 GB stop never fired.
- `lym gate redundant internal/boardengine` fails after 5 s: its mutation binary does not compile (`byte` compared with a rune-literal mutant from `9ce7233b`).
  The five known false candidates therefore cannot be shown gone.
- `internal/adoptcli` gave no candidates, and neither pilot candidate is still a test of the package.
  `internal/discussionparser` gave one candidate.
- Six single-test coverage runs agree with their map entries by covered blocks per file.
  The check cannot see which blocks, so it supports the map's totals, not block identity.
- Verdict: the maps are fit for the cut stage and `lym gate redundant` is not, until the `boardengine` build failure is fixed.
  E1 is not needed by the task's rule, since no crosscheck disagreed.

## Design decisions

- `internal/discussionparser` was run as a third package, one beyond the cap of two, because `internal/adoptcli` produced no candidate to crosscheck.
  It costs 35 s.
- The crosscheck used six tests, not five: the one candidate, its cover and four former `boardengine` candidates.
- A full map build on Linux exits 1 on ten Windows-only tests; the report notes it as a defect and leaves it unfixed.
