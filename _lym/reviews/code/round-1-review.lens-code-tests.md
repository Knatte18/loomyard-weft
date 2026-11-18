```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test row that pins it.

**Behaviour → pinning test**

1. `stepRunsNote` renders stopped, untied and failed runs and the stop error, and both hooks pass `{StepsDir, TraceID}` → `internal/loomcli/looparm_test.go` `TestStepRunsNote_StopsTheStepsRunsAndNamesTheRest`, rows × `WatchdogKill`/`OrphanedStep`. Pinned.
2. `specFor` wires `spec.Loop.OnStepOrphaned` to `orphanedStepNote(c.stopStepRuns, spec.StepsDir)` (`internal/loomcli/arm.go:445`) → none. No loomcli test sets or reads `OnStepOrphaned` or `OnWatchdogKill`.
3. `runChild`: a child that exits with no envelope calls `OnStepOrphaned(traceID)` and its note ends the cause → `loop_test.go` `TestLoop_InterruptedChildFillsTheErrorForm/child ends without an envelope`. It asserts the trace id, the cause text and the `AfterInterrupt` cause. Pinned.
4. `runChild`: no call when the watchdog killed the child or the child left a record → `TestLoop_WatchdogKillsAnIdleChild` (`len(orphaned) != 0`). Pinned.
5. `runChild` / `deadLoopStop` / panic path: an empty note is skipped and adds no `"; "` → none. Every fake hook returns a non-empty note.
6. `stepUntilStop` panic recovery calls `OnStepOrphaned(pid.TraceID)` for an in-flight child and appends the note → none. The panic row (`loop_test.go:466`) sets no hook, and its panic fires inside the runner before a child is recorded, so `pid.hasChild()` is false either way.
7. `deadLoopStop` calls the hook only with a child in flight under a recorded trace id, and appends its note to the detail → `loopwait_test.go` `TestUntilStop_DeadLoopStopsLoopExited`. The rows vary `traceID`/`noChild` and check `wantOrphaned` and `wantCause`. Pinned.
8. `StopStepRuns` stops a tied running run whose strand is a dead window-0 one or untracked, records it stopped and removes nothing → `stop_test.go` `TestRunner_StopStepRuns_StopsOnlyTheStepsRuns`. It uses the `dead-window0` and `untracked` seeds, the `Stopped` list, `RemoveStrand` calls == 2 and the outcome map. Pinned.
9. `StopStepRuns` keeps listing an untied run only when its strand is removable → partly. The live untied row is listed. No row has an untied run with a dead or untracked strand, but this behaviour did not change, so no finding.
10. `liveStrand` wraps `errStrandGone` for its three branches (untracked, cleared pane binding, dead pane) → only the untracked branch is reached, through `TestGate_RepromptToGoneStrand` (`StatusResult{}`). Nothing pins the cleared-binding and dead-pane branches as triggering the resume.
11. `relaunchForReprompt` success: the session resumes in a fresh strand with no done-when list, the re-prompt is the prompt, skills are nil, the record names the new strand, and the resume counts as one attempt → `TestGate_RepromptToGoneStrand/a resumable session with no holder resumes`. Pinned.
12. `relaunchForReprompt` with a live holder, and with an engine that is not a `SessionHolderFinder`: the loop ends naming both failures, with no `AddStrand` → the `TestGate_RepromptToGoneStrand` rows `a live holder keeps the session` and `an engine that cannot name a holder`. Pinned.
13. `removeGoneStrand` removes a strand reed still tracks (dead pane or cleared binding) before the new strand is added → none. The only resume row uses an untracked strand, so this removal branch never runs.
14. `relaunchForReprompt`: a startup that never becomes ready returns `finished=true`, and `handleGatedBoundary` returns the teardown's result → none.
15. `relaunchForReprompt` failure paths that end the loop, each naming both failures: empty `SessionID`, `SessionHolder` error, `checkResume` refusal, `Prepare` error, `AddStrand` error, and a failed save that removes the new strand → none. The failed save that removes the new strand is a state cleanup.
16. `strandAddSpec` extraction from `launch` → covered by the existing launch tests and by `TestGate` asserting `AddStrandCalls[0].SessionID`. Pinned.

**Findings**

- **MEDIUM, design.** `internal/shuttleengine/relaunch.go:271` (`removeGoneStrand`) and `run.go` `liveStrand`'s cleared-binding and dead-pane branches. The resume after a strand reed still tracks (dead pane or cleared pane binding) is untested: no row checks that the old strand is removed before the fresh one is added, nor that those two `liveStrand` errors wrap `errStrandGone` and reach the resume. Evidence: `gate_test.go` diff lines 70 and 99, where the only status queue is `notTracked`.
- **MEDIUM, design.** `internal/shuttleengine/relaunch.go:262-264`. A resumed provider that never becomes ready (`awaitStartup` error → `finished=true`, which `handleGatedBoundary` returns as is at `wait.go` near line 612) has no test row. Evidence: the `TestGate_RepromptToGoneStrand` rows are only success, holder and non-resuming engine.
- **LOW, design.** `internal/shedverbs/loop.go:376-380`. The panic path's `OnStepOrphaned` call for an in-flight child is untested. Evidence: `loop_test.go:466-478`, where the panic row sets no hook and checks the cause `loop-panicked: boom` only.
- **LOW, design.** `internal/shuttleengine/relaunch.go:207-260`. The resume's refusal and failure paths before the strand registers have no rows: no session id, holder lookup error, `checkResume` refusal, `Prepare` error, `AddStrand` error. The most important of these is the failed record save that removes the new strand (`relaunch.go:254-259`), since it is a cleanup that keeps an unrecorded strand from leaking. Evidence: the `TestGate_RepromptToGoneStrand` rows.
- **LOW, design.** `internal/loomcli/arm.go:445`. Wiring `OnStepOrphaned` into the loop spec is not pinned: dropping the line passes every test. Evidence: no loomcli test references `OnStepOrphaned` (grep of `internal/loomcli/*_test.go`).
- **NIT, design.** `internal/shedverbs/loop.go:609-613` and `loopwait.go:583-587`. The "an empty note is skipped" contract (`LoopSpec.OnStepOrphaned` doc, `loop.go:92`) has no row, because every fake hook returns a non-empty note. Evidence: `loop_test.go` diff line 109, `loopwait_test.go` diff line 190.

**Tool calls**

1. One grep over `shedverbs/loop_test.go`, `loomcli/*_test.go`, `shuttleengine/stop_test.go` and `shuttleengine/gate_test.go`. My context held only the diff hunks, not the panic row, the loomcli wiring tests, the strand seeds or the gone-strand variants.
