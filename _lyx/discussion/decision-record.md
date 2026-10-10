## Goal

The test suite costs a fraction of what it does today, measured per tier and per package before and after, and leaves nothing running behind it:

- `cmd/testtiming` measures every tier per package: wall time, CPU, peak memory, process count, git processes by subcommand split into fixture and code-under-test, and leftover processes.
- Tier 3 panes run a non-login shell on a hermetic tmux server, and tests that name their own sessions share one server per package.
- `lyx` is built once per gate run and reused by every package.
- Hub fixtures are copied from a per-binary template instead of cloned and wired per test; read-only tests share the template.
- `lyx`'s own lookups of repository geometry and refs run in-process, so every `lyx` command spawns fewer git processes.
- Fixture repositories never gc in the background, and a package fails when a test leaves a child running.
- A scan keeps tier 1 and tier 2 free of tmux, and a scan pins `PATTERN-shuttle-stop`.

## Scope

In:

- `cmd/testtiming`: a per-package resource mode, a gate slot, the before and after report under `docs/benchmarks/`.
- `internal/gitkit` and `internal/hubforge`: the fixture git marker, the hub template cache, the relocating hub copy, the shared read-only hub and its write check.
- `internal/testkit/tmuxkit`: the private temp directory, the hermetic pre-started server with a non-login `default-command`, the package server, the end-of-package socket and leftover-process check.
- `internal/testkit/lyxbin`: the prebuilt-binary variable, once-per-binary build, and running the built `lyx` under group kill.
- `internal/gatecli`: build `lyx` once for a tagged run and export it.
- `internal/lyxcwd`, `internal/gitrepo`, `internal/fabricengine`: in-process geometry and ref lookups replacing `rev-parse` spawns.
- `cmd/lyx` scans: tmux spawns barred from `integration`-only files, the `RemoveStrand` call-site scan.
- Every tier-2 test that starts reed moves its reed-starting half into a `tmux` file.
- Every hub-taking test is reclassified onto the shared template, a copy, or a fresh build.
- Docs: `PATTERN.md` and the pattern background files named under Constraints, the package docs of every package above, `docs/benchmarks/running-tests.md`, `cmd/lyx/testdata/test-budget.yaml`.

Out:

- The hub stores' gc at Remove and repeat repacks (`bolt-and-store`, running).
- Daemon wait intervals and the watchdog's idle exit (`daemon-wakeups`, running); this task tolerates the watchdog's exit schedule instead of changing it.
- Folding `lyx gate test`'s child lifetime into `internal/proc` (`proc-child-lifetime` note).
- Pane start mode in production reed: strand panes stay tmux's default login shell outside tests.
- Reducing `worktree`, `commit`, `add`, `push`, `clone` and `diff` spawns: tree-mutating and remote work stays with `gitexec` under `PATTERN-gitrepo-client-boundary`.
- Windows measurement: the resource columns are Linux-first with a documented fallback; no Windows run is part of this task.

## Decisions

### D1. `cmd/testtiming -resources` measures one package at a time

Decision: a `-resources` flag runs the tier's packages serially, each as its own `go test -json -count=1` child, and prints per package: wall, CPU, peak memory, process count, git processes by subcommand split fixture/code, leftover processes.
Inside a hub it takes a gate slot through `internal/gateslot` for the whole run and caps `-p` from the hub's limits; outside a hub it runs under the template cap, as `lyx gate test` does.
It prints the 1-minute load average at start and end so a report carries the machine's state.
Sources, Linux first: each child runs under a transient user scope (`systemd-run --user --scope`) and CPU and peak memory come from the scope's `cpu.stat` and `memory.peak`; when `systemd-run` is absent the child's rusage gives CPU and the largest single process's RSS, marked as such.
Process count is the delta of the fork counter in `/proc/stat` across the package, blank off Linux; it is system-wide, which is why the report says "quiet machine".
Leftover processes are those still referencing the package's private temp directory after the child exits (D8's rule, applied from outside).
`-full`, `-tags`, `-top` and `-redundancy` keep their meaning; `-resources` with `-redundancy` is refused.
Rationale: the goal names those columns, `go test -json` gives wall time only, and cgroup counters are the one source that sums a whole process tree.

### D2. Fixture git carries a marker; the harness splits the census on it

