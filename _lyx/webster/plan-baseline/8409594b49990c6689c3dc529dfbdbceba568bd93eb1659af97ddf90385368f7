# Card 5 — hibernate-cli

**Create:**
- `internal/hibernatecli/doc.go`
- `internal/hibernatecli/cli.go`
- `internal/hibernatecli/deps.go`
- `internal/hibernatecli/spawn.go`
- `internal/hibernatecli/testmain_test.go`
- `internal/hibernatecli/cli_integration_test.go`
- `internal/lyxdirs#HibernateDirName`
- `internal/hibernatecli#Command`
- `internal/hibernatecli#requestCmd`
- `internal/hibernatecli#cancelCmd`
- `internal/hibernatecli#statusCmd`
- `internal/hibernatecli#watchCmd`
- `internal/hibernatecli#RunCLI`
- `internal/hibernatecli#RunCLIIn`
- `internal/hibernatecli#requestFileName`
- `internal/hibernatecli#watchLogFileName`
- `internal/hibernatecli#hibernateDir`
- `internal/hibernatecli#hubDeps`
- `internal/hibernatecli#landLockHeld`
- `internal/hibernatecli#readRunStatus`
- `internal/hibernatecli#spawnDaemon`
- `internal/hibernatecli#runHibernate`
- `internal/hibernatecli#killDaemon`
- `internal/hibernatecli#TestHibernateVerbs_WireTheHub`

**Edit:**
- `cmd/lyx#newRoot`
- `cmd/lyx#excludedModules`
- `cmd/lyx/seamsignature_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/tendercli#HubDeps`
- `internal/hibernateengine#Submit`
- `internal/hibernateengine#SubmitRequest`
- `internal/hibernateengine#Cancel`
- `internal/hibernateengine#Status`
- `internal/hibernateengine#Watch`
- `internal/hibernateengine#Deps`
- `internal/hibernateengine#RunStatus`
- `internal/hibernateengine#PauseCaller`
- `internal/orchcli#NotifyPrime`
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`
- `internal/shedrun#List`
- `internal/shedrun#LandLock`
- `internal/parentlock#ProcessAlive`
- `internal/pauserecord#MonotonicAwake`

**Intent:**
Add the `lyx hibernate` module: a cobra subtree under the root in `cmd/lyx/main.go` (`PATTERN-cli-cobra`) with `request`, `cancel` and `status`, each annotated `clihelp.AudienceOperator` so the operator command index the orch's role file carries names them (Decision 11), and the hidden `watch` daemon verb, annotated `clihelp.AudienceInternal`.
It is hub-only and not standalone-capable: every verb resolves the hub with `lyxcwd.CwdFrom` and `lyxcwd.Resolve` from any worktree, as tendercli does, and `internal/cliwire` does not serve it.
Every verb prints the engine's result in the standard envelope under `request`, `cancel`, `status` or `watch`, and a refusal through `output.Err` with the engine's message, which names its way forward.

**Verbs.**
`request [--after <slug>] [--after-all]` calls `hibernateengine.Submit` and returns once the daemon is spawned.
`cancel` calls `Cancel`.
`status` calls `Status` and writes nothing; it builds no tender wiring.
`watch` calls `Watch`; it is what `request` spawns, and run by hand with no request, or beside a live daemon, it returns at once doing nothing.
Each Long says what the verb does, the triggers and the caller rule in a few lines, with an example, under the 2000-byte cap.

**Wiring (`deps.go`).**
`hibernateDir` is `filepath.Join(fabricengine.HubScratchDir(location.HubPath), lyxdirs.HibernateDirName)`, the hub's board-level `.lyx` beside the pause directory (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`); the request file is `requestFileName` and the daemon's log `watchLogFileName` in it.
`lyxdirs.HibernateDirName` is declared beside `PauseDirName` in `internal/lyxdirs/dirs.go`, with its doc comment.
`hubDeps` resolves the prime through `fabricengine.PrimeName` and `lyxcwd.ResolveWorktree(fabricengine.WorktreePath(location, primeName))`, as `orchcli.NotifyPrime` does, and fills `hibernateengine.Deps`:
the caller from `os.Getenv(agentname.StrandNameEnv)`; `runtime.GOOS`; `exec.LookPath`; `Spawn` as `spawnDaemon(prime.AnchorPath(), <log path>)`; `os.Getpid()` and its `proc.StartTime`; `parentlock.ProcessAlive`; `killDaemon`; `RunStatus` as `readRunStatus`; `RunSlugs` as `shedrun.List(prime)`; `LandLockHeld` as `landLockHeld(shedrun.LandLock(prime))`; `Pause`, `TenderStatus` and `Lift` each building `tendercli.HubDeps(location, hibernateengine.PauseCaller)` at the call and running `tenderengine.Pause`, `Status` or `Lift` over it, so each reading sees the hub's current pairs and the daemon's pause entry carries the caller label `hibernate`; `Hibernate` as `runHibernate`; `Notify` as `orchcli.NotifyPrime(location, line)`, the orch notice queue (`PATTERN-orch-pane-single-writer`); `time.Now`, `time.Sleep`, and one `pauserecord.MonotonicAwake()` reading per process.
`readRunStatus` reads the prime's batten status through `shedrun.StatusFile(prime, slug)` and `shedrun.StatusLock(prime, slug)` with `state.ReadJSONNoMkdir[shedengine.Status]`, creating the lock's directory first as tendercli's `Running` does, and reports not found when the status file does not exist; the paths come from their owning declaration (`PATTERN-shed-run-directory`).
`landLockHeld` reads not held when the lock's directory does not exist, and otherwise takes `lock.TryAcquireWriteLock` once: acquired is released at once and reads not held, contended reads held.
Bound: the probe holds the land lock for one acquire and release, so a landing that tries it in that instant polls again a second later, as a contended lock already makes it.

