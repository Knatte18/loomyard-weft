# Card 10 — orch-cli-skeleton

**Create:**
- `internal/orchcli/cli.go`
- `internal/orchcli/paths.go`
- `internal/orchcli/refusal.go`
- `internal/orchcli/status.go`
- `internal/orchcli/cycle.go`
- `internal/orchcli/stop.go`
- `internal/orchcli/cli_test.go`
- `internal/orchcli/testmain_test.go`
- `internal/orchcli/testmain_integration_test.go`
- `internal/orchcli/prime_integration_test.go`

**Edit:**
- `cmd/lyx/main.go`
- `cmd/lyx/helptree_test.go`
- `cmd/lyx/seamsignature_test.go`
- `cmd/lyx/sandbox_coverage_test.go`

**Uses:**
- `internal/battencli/refusal.go`
- `internal/shuttlecli/cli.go`
- `internal/orchengine/state.go`
- `internal/orchengine/watchloop.go`
- `internal/orchengine/config.go`
- `internal/shuttleengine/session.go`

**Intent:**
Stand up `lyx orch` as a CLI module (D1) with its prime-only resolution (D2) and the three verbs that need no session spawn: `status`, `cycle` and `stop` (D14).

`cli.go` follows the CLI/Cobra Invariant and `shuttlecli`'s shape.
It exposes `Command()`, `RunCLI` and `RunCLIIn`, the group command uses `clihelp.GroupRunE`, and every command carries a `Short` and checks `clihelp.ShouldAbort` first.
The group's `PersistentPreRunE` skips the bare group and otherwise resolves once into an `orchCLI` receiver:

1. `lyxcwd.CwdFrom`, then `lyxcwd.Resolve`.
2. The prime check: `fabricengine.RequireDrivableWorktree(location)`, then `fabricengine.PrimeName(location)` compared through `refusal.go`'s pure `refuseNonPrime(worktreeName, primeName string, primeNameErr error) error`.
   This is `battencli.RefuseUnlessPrime`'s shape with `orch:` wording ("lyx orch runs from the hub's prime worktree only"), duplicated rather than imported because a `<module>cli` importing another `<module>cli` would couple two cobra seams.
   Both refusals land on the envelope before anything is created.
3. `orchengine.LoadConfig`, `shuttleengine.LoadConfig` and `reedengine.LoadConfig`, all anchored at `location.AnchorPath()`.
   The receiver's copy of the shuttle config gets `ClaudeDenyAgentTool` set false (see the `orch-agent-tool` decision).
4. `reedengine.New(reedCfg, hubgeom.ReedGeometry(location))` and `shuttleengine.NewRunner(reed, claudeengine.New(), …)`, exactly as `shuttlecli` builds them.
5. `orchPaths(location)` from `paths.go`, and `fabricengine.StencilsDir(location.HubPath)` as the stencils directory.

`paths.go`'s `orchPaths(location *lyxcwd.Location) orchengine.Paths` joins the `orch-dir-layout` decision's names onto `filepath.Join(location.AnchorPath(), lyxdirs.DotLyxDirName, "orch")`.
It is the module's one scratch accessor, per the Durable-vs-Ephemeral State and Lyxdirs Single-Declarer invariants.

The verbs read reed through a receiver seam, `strandOps` (`Strands() ([]reedengine.StrandStatus, error)`, `RemoveStrand(guid string) error`), adapted from the reed engine the way `loomcli`'s `reedDriverPaneProbe` is, so untagged tests use a fake:

- `status` emits one envelope with `strand`, `strand_live`, `watcher_live` (`orchengine.WatcherLive`), `context_tokens` (null while unknown), `threshold_tokens`, `phase`, `cycle_count`, `last_handoff`, `last_abort_reason` and `watcher_exit`.
- `cycle` calls `orchengine.RequestCycle` and reports `requested: true` plus `watcher_live`, so an operator sees at once when no watcher will pick the request up.
- `stop` removes the recorded strand when reed tracks it and reports `removed` and `strand`.
  An untracked or unrecorded strand reports `removed: false` rather than an error.
  The watcher then exits on its own (D14), and the next `start` resets any persisted phase (D10).

`cli_test.go` builds receivers by hand over a temporary `Paths` and a fake `strandOps`, and covers:
`refuseNonPrime`'s three outcomes;
`status` reporting every field for a populated state and null tokens for an unknown reading;
`cycle` writing the request;
`stop` removing a tracked strand and reporting `removed: false` for an untracked one;
and bare `lyx orch` listing its subcommands.

`prime_integration_test.go` (`integration` tag) builds a hub with `hubforge.NewHub` and a pair with `hubforge.AddPair`.
It runs `status` through `RunCLIIn` from the task worktree and from the weft prime (`PrimeWeft`), and asserts exit 1, an envelope naming the prime-only rule, and no `.lyx/orch` directory created.
The two testmain files wire `gitkit.HermeticGitEnv()` under `!integration` and `integration` tags, as `internal/battencli`'s do.

Register the module: add `orchcli.Command()` to `cmd/lyx/main.go`'s `root.AddCommand` list and name `orch` in the root command's `Long` (`longlist_test.go` derives its set from the tree).
Add `orch` to `helptree_test.go`'s root module list plus a verb-module row with `status`, `cycle` and `stop`.
Add `orchcli.RunCLI` and `orchcli.RunCLIIn` to `seamsignature_test.go`'s slices.
Add `orch` to `sandbox_coverage_test.go`'s `excludedModules`, with the reason that it spawns a long-lived interactive Claude session and a detached watcher in the hub prime and is exercised by `internal/orchcli`'s smoke test instead.

**ImpactSummary:** Registers one new root command; the four edited `cmd/lyx` files only gain entries in pinned lists and the root `Long`.
