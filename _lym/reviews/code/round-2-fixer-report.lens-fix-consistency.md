```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for places that disagree with the changed text, or twins left unchanged.

No findings.

- The only other statement that the entry's dates stay was the pair of sentences the fixer changed (`internal/boardengine/doc.go` "# Sketch" and `_lym/landing/summary.md`). Grep over `.go` and `.md` found no third wording.
- The `sketchOf` comment in `internal/boardengine/sketch.go` says "Every other field ... stays". That is true of `sketchOf`, which returns the task with `Revised` unchanged. The `stampDates` restamp happens in the write path, so the comment does not contradict the new doc text.
- No other text claims the motif-finish fallback is "covered only by reading the code" or that "no test drives a motif finish". The F3 deletion left no stale twin.
- The new `SketchThatKeepsItsRunStatusIsRefused` row sits among the existing rows of `TestRunLock_SketchExemptionIsTheWholeNetChange`, and the test's doc comment ("only when it becomes a status-less Sketch with no other scope change") still holds. The summary's "Run lock" bullet ("a Sketch with no status") also agrees.
- The Created and Revised assertions in `HeldTaskBecomesASketch` read the `Created` value before `MakeSketch`. That matches the corrected doc: Created kept, Revised not before it.

Further calls:
1. `grep` for "dates stay" wordings across `.go` and `.md` files: to find twins of the changed sentences outside the files in my context.
2. `grep` for "covered only by reading" and "no test drives a motif": to find a twin of the deleted summary bullet outside my context.
