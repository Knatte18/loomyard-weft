```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → test that pins it**

1. Memory guard kills a test binary's tree over `memory_quit_mb` whole, and only once: `TestMemoryGuard` row "a tree over the cap is killed whole, once".
2. A tree exactly at the cap is spared: `TestMemoryGuard` row "a tree at the cap is left alone".
3. A process that is not a `.test` binary is never killed: `TestMemoryGuard` row "a process that is not a test binary is never killed".
4. When trees are nested, the inner binary is killed and the outer one spared: `TestMemoryGuard` row "an inner test binary over the cap…". The same row covers the "directory could not be read" line and the line with no `-test.run`.
5. Under a short memory limit, the largest tree at or above the floor is killed, and the pressure reason line is pinned: `TestMemoryGuard` row "a short memory limit kills the largest tree…".
6. Under a short limit, a tree below the floor is spared: `TestMemoryGuard` row "…spares a tree under the floor".
7. An ample limit, or no limit found, kills nothing below the cap: `TestMemoryGuard` rows "an ample memory limit…" and "a tree at the cap…" (no limit found).
8. A sampler error is logged and sampling continues: `TestMemoryGuard` row "a sampler failure…".
9. An unsupported platform ends the guard: `TestMemoryGuard` row "an unsupported platform…".
10. A pressure kill skips a binary that was already killed (the `killed` map in the pressure branch): none. This is minor and I did not file it.
11. `memoryGuardApplies` (limit 0, ancestor, non-ancestor, empty, unparsable): `TestMemoryGuardApplies`.
12. `guardOwnTests` runs the guard over the calling process in `lym gate redundant` and `lym gate testmap`, honours `LYM_GATE_TEST_PID`, and its stop waits for the guard to end: **none**.
13. Tier wiring of guard kills in `runInvocations`/`runTier`: the guard starts when `request.memoryGuarding` is set, a kill fails a command that exited 0, its lines reach the log and causes, and `memoryStopped` blocks a rerun in `failedOnShortage`. Pinned by: **none**. `TestRunTierProcessCap` pins the same wiring for the process cap, but `newMemoryGuard` cannot be injected through `testRequest`, so no row reaches this one.
14. A tier's `memory_mb` runs its command under `proc.RunMemoryCappedVia` through `runTiedChild`, and `capKilled` becomes `stops.memoryCap`. That means `tierMemoryCapLine` is printed, the run fails and is never rerun: **none**.
15. `runTiedChild`'s refactor keeps its uncapped behaviour, including the `killLeftovers` and `ErrWaitDelay` handling: covered indirectly by the existing `TestRunTierProcessCap` and `TestRunTierNoTestNotice`, which run real tier commands through it.
16. `proc.RunMemoryCappedVia` starts through the caller's start exactly once and reports a child killed at the cap as a tree kill: `TestRunMemoryCappedVia_StartsThroughItsStartAndReportsAChildKilledAtTheCap` (tier1; it skips without a capping systemd).
17. `RunMemoryCappedVia` on the data-segment fallback (no scope) with a caller's start: none. The export helpers use only `startDirect`. This is minor and I did not file it.
18. `RunMemoryCapped`'s result is unchanged after the refactor: the existing tests in `memorycap_linux_test.go` and `memorycap_test.go` through `RunMemoryCappedWithStarter`/`WithoutScope`.
19. `TreeSample` now carries `Parent` and `RSS`: `TestProcessTreeReads`, with its new assertions.
20. `ReadProcess`: indirectly through `TestProcessTreeReads`' `ListProcesses` assertions (`ListProcesses` now delegates to it).
21. `memory_quit_mb` key:
    - default 4096: `templateDefaults` in `config_test.go`.
    - 0 accepted: `TestLoadConfig` row "memory_quit_mb zero…".
    - negative refused: row "negative memory_quit_mb".
22. Tier `memory_mb` key:
    - loads: `TestLoadConfig` row "a tier with slots all, covers and every key loads".
    - below 1 refused: row "tier with memory_mb 0".
23. `Config.MemoryQuit` and `Tier.MemoryLimit` (MB to bytes): none directly. Their only effect shows through the uncovered wiring in items 13 and 14.
24. Watchdog rename to `ProcessSample`: existing `watchdog_test.go` rows.

**Findings**

1. **MEDIUM** · design · `internal/gatecli/tierrun.go:386-405`, `internal/gatecli/tierrun.go:117-135`, `internal/gatecli/tierrun.go:241`
   - **Gap:** no test pins how a tier run handles a memory-guard kill or a kill at its `memory_mb` cap.
   - **Effect:** no row checks that either kind of kill fails a command that exited 0, puts its line in the log and `causes`, sets `memoryStopped`, or keeps `failedOnShortage` from rerunning the tier. A regression that drops `stops.memoryLines()` from the exit, the causes or the `memoryStopped` flag would pass every test.
   - **Evidence:** `TestRunTierProcessCap` (`internal/gatecli/tierrun_test.go:137-229`) pins this wiring only for the process cap. `testRequest` has no seam for the memory guard: `newMemoryGuard` is called directly at `internal/gatecli/tierrun.go:390`.
   - **Fix:** add rows to the tier-run test for a guard kill and a cap kill.
2. **MEDIUM** · design · `internal/gatecli/child.go:63-70`, `internal/gatecli/tierrun.go:404-406`, `internal/gatecli/tierrun.go:196-201`
   - **Gap:** no test covers the `memory_mb` path end to end. That means `runTiedChild` with `memoryLimit > 0` going through `RunMemoryCappedVia`, `capKilled` set, and `tierMemoryCapLine` produced.
   - **Evidence:** a grep found no test that references `tierMemoryCapLine`, `runTiedChild` with a limit, or `Tier.MemoryLimit`. The only `RunMemoryCappedVia` test (`internal/proc/memorycap_linux_test.go:106-130`) uses a plain `cmd.Start` start, not the tied start.
3. **LOW** · design · `internal/gatecli/memoryguard.go:423-439`, `internal/gatecli/redundant.go:703`, `internal/gatecli/testmap.go:780`
   - **Gap:** `guardOwnTests` has no test: its no-op stop when the guard does not apply, and the stop that cancels the guard and waits for it to end.
   - **Gap:** nothing pins that `lym gate redundant` and `lym gate testmap` start the guard at all.
   - **Mitigation:** the decision inside it is covered by `TestMemoryGuardApplies` (`internal/gatecli/memoryguard_test.go:634-661`), and the loop by `TestMemoryGuard`. What is left is the wiring.

**Tool calls**

1. Grep of test files for the changed tier-run and proc symbols: the manifest did not list the existing tests of `runTier`, `runTiedChild` and `retryTier`.
2. Read of `internal/gatecli/tierrun_test.go:38-230`, to check whether `TestRetryTier` or `TestRunTierProcessCap` already reaches the `memoryStopped` or `memory_mb` paths: they do not.
