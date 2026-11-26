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

## What to find out

- Every user of the coverage maps in lym: the reach, redundancy and mutation report steps, `lym gate mutate`, `lym gate test --map`, and any other reader.
- For each: what it has given in landed runs (tests added, findings raised and fixed) against what it costs in gate time, memory and upkeep.
- The effect of turning the mutation step off: whether review findings, or the defects that land, change in the runs after it.
- A recommendation per user: keep, tune or remove; and whether the maps themselves, `internal/testcover` and their upkeep, go when no user is kept.
