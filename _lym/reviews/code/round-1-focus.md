---
round: 1
focus:
  - "Scope against the Goal: does every agent pane in every hub and engine run under the cap, or only the panes `Engine.launchStrandLocked` in `internal/reedengine/spawn.go` spawns? Find each other pane-spawn path and settle whether the diff reaches it."
  - "Cap config in `internal/reedengine/config.go` and the two templates: is the cap a hub config value with a default that fits an agent session plus builds and tests, and does a hub's own value reach `capPaneMemoryLocked`?"
  - "Degraded path in `internal/reedengine/panememory.go` and `internal/proc/processscope_linux.go`: when the scope cannot get the cap, does the pane still start with a warning that names the cause, and not a refusal or a silent skip?"
  - "OOM notice in `internal/reedcli/oomnotice.go` and `watchdog.go`: does the notice reach the run's parent and name both the pane and the killed process? Check the kill count read from the cgroup (`parseOOMKills`, `oomKillCount`) for wrong-file, race and counter-reset failures."
  - "`startScopeArgs` and the `MemoryMax` and `MemorySwapMax` properties: are both set, and does the invocation work without root on a user manager, on kernel 7.0 and on Ubuntu 24.04?"
  - "Windows and non-Linux stubs in `processscope_windows.go` and `processscope.go`: does the pane still launch normally there, and do the build tags match across the files?"
  - "Test economy and mutation survivors: the survivors at `processscope_linux.go` lines 47, 52 and 82, `panememory.go` lines 35 and 40, and `spawn.go` line 101. Decide which behavior a test could assert, such as the cap reaching the launch script or the warning text."
  - "Changes in `cmd/lym/spawnobservability_test.go` and `internal/creelcli/smoke_helpers_test.go` are not named by the Goal: check that they are licensed, and that no assertion or limit in an existing test is loosened."
---

Seed focus only. The decision record holds the board Goal alone, with empty Decisions and Acceptance criteria, so the measuring stick is the Goal's five Fix bullets.
