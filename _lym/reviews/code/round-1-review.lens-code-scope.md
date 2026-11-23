```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the Goal bullets in the decision record and found no scope findings.

**Decisions.** The record has empty Decisions and no `Added after Discussion` entries, so the decisions are the five Fix bullets under `## Goal`:

- **G1, the cap on every pane.** Every agent pane lym spawns, in every hub and engine, runs in a systemd user scope with `MemoryMax` and `MemorySwapMax`.
- **G2, the config value.** The cap is a hub config value, with a default picked from the memsample journal.
- **G3, the warning.** A pane whose scope cannot get the cap starts with a warning naming the cause, not a refusal.
- **G4, the notice.** An OOM kill in an agent's scope reaches the run's parent as a notice naming the pane and the killed process.
- **G5, the hosts.** It works without root on Ubuntu 26.04 with kernel 7.0 and on Ubuntu 24.04.

**Changes and what they trace to:**

1. **`proc` scope primitives** (`processscope.go`, `processscope_linux.go`, `processscope_windows.go`, `doc.go`). `CapProcessMemory` uses busctl `StartTransientUnit` with `MemoryMax`, `MemorySwapMax=0`, `OOMPolicy=continue` and `CollectMode`. It builds G1. The user-manager D-Bus call needs no root, which builds G5. Its `ErrMemoryCap` error causes feed G3.
2. **`proc` OOM reading** (`ProcessOOMKills`, `OOMKillsInCgroup`, `parseOOMKills`, `oomKillCount`): builds G4.
3. **`proc` tests** (`processscope_linux_test.go`, `processscope_parse_linux_test.go`, `export_linux_test.go`): forced by changes 1 and 2.
4. **The pane hook.** `panememory.go` (`capPaneMemoryLocked`) and the `spawn.go` call in `launchStrandLocked` build G1. The Warn on every failure path, with the launch going on, builds G3.
   - Every strand-realizing path funnels through `launchStrandLocked`, as the grep showed: `hold.go:113`, `strand.go:423/458/617` and `lifecycle.go:882`.
   - `OpenWindow` runs a `lym` command in a hidden window, not an agent (`window.go:52-66`), so leaving it uncapped is no G1 gap.
   - Selvage and the shell window are not agent panes.
5. **The config key.** `config.go` (the `PaneMemoryMaxMiB` field and its category), both templates and `defaultPaneMemoryMaxMiB` in `panememory.go` build G2. The 8192 default rests on the summary's memsample reasoning.
6. **The `reedengine` tests and doc** (`config_test.go`, `panememory_integration_test.go`, `doc.go`): forced by changes 4 and 5.
7. **The watchdog notice** (`oomnotice.go`, the `watchdog.go` struct fields, `enterSession` and cycle step 7, `reedcli/doc.go`, `oomnotice_test.go`): builds G4.
8. **`cmd/lym/spawnobservability_test.go`.** The allowlist entry for `processscope_linux.go`, which spawns busctl and journalctl, is forced by change 1 under PATTERN-child-processes.
9. **`internal/creelcli/smoke_helpers_test.go`.** Forced by change 7: `reedcli` now imports `orchcli`, which reaches `creelcli`, so the test's `reedcli` import closed a cycle (commit fa1c3aa80). Its helpers now call `smoke.reed.Up`, `Down` and `AddStrand` directly. No assertion of a test is loosened: the replaced GUID-parse check was helper plumbing, and `addProviderStrand` returns the strand from the engine.

**Decisions and the changes that build them:**

| Decision | Built by |
|---|---|
| G1 | 1, 4 (tests 3, 6) |
| G2 | 5 (tests 6) |
| G3 | 1 (error causes), 4 (Warn and go on) |
| G4 | 2, 7 (tests 3, 7) |
| G5 | 1: busctl `--user` with no root, checked live on this host; Ubuntu 24.04 is not verifiable from here, but nothing in the diff needs root or a newer systemd than 24.04 ships |

**Findings:** none. Every change traces to a Goal bullet or is forced by one that does, and every Goal bullet has a change that builds it.

Tool calls:
1. I grepped for `OpenWindow` and `OpenWindowIn` callers and read `internal/reedengine/window.go:55-80`. My context did not show what `OpenWindow` runs, which I needed to settle whether another pane-spawn path holds an agent.
