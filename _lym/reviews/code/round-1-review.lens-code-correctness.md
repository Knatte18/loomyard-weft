```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, tracing each function and branch the diff adds or changes with a concrete input.

**Per function or branch: the case I ran, and whether it is correct**

- `stopStepRunsWhere`, root missing: `os.ReadDir` gives NotExist, so it returns an empty report and nil. Correct.
- `stopStepRunsWhere` with `toldErr` set: returns `toldErr` before scanning. Correct.
- `stopStepRunsWhere`, a record that fails to decode, or one with outcome `died`: skipped. Correct.
- `stopStepRunsWhere`, no candidates (for example only live-waiter runs): returns before `reed.Status()`, so a reed probe failure goes unreported. Correct, and documented.
- `stopStepRunsWhere`, candidates present and `reed.Status()` fails: returns the empty report with the wrapped error. Correct; nothing has been changed by then.
- `stopStepRunsWhere`, untied run with `listUntied=true`: listed under Untied only when tracked and removable; otherwise `continue`. Matches the old switch. Correct.
- `stopStepRunsWhere`, untied run with `listUntied=false`: never becomes a candidate. Correct.
- `stopStepRunsWhere`, candidate with an untracked strand and a session that may be held: listed under Held. Correct.
- `stopStepRunsWhere`, candidate whose `stopRecorded` fails: listed under Failed and the loop goes on. Correct.
- `stopStepRunsWhere`, candidate whose output files all exist: the record stays `running` and the run is listed under Finished. Correct.
- `stopStepRunsWhere`, waiter attaches between the scan and the stop: waiter liveness is not re-checked before `stopRecorded`. Not correct; see finding 1.
- `StopStepRuns` selector `*rs.Step == tie`: called only when `rs.Step != nil`, so no nil dereference. Correct.
- `StopOrphanedStepRuns` selector, `StepsDir` match with PID 0, -1, a dead pid and a live pid: the first three are selected and the live pid is not. `spec.StepsDir` (`arm.go:441`) and the tie (`sharedbootstrap.go:474`) both come from `shedrun.StepsDir(c.location, c.runID)`, so the strings match. Correct.
- `stepStopNote`, empty report and nil error: returns `""`, so the loop writes no mark. Correct.
- `stepStopNote`, error only: returns "could not stop the step's runs: …". The text is misleading for the reconcile; see finding 2.
- `stepRunsNote`: the same tie as before, rendered through `stepStopNote`. Correct.
- `orphanedStepsNote`: calls `stop(stepsDir)` and renders the result. Correct.
- `stopOrphanedStepRuns`, seam set: delegates to the seam. Correct.
- `stopOrphanedStepRuns`, `stepStopRunner` fails (for example `reedengine.LoadConfig`): the error is returned, rendered into the note and written to the loop log. Not swallowed. Correct.
- `stepStopRunner`, `c.runner` non-nil: returned as is. With `c.runner` nil, it builds a runner the way the old inline code did, returning on each error. Correct.
- `runLoop` reconcile, hook nil: skipped. Hook returns `""`: no mark. Hook returns a note: the mark is written before `stepUntilStop` on every path past `loopStartRefusal`. Correct.
- `loopExitStop`: only a comment changed. Correct.

**Findings**

1. **LOW, design.** `internal/shuttleengine/stepstop.go:113-128` with `internal/shuttleengine/stepstop.go:79`.
   - The orphan selection (`!runIsLive(rs)`) is decided on the record read during the scan and is not re-checked before `stopRecorded`.
   - Case A: a run that another process re-attaches between the scan and the stop is stopped while its new waiter lives. That process could be an arbiter, a tackler or an operator await outside the loop; Attach rewrites the pid at `attach.go:333-338`.
   - Case B: an attach whose pid save failed leaves the old dead pid on record. `attach.go:336-338` only logs that failure and goes on, so the reconcile stops a run whose attaching waiter is alive.
   - Both cases are narrow; the loop lock rules out its own steps, but not attachers from other processes.
   - Evidence: `stepstop.go:79`, `stop.go:87-101` (`stopRecorded` checks only that the outcome is non-terminal) and `attach.go:333-338`.
2. **NIT, consistency.** `internal/loomcli/looparm.go`, `stepStopNote` error branch.
   - The shared renderer words a stop error as "could not stop the step's runs: …".
   - That sentence also ends the reconcile's note, which covers every gone step of the run rather than one step. The loop log then reads "reconciled the steps whose process is gone: could not stop the step's runs: …", which misnames the scope.
   - Evidence: `looparm.go` (the `err != nil` branch of `stepStopNote`) and `loop.go:244-245`.

**Tool calls**

- Read `stepstop.go:88-120` and the body of `stepStopNote` in `looparm.go`: my context had the diff hunks but not the unchanged root-read and NotExist handling, nor the note's full branch list.
- Grepped `StepsDir` assignments in loomcli: to confirm that the steps directory passed to the reconcile matches the one in the step tie, which the manifest had not shown for `arm.go:441`.
