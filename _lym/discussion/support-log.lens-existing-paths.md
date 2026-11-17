```yaml
lens: existing-paths
outcome: reported
```

Task: review the decision record through the existing-paths lens, checking which code already running writes, reads or reapplies the state each decision touches.

**Findings**

1. MEDIUM — D1, detached runner. A `--map` call can join a normal call's detached run, and the record doesn't say otherwise.
   - Evidence: `internal/gatecli/detachfront.go` (`detachKeyDir`) hashes the verb, the run directory and the call's words, and calls with the same key share one current run.
   - The words builder (around lines 360–378) appends `--guards`, the targets and the `--` args. Unless `--map` is added to the words, an agent's `lym gate test --map internal/x` keys the same as `lym gate test internal/x`.
   - It then joins a live plain run and prints that run's result instead of building maps, or a plain call joins a map run.
   - Fix: D1 should state that `--map` is part of the detach key, so a map call never joins a plain call's run, and add an acceptance criterion for it.

2. MEDIUM — D7, run records. The map run's record has no label of its own, so it mixes with manual tier runs in `lym gate report`.
   - Evidence: `report.go` `recordLabel` groups a record by its gate key, or by its tier list and scope form when the key is empty.
   - A map run has no gate key, so a record shaped like a tier run (tiers 0,1, scope `all`) totals with manual `--upto 1 --all` runs and skews their counts and seconds.
   - Fix: D7 should say the map run's record carries a mark of its own that `lym gate report` groups apart, or that it writes no run record and only a log.

3. LOW — D7 and D1, green record and carry. This survives only if map mode stays off the shared report path.
   - Evidence: `doc.go` says each listed tier first subtracts what `greenRecord.covered` (`green.json`) holds green for the current tree, and `reportCarried` passes a carried tier.
   - The record says a map run writes no green record (D7) and runs no carry (D1 bound), which is right. But if the plan routes the map run through the tier path (`runWithSlots`, then `e.report`), passes would write `green.json` entries, and a green tier 0 would carry the map run as "passed" without measuring.
   - Suggest: state in D7 that map mode bypasses both the green subtraction and the green write.

4. LOW — D8 and D9, files left in the maps directory. Removing the build lock leaves `<map>.build.lock` files in existing map directories.
   - Evidence: the lock path is `testMapBuildLockPath(mapsDir, pkg)`, which sits beside the maps.
   - Readers I checked open maps by package path (`coverageMapsDir`, plus `testcover.Dir(BoltDir)` in the redundancy step), so they survive the leftovers. The record could say the old lock files are inert and need no cleanup.

5. LOW — D8, the report step reads only the hub's maps (an existing gap, unchanged).
   - Evidence: `redundancy.go:286` reads `testcover.Dir(fabricengine.BoltDir(...))`, while `redundant.go:156` reads `coverageMapsDir(location)`.
   - Maps a non-prime worktree builds with `--map` reach `lym gate redundant` but never the gate's `redundancy` step. D8's claim that "nothing changes what they read" holds; this is not a regression.

**Paths traced, per decision**

- D1:
  - Detached front's join key: **breaks** (finding 1).
  - Cobra flag parsing next to `--gate`, `--tier`, `--upto` and `--guards`: survives, given the refusals.
  - The `LYM_GATE_TEST_PID` guard-off rule for a nested call: survives.
- D2: the old build-lock wait goes with the verb, and no other caller takes the lock (survives). The run-record ordering read in `testmap.go` goes (survives).
- D3: `testcover` reduce and merge, and `lym gate redundant`'s reuse of `RunAlone`: survives.
- D4:
  - `memoryCappedParallel` is shared with `lym gate redundant`: survives, as long as the tier's `memory_mb` clamp is added through a parameter and not changed for `redundant`.
  - `guardOwnTests`: survives.
- D5: alone-rerun in the existing loop: survives.
- D6: green record and verify are untouched by a refusal: survives.
- D7:
  - `green.json` subtraction and write: survives only if map mode stays off the shared path (finding 3).
  - `lym gate report` grouping: **breaks**, mixes with manual runs (finding 2).
  - `chooseCap`/`longWait` weigh only records with a gate key: survives.
  - `lym gate status` lists pool holders: survives.
- D8: `redundant.go` reads via `coverageMapsDir`: survives. `redundancy.go` reads `BoltDir`: survives, unchanged (finding 5). `state` lock on map writes: survives.
- D9: `redundant.go` refusal text (lines 104, 129, 162) and `cmd/lym/daemonwakeups_test.go` row: the record covers both.

**Tool calls**

- Grepped run-record readers and the join logic in `detachfront.go`: my context had not seen the detached runner's join rule or who reads run records.
- Read `detachKeyDir`, `chooseCap` and the run-record write in `run.go`: to see what the detach key hashes and whether carry or record writes reach a map run.
- Grepped green-record functions, status holders and the map-dir readers in `redundancy.go` and `redundant.go`: to confirm how carry and map reads behave.
- Read `report.go`'s `recordLabel` and the dir helpers in `coveragemap.go`: to see how a record without a key is grouped.
