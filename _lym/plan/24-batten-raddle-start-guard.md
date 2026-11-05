# Card 24 — batten-raddle-start-guard

**Create:**
- `plan:internal/battencli#raddleStartRefusal` -> `func raddleStartRefusal(entry boardengine.Task, recordStored bool, store boardengine.RaddleStore) error`
- `plan:internal/battencli#TestRaddleStartRefusal` -> `func TestRaddleStartRefusal(t *testing.T)`

**Edit:**
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#TestStartGuard_Refuse`

**Uses:**
- `internal/battencli#boardRecipe`
- `internal/boardengine#ReadTasks`
- `plan:internal/boardengine#RaddleStore.Holds`
- `internal/boardengine#DecisionFileName`
- `plan:internal/shedrun#RecipeRaddle`

**Intent:** D2's start guard, batten's own check beside the board's refusal at promotion (card 14).
`startGuard` gains `Raddle func() error`, nil checking nothing as every seam of a hand-populated receiver.
`hubStartGuard` fills it: it reads the slug's entry and, when the entry's recipe (`boardRecipe`) is `raddle`, calls `raddleStartRefusal` with whether the entry is flagged discussed with its record file in the board directory, and the store from `ReadRaddleStore`.
`raddleStartRefusal` refuses:
- an entry with no stored record: "%s runs the raddle recipe, which splits a Sketch's stored decision record, and it holds none; way forward: run its Discussion first with a motif run: `lym board upsert '{\"slug\":\"%s\",\"recipe\":\"motif\"}'`, release it with `lym board set-status '{\"slug\":\"%s\",\"status\":null}'`, and once the motif run has made it a Sketch, `lym board promote %s --recipe raddle`";
- an entry whose slug the store holds as another Sketch's, as its own Sketch slug under another recorded `created` stamp, or as a slug any stored proposal filed: naming `lym board rename` of the entry to a slug the store does not hold, one it names, as the way forward.
It passes the entry whose slug the store holds as its own Sketch slug with the entry's own `created` stamp, so a resumed raddle run past its filing starts.
`refuse` runs it for a run or step with no status file, after the unreleased check, wrapping a probe error with `probeRefusal`.
The reaching inputs: `lym batten run <slug>` of a raddle task whose entry is not discussed (a task upserted with recipe `raddle` by hand, which the label rule admits only with no `sketch` label and no stored record); and of a raddle task whose slug the store holds under another `created` stamp or as a filed slug, which a board commit replayed from a machine that had not pulled the store can leave (card 14's bound).
`TestRaddleStartRefusal` (tier0, fixture entries and stores) drives `raddleStartRefusal` itself: an entry with no record (refused naming the motif upsert and the raddle promote); a store holding the slug as another Sketch's (refused naming `lym board rename`); the slug as its own Sketch under another `created` stamp (refused the same); the slug as a filed slug (refused the same); the slug as its own Sketch with its own stamp (passes); a store not holding it (passes).
`TestStartGuard_Refuse` gains rows for the plumbing alone: a `Raddle` seam refusing a run with no status file (its refusal returned), the same with a status file (not judged), and a `Raddle` probe error (wrapped by `probeRefusal`).

**ImpactSummary:** One new seam in batten's pre-run guard, judged only for an entry whose recipe is raddle; every other recipe's start is unchanged.
