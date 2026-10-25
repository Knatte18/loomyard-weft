# Card 27 — dobby-start

**Create:**
- `plan:internal/dobbyengine#StartDeps` -> `type StartDeps struct`
- `plan:internal/dobbyengine#StartReport` -> `type StartReport struct`
- `plan:internal/dobbyengine#EnsureLive` -> `func EnsureLive(deps StartDeps) (StartReport, error)`
- `plan:internal/dobbyengine#Live` -> `func Live(geom Geometry) (bool, error)`
- `plan:internal/dobbyengine#SpawnDetached` -> `func SpawnDetached(geom Geometry) error`
- `plan:internal/hubgeom#DobbyGeometry` -> `func DobbyGeometry(prime *lyxcwd.Location, worktree string) (dobbyengine.Geometry, error)`
- `plan:internal/hubgeom#DobbyStart` -> `func DobbyStart(prime *lyxcwd.Location) dobbyengine.StartDeps`
- `plan:internal/dobbyengine#TestEnsureLive` -> `func TestEnsureLive(t *testing.T)`
- `plan:internal/hubgeom#TestDobbyGeometry` -> `func TestDobbyGeometry(t *testing.T)`

**Uses:**
- `plan:internal/dobbyengine#Geometry`
- `plan:internal/lyxdirs#BeamDirName`

**Intent:**
Add the start of every dobby this machine owns, the body `lyx beam start` and its callers share (D2); it lives below every CLI package, since `orchcli`, `battencli`, `beamcli` and `adoptcli` all call it and `battencli` imports `orchcli`.
`hubgeom.DobbyGeometry` tells the engine its paths for the worktree named `worktree` (the prime's name or a beam's slug): the hub root, the board dir, the prime's anchor, the worktree's root and name, the beam slug (empty for the prime), the lock, state and marker paths under that worktree's `.lyx/beam/` built from `lyxdirs.DotLyxDirName` and `lyxdirs.BeamDirName`, the prime's batten runs directory and the lyx executable; an absent worktree is an error naming it.
`Live` reports the dobby live when its lock is held, by a non-blocking try that releases at once on success.
`SpawnDetached` starts `<lyx> beam dobby <worktree root>` detached in its own process group with the hub root as its working directory and its output appended to the marker directory's log, logs the spawn at Info (`PATTERN-spawn-observability`), and starts nothing under `go test`.
`hubgeom.DobbyStart` fills `StartDeps` over the hub: this machine's name from `internal/machinename`, the board's owners, the prime owner claim (`boardengine.SetOwners` with `Prime`, as a batten resume claims it), the local pairs whose origin record carries the beam mark with their effective owners through `hubgeom.ParentOf`, `DobbyGeometry`, `Live` and `SpawnDetached`.
`EnsureLive` records this machine as the prime owner when none is recorded; with another machine recorded it starts no prime dobby and reports `prime: owned by <machine>; lyx adopt --prime takes it over here`; it then starts the dobby of the prime, when this machine owns it, and of every local beam pair this machine owns, each only when `Live` reads it not live, and reports per worktree `started` or `live`; a failed spawn is reported for its worktree and the others still start, and the error lists the failures.
It is idempotent: a second call reports every dobby `live` and changes nothing.
Tests: `TestEnsureLive` (tier0, parallel) over fake deps: no prime owner records this machine and starts the prime and an owned beam; a second call reports both live and spawns nothing; another machine's prime starts no prime dobby and reports its owner; a beam owned elsewhere is skipped; one failed spawn still starts the rest and returns an error naming it.
`TestDobbyGeometry` (tier0, over a temp layout with no git) asserts the paths lie under the named worktree's `.lyx/beam/` and the batten runs directory under the prime, and that a worktree name with no directory is an error naming it.

**ImpactSummary:** New start helpers in the engine and two hub constructors; nothing calls them yet.
