# Lighter test suite: measured tiers, shared fixtures, no leftover jobs

Closes #472

The test suite spent most of its cost on login shells in tier 3 panes, repeated `lyx` builds, per-test hub clones and `git rev-parse` spawns.
It also left children running, which made `TestLoomResumeReconcilesTheHubConfigBeforeArming` flake on `t.TempDir` cleanup when a detached auto-gc wrote into a fixture's object store.
This change measures that cost per package, cuts it, and makes a package fail when a test leaves a child behind.

## What changes

- **Measurement.** `cmd/testtiming -resources` runs a tier's packages serially and prints wall, CPU, peak memory, process count, git processes by subcommand (split fixture and code under test) and leftover processes.
  It takes a gate slot, caps `-p`, and prints the load average at start and end.
  `docs/benchmarks/test-suite-weight.md` holds the before and after tables per tier; `docs/benchmarks/running-tests.md` names the regenerate command.
- **tmux tier.** `tmuxkit` pre-starts each server it covers with its own config: non-login `default-command` and `exit-empty off`; `~/.tmux.conf` is never read.
  `tmuxkit.PackageServer` gives tests that name their own sessions one server per package.
  `tmuxkit.Main` points `TMPDIR` at a private directory per test binary and, after `m.Run`, kills and fails on any process still referencing it (a leftover `lyx reed watchdog` is killed without failing).
  It also fails a package that creates a tmux socket without the `tmux` or `llm` tag, or one that was not registered.
  The kit's sweep reaps the pane process trees of every server it kills, as reed's own teardown does.
  `reedcli`'s watchdog recognizer and `/proc` probes now live in `tmuxkit`.
- **One `lyx` build.** `lyxbin` honours a prebuilt-binary variable and builds once per test binary otherwise.
  `lyx gate test --tags …` builds once inside its slot and exports the variable; an untagged run strips an inherited value.
  The variable is a denied name in spawned agents' shell commands.
  `lyxbin.Run` runs the built `lyx` under group kill, and the per-package runners in `loomcli`, `reedcli`, `orchcli` and `cmd/lyx` use it.
- **Hub fixtures.** `hubforge` builds a template hub per shape once per binary.
  `CopyHub` relocates a copy (text rewrite, relinking through `fslink`, a verification scan for the template root).
  `SharedHub` hands read-only tests the template and fails the test, then poisons the shape, when the template's fingerprint changes.
  Hub-taking tests are reclassified onto the shared template, a copy, or a fresh `NewHub`.
- **Fewer git spawns in `lyx` itself.** New leaf package `internal/dotgit` parses `.git`.
  `lyxcwd` finds the worktree root through it, resolving the start's symlinks first as `rev-parse` did, and spawns no git on the happy path.
  `gitrepo` reads geometry through `dotgit` and refs through go-git; `fabricengine`'s `rev-parse` spawns move onto those reads, with new `gitoracle` parity rows.
  `gitexec` drops `GIT_DIR` and `GIT_WORK_TREE` from git children, so an operator who set either to point `lyx` elsewhere has it ignored.
- **Fixture git.** Fixture git carries a marker variable for the census, and the hermetic config adds `gc.autoDetach=false`.
- **Scans.** The tier-purity scan bars tmuxkit spawns from files that are not `tmux` or `llm` tier, and bars `hubforge.CopyHub` and `hubforge.SharedHub` from untagged files.
  A new scan pins `PATTERN-shuttle-stop` by barring `.RemoveStrand` outside its allowlist.
  Tier-2 tests that start reed moved their reed-starting half into `tmux` files.
- **Docs.** `PATTERN.md` and the affected `pattern/` files, package `doc.go` files and `cmd/lyx/testdata/test-budget.yaml` are updated.

On the recorded runs, tier 1 CPU fell from 125.43s to 91.88s, tier 2 from 330.30s to 248.08s and tier 3 from 178.96s to 171.34s.
`rev-parse` spawns fell from 26989 to 5972 in tier 2 and from 1664 to 134 in tier 3.
Every after run started at a higher load average than its before run, so wall times (tier 2: 128.63s to 120.72s) are not a fair comparison until the commands under Regenerate in the report are repeated on a quiet machine.

## Design decisions

- `reedengine`'s stale-holder probe skips a holder whose `@lyx_test_server` option is on, which only tmuxkit's config sets; otherwise reed would reap the session-less hermetic server.
  An operator config that sets that option disables the reap for its own servers.
- Reed's pane start mode is unchanged.
  A server reed boots itself after a test's own down reads the operator config and starts login shells; tests that pin reed's own start or recovery run on that real path.
- `tmuxAllowed` is one `var` plus an `init` in a `tmux || llm` file, not two build-tagged consts.
- The fixture/code git split is approximate where hub builds overlap other tests, because the marker is process-wide during a build; compare before and after on per-subcommand totals.
- `rev-parse --git-path hooks` and the `--disambiguate` and `<commit>:<path>` forms still spawn, because they depend on config or git's own disambiguation.
- The existing watchdog recognizer copies in `loomcli` and `orchcli` stay; folding them onto `tmuxkit` is out of scope.
- Out of scope: hub-store gc, daemon wait intervals and the watchdog's idle exit, `gitexec` child lifetime, and Windows measurement.

## To check by hand

- Run `lyx gate test ./...` and `lyx gate test --tags integration ./...` on the merged tree; this description was written without a full run.
- Repeat the `docs/benchmarks/test-suite-weight.md` regenerate commands on a quiet machine for fair wall times.
- The socket-directory check in `tmuxkit.Main` fails packages that only probe tmux as a client; check that no such package remains without the tag or a registered key.
- A reed-booted server torn down inside a test re-boots on the operator config; check that no test depends on the hermetic config after that point.
- The prebuilt-binary variable's deny matches static text only, and a forged value affects ungated raw runs only; check that no verify path reads the variable from a forged source.
- Relocation completeness: a wired hub may carry an absolute path in a file the text rewrite does not reach; check `CopyHub`'s verification scan against a hub with a pair, a records sibling, portals and launchers.
- Some `fabricengine` and `fabriccli` tests stay on `NewHub` because a relocated copy leaves the `_board` worktree dirty; check that this is still the right call.
- `internal/lyxcwd`: the walk-up must match `git rev-parse --show-toplevel` for a `.git` directory that is not a repository, for a `.git` file whose target is not one, and for a gitfile whose `commondir` target is missing (`ErrNotAGitRepo`, never an enclosing repository).
- `rev-parse --git-path hooks` survives in `fabricengine` so `core.hooksPath` is honoured; check no other path assumes the common dir.
- `testtiming`'s prebuild of `lyx` is covered only by a stand-in-`go` test; check its `-p` cap on the build in a real gate slot.
