```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the promote-raddle diff. I ran each changed branch on one concrete input and made no tool calls.

**List: function or branch, case run, result**

1. `Store.promoteTo`, Sketch with a stored record and the sketch label, recipe `""`: it takes recipe raddle, `validateWrite` passes, and `promoted` releases it. Correct.
2. `Store.promoteTo`, stripped discussed note (no label) with a stored record, recipe `""`: it takes raddle and the label is restored. Correct.
3. `Store.promoteTo`, Sketch, recipe `darn`: `checkSketchLabel` refuses via the discussed-prior branch, naming the plain promote, which does work for a Sketch with a stored record. Correct.
4. `Store.promoteTo`, stripped discussed note with a missing record, recipe `""`: no recipe, no label, so it promotes as any note. Correct.
5. `Store.promoteTo`, discussed note carrying the sketch label with a missing record, recipe `""`: **wrong**, see finding 1.
6. `Store.promote` / `promoteTo` with a nil `recordStored`: `recordHeld` is false and behavior is unchanged. Correct.
7. `Store.promoteTo`, entry already a task or a project: unchanged return and the existing project refusal. Correct.
8. `boardCriticalSection` release: a plain Sketch promote now carries recipe raddle into `promoted`, so `released` is set. Correct.
9. `checkDiscussedPromotion`, reached from an upsert of a stripped note with a stored record to a `darn` task: the refusal names the plain promote, and that promote succeeds because the record is stored. Correct.
10. `checkSketchLabel` from an upsert with a discussed prior, record stored: the message is true. Correct.
11. `SketchPromoteCommand`, normal slug: it yields `lym board promote '{"slug":"x"}'`, which `resolveLookup` parses as JSON. Correct.
12. `S3` handle through `resolveLookup`: `ParseHandle` matches, so the selector is a handle and `GetTask` resolves it. Correct.
13. `raddleStartRefusal`, no stored record: the text ends with the quoted promote command. Correct.

**Finding 1**
- **Severity and class:** MEDIUM, `design`.
- **Summary:** For a discussed note that still carries the sketch label but whose record file is missing, the refusals name a plain promote that cannot succeed. The refusal says the plain promote "gives it recipe raddle", and that is false here.
- **Trace:** `recordHeld` is false, so `promoteTo` leaves recipe `""` and the label stays. `checkSketchLabel` finds `t.Recipe != raddle`, a prior of kind note rather than task, and a discussed prior, so it refuses at `labels.go:66-67`, naming `lym board promote '{"slug":…}'`. That is the same command the operator just ran, so it is a dead-end loop.
- **Regression:** Before the diff, `promote --recipe raddle` succeeded here. The label rule accepts recipe raddle, and the missing record skips the promotion rule. The refusal's old way forward was therefore reachable, and the new one is not.
- **Docs also wrong:** `boardengine/doc.go:328` ("a discussed note whose record file is missing is led to the plain promote as well") and the `checkSketchLabel` doc at `labels.go:52` now claim the plain promote helps.
- **Evidence:** `store.go:783-792`, `labels.go:53-70`, `sketch.go:108`, `doc.go:327-328`.
- **Test gap:** `PlainPromoteOfANoteWithAMissingRecordFileTakesNoRecipe` uses `recordMissing`, which starts from `stripped`, so the label is absent. No row covers the labelled case (`sketch_test.go:215-218,293`).

Tool calls: none; the manifest already held every definition this lens needed.
