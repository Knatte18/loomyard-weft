---
round: 1
focus:
  - "PATTERN.md, merged result: do the task's edits to hub-containment/hub-suffix entries and the parent's edit to `PATTERN-shed-recipe-registry` sit side by side without dropping or duplicating a line, and do the PATTERN.md and `pattern/` entry texts still agree with each other?"
  - "Parent's `PATTERN-shed-recipe-registry` change (`Implementer` exempt from the `Lookup` map, new `internal/shedrecipe/entries_implementer.go`): does any task code in `internal/fabricengine`, `internal/ideengine`, `internal/hubforge` or `internal/loomshed` register or look up producers in a way the new wording or the `Names` removal breaks?"
  - "Parent's removal of the `Batchifier` row and its `webster.go` / `loompreflight.go` changes in `internal/loomshed`: does any task code, doc or test (README.md, docs/overview.md, `contracts/recipes/loom-recipe.yaml` consumers) still assume the old row order or the removed symbols?"
  - "Task's portal and containment rework in `internal/fabricengine` (portals.go, reconcile.go, remove.go, prune.go) against the parent's new code: does the parent add any caller of the changed fabric/fslink/hubforge contracts that relies on the old behavior?"
  - "Test economy across the merge: does a parent-added test (`entries_implementer_test.go`, `webster_test.go`, `tools/plandag/main_test.go`) duplicate a task-added test, and is any behavior where the two sides meet left without coverage?"
---

Overlap is limited to `PATTERN.md`, `README.md` and `docs/overview.md`, with no package overlap.
The first entry covers the textual merge of the shared files; the others test each side's changed contracts against the other side's new callers.
