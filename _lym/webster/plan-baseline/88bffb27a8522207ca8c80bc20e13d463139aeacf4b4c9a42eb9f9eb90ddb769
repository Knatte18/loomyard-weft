# Card 10 — gate-detached-runner

**Create:**
- `//internal/gatecli/detachrun.go`
- `internal/gatecli#detachDirName`
- `internal/gatecli#detachRun`
- `internal/gatecli#detachRequest`
- `internal/gatecli#runPhase`
- `internal/gatecli#detachExit`
- `internal/gatecli#detachRun.writeLease`
- `internal/gatecli#detachRun.expireLease`
- `internal/gatecli#detachRun.leaseHolds`
- `internal/gatecli#detachRun.claim`
- `internal/gatecli#detachRun.writePhase`
- `internal/gatecli#detachRun.writeExit`
- `internal/gatecli#startReading`
- `internal/gatecli#runnerCmd`
- `internal/gatecli#runDetached`
- `internal/gatecli#TestGateRunner_Refusals`
- `internal/gatecli#TestGateRunner`
- `internal/gatecli#helperDelayEnv`

**Edit:**
- `internal/gatecli#newCommand`
- `internal/gatecli#testRequest`
- `internal/gatecli#acquireSlot`
- `internal/gatecli#execution.runWithSlots`
- `internal/gatecli#runStandIn`

**Extends:**
- `//internal/gatecli/doc.go`
- `cmd/lym#unboundedWaitFiles`
- `//pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/gatecli#catchTerminatingSignals`
- `internal/gatecli#startGate`
- `internal/gatecli#runningGate`
- `internal/gatecli#eventually`
- `internal/gatecli#scenarioConfig`
- `internal/gatecli#processGone`
- `internal/gateslot#WaitDir`
- `internal/proc#StartTime`
- `internal/proc#CreationTime`
- `internal/clihelp#RunRootCtx`
- `internal/state#UpdateJSON`

**Intent:** D1's runner half and D3's lease, as a hidden internal-audience verb under `lym gate` (PATTERN-cli-cobra), in the new file `internal/gatecli/detachrun.go`. Card 11 adds the front that starts and joins it.

