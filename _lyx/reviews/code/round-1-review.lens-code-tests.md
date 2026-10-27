```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing the test that pins each behaviour the diff changes.

**Behaviours and their pinning tests**

- **Geometry: `LogPath` and `TidyDir` lie directly under `.lym/beam/`.**
  - Pinned by `internal/hubgeom/dobbygeom_test.go:94`.
  - It checks that the directory of each path equals `beamDir`.
  - It does not pin the file name `dobby.log` or the directory name `tidy`, because it checks only the parent directory.
  - The base-name gap is below the finding threshold, since the names are constants in the production code.
- **The tidy report is named by UTC timestamp alone, `<TidyDir>/<20060102T150405Z>.md` (`internal/dobbyengine/tidy.go`, `reportPath`).**
  - Pinned by none, as far as I could see.
  - `tidy_test.go:183` reads `ReportPath` back from state and only asserts the notice text built from it.
  - `tidy_test.go:233` and `:237` assert on a hard-coded path string passed to `TidyPrompt`, not on the path the daemon builds.
  - The old `board-tidy-` prefix and the new bare-timestamp name are therefore both unpinned.
- **The tidy report goes in `TidyDir`, which is created by `MkdirAll` (`tidy.go:166`).**
  - Partly pinned: the harness sets `TidyDir` to `root/TidyDirName` at `tidy_test.go:54`.
  - No assertion checks that the report path is under `TidyDir`.
- **`SpawnDetached` writes its log to `geom.LogPath` and creates its parent directory (`start.go:264`).**
  - Pinned by none.
  - `SpawnDetached` returns `ErrSpawnUnderTest` in a test binary, so no test reaches the `LogPath` use.
  - This is a pre-existing limitation, and the logic changed only from `Join(MarkerDir, LogFileName)` to `LogPath`.
- **`Live` treats a missing `.lym/beam/` directory as not live (`start.go:243`).** I did not open the existing `Live` tests, so I cannot say which test pins it. The behaviour is unchanged apart from the rename.
- **`MarkerDir` and `MarkerDirName` are removed.** Nothing to test: the build fails if anything still references them.

**Test economy:** I found no new test or row that duplicates an existing one. The diff changes tests only by renaming and by the `TidyDir` and `LogPath` field changes.

**Findings**

1. **LOW, design.**
   - The location is `internal/dobbyengine/tidy_test.go:183`, covering `internal/dobbyengine/tidy.go:166-168`.
   - The new report location `<TidyDir>/<UTC timestamp>.md` is asserted by no test, so reverting to the `board-tidy-<ts>.md` name would still pass.
   - Fix: add an assertion to the existing tidy-step test that `h.state().Tidy.ReportPath` equals `filepath.Join(geom.TidyDir, <fixed-now UTC "20060102T150405Z">+".md")`.
   - That test is the one that reads `ReportPath` at line 183, so this adds an assertion rather than a new test.

**Tool calls (1)**
- `sed -n 85,110p internal/hubgeom/dobbygeom_test.go` plus a grep of `internal/dobbyengine/tidy_test.go`: the context lacked the test bodies, and I needed them to see what they assert about `LogPath`, `TidyDir` and the report name.
