# Card 25 — pawl-raddle-handoff

**Create:**
- `plan:internal/loomcli#raddleHandOff` -> `func raddleHandOff(location *lymcwd.Location, project string) (line string, found bool, err error)`
- `plan:internal/loomcli#TestRaddleHandOff` -> `func TestRaddleHandOff(t *testing.T)`

**Edit:**
- `internal/pawlengine#Deps`
- `internal/pawlengine#Paths`
- `internal/pawlengine#Deps.pathsSeams`
- `internal/pawlengine#BeamUp.Call`
- `internal/pawlengine_test#world`
- `internal/pawlengine_test#world.deps`
- `internal/pawlengine_test#TestBeamUp`
- `internal/pawlengine_test#TestBeamUp_RerunIsIdempotent`
- `internal/pawlengine_test#TestNewProducers_RefuseAnUnsetSeam`
- `internal/shedrecipe#beamUpEntry`
- `internal/loomcli#pawlDeps`

**Extends:**
- `internal/pawlengine/doc.go`

**Uses:**
- `plan:internal/boardengine#RaddleStore.Holds`
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

**Intent:** D11: a beam's orch receives the Sketch and the proposal at its first start.
`pawlengine.Paths` gains `HandOffMarker`, a file beside the land request that BeamUp never removes; `pathsSeams` names it.
`pawlengine.Deps` gains `HandOff func() (line string, found bool, err error)`.
`BeamUp.Call`, after its injected steps have brought the beam's dobby and orch up, and only while the marker is absent, calls `HandOff`; when found, it mails the line through `Deps.Notify` (which mails the beam's own orch, `orchcli.NotifyOrchAt` of the beam's worktree) and then writes the marker; a beam with no proposal writes the marker with no mail.
An interruption between the mail and the marker lets a re-invoked BeamUp mail a second identical hand-off, which D11 accepts; a BeamUp re-invoked or run later with the marker present sends none.
A `HandOff` or marker-write failure is `Stuck` with `Halt`, as every BeamUp step's is, so a resume re-runs it.
`raddleHandOff` reads the raddle store of the hub and, when a stored proposal filed the beam's project, returns the line naming the Sketch slug, `lym board get <project> --record` and `lym board raddle <project>`.
`beamUpEntry` requires the new seam and path; `pawlDeps` fills them.
`TestBeamUp` gains rows: a stored proposal mails the hand-off after the orch step and writes the marker; no proposal sends none and writes the marker; a `HandOff` error halts; a `Notify` failure after a found line halts with no marker written.
`TestBeamUp_RerunIsIdempotent` gains a rerun with the marker present (no second mail) and a rerun after the test removes the marker the first call wrote, standing for an interruption between the mail and the marker (a second identical mail).
`TestNewProducers_RefuseAnUnsetSeam` gains the unset `HandOff` row.
`world` and `world.deps` gain the hand-off seam, its scripted line and error, and the marker path.
`TestRaddleHandOff` (tier0, a temp store) has rows: a store whose proposal filed the project (line naming the three), a store with none (not found).

**ImpactSummary:** One new BeamUp step after the orch comes up, guarded by its own marker; the pawl wiring and its registry entry move with the new seam, and `TestWire_EverySeamFilled` checks it is filled.
