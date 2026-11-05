```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency (check whether the change set left a twin stale): no BLOCKING or MEDIUM findings, one LOW.

**LOW — the done-motif-run exception still says "recipe raddle" in two places that now mean the board-resolved recipe.**
- Changed text: `internal/battencli/dobbyreads.go:287` was updated to "whose recipe the board resolves to raddle". Its twins were not:
  - `internal/dobbyengine/doc.go:109` was edited in the same commit but still reads "a task whose recipe is raddle".
  - `internal/battencli/cli.go:221-222` (the `run` Long text) reads "a task with recipe raddle" and was not touched.
- Evidence: F4 made `startClearsMotifRun` resolve the recipe through `boardRecipe(board, task)`, so an entry with an empty recipe field under a board whose default is raddle is cleared. The two phrases read as the entry's own field only.
- Fix: word both like the `DobbyTasks` godoc, "whose recipe the board resolves to raddle".

Checked and consistent:
- The `cleanup.go:2` header and the `cleanup.go:105` `ClearMotifRun` godoc still hold, since removal is unchanged and only the call site moved.
- The `beamcli` doc and `cleanup.go:524` describe `lym beam create`, which is unaffected.
- The F3 twins agree: the `finalizeEntered` godoc, `fabricengine/doc.go:1302` and the `loomcli/doc.go:290` sentence all say "origin shows done".
- Every caller of `finalizeEntered` and `finalizeEnteredRead` was updated, and `withoutFinalizeEntered` keeps its signature.
- `loomcli/doc.go` lines 25-28 (the finish paragraph) are unaffected by F5 and F8.

Tool calls:
1. `grep` for the prose twins of the clear: the sweep needed more than the files in context.
2. `Read` of `cli.go:219-226`: only part of that file was in context, and I needed the exact `run` Long text.
