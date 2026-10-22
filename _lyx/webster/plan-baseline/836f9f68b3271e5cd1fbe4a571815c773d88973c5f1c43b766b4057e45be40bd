# Card 9 — Pair teardown ends the pair's owned processes before its loop and session

**Create:**
- `internal/pairteardown#Teardown.endOwnedProcesses`
- `internal/pairteardown#Teardown.driverLockHeldIn`
- `internal/pairteardown#Teardown.registeredBattenAlive`
- `internal/pairteardown#Teardown.driverRecordPID`
- `internal/pairteardown#claimHeldGone`

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#SessionResult`
- `internal/pairteardown#Teardown.EndSession`
- `internal/pairteardown#Teardown.teardownEnded`
- `internal/pairteardown#settleClaim` -> `func settleClaim(status *string, landed, battenSeeded bool, battenState string, battenAlive bool) claimSettlement`
- `internal/pairteardown#Teardown.settleBoardClaim`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/pairteardown/doc.go`
- `pattern/PATTERN-pair-teardown.md`

**Uses:**
- `internal/runproc#End`
- `internal/runproc#List`
- `internal/runproc#Ended`
- `internal/runproc#SurvivorsError`
- `internal/runproc#RoleBatten`
- `internal/runproc#RoleDriver`
- `internal/fabricengine#RunProcDir`

**Intent:**
D4's base sequence: `lyx fabric remove`, batten's Worktree-Teardown and the reconcile's removals end batten, the driver, the step loop and the pair's pushes before the loop, the session and the worktrees go, so neither batten's strand revival nor the driver's re-step races the session end.
Card 10 adds the driver-check skip; this card's order is the one D4 gives when the checks run: the quiet wait as today, the refusal probe, `End`, the driver-lock re-check, the loop end, its run-lock wait, the session end.

New substrates on `Teardown`, wired in `New` and faked in `fakeSubstrates.teardown`:
`endOwned` (`endOwnedProcesses`: `runproc.End(RunProcDir(hub), slug, os.Getpid())`, so batten's own Worktree-Teardown, inside a registered `lyx batten run`, spares itself and its ancestors), `driverLockHeld` (`driverLockHeldIn`: the task worktree's `shedrun.DriverLock` probed with `runLockFree`), `battenAlive` (`registeredBattenAlive`: a live `runproc.List` record of the slug with role `batten`) and `driverPID` (`driverRecordPID`: the pid of a live record of the slug with role `driver`).

`EndSession`:
- A refusal from the probe returns before `End`, so a refused removal ends no owned process and leaves the session live.
- `End` runs after the refusal probe, for a present or a gone task worktree; `SessionResult` gains `EndedProcesses []runproc.Ended` and `Survivors []runproc.Ended`.
- A `*runproc.SurvivorsError` with `RefuseWhenBusy` returns an error wrapping `ErrDriverBusy` that names each survivor's pid and role and the way forward of retrying the removal once it is gone, with no loop end and no session end; without `RefuseWhenBusy` (batten's teardown, the reconcile's forced teardown) the survivors land in `SessionResult.Survivors`, are logged at Warn and the sequence goes on.
- Any other `End` error, such as an unreadable record directory, is returned for every caller, with no loop end and no session end.
- With the task worktree present, the driver-lock re-check follows `End`: a held lock means a process `End` could not reach, such as one an older binary started.
  With `RefuseWhenBusy` it refuses with `ErrDriverBusy` naming the lock, nothing ended further and nothing removed, and a way forward that progresses from that state:
  on Linux, find the holder in `lyx tender status`'s replaced-binary part or with `ps`, end it, then the removal again;
  on Windows (`Teardown`'s `goos` field, `runtime.GOOS` in `New`), where `End` ends no recorded pid, the driver's pid from the owner record (`driverPID`) and `taskkill /PID <pid> /T /F`, then the removal again, or, with no driver record, the lock path and ending the `lyx loom drive` process of the pair in Task Manager, then the removal again.
  Without `RefuseWhenBusy` it sets `SessionResult.DriverWasLive` and the loop end and session end run.
- A pair whose task worktree is gone skips the quiet wait, the re-check, the loop end and the run-lock wait as today, and still runs the refusal probe, then `End` for the slug, then the session end by exact name.
`teardownEnded` logs a forced teardown's survivors at Warn.

Board claim.
Until `batten-cancel-verb` settles a killed batten's record, a `running` batten record holds the claim only while a registered batten of the slug is alive: `settleClaim` takes `battenAlive`, and a `running` record with no live registered batten settles `claimHeldGone`, which keeps the claim and warns that batten's process is gone, naming `lyx board set-status <slug>` instead of `lyx batten pause <slug>`.
`settleBoardClaim` reads `battenAlive` only for a `running` record.

`TestTeardown_CallSequence` gains fake fields for the four substrates and rows: the full order `quiet, refusal, endOwned, driverLock, kill, seize, await, end`; a refused probe calls no `endOwned`; survivors under `RefuseWhenBusy` return `ErrDriverBusy` naming the pid with no `kill` and no `end`; survivors without it reach `end` and appear in `Survivors`; a driver lock still held under `RefuseWhenBusy` refuses with no `seize`, naming `lyx tender status` on `goos` linux, `taskkill /PID` with the fake `driverPID` on `goos` windows, and the lock path with no driver record on `goos` windows, and without it sets `DriverWasLive`;
an `endOwned` error that is not a `*runproc.SurvivorsError` is returned with no `kill` and no `end`; a gone task worktree calls `refusal, endOwned, end` only; `EndedProcesses` carries what `endOwned` returned; a `running` batten record with no live registered batten keeps the claim and logs the `lyx board set-status` way forward, and with a live one keeps the existing `lyx batten pause` warning.
`doc.go`'s `# Sequence` and `# Board claim` sections state the new steps.
`pattern/PATTERN-pair-teardown.md`'s session-end bullet names `runproc.End` ahead of the loop end and the driver-lock re-check after it, and its board-claim bullet names the gone-batten reading.

**ImpactSummary:** EndSession gains steps and SessionResult two fields; settleClaim's only caller is settleBoardClaim; Run, RemovePair and every CLI caller keep their signatures.

**Commit:** 9: pair teardown ends the pair's owned processes first
