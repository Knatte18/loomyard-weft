# Card 2 — spawn-watchdog-env

**Create:**
- `plan:internal/reedengine#spawnWatchdogEnv` -> `func spawnWatchdogEnv(environ []string) []string`

**Edit:**
- `internal/reedengine#SpawnWatchdog`
- `internal/reedengine/spawnwatchdog_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision D2, the spawn half: `SpawnWatchdog` passes the detached daemon its own environment minus `LYX_LOG_FILE`, so once card 3 lets the daemon's stderr half through, an inherited `LYX_LOG_FILE` cannot reroute a detached daemon's stderr half into an uncapped, unswept file for the daemon's whole life.
`LYX_LOG_LEVEL` and everything else pass on unchanged, so `lyx reed up`, `attach` or `resume` run with `LYX_LOG_LEVEL=debug` spawns a daemon that starts at Debug.

`spawnWatchdogEnv` returns `environ` with every entry whose key is exactly `LYX_LOG_FILE` dropped, in the shape `stripTraceID` in `lifecycle.go` already uses.
`SpawnWatchdog` sets `cmd.Env = spawnWatchdogEnv(os.Environ())` before `proc.Detach`.
Its doc comment and the file comment stop saying the child's stderr is discarded: the child's nil stdio gives it the null device, and the environment keeps the operator's log file out of it.

Test: a new `TestSpawnWatchdogEnv_DropsLogFileOnly` in `spawnwatchdog_test.go`, a table over an environ carrying `LYX_LOG_FILE`, `LYX_LOG_LEVEL`, a longer key sharing the `LYX_LOG_` prefix and an unrelated key, asserting exact equality of the result with the input less the one entry, and an environ without the key returned as it was.
The test spawns nothing, like the file's existing test.
The `internal/reedengine` row in `test-budget.yaml` rises by one, to 243.

**ImpactSummary:** One spawn site gains an explicit environment; `reedcli`'s `ensureWatchdogSpawned` and every other caller are untouched, and the only observable change is the absent `LYX_LOG_FILE` in the daemon's environment.
