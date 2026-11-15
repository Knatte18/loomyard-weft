# Build coverage maps only on command; the gate runs whole packages

Coverage maps are now built only when the operator runs `lym gate testmap`.
No dobby, landing or gate starts a build, and the gate no longer selects tests from the maps.

The payoff report (`docs/research/speed/testmap-payoff-2026-11-15.md`) found the prime dobby's upkeep pass after a landing took 7.7 to 31 minutes.
It saved the gate 0 to 4.2 s on the packages each diff touched.
The smart-build study (`docs/research/speed/testmap-smart-build-2026-11-15/report.md`) puts the cost in `lym`'s bookkeeping, not in the method.
Maps that no upkeep keeps go stale after the first landings, so a gate selecting from them cannot be trusted.
The maps stay as the input of `lym gate redundant`, which judges what each test covers so redundant tests can be cut.

## What goes

- The prime dobby's upkeep pass, with its `--upkeep` flag, its `runproc.RoleTestMap` registration and its exit record.
  `lym orch stop` and `lym beam` no longer end or wait on such a pass.
- The `test_map_upkeep` key of `beam.yaml`, with the `MigrateConfig` migration that carried `test_map_every` into it.
  `test_map_upkeep`, `test_map_every` and `test_map_batch` are all retired keys now: a layer that still holds one loads as if it did not, and a reconcile prunes it.
- The `coverage` gate scope and the map-based selection behind it (`internal/gatecli/coveragescope.go`).
  The `webster-fix`, `darn-write` and `code-burler-fix` keys in the template and docs move to scope `diff`.
- The upkeep-only parts of the slot pool (`TryAcquireSpare`, `Contended`) and of `lym gate testmap` (the yielding pass and its slot wait).
- `impactset`'s split of the unchanged packages into importers and binary builders, which only the coverage scope read.
- `TestFunc.Parallel` and its source scan, since the build no longer groups tests into rounds by it.

## What `lym gate testmap` does now

It runs only on command, in the prime worktree, over the packages named or over every stale one.
There is no default package cap per call; `--max` still caps it when given.
The measure loop follows the smart-build report, keeping one process per test and Go's block coverage:

- Each test's profile is read once.
  `testcover.ProfileByPackage` groups its blocks by package, and each linked package's `FuncIndex` names covered functions from its own group.
  `FuncIndex.CoveredFuncs` and `ProfiledFuncs` now take that group instead of filtering the whole profile per package.
- Up to the slot's tier0 parallel run side by side, in separate processes with separate profiles, whether or not a test calls `t.Parallel`.
- A test that fails or times out beside others runs once more alone, and that outcome stands.
- The map is written once per package, after its tests ran.
- The listing and coverage build of the next queued package start once the current binary is ready, so they overlap the current tests.
  A build whose package then needs no binary is cancelled.

A call naming packages waits for each package's build lock, and that wait moved to `internal/gatecli/testmapwait.go`.

## Design decisions to check

- Running tests side by side shares the package directory, so a test that writes a fixed path there can disturb another.
  The re-run alone bounds a failure beside others, but a disturbed test that still passes is recorded as it ran.
  Running them in rounds by `t.Parallel` would avoid that, at the cost of the wall time the side-by-side loop saves.
- Nothing in the diff reports what `testmap-miss-causes` recommends about following names.
  The build keeps the existing plan that rebuilds a package whole on a selector miss, now described in `internal/gatecli/doc.go`.

## To check or run by hand

- Time one full build on this machine with `lym gate testmap` over the whole module, and set it beside the report's estimate of about 16 CPU-minutes, about 4 minutes at four tests side by side.
  The diff holds no such measurement.
- Run `lym gate redundant` on a built package to confirm it reads the maps the new build wrote.
- Look at the side-by-side runs for tests that share a fixed path in their package directory.
