# Evaluate the coverage maps (testmap), every user of them and what each gives, and whether to remove them

`testmap-worth-evaluation` · Note · research, testmap, gate

## Why (operator, 2026-11-26)

Much effort went into the per-test coverage maps, and little use has been found for them.
The measurement in `docs/research/speed/testmap-use-measure-2026-11-21/report.md` found, over 10 darn runs:
the reach turn added tests in 2 runs; the mutation step raised 6 LOW findings from 27 survivors; the "reaches none" triage line was wrong every time and is removed;
reach and mutation together cost about 2 minutes of gate wall time per run.

## Done meanwhile (operator, 2026-11-26)

The mutation report step is off in every gate key of the loomyard hub's `gate.yaml` (`reports` without `mutation`; `webster-fix` and `pr-review` with `reports: []`).
Reach and redundancy stay on.

## Changed since the measurement (`e13faedc2`, landed 2026-11-27)

The map run now runs tests marked unresolved, and their coverage counts in `lym gate reach`, the reach report and the mutation report's possible killers;
before, a tier1 test that called a function directly could be missing from reach.
Reach names one caveat per cause.
A full `lym gate test --map` takes longer for it, by an amount not yet measured,
so the measured value and cost of reach predate this change.

## What to find out

- Every user of the coverage maps in lym: the reach, redundancy and mutation report steps, `lym gate mutate`, `lym gate test --map`, and any other reader.
- For each: what it has given in landed runs (tests added, findings raised and fixed) against what it costs in gate time, memory and upkeep.
- The effect of turning the mutation step off: whether review findings, or the defects that land, change in the runs after it.
- A recommendation per user: keep, tune or remove; and whether the maps themselves, `internal/testcover` and their upkeep, go when no user is kept.
