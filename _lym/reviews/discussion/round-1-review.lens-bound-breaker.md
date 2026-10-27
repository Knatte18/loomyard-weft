```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked every bound, guarantee and claim about existing code that the record states. Each claim is listed below as held or broken.

**Claims attacked**

- D1, "every lock prober holds only between a try and a release": **held**. Every prober goes through `dobbyengine.Live` or `battencli.lockHeld`, a try followed by a release (`start.go:243-255`, `dobbyreads.go:198-210`). That includes the probers the record doesn't name: `lym beam stop` (`orchcli/cli.go:256`) and pawl's `DobbyLive`/`stopBeamDobby` (`loomcli/pawlwiring.go:131,285`).
- D1, "a process that loses the lock … its run-process record is never written": **held, with a gap**. A re-exec loser keeps the old image's record, which has the same pid and start time and so the same file (`runproc/record.go:99`). The record exists, but it was not written by the loser.
- D2, "no other dobby can be legitimately live with this pid in the record": **broken**, see F3.
- D2, "a second dobby inside the exec gap is the way to lose; the re-exec exits after at most ten seconds": **held**. `AcquireWriteLockWithin` is bounded (`lock.go:53-66`). A `lym beam stop` during the wait signals the waiter through the shared record. The waiter then takes the lock once the winner dies and exits at the loop's `ctx.Err()` check, so the stop still ends it.
- D3, "every way a dobby ends leaves a line in `dobby.log`": **broken**, see F1.
- D3, failed exec "posts one notice": **held**, one notice per failed binary identity (`selfexec.go:71-82`, `hasFailed`).
- D3, "a refused fresh start is the idempotent case `EnsureLive` relies on": **held**. `ensureOne` spawns only on `Live` false (`start.go:227-236`).
- D4, "a re-exec reads its own pid and keeps `State.Tidy` … settles the first": **held in outcome**. `tidyStep` settles any `State.Tidy` whatever the pid (`tidy.go:146-150`), and `StopStrand` settles a run record across processes (`shuttleengine/stop.go:51-82`). The pid condition the record attaches is not what keeps it.
- D5, "`dobby.out` grows between spawns by stderr alone": **held**. A re-exec does not reopen the fds.
- D5, rotation of `dobby.log` by its one writer: **broken in a corner**, see F5.
- D6, "a dead dobby shows within ten minutes": **held**. The loop sleeps at most `max(idleCeiling, probe)` (`loop.go:240-244,374-379`).
- D7, the history entry names the landing machine: **held**. Entries are appended when the row completes, and nothing rewrites them except the producer rename (`shedengine/rename.go:66`).
- D8 bound, "its state record still lists them as landed since the last tidy": **held**. Only `settleTidy` clears `LandedSinceTidy` (`tidy.go:203`), and no start path resets it (`loop.go:190-204`).
- D9, migration "once" and "a second pass changes nothing": **held for idempotence, broken for "once"**, see F6.
- D10 bound, "the gate slots serialize it with every other gate run": **broken**, see F2.
- D10, "the record is rewritten fresh at each start": **broken** when two checks overlap, see F2.
- D11, "a record that reads running whose pid is not alive shows ended without a verdict": **broken by pid reuse**, see F4.

**Findings**

- **F1.** MEDIUM, design. Location: Decision 3, and Decision 5 which moves stdout. Decision 3's list of exits covers only exits from inside `Run`. `runDobby` can also end before `Run`:
  - `placeDobby` refuses.
  - `hubDobbyDeps` fails: `ResolveWorktree`, `PauseGeometry`, or `noticeProbeInterval` parsing the new binary's recipe.
  - A runtime panic.

  Counterexample: a deploy ships a binary whose `placeDobby` or `hubDobbyDeps` fails on this hub. The re-exec'd image prints an error envelope to stdout, which is `dobby.out` after Decision 5. It writes no `dobby.log` line and posts no notice, so it ends silently: the incident's own pattern, contradicting the Goal's "never ends silently".
  Evidence: `internal/beamcli/cli.go:373-384`, `internal/beamcli/dobbydeps.go:73-85`, `internal/dobbyengine/start.go:274-276`.
- **F2.** MEDIUM, design. Location: Decision 10's bound. The bound says the gate slots serialize the check with every other gate run. They don't: the hub's `slots: 2` (`_board/_lym/config/gate.yaml` via `template.yaml` `slots: 2`) lets two gate runs go at once, and `gatecli` has no per-worktree lock (no `lock.` use in `internal/gatecli` beyond the green record, `green.go:109`). Counterexample: `lym adopt root`, then `lym adopt &lt;beam&gt;` minutes later, both record an owner and each spawns a check (`adopt.go:323-330`). The second check truncates `suite.log` and rewrites `check.json` while the first still runs. Both then write per-tier progress into the one record, and both post tier lines and end notices to the orch. `lym beam status` shows whichever wrote last. The Decision states nothing about a second concurrent check in the same prime.
- **F3.** LOW, design. Location: Decision 2 rationale, and Decision 3's re-exec notice. Telling a re-exec apart by "the record holds this pid" breaks on pid reuse. A dobby killed from outside (Open risks: SIGKILL/OOM) leaves its pid in `state.json`. After a reboot, a fresh `lym beam dobby` can get the same pid. It is then treated as a re-exec: it waits ten seconds, logs a re-exec start, and on refusal posts the re-exec notice. The record's start time could tell the cases apart, and `proc.StartTime` exists (`runproc/record.go:72`). Evidence: `internal/dobbyengine/loop.go:190-196`.
- **F4.** LOW, design. Location: Decision 11. The dead-pid check uses the pid alone. `check.json` lives in the never-tracked `_board/.lym/adopt` and survives a reboot. A check ended by a reboot leaves `running` with a pid a later process can reuse, so `lym beam status` shows it running indefinitely. The fix is to compare the start time `check.json` already records, or the process start ticks, as `runproc` keys its records (`runproc/record.go:70-73,99`).
- **F5.** LOW, design. Location: Decision 5's rotation. `dobby.log` has a second writer: a process refused the lock writes its refused line through the same seam (Decision 3), concurrently with the live dobby. Counterexample: both processes see the file past 1 MiB and both rename `dobby.log` to `dobby.log.1`. The second rename replaces the first's `.1` with an almost empty file, which loses the previous MiB. Nothing in the Decision makes the size check and rename atomic across processes. Evidence: Decision 5 text; `internal/dobbyengine/passes.go:256-259` (the current single writer).
- **F6.** NIT, consistency. Location: Decision 9, "once". The migration runs at every reconcile where `adopt` is absent and `finalize` is tuned. An operator who removes `adopt` from the hub file to fall back to the template value gets it re-copied from `finalize` at the next reconcile. The real bound is "while `adopt` is absent", not "once". Evidence: `internal/gateslot/config.go:427-470` (the migration's held-key rule).

**Tool calls**

1. `internal/runproc/record.go` and `end.go` (`Register`, `SelfRecord`, `EndRole`): to see whether a re-exec keeps the same run-process record file, which is beyond the manifest.
2. `internal/gatecli` grep for locks and concurrency: to test D10's serialization claim against concurrent gate runs in one worktree.
3. `internal/shuttleengine/stop.go` (`StopStrand`): to test D4's "settles the first" across a re-exec.
4. `internal/shedengine` history append sites and `HistoryEntry`: to test whether D7's machine stamp could name a machine other than the one that finished Land.
