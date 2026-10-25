# Dobby, the beam daemon that drives batten passes

All twelve batches (cards 1 to 42) recorded `done`, with no recovery and no departures (no digest listed a `Pattern-Departure:` trailer).
Head after the last batch: `4ffc28d06`.

## Batches

- **01 (cards 1-6):** beam-scoped program sender in agentname, scoped creel and orch notices, the dobby runproc role, dobbyengine records (geometry, state, pass exit codes, config), `draft` as a board status at the commit point, and the one-time draft migration.
  Deviations: boardengine (`render.go`, `replay.go` and tests), dobbyengine record files and tests.
- **07 (cards 7-12):** draft settlement writers and start guard, the shed `Yield` outcome, yield handling in the step envelope and Go driver, and idle-watch and failure-episode snapshots.
  Deviations: `battencli/drive.go`, `battencli/startguard.go`, `shedverbs/loop_test.go`.
- **13 (card 13):** InnerRun's one check per pass, with a per-run memory record.
  Deviations: battencli `run_test.go` and battenshed InnerRun, notice and pause files with their tests.
- **14 (cards 14-17):** spawn hold yields, respawn cap, API-error revive, and the dropped ready snapshot and `landed` notice.
  Deviations: battenshed `land`, `notice`, `innerrun` files and tests, and the `Land.ReadyTasks` removal from loomcli and shedrecipe test fixtures.
- **18 (cards 18-20):** Land and Teardown failure notices after the stop chain, arbiter keys in shed.yaml, and the loom, darn and batten overrides.
  Deviations: battenshed `land`, `teardown`, `rownotice`, `config`, loomengine and sheddriver config, envkit, and wiring seam tests.
- **21 (cards 21-22):** arbiter key copy into darn.yaml and the new `internal/shedarbiter` package.
  Deviations: `configreg_test.go`, `configsync/movekeys.go`, `loomcli/arbiterprompt.go`, shedarbiter files.
- **23 (cards 23-24):** loomcli delegates to shedarbiter, and batten's arbiter eligibility rule.
  Deviation: `arbiterstop/arbiterstop.go`.
- **25 (cards 25-27):** batten's arbiter wiring, pass exit codes for owned-elsewhere and covered refusals, and dobby start.
  Deviations: battencli `arm.go`, `drive.go`, tests, dobbyengine `geometry.go` and `start` files, `hubgeom/dobbygeom.go`.
- **28 (cards 28-30):** `--window` retired with a detached first pass, the origin-owed marker, and the continue pass.
  Deviations: battencli `arm.go`, `cli.go`, `resumereconcile.go`, and `shedrun/paths.go`.
- **31 (cards 31-35):** InnerRun pass seams from the scheduling dobby, dobby loop, pass scheduling, owned-elsewhere handling, and ready starts.
  Deviations: battencli `wire.go` and `dobbyreading_test.go`, dobbyengine `owned.go`, `passes.go`, `starts.go` and tests.
- **36 (cards 36-39):** board-tidy agent, `lyx beam start` and `lyx beam dobby`, `lyx orch start` starting the dobbies first, and adopt refusals.
  Deviations: adoptcli, beamcli, `battencli/dobbywire.go`, dobbyengine `state.go` and `tidy` files, `orchcli/start.go`.
- **40 (cards 40-42):** dobby-driven liveness, reed down sparing dobby, Finalize's key-by-key config notice, and the orch role stencil.
  Deviations: new `configengine/changedkeys.go` and test, `fabricengine/configchanges.go`, `landingshed/finalize_integration_test.go`, `pairteardown/teardown.go`.

## Verify gate findings

The plan-level verify failed in `cmd/lyx`, where the CLI-form test `TestCLIForm_EveryCommand` found a `Long` help text over its size cap.
One fixer fork shortened `lyx orch start`'s help text in `internal/orchcli/start.go`.

## Scope guard

Every deviation above is informational: files changed outside the cards' listed targets, which the forks reported as follow-on edits to the listed ones.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `28fe6e3f210d07e1becf93ca460a99d98234e4e9` fix: trim lyx orch start's Long under the 2000-byte cap
