---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/engine.go:23
    summary: Three standing claims that reed no longer tracking a strand is never OutcomeDied are now false for a handle whose own Stop removed the strand, and the diff updated only some of the docs.
    fix: Add the stop-marked exception to each claim. In internal/shuttleengine/engine.go, in the OutcomeDied doc comment (lines 23-28), after the sentence that a strand reed no longer tracks at all is deliberately not this outcome, add that a handle whose own Stop removed the strand is the exception and ends OutcomeDied, which finalize records as stopped. In internal/shuttleengine/wait.go lines 5-6 of the file header, qualify that died is reserved for a still-tracked strand and that an untracked strand is a mechanism failure with the same exception. In docs/overview.md line 420, qualify the sentence that reed's strand table losing the strand entirely is reported as a mechanism failure, never as died, with the same exception. Word all three to match the Wait doc comment at wait.go:226-227.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/stop_test.go:282
    summary: No test pins that the output-files check runs before the markedStopped branch, so swapping the two in checkLivenessTick would record finished work as stopped and no test would fail.
    fix: Turn TestStop_WaitThatSeesTheStoppedStrandGoEndsDied into a table-driven test and add a row where the output file exists before Wait (written into runDir after Stop), expecting Wait to return OutcomeDone with a nil error. Keep the current missing-file case as the other row. Do not add a separate test function.
---

### [MEDIUM] Stale never-died claims in engine.go, wait.go header and docs/overview.md

**Location:** internal/shuttleengine/engine.go:23, internal/shuttleengine/wait.go:5-6, docs/overview.md:420
**Issue:** The diff makes an untracked strand end `OutcomeDied` when the handle's own `Stop` removed it.
It records that in the `Wait` and `checkLivenessTick` doc comments and in `doc.go`.
Three other statements still say the opposite without an exception.
The `OutcomeDied` constant's doc at `engine.go:23-28` says a strand reed no longer tracks is deliberately not this outcome.
The `wait.go` file header at lines 5-6 says `died` is reserved for a strand reed still tracks.
`docs/overview.md:420` says reed's table losing the strand is reported as a mechanism failure, never as `died`.
A reader of the exported constant or the overview gets a contract the code no longer keeps.
I read `engine.go:20-28` and `docs/overview.md:420` to confirm the wording.

### [LOW] Ordering of the file-contract check and the stop branch is unpinned

**Location:** internal/shuttleengine/stop_test.go:282
**Issue:** The new branch sits after `allOutputFilesExist` on purpose, so finished work stays `OutcomeDone`.
`TestStop_WaitThatSeesTheStoppedStrandGoEndsDied` seeds only a missing output file, and `TestStop_MarkedHandleFinalizesStoppedUnlessDone` calls `finalize` directly, so neither reaches `checkLivenessTick` with the files present.
Moving the stop branch above the file check would turn a finished run into a `stopped` record with every test green.
The completion-signal tripwire counts return sites, not their order.

## Rejected

- Lens code-correctness, LOW: `stopMarked` stays set when `Stop`'s removal fails or does nothing, so a strand lost later for another reason reads as the stop. `Stop`'s own doc and the package doc already state that a Wait on a stopped handle finalizes with the stop mark, and the record already reads stopped from the caller's request. The branch adds no new contract beyond that one, and the strand cannot be told apart from reed's side.
- Focus question 1 (scope against the board entry): the diff addresses the cause. `seatengine.join` calls `stopSeat(chair)` when a writer's end fires the step ender (`internal/seatengine/engine.go:384-396`), which removes the chair's strand through `Stop`, and the chair's `Wait` then failed on `errStrandNotTracked`. The chair is stopped on purpose, so keeping it tracked would be wrong; the defect was the stopped handle's `Wait` misreading its own stop. The summary's claim that the producer then maps the writer's exit through `mapOutcome` and `writerExit` matches `internal/shedadapters/multillm.go:148,210`.
- Focus question 2 (masking): the new branch runs after `allOutputFilesExist`, returns `OutcomeDied` directly rather than through the retry counter, and is reached only for a handle where `Stop` saved the record first. A mark set while the strand is live changes nothing, since a tracked strand never reaches the branch. `awaitStartup` also calls `checkLivenessTick`, but no `Stop` can reach a handle before `Start` issues it.
- Focus question 3 (lock): `markedStopped` takes `recordMu` for one field read. `Stop` releases the lock before `removeStrandIfPresent`, and `finalize` holds it only around the record save, so no path nests it and there is no deadlock. `stopMarked` is set only inside the non-terminal branch and restored when the save fails.
- Focus question 4 (duplication): the new test is the only one driving `Wait` through the liveness path on a marked handle, so it duplicates neither `TestStop_MarkedHandleFinalizesStoppedUnlessDone` nor the step-stop test.
- Focus question 5 (`completionsignal_enforcement_test.go`): its lines changed because the audited map and its comment count `checkLivenessTick` `OutcomeDied` returns, and the diff adds a third behind `allOutputFilesExist`. The change is the tripwire doing its job and is licensed by the new return site.
- Focus question 6 (`doc.go` and `wait.go` comments): the new sentences match the code (`wait.go:1267-1272`, `wait.go:1796`). The stale statements elsewhere are filed as F1.
- Pattern fit: the diff changes no `PATTERN.md` or `pattern/` file, the directive declares none, and the commit message needs no `Pattern-Departure:` trailer. The Completion Signal rule holds, since the new `OutcomeDied` return follows `allOutputFilesExist`.
