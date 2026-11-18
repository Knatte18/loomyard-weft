# Card 7 — Readers of Claude's config directory read every known directory

**Create:**
- `plan:internal/shuttleengine/claudeengine#Claude.knownConfigDirs` -> `func (c *Claude) knownConfigDirs() []string`
- `plan:internal/shuttleengine/claudeengine#Claude.sessionProjectDir` -> `func (c *Claude) sessionProjectDir(sessionID, workdir string) (string, error)`
- `plan:internal/shuttleengine/claudeengine#newestTranscript` -> `func newestTranscript(configDirs []string, workdir, sessionID string) (string, bool)`
- `plan:internal/shuttleengine/claudeengine#TestClaude_LiveReadersReadTheLaunchedDirectory` -> `func TestClaude_LiveReadersReadTheLaunchedDirectory(t *testing.T)`
- `plan:tools/tokencount#TestLocateSessions_ListedAccount` -> `func TestLocateSessions_ListedAccount(t *testing.T)`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.LocateSessions`
- `internal/shuttleengine/claudeengine#Claude.SessionHolder`
- `internal/shuttleengine/claudeengine#sessionHolder` -> `func sessionHolder(sessionID string, registryDirs []string, alive func(pid int) bool, startTime func(pid int) (string, bool)) (int, bool, error)`
- `internal/shuttleengine/claudeengine#Claude.ProcessLiveness`
- `internal/shuttleengine/claudeengine#processLiveness` -> `func processLiveness(sessionID string, registryDirs []string, alive func(pid int) bool, startTime func(pid int) (string, bool)) shuttleengine.Liveness`
- `internal/shuttleengine/claudeengine#Claude.AuditForksIncremental`
- `internal/shuttleengine/claudeengine#Claude.OutstandingForks`
- `internal/shuttleengine/claudeengine#Claude.SessionNameDrift`
- `internal/shuttleengine/claudeengine#Claude.SessionUsage`
- `internal/shuttleengine/claudeengine#Claude.installedPlugins` -> `func (c *Claude) installedPlugins(configDir string) ([]string, error)`
- `internal/shuttleengine/claudeengine#Claude.sessionScopeFor` -> `func (c *Claude) sessionScopeFor(cfg ScopeConfig, skills []string, role, configDir string) sessionScope`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#Claude.OpenLaunch`
- `internal/shuttleengine/claudeengine#Claude.resolvedCandidates`
- `internal/shuttleengine/claudeengine#TestPrepare_ExportsTheResolvedAccount`
- `internal/shuttleengine/claudeengine#TestLocateSessions`
- `internal/shuttleengine/claudeengine#TestSessionHolder`
- `internal/shuttleengine/claudeengine#TestProcessLiveness`
- `internal/shuttleengines#SessionNamers` -> `func SessionNamers(dirs configengine.Dirs) map[string]reedengine.SessionNamer`
- `internal/shuttleengines#RecordReaders` -> `func RecordReaders(dirs configengine.Dirs) map[string]shuttleengine.RecordReads`
- `internal/shuttleengines#TestSessionNamers`
- `internal/shuttleengines#TestRecordReaders`
- `internal/reedcli#enterSession`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.stopStepRuns`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.primeShuttle`
- `internal/gatecli#newStrandRig`
- `tools/tokencount#main`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengines/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#canonicalConfigDir`
- `internal/shuttleengine/claudeengine#projectDirUnder`
- `internal/shuttleengine/claudeengine#locateSessions`
- `internal/shuttleengine/claudeengine#Claude.resolveLaunchAccount`
- `internal/envconfig#LoadConfig`
- `internal/fabricengine#ConfigDirs`
- `internal/reedcli/doc.go`
- `internal/loomcli/doc.go`
- `internal/gatecli/doc.go`

**Intent:**
D8.
`Claude.knownConfigDirs` returns, each once by `canonicalConfigDir`: every listed account's directory, retired ones included; the `env` module's `CLAUDE_CONFIG_DIR` when set; and `~/.claude`.
An engine told no hub board directory takes the process's own `CLAUDE_CONFIG_DIR` and `~/.claude`.
A `claude.accounts` that fails to load is logged at Warn and the remaining directories are read.

