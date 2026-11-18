---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shedverbs/loop.go:471
    summary: The re-step reconcile looks for a kind-interrupted step record, but the dead-loop stop (cause loop-exited, the case issue 138 reports) writes none, so a failed or unreported stop of the dead loop's in-flight step is never retried and the re-step reconciles nothing or an older unrelated trace.
    fix: In internal/shedverbs/loopwait.go deadLoopStop, when rec.hasChild() and rec.TraceID is non-empty and no record exists yet at <StepsDir>/<rec.TraceID>.json, write one through interruptedRecord(spec, rec.TraceID, <the interrupted message and way forward loopExitStop writes>, <the child's first trace file or the loop log>, friction) before the envelope is delivered and the pid file retired, so newestInterruptedTraceID in internal/shedverbs/loop.go finds the in-flight child's trace id on the re-step. Add a row to TestLoop_ReconcilesTheInterruptedStepBeforeTheFirstChild in internal/shedverbs/loop_test.go, or a case in TestUntilStop_DeadLoopStopsLoopExited, that runs the dead-loop stop with an OnStepOrphaned hook returning a failure note and then a loop start on the resulting status, asserting the hook gets the dead loop's in-flight trace id before the first child.
    carries: F7
    origin: handler
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/stepstop.go:135
    summary: sessionMayBeHeld trusts the engine's SessionHolder answer, and Claude's holder lookup skips unreadable and undecodable registry entries and reports no holder, so a run whose live holder sits behind such an entry is recorded stopped and Attach then respawns beside it at any age.
    fix: In internal/shuttleengine/claudeengine/resume.go sessionHolder, return an error naming the entry when a registry record has a ReadErr or DecodeErr, instead of skipping it, so sessionMayBeHeld in internal/shuttleengine/stepstop.go reads the run as held and lists it under Held; relaunchForReprompt already refuses on a holder-lookup error. Add a row to the claudeengine holder test with an unreadable entry, asserting the error, and a row to TestRunner_StopStepRuns_StopsOnlyTheStepsRuns whose fake engine's SessionHolder fails, asserting the run is listed under Held and keeps reading running.
    carries: F5
    origin: assist:assist
prior:
  - id: F1
    status: resolved
    answer: relaunchForReprompt records the events size before Prepare and sets run.startupEventsOffset; checkLivenessTick's eventsProof branch and sessionStartRecorded read past it, and the never-ready row seeds an earlier session start and turn start.
  - id: F2
    status: resolved
    answer: The spec keeps Skills and sets SkipSkillLoadTurn; both engines' Prepare keep the pointer on the launch line while scope, settings and passages still come from Skills.
  - id: F3
    status: resolved
    answer: copyManifest runs after the save that names the new strand, as Runner.launch does.
  - id: F4
    status: resolved
    answer: The explicit removal now follows the save naming the new strand; reed's own dead-name reclaim inside AddStrand removes only a strand whose engine is dead, which is judged under Rejected.
  - id: F5
    status: open
    answer: The Held guard landed as the fix asked, but Claude's holder lookup reports no holder when the holder's registry entry is unreadable, so the guard still lets that run be stopped; carried at LOW as F2.
  - id: F6
    status: resolved
    answer: A non-empty checkResume warning returns an error before Prepare, and the row asserts it in EndedEarly with no strand added.
  - id: F7
    status: open
    answer: The fixed-differently route works for an interrupted child of a live loop, whose record runLoop writes, but not for the dead-loop stop, which writes no interrupted step record; carried as F1.
  - id: F8
    status: resolved
    answer: The table holds the dead pane, cleared binding, never-ready, holder lookup, holder present, no-holder engine, checkResume error and warning, Prepare, AddStrand and record-save rows; the cleared binding is not removable, so expecting no removal is right.
  - id: F9
    status: resolved
    answer: The panic row panics through Loop.Now after the child is recorded and asserts the hook's trace id and the cause loop-panicked boom followed by the note.
  - id: F10
    status: resolved
    answer: run.state is copied before the assignments and restored when saveState fails, before the new strand is removed.
  - id: F11
    status: resolved
    answer: The replacement of prompt.md is documented in relaunchForReprompt's doc, doc.go's gate paragraph and the summary.
  - id: F12
    status: resolved
    answer: StepStopReport.Finished lists a run whose record still reads running after the stop; stepRunsNote renders it and the stop test has a finished row.
  - id: F13
    status: resolved
    answer: The summary now says Attach refuses a young untracked strand and allows it past minAge, and that the holder check rules out a live writer.
  - id: F14
    status: resolved
    answer: The stopRuns seam lets the specFor subtest call OnStepOrphaned and assert the steps directory and trace id it reaches.
  - id: F15
    status: resolved
    answer: The empty-note row asserts the cause carries no trailing separator.
---

### [MEDIUM] The re-step reconcile misses the dead-loop stop's in-flight step

**Location:** internal/shedverbs/loop.go:471

**Issue:** `reconcileInterruptedStep` passes the status check for a loop-exited stop, since `loopExitStop` writes the status error through `interruptedError`.
It then takes the trace id from `newestInterruptedTraceID`, which reads only records whose top-level `kind` is `interrupted` with a `trace_id`.
The only writer of such a record is `interruptedRecord`, called from `runLoop` at `internal/shedverbs/loop.go:296` for a child of a live loop.
`deadLoopStop` in `internal/shedverbs/loopwait.go:554` writes the failed status and the loop envelope file, which nests the envelope under `envelope` and carries no step trace id, and then retires the pid file that held `rec.TraceID`.
So after the dead-loop stop of issue 138, when its `OnStepOrphaned` call at `loopwait.go:584` failed or a run stayed held, the re-step finds no record for that step.
It then reconciles nothing, or the trace id of an older, unrelated interrupted child, and the leftover run meets the re-step as before.
The board entry asks for this reconcile on the re-step, and the summary claims "a stop that failed at the interruption is retried by the re-step".
The test seeds the interrupted record by hand, so it never runs the dead-loop path.

### [LOW] An unreadable registry entry hides a live holder from the step stop

**Location:** internal/shuttleengine/stepstop.go:135

**Issue:** `sessionMayBeHeld` reads the run as free when the engine's `SessionHolder` finds no holder.
Claude's `sessionHolder` in `internal/shuttleengine/claudeengine/resume.go` skips a registry record with a read or decode error and returns no holder, while its resume check turns the same record into a warning.
The resume path refuses on that warning (prior F6), but the step stop asks only `SessionHolder`.
A tied run whose strand is untracked or has a cleared binding is then recorded stopped, and `leftoverThenAgeVerdict` in `internal/shuttleengine/attach.go:595` makes a terminal record respawn-eligible at any age.
That is the hazard prior F5 named, now narrowed to an unreadable entry.

## Rejected

- Assistant F1, reed reclaims the old strand inside AddStrand before the record names the new one: `reclaimDeadNameLocked` in `internal/reedengine/strand.go` removes the name's holder only when its engine reads dead, so no live agent loses its strand. A failed save restores the record and ends the loop, and the run then finalizes. A kill in that window leaves the record on an untracked strand, the state the untracked branch starts from, next to the new orphan strand of launch's documented two-phase window. The explicit removal the fix asked to reorder now follows the save, so prior F4 is resolved.
- Assistant F3 repeats this review's F1 and is merged into it.

## Focus verdicts

- The resumed startup and the reused events file: not a defect. The offset is the file's size before `Prepare`. Neither engine's `Prepare` truncates `events.jsonl`, and both readers parse only past the offset.
- Settings, scope and passages of the resumed session: not a defect. `Skills` stay in the spec and only the skill-load turn is skipped.
- Manifest copy: consistent after a successful resume. A resume that fails after `Prepare` leaves the run directory's `manifest.json` describing the refused launch while the copy keeps the first one. This finding is below this round's BLOCKING-only threshold for new findings and is not raised.
- Record names an existing strand: holds at each step the resume itself takes. Reed's dead-name reclaim is judged under Rejected.
- `StopStepRuns` against `leftoverThenAgeVerdict`: holds behind the holder check, except for the unreadable-entry gap in F2.
- The checkResume warning: refuses the resume, so it is not a defect.
- The re-step reconcile after a failed stop: works for a live loop's interrupted child and fails for the dead-loop stop (F1).
- Test gaps: every branch the focus lists has a row. The only branch without a test is relaunchForReprompt's guard for a run that records no session, which is below this round's threshold.
