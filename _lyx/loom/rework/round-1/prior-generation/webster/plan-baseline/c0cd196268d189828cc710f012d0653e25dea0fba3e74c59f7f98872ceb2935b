# Card 7 — Shuttle's stop verb records the stop before removing the strand

**Create:**
- `plan:internal/shuttleengine#runOutcomeStopped` -> `const runOutcomeStopped = "stopped"`
- `plan:internal/shuttleengine#stopOutcome` -> `func stopOutcome(outcome string, outputFiles []string) string`
- `plan:internal/shuttleengine#Run.Stop` -> `func (run *Run) Stop() error`
- `plan:internal/shuttleengine#Runner.StopStrand` -> `func (r *Runner) StopStrand(guid string) error`
- `plan:internal/shuttleengine#Runner.stopRecorded` -> `func (r *Runner) stopRecorded(runDir, guid string) error`
- `plan:internal/shuttleengine#removeStrandIfLive` -> `func removeStrandIfLive(reed ReedOps, guid string) error`
- `plan:internal/shuttleengine#Run.saveState` -> `func (run *Run) saveState() error`
- `internal/shuttleengine/stop_test.go`

**Edit:**
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#Run.notifyHeld`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#isTerminalOutcome`
- `internal/shuttleengine#Runner.removeSupersededStrands`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/shuttleengine/attach_test.go`
- `internal/shuttleengine/doc.go`
- `pattern/PATTERN-completion-signal.md`

**Uses:**
- `internal/shuttleengine#FindRun`
- `internal/shuttleengine#saveRunState`
- `internal/shuttleengine#runOutcomeRunning`

**Intent:**
Implement D7's shuttle side and the record write path decided after Discussion.
The verb lives in a new file `internal/shuttleengine/stop.go`.

`runOutcomeStopped` is a record-only terminal outcome: never a `Result.Outcome`.
`stopOutcome` is the one function that chooses the outcome a stop leaves: the outcome it is handed when that is `done` or `allOutputFilesExist(outputFiles)` holds, else `runOutcomeStopped`; it is the constant's single return site, and its `allOutputFilesExist` call guards both writes below (`PATTERN-completion-signal`).

`Run` gains a record mutex and a stop mark.
The mutex is a record-write lock: it guards the mark, `run.state.Outcome` and every save of `run.state`, all of which go through `saveState` (lock, `saveRunState(run.runDir, run.state)`); `finalize`'s outcome write, `notifyHeld`'s `NotifiedOffset` save and `checkLivenessTick`'s `Started` save call it, and the field assignments they make before saving happen under the same lock.

`Run.Stop`, under the mutex: a record already terminal keeps its outcome; otherwise it sets the mark and `run.state.Outcome = stopOutcome(running, outputs)` and saves.
A save that fails restores the previous outcome and mark and returns the error with the strand untouched.
After unlocking it removes the strand through `removeStrandIfLive`, which probes reed, removes a live strand logging at Warn (`PATTERN-spawn-observability`), and returns a probe or removal error; the record stays terminal.
Because the mark is set before the strand is removed, the handle's own `Wait` that sees the strand go reads it: `finalize` stores `stopOutcome(outcome, outputs)` on a marked handle, so a `done` reached through the file contract still wins, and the `Result` it returns keeps the classified outcome.

`Runner.StopStrand(guid)` is the guid form for a caller holding no handle: it refuses a bad told pair as every public method does, resolves the run with `FindRun`, and calls `stopRecorded`, which reads `run.json`, applies the same terminal and file rules, writes it back, then removes the strand; a guid with no run record is stopped by `removeStrandIfLive` alone.
It has no in-process `Wait` to race; a `Wait` in another process may overwrite `stopped` with its own terminal outcome, and either answer is terminal.
`removeSupersededStrands` stops each respawn-eligible candidate's live strand through `stopRecorded` over the candidate's run dir in place of its raw `RemoveStrand`, keeping its way-forward error text.

`isTerminalOutcome` recognises `stopped`, and its doc comment widens from the values finalize wrote to every terminal value a record holds; that is attach's only classification change, so a `stopped` record whose strand is gone is respawn-eligible at any directory age, and attach's file-contract-first branch never sees one.

Shuttle's own teardowns inside the run lifecycle (finalize's cleanup, `abandonStartup`) write the record themselves and stay as they are.

`completionsignal_enforcement_test.go`, in this commit: `stop.go` joins `completionSignalScannedFiles`, `runOutcomeStopped` joins `negativeVerdictMarkers`, `stopOutcome`'s return joins `auditedNegativeVerdictReturns` and its `allOutputFilesExist` call `auditedFileContractCallSites`, and every `Errorf` return in `stop.go` joins the audited set with its justification line.
`pattern/PATTERN-completion-signal.md` and `wait.go`'s file doc comment add `stopped` to their list of negative answers; `doc.go` describes the stop verb and the record lock.

Bound (this card adds a verb that kills a strand): it writes the record first, so a crash between the two leaves a terminal record over a live strand, which attach reads as respawn-eligible and stops as superseded; it never writes `stopped` over a record whose output files all exist.

Tests: `stop_test.go` tables both forms against `fakeReed`: record written before removal (call order), removal failure leaving the record terminal and returning the error, record-write failure returning the error with no removal, a terminal record keeping its outcome, a `running` record with every output file present keeping `running` and losing its strand, a guid with no record removed by strand alone, and a handle stopped during its `Wait` reading `stopped` after `Wait` finalizes with any outcome but `done`.
`attach_test.go`'s `TestAttach_OutcomeDisposition` gains a `stopped`-record row (respawn-eligible at any age, strand gone), `TestAttach_RunningRecordUnsatisfiedFileContract_RespawnsOrErrors` keeps its young-untracked-`running` refusal row, and `TestAttach_RemovesSupersededStrands` asserts the superseded record reads `stopped`.

**ImpactSummary:** Every `run.state` save in `wait.go` and `hold.go` takes the new record lock; attach's superseded removal and `isTerminalOutcome` change, no exported signature changes, and no caller outside shuttle uses the verb yet.

**Commit:** 7: shuttle: a stop verb records stopped before it removes a run's strand
