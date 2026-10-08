# Card 8 — Burler stops a half through the run handle

**Edit:**
- `internal/burlerengine#Handle`
- `internal/burlerengine#Engine`
- `internal/burlerengine#New` -> `func New(shuttle Shuttle, geom Geometry, cfg Config, stencilsDir, frictionDir string) *Engine`
- `internal/burlerengine#Engine.stopHalf`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/loomcli#loomCLI.wire`
- `internal/shedadapters#BurlerDeps`
- `internal/shedadapters#BurlerProducer`
- `internal/shedadapters#NewBurlerProducer`
- `internal/shedadapters#BurlerProducer.stopLiveHalf`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#burlerRoundEntry`
- `internal/testkit/envkit#FullEnv`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/burlercli/wiring_test.go`
- `internal/shedadapters/attachprobe_test.go`
- `internal/shedadapters/burler_test.go`
- `internal/shedadapters/bouncer_fixture_test.go`
- `internal/shedrecipe/fixture_test.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `internal/battencli/wire_seams_test.go`
- `internal/burlerengine/doc.go`
- `internal/shedadapters/doc.go`

**Uses:**
- `plan:internal/shuttleengine#Run.Stop`

**Intent:**
Move burler's half stop onto shuttle's stop verb (D7), so a half lyx ends reads `stopped` and the next attach probe never halts on it (#444).

`burlerengine.Handle` gains `Stop() error`, which `*shuttleengine.Run` implements; its doc comment says the engine stops a half through it.
`Engine.stopHalf` calls `handle.Stop()` in place of the guid remover, keeping its `ErrHalfNotStopped` wrapping and way-forward text, so a half stopped during `join` while its own `Wait` runs is marked and reads `stopped` after that `Wait` returns.
`Engine` drops its remover field and `New` drops its `remover` parameter; every caller drops the argument: `burlercli`'s two wirings, `loomcli`'s `wire` (whose `burlerRemover` local goes), and the three test files that build an engine.

`shedadapters`: `BurlerDeps.Remover`, the producer's remover field and `NewBurlerProducer`'s nil-remover refusal go; `stopLiveHalf` stops the live half with `half.Stop()`, keeping its error and way forward.
`shedrecipe`: `Env.BurlerRemover`, `burlerRoundEntry`'s `requireSeam` check on it and its `Remover:` argument go; `envkit.FullEnv` and `loomcli.wire` stop filling the field.

Test fakes follow: `engine_test.go`'s `fakeRemover` goes, and its `fakeHandle` gains `Stop`, which records the call and ends that handle's `Wait` as died, the behaviour the remover gave it; `burlercli`'s `diedHandle` and `shedadapters`' `probedHandle` gain a recording `Stop`.
Assertions on removed guids become assertions on stopped handles; the nil-remover rows (`burler_test.go`'s `NilRemover`, `entries_burler_test.go`'s `NilBurlerRemoverSeamIsRefused`) and `loomcli`'s "env burler remover is wired" subtest go, and `battencli`'s seam-exclusion table drops its `BurlerRemover` row.
`engine_test.go`'s round table asserts that a round whose second half's start fails `ErrNotStarted` ends as that half's `NotStarted` died outcome with the first half stopped; `burler_test.go` keeps (or gains, if no row pins it) the row where the adapter's attempt-2 failure carries the agent-start transient class.
`burlerengine/doc.go` and `shedadapters/doc.go` say a half is stopped through its handle's `Stop`, which records the stop.

`burlerengine.StrandRemover`, `NewReedStrandRemover` and `shedfake.StrandRemover` are left unused here and deleted by card 10.

**ImpactSummary:** `burlerengine.New` is re-signed with all six callers (three production wirings, three test files) on this card; the remover seam leaves `shedadapters`, `shedrecipe.Env`, `envkit` and `loomcli`'s wiring, and burler's stop behaviour changes only by recording the stop.

**Commit:** 8: burler: stop a round's half through its run handle, recording the stop
