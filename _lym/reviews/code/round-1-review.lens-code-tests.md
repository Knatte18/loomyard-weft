```yaml
lens: code-tests
outcome: reported
```

Restating my task: for each behaviour the diff adds or changes, name the test that would fail if it broke, and file a finding for each behaviour with none.

**Behaviour → test that pins it**
1. A strand pane's shell lands in a scope whose `memory.max` is the template's 8192 MiB (the cap call in `launchStrandLocked`, `spawn.go:101`, and the template default reaching it): pinned by `TestAddStrand_ThePaneRunsInAScopeHeldToTheTemplatesMemoryCap` (tier2). That test is outside the coverage maps, which is why the `spawn.go:101` mutant survived.
2. The template default `pane_memory_max_mib: 8192` parses: pinned by `TestLoadConfig_TemplateDefaultsResolve`, `config_test.go:76`.
3. A hub's own non-default `pane_memory_max_mib` reaches the cap, and a value of 0 or less falls back to the default (`paneMemoryMax`): **none**.
4. A pane the cap cannot reach still launches: covered only incidentally. The fake tmux's `display-message` answer fails the pid parse, so the `TestLaunchStrandLocked_*` tests run the uncapped path and pass. No assertion checks it.
5. The Warn on the uncapped path names the cause (pid unreadable, no pid, `CapProcessMemory` error): **none**. This is why the mutants at `panememory.go:35` and `:40` survived.
6. `CapProcessMemory` sets `MemoryMax` and the scope reports it: pinned by `TestCapProcessMemory_KillsARunawayChildInTheScopeAndTheProcessLivesOn` (the `memory.max` assertion).
7. `MemorySwapMax=0` (`startScopeArgs`, `processscope_linux.go:82`): **none**. No test reads the scope's `memory.swap.max`, so the mutant changing that constant survived.
8. `OOMPolicy=continue`, so the pane lives on after a kill: pinned by the KillsARunaway test (`survived` line).
9. A capped process's OOM kill count (`ProcessOOMKills`): pinned by the KillsARunaway test (`kills != 1`).
10. Parsing a cgroup's `memory.events`: pinned by `TestOOMKillCount`.
11. Parsing kernel `oom-kill` lines and filtering them by cgroup: pinned by `TestParseOOMKills`.
12. `OOMKillsInCgroup` reads the journal, and journalctl's exit 1 on no match reads as no kill: weakly pinned by the KillsARunaway test (`err == nil`, with any named kill being the hog).
13. Capping a process that has already ended is an `ErrMemoryCap` error: pinned by `TestCapProcessMemory_AnEndedProcessIsACapError`. The test cannot tell which branch produced the error (see finding 4).
14. Notice planning (a new kill is told once, a first-seen cgroup counts from 0, unreadable and dead panes are skipped, gone cgroups are forgotten): pinned by `TestPlanPaneOOMNotices`.
15. Notice text (kill named, kill unnamed, journal unreadable): pinned by `TestPaneOOMNoticeLine`.
16. The watchdog cycle posts each notice to the parent orch (`reportPaneOOMKills`: `Status` → plan → journal → `NotifyAsker`, the step-7 call in `runWatchdogLoop`, and `oomKills` initialised in `enterSession`): **none**.
17. The Windows stubs return `ErrMemoryCap` with `ErrUnsupported`, and `ErrUnsupported` alone for the two OOM reads: none. These are platform-only branches, so no finding.
18. The `creelcli` smoke helper rewrite (engine `Up`, `Down` and `AddStrand` instead of the CLI): this is test support, not a behaviour. `addProviderStrand` still fails on an `AddStrand` error.
19. The spawn-observability allowlist entry: this is the guard's own data, so no finding.

**Findings**
1. MEDIUM · design · `internal/reedcli/oomnotice.go:101` (`reportPaneOOMKills`) and `watchdog.go:624`. Nothing tests that a new kill actually becomes a creel notice to the parent orch. The planner and the line formatter are tested in isolation, but a wiring break (step 7 dropped from the loop, `oomKills` nil, a wrong receiver or signer) passes every test. The Goal's fourth bullet asks for exactly this behaviour. Evidence: grep finds no test referencing `reportPaneOOMKills` or `oomKills`.
2. MEDIUM · design · `internal/reedengine/panememory.go:33-41` (`capPaneMemoryLocked`). No test asserts that a pane the cap cannot reach logs a warning naming the cause, which the Goal's third bullet requires. The mutants at lines 35 and 40 survive. A tier0 `TestLaunchStrandLocked_*` row with the fake tmux's `display-message` scripted, plus a log capture of the Warn, would pin it. Evidence: `faketmux_test.go:278` handles `display-message` generically, and no test greps the warning.
3. LOW · design · `internal/reedengine/panememory.go:20-26` (`paneMemoryMax`). No test shows a hub's own `pane_memory_max_mib` reaching the cap, or a value of 0 or less flooring to the default. Only the template default is asserted, so a mistake in the floor or in the MiB-to-bytes shift for a non-default value goes unseen. Evidence: grep finds only `config_test.go:76`.
4. LOW · design · `internal/proc/processscope_linux_test.go` (the KillsARunaway test). The Goal names `MemorySwapMax` explicitly, yet no assertion reads the scope's `memory.swap.max`. The mutant of the `"0"` at `processscope_linux.go:82` survives. Add an assertion that `memory.swap.max` is `0` beside the existing `memory.max` check. Evidence: `processscope_linux_test.go:343-346` checks only `memory.max`.
5. NIT · design · `internal/proc/processscope_linux_test.go:377-393` (`TestCapProcessMemory_AnEndedProcessIsACapError`). The test asserts only `errors.Is(err, ErrMemoryCap)`, so it passes whichever branch produced the error. That is why the mutants at `processscope_linux.go:47` and `:52` survive. An assertion that the error names the user manager's refusal would make it pin the refusal path it is named for.

**Tool calls**
- One grep over the reedengine tests for `display-message`/`pane_pid`, and over all tests for `reportPaneOOMKills`, `oomKills`, `PaneMemoryMax`, `memory.swap.max` and `MemorySwapMax`. My context showed which tests reach these functions but not which assert the warning, the wiring, a non-default value or the swap cap.
