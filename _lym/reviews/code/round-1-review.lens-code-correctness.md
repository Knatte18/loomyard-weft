```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens on the diff's changed functions, branches and error paths.

## Cases run

- `handleGatedBoundary`, re-prompt send returns a `strandGoneError` (untracked strand): `requireLiveStrand` returns it unwrapped (`send.go:205`), so `errors.Is(serr, errStrandGone)` holds. The relaunch runs; on success `serr` becomes nil and the attempt counts. **Correct.**
- `handleGatedBoundary`, relaunch refused (live holder 4242): `rerr` is non-nil and not finished. `serr` becomes "&lt;gone msg&gt;; resuming … failed: process 4242 …", which is not busy, so the loop finalizes Done with `gateEndCause`. **Correct.**
- `handleGatedBoundary`, relaunch reaches `awaitStartup` and its teardown finalizes the run: it returns `(result, true, rerr)`. **Correct as documented.**
- `relaunchForReprompt`, empty `SessionID`: returns an error. **Correct.**
- `relaunchForReprompt`, engine lacks `SessionHolderFinder` (codex): returns an error. **Correct.**
- `relaunchForReprompt`, `SessionHolder` errors: the error is wrapped and returned. **Correct.**
- `relaunchForReprompt`, `checkResume` refuses: the error is returned before any side effect. **Correct.**
- `relaunchForReprompt`, `Prepare` succeeds and then `removeGoneStrand` or `AddStrand` fails: `Prepare` has already rewritten `prompt.md`, settings and the manifest in the live run directory, and nothing restores them. **Finding 3.**
- `relaunchForReprompt`, `saveState` fails: the new strand is removed, but `run.state` keeps the new strand's GUID, name and session, with Started=false and PromptLine="", and nothing rolls them back. The loop then finalizes, so the saved record names a strand that was just removed. **Finding 2.**
- `relaunchForReprompt`, claude engine, success path: `awaitStartup` runs `checkLivenessTick(..., eventsProof=true)` on the reused events file. The first incarnation's SessionStart and turn-start lines satisfy `turnStartedAfterSessionStart` on the first tick, before the resumed provider is up. **Finding 1.**
- `removeGoneStrand`, untracked: no-op. Tracked with a dead pane: removed. `Status` fails: the error is returned. **Correct.**
- `strandAddSpec`: the fields are identical to the old literal, and nil `doneWhen` is passed for the resume. **Correct.**
- `liveStrand`'s three gone branches: the messages are unchanged and wrap `errStrandGone`. Callers that wrap with `notShuttleOrLiveStrandError` are unaffected. **Correct.**
- `StopStepRuns`, tied run with an untracked strand: `stopRecorded` writes stopped and `removeStrandIfPresent` is a no-op. Tied run whose output files all exist: the record stays running but is still listed under `Stopped`. **Finding 4.**
- `StopStepRuns`, untied run with an untracked strand: skipped. Untied run with a removable strand: listed under Untied. Run of another step: skipped. **Correct.**
- `runChild`, child exits non-zero with no envelope and `OnStepOrphaned` set: the note is appended. Nil hook: skipped. Hook returns empty: nothing appended. Envelope present, or watchdog kill: the hook is not called. **Correct.**
- `stepUntilStop`, panic with a child in flight: `killStepTree` takes its argument by value, so `pid.Child` survives, `hasChild` is true and the hook runs. Panic after the child exited: `record(zero)` cleared the child, so the hook is skipped. **Correct.**
- `deadLoopStop`, in flight with a trace id: the hook runs before `loopExitStop`. When that stop is busy, the pid file stays and the next invocation re-runs the hook. That is idempotent, but the retry's note comes back empty, since the runs are already stopped. **Correct (benign).**
- `stepRunsNote`: empty report and nil error give "". A stop error alone gives "could not stop the step's runs: …". **Correct.**

## Findings

1. **BLOCKING · design** · `internal/shuttleengine/relaunch.go:262`
   The resumed session's `awaitStartup` reuses the run's existing events file with `eventsProof` set. The first session's SessionStart and turn-start lines then mark the new strand started on the first tick, before the resumed provider has booted. The startup gate and trust dismissal never run, and the readiness wait is skipped.
   Evidence: `wait.go:1469-1473` states "Only the launch's own startup wait sets eventsProof. Its events file is created fresh for the launch, so only this launch's hooks satisfy it". `wait.go:1516` is the check, and `wait.go:1689-1711` reads from the file's start. The test misses it because it seeds only `STOP:turn1` (`gate_test.go` diff line 88).
2. **LOW · design** · `internal/shuttleengine/relaunch.go:246-260`
   On a failed `saveState`, the in-memory `run.state` keeps the new strand's GUID, name and session, with Started=false and PromptLine="", after that strand is removed. The finalize that follows writes a record naming the removed strand, not the run's last real one.
   Evidence: `relaunch.go:247-252` mutates the state before the save, and `relaunch.go:254-259` restores nothing.
3. **LOW · design** · `internal/shuttleengine/relaunch.go:232`
   `Prepare` runs before `removeGoneStrand` and `AddStrand`. Claude's `Prepare` overwrites `prompt.md` (`claudeengine.go:223`), along with the settings and the launch manifest. So when the resume fails after it, the run directory's `prompt.md` (`RunState.PromptPath`) holds the re-prompt instead of the original prompt. On success too, the original prompt artifact is lost.
4. **LOW · design** · `internal/shuttleengine/stepstop.go:93-97`
   A tied run whose output files all exist keeps reading running (`stop.go:91`, `stopOutcome`), yet it is appended to `report.Stopped`. The note then claims "stopped shuttle runs: X". When its strand is now untracked, nothing waits on the run, so it still reads running with a gone waiter: the case the board entry asks to reconcile.

## Tool calls

1. `send.go:195-215`, `sendVerified` error returns, and `wait.go` `sessionGone`: my context lacked whether the send path wraps `requireLiveStrand`'s error, and how session liveness reads the reused events file.
