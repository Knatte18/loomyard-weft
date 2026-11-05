```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 change set (`store.go` `promoteTo`, `sketch_test.go` new row, `doc.go:327`) for correctness.

**Findings**

1. LOW — `internal/boardengine/board.go:415`, the `PromoteTo` doc comment: it was not updated with the behaviour change. It says "A discussed note whose decision record is stored takes the raddle recipe when recipe is empty". A discussed note that carries the sketch label also takes raddle now, even with its record file missing (`store.go:784`). The reader of `PromoteTo` gets the old rule. `store.go:764-767` and `doc.go:327` were updated, so this is the one doc left behind.

**Traces that held**

- **Input:** a discussed note labelled `sketch` with its record file removed, plain `PromoteTo("idea", "")`.
  1. `recordHeld` is false, because `recordStored` reports no file.
  2. The new condition holds: `recipe == "" && Discussed && (false || label)`, so recipe becomes raddle.
  3. The restore branch at `store.go:790` is skipped, and the label is already there.
  4. `validateWrite` accepts it: `checkSketchLabel` returns nil for a raddle task, and `checkDiscussedPromotion` skips a note whose record is missing (`sketch.go:108`).
  5. The result is a task with recipe raddle and the label kept.
- **The new row fails with the defect in place.** With the old condition, recipe stays empty and `checkSketchLabel` refuses at `labels.go:66-67`. The row wants a nil error, so it fails. `wantDraft` is false and `wantRecord` is false, which fits a record-less Sketch.
- **Callers:**
  - `Board.PromoteTo` and `Board.Promote` go through `b.recordStored`, so they are unaffected beyond the intended change.
  - `Store.Promote` passes a nil `recordStored`. It now gives raddle to a discussed note that carries the label, but only the Store-level tests call it directly. None of the callers I saw sets both the discussed flag and the label.
- **Existing rows:** the undiscussed note and the stripped missing-record row (no label) keep the old behaviour, because the new clause needs both `Discussed` and the label.

**Tool calls**
1. grep for `promoteTo(`, `recordStored` and `checkDiscussedPromotion` across non-test Go files: my context lacked the callers and the promotion rule.
2. grep for `.Promote(` callers: to see whether a nil `recordStored` reaches production.
3. Read `board.go:405-455`: to see `PromoteTo`'s wiring and its doc comment.
