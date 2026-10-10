# Card 5 — reed-wake-cadence

**Create:**
- `internal/reedengine#NextWakeCadence`
- `internal/reedengine#DiscoverSignalFileName`
- `internal/reedengine#nameRepairCeiling`
- `internal/reedengine#nameRepairTiming`
- `internal/reedengine#nameRepairDefaultTiming`
- `internal/reedengine#Engine.watchNames`
- `internal/reedengine#Engine.touchDiscoverSignal`

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
`nameRepairTiming` also carries `After func(time.Duration) <-chan time.Time`, the timer seam `watchNames` waits on, which `nameRepairDefaultTiming` fills with `time.After`, so no test reads the wall clock.
`namerepair_test.go` adds the repaired answer to its existing pass tests and one test of what the `NextWakeCadence` table cannot show, that `repairNames`'s answer chooses the next wait: it drives `watchNames` against the fake tmux with an `After` that records each requested wait and fires at once, and asserts the recorded waits move to the ceiling while nothing drifts and return to the base after a pass that repaired something; `naming_integration_test.go` takes the new return.
`watchdog_test.go` gains a table test over `NextWakeCadence`.
`ensuresession_integration_test.go` asserts a cold boot leaves the discover signal file at the told path and that an ensure against the live session afterwards touches nothing.
`hubgeom.reedGeometry` fills `DiscoverSignalPath` with `DiscoverSignalFileName` under `fabricengine.HubScratchDir(l.HubPath)`, which `TestReedGeometry` asserts, and `hubgeom/doc.go` names the field beside `LogsDir`.
`reedengine/doc.go`'s watchdog section states the discover signal, who touches it and the name-repair backoff.

**ImpactSummary:** One told geometry field filled by the one hub teller, one exported cadence helper and one exported file name; `repairNames` gains a result whose callers are `WatchNames` and two tests, the boot path gains a non-fatal touch, and the deleted constant has no reader left.
