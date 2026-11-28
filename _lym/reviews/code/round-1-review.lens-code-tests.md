```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens, mapping each behaviour the diff adds or changes to the test row that would fail if it broke. This is from reading the tests; I ran no test or mutation.

## Behaviour → pinning test

**Config and settings**
- `beam.window_placement` lookup, refusal, local/run_settings check: `TestLookupTaskSettings` window_placement subtest, the refusal row, dobbyengine `TestLoadConfig`, `TestPrepareRunSettings`
- Desktop keys, defaults, override, machine-only: desktophook `TestLoadConfig`, `TestModules_EveryTemplateKeyClassified`
- Placement and tile-width refusals: `UnknownPlacementRefused`, `ZeroTileWidthRefused`
- Tile-height refusal: **none** (F10)
- `AfterPlacement` (empty, bad config, no wait, cancel ends the group): `TestAfterPlacement`; timeout: `TestAfterWindow`

**Launch and window verbs**
- `--separate` arguments, `IsKonsole`, `Open` and its pid: `SeparateKonsole`, `TestResolve_...`, `TestLauncher_OpenRunsTheChosenCommandLine`
- Pre-launch checks: `TestPreLaunch`; `OpenWindow` passing their result to `Open`: **none** (F10)
- `reed window` hand-off, and fallback on failure: `TestWindowCmd` new rows
- `handTerminalWindow` outside a worktree: its integration test; inside a worktree: **none** (F4)
- `machine start` hand-off, warning, unplaced: `TestStart_HandsEachPlacedWindow...`

**Placement child**
- Spawn, runproc registration, prime/unmarked, pre-run skip, settle on failure, refused mode: `TestPlaceWindowChild`
- Resolving parent, mode, tile sizes, prime and worktree names: **none** (F2)
- SIGTERM to the verb: **none** (F3)

**Readers and runner**
- Report parsing over recordings, no-output, end marker, other run ids: `TestRecordings_EveryRecordingReads`
- Undecodable line or unknown kind: **none** (F10)
- Enabled-output filter: `Kscreen`; lowest priority: **none** (F5)
- `ApplyScript` input: `TestApplyScript_...`; `QueryScript` input: tier3 only (F10)
- `Window.Matches`: `TestWindow_Matches`
- Refused load, call timeout, leftover, stays loaded, end-marker timeout, cancel, held lock, shared report: `TestRunner_ScriptRuns`
- Journal and unload re-polling, journalctl arguments, script object path: **none** (F7)
- Cache not reused for another tiling output: **none** (F8)
- Apply takes the script lock, runs don't overlap: only a timing-dependent test (F9)
- `acquireWithin` bound and abandon: `TestAcquireWithin`; its ctx-cancel branch: **none** (F3)

**Places, planner, flow**
- D9 update rules: `TestUpdatePlaces`; file read, heal, record, settle, concurrency: `TestPlacesFile`
- Grid sizes, anchor, current order, plan rules: `TestCapacityAndGrid`, `TestAnchor`, `TestCurrentOrder`, `TestPlan`
- `next` starting past the last desktop, and anchor not among the desktops: **none** (F6)
- Window, parent and lock bounds, backoff, off mode, probe failure, no output, failed apply, unwritable places: `TestFlow_Place` rows
- Cancel before or during the command: two cancel rows; cancel during the waits: **none** (F3)
- `reportUpdateFrom` using the apply report and recording other windows: **none** (F1)
- Contract with real KWin: the two `TestContract_*` (F11)
- Guard list additions (daemon-wakeups, hermetic allowlist, pattern rows): the existing guards

## Findings

**F1. MEDIUM, design.** `internal/kwinplace/flow_test.go` (`ReopeningUnderAPanelKeepsEveryKey`), for `flow.go` `reportUpdateFrom`.
No test fails if `reportUpdateFrom` ignores the apply report or records only the placed window.
- The row's keys 0, 1, 2 equal the stacking order of windows 1, 2, 3. With `tiles = report.Tiles` removed, `snap.Tiles` is nil, `CurrentOrder` falls back to stacking, and the keys come out the same.
- Dropping the join of applied desktops onto the snapshot windows also leaves every assertion true.
- No flow row asserts that another worktree's entry changes, so hand moves and renumbering through the flow are unpinned. This is D9's "the places file holds every reported window".
- `TestUpdatePlaces` feeds `SeenWindow` values directly and does not cover this.
- Evidence: `flow.go` `reportUpdateFrom`; `flow_test.go` that row and `planWindow` (`Stacking: n`).

**F2. MEDIUM, design.** `internal/reedcli/placewindowchild.go` `placementRequest`.
Nothing asserts what the verb resolves: the parent (including `""` becoming the prime's name), the empty slug for the prime, per-task over machine mode, the tile sizes, or the `Worktrees` basenames.
- `TestPlaceWindowChild` fails every run at `ReadTilingOutput` and checks only settle, marker, records and the refusal logs.
- A wrong `Worktrees` list would make `updatePlaces` drop live entries with every test green.
- Evidence: `placewindowchild_integration_test.go` rows.

**F3. MEDIUM, design.** `reedcli/placewindowchild.go`, `kwinplace/runner.go` `acquireWithin`, `flow.go`.
D12 and D9 under SIGTERM are only partly pinned.
- No test signals the child, so removing `signal.NotifyContext` fails nothing. SIGTERM would then kill it with no settle and a running window command left behind.
- No row cancels during the window wait, the parent wait or a lock acquire. The `ctx.Done()` branch of `acquireWithin`, the `ctx.Err()` returns in `awaitWindow`, `awaitParent` and `takeLock`, and `sleepWithin`'s ctx arm have no coverage.
- Evidence: `flow_test.go` has only `ACancelBeforeTheCommandSkipsIt` (cancel inside apply) and `ACancelDuringTheCommandReachesIt`; `TestAcquireWithin` never cancels.

**F4. LOW, design.** `internal/reedcli/window.go` `handTerminalWindow`.
The in-worktree path (prime detection, and the hub, worktree, title and pid passed to `Spawn`) has no test. `TestWindowCmd` stubs it, and its only direct test is the outside-a-worktree case. `TestPlaceWindowChild` calls `Spawn` itself.

**F5. MEDIUM, design.** `internal/kwinplace/recordings_test.go` `Kscreen`.
D4's "lowest priority" is unpinned. In both recordings DP-2 (priority 1) is the first enabled output, before HDMI-A-1 (priority 2). A reader returning the first enabled output passes. Reordering the recorded outputs in the test would pin it.
Evidence: `neve-kscreen/kscreen.json` lines 805-838 and 1225-1234.

**F6. LOW, design.** `internal/kwinplace/planner.go` `Plan`.
Two boundaries have no `TestPlan` row:
- `next` anchored on the last desktop, where `start == len(desktops)`. Without the `min` clamp it would panic.
- An anchor that is `""` or a removed desktop (a settled parent with no live window), which falls back to the current desktop.

**F7. LOW, design.** `internal/kwinplace/runner.go` `readReport`, `unload`, `runLocked`.
- No row has the end marker appear on a second journal read, or the script leave on a second `isScriptLoaded`. A single-check implementation passes everything, against D3's "reads again until".
- `recordedExec` ignores the journalctl arguments (unit, `--since`, `--user`) and the `/Scripting/Script<id>` object path, so a wrong one fails no tier0 test.

**F8. LOW, design.** `internal/kwinplace/runner.go` `Query`.
No row has a cached report for a different tiling output, so removing `cached.TilingOutput == tilingOutput` passes. `NewerSharedReportLoadsNothing` uses DP-2 only.

**F9. LOW, design.** `runner_test.go` `TestRunner_OverlappingRunsTakeTheScriptLockInTurn`, `flow_test.go` `TwoFlowsApplyInTurn`.
Neither forces an overlap, so both can pass with the lock broken.
- The runner test starts two goroutines over an instant fake; with no script lock they would usually still not interleave. No other row shows that `Apply` takes the lock.
- In the flow test the first apply returns as soon as the second *reaches* the acquire, so releasing the lock before apply would usually go unnoticed.

**F10. NIT, design.** Smaller unpinned behaviours:
- the `tiling.min_tile_height` refusal (`desktophook/config.go`);
- `OpenWindow` passing `placed` to `Launcher.Open` (`prelaunch.go`);
- the parse error on an undecodable line or unknown kind (`report.go`);
- `QueryScript`'s input JSON names (`scripts.go`);
- `Place` joining `afterPlace`'s error into its return, since the harness's `afterPlace` always returns nil (`flow.go`).

**F11. BLOCKING, design.** `internal/kwinplace/contract_test.go` `TestContract_TheRealKWinAppliesAsRecorded`.
It does not compare what lym reads from the real apply with the recording, against `PATTERN-test-stand-ins` ("compares ... the fields, the event kinds") and card 8 ("compared by kind and fields with `neve-apply`").
- It never compares line kinds, and never checks `report.Tiles`, `Zone` or `Tiled`.
- Its only use of the recording is that `recorded.Area` is non-zero, which says nothing about the live run.
- With `Rows: 1, Cols: 1` the script's `leavesOf(root)` is empty, so the run cannot read any tile even if asserted.
- The read-only test also compares neither the `neve-qdbus/load.txt` answer, the kscreen shape, nor the `neve-query-no-output` kinds.
- Evidence: `contract_test.go`; `apply.js` `leavesOf` and the `report("tiles", ...)` line.

## Tool calls
1. `grep` of name, enabled and priority in the two `kscreen.json` recordings: the manifest held no kscreen recording contents, and F5 depends on output order.
