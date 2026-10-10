---
round: 4
focus:
  - "D4 in `_lyx/discussion/decision-record.md` names one binary for the orch watcher's help index and for the orch pane's `LYX_BIN`. Settle whether they are the same on every path that starts the watcher or the strand: the `StartRelaunch` branch of `internal/orchcli/start.go`, the `StartSpawnWatcher` branch (`orchengine.DecideStart`, `spawnWatcher` from `os.Executable()` in `internal/orchcli/spawn.go`), and a reed resume relaunch (`composePaneLaunchLine` in `internal/reedengine/panebin.go`). Where they differ, does the record either give the watcher the pane's binary or state the mismatch with its causes?"
  - "D4's Bound in `_lyx/discussion/decision-record.md` says a missing recorded path or a failed run makes the render fail and the watcher go idle. In `internal/orchengine/watcher.go`, `startClearing` and `reloadAfterCompaction` turn a `RenderRoleFile` error into `w.toIdle`, while `startAutoReload` returns it as a tick error, and `internal/orchengine/watchloop.go` ends the watcher after `maxConsecutiveTickErrors` consecutive ones. Settle whether the stated bound, the acceptance criteria and Scope's out-list (which excludes the watcher's reload steps) agree with the behaviour at each of the three sites."
  - "D9 in `_lyx/discussion/decision-record.md` says a group's `Long` does not restate its subcommands. The root `Long` in `cmd/lyx/main.go` lists every module, and `TestCLITree_EveryCommand`'s subtest `root.Long names every mounted module` requires it. Settle whether the record says the root is exempt or removes the list and the subtest."
---

Each entry names a site and the question to settle; none caps a severity or states a verdict.
