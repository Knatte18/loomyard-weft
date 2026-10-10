---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedcli/watchdog.go:660
    summary: The documented LYX_LOG_LEVEL=debug route to a Debug trace of the detached daemon does not work, because the root hook calls SetVerbosity with the -v count and resets the level the variable set
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/reedengine/lifecycle.go:299
    summary: The socket-path and shell refusals at the planned site in ensureServerAndSessionLocked are reached by no test; the Up row passes through an unplanned duplicate call in zoom.go
---

### [BLOCKING] LYX_LOG_LEVEL=debug does not give the spawned daemon a Debug trace

**Location:** `internal/reedcli/watchdog.go:660` (the `watchdog` verb's `Long` help), `internal/reedcli/doc.go:12`, `internal/reedengine/spawnwatchdog.go:33`, `internal/logger/logger.go:374`

**Issue:**
Cards 2 and 3 exist so that `LYX_LOG_LEVEL=debug lyx reed up|attach|resume` starts a daemon whose Debug lines reach the hub logs directory.
Card 2 keeps `LYX_LOG_LEVEL` in the daemon's environment, and card 3's help text and the package doc name this as one of the two routes.
The logger reads `LYX_LOG_LEVEL` once at init (`configureFromEnv`, `internal/logger/logger.go:353`).
`cmd/lyx/main.go:96` then runs `logger.SetVerbosity(verbosity)` in the root `PersistentPreRunE`, unconditionally, and with no `-v` the count is 0, which sets the level back to Warn.
The spawned `lyx reed watchdog` goes through that hook, so it starts at Warn whatever the variable holds, and the durable sink never admits a Debug record.
The diff's own benchmark write-up records this: `docs/benchmarks/daemon-idle-cpu.md` ("At `LYX_LOG_LEVEL=debug` the foreground daemon wrote no Debug lines to stderr and none to the durable log, because the CLI's root hook calls `SetVerbosity` with the `-v` count, which resets the level the variable set").
The help text, `reedcli/doc.go`, the `SpawnWatchdog` doc and the `Debug` doc in `logger.go` still state the route as working.
The foreground `-vv` route does work.

**Fix:**
Make the claim true or remove it.
Either let the root hook leave the environment's level in place when no `-v` is given (so `SetVerbosity(0)` does not lower a level `LYX_LOG_LEVEL` raised), with a test over the CLI hook; or drop the `LYX_LOG_LEVEL` route from the help, `reedcli/doc.go`, the `SpawnWatchdog` doc and the `Debug` doc, and have the spawn pass the daemon its verbosity another way (for example a `-vv` argument).
If the plan's premise (decision D1, "set by `-vv` or `LYX_LOG_LEVEL=debug`") is what is wrong, say so to `ly:orch` rather than leaving the contradiction in the benchmark's deviation list.

### [MEDIUM] Boot refusals at the planned site have no test that reaches them

**Location:** `internal/reedengine/lifecycle.go:299`, `internal/reedengine/zoom.go:85`

**Issue:**
Cards 7 and 9 place the socket-path and shell refusals in `ensureServerAndSessionLocked`, after `validateBootConfig` and before `probeCapabilityLocked`, so `TestUp_BootValidation`'s zero-tmux-calls rows cover them.
The diff also adds `e.preflightBootHost()` to `withBootOpLockKeepingZoom` in `zoom.go`, which no card lists, because `keepZoomLocked` would otherwise read tmux before the refusal.
`Up` and `Resume` both enter through `withBootOpLockKeepingZoom`, so the `SocketPathTooLong` and `ShellUnresolvable` rows are refused by the `zoom.go` call and never reach the call in `ensureServerAndSessionLocked`.
Deleting the call at `lifecycle.go:299`, or moving it after the probe, keeps every test green.
The only paths that depend on the planned site alone are `EnsureSession` and `upLocked` reached through `withOpLockKeepingZoom`, and the only direct caller of `ensureServerAndSessionLocked` in the tests (`revive_test.go`) sets a resolvable shell and a short socket path.
The refusals also run twice per `Up`, once per call site.

**Fix:**
Add a row or assertion that reaches `ensureServerAndSessionLocked` without the zoom wrapper (for example a case in `TestEnsureServerAndSession_RevivalTrigger`'s style, or `EnsureSession` on a cold session) and expects the refusal with zero tmux calls.
Keep or drop the `zoom.go` call deliberately; if kept, state in `zoom.go` or `doc.go` that the two sites are intentional and why the second is needed.
