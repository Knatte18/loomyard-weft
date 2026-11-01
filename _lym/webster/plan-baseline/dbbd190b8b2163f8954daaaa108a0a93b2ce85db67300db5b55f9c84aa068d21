# Card 11 — gate-capped-front

**Create:**
- `//internal/gatecli/detachfront.go`
- `plan:internal/gatecli#detachSeams` -> `type detachSeams struct`
- `plan:internal/gatecli#defaultDetachSeams` -> `func defaultDetachSeams() detachSeams`
- `plan:internal/gatecli#runnerSpawn` -> `type runnerSpawn struct`
- `plan:internal/gatecli#launchRunner` -> `func launchRunner(spawn runnerSpawn) (int, error)`
- `plan:internal/gatecli#frontCall` -> `type frontCall struct`
- `plan:internal/gatecli#frontDecision` -> `func frontDecision(dir string, wait bool, seams detachSeams) (call frontCall, capped bool)`
- `plan:internal/gatecli#detachKeyDir` -> `func detachKeyDir(anchor, verb, runDir string, words []string) string`
- `plan:internal/gatecli#runFront` -> `func runFront(ctx context.Context, out, stderr io.Writer, call frontCall, seams detachSeams) int`
- `plan:internal/gatecli#answerRunGoing` -> `func answerRunGoing(out io.Writer, pool *gateslot.Pool, phase runPhase, waited time.Duration) int`
- `plan:internal/gatecli#helperCapEnv` -> `const helperCapEnv = "GATECLI_TEST_CAP"`
- `plan:internal/gatecli#helperLockMarginEnv` -> `const helperLockMarginEnv = "GATECLI_TEST_LOCK_MARGIN"`
- `plan:internal/gatecli#helperClaimBoundEnv` -> `const helperClaimBoundEnv = "GATECLI_TEST_CLAIM_BOUND"`
- `plan:internal/gatecli#helperGraceEnv` -> `const helperGraceEnv = "GATECLI_TEST_GRACE"`
- `plan:internal/gatecli#helperLaunchEnv` -> `const helperLaunchEnv = "GATECLI_TEST_LAUNCH"`
- `plan:internal/gatecli#helperLaunch` -> `func helperLaunch(spawn runnerSpawn) (int, error)`
- `plan:internal/gatecli#helperRoleRunGate` -> `const helperRoleRunGate = "rungate"`
- `plan:internal/gatecli#runGateRoleHelper` -> `func runGateRoleHelper() int`
- `plan:internal/gatecli#joinable` -> `func joinable(run joinState, now time.Time, live func(int) bool, reading func(int) (string, bool)) bool`
- `plan:internal/gatecli#joinState` -> `type joinState struct`
- `plan:internal/gatecli#TestJoinable` -> `func TestJoinable(t *testing.T)`
- `plan:internal/gatecli#TestGateTest_DetachedRun` -> `func TestGateTest_DetachedRun(t *testing.T)`

**Edit:**
- `internal/gatecli#newCommand` -> `func newCommand(goBinary string, samplePeriod, preTierCeiling, timeoutUnit, shortageRetryDelay time.Duration, seams detachSeams) *cobra.Command`
- `internal/gatecli#Command`
- `internal/gatecli#testRequest`
- `internal/gatecli#acquireSlot`
- `internal/gatecli#RunGate`
- `internal/gatecli#runGateHelper`
- `internal/gatecli#init`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#SlotBusyExit`
- `internal/gatecli#runTest`
- `internal/gatecli#haltExitCode`
- `//internal/gatecli/doc.go`
- `//internal/gateslot/doc.go`
- `//docs/overview.md`
- `//docs/dotnet-hub.md`

**Delete:**
- `internal/gatecli#answerSlotBusy`
- `internal/gatecli#helperSlotWaitEnv`

**Extends:**
- `cmd/lym#unboundedWaitFiles`
- `//pattern/PATTERN-daemon-wakeups.md`
- `PATTERN.md`

