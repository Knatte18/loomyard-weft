# Card 23 — pause-cli

**Create:**
- `internal/pausecli/doc.go`
- `plan:internal/pausecli#Command` -> `func Command() *cobra.Command`
- `plan:internal/pausecli#ResumeCommand` -> `func ResumeCommand() *cobra.Command`
- `plan:internal/pausecli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/pausecli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/pausecli#InstallNetworkGate` -> `func InstallNetworkGate(ctx context.Context)`
- `plan:internal/pausecli#caller` -> `type caller struct`
- `plan:internal/pausecli#callerOf` -> `func callerOf(strand string) caller`
- `plan:internal/pausecli#refuseCaller` -> `func refuseCaller(c caller, slug string) error`
- `plan:internal/pausecli#hubDeps` -> `func hubDeps(location *lyxcwd.Location, strand string) (pauseengine.Deps, []string, error)`
- `plan:internal/pausecli#shuttlePair` -> `type shuttlePair struct`
- `plan:internal/pausecli#shuttlePair.Slug` -> `func (p shuttlePair) Slug() string`
- `plan:internal/pausecli#shuttlePair.Sessions` -> `func (p shuttlePair) Sessions() ([]pauseengine.SessionInfo, error)`
- `plan:internal/pausecli#shuttlePair.Busy` -> `func (p shuttlePair) Busy(guid string) (bool, error)`
- `plan:internal/pausecli#shuttlePair.Live` -> `func (p shuttlePair) Live(guid string) (bool, error)`
- `plan:internal/pausecli#shuttlePair.Interrupt` -> `func (p shuttlePair) Interrupt(guid string) error`
- `plan:internal/pausecli#shuttlePair.Send` -> `func (p shuttlePair) Send(guid, text string) error`
- `plan:internal/pausecli#shuttlePair.Running` -> `func (p shuttlePair) Running() (string, bool, error)`
- `internal/pausecli/cli_test.go`
- `internal/pausecli/testmain_test.go`

**Edit:**
- `cmd/lyx#newRoot`
- `docs/overview.md`
- `tools/sandbox/SANDBOX-CORE-SUITE.md`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-cli-cobra.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the `pause` module (D1): `lyx pause [<slug>]`, `lyx pause --status`, and `lyx resume [<slug>]` mounted at the root beside it, over `pauseengine` (card 22).

Commands, operator audience, each `RunE` checking `clihelp.ShouldAbort` first and answering in JSON through `internal/output`:
- `Command()` is `pause [<slug>]` with `--grace <seconds>` (default 10) and `--status`; `--status` prints `pauseengine.Status`'s live entries and holds and changes nothing.
- `ResumeCommand()` is `resume [<slug>]`, a second root command of the same module, mounted in `newRoot` beside `Command()` the way `loomcli.StartAliasCommand` is; `Long` texts name the other verb and D3's and D6's sequences in short.
- `RunCLI` and `RunCLIIn` follow `PATTERN-cli-cobra`, running `Command()`; tests drive `ResumeCommand()` through `clihelp.ExecuteIn`.
Hub resolution: both verbs resolve the location with `lyxcwd.Resolve` from any worktree of the hub, as `cleanupcli` does; `internal/cliwire`'s dependency set bars `lyxcwd` and `fabricengine` and it serves the standalone-capable CLIs only, so D1's "through `internal/cliwire`" is met by the module's own resolution (Shared Decision `pause-hub-resolution`).
`hubDeps` fills `pauseengine.Deps`: the record path from `hubgeom.PauseGeometry`, every task pair of the hub from `fabricengine.List` (the prime excluded) as a `shuttlePair` over a runner built like the pair's own wiring (reed and shuttle config at the pair's location, `hubgeom.ReedGeometry`), `NetOps` over `parentlock.ProcessAlive`, `Locks` over the hub's `parentlock.Store`, `OrchWatcherLive` over `orchengine.WatcherLive` at `orchcli.PrimePaths` of the prime, and the real clock; it also returns the pair slugs.
`shuttlePair.Sessions` lists the pair's running runs (`shuttleengine.ReadSessionStates`) whose strand reed reports live, `Busy` reads the run's session state as busy, `Interrupt` and `Send` call the runner's, and `Running` reads the pair's loom status file.

