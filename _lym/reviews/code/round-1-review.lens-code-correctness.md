```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes through concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `hibernateengine.PauseAndConfirm`, Pause returns an error before any entry is added: result zero, `ConfirmationNone`, err returned. Correct.
- `PauseAndConfirm`, Pause adds an entry and a later step errors: `Owned` is set and the outcome is `ConfirmationNone` with err, so the caller can still lift. Correct.
- `PauseAndConfirm`, a hub entry already stands (`AlreadyPaused`): Added is not called, `Owned` is nil and the wait runs. Correct; it owns none and lifts none.
- `PauseAndConfirm`, `ResumedDuringGrace`: `ConfirmationLifted` with no reading. Correct.
- `PauseAndConfirm`, `Confirming` returns false, or nil and an error: `ConfirmationStopped` with that error. Correct.
- `PauseAndConfirm`, `Carried` is nil (the hub stop): defaults to true, so the wait runs to Reached or TimedOut. Correct.
- `awaitConfirmation` with `carried` false at reading 0: `ConfirmationStopped`. This matches the old `stillOurs` path. Correct.
- `carry`, Pause error: `ConfirmationNone`, then `fail(kept, err.Error())`. err is non-nil on that outcome. Correct.
- `carry`, `recordOwned` fails in Added: `Owned` is set, `ConfirmationNone`, then `fail` with kept. Same as before. Correct.
- `carry`, advance to Confirming does not move: `ConfirmationStopped`, then `withdrawn(r, kept, nil)`. One small difference: the old code returned advance's zero `Request{}`, while the new code returns the prior `r`. The lift still happens; only the returned value differs, and that is benign. Correct.
- `carry`, TimedOut: `fail(kept, Withheld)`. Correct.
- `carry`, Lifted: `finish Cancelled`. Correct.
- `hibernatecli.PauseHub`, `hubDeps` fails: zero `HubPause` and the error. `runHubStop` records the failure and goes on. Correct.
- `orchcli.beamDown`, no strand and no live dobby: `endOrch` still runs and reports `OrchProcessEnded` false. Correct.
- `beamDown`, `endOrch` errors: returns the report with err, wrapped. Correct.
- `beamDown`, `stopOrch` refuses or `endDobby` fails: returns early and `endOrch` never runs, so the orch's provider process survives. See F2.
- `reedcli.down`, `endProcesses` errors: `DownReport{}` and err; the verb prints err. Same as before. Correct.
- `reedcli.down`, engine Down errors: report with Ended, and err; the verb prints only err. Same as before. Correct.
- `reedcli.down`, `EndedServer`: the watchdog wait defaults apply on a fresh `reedCLI`. Correct.
- `DownWorktree` on the prime: `endPairProcesses` returns nil for the prime, then the session is killed. Correct.
- `DownWorktree`, `wireLocation` fails: error returned, recorded as a failure. Correct.
- `beamcli.stopAll`, slug together with `--all`: refused before any work, with a way forward. Correct.
- `stopAll`, not detached: spawns and prints the pid. A spawn error is wrapped with a way forward. Correct.
- `stopAll`, detached: runs the sequence and does not re-spawn. Correct.
- `spawnDetachedStop`: `selfexec`, `proc.Detach`, nil stdio, `proc.Reap`, spawn logged at Info, failure at Warn. Correct.
- `hubStopWorktrees`: `HubOrchs` always puts the prime first, so `orchs[1:]` is safe. A beam that `HubOrchs` skips becomes a task pair. See F3.
- `runHubStop`, pause errors: the failure is named, `Owned` is nil, `Unconfirmed` is "", and every later step still runs. Correct.
- `runHubStop`, worktrees errors: nil lists; prime BeamDown, reap, lift and the prime's down still run. Correct.
- `runHubStop`, reap errors: the partial Ended list is kept and the failure named. Correct.
- `runHubStop`, Lifted outcome with `Owned` set: Lift matches nothing and succeeds, so `PauseLifted` reads true for an entry the stop never lifted. See F1.
- `runHubStop`, lift errors: `PauseLifted` stays false, the failure is named, and the prime's down still runs last. Correct.
- `hubStopStepsFor`, `tendercli.HubDeps` fails inside reap or lift: the error is returned and recorded. Correct.

**Findings**

- **F1.** NIT · design · `internal/beamcli/hubstop.go:384-390`
  When the operator resumes the hub during the grace (`ConfirmationLifted`), `Owned` is already set. The no-op `Lift` then succeeds and the report sets `pause_lifted: true` for an entry the stop never lifted.
  Evidence: `internal/hibernateengine/watch.go` (PauseAndConfirm sets `Owned` in Added before the `ResumedDuringGrace` return); `internal/tenderengine/resume.go:185-201` (Lift matches nothing and succeeds).

- **F2.** LOW · design · `internal/orchcli/stop.go:57-79`
  `beamDown` returns before its last step whenever `stopOrch` refuses or `endDobby` fails, so the orch's registered Claude process is never ended. One case is `start.lock` held past `startLockWait`. In the hub stop that leaves an orch process running, against D9's criterion "leaves no … orch Claude process". The failure is named, but the independent last step is skipped.
  Evidence: `internal/orchcli/stop.go` (early `return report, err` before `c.endOrch()`).

- **F3.** LOW · design · `internal/beamcli/hubstop.go:295-322`
  `HubOrchs` skips, with a warning, a beam whose `_lym` junction or origin record cannot be read. `hubStopWorktrees` then puts that beam into `tasks`. It is downed through reed's pair path, and `BeamDown` never runs for it, so its orch strand state is not removed and its dobby is ended only as a pair process.
  Evidence: `internal/orchcli/live.go:40-56` (the skips); `hubstop.go:318-320` (non-beam names become tasks).

Tool calls:
- `grep`/`sed` of `internal/hibernateengine/watch.go` for `advance`, `withdrawn`, `fail` and `carry`'s head: my context lacked their bodies, which I needed to compare `carry`'s old and new withdraw return.
