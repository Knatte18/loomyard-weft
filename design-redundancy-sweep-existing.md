# Cut the existing redundant tests of every package, found by the coverage maps

`redundancy-sweep-existing` · Task · enhancement, testkit, gate · draft

Depends on: [`redundant-timeout-not-kill`](design-redundant-timeout-not-kill.md)

## Seen (orch, 2026-11-09)

The coverage maps (`internal/testcover`, built by `lym gate test --map` since `b92c95ee`) record, per package, the blocks each tier 0 and tier 1 test covers.
The gate report (the advisory redundancy step of `lym gate test`, landed as `7f21142e`) judges only the new tests of a diff against the map;
nothing yet uses the maps to find redundant tests that already exist.
The operator wants the existing ones cut, across the packages, not only kept from growing.

The survey `redundancy-confirmed-survey` (`53d40bd0`, report `docs/research/speed/redundancy-confirmed-survey-2026-11-16/report.md`)
ran the mutation-confirmed `lym gate redundant` over 121 packages:
430 tests stay confirmed, 2.89 s of removable standalone time, and 27 of 30 sampled are false confirmations,
each asserting something its covering tests do not.
So the list is a reading filter, never a verdict:
the cut is a test-quality cleanup that reads every confirmed candidate against its covering tests, not a speed task.

## Idea

1. A candidate list per package from `lym gate redundant <package>`, which omits `//testtiming:keep` and spawn-marked tests.
2. Cut tasks on that list, each run delegating the per-test reading to subagents:
   a candidate is cut or folded into another test only when its assertions add nothing the covering tests check.
   The survey names what to look for first: absent side effects, persisted state, and inputs the covering tests do not feed.
3. `internal/fabricengine` was swept by hand on 2026-11-08, so it comes last or not at all.
4. Before judging a package, the sweep builds that package's map fresh in its own worktree with `lym gate test --map`,
   since no landing keeps the maps fresh; spawn-marked tests are neither candidates nor covers.
