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
- `cmd/lyx/exitsweep.go`
- `cmd/lyx/exitsweep_test.go`

**Uses:**
- `plan:internal/logger#SinkArmState`
- `plan:internal/logger#CurrentSinkArmState`
- `internal/logger#DefaultRetentionBounds`
- `internal/loggerconfig#Load`
- `internal/loggerconfig#ConfigPath`
- `internal/logger#Sweep`
- `internal/logger#NotifyExit`
- `internal/logger#Warn`

**Intent:**
Move the sweep from sink arm to process exit and feed it the configured bounds (decision record D3, D5).
The exit hook lives in `cmd/lyx`, the CLI-layer root D4 names, beside the root's existing pre-run helper `cmd/lyx/stencilseed.go`.
D4's other named site, a cliwire helper, is ruled out: `internal/cliwire`'s fixed production dependency set excludes `internal/configengine`, which `loggerconfig.Load` pulls in.

In `sink.go`'s `armDurableSinkLocked`, remove the `Sweep` call; arming no longer sweeps.
In `isLyxWorktree`'s doc comment, reword "cmd/lyx's logger.NotifyExit(code) force-arms the sink" to "cmd/lyx's exit hook, through logger.NotifyExit(code), force-arms the sink", since `main` no longer calls `logger.NotifyExit` directly.

In `cmd/lyx/exitsweep.go`, add `notifyExitAndSweep(code int)`, which runs the exit path in D3's order:

1. `logger.NotifyExit(code)`, which force-arms the sink on a non-zero code, unchanged.
2. `logger.CurrentSinkArmState()`.
3. Not armed: return — no config load, no sweep.
4. Armed and redirected: sweep with `logger.DefaultRetentionBounds()` (equal to the template defaults, pinned by card 2's test) without loading `logger.yaml`.
5. Armed and not redirected: `loggerconfig.Load(state.AnchorPath)`.
   On error, emit one `logger.Warn` record carrying `loggerconfig.ConfigPath(state.AnchorPath)` and the error, then sweep with `logger.DefaultRetentionBounds()`.
   On success, sweep with the loaded bounds.

The sweep is `logger.Sweep(state.Dir, bounds)`, called outside `sinkMu`, with its error ignored.
Split the bounds choice (steps 4–5) into an unexported helper, `exitSweepBounds(state logger.SinkArmState) logger.RetentionBounds`, so tests reach it without arming a cwd-anchored sink.
`notifyExitAndSweep` never changes the exit code, and returns nothing.
Give the file a header comment stating the D3 order and why the config load waits for the arm check (a degrading load logs at Info, and an Info record arms the sink).

In `cmd/lyx/main.go`, replace `logger.NotifyExit(code)` in both `main` and `run` with `notifyExitAndSweep(code)`, and reword the adjoining comments to say the exit hook force-arms on non-zero and sweeps an armed sink's directory.

`exitsweep_test.go` (untagged, `t.TempDir()` fixtures only).
Each case redirects the sink with `logger.SetDurableSinkDir(t.TempDir())` and registers a `t.Cleanup` that resets it with `logger.SetDurableSinkDir("")`;
cases that check the Warn half capture it with `logger.SetOutput` and restore it in the same cleanup.
An invalid `logger.yaml` here and in the integration tests below is a valid file carrying both keys with `trace_retention_count: 0`, so the case exercises value validation rather than `configengine`'s missing-key check.

- Code 0 with no prior Info+ record: a dead-pid trace older than 14 days in the redirected directory survives, and no new trace file appears.
- Code 1 with no prior record: the sink arms, the aged dead-pid trace is deleted, and this process's own trace file survives.
- Code 0 after one `logger.Info` record: the aged trace is deleted.
- `exitSweepBounds`, given a redirected state whose `AnchorPath` holds an invalid `logger.yaml`, returns `logger.DefaultRetentionBounds()` and writes no Warn.
- Given a non-redirected state whose anchor holds a valid `logger.yaml` carrying both keys at non-default values, it returns those bounds and writes no Warn.
- Given a non-redirected state whose anchor holds an invalid `logger.yaml`, it returns `logger.DefaultRetentionBounds()` and writes exactly one Warn record containing `loggerconfig.ConfigPath(anchor)`.
- Given a non-redirected state whose anchor has no `_lyx/`, it returns `logger.DefaultRetentionBounds()` and writes no Warn.

In `sink_test.go`, add a test that arming the sink (one `Info` record into a redirected directory) leaves a pre-seeded, aged, dead-pid trace file in place.

In `main_integration_test.go`, add end-to-end tests against the built binary, reusing `buildLyxBinary` and the `git init` + `_lyx/` fixture of `TestRootHookWritesTraceFileOnNonZeroExit`, with `.lyx/logs` pre-seeded with dead-pid traces whose mtimes are set by `os.Chtimes` to within the last 14 days:

Every `logger.yaml` fixture carries both keys, because `configengine`'s missing-key check rejects a file lacking either and the exit hook would then fall back to the defaults.

- With `_lyx/config/logger.yaml` setting `trace_retention_count: 2` and `trace_retention_days: 14`, `lyx bogus-subcommand` exits non-zero and leaves exactly its own trace plus the two seeded traces with the newest mtimes.
- With `trace_retention_count: 0` and `trace_retention_days: 14`, the same command exits with the same code as without any `logger.yaml`, its output carries one Warn line naming `logger.yaml` and `trace_retention_count`, and every seeded trace survives under the 200-trace default.
- A zero-exit invocation that emits no Info+ record (`lyx --help`), run twice — once with no `logger.yaml` under `_lyx/config/` and once with an invalid one — writes no trace file, prints no Warn, and leaves every seeded trace in place in both runs.
  The absent-file run is the one a stray config load would betray: `configengine.LoadOrTemplate` logs at Info when it degrades to the template, and that record would arm the sink.

In `logger.go`'s "The durable sink and its retention" section, replace "run once per sink open" with the exit-time rule: a process sweeps its sink directory once at exit, only when its sink armed in the current generation, through `cmd/lyx`'s exit hook; bounds come from `_lyx/config/logger.yaml`, with the compiled-in defaults for a redirected sink or an unusable file.
In `docs/shared-libs/README.md`, reword `internal/logger`'s "retained by age and count" to "swept at exit by trace group and last activity, bounds from `logger.yaml`", and extend `internal/loggerconfig`'s line to say `cmd/lyx`'s exit hook loads it only after the sink has armed.

**ImpactSummary:** Every `lyx` invocation whose sink armed now sweeps once at exit instead of at arm; arming alone never deletes a file, and invocations that never arm touch neither the config nor the logs directory.

**Commit:** 5: logger: sweep trace files at exit with logger.yaml bounds
