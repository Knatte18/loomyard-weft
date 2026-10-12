# Card 4 — pause-geometry

**Create:**
- `plan:internal/hubgeom#PauseGeometry` -> `func PauseGeometry(l *lyxcwd.Location) (pauserecord.Geometry, error)`
- `plan:internal/hubgeom#pauseGeometry` -> `func pauseGeometry(l *lyxcwd.Location, prime bool) pauserecord.Geometry`
- `plan:internal/standalonegeom#PauseGeometry` -> `func PauseGeometry(stateDir string) pauserecord.Geometry`

**Edit:**
- `internal/hubgeom#TestReedGeometry`
- `internal/standalonegeom#TestStandaloneDirs`
- `internal/hubgeom/doc.go`
- `internal/standalonegeom/doc.go`

**Intent:**
Put the pause record's path into the hub and standalone geometries (D2, `PATTERN-told-geometry`), so every reader is handed it and no bound engine derives it.

- `hubgeom.PauseGeometry` resolves whether `l` is the prime the way `ReedGeometry` does (`isPrimeWorktree`) and returns `pauseGeometry(l, prime)`.
- `pauseGeometry` returns `RecordPath` = `filepath.Join(fabricengine.HubScratchDir(l.HubPath), lyxdirs.PauseDirName, pauserecord.RecordFileName)`, the board-level `.lyx` every worktree of the hub reaches by path, and `Slug` = `l.WorktreeName` for a task worktree and empty for the prime.
- `standalonegeom.PauseGeometry(stateDir)` returns `RecordPath` = `filepath.Join(stateDir, lyxdirs.DotLyxDirName, lyxdirs.PauseDirName, pauserecord.RecordFileName)` and an empty slug: a standalone run belongs to no hub pair, so only a hub-scope entry in that record would cover it.

Tests: `TestReedGeometry`'s rows already cover a task worktree and the prime; each also asserts `pauseGeometry(l, tt.isPrime)`'s record path under the hub's board `.lyx` and its slug equal to the row's `wantSlug`.
`TestStandaloneDirs` gains a row for `PauseGeometry(stateDir).RecordPath` beside the three other state paths, which its distinctness check then also covers, and asserts the empty slug.
Both packages' `doc.go` name the new constructor beside the others.

**ImpactSummary:** Two new geometry constructors and two extended geometry tests; nothing reads them until card 7.