- `LocateSessions` lists the worktree's project in every known directory through `locateSessions` and returns a session id found in several once: the most recently written copy, which by D6 holds every turn.
- `sessionProjectDir` is the project directory of the session's launched directory (D8): the known directory whose `sessions` registry holds a live entry for the session, which a running session has only under the directory it was launched with; for a session with no live entry, as a run from before the change, the worktree's resolution (`resolveLaunchAccount`), then the other known directories, the first holding `<sessionID>.jsonl`.
  The live-transcript readers `AuditForksIncremental`, `OutstandingForks`, `SessionNameDrift` and `SessionUsage` read through it instead of `claudeProjectDirFor`.
- `sessionHolder` and `processLiveness` take every known directory's `sessions` registry as one set: any live entry reads alive or holds; dead needs a dead entry and no unreadable registry or entry; anything else is unproven, and for `sessionHolder` an unreadable registry is an error unless a live holder is found.
- `installedPlugins`, `sessionScopeFor` and the launch manifest's user setup (`resolvedCandidates`, which hands it `claudeDir`) take the launch's effective directory from card 6's resolution, which `Prepare` and `OpenLaunch` pass.

The live readers get only a session id and a workdir from shuttle, so they find the launched directory through the live registry entry Claude Code writes under it, not through card 6's `account.json` in a run directory they are not told.

Every engine that reads the known directories is built with the hub's config `Dirs`: `SessionNamers` and `RecordReaders` gain a `dirs` parameter and pass it to the constructors.
Callers: `loomCLI.wire`, `loomCLI.stopStepRuns`, `battenCLI.wire`, `battenCLI.primeShuttle` and `newStrandRig` pass the `ConfigDirs` of the location they already hold; `reedcli`'s `enterSession` passes `fabricengine.ConfigDirs(location)` of the watched session's location it already resolves, so the namers read that worktree's `env` value; `tokencount`'s `main` resolves the worktree it is pointed at through `lymcwd` and passes its `ConfigDirs`, and `configengine.Dirs{}` outside a hub.
Hook-driven readers take `transcript_path` from the payload and do not change.

The docs' one-directory sentences are rewritten.
In `claudeengine/doc.go`: the worktree's project directory as `<home>/.claude/projects/<dir>` becomes the project directory under every known directory; the plugins file under `CLAUDE_CONFIG_DIR` or else `~/.claude`, and the config directory as `CLAUDE_CONFIG_DIR`, else `.claude` under the home directory, become the launch's effective directory; the prober's liveness read from one registry, and the one reader of registry entries the resume check and the prober share, become every known directory's registry read as one set by `SessionHolder` and the prober.
In `shuttleengines/doc.go`: `RecordReaders` built with no config directories, as `SessionNamers` builds its engines, becomes both built with the hub's `Dirs`.

Tests are tier0 with `HOME` and the account directories under `t.TempDir()`:
`TestLocateSessions` rows: a session under a listed non-default directory and one under a retired account's directory are found; a session id in two directories is returned once, as the newer copy; a machine layer whose `accounts` fails to load is logged and `~/.claude` is still read; an engine told no board directory reads the process's `CLAUDE_CONFIG_DIR` and `~/.claude`.
`TestSessionHolder` and `TestProcessLiveness` pass two registry directories: a live entry in the second holds or reads alive; a dead entry beside an unreadable second registry reads unproven.
`TestClaude_LiveReadersReadTheLaunchedDirectory`: a session registered live under account A after the worktree's entry moved to B, with an older copy under B, is read by the fork audit and the outstanding-forks read from A; with no live entry it is read under the worktree's resolution, B.
`TestPrepare_ExportsTheResolvedAccount` gains a row: the scope's installed plugins and the manifest's user setup come from the exported account's directory, not `~/.claude`.
`TestRecordReaders` gains a row, with `HOME` set and so not parallel: built with dirs whose machine layer lists a non-default account, its transcripts reader locates and reads a session there, which the session-end usage recorder reads through.
`TestLocateSessions_ListedAccount` (tier0, `tools/tokencount`) counts a session run on a listed non-default account through `RecordReaders` built with the hub's dirs.
`TestSessionNamers` passes `configengine.Dirs{}`.

**ImpactSummary:** Read paths of the claude engine widen from `~/.claude` to the known set; two `shuttleengines` builders gain a `dirs` argument whose seven production callers already hold or can resolve a location.

**Commit:** 7: claudeengine: read sessions, registries and plugins under every known config directory
