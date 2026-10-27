```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the code the diff changes, from the `code-correctness` lens.

**Per function or branch (case run, verdict)**

- `Geometry` fields and `TidyDirName` in `internal/dobbyengine/geometry.go`: `MarkerDir` and `MarkerDirName` are gone, and no reference to them remains (grep clean). Correct.
- `DobbyGeometry` in `internal/hubgeom/dobbygeom.go`, prime and beam case: `beamDir` is `<root>/<anchorRel>/.lym/beam`.
  - `LogPath` comes out as `.lym/beam/dobby.log` and `TidyDir` as `.lym/beam/tidy`, which match the board entry.
  - The test at `internal/hubgeom/dobbygeom_test.go:94` checks both fields.
  - Correct.
- `SpawnDetached` in `internal/dobbyengine/start.go`, empty and fresh hub: `MkdirAll(Dir(LogPath))` creates `.lym/beam/`, then the log opens in append mode.
  - Failures of `MkdirAll` and `OpenFile` are returned unchanged. Correct.
  - Behavior change: it no longer creates a `markers/` directory. `Live` stats `Dir(LockPath)`, which is `.lym/beam`, and that path is unchanged. Correct.
- `spawnTidy` in `internal/dobbyengine/tidy.go`:
  - `MkdirAll(TidyDir)` runs first, then the path is `tidy/<UTC timestamp>.md`.
  - A `NearestOrch`, `MkdirAll`, `TidyPrompt` or `Start` failure returns before the state record is written.
  - An `UpdateState` failure stops the strand, so no unrecorded agent is left running.
  - Two spawns in the same second would share a report name. That is not reachable, because only one tidy agent runs at a time.
  - Correct.
- `settleTidy` in `tidy.go`: it reads the report through the stored `tidy.ReportPath`.
  - A state record written before the change would hold an old `markers/board-tidy-…md` path. It still resolves, and no migration is in scope. Correct.
- `LoadConfig` error text in `internal/dobbyengine/config.go`: only the command spelling changed. Correct.
- Mechanical case-mapping hazards, sampled in the dobby and hubgeom diffs: renamed identifiers (`LymExecutable`, `lymdirs.DotLymDirName`) and string literals are consistent. I saw no change of ordering, width or meaning. The full-tree check is not done.

**Findings**

1. LOW, `consistency`, `internal/dobbyengine/start.go:20`: the doc comment of `LogFileName` says "dobby log file in the marker directory". The log now sits in the worktree's `.lym/beam/` directory.
   - Fix: say "in the worktree's `.lym/beam/` directory".
2. LOW, `consistency`, `internal/dobbyengine/start.go:242`: the `Live` doc says "A worktree whose marker directory does not exist yet has never run a dobby". The code stats `filepath.Dir(geom.LockPath)`, the `.lym/beam/` directory.
   - Fix: say "whose `.lym/beam/` directory does not exist yet".
3. LOW, `consistency`, `internal/dobbyengine/start.go:258`: the `SpawnDetached` doc says "appended to the marker directory's log". The code appends to `geom.LogPath`.
   - Fix: say "appended to the file at `geom.LogPath`".
4. LOW, `consistency`, `internal/hubgeom/dobbygeom.go:19`: the `DobbyGeometry` doc says "The lock, state and marker paths lie under …". The fields are now lock, state, log and tidy.
   - Fix: say "The lock, state, log and tidy paths".

No correctness defect in the code paths: no wrong result, swallowed error or inconsistent state on a failure path.

**Tool calls**
1. Read `internal/dobbyengine/tidy.go` lines 155–200, for the `spawnTidy` and `settleTidy` bodies, which the manifest showed only as diff lines.
