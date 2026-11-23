```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function and branch the diff adds or changes through a concrete case.

**Per function or branch, the case I ran, and the result**

- `crashresume.go` `resumableAfterInterruption`:
  - `NoResume` true, or `SessionID` empty → false. Correct.
  - died → true; stopped and `Interrupted` → true; stopped without it → false. Correct.
  - running with a dead pid and no relaunch mark → true; running with a live pid or a relaunch mark within its bound → false. Correct.
  - timeout, api-error or done → false. Correct.
- `resumeInterrupted`:
  - `found` false, `Unawaited`, a caller-set `ResumeSessionID`, or output files that differ → false. Correct.
  - Engine without `SessionResumer` → false. Correct.
  - `checkResume` fails with `ErrNoTranscript` → logged, fresh start. Correct.
  - `checkResume` returns a warning but no error → resumes with the warning. This matches how `start` treats a caller's own resume. Correct.
- `setAside`:
  - Missing directory → returns. Correct.
  - `updateRunState` fails → only logged. This is documented. Correct.
- `markNoResume`: record absent or already set aside → `errLeaveRunRecord`, which `updateRunState` turns into nil (`rundir.go:241-248`). Correct.
- `RetireInterruptedRuns`:
  - `toldErr` set → returned. Missing root → 0, nil. A record that does not decode is skipped. A failed write is joined into the error and the scan goes on. Correct.
  - Race: the record changes between `loadRunState` and `updateRunState`. `markNoResume` does not re-check resumability, and a record that vanished still counts as set aside. A count-only NIT, so I filed no finding.
- `carryover.go` `newestPredecessor` and `carriedInstructions`:
  - Empty key, unreadable root, or no run of the key → none. A tie on `CreatedAt` is broken by `runDir`. Same behavior as before the refactor. Correct.
- `run.go` `start` (lines 642-677):
  - Successful resume: spec, warning and copy flag replaced; carry-over block left out of the prompt; carried instructions still recorded; notifier skipped; `setAside(pred)`. Correct.
  - Successful fresh start with matching output files: `setAside(pred)`. Correct.
  - Resumed attempt fails with `run != nil` and a retryable result: **defect**, finding 1.
  - Resumed attempt fails with `run == nil` (a Prepare or AddStrand failure): nothing is set aside, so the next start resumes again. Accepted: such a failure is not specific to resuming.
- `stop.go` `stopRecorded`, `interrupted` parameter:
  - Terminal record → left unwritten, flag untouched. Running record whose output files all exist → stays running. Running record → stopped with the flag. Correct.
  - Failed write → error, strand untouched. The `StopStrand` path passes false. Correct.
- `stepstop.go` `stopStepRunsWhere` passes true. Both step-stop callers end a step that never finished. Correct.
- `attach.go` `removeSupersededStrands` passes `c.state.Outcome == runOutcomeRunning`:
  - Terminal candidate → `stopRecorded` leaves it alone, so the flag does not matter.
  - Running candidate whose pane is dead → stopped and marked interrupted. Correct.
- Clobbering: the pred's own waiter is still alive when a fresh start of the same output files takes its place. `setAside` writes `NoResume`, and the waiter's later `finalize` → `saveState` writes the whole in-memory record (`stop.go:31-32`). That drops `NoResume`. The newer run then supersedes it as newest. It matters again only if the newer run's directory is removed on done and the old record outcome is died. Filed as finding 2, LOW.
- `loomcli/gotoresume.go` `gotoAdmitted`:
  - No location → nil hook, no retire. Correct, since the runner needs the location.
  - Location set: retire, then reset; the reset's error is returned. Correct.
- `retireInterruptedRuns`:
  - `retireRuns` nil → falls back to `stepStopRunner`; a config failure is logged. Correct.
  - Error together with `retired &gt; 0` → logs both. Correct.

**Findings**

1. **MEDIUM**, design, `internal/shuttleengine/run.go:667-676`: a failed resumed start is retried as a resume, not as a fresh start.
   - When a resumed attempt fails with a retryable result (provider died before its first input, strand removed), `start` sets both runs aside and loops again.
   - The loop still holds the same `spec`, with `ResumeSessionID`, the preamble and `SkipSkillLoadTurn`. So every retry re-launches the session that just failed to come up.
   - The comment at line 668 and the summary ("so the next start runs fresh instead of retrying the resume") claim the opposite.
   - Fix: on that branch, go back to the original spec with the carry-over block applied, and to the check results from before the resume, before the next attempt. Or end the loop, and make the comment and summary say so.
   - Evidence: `run.go:644-649` (spec replaced once, before the loop), `run.go:656-676` (the loop reuses `spec`), `run.go:687-688` (`startRetryable`).
2. **LOW**, design, `internal/shuttleengine/run.go:659-661` with `internal/shuttleengine/stop.go:31-32`: a still-waited pred can lose its `NoResume` mark.
   - `takesPlace` calls `setAside(pred.runDir)` without checking that pred's waiter is gone.
   - A pred whose waiter is still alive later finalizes through `saveState`, which writes its whole in-memory `RunState` and drops `NoResume`.
   - If the superseding run's directory is then removed on done, a later start of the same output files can resume the stale died session.
   - Evidence: `wait.go:2031-2037` (`finalize` saves `run.state` whole) and `stop.go:31-32`.

Tool calls:
- Read `saveState`, `spec.go`'s resume fields and `updateRunState`/`errLeaveRunRecord`, to settle whether `finalize` can clobber `NoResume` and whether `markNoResume`'s leave counts as success. My context did not hold those definitions.
