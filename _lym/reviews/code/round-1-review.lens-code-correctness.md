```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff. I ran concrete cases per function and found no BLOCKING or MEDIUM defects. Two LOW findings come after the table.

**Per-function list (case ran → verdict)**
- `refuseAtStart`, done status, `run`, entry is raddle and child recipe is motif: `ClearMotifRun` returns true, `found=false` and `status={}`, and `refuse` judges a fresh start. Correct.
- `refuseAtStart`, `goto`, a running status, or a nil seam: not offered. Correct.
- `refuseAtStart`, clear returns an error: the start is refused with a way forward. Correct, but see the first finding below.
- `refuseAtStart`, clear returns `(false, nil)`, for example the run lock is held: status stays done, `refuse` runs with `found=true`, and `checkRunStatus` later refuses with `doneSlugRefusal`. Correct.
- `clearMotifRunOfRaddleEntry`, `ReadTasks` fails, slug absent, or recipe not raddle: returns an error, `(false, nil)`, or `(false, nil)` respectively. Correct. `c.location` is already the prime at this point (`arm.go:374-376`), so `startClearsMotifRun` and `ClearMotifRun` read the same run directory.
- `startClearsMotifRun`, `ReadChildRecipe` error: `recipe` is "" so it returns `(false, err)`. Correct. `DobbyTasks` warns and keeps the refusal, and the guard side returns the error.
- `DobbyTasks`: the inner `err` shadow is harmless. A done motif run of a raddle entry sets no `StartRefusal`. A done raddle run of a raddle entry and a done motif run of a darn entry both keep it.
- `clearMotifRun`, commit message changed to `batten:`: nothing reads that string (grep found only the writer).
- `finalizeEnteredRead`, pair status at `Raddle-File`: returns true, so the entry is not treated as ended. A `Raddle-File` history row with current producer `Raddle-Write`: false, so a genuine end still reconciles. An absent pair: falls through to the prime status, unchanged. Correct for the finish case, but see the second finding.
- `sketchFinish`, `MakeSketch` succeeds: returns nil, no notice. Correct.
- `sketchFinish`, other error: returned, so the seam retries. Correct.
- `sketchFinish`, refusal and `SetRunStatus` fails: the error is returned before any notice, so the retry re-refuses, sets draft and notifies once. No duplicate notice.
- `sketchFinish`, refusal and the notify fails: warns, returns nil, `finished` is set, no retry. This matches the stated design.
- `sketchFinish`, refusal and no orch strand recorded: `NotifyParent` returns `(false, nil)`, the notice is only an Info log in orchcli (`notify.go:160`), and `sketchFinish` treats it as sent. Not a defect.
- `sketchRefusedNotice`: the format arguments match the verbs, and the JSON quoting renders as a valid `lym board upsert` payload.

**Findings**

1. LOW, `design`, `internal/battencli/startguard.go:438-455`: the clear commits and pushes the motif run's removal before `guard.refuse` judges the start. If the fresh-start checks then refuse (raddle store collision, dirty source, left pair parts, owned elsewhere), the done motif record is already gone.
   - The refusal also contradicts `startguard.go:421`, which says the checks apply to the fresh start the clear makes.
   - A retry reaches the same checks, so nothing breaks. It is irreversible state change on a refusal path, and the only record of the motif run is lost.

2. LOW, `design`, `internal/fabricengine/replay.go:250`: the Raddle-File exemption applies to every ended verdict, including `removed`. An operator who removes the entry from the board while the pair status is at `Raddle-File` is no longer reconciled as ended.
   - That pair is left for batten to carry to Land and teardown.
   - The pair status stays at `Raddle-File` after the run is done, so the mask lasts until batten's Land or later status takes over (`replay.go:274-278`).
   - The new reconcile test covers only a `done` origin (`endedHub(t, "task")`), not `removed`.

3. LOW, `design`, `internal/battencli/startguard.go:138`: `startClearsMotifRun` tests `task.Recipe` directly, while `readRaddleStartRefusal` uses `boardRecipe(board, entry)`, which calls `board.RecipeOf(task)` (`wire.go:546-551`).
   - An entry whose recipe `RecipeOf` resolves to raddle with an empty `Recipe` field is judged raddle by the guard and not raddle by the clear and by dobby's read.
   - The done motif record then keeps blocking that entry. I did not confirm that `RecipeOf` resolves a default or inherited recipe.

**Tool calls (3)**
1. Read `arm.go:340-415`: to confirm `c.location` is the prime when `refuseAtStart` runs.
2. Grep `boardRecipe`, `withoutFinalizeEntered` callers and the `ResolveBattenPrime` definition: to compare recipe resolution and find the callers of the Raddle-File exemption.
3. Same grep, third pattern: whether any code or test reads the `beam create:` or `batten:` commit message. None does.
