# Relevant code

- `internal/reedcli/watchdog.go` `runWatchdogLoop`, `sessionsAreIdle`, `discoveryWake`, `consumeDiscoverSignal`, `exeReplaced`, `errWatchdogBinaryReplaced`, `watchdogCmd`, `watchdogLockFileName` — the daemon loop, its idle exit, its replaced retire and respawn, and its lock
- `internal/reedcli/up.go` `downCmd` — the down verb that gains the bounded lock wait
- `internal/reedcli/spawnwatchdog.go` `ensureWatchdogSpawned` — the CLI's spawn route the daemon's successor also takes
- `internal/reedengine/lifecycle.go` `Engine.Down`, `touchDiscoverSignal` — the server teardown that writes the stop, and the cold-boot touch that truncates it
- `internal/reedengine/spawnwatchdog.go` `SpawnWatchdog` — the detached spawn of the daemon
- `internal/reedengine/watchdog.go` `DiscoverSignalFileName` — the discover signal file's name
- `internal/reedengine/panebin.go` `lyxBinEnvKey`, `paneBinPrelude`, `composePaneLaunchLine` — the pane prelude that exports `LYX_BIN` and prepends its directory
- `internal/shuttleengine/claudeengine/command.go` `envFileKey`, `envFileName`, `envFileContent`, `buildLaunchCmd` — the `CLAUDE_ENV_FILE` content that gains the PATH statement
- `internal/shuttleengine/claudeengine/doc.go` — states the env file's current one-statement content
- `tools/sandbox/SANDBOX-REED-SUITE.md` — the reed suite's `command -v lyx` pre-condition
- `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` — the shuttle suite's agent-pane pre-condition
- `tools/sandbox/SANDBOX-CREEL-SUITE.md` — the creel suite's pre-condition and its bash and plain Claude strands
- `internal/creelengine/reminder.go` — the reminder tells a strand to run bare `lyx creel read`
