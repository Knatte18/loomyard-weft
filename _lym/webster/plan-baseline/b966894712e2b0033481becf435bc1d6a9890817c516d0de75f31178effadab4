# Card 10 — slack-cli

**Create:**
- `internal/slackcli/doc.go`
- `plan:internal/slackcli#Command` -> `func Command() *cobra.Command`
- `plan:internal/slackcli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/slackcli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`
- `plan:internal/slackcli#slackCLI` -> `type slackCLI struct`
- `plan:internal/slackcli#slackCLI.command` -> `func (c *slackCLI) command() *cobra.Command`
- `plan:internal/slackcli#slackCLI.preRun` -> `func (c *slackCLI) preRun(cmd *cobra.Command, args []string) error`
- `plan:internal/slackcli#paneRefusal` -> `func paneRefusal(caller paneCaller, allowOrch bool, wayForward string) error`
- `plan:internal/slackcli#paneCaller` -> `type paneCaller struct`
- `plan:internal/slackcli#programMailbox` -> `type programMailbox struct`
- `internal/slackcli/testmain_test.go`
- `internal/slackcli/cli_test.go`
- `plan:internal/slackcli#TestSlackCLI` -> `func TestSlackCLI(t *testing.T)`

**Edit:**
- `cmd/lym#newRoot`
- `cmd/lym#excludedModules`
- `cmd/lym/seamsignature_test.go`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/creelengine#PaneOf`
- `internal/creelengine#IsOrchStrand`
- `internal/creelcli#HubStrands`
- `internal/creelcli#GoClient.ReadWait`
- `internal/creelcli#NewProgramClient`
- `internal/fabricengine#PrimeName`
- `internal/lymcwd#NamedWorktree`
- `internal/hubgeom#SlackGeometry`
- `internal/hubgeom#PauseGeometry`
- `internal/hubgeom#DobbyStart`
- `internal/pauserecord#ReadCoverage`
- `plan:internal/slackengine#RunRelay`
- `plan:internal/slackengine#EnsureRelay`
- `plan:internal/slackengine#StopRelay`
- `plan:internal/slackengine#SetPresence`
- `plan:internal/slackengine#AwaitFirstRead`
- `plan:internal/slackengine#AwaitReparent`
- `plan:internal/slackengine#SpawnRelayFromIntermediate`

**Intent:** Add the `lym slack` CLI module (D9), mounted in `cmd/lym/main.go`'s `newRoot` beside `creelcli.Command()` (`PATTERN-cli-cobra`), wiring slackengine over the hub.

`Command` is the `slack` group, which carries no audience, with the verbs below, each with a one-line `Short`, `Long` under the byte cap and an audience: every verb `operator` but `relay`, which is hidden with the `internal` audience, since only the start's double fork runs it.
What `relay --fork` lets through: a process in any pane, a non-orch strand's included, starts the hub's relay as `EnsureRelay` does, out of every pane, past `lym slack start`'s pane refusal.
Its bound: it starts only a configured hub's relay, writes no presence, token or setup file, and starts what the next prime dobby tick while presence is `slack`, or the next bring-up of the prime, would start.
A run by hand without `--fork` is bounded on Linux by the refused first read (D3).
Off Linux creel places a program claim by environment and never refuses it by pane, so a by-hand run is accepted as `SLACK` there, bounded by creel's environment-only identity, which D7's trust bound already rests on.
`away`, `back` and `status` are operator verbs, so the orch's command index lists them (D9).
Each `RunE` checks `clihelp.ShouldAbort` first and prints the JSON envelope through `internal/output`; a refusal exits 1 and writes nothing.
`slackCLI` holds the location, the prime's name, the `slackengine.Geometry` and seams (location resolver, seam wiring, pane caller, Slack client constructor with its base URL, mailbox constructor, ensure, stop, await, reparent wait, intermediate spawn, relay run, clock, beams and pause readers).
`Command` sets `slackCLI.preRun` as the group's `PersistentPreRunE`, over the production location resolver (`lymcwd.Resolve`) and seam wiring, and hands the tree `slackCLI.command` builds.
For every verb but `relay`, `preRun` resolves the location, then the prime's name through `fabricengine.PrimeName`, which the ensure seam passes to `EnsureRelay`, and fills every seam with its production value.
For `relay` it resolves no location, as creel's `daemon --hub-path` verb resolves none, since the relay runs from the hub directory, which is no worktree; it fills only the intermediate spawn, the reparent wait and the relay run, each taking the hub path and prime name from `--hub-path` and `--prime`.
A tier0 test builds a `slackCLI` with every seam a fake and its geometry under `t.TempDir()` and runs `command()`, with no pre-run but in the one row that installs `preRun` over a fake resolver and wiring, so it drives every verb without a hub, Slack or creel. No seam is nil once wired, and a test sets every seam its verb reads.

