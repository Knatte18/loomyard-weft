# Card 5 — Sweep at process exit with configured bounds

**Edit:**
- `internal/logger/sink.go`
- `internal/logger/sink_test.go`
- `internal/logger/logger.go`
- `cmd/lyx#main`
- `cmd/lyx#run`
- `cmd/lyx/main_integration_test.go`
- `docs/shared-libs/README.md`

**Create:**
- `internal/loggerconfig/exit.go`
- `internal/loggerconfig/exit_test.go`

**Uses:**
- `plan:internal/logger#SinkArmState`
- `plan:internal/logger#CurrentSinkArmState`
- `plan:internal/logger#DefaultRetentionBounds`
- `plan:internal/loggerconfig#Load`
- `internal/logger#Sweep`
- `internal/logger#NotifyExit`
- `internal/logger#Warn`
- `internal/configengine#ConfigFile`

**Intent:**
Move the sweep from sink arm to process exit and feed it the configured bounds (decision record D3, D5).

In `sink.go`'s `armDurableSinkLocked`, remove the `Sweep` call; arming no longer sweeps.

In `internal/loggerconfig/exit.go`, add `NotifyExitAndSweep(code int)`, which runs the exit path in D3's order:

1. `logger.NotifyExit(code)`, which force-arms the sink on a non-zero code, unchanged.
2. `logger.CurrentSinkArmState()`.
3. Not armed: return — no config load, no sweep.
4. Armed and redirected: sweep with `logger.DefaultRetentionBounds()` (equal to the template defaults, pinned by card 2's test) without loading `logger.yaml`.
5. Armed and not redirected: `Load(state.AnchorPath)`.
   On error, emit one `logger.Warn` record carrying the `configengine.ConfigFile(state.AnchorPath, moduleName)` path and the error, then sweep with `logger.DefaultRetentionBounds()`.
   On success, sweep with the loaded bounds.

The sweep is `logger.Sweep(state.Dir, bounds)`, called outside `sinkMu`, with its error ignored.
Split the bounds choice (steps 4–5) into an unexported helper taking a `logger.SinkArmState`, so tests reach it without arming a cwd-anchored sink.
`NotifyExitAndSweep` never changes the exit code; it returns nothing.

In `cmd/lyx/main.go`, replace `logger.NotifyExit(code)` in both `main` and `run` with `loggerconfig.NotifyExitAndSweep(code)`, and reword the adjoining comments to say the exit hook force-arms on non-zero and sweeps an armed sink's directory.

`exit_test.go` (untagged, `t.TempDir()` fixtures, `logger.SetDurableSinkDir` redirects with `t.Cleanup` resets, `logger.SetOutput` to capture the Warn half):

- Code 0 with no prior Info+ record: a dead-pid trace older than 14 days in the redirected directory survives, and no new trace file appears.
- Code 1 with no prior record: the sink arms, the aged dead-pid trace is deleted, and this process's own trace file survives.
- Code 0 after one `logger.Info` record: the aged trace is deleted.
- The bounds helper, given a redirected state whose `AnchorPath` holds an invalid `logger.yaml`, returns `logger.DefaultRetentionBounds()` and writes no Warn.
- Given a non-redirected state whose anchor holds a valid `logger.yaml`, it returns those bounds and writes no Warn.
- Given a non-redirected state whose anchor holds an invalid `logger.yaml`, it returns `logger.DefaultRetentionBounds()` and writes exactly one Warn record containing the config file's path.
- Given a non-redirected state whose anchor has no `_lyx/`, it returns `logger.DefaultRetentionBounds()` and writes no Warn.

In `sink_test.go`, add a test that arming the sink (one `Info` record into a redirected directory) leaves a pre-seeded, aged, dead-pid trace file in place.

In `main_integration_test.go`, add end-to-end tests against the built binary, reusing `buildLyxBinary` and the `git init` + `_lyx/` fixture of `TestRootHookWritesTraceFileOnNonZeroExit`, with `.lyx/logs` pre-seeded with dead-pid traces whose mtimes are set by `os.Chtimes`:

- With `_lyx/config/logger.yaml` setting `trace_retention_count: 2`, `lyx bogus-subcommand` exits non-zero and leaves exactly its own trace plus the two seeded traces with the newest mtimes.
- With `trace_retention_count: 0`, the same command exits with the same code as without any `logger.yaml`, its output carries one Warn line naming `logger.yaml`, and every seeded trace survives under the 200-trace default.
- With an invalid `logger.yaml`, a zero-exit invocation that emits no Info+ record (`lyx --help`) writes no trace file, prints no Warn, and leaves every seeded trace in place.

In `logger.go`'s "The durable sink and its retention" section, replace "run once per sink open" with the exit-time rule: a process sweeps its sink directory once at exit, only when its sink armed in the current generation, through `internal/loggerconfig`'s exit hook; bounds come from `_lyx/config/logger.yaml`, with the compiled-in defaults for a redirected sink or an unusable file.
In `docs/shared-libs/README.md`, reword `internal/logger`'s "retained by age and count" to "swept at exit by trace group and last activity, bounds from `logger.yaml`", and extend `internal/loggerconfig`'s line to name `NotifyExitAndSweep` as the CLI's exit hook.

**ImpactSummary:** Every `lyx` invocation whose sink armed now sweeps once at exit instead of at arm; arming alone never deletes a file, and invocations that never arm touch neither the config nor the logs directory.

**Commit:** 5: logger: sweep trace files at exit with logger.yaml bounds
