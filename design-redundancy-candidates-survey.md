# Rebuild every coverage map, time it, and list the redundant-test candidates of each package

`redundancy-candidates-survey` · Task · research, gate, testkit · running · Open-Window

## Goal

The first step of `redundancy-sweep-existing`: fresh coverage maps for every package, timed,
and each package's redundancy candidates in one report the operator reads before any cut is filed.
The run changes no code and no test; it lands only the report.

## The runs

1. One full build: `lym gate testmap` over every package of the module, from the prime worktree as the command requires, once.
   Record its wall time and CPU time, the packages built, any that failed and why, and the time per package.
   Set the wall time beside the smart-build report's estimate (`docs/research/speed/testmap-smart-build-2026-11-15/report.md`: about 16 CPU-minutes, about 4 minutes at four tests side by side) and the 44 minutes of the last full build on 2026-11-12.
2. `lym gate redundant <package>` once on every package the build mapped.

No other build, repeat or test run.

## The report

Under `docs/research/speed/`:

- The build's timing, as above.
- Per package with candidates, slowest package first: each candidate with its tier, seconds, covering tests, and whether it is in the removable set, with the package's total candidate seconds.
- A table of the packages with no candidates.
- A short read of the largest packages' candidates (at most five packages), as the pilot did for `internal/adoptcli`:
  whether the candidate asserts something its covering tests do not, since equal coverage is not equal checks.
  It cuts nothing.
- Any test the side-by-side build reran alone after failing beside others, as a sign of tests that share a fixed path.

## Bounds

Change no production code, no test and nothing under Bolt.
`internal/fabricengine` is listed but not read, as its serial-phase rework is changing it.
