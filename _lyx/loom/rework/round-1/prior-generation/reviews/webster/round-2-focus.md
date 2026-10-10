---
round: 2
focus:
  - "Does `LYX_LOG_LEVEL=debug lyx reed up|attach|resume` give the spawned `lyx reed watchdog` daemon a Debug trace in the hub logs directory? Check the root `PersistentPreRunE` in `cmd/lyx/main.go` (its `logger.SetVerbosity` call with the `-v` count) against `configureFromEnv` in `internal/logger/logger.go`, and against the claims in the `watchdog` verb's `Long` help in `internal/reedcli/watchdog.go`, `internal/reedcli/doc.go` and the `SpawnWatchdog` doc in `internal/reedengine/spawnwatchdog.go`."
  - "Does a test reach the socket-path and shell boot refusals at their site inside `ensureServerAndSessionLocked` in `internal/reedengine/lifecycle.go` without passing through `withBootOpLockKeepingZoom` in `internal/reedengine/zoom.go`, with zero tmux calls? Would deleting the call in `lifecycle.go` leave every test green?"
---

Both entries re-check the sites the round-1 range touches, now against the committed fixes.
