```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the prime-dobby carrier diff. I ran each case by reading the code, not by executing it.

**Per function or branch: case, verdict**

- `Carrier.Tick`, no sequence running, pending waiting request: takes it up, starts the goroutine, returns at once. Correct.
- `Carrier.Tick`, `running` true: returns nil. Correct.
- `Carrier.Tick`, takeUp error (failed lift): `running` stays false, nothing written, the error returns to dobby's tick and is retried next tick. Correct.
- `Carrier.setPhase` through `Deps.tellPhase`: `advance` and `finish` run in the goroutine without `c.mu`, and `Tick` holds `c.mu` only across takeUp, which never tells a phase. No deadlock. Correct.
- `Carrier.HoldsReexec`, phase pausing, confirming, hibernating or resuming: true. Waiting and terminal phases: false. Correct. The window between the sequence's file write and `setPhase` is documented as a bound.
- `takeUp`, absent or terminal request: `errUnchanged`, `ok` stays false. Correct.
- `takeUp`, request named by a dead pid or by this process after a re-exec: taken up. Correct.
- `takeUp`, request named by a live other pid: left alone. Correct.
- `takeUp`, pausing or confirming with `Owned` set: lifts the entry, `Phase` returns to waiting. The reason string reports the old phase because Go evaluates the right side of the tuple assignment first. Correct.
- `takeUp`, pausing with `Owned == nil` (entry added but not yet recorded when the carrier died): lifts nothing and the entry stays live and unowned. Documented bound, not a finding.
- `carry`, taken up in hibernating or resuming: `failAtWake(…, nil, …)` writes failed via `finish`, which passes `ours` because takeUp recorded this process. Lifts the entry only if `ResumeOnWake`. Correct.
- `carry`, taken up in resuming with `ResumeOnWake`: lifts the hub pause with no reconcile and no pair holds, and the notice says only "its pause was lifted". See finding 2.
- `withdrawn`, `kept == nil`: returns `(r, err)`. Correct.
- `withdrawn`, `kept != nil`: lifts by identity, and a repeat lift is a no-op. A lift error is joined to `err`. Correct.
- `advance(Confirming)` write error: `withdrawn` lifts `kept`. The request stays non-terminal, still naming this process, in the pausing phase. The next `Tick` finds `ours` and phase pausing, lifts `Owned`, and restarts from waiting. Consistent.
- `advance(Hibernating)` returning `!ok` after a cancel: `withdrawn` lifts `kept`. Correct.
- `awaitConfirmation` returning `confirmationStopped`: `withdrawn` lifts `kept`. Correct.
- `finish`/`failRequest` when `Pause` errors after a cancel: `finish` is a no-op because the request is already terminal. `failRequest` still lifts and queues a "failed" notice on a cancelled request. Pre-existing behaviour, not a regression.
- `Submit`, pending request with no carrier or a dead one: refused with the cancel way forward and nothing written. Matches the decision.
- `Submit`, terminal request with a kept live entry: `refuseKeptPause` refuses it. Equivalent to the old condition. Correct.
- `Submit`, `DobbyLive` error: logs a Warn, `DobbyLive` false, `WayForward` set, the request is still written. Correct.
- `Submit`, two racing submits: the lock admits one and refuses the other. Correct.
- `Cancel`, hibernating with a live carrier: refused. Hibernating with no live carrier: cancelled in any phase. Correct.
- `Cancel`, pause grace running with nothing owned yet: the later `recordOwned` writes `Owned` into the cancelled request, and the sequence then reaches `advance`, which fails, and `withdrawn` lifts `kept`. Correct.
- `Status` carries the same shape as before. Correct.
- `hibernateDaemonLive` in fabriccli: ended phases give false, a pending request whose carrier is dead gives false, and a waiting request with `daemon_pid` 0 gives false. A pending request taken up by a live prime dobby gives true. Correct. A prime `--set` with an untaken request ends the dobby, and the request waits on disk for the next one.
- `holdReexec`: runs the check only while `holds()` is false. Correct.
- `dobbyWatchDirs`, empty `hibernateRequestPath` (a beam dobby): adds nothing. Correct.
- `dobbyWatchDirs`, hibernate directory not yet existing: see finding 1.
- Pause scope: a gate-held call keeps a `WaitingMarker`, which `ListNetOps` never reads, so the dobby's own held calls never withhold the confirmation. The sequence runs in a goroutine outside the tick steps a pause holds back. The pause does not stop the wait. Correct.
- `waitForTrigger` with `--after-all`: uses the unchanged `RunSlugs`, `RunStatus` and `StateRunning` reading. It does not use `battenrecipe.PastLand`. Unchanged by the diff, so out of this lens.

**Findings**

1. LOW, design. `internal/beamcli/dobbywatch.go:129-131` (`dobbyWatchDirs`), `internal/hibernatecli/deps.go:42-48`.
   - The request's directory (`lymdirs.HibernateDirName`) may not exist when dobby opens its watch on a hub that never hibernated.
   - `openDobbyWatch` skips a missing directory and retries only on the next event from another watcher.
   - Nothing in the diff creates the directory at dobby start.
   - A fresh request on an idle hub is then taken up on the loop's timer, up to `idleCeiling` of one minute (`internal/dobbyengine/loop.go:27`), not on the write.
   - This contradicts the claims in `internal/beamcli/doc.go` ("a written request wakes the tick that takes it up") and in `_lym/landing/summary.md`. The summary's "To check by hand" says the dobby should take the request up "within a second".
   - Fix: create the hibernate directory before the watch opens.

2. LOW, design. `internal/hibernateengine/watch.go:67-70` (`carry`) and `internal/hibernateengine/carrier.go:90-96`.
   - A request taken up in the resuming phase with `ResumeOnWake` set ends failed and lifts the hub pause.
   - That skips the origin reconcile and the pair holds the wake would have made.
   - The notice says only "its pause was lifted". It neither names the skipped reconcile nor points to `lym cleanup --apply`.
   - The request ends failed with the hub unpaused and unreconciled, and the operator is not told.
   - Fix: add the cleanup way forward to that notice.

3. LOW, design. `internal/hibernateengine/carrier.go:46-69` and `internal/hibernateengine/watch.go:341-371`.
   - After a cancel followed by a fresh submit, the old goroutine stays in `running` until its next `stillOurs` check, which is up to the 30-second trigger poll ceiling (5 seconds in confirmation).
   - `Tick` takes up nothing while it runs.
   - No event wakes dobby when the goroutine ends, so the new request can wait for the loop timer, up to a minute.
   - The delay is bounded and no state is wrong, so I rate this LOW.

4. LOW, design. `internal/hibernatecli/deps.go:84-90`, `internal/beamcli/dobbydeps.go:190-198`, `internal/hibernatecli/deps.go:63-71`.
   - `hubDobbyDeps` for the prime now calls `NewDobbyCarrier`, which calls `hubDeps`. `hubDeps` returns an error if `PrimeName` or `ResolveWorktree` fails.
   - A hibernate-wiring error now stops the prime's dobby from starting at all.

I found nothing that is wrong on the main path.

Tool calls (1):
- `grep` of `internal/pauserecord` for `NetOp`, `WaitingMarker` and the gate (`hold.go:124-146`, `272-336`, `390-440`; `doc.go:33-38`), to check whether gate-held calls count as in-flight network operations and could withhold the confirmation. My context lacked that code.
