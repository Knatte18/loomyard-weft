---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/battencli/startguard.go:438
    summary: A raddle start removes and pushes the done motif run before the owner check and the guard's refusals, so a start another machine owns or a guard refuses still destroys the run record
    fix: >-
      In refuseAtStart (startguard.go:424-456) stop calling guard.ClearMotifRun before the guard judges the start.
      Add a read-only guard seam (for example ClearsMotifRun, built over startClearsMotifRun and the board entry of the slug) and call it when the status is done and the verb is run or step.
      When it says the run would be cleared, call guard.refuse with found=false and a zero status, so the raddle, unreleased, dirty-source and left-pair checks still judge the fresh start, and record the pending clear on the battenCLI receiver.
      In arm.go (the block at lines 395-406) make the owner check run when the status file is absent or a clear is pending, and perform the removal through ClearMotifRun only after refuseStartOwnedElsewhere passes and before armSeed, returning the existing way-forward error when it fails.
      Extend TestRefuseAtStart_ClearsADoneMotifRunBeforeJudgingTheStart, and add a row for an owner check that refuses, so a start that refuses leaves the done run directory in place.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/startguard.go:146
    summary: clearMotifRunOfRaddleEntry, the closure that gates the clear on the board entry, has no test
    fix: >-
      Add a tier0 table-driven test in startguard_test.go over a fixture board and a fixture run directory that calls clearMotifRunOfRaddleEntry (or the hubStartGuard closure) with the same fakes the cleanup tests use.
      Rows (each asserting cleared and whether the run directory is gone) are a slug absent from the board, a board that does not read, an entry of recipe darn with a done motif run, a raddle entry whose done run's child ran raddle, and a raddle entry whose done run's child ran motif, which is removed.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/replay.go:250
    summary: The Raddle-File exemption holds for every ended verdict, so an entry removed from origin's board while the pair stands at Raddle-File is no longer reconciled as ended
    fix: >-
      In withoutFinalizeEntered (replay.go:310-323) pass the verdict how to finalizeEntered, and in finalizeEnteredRead count a current producer Raddle-File only when how is endedDone.
      A removed entry then reconciles as ended at Raddle-File as before.
      Add a row to TestReconcileOrigin_Ended in originreconcileended_integration_test.go for a pair at Raddle-File whose origin removed the entry, expecting one teardown call, and update the finalizeEntered godoc and the fabricengine doc.go paragraph to say the exemption covers a done entry only.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/battencli/startguard.go:138
    summary: startClearsMotifRun reads task.Recipe while the guard judges a raddle entry through the board's recipe resolution, so an entry whose recipe resolves to raddle by the board default keeps its done motif record
    fix: >-
      Judge the recipe in startClearsMotifRun the way readRaddleStartRefusal does, through boardRecipe(board, task) with a board opened by boardengine.OpenHub on the hub, instead of task.Recipe.
      Give DobbyTasks and clearMotifRunOfRaddleEntry the same resolved recipe, and add a row to the dobby and the guard tests for an entry with an empty recipe field under a board whose default recipe is raddle.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:246
    summary: The new failure paths of sketchFinish, a notice that cannot be sent and a draft write that fails before the notice, have no test row
    fix: >-
      In TestSketchFinish_TellsTheParentOnlyOfARefusedSketch (commitstatus_integration_test.go) give the table a notify field and add a row for a refused Sketch whose notify returns an error, expecting Finish to return nil after the one attempted notice.
      The draft-write failure needs no row beyond that unless a seam for SetRunStatus already exists.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/battencli/dobbyreads.go:287
    summary: The DobbyTasks godoc and dobbyengine/doc.go still say a released task with a finished run record always carries the start refusal, which no longer holds for a done motif run of a raddle task
    fix: >-
      In the DobbyTasks godoc (dobbyreads.go:287) and in internal/dobbyengine/doc.go:109 add the exception that a done motif run of a task with recipe raddle carries no refusal, since its start clears that run.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/doc.go:290
    summary: The loomcli doc says a run origin ended is paused unless its parent holds its work, but a raddle run standing at Raddle-File is also not treated as ended
    fix: >-
      In internal/loomcli/doc.go:290 add that a raddle run standing at Raddle-File is not read as ended either, since its own Raddle-File row ended the entry and Land and Worktree-Teardown are still to come.
  - id: F8
    severity: LOW
    class: design
    origin: handler
    location: internal/loomcli/wiring.go:258
    summary: The notice's first way forward says to make the entry a Sketch with lym board upsert and gives no payload, so the parent cannot run it as typed
    fix: >-
      In sketchRefusedNotice (wiring.go:256-259) spell the command that makes the entry a Sketch with its payload filled in for slug, taken from the hand-write route boardengine/doc.go:251 and :315-317 describe, as the raddle way forward already does.
      Keep the test's wanted fragments in step, and have it assert that fragment.
---

### [BLOCKING] A raddle start destroys the done motif run before the owner check and the guard's refusals

