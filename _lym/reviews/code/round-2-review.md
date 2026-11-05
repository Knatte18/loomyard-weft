---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/wiring.go:258
    summary: the refused-Sketch notice's first way forward names `lym board set-deps` with no payload, so it is not a runnable command
    fix: >-
      In sketchRefusedNotice, word the first way forward with a filled-in example, as the other commands already are.
      For instance, drop or move those dependencies with `lym board set-deps '{"slug":"<dependent>","depends_on":[...]}'` for each entry the refusal names, keeping its other dependencies.
      The dependents' slugs and their remaining dependencies are the parent's to choose, so the payload carries a placeholder for the dependent that the refusal text above it names.
      Keep the sketch upsert and the raddle upsert as they are.
      Leave TestSketchFinish_TellsTheParentOnlyOfARefusedSketch unchanged, since its set-deps assertion still holds.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/battencli/cli.go:90
    summary: the package doc says a raddle start clears the motif run before the start guard judges the start, but the code judges first and removes in claimStart after the owner check
    fix: >-
      In the comment block at internal/battencli/cli.go:90, replace the words calls it before the start guard judges the start (startguard.go) with wording that matches startguard.go and arm.go.
      The start guard judges the start as a fresh one first, and the start removes the run once its owner check has passed (claimStart), so a refused start leaves the run in place.
      Keep the remainder of the sentence.
  - id: F3
    severity: LOW
    class: scope
    location: internal/battencli/dobbyreads.go:329
    summary: the branch of DobbyTasks where startClearsMotifRun fails, which holds the start and logs one episode, has no test row
    fix: >-
      Add a row to TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord (internal/battencli/dobbyreading_test.go) for a done run of a raddle entry whose child-recipe record does not read, for example a child-recipe file the test makes unreadable or undecodable, with wantRefused true.
      That row pins that an unreadable judgment holds the start instead of clearing it.
---

### [MEDIUM] The refused-Sketch notice's first way forward is not a runnable command

**Location:** internal/loomcli/wiring.go:258
**Issue:** The notice names two ways forward. Its second, third and fourth commands (`lym board upsert` with the Sketch payload, `lym board upsert` with recipe raddle, `lym board set-status`) carry their payload filled in with the slug. The first, "drop or move those dependencies with `lym board set-deps`", is a bare verb. `set-deps` takes a JSON payload naming the dependent entry and its new `depends_on` list (`internal/boardcli/cli.go:1037`), so the parent cannot run what the line says. The board's own refusal at `internal/boardengine/store.go:389` words the same step the same way, but the notice exists so the parent can act without the board's context, and the board-note's request was for the way forward to be named. The focus asked specifically whether the first way forward is runnable with its payload filled in; it is not.

### [LOW] The battencli package doc contradicts the order of the guard and the removal

**Location:** internal/battencli/cli.go:90
**Issue:** The added sentence says a run or step of a raddle task "calls it before the start guard judges the start". The code does the opposite on purpose: `refuseAtStart` asks `ClearsMotifRun`, which removes nothing, and judges the start as fresh, and `claimStart` runs the owner check and only then calls `ClearMotifRun` (`internal/battencli/arm.go:390-397`, `internal/battencli/startguard.go:435-500`). The doc of `startguard.go` and the decision to leave a refused start's run in place both say so. A reader of the package doc learns the wrong order.

### [LOW] An unreadable start judgment in DobbyTasks is untested

**Location:** internal/battencli/dobbyreads.go:329
**Issue:** The diff adds a branch in `DobbyTasks`: when `startClearsMotifRun` returns an error, `clears` is false, the refusal stays on the view, and `unreadableDoneRunStart.Fail` logs one episode; a successful read calls `Recover`. The rows of `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord` all use readable boards and child-recipe records, so the error path is not reached. `TestClearsMotifRunOfRaddleEntry` pins the error of `clearsMotifRunOfRaddleEntry` (an unreadable board), which is the guard's closure and not the dobby call site's hold.

## Settled focus questions

Each is a verdict on a focus target; none is a finding.

- **Order of the removal (focus 1).** Not a defect. `refuseAtStart` removes and pushes nothing; it only sets `motifRunClearPending` and judges the start as fresh, so the cancel marker, promoted-pair, unreleased, raddle, dirty-source and left-pair checks all run before `claimStart`. `claimStart` then runs the owner check and judges the start again after the pull before it calls `ClearMotifRun`. A refusal by the guard or by the owner check leaves the done run directory in place; `TestRefuseAtStart_JudgesADoneMotifRunAsTheFreshStartItsClearMakes` and `TestClaimStart_RemovesTheDoneMotifRunOnlyOnceTheOwnerCheckPasses` pin both.
- **Recipe resolution (focus 2).** Not a defect. `startClearsMotifRun` reads the task's own recipe when it is a task with one, and otherwise takes `boardRecipe`, the function the raddle refusal in `readRaddleStartRefusal` uses, which is `board.RecipeOf` and the board default. A task's own non-empty recipe is what `RecipeOf` returns, so the two agree. `TestClearsMotifRunOfRaddleEntry` reaches a slug absent from the board, an unreadable board, a non-raddle entry, a raddle entry whose child ran raddle, and the default-recipe rows. A non-task entry reads `boardRecipe`, which gives pawl for a project, so it never clears.
- **Raddle-File exemption (focus 3).** Not a defect. `finalizeEntered` takes `how` and exempts Raddle-File only when `how == endedDone`; `readEndedSet` marks a pruned run done only when the newest commit holding it marks it ended. The integration rows cover an entry removed from origin while the pair stands at Raddle-File (ended as removed and torn down), a done entry pruned (not ended), and a return to Raddle-Write (ended).
- **Sketch finish (focus 4).** A failed notice leaves `Finish` returning nil, and the row `ANoticeThatCannotBeSentStillFinishesTheRun` pins it. A failed draft write returns its error, so the seam retries at the next finished transition, which the doc comment of `sketchFinish` states; no row pins that path, and it is unchanged behavior, so it is no finding. The first way forward is F1.
- **Docs (focus 5).** `DobbyTasks`, `internal/dobbyengine/doc.go`, `internal/loomcli/doc.go` and `internal/fabricengine/doc.go` each state the exception. The `internal/battencli/cli.go` doc is F2.

## Rejected

- The `DobbyTasks` recipe rows repeat recipe cases of `TestClearsMotifRunOfRaddleEntry`; they pin the dobby call site against the guard's closure, so they are not a duplicate test.
