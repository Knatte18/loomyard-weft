# Cut the existing redundant tests of every package, found by the coverage maps

`redundancy-sweep-existing` · Note · enhancement, testkit, gate

## Seen (orch, 2026-11-09)

The coverage maps (`internal/testcover`, rebuilt by `lym gate testmap`) record, per package, the blocks each tier 0 and tier 1 test covers.
The gate report (the advisory redundancy step of `lym gate test`, landed as `7f21142e`) judges only the new tests of a diff against the map; nothing yet uses the maps to find redundant tests that already exist.
The operator wants the existing ones cut, across the packages, not only kept from growing.

## Idea

1. A candidate list per package: every test whose covered blocks the package's other tests already cover, with its run time, using the same judgment as the gate report.
   Tests marked `//testtiming:keep` are left out.
2. Cut tasks on that list, slowest packages first, each run delegating the per-test reading to subagents:
   a candidate is cut or folded into another test only when its assertions add nothing the covering tests check, since equal coverage does not mean equal checks.
3. `internal/fabricengine` was swept by hand on 2026-11-08 and is being reworked by its serial-phase tasks, so it comes last or not at all.
