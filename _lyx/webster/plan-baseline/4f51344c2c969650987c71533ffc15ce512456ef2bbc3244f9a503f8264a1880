# Card 4 — dobby-records

**Create:**
- `internal/dobbyengine/doc.go`
- `internal/dobbyengine/template.yaml`
- `internal/lyxdirs#BeamDirName`
- `internal/dobbyengine#Geometry`
- `internal/dobbyengine#State`
- `internal/dobbyengine#RunState`
- `internal/dobbyengine#StartState`
- `internal/dobbyengine#ReadState`
- `internal/dobbyengine#UpdateState`
- `internal/dobbyengine#PassExitOwnedElsewhere`
- `internal/dobbyengine#PassExitCovered`
- `internal/dobbyengine#PassExitContinueRefused`
- `internal/dobbyengine#Config`
- `internal/dobbyengine#ConfigTemplate`
- `internal/dobbyengine#ConfigSpec`
- `internal/dobbyengine#LoadConfig`
- `internal/dobbyengine#TestState_RoundTripKeepsUnknownKeys`
- `internal/dobbyengine#TestLoadConfig`
- `internal/dobbyengine#TestToldGeometryInvariant_AllowlistOnly`
- `internal/dobbyengine#dobbyengineAllowedImports`

**Edit:**
- `internal/configreg#Modules`
- `internal/lyxdirs/dirs.go`
- `cmd/lyx#configStrictnessStrictSet`
- `pattern/PATTERN-cwd-resolution.md`

**Intent:**
Start `internal/dobbyengine`, the beam module's told-geometry daemon engine (D1), with the records every later card reads and writes, and the beam module's config, `beam.yaml`.
`lyxdirs.BeamDirName` names the `.lyx/beam/` directory of a worktree, the only place dobby's lock, state record and markers live (`PATTERN-durable-vs-ephemeral-state`); `dirs.go`'s header comment lists it.
`Geometry` holds absolute paths only, told by the hub wiring (`PATTERN-cwd-resolution`, the package never imports `internal/lyxcwd`): the hub root, the board dir, the prime's anchor, the worktree's root and name, the beam's slug (empty for the prime, the root beam), the lock path, the state path and the marker directory under `.lyx/beam/`, the prime's batten runs directory and the lyx executable.
`State` is a tolerant JSON record written only through `internal/state` and keeping unknown keys (`PATTERN-tolerant-records`): the dobby's pid and start time, the sleep intervals it saw (`[]pauserecord.Interval`), `Runs` keyed by slug (`RunState`: full pass owed, owned elsewhere, owner notice sent, owner pause failed, last pass exit time, the failing-pass error text with its count and the texts noticed), `Starts` keyed by slug (`StartState`: next try, interval, the refusal texts noticed, the board entry fingerprint), the landings counted by run, the landed slugs since the last tidy, and the tidy in flight (strand guid, report path, deadline).
`ReadState` reports found false for an absent file; `UpdateState` reads, mutates and writes under the record's lock.
Both read an undecodable record (`state.ErrDecode`) as absent with one Warn, as card 13's `loadMemory` reads its own record, and `UpdateState` then writes a fresh record, so a hand-damaged file never wedges the beam; card 32 reads a fresh record, which holds no pid, as a start wherever it meets it, at `Run`'s start or on a tick mid-run, so it re-records its pid and owes every run a full pass, and card 36 counts every Land Done already there as seen.
The three exit codes are what a `lyx batten run` pass exits with for a refusal dobby reads by code (D4), declared here because battencli imports this package and dobby reads them; `gatecli.SlotBusyExit` keeps 75.
`beam.yaml` holds `max_runs: 3`, `board_tidy_every: 5` and `board_tidy_timeout_min: 30`, each a per-worktree key open to every layer (`configengine.CategoryNone`), loaded with the strict `configengine.Load` (`PATTERN-config-layers`), since dobby only ever runs inside a hub, so `internal/dobbyengine` joins `cmd/lyx#configStrictnessStrictSet`, the pinned set of strict loaders; `LoadConfig` refuses a negative value naming the key and `lyx config beam --set`, and 0 means no start for `max_runs` and no tidy for `board_tidy_every`.
`configreg.Modules` registers `beam` in sort order between `batten` and `board`, with its template, spec and `CheckLoad`.
The package doc states the engine's place: the daemon behind the hidden `lyx beam dobby` verb, one per beam worktree on its owning machine, told every path.
Tests (tier0, parallel): `TestState_RoundTripKeepsUnknownKeys` writes a state with a foreign top-level key and a foreign run key and reads both back after an `UpdateState`, and a row writes an undecodable file that `ReadState` reads as absent and `UpdateState` rewrites as a fresh record; `TestLoadConfig` rows: template defaults, a worktree layer's `max_runs: 0`, a negative `board_tidy_every` refused naming the key.
`TestToldGeometryInvariant_AllowlistOnly` (tier0, in `seam_enforcement_test.go`, as `internal/sheddriver`'s and `internal/battenshed`'s are) pins the package's production imports to `dobbyengineAllowedImports` with `internal/lyxcwd` and every CLI package denied, through `scankit.AssertImportAllowlistNoStale`, which fails on a stale entry as on a missing one, so the list holds only this card's own imports (`internal/configengine`, `internal/state`, `internal/pauserecord`, `internal/logger` for the Warn, and `gopkg.in/yaml.v3`, through which `LoadConfig` decodes the document `configengine.Load` returns, as `internal/battenshed`'s `LoadConfig` does) and each later card that adds an import to the package adds it to the list.
`pattern/PATTERN-cwd-resolution.md`'s "Bound packages" list gains `dobbyengine`.

**ImpactSummary:** A new package and one registry row; configreg's template-classification test covers the new keys, and nothing else imports the package yet.
