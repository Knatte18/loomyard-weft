# Card 15 — loom-arms-loop

**Create:**
- `plan:internal/loomcli#loopSpecFor` -> `func loopSpecFor(loc *lyxcwd.Location, runID string, idle time.Duration, activity func() (time.Time, bool, error)) shedverbs.LoopSpec`
- `plan:internal/loomcli#agentActivityFor` -> `func agentActivityFor(loc *lyxcwd.Location, cfg shuttleengine.Config) func() (time.Time, bool, error)`
- `plan:internal/loomcli#loomCLI.loomAfterInterrupt` -> `func (c *loomCLI) loomAfterInterrupt(ctx context.Context, producer, cause, traceFile string) string`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli/cli_test.go`

**Uses:**
- `plan:internal/shedverbs#LoopSpec`
- `internal/shedrun#LoopLock`
- `internal/shedrun#LoopPIDFile`
- `internal/shedrun#LoopLog`
- `internal/shedrun#LoopEnvelope`
- `internal/shedrun#LoopDelivered`
- `internal/shedrun#LoopJobName`
- `internal/shedrun#StepTraceCopy`
- `internal/shedrun#StepStderr`
- `plan:internal/logger#TraceFilesFor`
- `internal/logger#TraceDir`
- `internal/shuttleengine#ReadAgentActivity`
- `internal/shuttleengine#ReadWaitMarkers`
- `internal/shuttleengine/claudeengine#NewFromConfig`
- `internal/verifytree#ReadMarker`
- `internal/loomcli#loomCLI.reflectHalt`

**Intent:** Loom arms the loop: its files, its trace listing, its watchdog window, its activity reading and its interrupted-stop friction, so `lyx shed step <run-id> --until-stop` and `lyx loom step --until-stop` run detached under loom's recipe.
`loomCLI` gains `shuttleCfg shuttleengine.Config` and `stepIdleTimeout time.Duration`, which `wire` fills from the shuttle config it already loads and from `loomCfg.StepIdleTimeoutMin` in minutes; the lightweight wiring leaves both zero, since `status` and `pause` never run the loop.
`specFor` fills `spec.Loop` through `loopSpecFor` when `c.location` is non-nil: the loop paths and the job name from `internal/shedrun`'s constructors over the resolved run-id, `Delivered` as `LoopDelivered` over that run-id and the loop id it is given, `StopFiles` from `StepTraceCopy` and `StepStderr`, `TraceFiles` as `logger.TraceFilesFor` over `logger.TraceDir()`, the loop process's own sink directory, which its children share because they run the same binary from the same cwd, the executable from `os.Executable`, `IdleTimeout` from the receiver and `Activity` from `agentActivityFor`; it also fills `Hooks.AfterInterrupt` with `c.loomAfterInterrupt`.
`agentActivityFor` returns the reading batten's quiet notice already trusts, taken from the same sources without importing `battenshed`: the newest `LastActivity` over every live agent run `shuttleengine.ReadAgentActivity` reports for the worktree through a `claudeengine.NewFromConfig` engine, which lists every strand and so counts a MultiLLM row's chair and each advisor; a running verify marker (`verifytree.ReadMarker`) or a shuttle wait marker that is not a held turn end (`shuttleengine.ReadWaitMarkers`) reads as activity now; the boolean is false when nothing is live.
`loomAfterInterrupt` writes the halt friction note a `failed` halt writes, through `reflectHalt` with state `failed`, the cause as the reason and the child's trace file, and returns the friction status, so an interrupted stop under loom leaves a note the next reflection covers (`PATTERN-friction-capture`); with Tier 2 off it reports skipped.
`cli_test.go`'s `TestSpecFor_ScratchAndFrictionDir` gains subtests through the spec `specFor` returns: the loop paths, job name, stop files, trace listing and the interrupt hook are filled for a located receiver and left zero for a location-less one; calling the filled `Hooks.AfterInterrupt` writes a halt note naming the producer, state `failed`, the cause and the trace file; and with no friction directory it reports skipped.

**ImpactSummary:** `specFor` fills fields the generic verbs ignore outside `--until-stop`; `wire` stores two values it already computed; no verb's envelope changes.