The pane check (D2, D3, D6), `paneRefusal`: `paneCaller{PID, Strands, StrandsErr, ParentPID, GOOS, StrandName}` is filled from `os.Getpid()`, `proc.ParentPID` and `creelcli.HubStrands(hub)`, and the walk is `creelengine.PaneOf`, creel's own.
A caller in a live strand's pane is refused unless `allowOrch` holds and `creelengine.IsOrchStrand` names that strand; the refusal names the strand and the way forward.
A caller in no known pane while `PaneOf` reports what is unknown (a live strand's pane process, or the hub's strands unreadable) is refused naming it, with the way forward "wait until it is known or that strand ends, then run the verb again".
Off Linux the strand comes from `paneCaller.StrandName` (`LYM_STRAND_NAME`) naming a strand of the hub, as creel's resolution reads it there.
What the check lets through: a process outside every pane while every live pane process is known, and an orch's pane where allowed; a strand that starts a process outside its pane on purpose is not stopped (D6's bound).

The verbs:
- `login` (no orch allowed; way forward: the operator's terminal) reads the token from standard input, never an argument, refuses an empty one, calls `AuthTest`, refuses a token Slack rejects naming Slack's error code with the way forward "copy the app's Bot User OAuth Token (`xoxb-…`) from its OAuth & Permissions page and run `lym slack login` again", and only then writes the token file (D2). Its `Long` names the bot scopes `channels:history` and `channels:read` for a public channel or `groups:history` and `groups:read` for a private one, `users:read` and `chat:write`.
- `setup` (Use `setup --channel <id> --operator <user-id>`, both flags marked required; no orch allowed; way forward: the operator's terminal) refuses with no token file naming `lym slack login`, checks the channel with `ConversationInfo`, and refuses a channel the bot is not a member of, or one Slack answers `channel_not_found` for, as Slack does for a private channel the bot was not invited to, naming `/invite @<app>` in that channel and a re-run of `lym slack setup`; a `missing_scope` names the bot scopes itself, `channels:history` and `channels:read` or `groups:history` and `groups:read`, `users:read` and `chat:write`, as login's `Long` does, and a reinstall of the app; it checks the user with `UserInfo`, and a `user_not_found` names how to copy the member id; only then it writes the setup file. Its `Long` says to invite the bot, how to copy a member's Slack user id from their profile, and to give each machine's hub its own channel.
- `start` (orch allowed; way forward: the operator's terminal or an orch's pane) refuses through `RequireConfigured`, then `EnsureRelay` with the prime's name and reports `started` or `live`.
- `stop` (orch allowed; same way forward) sets presence `terminal` through `SetPresence`, with its notices, then ends the relay through `StopRelay` with its own pid, so no dobby tick restarts it; a `StopRelay` error names "run `lym slack stop` again", which progresses since the presence write is then no change and sends no notice.
- `away` (orch allowed; way forward: the operator's terminal or the Slack text `away`) refuses through `RequireConfigured`, `EnsureRelay`s, waits with `AwaitFirstRead` up to 30 seconds and refuses with its message, which names the relay log or a live relay's recorded creel refusal and a re-run of `lym slack away`, and only then sets `slack` through `SetPresence`.
- `back` (orch allowed; way forward: the operator's terminal or the Slack text `back`) sets `terminal` through `SetPresence`.
- `status` (no pane check) prints the presence, whether the relay is live, whether a pause covers the prime orch (`hubgeom.PauseGeometry` of the prime and `pauserecord.ReadCoverage`), the channel and operator id from the setup file and `RelayState.LastSlackOK`.
- `relay` (Use `relay --hub-path <hub> --prime <name>`, both flags marked required, with `--fork` and `--after <pid>` declared as optional flags the Use string does not list, so `TestCLIForm_EveryCommand`'s Use check admits it): with `--fork` it is the double fork's intermediate and calls `SpawnRelayFromIntermediate` with the prime's name, then exits; otherwise, with `--after`, it first calls `AwaitReparent` and exits 1 with its error when the bound passes, taking no lock; then the relay run builds the prime's location as `lymcwd.NamedWorktree(hub, name)`, since `fabricengine.PrimeName` lists worktrees from a worktree's anchor and a hub path names none, and builds `RelayDeps` over it — `programMailbox` over `creelcli.NewProgramClient(prime, slackengine.SlackProgram)`, `slackengine.NewAPI(slackengine.DefaultBaseURL, token, …)`, `Hub.Beams` from `hubgeom.DobbyStart(prime).Beams`, `Hub.Paused` from the receiving orch's pause coverage over `hubgeom.PauseGeometry` of that orch's worktree, the prime for `<shortname>:orch` and `lymcwd.NamedWorktree(hub, slug)` for a beam's orch, `hubgeom.SlackGeometry(hub)` as its geometry, `PID` `os.Getpid`, `ReexecLockWait` 10 seconds, `selfexec.New().CheckWith`, `proc.ReapInheritedChildren` and the runproc registration — and runs `RunRelay` until a signal ends it.
`SetPresence`'s mailbox in `stop`, `away` and `back` is the same program client, so every notice comes from `<shortname>:SLACK`.

`cmd/lym/sandbox_coverage_test.go`'s `excludedModules` gains a `slack` entry, since the module is now registered: "its relay calls the live Slack Web API with the machine's bot token, which no sandbox scenario may reach; exercised by internal/slackcli's and internal/slackengine's own tests instead".
`cmd/lym/seamsignature_test.go`'s two signature lists gain `slackcli.RunCLI` and `slackcli.RunCLIIn`.
`docs/overview.md` gains the two package-tree lines for `internal/slackengine` and `internal/slackcli` and a module bullet for **slack** (the per-hub relay, its verbs and its files).
`internal/slackcli/doc.go` documents the package.
`internal/slackcli/testmain_test.go` is `untagged`, through `tmuxkit.Main`.
`internal/slackcli/cli_test.go` is tier0, driving `slackCLI.command()` over a `slackCLI` with fakes and an `httptest.Server` as Slack; `TestSlackCLI` rows:
- `login` reads the token from stdin, writes the token file with mode 0600 after `auth.test` answers ok, and leaves no other file; `login` with `invalid_auth` refuses naming the OAuth page and `lym slack login`, and writes nothing; `login` with an empty standard input refuses, calls no `auth.test` on the `httptest` server and writes no token file.
- `login` and `setup` run from an orch's pane, from a non-orch strand's pane, outside every pane while a live pane process is unknown, and outside every pane while the hub's strands cannot be read each refuse and write nothing; off Linux, a `StrandName` naming a non-orch strand of the hub refuses `start`.
- `setup` with a non-member channel, and with `channel_not_found`, refuses naming `/invite`; `missing_scope` names the scopes; `user_not_found` names the member id; with no token it names `lym slack login`.
- `start` and `away` with no token or no setup refuse naming `lym slack login` or `lym slack setup`; `start` from a non-orch strand's pane, or outside every pane while a pane process is unknown, refuses and ensures nothing; `start` from an orch's pane ensures once.
- `away` calls the presence change only after the await returns, and refuses with the await's message, naming `lym slack away`, leaving `terminal`.
- `away`, `back` and `stop` from a non-orch strand's pane, or outside every pane while a pane process is unknown, refuse and leave the presence and the relay unchanged; from an orch's pane, or outside every pane with every pane known, they take effect.
- `stop` with presence `slack` sets `terminal` and calls the stop once; a stop error names `lym slack stop`.
- `back` sets `terminal`; `status` of a hub with no presence file prints `terminal`; `status` with presence `slack`, a live relay, a pause covering the prime orch, a setup file and a `LastSlackOK` prints each of them.
- `relay --fork` calls the intermediate spawn once, with the `--prime` name, and nothing else; `relay --after <pid>` calls the reparent wait with that pid before the relay run, and with a reparent wait that errors exits 1 and never runs the relay.
- With `preRun` installed over a location resolver that refuses every cwd, as `lymcwd.Resolve` refuses a hub directory, and a fake seam wiring, `relay --hub-path <temp hub> --prime <name> --after <pid>` run with the cwd at a temp hub directory that is no worktree reaches the fake relay run with that hub path and prime name, while `status` refuses with the resolver's error.

`cmd/lym/clitree_test.go` and `registration_test.go` then hold the module's tree form and mount, as D9's criterion asks; no list in them changes, and `newRoot`'s `Long` text, whose "Available modules" list `clitree_test.go` checks for every mounted module, gains `slack` beside `creel`.

**ImpactSummary:** One new CLI module mounted in `newRoot`, one sandbox exclusion entry, two signature-list entries and a doc page extension; no existing module's behavior changes.