**Uses:**
- `plan:internal/gatecli#detachRequest`
- `plan:internal/gatecli#detachExit`
- `plan:internal/gatecli#runPhase`
- `plan:internal/gatecli#detachDirName`
- `plan:internal/gatecli#detachRun.writeLease`
- `plan:internal/gatecli#detachRun.expireLease`
- `plan:internal/gatecli#detachRun.leaseHolds`
- `plan:internal/gatecli#detachRun.claim`
- `plan:internal/gatecli#startReading`
- `plan:internal/runproc#RoleGate`
- `internal/shuttleengines#KeepWarm`
- `internal/shuttleengines#KeepWarmAnswer`
- `internal/runproc#Register`
- `internal/runproc#ChildRecord`
- `internal/fabricengine#RunProcDir`
- `internal/verifytree#DirtyPaths`
- `internal/gatecli#headIdentity`
- `internal/gatecli#resolveHub`
- `internal/gatecli#originSlug`
- `internal/gatecli#catchTerminatingSignals`
- `internal/gatecli#startGate`
- `internal/gatecli#eventually`
- `internal/gatecli#strandEnvEntry`
- `internal/gateslot#InheritEnv`
- `internal/hubgeom#GateSlots`
- `internal/lock#TryAcquireWriteLock`
- `internal/lock#AcquireWriteLock`
- `plan:internal/gatecli#helperDelayEnv`
- `internal/runproc#EndSparing`
- `internal/state#WriteJSON`

**Intent:** D1's front, D2's choice and D3's spawn and registration for `lym gate test`, in the new file `internal/gatecli/detachfront.go`; card 12 adds the long wait and card 13 card-verify.

**Seams.** `detachSeams` holds `keepWarm func(configengine.Dirs, string) shuttleengines.KeepWarmAnswer`, `launch func(runnerSpawn) (int, error)`, `claimBound` (30s), `lockMargin` (30s) and `grace` (120s); `defaultDetachSeams` fills them with `shuttleengines.KeepWarm`, `launchRunner` and those values, and `Command` passes it.
`newCommand` takes the seams in place of `slotWait`, hands them to the test verb and to `runnerCmd`'s gate tree; `testRequest` loses `slotWait`, and `RunGate` stops setting it.

**Choice (D2).** The test verb's RunE, after `flagShapeRefusal` passes, asks `frontDecision(dir, wait, seams)`: it answers not capped when `--wait` is given, when `agentname.StrandNameEnv` is empty, when the directory resolves to no worktree, when it runs inside an inherited slot (`gateslot.InheritEnv` naming a held slot of the hub's pool), and when the resolver's answer has a zero `Cap`; a resolver fallback is logged at Warn once per call and still caps at its `DefaultAwaitCap`.
Not capped, the call runs `runTest` in-process as today, and `acquireSlot` now waits for its slots without a cap: an operator call blocks until its slots free, the change D2 states, and `answerSlotBusy` goes with the cap.
Bound: an in-process call now waits for its slots without a cap, ended only when a holder releases its slots, a holder being bounded as before this card: a gate tier or the build step by its `timeout_min`, a slot-held card-verify line by `cardverify`'s per-line timeout, and any other slot taker of the pool, such as a drift step run or a plan-glyph type load, only by its own run, which this card neither changes nor bounds; or the wait ends by the caller's own interrupt, which cancels the wait and exits 128 plus the signal number as today.
An in-process call therefore never answers 75 of its own: `haltExitCode` loses its busy-hub case and that sentence of its comment, `SlotBusyExit`'s comment says it is the capped front's answer that the run goes on, `runTest`'s comment drops the slot-wait clause, and the doc's Exit codes paragraph says 75 is a capped call's `done: false` answer.
Bound: the strand is read from the caller's environment, so a forged or absent strand chooses only how long a call blocks, never what runs or its verdict.
Bound: the inherited-slot branch is reached only by a hand-set slot variable beside a strand, since every lym path that holds a slot runs its child without the strand (the tier environment, `verifytree`, `cardverify` from card 5 and the runner below); it stays because D2 names it, and the standing agent deny on the variable keeps an agent from reaching it.
`RunGate`, a nested call under a verify or card-verify line (which run without the strand, card 5) and every Go site's `--wait` call reach no front.

