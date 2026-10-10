# Card 5 — reed-wake-cadence

**Create:**
- `plan:internal/reedengine#NextWakeCadence` -> `func NextWakeCadence(current, base, ceiling time.Duration, changed bool) time.Duration`
- `plan:internal/reedengine#DiscoverSignalFileName` -> `const DiscoverSignalFileName = "reed-discover.signal"`
- `plan:internal/reedengine#nameRepairCeiling` -> `const nameRepairCeiling = 60 * time.Second`
- `plan:internal/reedengine#nameRepairTiming` -> `type nameRepairTiming struct`
- `plan:internal/reedengine#nameRepairDefaultTiming` -> `func nameRepairDefaultTiming() nameRepairTiming`
- `plan:internal/reedengine#Engine.watchNames` -> `func (e *Engine) watchNames(ctx context.Context, namer SessionNamer, t nameRepairTiming) error`
- `plan:internal/reedengine#Engine.touchDiscoverSignal` -> `func (e *Engine) touchDiscoverSignal()`

**Edit:**
- `internal/reedengine#Geometry`
- `internal/reedengine#nameRepairCycle`
- `internal/reedengine#Engine.WatchNames`
- `internal/reedengine#Engine.repairNames` -> `func (e *Engine) repairNames(namer SessionNamer) (repaired bool, err error)`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine/namerepair_test.go`
- `internal/reedengine/naming_integration_test.go`
- `internal/reedengine/watchdog_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `internal/reedengine/doc.go`
- `internal/hubgeom#reedGeometry`
- `internal/hubgeom/hubgeom_test.go`
- `internal/hubgeom/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/reedengine#watchdogSignalTick`

**Intent:** The name-repair loop backs off while nothing drifts, and a cold session boot raises the hub-level discover signal (D5, the `reedengine` half).
`watchdogSignalTick`, unread since the previous card dropped `watchTiming.SignalTick`, is deleted here together with the `doc.go` sentence that names it.
`NextWakeCadence` is the one backoff rule both daemon loops share: it returns `base` when `changed` is true, else the lesser of twice `current` and `ceiling`; `reedcli`'s discovery loop calls it in the next card.
`Geometry` gains `DiscoverSignalPath string`, the hub-level discover signal file the engine is told; empty in standalone mode, where nothing is touched.
`DiscoverSignalFileName` is the file's name, declared here as `resizeSignalFileName` is, so the teller and the daemon spell it once.
`ensureServerAndSessionLocked` calls `touchDiscoverSignal` right after a fresh session exists, on the boot path alone, so every lyx path that boots a session raises the signal and an attach to a live session touches nothing; `touchDiscoverSignal` creates or truncates the told path and logs a failure at Warn without failing the boot.
`WatchNames` keeps its signature and runs `watchNames` with `nameRepairDefaultTiming()`, whose `Base` is `nameRepairCycle` and `Ceiling` `nameRepairCeiling`; `watchNames` runs a pass per wake on a timer re-armed with `NextWakeCadence`, reset to the base by a pass that repaired something, from `repairNames`'s new `repaired` answer, which is true when any title or session name was repaired.
`namerepair_test.go` adds the repaired answer to its existing pass tests and a test driving `watchNames` with a short timing against the fake tmux, asserting the pass cadence doubles to the ceiling while nothing drifts and resets after a repair; `naming_integration_test.go` takes the new return.
`watchdog_test.go` gains a table test over `NextWakeCadence` and pins `DiscoverSignalFileName`.
`ensuresession_integration_test.go` asserts a cold boot leaves the discover signal file at the told path and that an ensure against the live session afterwards touches nothing.
`hubgeom.reedGeometry` fills `DiscoverSignalPath` with `DiscoverSignalFileName` under `fabricengine.HubScratchDir(l.HubPath)`, which `TestReedGeometry` asserts, and `hubgeom/doc.go` names the field beside `LogsDir`.
`reedengine/doc.go`'s watchdog section states the discover signal, who touches it and the name-repair backoff.

**ImpactSummary:** One told geometry field filled by the one hub teller, one exported cadence helper and one exported file name; `repairNames` gains a result whose callers are `WatchNames` and two tests, the boot path gains a non-fatal touch, and the deleted constant has no reader left.
