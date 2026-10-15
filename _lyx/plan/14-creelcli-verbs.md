# Card 14 — creelcli-verbs

**Create:**
- `internal/creelcli/doc.go`
- `internal/creelcli/cli.go`
- `internal/creelcli/spawn.go`
- `internal/creelcli/cli_test.go`
- `plan:internal/creelcli#Command` -> `func Command() *cobra.Command`
- `plan:internal/creelcli#RunCLI` -> `func RunCLI(out io.Writer, args []string) int`
- `plan:internal/creelcli#RunCLIIn` -> `func RunCLIIn(cwd string, out io.Writer, args []string) int`

**Edit:**
- `cmd/lyx#newRoot`
- `//tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Give creel its `lyx creel` subtree (`PATTERN-cli-cobra`), mounted under the root, and its daemon start.
Verbs, each with a one-line lower-case `Short` and an `audience` annotation: `post [<receiver>] <text>` (`--reply <id>`, `--wait-reply`, `--timeout <duration>`), `inject <receiver> <text>` (`--interrupt`, `--ttl <duration>`, `--wait`, `--timeout <duration>`), `interrupt <receiver>`, `forward <id> <receiver>`, `read` (`--from <sender>`, `--all`, `--wait`, `--timeout <duration>`), `inbox`, `log` (`--id`, `--from`, `--to`, `--since`), `hold <receiver> --for <duration>`, `release <receiver>`, and the hidden `daemon --hub-path <path>` (audience `internal`).
`post` takes two arguments, receiver and text, without `--reply`, and one, the text, with it, since a reply addresses the original message's sender (D12); any other count is refused naming both forms.
Every operator verb resolves the mode with `preflight.ResolveMode` on the command's cwd; standalone mode refuses with an error naming that creel needs a hub, and a refused resolve passes its error through.
In hub mode a verb opens the lyxtemp root with `lyxtemp.ProcessRoot()` (a failure passes through naming the root), builds `hubgeom.CreelGeometry(location.HubPath, root.Path())` and a `creelengine.Client` whose `Claim` carries `$LYX_STRAND_NAME`, the controlling terminal, the parent process's name and the cwd, and no program name; it sends the verb's request and prints the answer through `internal/output`.
The client's clock and spawn are an unexported package seam, the real clock and `spawn.go`'s spawn in production, which a test replaces.
`read` prints the messages with envelope and body, then acknowledges them on the same connection; with nothing unread it prints that and exits 0.
Exit codes: 0 on success and on `inject --wait` reporting injected; distinct non-zero codes, declared as constants, for an inject unconfirmed, an inject expired, a waiting verb's `--timeout` expiry (which prints that no outcome arrived), and a `post --wait-reply` answered returned-undelivered; every refusal exits 1.
`daemon` runs `creelengine.Run` from the hub directory with the geometry for `--hub-path` (an absent or relative value is refused), the hub delivery of card 13 with `claudeengine.NewFromConfig`, `proc.ParentPID`, `runtime.GOOS` and `selfexec.New().Replaced`; `creelengine.ErrDaemonRunning` ends it at once with a message and exit 0.
`spawn.go`: the client's `spawn` seam starts `<this binary> creel daemon --hub-path <hub>` detached with `proc.Detach`, `cmd.Dir` the hub and null stdio, and logs the spawn at Info (`PATTERN-spawn-observability`); under `go test` (`testing.Testing()`) it starts nothing and returns an error, by the same suppress argument `reedengine.SpawnWatchdog` takes.
Bound: any process of the operator's user that runs a verb outside a strand pane posts as EXTERNAL or, through the Go client only, as a program sender, each privileged under D9; the socket's 0700 lyxtemp root is the boundary.
`doc.go` is the CLI package header naming the verbs and pointing at `internal/creelengine` for the design.
`cmd/lyx#newRoot` mounts `creelcli.Command()`, and `creel` joins the `Available modules` list in its `Long`, which `cmd/lyx/clitree_test.go` checks against every mounted module.
`tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` gains a scenario tagged `**Covers:** creel`: post to a hand-added `lyx reed add --cmd claude` strand, watch the reminder typed at idle, have the agent run `lyx creel read`, inject a line and see it typed, and check `lyx creel log --id` shows the events with no body.
`TestCreelCLI` (`cli_test.go`) rows, each over the package seam with `creelfake`'s clock, so no row waits out real time:
- every verb refuses in a plain git repository (standalone mode) naming that creel needs a hub, and a subdirectory of a hub worktree passes `ResolveMode`'s refusal through;
- `daemon` refuses an absent and a relative `--hub-path`, and one started while another daemon on the hub holds the lock exits 0 with its message;
- a verb in a hub whose daemon is absent fails after the capped retries, naming the socket, since the spawn is suppressed under go test;
- `post` refuses one argument without `--reply` and two with it;
- with a daemon run in process on the hub's geometry over a `creelfake` delivery, and as EXTERNAL: `post` prints the id; `inject --wait` exits 0 on a send that lands, with the unconfirmed code on a send answering `ErrNotLanded`, and with the expired code when the fake clock passes its TTL while the receiver reads busy; `post --wait-reply` exits with the returned-undelivered code once its receiver leaves `creelfake`'s strands for two sweeps; `inject --wait --timeout 5s` exits with the timeout code once the fake clock passes five seconds;
- as a strand (a `creelfake` strand whose `PanePID` is `os.Getpid()`): `read` prints the message and acknowledges it, `read` with nothing unread prints that and exits 0.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/creelcli` by one.

**ImpactSummary:** Adds the creel subtree to the root command and its Long's module list, and a sandbox scenario; the cobra tree, registration and sandbox-coverage guards in cmd/lyx now see one more module.
