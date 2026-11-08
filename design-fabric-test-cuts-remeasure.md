# Measure the fabricengine tier 1 time the two test cuts saved

`fabric-test-cuts-remeasure` · Task · research, fabric, testkit · running · Open-Window

## Seen (orch, 2026-11-08)

Two cuts dropped `internal/fabricengine` tier 1 tests: `f2ac2bf7` (folds, tier 0 moves) and `cbdfc33d` (188 tests from the coverage study's removable set; 762 test functions to 574).
Neither PR recorded the package's tier 1 time before and after.
The last measurement is `docs/research/speed/fabric-test-speed-remeasure-2026-11-07.md` (fabricengine about 47.7 s inside a whole tier 1 pass, 65.4 s for the pass).

## Measurement (operator, 2026-11-08)

A darn `docs` run that measures what the two cuts saved, on one machine, in its own worktree:

1. **After:** the worktree as branched from `main`.
2. **Before:** the same tree with the two cut commits reverted without committing (`git revert --no-commit cbdfc33d f2ac2bf7`), restored with `git reset --hard` once measured.

For each state, run `lym gate test --tier 1 ./internal/fabricengine` twice, alternating states (after, before, after, before): four runs in all, no more.
Record each run's wall time and the package's own time from the gate's report.
Also run one whole tier 1 pass (`lym gate test --tier 1`) in each state, once: two runs in all.
Note the machine's load during each run, and check the system log for a suspend window as the role says before trusting a run.

The report lands under `docs/research/speed/` as `fabric-test-cuts-remeasure-2026-11-08.md`, with a table of every run, the median saving of the package and of the pass,
and a line comparing the result with the coverage study's estimate (the dropped tests cost up to 78 s standalone).
No code or test changes; the worktree ends with only the report.
