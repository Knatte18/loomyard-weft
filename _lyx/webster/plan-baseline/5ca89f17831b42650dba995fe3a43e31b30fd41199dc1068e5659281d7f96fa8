# Card 14 — reed-resume-hold

**Edit:**
- `internal/reedengine#Geometry`
- `internal/reedengine#ResumeResult`
- `internal/reedengine#Engine.Resume`
- `internal/reedengine#planResumeLaunches` -> `func planResumeLaunches(strands []Strand, liveIDs map[string]bool, exists func(path string) bool, paused bool) (launch, drop, held []Strand)`
- `internal/reedengine#TestPlanResumeLaunches_ThreeLifecycleStates`
- `internal/hubgeom#reedGeometry`
- `internal/hubgeom#TestReedGeometry`
- `internal/reedcli#reedCLI.resumeCmd`
- `internal/battencli#battenCLI.wire`
- `internal/reedengine/doc.go`

**Intent:**
Make `lyx reed resume`, and batten's revive of a dead driver, which goes through the same `Engine.Resume`, leave a covered pair's strands down and report them (D4), so a reboot under a live entry does not relaunch frozen agents.

- `Geometry` gains `PauseRecordPath`; `hubgeom.reedGeometry` fills it from `pauseGeometry` (card 4), and standalone geometry leaves it empty.
- `Engine.Resume` reads `pauserecord.Read(PauseRecordPath).Covering(NameSlug)` once, inside the boot op lock, for a geometry with both fields set; the prime (empty `NameSlug`) is never held, so the orch strand relaunches as today.
  An unreadable record holds nothing and logs one Warn.
- `planResumeLaunches` takes `paused`: when true, every strand it would relaunch goes to `held` instead, while finished strands are still dropped and hidden strands still skipped.
- `ResumeResult` gains `Held []string`, the held strands' names; reed's server revive relaunches no strand and needs no hold.
- `resumeCmd`'s envelope gains `held`, and its help says a covered pair's strands are left down until `lyx resume`.
- Batten's `ReviveStrands` closure in `battenCLI.wire` logs the held names beside resumed and dropped, so its revive reads as skipped for a covered pair.

After the resume the strands are replayed by the next `lyx reed resume` or batten's driver-dead path, as today.
Bound: a held strand is left exactly as a dead one is; nothing is killed or removed.

Tests: `TestPlanResumeLaunches_ThreeLifecycleStates` gains a `paused` field and a `wantHeld` field, its existing rows unpaused, and two rows: a paused plan over a not-live strand, a live one and a hidden one holds the not-live strand and launches nothing; a paused plan over a finished strand still drops it.
`TestReedGeometry` asserts `PauseRecordPath` equals the pause geometry's record path.
`doc.go`'s resume paragraph names the hold.

**ImpactSummary:** planResumeLaunches re-signed with its one caller, Engine.Resume, on this card; the reed resume envelope and result gain a held list, and the prime's resume is unchanged.
