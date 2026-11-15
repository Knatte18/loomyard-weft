# Coverage maps built only on command; the gate stops selecting from them

The payoff report (`docs/research/speed/testmap-payoff-2026-11-15.md`) found that the prime dobby's upkeep pass took 7.7 to 31 minutes after a landing, and saved the gate 0 to 4.2 s.
The operator decided that the maps are built only on command and serve to trim the test suites,
and that the gate always runs whole packages.
This change removes the upkeep pass and the gate's map-based selection.
It also turns `lym gate testmap` into the lean on-command build the smart-build report recommends.

## What goes

- **The dobby's upkeep pass.**
  The prime dobby no longer starts `lym gate testmap --upkeep`.
  Its step, its `Deps` seams, the beam key `test_map_upkeep`, `runproc.RoleTestMap` and the upkeep exit record (`testcover.ExitRecord`, `lymdirs.TestMapExitFileName`) are gone.
  `test_map_upkeep` joins `test_map_every` and `test_map_batch` as a retired beam key, which a reconcile prunes;
  beam's `MigrateConfig`, which carried `test_map_every: 0` into `test_map_upkeep: false`, goes with it.
  The orch's stop no longer ends an upkeep pass, and an anchor move no longer spares one.
- **The gate's selection from the maps.**
  The gate key scope `coverage` is retired, with `internal/gatecli/coveragescope.go` and the selected-tests run of a tier.
  The template's `webster-fix`, `darn-write` and `code-burler-fix` keys take `scope: diff`.
  `gateslot.MigrateConfig` rewrites a `scope: coverage` to `scope: diff`,
  and a load that still meets the retired value refuses it, naming `lym fabric reconcile` as the way forward.
- **The upkeep-only parts of `lym gate testmap`.**
  The hidden `--upkeep` flag and its registration go.
  So does the spare-slot pass that gave its slot back to gates, with `gateslot.Pool.TryAcquireSpare` and `Pool.Contended`.
  The default cap of five packages goes too.

## The on-command build

`lym gate testmap` with no package plans every map at the prime's HEAD and measures what the plan queues.
Named packages are rebuilt whole.
A call waits for one gate slot and holds it to its end.
`--max` caps the packages only when given.
The measure loop follows the smart-build report's design:

- **Each profile is read once.**
  `testcover.ProfileByPackage` groups a profile's blocks by package in one pass,
  and `FuncIndex.CoveredFuncs` and `ProfiledFuncs` now take a package's own group rather than filtering the whole profile per linked package.
  A run is reduced to its covered positions and linked functions as it ends, and its profile is dropped.
- **Tests run side by side.**
  Every queued test runs alone in its own process, up to the slot's tier0 parallel at once, whether or not it calls `t.Parallel`.
  A test that fails or times out beside others runs once more alone, and that run stands.
  `testcover.TestFunc.Parallel`, which only grouped tests into rounds, is removed.
- **One write per package.**
  The worktree status and HEAD are read and the map is written once per package, after its tests ran, not after each round.
- **Builds ahead.**
  The next package's `go list` and coverage build start once the current package's binary is ready, so they overlap its tests.

The command the operator runs to trim a suite is unchanged: `lym gate testmap <package>`, then `lym gate redundant <package>`.
`lym gate testmap` over the whole module comes first before a sweep such as `redundancy-sweep-existing`.

## Timed full build

One full build ran on this machine with this branch's binary, from the prime at `5dc7a587`, naming all 211 packages so each map was rebuilt whole.
The hub's tier0 parallel is 4.

| | Smart-build report's estimate | Measured |
|---|---|---|
| Wall time | about 4 minutes at four tests side by side | 6 min 38 s |
| CPU time | about 16 CPU-minutes | 21.4 CPU-minutes (16 min 55 s user, 4 min 31 s system) |
| First build of the set, 2026-11-12 | 44 minutes | |

It built 211 maps holding 4,723 tests, with no failed build, skip, timed-out test, self-check mismatch or audit mismatch.
The packages' summed build seconds were 184 and their summed test-run seconds 186.
The rest of the wall time is not split by this run.
It holds the waits for builds that outlasted the previous package's tests, and each package's reads of its linked packages' files at HEAD.
The slowest packages were `internal/loomcli` (8.1 s build, 16.7 s runs, 266 tests) and `internal/fabricengine` (3.3 s, 20.8 s, 275 tests).
The machine has 12 cores.
Over the run it averaged 5.1 busy cores, of which the build used about 3.2, so other work held about 1.9.
Its one-minute load average was 2.10 at the start and 4.31 at the end.
One run, not a mean.

## Design decisions

- **The planner stays.**
  A call naming no package still plans each map against HEAD through `testcover.Selector` and re-measures only affected tests or missed packages.
  That is what keeps a later on-command build from re-measuring every package.
- **The miss-causes report's name-following is not taken.**
  `docs/research/speed/testmap-miss-causes-2026-11-15.md` finds that following the names a changed type is referenced by would narrow 64 of 119 rebuilds on one commit, and none on the other two.
  Its estimate reads syntax only, over- and under-counts, and bounds nothing; a sound build would need type information.
  The embedded-file, import and test-file misses, which were all of the small and medium commits' misses, are outside it.
  The report's larger finding, that per-round bookkeeping took 76 to 81 percent of a pass, is what the lean loop removes.
- **A retired value, not a reinterpretation.**
  `scope: coverage` leaves the accepted set instead of changing meaning in place, so it is migrated by `gateslot.MigrateConfig` like the retired list shape.
  It is also refused at load with a way forward; the hub's `gate.yaml` holds none.
- **The gate's advisory redundancy report stays.**
  A gate key's `reports: [redundancy]` step still judges a diff's new and changed tests against the maps and never changes the gate's result.
  It trims nothing, but with no upkeep its maps can be older than the code it judges.
  Whether to keep it is the operator's call.
- **No board entry changed.**
  The project `test-suite-trim` still names `testmap-maps-in-weft`, `testmap-upkeep-cost-and-guide` and `selector-non-body-miss` in its order.
  No entry with those slugs is on the board, so no planned upkeep-only work was left to remove.

## Docs

`PATTERN-test-speed` (gate key scopes, the coverage map section) and `PATTERN-daemon-wakeups` (the testmap wait) are updated.
So are `README.md`, `docs/overview.md` and the package docs of `dobbyengine`, `beamcli`, `orchcli`, `fabriccli`, `runproc`, `gateslot`, `gatecli` and `testcover`.
