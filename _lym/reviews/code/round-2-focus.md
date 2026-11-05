---
round: 2
focus:
  - "Check `Store.promoteTo` in internal/boardengine/store.go: for a plain promote (empty recipe) of a discussed note that carries the `SketchType` label but whose record file is missing, does the recipe come out raddle, so the refusal in `checkSketchLabel` (internal/boardengine/labels.go) no longer sends the operator back to the command just run?"
  - "Check that the doc comments on `promoteTo` and `PromoteTo` in internal/boardengine/store.go, the `checkSketchLabel` comment in internal/boardengine/labels.go and the promote sentence in internal/boardengine/doc.go describe the same rule the code applies for a labelled note with a missing record."
  - "Check internal/boardengine/sketch_test.go: is the labelled-note-with-missing-record plain promote covered by a row whose setup keeps the label, since the existing missing-record rows start from a stripped note without the label?"
---
