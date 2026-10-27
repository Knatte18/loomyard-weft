```yaml
lens: code-affected-set
outcome: reported
```

Task: lens `code-affected-set`, listing per change the set found and whether it is covered.

**Per change**

| Change | Set found | Covered |
|---|---|---|
| `Geometry.MarkerDir`/`MarkerDirName` replaced by `LogPath`/`TidyDir`/`TidyDirName` | The geometry builder `hubgeom.DobbyGeometry`, `start.go` `SpawnDetached`, `tidy.go` and its tests, `loop_move_test.go`, `hubgeom/dobbygeom_test.go` | Code and tests covered: no `MarkerDir` identifier is left. |
| Tidy report name `board-tidy-<ts>.md` → `<ts>.md` | `tidy.go`, `tidy_test.go` paths, `overview.md`, `beamcli/doc.go`, `dobbyengine/doc.go`, the board-tidy stencil | Covered: the grep finds no `board-tidy-` left, and the stencil takes `{{.report_path}}`. |
| `dobby.log` moved to `.lym/beam/dobby.log` | `LogFileName` in `start.go`, `hubgeom`, `overview.md:448`, `beamcli/doc.go:117` | Docs are right. The comments in `start.go` and `hubgeom` are stale (F1, F2). |
| `lyx` → `lym` text and paths | The `git grep -il lyx` over the tree excluding `go.sum` and `docs/research` returns only the `LyX` guard in `tools/deploy/main_test.go`, and `git ls-files` has no `lyx` path | Covered. |
| PATTERN and `pattern/` | 25 files in the diff | Covered, nothing stale found. |

**Findings**

F1: LOW, consistency, `internal/dobbyengine/start.go:20`
- `LogFileName` is still described as "the dobby log file in the marker directory".
- The directory is gone: the file now sits directly in `.lym/beam/`.
- Evidence: `start.go:20` against `geometry.go`'s `LogPath` comment, which says "under the worktree's `.lym/beam/` directory".

F2: LOW, consistency, `internal/dobbyengine/start.go:242`
- The `Live` doc says "A worktree whose marker directory does not exist yet has never run a dobby".
- The code stats `filepath.Dir(geom.LockPath)`, the `.lym/beam/` directory.
- Evidence: `start.go:242-244`.

F3: LOW, consistency, `internal/dobbyengine/start.go:258`
- The `SpawnDetached` doc says "appended to the marker directory's log".
- The code writes to `geom.LogPath`.
- Evidence: `start.go:258`, `start.go:264-267`.

F4: LOW, consistency, `internal/hubgeom/dobbygeom.go:21`
- The `DobbyGeometry` doc says "The lock, state and marker paths lie under that worktree's `.lym/beam/` directory".
- The paths are now lock, state, log and tidy.
- The marker paths (`LandRequest`, `DobbyStopMarker`, `AwaitMarker`) belong to `PawlPaths`, not this function.
- Evidence: the `DobbyGeometry` doc in the diff, and the `Geometry` literal at `dobbygeom.go:49-62`.

**Tool calls (2)**
1. `git grep -n -i 'marker'` over `internal/dobbyengine`, `internal/beamcli`, `internal/hubgeom/dobbygeom.go`, `docs/overview.md` and `contracts/stencils/beam`, to look for stale marker wording. My context had no scan of those paths, and this confirmed that no other stale `markers` wording remains there.
2. A geometry/callers check was already in my context, so I made no further reads.
