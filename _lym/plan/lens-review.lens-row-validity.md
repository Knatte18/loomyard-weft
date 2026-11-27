```yaml
lens: row-validity
outcome: reported
```

The plan's test rows have **2 BLOCKING** findings, both in card 11's `tier1` test, and 5 MEDIUM. Rows not listed below reach their branch and would fail if the code were wrong.

**BLOCKING**

1. **Card 11, `TestPlaceWindowChild_RunsTheWindowCommandWithNoWait`: the PATH setup stops the child before it places anything.**
   - The row sets `PATH` to a directory holding only `sh`, so the spawned child finds no `qdbus6` or `kscreen-doctor`.
   - The verb's `RunE` resolves the worktree with `lymcwd.NamedWorktree` and lists worktrees with `fabricengine.List`, and both run git. `internal/lymcwd/lymcwd.go:121` calls `gitWorktreeRoot(cwd)`.
   - With no git on `PATH`, resolution fails before `Flow.Place`. So there is no `RecordOpen` and no settled places entry. The worktree's config dirs come from that same resolution, so no `afterPlace` runs and no marker appears.
   - `Spawn` takes no env parameter, so the child inherits the test's `PATH`. A `sleep` started by bare name also fails under that `PATH`.
   - Fix: keep git (and sleep) reachable, and make only the desktop tools absent. For example, use a `PATH` holding links to `sh` and `git` alone, or give `Spawn` an env seam.

2. **Card 11, the same test: the "one `RolePlacement` record" assertion races the child's exit.**
   - `internal/runproc/record.go:116,167`: a read of the records prunes every record whose process is gone.
   - The child fails fast, then runs the window command and exits. A `runproc.List` after that sees no record, so the row flakes or fails.
   - Fix: keep the child alive while the test reads the record. For example, make the window command write the marker, then wait on a file the test removes after asserting the record.

**MEDIUM**

3. **Card 1, `TestLookupTaskSettings`: the planned window_placement subtest goes through a helper the card does not list.**
   - `internal/dobbyengine#lookupAsText` (`tasksetting_test.go:31-54`) sends every key that is not land_notice, autonomy or run_watch to `LookupPullRequest`.
   - A window_placement subtest that goes through it reads pull_request and fails. If it calls `LookupWindowPlacement` directly, the helper is untouched.
   - Fix: list `internal/dobbyengine#lookupAsText` under Edit with a `WindowPlacementKey` case, or have the Intent say the subtest calls `LookupWindowPlacement` directly.

4. **Card 9, `TestPlan` row "the prime with no parent starts at the current desktop": the row cannot reach that branch.**
   - `Plan` takes the anchor from `PlanInput` and has no notion of the prime. Card 10 step 5 is what anchors the prime's window on the current desktop.
   - The row only re-tests `same` from a given anchor, and `TestFlow_Place` has no prime row.
   - Fix: move the row into `TestFlow_Place` as a request with `Parent == ""` that never calls `awaitParent`.

5. **Card 10, `TestFlow_Place` row "a waiting child makes no lock attempt until the lock frees": the assertion cannot fail.**
   - Nothing observable separates a blocking `lock.AcquireWriteLock` from a poller such as `AcquireWriteLockWithin`, which retries every 10ms (`internal/lock/lock.go:16,62`). Both return once the lock frees.
   - Fix: assert something a poller would break. For example, count acquire calls through a seam, or assert that `acquireWithin` uses the blocking acquire.

6. **Card 10, no row covers SIGTERM during a running window command.**
   - D12's criterion is that a child ended by SIGTERM "ends one already running". Card 3 tests cancellation inside desktophook only.
   - No `TestFlow_Place` row checks that cancelling `Place`'s ctx while `afterPlace` runs cancels the ctx `afterPlace` received. The flow can pass `context.Background()` and every planned row still passes.
   - Fix: add a row whose `afterPlace` blocks on its ctx; cancel the outer ctx and assert `afterPlace` returns with the cancel.

7. **Card 10, row "a parent unsettled under the lock sends the child back to the wait": the card never says how the test produces that state.**
   - `awaitParent` returns only once the parent's entry is settled. To find it unsettled under the lock, the parent's entry must be re-opened (`RecordOpen`) between the wait and the locked `Query`.
   - That works only if the test's `Query` function field writes `RecordOpen` for the parent on its locked call. The card does not say so.
   - Fix: state that mechanism in the Intent.

**LOW**

8. **Card 3, `TestAfterPlacement`: two rows repeat existing tests.**
   - `HangingCommandIsCutOffAtTheTimeout` repeats `internal/desktophook_test#TestAfterWindow`'s row of the same name, through the same `runWindowCommand`.
   - `UnsetCommandRunsNothing` largely repeats `TestAfterWindow`'s unset row. Keep it only if `AfterPlacement` has its own early return; otherwise drop it.

9. **Card 2, `TestLoadConfig` (desktophook) row `LocalPlacementRefused`: it repeats `LocalOverrideRefused`.**
   - Both rows test the configengine closure of a machine key to the local layer. The planned `wantMachineKeys` change in `TestModules_EveryTemplateKeyClassified` already pins that classification.

10. **Card 8, `TestPlacesFile` row "a key the struct does not declare survives a write": it repeats `internal/state`'s own unknown-key tests (`keep.go`).**
    - `PlacesFile` writes only through `state.UpdateJSON`, so the row adds nothing.

11. **Card 13: the fake's new hand-off record is not listed.**
    - `newStartDeps` "records hand-offs", which needs a new field on `internal/machinecli#startCalls` (`start_test.go:95-100`). The card does not list that type.

Tool calls:
- `internal/dobbyengine/tasksetting_test.go` (lines 30-60 and 244-262) and `internal/configreg/runsettings_test.go` (lines 95-120): to see the lookup test's dispatch helper and the refusal and run_settings row shapes, which my context lacked.
- `lymcwd.NamedWorktree`'s git use, `proc.ConfigureGroupTerminate`'s signature, `termwindow_integration_test.go`'s package and build line, and runproc's pruning: to check whether the rows on cards 3, 4 and 11 can be reached; my context lacked all four.
- `internal/machinecli/start_test.go` (lines 60-140): to see `newStartDeps` and `startCalls`, which my context lacked.