Decision: every git process spawned by `gitkit` or `hubforge` carries an environment variable naming it fixture git (one constant, declared in `gitkit`, set by both packages' spawn helpers).
The harness points `GIT_TRACE2_EVENT` at a per-package directory, sets `GIT_TRACE2_ENV_VARS` to that variable and `trace2.maxFiles` to unlimited, and counts one file per git process, by subcommand, split on whether the marker was present.
Git spawned by `lyx` (in-process `gitexec` or the built binary) carries no marker and counts as code under test.
Rationale: the board names this split as the measurement's first job, and trace2's env-var capture is the one mechanism that labels a process by its spawner without touching production code.

### D3. The report is measured twice in the same pair

Decision: the first card lands the harness and records the before-state in `docs/benchmarks/test-suite-weight.md`, one table per tier, with the load average and the git census; the last card re-measures into the same file as the after-state.
The file is a measurement report and may go stale; `running-tests.md` names the command that regenerates it.
Rationale: a before-state must be captured on the unchanged suite, and the operator can repeat the command on a quiet machine when the pair's run was loaded.

### D4. tmuxkit pre-starts every server it covers, hermetic and non-login

Decision: `Socket` and `KillOnCleanup` start the server for their key before returning, with `-f` naming a config file tmuxkit writes: `default-command` set to the operator's `$SHELL` run as a non-login shell (`sh` when `$SHELL` is unset), and `exit-empty off` so the session-less server waits for reed's `new-session`.
The operator's `~/.tmux.conf` is never read by a test server.
Reed is unchanged: production strand panes stay tmux's default login shell.
Bound: a test that starts a server without registering its key with tmuxkit runs the operator's config and login shells; D6's end-of-package check fails the package because the socket is unregistered.
Rationale: the pane census blamed the login profile per pane, and `default-command` is the one tmux lever that changes how a strand pane's shell starts without touching reed's split call.

### D5. One server per package for tests that name their own sessions

Decision: `tmuxkit.PackageServer(t, tmux)` returns one key per test binary, started once under D4's config and killed by `Main`'s sweep.
A test uses it when it names its own sessions, kills no server, and asserts nothing about the server's session set; a test that drives reed through a hub keeps the hub's `reedengine.ServerName` key, because production derives the key from the hub path.
The plan classifies each tmux-tier test by that criterion.
Rationale: the criterion is where isolation allows sharing; the hub-keyed server cannot be shared without changing production naming, which no goal asks for.

### D6. Tier 1 and tier 2 are tmux-free, by a runtime check and a scan

Decision, runtime: `tmuxkit.Main`, when the binary was built without the `tmux` or `llm` tag (a tag-constrained file inside tmuxkit sets the allowance), fails the package after `m.Run` when any socket file appeared under its directory, naming the sockets.
Under those tags it fails the package when a socket appeared whose key was not registered through `Socket`, `KillOnCleanup` or `PackageServer`.
Decision, static: the tier-purity scan, which bars tmuxkit spawns from untagged files, also bars them from files whose constraint lacks `tmux` and `llm`.
Consequence: every tier-2 test that starts reed today, such as `loomcli`'s resume-after-reconcile test, splits its reed-starting assertions into a `tmux` file; the reconcile assertions stay tier 2 behind a verb that does not arm.
Bound: a test that points `TMUX_TMPDIR` elsewhere escapes the runtime check, and a server started by a binary other than tmux is outside it.
Rationale: starting tmux has no static shape, so the socket directory tmuxkit already owns is the one place every server shows up.

### D7. `lyx` is built once per gate run and named by an environment variable

Decision: `lyxbin` reads a variable naming a prebuilt `lyx`; when set, every `Build*` returns it after checking it exists and is executable, and fails the test naming a missing file as stale.
When unset, `lyxbin` builds once per test binary into the private temp directory of D8, so a raw `go test` on one package builds once, not once per test.
`lyx gate test` with a non-empty `--tags` builds `./cmd/lyx` once into a per-invocation temp directory, inside the slot it holds, and exports the variable to `go test`; `cmd/testtiming` does the same for a tagged run.
The variable joins the names an agent's shell command may not spell, beside the slot and strand variables.
Bound: a forged value points every package at a binary from another tree; the agent deny closes it for spawned sessions, and an operator shell is outside it.
Rationale: four packages link tree-sitter through cgo on their own today; one build per run is the goal, and tier 1 never needs the binary by the tier-purity scan.

### D8. A private temp directory per test binary and a leftover-process check

Decision: `tmuxkit.Main` points `TMPDIR` (`TMP` and `TEMP` on Windows) at a private directory for the binary's life, so every `t.TempDir`, `os.MkdirTemp("")`, `lyxbin` build and child temp file lands there.
After `m.Run` it waits a bounded grace (a constant in tmuxkit, injectable for its own test), then on Linux scans `/proc` for processes whose cwd, exe or argv reference that directory, kills them, and fails the package naming pid and argv.
A `lyx reed watchdog` is killed without failing the package: reed's daemon idles out on its own schedule and `daemon-wakeups` owns that schedule.
Then the directory is removed.
Off Linux the scan is skipped and the directory is removed.
Bound: a child whose cwd, exe and argv do not mention the directory escapes; a package that does not call `tmuxkit.Main` (untagged-only packages) has no check, which is fine because tier 1 spawns nothing.
Rationale: this is the goal's "every test waits for the git and lyx children it starts", enforced without knowing how each test spawns, and it is where #472's lingering writer becomes a named failure instead of a flaky `TempDir` cleanup.

### D9. Fixture git never gcs in the background

Decision: `gitkit.HermeticGitEnv`'s neutral config adds `gc.autoDetach=false` beside its existing `gc.auto=0` and `maintenance.auto=false`; the template repositories keep their local keys.
The hermetic variables reach every `lyx` child a test runs through D10's runner, and reed's clean environment already passes them through.
Rationale: the neutral config already disables auto gc, so the remaining exposure is a child built with a fresh environment; `autoDetach=false` makes any gc that still runs foreground and waited.

### D10. Tests run `lyx` through `lyxbin.Run`

Decision: `lyxbin` gains a runner that runs the built `lyx` with a directory, a timeout and arguments, in its own process group through `internal/proc.ConfigureGroupKill`, kills the group when the timeout expires, waits for it, and returns combined output, exit code and error without calling `testing.TB` methods.
The per-package runners in `loomcli`, `reedcli`, `orchcli` and `cmd/lyx` move onto it.
`lyxbin`'s exemption widens from "go build of ./cmd/lyx" to "go build of ./cmd/lyx and running that binary under group kill", recorded in `PATTERN-testkit`.
Bound: a child that detaches into its own session escapes the group kill; D8 catches it.
Rationale: today a timed-out `lyx` leaves its git children running, and `PATTERN-testkit` forbids four copies of the same runner.

### D11. Hub fixtures are templates, copies and a shared read-only hub

Decision: `hubforge` builds, once per test binary and per shape, a template hub through `NewHub` plus `AddPair` for the shape's pairs; a shape is an anchor plus an ordered list of pair slugs, from a closed set declared in `hubforge`.
`CopyHub(tb, shape)` returns a relocated copy of the template: every regular file copied without following links, the template root path rewritten to the copy's in every text file (both separator spellings), every link recreated through `fslink` with its rewritten target, the bares copied and their URLs rewritten, and the copy verified to contain the template root nowhere before it is returned.
`SharedHub(tb, shape)` returns the template itself for a test that only reads it, and registers a cleanup that fingerprints the template's file tree in-process (paths, sizes, mtimes) and fails the test when the fingerprint changed since the template was built, naming the fixture; with `t.Parallel` the blame may land on a sharer, and the message says so.
`NewHub` stays for a shape outside the set.
A write is any fabric mutating verb, any git write, `SeedConfig`, `AddPair`, and any run of the `lyx` binary, since every `lyx` command's pre-run seeds and commits.
The plan classifies every hub-taking test: reads only, shared; writes, copy of the nearest shape; shape outside the set, fresh.
`gitkit.CopyRepo` is unchanged and stays pinned to `lyxcwd`.
Bound: the relocation rewrites text files only; a binary file carrying an absolute path would survive it, and the verification scan is what catches one.
Rationale: a hub cannot be copied byte for byte because its links and worktree pointers carry absolute paths, and a text rewrite plus relinking is the one relocation that needs no knowledge of which files carry them.

### D12. Geometry and ref lookups run in-process

Decision:

- `lyxcwd` finds the worktree root by reading the `.git` entry at cwd (a directory, or a `gitdir:` file whose target exists) and spawns no git on the happy path; a cwd with no usable `.git` entry is refused as today, with the same message.
- `gitrepo` reads a repository's geometry (git dir, common dir, toplevel) from its `.git` entry and `commondir` file once per handle and caches it for the handle's life; a `--git-path` derivation becomes a join on that geometry, per-worktree paths (`HEAD`, `MERGE_HEAD`, `index`) on the git dir and shared ones on the common dir.
- Ref reads go through go-git: branch existence, HEAD's branch or detached state, HEAD's tree, the upstream of a branch, a ref's SHA.
- `fabricengine`'s `rev-parse` spawns move onto those reads; a `rev-parse` survives only where the answer is git's own disambiguation (`--disambiguate`, `<commit>:<path>` existence).
- Every new read gains a row in the `gitoracle` parity test, the unborn branch and detached HEAD included.

Nothing else is cached: HEAD and refs change under a process.
`GIT_DIR` and `GIT_WORK_TREE` are not honoured by the in-process reads; a bare repository has no worktree and is refused as today.
This refines `PATTERN-gitrepo-client-boundary`: go-git owns local reads, and `gitrepo`'s dotgit geometry read is the one local read beside it.
Rationale: `rev-parse` was 38% of the census, nearly all geometry or ref lookups, and the goal is fewer spawns in every `lyx` command, not only in tests.

### D13. A scan pins `PATTERN-shuttle-stop`

Decision: a `cmd/lyx` scan over production Go files fails when `RemoveStrand(` is called outside `internal/reedengine` (the owner), `internal/reedcli` (reed's own verb), `internal/shuttleengine`, `internal/loomcli`, `internal/orchcli` and `internal/testkit/shuttlefake`, naming the file; the allowlist is written once in the scan and matches the entry's exemption list, and the entry gains "(test)".
Bound: the scan sees the call site, not whose strand it removes; `loomcli` and `orchcli` removing a shuttle run's strand is review discipline.
Rationale: the entry was found false with nothing to catch it; a call-site scan is what makes the next drift visible.

## Constraints

- `PATTERN-test-speed`: tier 2 has no tmux server; the tier table, gate table and "untagged tests spawn nothing" section record D6, D7 and D8.
- `PATTERN-test-isolation`: records D4 (hermetic config, non-login), D5, D6's runtime check, D8's `TMPDIR` and leftover check.
- `PATTERN-testkit`: `lyxbin`'s bound widens per D10; `tmuxkit`'s bound covers starting its servers with its own config and the `/proc` scan; no kit imports `hubforge`, so D11's fingerprint lives in `hubforge`.
- `PATTERN-hubforge-fixtures`: every hub fixture is still built through `fabriccli.CloneAndWire`; a copy is a relocation of such a hub, and the entry says so.
- `PATTERN-gate-slot`: the harness takes a slot, and `lyx gate test`'s build runs inside the slot it holds.
- `PATTERN-cwd-resolution`: `lyxcwd` keeps sole ownership of worktree-root resolution; D12 changes its implementation, not its boundary, and the background's "imports gitexec only" line is updated if the import goes.
- `PATTERN-gitrepo-client-boundary`: refined by D12 as stated there.
- `PATTERN-gogit-read-helper`: every new go-git read runs through `readGoGit`.
- `PATTERN-hub-store-gc`: untouched; D9 configures fixture repositories, not hub stores.
- `PATTERN-spawn-observability`: `lyx gate test`'s build logs its spawn; the harness and the kits are outside the rule.
- `PATTERN-no-denied-recovery`: the new variable is a denied name, so no refusal or stencil may spell it.
- `PATTERN-shuttle-stop`: pinned by D13.
- `PATTERN-test-economy` and the test budget: new behaviour is covered at the kits' public surface; rows for `tmuxkit`, `lyxbin`, `hubforge`, `gitkit`, `cmd/testtiming`, `cmd/lyx`, `lyxcwd`, `gitrepo` and `gatecli` are raised in the same commit as the tests that need it.
- Docs land in the same commit: package docs of every changed package, the pattern background files above, `running-tests.md`; `docs/overview.md` is unchanged unless the module table changes.
- CLAUDE.md's push rule is unchanged: both tiers through `lyx gate test`.

## Auto-mode assumptions

- The resource columns are Linux-first with cgroup counters; the operator measures on this Linux machine, and no Windows measurement is required.
- The marker variable, the prebuilt-binary variable and the grace constants are named by the plan, not here.
- The shape set starts with anchor `.` and `backend`, each with no pair and with one pair, and grows only when the plan's classification needs another.
- The watchdog allowance in D8 is permanent: `daemon-wakeups` owns the watchdog's exit schedule.
- `exit-empty off` keeps a pre-started session-less server alive for reed's `new-session`; the plan verifies against tmux before relying on it.
- The non-login `default-command` is the operator's `$SHELL`, rc files included, since reed's typed prelude and launch script expect that shell.
- The before-state is measured in the pair at the first card, loaded or not, and the report records the load average.

## Open risks

- Relocation completeness: a wired hub may carry an absolute path in a file the text rewrite does not reach; the verification scan fails the copy, and the plan's first copy test exercises a hub with a pair, a records sibling, portals and launchers.
- Shared-hub blame: a parallel sharer's write can be blamed on a reader; the message names the fixture so the writer is found by `-run`.
- Process count noise: the fork counter is system-wide; a loaded run inflates it, and the report says so.
- `go test` passes the environment through to test binaries, and reed strips only its own keys; a future stripper could drop the prebuilt-binary variable, which `lyxbin` reports as a missing file.
- Moving reed-starting tests to tier 3 moves their coverage from the batch gate to Publish; the tmux-tier impacted pass at the round gate with a Publish failure record is the backstop.
- go-git's view of an unborn branch and a detached HEAD must match `rev-parse --abbrev-ref HEAD` with the unborn fallback; the parity rows pin it.

## Acceptance criteria

- `go run ./cmd/testtiming -resources` prints, for every package of the tier, the columns of D1, and the report file holds a before and an after table per tier with the load average.
- A git spawned by `gitkit` or `hubforge` is counted as fixture git; one spawned by `gitexec` in the same binary is counted as code under test.
- A tmux server started through `Socket`, `KillOnCleanup` or `PackageServer` reads no `~/.tmux.conf`, and a strand pane reed splits on it runs a non-login shell.
- Two tests of one package share one `PackageServer` key and a hub-driven test keeps its hub key.
- A package built without `tmux` or `llm` whose test starts a tmux server fails after `m.Run`, naming the socket; a `tmux` build with an unregistered socket fails the same way.
- A file constrained to `integration` alone that references a tmuxkit spawn fails the tier-purity scan.
- With the variable set to an existing executable, `lyxbin.Build` returns it without building; with it set to a missing path, the test fails naming it stale; unset, two tests in one binary share one build.
- `lyx gate test --tags integration <pkg>` builds `lyx` once and the package's tests see the variable; `lyx gate test <pkg>` with no tags builds nothing.
- A spawned agent's command that names the variable is denied.
- A test that leaves a child whose argv names the package's temp directory fails the package after `m.Run`, naming pid and argv, and the child is gone; a leftover `lyx reed watchdog` is killed without a failure.
- `t.TempDir` of a tagged package lands under the private directory, and the directory is gone after `Main` returns.
- The hermetic config disables detached gc, and a `lyx` child run through `lyxbin.Run` sees the hermetic variables.
- `lyxbin.Run` on a command that outlives its timeout kills the command's group and returns a timeout error.
- `CopyHub` of a one-pair shape yields a hub where `fabric status` is clean, `git worktree list` resolves, the portals and launcher links resolve, and no file contains the template root.
- `SharedHub` fails the test that writes into it, naming the fixture.
- `lyxcwd.Resolve` on a worktree root, a linked worktree, and a `.git` file with a relative `gitdir:` spawns no git; a directory without `.git` is refused with today's message.
- `gitrepo`'s geometry read matches `rev-parse --git-dir`, `--git-common-dir` and `--show-toplevel` on a primary and a linked worktree, in the parity oracle.
- The ref reads match `rev-parse` on an unborn branch, a detached HEAD, a branch with and without upstream, in the parity oracle.
- `fabricengine`'s integration tier passes with `rev-parse` surviving only at the disambiguation sites, and the census shows the drop.
- A production file outside the allowlist calling `RemoveStrand(` fails the scan naming the file.
- `TestLoomResumeReconcilesTheHubConfigBeforeArming`'s reconcile half runs in tier 2 and starts no tmux; its arming half runs in a `tmux` file.

## Notes for the plan writer

- Card order: harness and report first (D1 to D3), then the kits (D4 to D10), then the fixture reclassification (D11), then the in-process lookups (D12), the scans (D6 static, D13), and the after-measurement last.
- `gitkit.doc.go` and `hubforge.doc.go` describe the existing template-and-copy model; D11 extends it to the hub.
- `internal/gatecli/run.go` builds the environment the child gets; the build step goes before `go test` there.
- `internal/reedengine/lifecycle.go` starts the server with `new-session` against a key; D4's pre-start must leave that call working.
- `cmd/lyx/tierpurity_test.go` and `tmuxkitspawn_test.go` are where D6's static half lives; `knownTierTags` is the tag vocabulary.
- `internal/reedcli/smoke_proctree_test.go` already recognises the watchdog daemon; D8 reuses that recognition.
- The census tool in `docs/benchmarks/fixture-copy.md` describes the trace2 directory method D2 reuses.
