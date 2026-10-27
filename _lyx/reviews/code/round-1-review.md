---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/dobbyengine/start.go:20
    summary: Three doc comments in start.go still describe a marker directory that this change removed
    fix: "In internal/dobbyengine/start.go reword three comments. Line 20, the `LogFileName` doc: say the dobby log file sits in the worktree's `.lym/beam/` directory. Line 242, the `Live` doc: say a worktree whose `.lym/beam/` directory does not exist yet has never run a dobby. Line 258, the `SpawnDetached` doc: say the child's output is appended to the file at `geom.LogPath`."
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/hubgeom/dobbygeom.go:21
    summary: The DobbyGeometry doc says lock, state and marker paths, but the geometry now carries lock, state, log and tidy paths
    fix: "In the `DobbyGeometry` doc comment in internal/hubgeom/dobbygeom.go, replace `The lock, state and marker paths lie under that worktree's `.lym/beam/` directory` with wording that names the lock, state and log paths and the tidy directory."
    origin: lens:code-affected-set
  - id: F3
    severity: LOW
    class: design
    location: internal/dobbyengine/tidy_test.go:129
    summary: No test pins the new tidy report name, a bare UTC timestamp under TidyDir, so a revert to the board-tidy- prefix or a path outside TidyDir would pass
    fix: "In the existing spawn row of the tidy-step test in internal/dobbyengine/tidy_test.go (the row that asserts the state's Tidy strand and deadline at line 129), add an assertion that `h.state().Tidy.ReportPath` equals `filepath.Join(h.d.geom.TidyDir, h.now.UTC().Format(\"20060102T150405Z\")+\".md\")`. Add no new test."
    origin: lens:code-tests
---

### [LOW] Stale marker-directory wording in start.go

**Location:** internal/dobbyengine/start.go:20, internal/dobbyengine/start.go:242, internal/dobbyengine/start.go:258
**Issue:** The change removes dobby's `markers` directory and puts `dobby.log` directly in `.lym/beam/`, but three doc comments still say "marker directory".
`Live` stats `filepath.Dir(geom.LockPath)`, which is `.lym/beam/`, and `SpawnDetached` opens `geom.LogPath`.
The name rewrite could not catch these, since the word `marker` is not part of the old spelling.

### [LOW] Stale marker wording in DobbyGeometry doc

**Location:** internal/hubgeom/dobbygeom.go:21
**Issue:** The doc names "lock, state and marker paths", but `Geometry` now has `LockPath`, `StatePath`, `LogPath` and `TidyDir`, and no marker path.
The marker paths belong to `PawlPaths`, not this function.

### [LOW] Tidy report name has no pinning assertion

**Location:** internal/dobbyengine/tidy_test.go:129
**Issue:** The change renames the report from `board-tidy-<ts>.md` in the marker directory to `<ts>.md` in `TidyDir`, which the board entry asks for.
The spawn row at line 129 says it checks the report, but asserts only the strand and the deadline.
`tidy_test.go:233` and `:237` assert a hard-coded path handed to `TidyPrompt`, not the path the daemon builds, and `internal/hubgeom/dobbygeom_test.go:94` pins only the parent directory of `LogPath` and `TidyDir`.
This is a behavior the diff changes with no test that would fail if it were broken.

## Rejected

- Lens code-tests, `LogPath` and `TidyDir` base names (`dobby.log`, `tidy`) unpinned in `internal/hubgeom/dobbygeom_test.go:94`: the lens itself rated it below threshold, and the names are constants in the production code.
- Lens code-tests, `SpawnDetached` log path unreached by any test: `SpawnDetached` returns `ErrSpawnUnderTest` in a test binary, and the logic changed only from a join to a field read, so it is a pre-existing limitation the diff does not widen.

## Handler checks, no finding

- `git grep -il lyx` outside `go.sum` and `docs/research/` returns only `tools/deploy/main_test.go`, whose `LyX document processor` line is the editor guard and keeps its original spelling; `git ls-files` holds no path with `lyx`.
- `Geometry.MarkerDir` and `MarkerDirName` are gone; `LogPath` is `.lym/beam/dobby.log`, `TidyDir` is `.lym/beam/tidy/`, and the report is named by its UTC timestamp alone (`internal/dobbyengine/tidy.go`, `internal/hubgeom/dobbygeom.go`).
- The reflow commit `a2ae85464` changes line breaks only: the comment words of every `.go` file it touches are identical before and after.
- The diff adds no migration code and does not touch `go.sum` or `docs/research/`; `cmd/lym` exists and `cmd/lyx` does not.
- No commit carries a `Pattern-Departure:` trailer, and the three commit messages match their diffs.
