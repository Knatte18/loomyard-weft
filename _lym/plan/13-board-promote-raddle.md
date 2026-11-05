# Card 13 — board-promote-raddle

**Create:**
- `plan:internal/boardengine#SketchRecipe` -> `const SketchRecipe = "raddle"`
- `plan:internal/boardengine#Board.PromoteTo` -> `func (b *Board) PromoteTo(idOrSlug any, recipe string) (Task, error)`
- `plan:internal/boardengine#Store.promoteTo` -> `func (s *Store) promoteTo(idOrSlug any, recipe string, recordStored func(slug string) bool) (task Task, changed bool, err error)`
- `plan:internal/boardengine#checkDiscussedPromotion` -> `func checkDiscussedPromotion(loaded, after []Task, recordStored func(slug string) bool) error`
- `plan:internal/boardengine_test#TestDiscussedPromotion` -> `func TestDiscussedPromotion(t *testing.T)`

**Edit:**
- `internal/boardengine#Board.Promote`
- `internal/boardengine#Store.Promote`
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#applyDraftRule` -> `func applyDraftRule(loaded, after []Task, released []string)`
- `internal/boardengine#validateLabels`
- `internal/boardengine#TestValidateLabels`
- `internal/boardengine/doc.go`
- `internal/boardcli#Command`
- `internal/boardcli_test#TestCLI`
- `internal/boardcli/doc.go`

**Uses:**
- `internal/boardengine#IsSketch`
- `internal/boardengine#DecisionFileName`
- `internal/boardengine#SketchType`
- `internal/boardengine_test#TestDraftStatus`

**Intent:** D2's board half: a Sketch becomes a task only as a raddle run's entry, released in the promoting write.
`SketchRecipe` names the one recipe that may hold a Sketch's record and label on a task.
`Board.PromoteTo` is `Promote` with a recipe set in the same write as the kind; `Board.Promote` becomes `PromoteTo(idOrSlug, "")` and `Store.Promote` `promoteTo(idOrSlug, "", nil)`, so their callers and tests stay as they are.
`promoteTo` with a recipe sets the entry's `recipe`; promoting a discussed note whose record file exists (`recordStored`, a stat of `DecisionFileName` in the board directory) with recipe `raddle` keeps the record and the flag and adds the `sketch` label first when a note write dropped it, so the label is restored.
`applyDraftRule` gains `released`, the slugs of new tasks it leaves with an empty status; every other note or project made a task, and every new task, reads draft as today.
`boardCriticalSection` passes the promoted slug as released when the `promoted` result carries recipe `raddle`, and none otherwise, so the promoted raddle task is released in its own write and its parent's dobby starts the run; every other caller passes none.
`checkDiscussedPromotion` runs in `boardCriticalSection` after `checkRunLocks` and before `checkNewSlugs`, over every write: an entry that was a note in `loaded`, flagged discussed with its record stored, and is a task in `after` must carry recipe `raddle`, the `discussed` flag and the `sketch` label, else the write is refused with "entry %q is a discussed note with a stored decision record, which becomes tasks only through a raddle run; way forward: `lym board promote <slug> --recipe raddle`", whether the note carried the label or not.
The rule keys on the flag and the stored record, never on the label, and admits exactly one recipe; a discussed note that never became a Sketch falls under it by intent.
Its reaching inputs: a plain `lym board promote` of a discussed note a note write stripped of its `sketch` label, and a `lym board upsert` setting `"kind":"task"` and a recipe other than `raddle` on such a note.
`validateLabels` refuses the `sketch` type label on a task whose recipe is not `raddle` and on a project, naming the label and, as the way forward, `lym board upsert` setting the entry's `recipe` to `raddle` or dropping the label; a note keeps it as today.
A plain promote of a Sketch still carrying its label meets this refusal first, inside the write's mutation, before `checkDiscussedPromotion` runs.
Such a task is no Sketch: `IsSketch` already reads kind note only, and the README renders it among tasks, not under Sketches.
`boardcli`'s `promote` verb gains `--recipe <name>`, passed to `PromoteTo`; its help names the raddle rule.
`TestDiscussedPromotion` (tier0) has rows: promote with recipe `raddle` of a Sketch (one write: task, recipe raddle, empty status, record kept, flag and label kept); the same of a discussed note a note write stripped of the `sketch` label (label restored, empty status); plain promote of a Sketch carrying the label (refused by the label rule, naming its way forward); plain promote of a discussed note stripped of the label (refused by `checkDiscussedPromotion` with the `--recipe raddle` way forward); promote with recipe `darn` of that stripped note (refused the same way); upsert to kind task with recipe `darn` of a Sketch (refused); a discussed note whose record file is missing (not judged); promote with recipe `darn` of an undiscussed note (task, recipe darn, draft as today); `IsSketch` false on the promoted entry.
`TestValidateLabels` gains the rows of `sketch` on a raddle task (admitted), on a darn task and on a project (refused, naming the upsert).
`TestDraftStatus` covers the draft rule for every other write as it stands.
`TestCLI` gains the `--recipe` row.
`boardengine/doc.go`'s `# Sketch` section and `boardcli/doc.go` describe the rule and the flag.

**ImpactSummary:** One new board-wide write rule, narrowed to discussed notes with a stored record becoming tasks, one label rule and one released-list parameter on the draft rule whose one caller passes it; existing promote callers keep their signature, and the run lock is untouched.