**Run directory.** A run lives in a subdirectory of a key directory under the worktree's never-tracked gate directory, `gateslot.WaitDir(anchor)/detach/<key>/<run id>` (PATTERN-durable-vs-ephemeral-state: the `.lym` part is `gateslot`'s, and `detachDirName` is gatecli's own subpath, as `greenFileName` is).
`detachRun` holds the subdirectory and names its files: `request.json` (a `detachRequest`: `Verb`, `test` here and `card-verify` from card 13, `Args`, the call's words after the verb, and `Cwd`), `lease.json` (the deadline), `process.json` (pid and start reading), `tree.json` (HEAD's tree and the sorted dirty paths the front recorded), `phase.json` (a `runPhase`: `State`, one of `starting`, `slots` or `running`, the tier's name for a tier run, and `Line` for a card's line), `stdout`, `stderr`, `runner.log` (the runner process's own output), `exit.json` (a `detachExit`: `Code` and `Abandoned`) and `delivered`.
`writeLease` and `expireLease` rewrite `lease.json` through `state.UpdateJSON(<lease path>, <lease path>.lock, mutate)` (PATTERN-tolerant-records, since fronts of different `lym` binaries rewrite one lease): `writeLease`'s mutate keeps the later of the stored and the new deadline, so the lease is the latest deadline any front wrote; `expireLease`'s writes `now` less a second, the one write that moves it earlier; `leaseHolds` is true while a lease exists and lies after `now`.
`claim` creates `process.json` exclusively (`O_CREATE|O_EXCL`), holding the pid and the start reading, and fails when it exists, so of two creates the second fails.
`writePhase` writes atomically and logs a failure; `writeExit` writes atomically through `fsx.AtomicWriteBytes`, the runner's last write.
`startReading(goos, pid, ...)` reads a start that tells a reused pid apart: `proc.StartTime` off Windows, and `proc.CreationTime` in Unix nanoseconds, as a decimal string, on Windows, where `StartTime` is unreadable.

**Runner verb.** `newCommand` adds `runnerCmd`, `lym gate run-detached <run-dir>`, `Hidden: true`, audience `clihelp.AudienceInternal`, whose `gateTree` builds a fresh gate tree with `newCommand`'s own parameters.
`runDetached` refuses, before it runs anything or writes an exit file, a run directory with no lease, a passed lease, a `request.json` that is missing or does not decode, a verb other than `test` (and, from card 13, `card-verify`), and a run its claim finds already claimed, with an error envelope naming `lym gate test` and `lym gate card-verify` as the way forward (`gate run-detached: <dir> holds no live lease of a gate call; way forward: run lym gate test or lym gate card-verify, which start and join their own runner`; the claimed case says the run is already claimed).
Bound: a direct call reaches only a subdirectory a front made and refuses one its runner claimed; a hand-written lease lets one run start, which ends at that lease like any other, so a direct call adds at most one detached run bounded by the lease it found.
The lease checks and the request read come first; the claim itself, `claim` with its own pid and `startReading`, is the one check for a claimed run, so a front's exclusive create that came first, before or after the lease checks passed, refuses the runner the same way.
A refusal before the claim leaves the run unclaimed, so the front that started it meets the start failure (card 11) and its way forward.
The runner then runs the request: for verb `test` it builds `tree := gateTree()`, sets `tree.SetOut` to the `stdout` file, `tree.SetErr` to the `stderr` file and `tree.SetArgs` to `test --wait <args>`, and runs `clihelp.RunRootCtx(lymcwd.WithCwd(callCtx, Cwd), tree, <the stdout file>)`, where `callCtx` is the context the lease read and the signal catch cancel; `clihelp.ExecuteIn` is not used, since it seeds `context.Background()` and merges stderr into stdout.
So the slot wait and the tiers run in-process exactly as a `--wait` call, and a cancel of `callCtx` reaches `runTest`'s run context.
Meanwhile it reads the lease at most once a second from its start, its slot wait included (PATTERN-daemon-wakeups), and cancels the call's context once the lease has passed; its own `catchTerminatingSignals` cancels it on a terminating signal too.
A call that ended under neither writes `{Code: <the call's code>}`; a passed lease writes `{Code: 1, Abandoned: true}` after appending `gate test: the run was abandoned, since no call renewed its lease; way forward: run the same command again` to `stderr`; a signal writes `{Code: 128 + n, Abandoned: true}`.
The tiers' trees are killed and the slots released by the cancelled call as a signal does today, and each record written after the cancel carries `Abandoned` (card 9).

**Phases.** `testRequest` gains the field `phase func(runPhase)`, nil outside a runner: `acquireSlot` calls it with state `slots` and the tier before it waits, and `execution.runWithSlots` with state `running` and the tier once the slots are held, or at once for a run that takes none; the runner sets it to `writePhase`.

**Wakeups.** `internal/gatecli/detachrun.go` joins `unboundedWaitFiles` in `cmd/lym/daemonwakeups_test.go`, and the background file's list gains the runner's lease read, which wakes once a second for as long as the lease lasts.
The doc gains a `# Detached run` section describing the run directory, the runner verb, its refusal and bound, the lease read and the exit file.

Tests:
- `TestGateRunner_Refusals`, tier0, runs `RunCLIIn(dir, out, []string{"run-detached", runDir})` over a run directory with no lease, one with a passed lease, one with a live lease and no `request.json`, one with a live lease and a request of verb `bogus`, and one with a live lease, a valid request and a `process.json` a front already created (D1's late claim, refused by the claim's exclusive create): each exits 1 with an envelope naming `lym gate test` and `lym gate card-verify`, the claimed row saying the run is already claimed, and writes no exit file.
- `TestGateRunner`, tier1, in `test_integration_test.go`'s harness with its own hubforge hub: it writes a request and a lease itself and starts the gate helper with `run-detached <run-dir>`.
  `runStandIn` gains `helperDelayEnv`: a tier invocation sleeps that long before it exits with its code, for a tier that runs a while and still ends.
  Rows: a tier whose stand-in sleeps 2s under `helperDelayEnv` and exits 7 reads `phase.json` named `running tier0` while it sleeps, then leaves `stdout` holding the tier's output, `stderr` holding the summary line and an exit file `{7, false}`;
  a lease of two seconds that passes while the stand-in holds kills it and its sleeper, frees the slot and writes `{1, true}` with the abandoned line, and the tier's run record carries `Abandoned`;
  the same with every slot held by the test, so the lease passes in the slot wait: no stand-in ran, the wait record is gone and the exit is `{1, true}` with `phase.json` named `slots tier0`;
  a SIGTERM to the runner writes `{143, true}` and frees the slot.

**ImpactSummary:** a new hidden verb and its run store in gatecli; the in-process call gains a nil-by-default phase hook, so every existing `lym gate test` path runs as before.
