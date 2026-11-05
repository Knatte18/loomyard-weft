# Card 25 — pawl-raddle-handoff

**Create:**
- `internal/loomcli#raddleHandOff`
- `internal/loomcli#TestRaddleHandOff`
- `internal/pawlengine#HandOffMarkerName`

**Edit:**
- `internal/hubgeom#PawlPaths`
- `internal/hubgeom#TestDobbyGeometry`
- `internal/pawlengine#Deps`
- `internal/pawlengine#Paths`
- `internal/pawlengine#Deps.pathsSeams`
- `internal/pawlengine#BeamUp.Call`
- `internal/pawlengine#awaitRun.notify`
- `internal/shedrecipe#validPawlDeps`
- `internal/testkit/envkit#FullEnv`
- `internal/pawlengine_test#world`
- `internal/pawlengine_test#world.deps`
- `internal/pawlengine_test#TestBeamUp`
- `internal/pawlengine_test#TestBeamUp_RerunIsIdempotent`
- `internal/pawlengine_test#TestNewProducers_RefuseAnUnsetSeam`
- `internal/shedrecipe#beamUpEntry`
- `internal/shedrecipe#requirePawl`
- `internal/shedrecipe#zeroEnvField`
- `internal/shedrecipe#simpleEntryCases`
- `internal/loomcli#pawlDeps`
- `internal/pawlengine/doc.go`

**Uses:**
- `internal/boardengine#RaddleStore.Holds`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/shedrecipe/doc.go`
- `internal/loomcli/doc.go`
- `internal/hubgeom/doc.go`

**Intent:** D11: a beam's orch receives the Sketch and the proposal at its first start.
`pawlengine.Paths` gains `HandOffMarker`, a file beside the land request that BeamUp never removes; `pathsSeams` names it.
`HandOffMarkerName` is the marker's file name, declared beside `AwaitMarkerName` in `internal/pawlengine/landrequest.go`.
`hubgeom.PawlPaths` sets `HandOffMarker` to `HandOffMarkerName` under the beam's `.lym/beam/` directory beside the land request, and its doc comment names the hand-off marker among the paths it tells, with no count of them.
`pawlengine/doc.go`'s sentence listing the files under the beam worktree's `.lym/beam/` (the land request, the reopen request, the dobby-stop marker and the await marker) gains the hand-off marker, and its Beam-Up description gains the hand-off step.
`TestDobbyGeometry`'s assertion that every pawl path lies directly under the dobby's directory gains `HandOffMarker`.
`pawlengine.Deps` gains `HandOff func() (line string, found bool, err error)`.
`Deps.Notify` becomes `func(line string) (posted bool, err error)`, reporting whether the post reached an orch: `pawlDeps` returns `NotifyOrchAt`'s posted result instead of dropping it, BeamAwait's `awaitRun.notify` ignores it and keeps its behaviour, and the test fakes in `world.deps`, `validPawlDeps` and `envkit.FullEnv` report posted.
`BeamUp.Call`, after its injected steps have brought the beam's dobby and orch up, and only while the marker is absent, calls `HandOff`; when found, it mails the line through `Deps.Notify` (which mails the beam's own orch, `orchcli.NotifyOrchAt` of the beam's worktree) and then writes the marker; a beam with no proposal writes the marker with no mail.
An interruption between the mail and the marker lets a re-invoked BeamUp mail a second identical hand-off, which D11 accepts; a BeamUp re-invoked or run later with the marker present sends none.
A `HandOff` or marker-write failure, and a hand-off post that reached no orch (posted false, such as when the beam's orch state records no strand), is `Stuck` with `Halt` with no marker written, as every BeamUp step's failure is, so a resume re-runs it.
`raddleHandOff` reads the raddle store of the hub and, when a stored proposal filed the beam's project, returns the line naming the Sketch slug, `lym board get <project> --record` and `lym board raddle <project>`.
`beamUpEntry` requires `HandOff`, `Notify` and `Paths.HandOffMarker` beside the fields it requires today, and `requirePawl`'s field map gains `HandOff` and `Paths.HandOffMarker`; `zeroEnvField` gains both, and `simpleEntryCases`' BeamUp row validates the three and takes `Pawl.Down` as its unread field in place of `Pawl.Notify`.
The test fixtures that build `pawlengine.Paths` by hand, `validPawlDeps` and `envkit.FullEnv`, set `HandOffMarker` beside `AwaitMarker` and fill `HandOff`.
`pawlDeps` fills the `HandOff` seam, and the path reaches it through the `Paths: hubgeom.PawlPaths(location)` it already takes, which it fills nothing beside, so `TestWire_BeamRecipe`'s existing comparison of `c.env.Pawl.Paths` against `hubgeom.PawlPaths` pins the value.
`TestBeamUp` gains rows: a stored proposal mails the hand-off after the orch step and writes the marker; no proposal sends none and writes the marker; a `HandOff` error halts; a `Notify` failure after a found line halts with no marker written; a `Notify` that posts nothing (posted false, nil error) after a found line halts with no marker written.
`TestBeamUp_RerunIsIdempotent` gains a rerun with the marker present (no second mail) and a rerun after the test removes the marker the first call wrote, standing for an interruption between the mail and the marker (a second identical mail).
`TestNewProducers_RefuseAnUnsetSeam` gains the unset `HandOff` row.
`world` and `world.deps` gain the hand-off seam, its scripted line and error, and the marker path.
`TestRaddleHandOff` (tier0, a temp store) has rows: a store whose proposal filed the project (line naming the three), a store with none (not found).

**ImpactSummary:** One new BeamUp step after the orch comes up, guarded by its own marker; the pawl wiring and its registry entry move with the new seam, `hubgeom.PawlPaths` tells the new marker path, and `TestWire_EverySeamFilled` checks it is filled.