**Front (D1).** `runFront` counts its cap from its own start, so its deadline is that start plus the cap, and catches terminating signals, which end the front alone with 128 plus the signal number while the runner goes on.
It forms the key directory `detachKeyDir(anchor, verb, runDir, words)`, a hash of the verb, the resolved run directory and, for `lym gate test`, the words after the verb as given, under `gateslot.WaitDir(anchor)/detach`.
It takes the key directory's lock, the file `lock` in the key directory through `lock.TryAcquireWriteLock`, trying at most once a second only while at least `lockMargin` of its cap remains; past that it answers as at a passed cap with the phase `starting`.
Under the lock it reads the current run id into a `joinState` (the lease, whether `delivered` exists, `tree.json`, the current HEAD tree and sorted dirty path set from `headIdentity` and `verifytree.DirtyPaths`, the exit file and `process.json`) and asks `joinable(state, now, proc.IsAlive, <startReading for this platform>)`.
`joinable` answers true only when the lease holds, `delivered` is absent, the current tree and dirty path set equal `tree.json`'s, and the run has an exit file without the abandoned mark or a `process.json` whose pid is live with a start reading, taken through the told reader, equal to the recorded one (so a Windows front joins a live runner and supersedes a reused pid through `proc.CreationTime`).
Bound: the plan lives under `_lym`, in the weft worktree, which neither `headIdentity` nor `verifytree.DirtyPaths` reads, so a `--card` call or a card-verify call (card 13) made after an edit of the plan, with the same words and an unchanged warp, joins the run of the plan as it was while its lease holds; D1 and D6 fix the join's operands and key, so the plan is not added to them, and the edit costs at most the earlier plan's verdict for the remaining lease, after which the same command starts afresh.
Joining, it writes its deadline plus `grace` through `writeLease`, which keeps the later lease.
Otherwise it expires the current run's lease, so a live runner ends at its next read, makes a new run id and subdirectory, writes `request.json`, `tree.json` and the lease (deadline plus grace) before spawning, points the key directory at the new id, and calls `seams.launch`.
The key directory names its current run id in the JSON record `current.json`, written through `state.WriteJSON(<key dir>/current.json, <key dir>/current.json.lock, ...)` (PATTERN-tolerant-records, since fronts of different `lym` binaries rewrite it); its lock path is its own, never the key directory's `lock`, which the front already holds.
`launchRunner` refuses under `testing.Testing()` (tests never re-exec `os.Executable()`), otherwise starts `<os.Executable()> gate run-detached <run dir>` detached (`proc.Detach`) in the run directory, with `runner.log` as its stdout and stderr and the environment less `agentname.StrandNameEnv`, logs the spawn at Info (PATTERN-spawn-observability) and returns the pid.
The front registers the pid with `runproc.Register(fabricengine.RunProcDir(hub), slug, runproc.ChildRecord(pid, runproc.RoleGate, argv))` when the call is in a hub and `originSlug` names a slug; in the prime and outside every hub it registers nothing (D3's bound).
It holds the lock until `process.json` exists, checking once a second for at most `claimBound`; when the runner has ended or not claimed by then, the front creates `process.json` itself through `claim` with its own pid: that create succeeding is the start failure, on which it expires the lease, copies `runner.log` to its stderr and exits 1 with an error envelope whose way forward is to run the same command again, and, when the start fails again, to run it with `--wait`, which runs the tiers in-process and needs no runner; that create failing means the runner claimed in time, and the front goes on.
A `seams.launch` that returns an error, with no pid, takes the same start-failure path at once: the front creates `process.json` through `claim`, expires the lease, writes the launch error to its stderr and exits 1 with the same envelope, so a later call never joins the dead run.
It then waits with `shuttleengine.AwaitAny` on `exit.json` for what remains of its cap.
An exit file with the abandoned mark answers as at a passed cap, replaying nothing.
An exit file without it: the front copies `stdout` to its stdout, then `stderr` to its stderr, creates `delivered` and exits with the recorded code, except that a recorded 75 exits 1 after the stderr line `gate test: the run exited 75, a test's own code here and no busy hub, so this call exits 1`.
A passed cap calls `answerRunGoing`: an ok envelope with `done: false`, the phase from `phase.json` (`starting` when there is none), and while the state is `slots`, `slot_free: false` and each holder's worktree, site and tier from the pool, with the message to run the same command again in a turn that does nothing else, so the prompt cache stays warm between calls; it exits `SlotBusyExit`.
Bound: the join compares the tree and the dirty path set, not the dirty files' content, so an edit to an already-dirty file between two calls with the same words gets the earlier content's run; a second front with the same words after another took the result starts a fresh run.

**Texts.** The test verb's help says an agent-strand call without `--wait` runs in a detached runner it starts or joins, answers ok with `done: false` and exit 75 when its keep-warm cap passes, and that the same command again prints the run's result once it ended; `--wait`'s usage says it blocks to the end in-process.
The doc's Slots section loses the `DefaultAwaitCap` cap and gains a `# Capped call` section with the choice, the key, the join rule, the lease, the start failure, the answers and the bounds above; `docs/overview.md`'s gate paragraph says the same in one sentence.
The gateslot doc's sentence on the retired `cli_wait_sec` says the slot wait of an in-process call has no cap, and a capped call's front is bounded by the caller role's keep-warm cap.
`docs/dotnet-hub.md`'s direct-call sentence says the direct call passes `--wait` so it runs its drain in-process to its end, dropping the reason about the cap.
`internal/gatecli/detachfront.go` joins `unboundedWaitFiles`, and the background file's list gains the front's lock, claim and exit waits, each once a second and bounded by the cap.
`PATTERN.md`'s `## Processes and waits` section gains this entry line, after `PATTERN-daemon-wakeups`:

```
- `PATTERN-keep-warm-cap` — Bounding a wait an agent blocks on: the cap from `shuttleengines`' one keep-warm resolver for the caller's role, never a constant bar its fallback; a long gate run goes on in a detached runner the same command joins.
```

**Tests.** The gate helper reads the new variables: `helperCapEnv` makes `keepWarm` answer that cap (with `LongCap` equal to it), `0` answering a zero cap and `fallback` answering `DefaultAwaitCap` with a non-nil `Fallback`, as the real resolver does over an unloadable claude.yaml, and unset uses the real resolver; `helperLockMarginEnv`, `helperClaimBoundEnv` and `helperGraceEnv` replace the bounds; `helperLaunchEnv` picks `helperLaunch`, which re-runs the test binary in the gate helper role over `run-detached <run dir>` detached with `runner.log` as its output, `none` writing the fixed line `gate helper: the runner was never started` to the run's `runner.log` and returning the pid of an already-ended process so the runner never claims, `late` starting it two seconds late, and `error` returning an error and no pid.
`init` registers the new helper role `helperRoleRunGate`, whose `runGateRoleHelper` calls `RunGate` over `helperCwdEnv` with the gate key in its first argument, in a process started with a parsing strand and `helperCapEnv`, so a test drives the in-process entry under a strand.
`TestJoinable`, tier0, drives `joinable` over built `joinState` values with injected liveness and readers: a live runner whose reading equals the recorded one joins; the Windows reused-pid row drives `joinable` with a reader built from `startReading("windows", ...)` over injected creation times, so one row pins that the reading is the creation time in Unix nanoseconds, that a failing `CreationTime` gives no join, and that the same pid with a different creation time does not join; a dead pid does not; an exit file without the mark joins with no `process.json` read; one with the mark does not; a passed lease, a present `delivered`, a changed HEAD tree with the same dirty set, and a changed dirty set with the same tree each do not.
`TestGateTest_RefusesBeforeResolvingTheHub` loses its subtest "help names the shared keep-cache-warm cap as the slot wait", since the help no longer names a slot-wait cap, and `cli_test.go` loses its then unused `internal/shuttleengine` import and its header's clause on the slot wait the help names.
`TestGateTest_Scenario`: the row "every slot held past the wait cap answers slot_free false naming the holders and exits SlotBusyExit" is removed, its case moving to the new test; "a tier with slots all needs every slot and holds them" first shows that a call with no strand and `helperCapEnv=50ms`, one slot held, still waits after 500ms, so a no-strand call that took the 50ms cap would have answered 75, and is stopped; `helperSlotWaitEnv` leaves the `--wait` row; the rows that start the gate with `strandEnvEntry` add `helperCapEnv=0`, so they keep pinning the in-process tier environment and reach D2's zero-cap branch, and "an inherited held slot runs without acquiring" keeps the strand with the real cap, D2's inherited branch.
`TestGateTest_DetachedRun`, tier1, its own hubforge hub, every capped row starting the gate with a parsing strand and `helperCapEnv`:
- a tier that outlasts a 1s cap exits 75 with `done: false` and phase `running tier0` while the stand-in goes on; the same command after the run ended prints the summary line and exits with the stand-in's 7; a third call starts a fresh run, the stand-in running twice;
- with both slots held by the test, a call answers `slot_free: false` naming both holders, a second call joins the same run id, and releasing the slots lets that run finish;
- a call after a new untracked file changed the dirty path set starts a fresh run, the superseded runner's process ends, and the next answer carries the new run's exit code, not the old one's;
- a call after a commit changed HEAD's tree, with the dirty path set the same as before, starts a fresh run and the superseded runner's process ends;
- a run whose stand-in exits 75 is replayed as exit 1 with the stderr line;
- a call after the runner was SIGKILLed without an exit file starts a fresh run;
- `helperLaunchEnv=none` with a 1s claim bound exits 1 with `runner.log`'s line `gate helper: the runner was never started` on stderr and an envelope naming the re-run and `--wait`, leaves the lease passed and `process.json` naming the front, and `run-detached` on that run then refuses (the passed lease comes first; the claim collision under a live lease is `TestGateRunner_Refusals`'s row);
- `helperLaunchEnv=error` exits 1 at once with the launch error on stderr and the same envelope, leaves the lease passed and `process.json` naming the front, and the next call starts a fresh run;
- `helperLaunchEnv=late`, two seconds late, with a 4s claim bound and a 6s cap answers before its cap ends;
- with the key directory's `lock` file held by the test through `lock.AcquireWriteLock`, a 2s cap and a 1s lock margin answers 75 `done: false` with phase `starting` and creates no run;
- `--wait` under a strand creates no key directory and exits with the tier's code;
- `helperCapEnv=fallback` still runs capped: the call creates a key directory and a runner, its stand-in exits 7 at once, the call replays exit 7, and the helper's stderr, where its logger writes, carries the Warn naming the fallback exactly once;
- the `rungate` helper role calling `RunGate` under a strand and a 1s cap, over a stand-in that sleeps 2s and exits 7, exits 7 and creates no key directory;
- a capped call in a git worktree outside every hub, with its own gate.yaml, runs unslotted in a runner, answers 75 `done: false` while the stand-in sleeps and replays its exit code on the next call, and registers nothing under the run-process directory;
- a capped call from a directory in no git worktree, under a strand and a cap, is not capped and refuses as today, naming `lym config gate` run in a git worktree;
- a 1s cap with a 1s grace and a holding stand-in, called once, ends the stand-in and frees the slot within a few seconds, the exit file marked abandoned; the next call answers 75 `done: false` and replays nothing from that run;
- a SIGTERM to a waiting front exits 143 while the runner keeps running, and the next call replays the run;
- a SIGTERM to the runner makes the next call answer 75 `done: false` and replay nothing, and the call after starts a fresh run;
- a 1s-cap call joining a 6s-cap call's run leaves the runner live past the short call's deadline plus grace;
- a 1s cap with a 1s grace over a stand-in that sleeps 4s: a second call, made once the first answered 75 and before the first call's lease passes, joins the same run id, and the runner is still live after the first call's deadline plus grace, so a joiner moves the lease later;
- in a pair worktree (`hubforge.AddPair`) the runner is registered under the slug with `RoleGate` and `runproc.EndSparing` on the slug sparing `RoleDobby`, as pair teardown calls it, ends it, and a prime call registers nothing.

**ImpactSummary:** `lym gate test` from an agent strand now starts or joins a detached runner, and every call without a strand waits for its slots without a cap; `newCommand`'s two callers and `RunGate` change, `answerSlotBusy` is deleted with its one caller and `haltExitCode` loses the busy-hub case only it reached.
