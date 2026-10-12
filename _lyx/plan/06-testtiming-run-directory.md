# Card 6 — testtiming-run-directory

**Edit:**
- `cmd/testtiming#prebuildLyx`
- `cmd/testtiming#runRedundancy`
- `cmd/testtiming#runResources`
- `cmd/testtiming#measurePackageResources` -> `func measurePackageResources(tags, importPath string, env []string, parallel int) (resourceRow, error)`
- `cmd/testtiming#leftoverProcesses` -> `func leftoverProcesses(marker string) []string`
- `cmd/testtiming#main`
- `cmd/testtiming#TestPrebuildLyx`

**Delete:**
- `cmd/testtiming#referencesDir`
- `cmd/testtiming#TestReferencesDir`

**Create:**
- `cmd/testtiming#leftoverMarkerEnv`
- `cmd/testtiming#carriesMarker`
- `cmd/testtiming#TestCarriesMarker`

**Uses:**
- `internal/lyxtemp#MkdirTemp`
- `internal/lyxtemp#Dir`
- `internal/lyxtemp#Remove`

**Intent:**
`cmd/testtiming`'s directories are created under the process run directory through `lyxtemp.MkdirTemp`, keeping their prefixes and their existing removal on normal exit (D5): the prebuilt-`lyx` directory in `prebuildLyx`, the coverage directory in `runRedundancy` and the per-package trace directory in `measurePackageResources`.
`main` calls `lyxtemp.Remove()` on every exit path, the error path before `os.Exit(1)` included.

The per-package `ttr-*` directory `runResources` hands each child as `TMPDIR`, `TMP` and `TEMP` goes away rather than moving.
Under the lyx root, a child given a `TMPDIR` that is not a run directory would put its own root and run directory below it, which pushes a socket path past `checkSocketPath`'s limit; a child that inherits testtiming's environment instead gets its run directory in the same root through `lyxtemp`'s beside-the-parent rule.
The directory existed only to attribute leftover processes, so attribution moves to an environment marker: `measurePackageResources` sets `leftoverMarkerEnv` to a value unique to the package's run (the import path and testtiming's pid) in the child's environment, and every descendant inherits it.
`leftoverProcesses(marker)` lists each process other than testtiming whose `/proc/<pid>/environ` `carriesMarker` reports, as `pid argv` as before; `carriesMarker` matches the whole `KEY=value` entry among the NUL-separated entries, never a substring of another entry.
`referencesDir`, whose only callers are `leftoverProcesses` and `TestReferencesDir`, is deleted with that test; `TestCarriesMarker` in `resources_test.go` replaces it, with rows for an exact entry, a longer value sharing the prefix, the marker key with another value, an empty environ and a marker in the last entry without a trailing NUL.
`main.go`'s package comment drops "Each package gets a private temp directory its child's TMPDIR, TMP and TEMP point at" and says LEFTOVER counts the processes still carrying the package's marker after the child exited.

`TestPrebuildLyx` asserts the built binary's directory lies under `lyxtemp.Dir()`.

**ImpactSummary:** testtiming's temp directories move into its run directory and the leftover column is attributed by an inherited environment marker; referencesDir's two callers are on this card.
