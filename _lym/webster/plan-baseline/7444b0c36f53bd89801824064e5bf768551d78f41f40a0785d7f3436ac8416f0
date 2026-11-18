# Card 3 — Tell the engines the worktree they launch in

**Create:**
- `plan:internal/shuttleengine#Worktree` -> `type Worktree struct`
- `plan:internal/shuttleengine#WorktreeKind` -> `type WorktreeKind string`
- `plan:internal/shuttleengine#WorktreeTask` -> `const WorktreeTask WorktreeKind = "task"`
- `plan:internal/shuttleengine#WorktreePrime` -> `const WorktreePrime WorktreeKind = "prime"`
- `plan:internal/shuttleengine#WorktreeBeam` -> `const WorktreeBeam WorktreeKind = "beam"`
- `plan:internal/lymdirs#ClaudeAccountsDirName` -> `const ClaudeAccountsDirName = "claude"`
- `plan:internal/fabricengine#ClaudeAccountsDir` -> `func ClaudeAccountsDir(hub string) string`
- `plan:internal/hubgeom#EngineWorktree` -> `func EngineWorktree(l *lymcwd.Location) shuttleengine.Worktree`
- `plan:internal/hubgeom#TestEngineWorktree` -> `func TestEngineWorktree(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#NewForWorktree` -> `func NewForWorktree(cfg shuttleengine.Config, dirs configengine.Dirs, worktree shuttleengine.Worktree) *Claude`
- `plan:internal/shuttleengines#NewForWorktree` -> `func NewForWorktree(cfg shuttleengine.Config, dirs configengine.Dirs, worktree shuttleengine.Worktree) (shuttleengine.EngineSet, error)`

**Extends:**
- `internal/shuttleengine/claudeengine#Claude`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengines/doc.go`
- `internal/hubgeom/doc.go`

**Edit:**
- `internal/shuttleengine/claudeengine#NewFromConfig`
- `internal/shuttleengines#constructor`
- `internal/shuttleengines#registry`
- `internal/shuttleengines#build`
- `internal/shuttleengines#New`
- `internal/shuttleengines#NewWithSeams`
- `internal/shuttleengines#SessionNamers`
- `internal/shuttleengines#RecordReaders`
- `internal/shuttleengines#TestNew`
- `internal/shuttleengines#keepWarmWith`
- `internal/shuttleengines#TestKeepWarm`
- `internal/loomcli#loomCLI.wire`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/orchcli#wireCLI`
- `internal/shuttlecli#Command`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.primeShuttle`
- `internal/battencli#primeShuttleRunner`
- `internal/webstercli#websterCLI.wireHub`
- `internal/gatecli#newStrandRig`

**Uses:**
- `internal/hubgeom#IsPrime`
- `internal/hubgeom#pairIsBeam`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#List`
- `internal/fabricengine#ConfigDirs`
- `internal/lymdirs/doc.go`
- `internal/fabricengine/doc.go`
- `internal/loomcli/doc.go`
- `internal/burlercli/doc.go`
- `internal/webstercli/doc.go`
- `internal/gatecli/doc.go`

**Intent:**
D4 says the runner tells the engine the worktree's name, whether it is the prime or a beam, the hub's account file location and, for a pick, a reader of the hub's current worktree names, as told geometry (`PATTERN-cwd-resolution`).
This card adds that told value and the path that carries it into every launching engine set.

`shuttleengine.Worktree` is engine-neutral: `Name string`, `Kind WorktreeKind` (`WorktreeTask`, `WorktreePrime`, `WorktreeBeam`), `AccountDir string` (the hub's machine-local directory for an engine's account file and usage records, absolute) and `Names func() ([]string, error)` (the hub's current worktree names).
The zero value is "told no worktree": card 6 then resolves only the hub pin.

`lymdirs.ClaudeAccountsDirName` names `.lym/claude` in the Bolt checkout (`PATTERN-lym-files`: a cache a fresh machine starts without), and `fabricengine.ClaudeAccountsDir(hub)` joins it under `HubScratchDir(hub)` the way `AdoptScratchDir` does.
`hubgeom.EngineWorktree(l)` fills the value: `Name` from `l.WorktreeName`; `Kind` prime when `IsPrime(l)`, beam when `pairIsBeam(l)`, task otherwise, a failed prime read logged at Warn and taken as prime, so a worktree of unknown kind is never picked (D9), and a failed beam read logged at Warn and taken as task; `AccountDir` from `fabricengine.ClaudeAccountsDir(l.HubPath)`; `Names` a closure over `fabricengine.List` of the location's worktree directory, returning the base names of the entries that are not prunable and exist on disk, the prime included.

`claudeengine.NewForWorktree` builds the engine `NewFromConfig` builds and keeps the worktree in a new `Claude.worktree` field; `NewFromConfig` now returns `NewForWorktree(cfg, dirs, shuttleengine.Worktree{})`, so its callers keep their meaning.
`shuttleengines`' unexported `constructor` type gains the worktree parameter, the registry's claude constructor calls `claudeengine.NewForWorktree` and codex's ignores it; `build`, `New`, `NewWithSeams`, `SessionNamers` and `RecordReaders` pass `shuttleengine.Worktree{}`, and the new `NewForWorktree` passes its own.
Every caller that launches sessions and holds a `*lymcwd.Location` switches from `shuttleengines.New(cfg, fabricengine.ConfigDirs(loc))` to `shuttleengines.NewForWorktree(cfg, fabricengine.ConfigDirs(loc), hubgeom.EngineWorktree(loc))`: the listed members.
`battenCLI.primeShuttle` passes the prime's location, so its sessions read as the prime's (D9).
`primeShuttleRunner` switches from `shuttleengines.New(shuttleConfig, dirs)` to `shuttleengines.NewForWorktree(shuttleConfig, dirs, hubgeom.EngineWorktree(location))`.
`beamcli`'s `runnerTidyShuttle.Start` launches the dobby's board-tidy agent in the beam worktree through `battencli.ShuttleRunner`, which returns that runner, so the agent runs under the beam's account (D4, D9).
Readers that only stop or read records keep `New`, since the known directories of card 7 come from the dirs, not the worktree.

`TestEngineWorktree` is tier1, in package `hubgeom`: on a `hubforge` hub with its prime, a task pair, a beam pair marked through the origin record and a pair whose board entry carries the beam flag before any origin mark, it asserts each location's `Name` and `Kind`, the last reading beam, the `AccountDir` path under the Bolt's `.lym`, and that `Names` lists all three and drops a pair whose directory was removed.
`keepWarmWith` and the constructor map literal of `TestKeepWarm` take the new parameter.
`TestNew` (tier0) gains a row that `NewForWorktree`'s claude engine carries the told worktree and `New`'s carries the zero value.

**ImpactSummary:** One told value added; `New`, `NewWithSeams`, `NewFromConfig` keep their signatures, and the nine launching wiring members switch to `NewForWorktree`, each a one-line change beside a Location it already holds.

**Commit:** 3: shuttleengines: tell launching engines their worktree's name, kind and account directory
