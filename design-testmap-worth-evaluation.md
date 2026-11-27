# Evaluate the coverage maps (testmap), every user of them and what each gives, and whether to remove them

`testmap-worth-evaluation` · Task · research, testmap, gate · landing · Finalize

## Why (operator, 2026-11-26)

Much effort went into the per-test coverage maps, and little use has been found for them.
The measurement in `docs/research/speed/testmap-use-measure-2026-11-21/report.md` found, over 10 darn runs:
the reach turn added tests in 2 runs; the mutation step raised 6 LOW findings from 27 survivors; the "reaches none" triage line was wrong every time and is removed;
reach and mutation together cost about 2 minutes of gate wall time per run.
The operator's view (2026-11-27): the tools work and can find redundant tests, but few tests turned out redundant.

## Done meanwhile (operator, 2026-11-26 and 2026-11-27)

Every report step, reach, redundancy and mutation, is off in every gate key of the loomyard hub's `gate.yaml` (`reports: []`).
The tools stay for manual use, such as an occasional redundancy sweep.
Note `whole-repo-guards-off-gates` moves the tests of the coverage-map tools out of the task gates.

## Changed since the measurement (`e13faedc2`, landed 2026-11-27)

The map run now runs tests marked unresolved, and their coverage counts in `lym gate reach`, the reach report and the mutation report's possible killers;
before, a tier1 test that called a function directly could be missing from reach.
A full `lym gate test --map` takes longer for it, by an amount not yet measured.

## What to find out

- Every user of the coverage maps in lym: the reach, redundancy and mutation report steps, `lym gate mutate`, `lym gate redundant`, `lym gate reach`, `lym gate test --map`, and any other reader, map runs that dobby or a row starts included.
- For each: what it has given in landed runs (tests added, findings raised and fixed) against what it costs in gate time, gate slots, memory, test time and upkeep.
- The effect of turning the report steps off: whether review findings, or the defects that land, change in the runs after it.
- A recommendation per user: keep for manual use, or remove; and whether the maps themselves, `internal/testcover` and their upkeep, go when no user is kept.

The run reports only; removing anything is a task of its own on the operator's word.
