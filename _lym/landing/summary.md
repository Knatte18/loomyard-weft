# Report: how to build and keep the coverage maps cheaply

Adds `docs/research/speed/testmap-smart-build-2026-11-15/report.md`, the study board task `testmap-smart-build` asked for,
with the three probes' results beside it and an entry in `docs/research/README.md`.
It changes no code.

## What it studied

How LoomYard can build and keep per-test coverage maps cheaply enough to pay for themselves,
for the gate's test selection and for `lym gate redundant`.
It weighs the task's five directions by reading `internal/testcover`, `internal/gatecli`, the earlier reports
and Go 1.26.8's sources, and by three probes on `internal/loomcli`'s 259 tier0 tests, each run once.

## Findings

- The cost is `lym`'s bookkeeping, not the method.
  Today's measure loop took 239 s for the 259 tests, and 205 s of it named the covered functions of the linked packages:
  `fillMapTests` filters each test's whole profile once per linked package, 157 times per test.
- The stored maps' 5,327 entries sum to 86 s of the tests' own time; the first build of the set took 44 minutes.
- One process per test, eight at a time, ran the 259 tests in 5.2 s.
  One process with a snapshot after each test took 3.3 s
  and left the logger's once-per-process set-up out of 84 tests' records.
- Static reachability gives every test the same 7,609 functions, 70 times the median test's coverage,
  and takes 2.0 s per test.
- An ordinary whole-package run under coverage is no slower (1.40 s and 1.61 s against 1.57 s),
  and it can record a package's reach, never a single test's.

## What it recommends

Keep one process per test and Go's block coverage, and make the measure loop lean:
read each profile once, run every queued test side by side, write once per package, build ahead.
A full build is then estimated at 16 CPU-minutes, about 4 minutes at four tests side by side, against 44 minutes.
Build on command, run no pass per landing,
and leave the gate's selection off until its saving on importing packages is measured.
The decision is the operator's.

## Design decisions

- The probes ran through a scratch Go program, not committed, as the payoff report's replay did.
  It is a module outside the tree that imports `internal/testcover`, so it measures with the real toolchain calls.
  It changed no repository file and wrote nothing under Bolt.
- The program's plumbing was checked once on `internal/agentname` with two tests before the probes.
  The report says so, and takes no figure from that check.
- The one-process arm injects its hook through `-overlay`, which cannot add a file to a package under coverage.
  `internal/testkit/tmuxkit` is therefore left out of that arm's coverage and out of every arm's block sets.
- The static probe roots each test with every package initializer and `TestMain`, since those run in every test process.
  The report names an analysis rooted at the test alone as not run.
- The full-build and per-landing figures are estimates, scaled from one package by its linked package count.
  The report shows the basis of each and lists the scaling as an open risk.
- Board task `testmap-miss-causes`, in flight, measures what sends a pass into whole-package rebuilds.
  This report leaves that question to it and says how the two results combine.

## Verification

No gate applies to a report.
The three result files are the probes' own output; `probe-3.json` is trimmed of the binaries' last output line.
