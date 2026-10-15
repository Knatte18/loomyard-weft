# Card 12 — hubgeom-creel-geometry

**Create:**
- `internal/hubgeom/creelgeom.go`
- `plan:internal/lyxdirs#CreelDirName` -> `const CreelDirName = "creel"`
- `plan:internal/hubgeom#CreelGeometry` -> `func CreelGeometry(hubPath, tempRoot string) (creelengine.Geometry, error)`
- `plan:internal/hubgeom#TestCreelGeometry` -> `func TestCreelGeometry(t *testing.T)`

**Edit:**
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Build creel's told geometry in the one place geometry structs are built (`PATTERN-told-geometry`).
`lyxdirs.CreelDirName` is the directory under the hub's board-level `.lyx` that holds the journal and the daemon lock (`internal/lyxdirs/dirs.go`).
`CreelGeometry` takes the hub path, not a Location, because the daemon runs from the hub directory, which is no worktree, and the lyxtemp root path, which its caller opens with `lyxtemp.ProcessRoot()` (card 14), so building the geometry opens nothing: it reads the shortname with `fabricengine.ReadShortname(fabricengine.BoardDir(hubPath))`, sets `StateDir` to `<HubScratchDir>/creel`, `JournalDir` to `StateDir/journal`, `LockPath` to `StateDir/daemon.lock`, and `SocketPath` to `creelengine.SocketName(shortname, hubPath)` joined to `tempRoot`.
A hub with no recorded shortname is an error naming the shortname record and the hub.
`TestCreelGeometry` (`internal/hubgeom/hubgeom_test.go`) builds a hub directory with a shortname record and asserts every field, that one hub path gives one socket path on two calls, that two hub paths of one shortname get different socket paths, and that a hub with no shortname record fails naming it.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/hubgeom` by one.

**ImpactSummary:** One additive lyxdirs constant and one new hubgeom constructor with its test; no existing caller changes.