**Location:** internal/battencli/startguard.go:438 (the call site is arm.go:391, the owner check arm.go:396-401)
**Issue:** `refuseAtStart` calls `guard.ClearMotifRun` as its first act over a done status.
`ClearMotifRun` removes the run directory and scratch, commits the removal to the records branch and spawns its push.
That happens before `guard.refuse` judges the start and before `refuseStartOwnedElsewhere` runs, which the code at arm.go:395 places first so that "a task another machine owns leaves no seed behind".
A `lym batten run <slug>` on a machine that does not own the raddle entry therefore removes and pushes the motif run's records, then refuses on ownership because the cleared status file is now absent.
The same holds for a dirty source, a raddle store collision, a left pair part and a cancel marker, all judged after the clear.
`PATTERN-owner-machine` allows starting a run only on the entry's owner machine, and this change writes a run's records on any machine.
The `refuseAtStart` godoc and the summary both say the guard's checks judge "the fresh start the clear makes", but the clear lands first and stays when they refuse.

### [MEDIUM] clearMotifRunOfRaddleEntry has no test

**Location:** internal/battencli/startguard.go:146
**Issue:** The closure wired into `hubStartGuard` is the one place that decides, from the board, whether a start clears the run, and it calls `ClearMotifRun`.
`TestRefuseAtStart_ClearsADoneMotifRunBeforeJudgingTheStart` fakes the seam, and `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord` reaches only `startClearsMotifRun`.
If the closure never cleared, read the wrong board or missed the slug, no test fails, though it is the whole fix for raddle-same-slug-completed.

### [LOW] The Raddle-File exemption ignores how origin ended the entry

**Location:** internal/fabricengine/replay.go:250
**Issue:** The decision asks that a raddle run stop being ended by its own entry's done mark.
The new condition keys on the pair's current producer alone, so it also exempts an entry removed from origin's board while the pair stands at Raddle-File, a real end that reconciled as ended before the diff.
The pair status stays at Raddle-File after the run is done, so the exemption lasts until batten's Land or later status takes over, and the new test covers only a done origin.

### [LOW] startClearsMotifRun and the guard resolve the raddle recipe differently

**Location:** internal/battencli/startguard.go:138
**Issue:** `readRaddleStartRefusal` judges a raddle entry through `boardRecipe`, which calls `Board.RecipeOf`, the entry's own recipe or the board's default recipe.
`startClearsMotifRun` tests the `Recipe` field directly, so an entry that is raddle only by the board default is held as raddle by the guard and not cleared, and dobby also holds its start with the done-run refusal.

### [LOW] The new failure paths of sketchFinish have no test row

**Location:** internal/loomcli/wiring.go:246
**Issue:** The new test passes a `notify` that always succeeds.
A regression that returned a failed notice's error would hold the run's finish and make the next transition retry the whole Sketch write, and no test fails.
A failing draft write before the notice is likewise unpinned.

### [LOW] The done-run refusal is documented without its new exception

**Location:** internal/battencli/dobbyreads.go:287
**Issue:** The `DobbyTasks` godoc and `internal/dobbyengine/doc.go:109` state that a released task whose slug has a finished run record carries the start refusal.
`DobbyTasks` now sets none for a done motif run of a raddle task (dobbyreads.go:325-333).

### [LOW] The loomcli doc leaves the Raddle-File exemption out of the ended-origin rule

**Location:** internal/loomcli/doc.go:290
**Issue:** The sentence says a run origin ended is paused unless its parent already holds its work.
`finalizeEntered` now also exempts a raddle run at Raddle-File, which never has parent work, so the ended set the pre-push check reads no longer holds that run.
The diff updated the package doc's finish paragraph at lines 25-28 and the fabricengine doc, not this one.

### [LOW] The first way forward in the refused-Sketch notice cannot be run as typed

**Location:** internal/loomcli/wiring.go:258
**Issue:** The notice tells the parent to make the entry a Sketch "with `lym board upsert`" and gives no payload.
Its other way forward and every neighbouring notice spell the command with the slug filled in.
The decision record asks the mail to name the way forward, and the parent has to find the payload itself for this one.

## Rejected

- The guard godoc at internal/loomcli/prepush.go:113 as stale (lens:code-affected-set): it describes what the guard does with an ended report, which stays true, and the exemption sits in the reconcile that builds the report, so the stale statement is the one in loomcli/doc.go, kept as F7.
- The new `raddleFileRow` constant repeating `loomshed.NameRaddleFile` (lens:code-affected-set, NIT): it joins `finalizeRow`, `landRow` and `worktreeTeardownRow` in the existing const block, and fabricengine cannot import loomshed under PATTERN-fabric-boundary, so no new departure from the repository's rules.
- No row for an unreadable child recipe in `DobbyTasks` (lens:code-tests, LOW): the branch only warns and keeps the start refusal as before the diff, and a file-read error cannot be produced at tier0 without permission tricks.
- `sketchFinish` treating `NotifyParent`'s `posted=false` with no error as sent (lens:code-correctness): orchcli logs the unposted notice at Info and the run's Warn for the refusal is kept, so nothing is lost.
- The `child_done_batten_halted` hint disagreeing with the Run-Shed refusal (focus directive): the hint code is unchanged, and once the Raddle-File exemption holds `lym batten run` does what the hint says, so the two agree; the exemption's reach is F3.
