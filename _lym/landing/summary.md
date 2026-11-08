# Measure the fabricengine tier 1 time the two test cuts saved

Adds `docs/research/speed/fabric-test-cuts-remeasure-2026-11-08.md`, the first before-and-after measurement of the cuts `f2ac2bf7` and `cbdfc33d`.
No code or test changes.

On one machine, the cuts save about 4.3 s of `internal/fabricengine`'s own tier 1 time (37.0 s against 41.3 s, median of two runs per state).
One whole tier 1 pass falls from 61.0 s to 56.5 s.
That is about 5 percent of the coverage study's estimate of up to 78 s standalone, because the package's tests run in parallel and their standalone costs do not add up to wall time.
The package is still the pass's longest.

## Design decisions

- The "before" state is the tree with both cuts reverted, uncommitted, and restored with `git reset --hard`; the worktree ends with only the report.
- The gate carries a package from an earlier green record when the tree and the args after `--` match, so each run used args no earlier run of its state had used.
  The first whole pass in the after state carried `internal/fabricengine` and is left out of the results; the report says so.
- The whole passes use `--all`, because without it the scope is the diff from the merge base, which is empty in the after state.
- Runs 3 to 6 waited for a one-minute load under 2.0; run 2 started at 3.65 and is the slowest package run.
  Nothing in the system log for the measured window records a suspend.
- A whole pass is one sample per state, so its saving is weaker evidence than the package medians.
