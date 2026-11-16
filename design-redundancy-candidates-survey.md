# Rebuild every coverage map, time it, and list the redundant-test candidates of each package

`redundancy-candidates-survey` · Task · research, gate, testkit · running · Drift-Baseline

## Goal

The first step of `redundancy-sweep-existing`: fresh coverage maps for every package, timed,
and each package's redundancy candidates in one report the operator reads before any cut is filed.
The run changes no code and no test; it lands only the report.

## The runs

1. One full build: `lym gate testmap` over every package of the module, once, in this task's own worktree at its own HEAD, with the maps kept in that worktree's `.lym/` or `.scratch/` so they are thrown away with it; never from the prime and never into the hub's shared maps.
   Record its wall time and CPU time, the packages built, any that failed and why, and the time per package.
   Set the wall time beside the smart-build report's estimate (`docs/research/speed/testmap-smart-build-2026-11-15/report.md`: about 16 CPU-minutes, about 4 minutes at four tests side by side) and the 44 minutes of the last full build on 2026-11-12.
2. `lym gate redundant <package>` once on every package the build mapped.

No other build, repeat or test run.

## The report

A bench report under `docs/research/speed/`, with its raw output beside it under `data/`.
The maps it builds serve this report only: they go with the worktree, and each later cut task rebuilds its own package's map at its own HEAD.

### The build (bench)

- Wall time, CPU time and peak memory of the one full build, from `/usr/bin/time -v`.
- Its scope: packages built, tests run per tier, tests skipped (spawn-marked, `//testtiming:keep`) and why, packages that failed and why.
- Time per package, build and run apart, with the ten slowest named.
- The wall time beside the smart-build report's estimate (`docs/research/speed/testmap-smart-build-2026-11-15/report.md`: about 16 CPU-minutes, about 4 minutes at four tests side by side) and the 44 minutes of the last full build on 2026-11-12; where it misses the estimate, where the time went.
- Any test the side-by-side build reran alone after failing beside others, as a sign of tests that share a fixed path.

### What the maps offer: how much, and how

- Per package with candidates, slowest package first: each candidate with its tier, seconds, covering tests, and whether it is in the removable set, with the package's total candidate seconds.
- A table of the packages with no candidates.
- A short read of the largest packages' candidates (at most five packages), as the pilot did for `internal/adoptcli`:
  for each, whether it asserts something its covering tests do not, since equal coverage is not equal checks,
  and which way it would go: cut, folded into a covering test as an extra assertion, or kept.
  It cuts nothing.
- The payoff, from those reads and the lists:
  the candidates' summed seconds per tier against the suite's own time per tier (from the gate's run records),
  the share the read found cuttable, applied to the whole list as an estimate with its spread stated,
  and what that would take off a full `lym gate test` run and off a typical darn verify gate, in seconds and in percent.
- A recommendation: whether the cut is worth tasks, which packages first, and the cut-task shape (one darn per package or per group of small packages).

## Bounds

Change no production code, no test, nothing under Bolt and not the hub's shared maps.
`internal/fabricengine` is listed but not read, as its serial-phase rework is changing it.