Input, each refusal naming its way forward:
- a slug with no pair refuses, listing the slugs that have one;
- `--grace` outside 0 to 60 refuses, naming the range;
- `lyx resume <slug>` with no live entry, and `lyx resume` with none, are no-op successes reporting nothing resumed; `lyx resume <slug>` under only a hub entry refuses, naming `lyx resume` (the engine's refusal).
Caller (`callerOf` over `os.Getenv(agentname.StrandNameEnv)`): empty is `EXTERNAL`; a name `agentname.Parse` reads as the orch role in the prime is the orch; any other strand name is an agent of the slug it carries.
The orch and `EXTERNAL` may run both verbs with or without a slug.
`refuseCaller` refuses an agent's hub-wide pause or resume, and either naming a slug other than its own pair's (an agent with no pair has none), with `pause: an agent may pause or resume only its own pair; way forward: ask your parent session, which runs "lyx pause" or "lyx resume" for you`.
The check guards against a mistaken call: the agents' settings deny any command naming the strand-name variable, so an agent cannot clear it to read as `EXTERNAL`.
The envelope reports the engine's report verbatim under `pause` or `resume`.

Network gate: `InstallNetworkGate(ctx)` resolves the context's cwd, and for a location in a hub installs `pauserecord.InstallGate` with `hubgeom.PauseGeometry`, the process's strand name, pid and start time (`proc.StartTime`), `HoldsParentLock` reading whether a live record of the hub's `parentlock.Store` names the gate's slug, and `Notify` writing its line to stderr and the trace; a location outside a hub installs nothing.
`newRoot`'s `PersistentPreRunE` calls it once, after the logger is armed, so every `lyx` process's network git and GitHub calls are held (cards 5 and 6) while the agents' own `lyx` calls and the operator's (`lyx board`, `lyx selfreport create`) each print the one line.
`newRoot` also mounts both commands and names `pause` and `resume` in `root.Long`'s module list.

`refusal-spec.md`'s `## pause` section gains rows, each with its trigger and way forward: `pause: unknown slug`, `pause: grace out of range`, `pause or resume: agent outside its own pair` (way forward: ask the parent session) and `resume <slug> under a hub-wide pause` (way forward: `lyx resume`); each is a correctness halt except the last, a transient.
`docs/overview.md`'s module list gains **pause**, `tools/sandbox/SANDBOX-CORE-SUITE.md` gains scenario S11 tagged `**Covers:** pause, resume` (pause with nothing running writes an entry and reports empty lists, `--status` shows it, `lyx board` held prints its line, resume lifts it, and the two input refusals), and `pattern/PATTERN-cli-cobra.md`'s package naming lists `pausecli` with `pauseengine`; the budget gains the package's row.

Tests, in `cli_test.go`, against a `hubforge` hub with one pair and no agents running: `pause` with nothing running writes the entry and reports empty lists, and a repeated pause adds none; `--status` lists the live entry and changes nothing; an unknown slug and `--grace 61` refuse with their ways forward; `resume` lifts the entry, and `resume` again reports nothing resumed; `resume <slug>` under only a hub entry refuses naming `lyx resume`; with `LYX_STRAND_NAME` set to an agent of the pair, a hub-wide pause, a hub-wide resume and a pause of another slug refuse naming the parent session, and a pause of its own slug succeeds; with it set to the orch, a hub-wide pause succeeds; a record left on disk survives a fresh resolution and `lyx resume` lifts it; `InstallNetworkGate` in the pair installs a gate whose slug is the pair's, and outside a hub installs none.

**ImpactSummary:** New CLI module mounted twice at the root, plus one call in the root pre-run that installs the network gate for every lyx process in a hub; docs, sandbox and refusal spec gain their rows.