**Spawns (`spawn.go`, `PATTERN-spawn-observability`).**
`spawnDaemon` starts `<os.Executable()> hibernate watch` detached through `proc.Detach`, from the prime's anchor, with its output appended to the watch log, and logs the spawn at Info with the pid and log; it reads the child's start time with `proc.StartTime`, reaps the child in a goroutine, and under `testing.Testing()` returns the error `hibernate: the daemon is not spawned under go test` with nothing started, so no test re-executes the test binary and no request is written with a pid nobody runs.
`runHibernate` runs `systemctl hibernate`, logging the spawn and its exit at Info, and returns an error carrying the exit and the output's last line on a non-zero exit.
`killDaemon` kills the pid through `proc.KillPID` and logs it at Info.

**Root, guards and docs.**
`cmd/lyx#newRoot` adds `hibernatecli.Command()` to the root and `hibernate` to the Available modules line of its Long, which `TestCLITree_EveryCommand` checks.
`cmd/lyx/seamsignature_test.go` adds `hibernatecli.RunCLI` and `hibernatecli.RunCLIIn` to its two pin slices.
`cmd/lyx#excludedModules` gains `hibernate` with the reason `its daemon hibernates the host machine, which no sandbox scenario may do` (`PATTERN-sandbox-coverage`).
`docs/overview.md` gains a `hibernate` module bullet after `tender`, naming both packages, the four verbs, the triggers, the daemon and its bounds in a few lines, and the two packages in its source tree.
`contracts/specs/refusal-spec.md` gains a `## hibernate` section after `## pause`, in that section's shape, with one row per refusal card 3 and card 4 prescribe: an agent caller on `request`, `cancel` or `watch`; a non-Linux machine; `systemctl` missing from `PATH`; `--after` beside `--after-all`; an `--after` slug with no batten status; a pending request with a live daemon; and `cancel` while the machine is going into hibernation; each Way forward cell quotes the message's own clause.
The spec and the docs never spell the sleep verbs after `systemctl` in one command, since card 6 adds those to the denied forms the no-denied-recovery scan reads in `contracts/specs`.
`internal/hibernatecli/doc.go` states the module: the verbs, the hub resolution, the wiring above, the spawn rules and the probes' bounds; the engine's sequence stays in `internal/hibernateengine`'s doc.

**Test.**
`testmain_test.go` runs `gitkit.HermeticGitEnv()` and `tmuxkit.Main`, as tendercli's does.
`TestHibernateVerbs_WireTheHub` (build constraint `integration && linux`, since `cancel` and `watch` refuse every other platform; no `t.Parallel`, since it sets the strand variable) builds one hubforge hub with one pair and runs, in order:
- `status` from the pair's code worktree as an external caller: ok, `request` null, and no `hibernate` directory under the hub's board-level `.lyx`;
- `cancel` from the prime as an external caller: ok with `cancelled` false, and still no request file;
- `request` with the strand variable set to an agent of the pair (`agentname.Format`): refused, naming the parent session, with no request file;
- `watch` from the prime as an external caller with no request: ok, and no request file;
- `clihelp.RenderIndex` over a root holding `Command()`, for the operator audience: it lists `request`, `cancel` and `status` and not `watch`.
`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/hibernatecli: 1`.

**ImpactSummary:** a new module wired into the root with one budget row; the root's Long, the seam pins, the sandbox allowlist, the overview and the refusal spec each gain the module's entry, and no existing behavior changes.

**Commit:** 5: hibernate: the lyx hibernate CLI module and its hub wiring
