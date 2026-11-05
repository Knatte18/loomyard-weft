```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F7), through the fix-proposal lens.

Proposals carried out as proposed: F1, F2, F3, F4, F5, F6, F7.

1. **LOW** — `endsAsSketch` in `internal/boardengine/sketch.go`, `scopeChanged` in `internal/boardengine/runlock.go`.
   - **Summary:** The exemption is not "exactly" `sketchOf(before)`. A hand write can pass it while keeping fields `sketchOf` clears.
   - **Evidence:**
     - `scopeChanged` never compares `Profile`, `RunSettings`, `NotifyOnLand` or `Discussed`.
     - A write that makes a held task a Sketch and keeps `profile`, `run_settings` or `notify_on_land` therefore passes.
     - The result is a Sketch carrying fields the sketch conversion promises are cleared.
     - The new doc text in `endsAsSketch` and `boardengine/doc.go` says "exactly", which overstates this.
   - **Impact:** Limited to a hand write on a run-held entry. `MakeSketch` clears those fields, and the run lock's scope concern does not cover them.
   - **Fix, if wanted:** Soften the wording to "no scope change", or also require the cleared fields to be zero.

2. **LOW** — the `NoteWithoutTheSketchLabelIsRefused` row in `TestRunLock_SketchExemptionIsTheWholeNetChange` (`internal/boardengine/sketch_test.go`).
   - **Summary:** The row would have passed on the old `endsAsSketch` too, so it pins existing behaviour and does not guard F1.
   - **Evidence:** The old `endsAsSketch` already required `IsSketch(now.Kind, now.Labels)`. A note with labels `[board]` fails that check, so it was refused before the fix.
   - **Impact:** Only the title and body rows fail with the defect in place. The reviewer asked for this row, so this is minor.

I built cases against the other proposals and found no defect:
- **F1 caller wiring:** `board.go` passes `b.vocab` (the same vocabulary `MakeSketch` uses) and `replay.go` passes `*r.store.vocab` (always set before `Apply`).
- **F4:** A post-save `DecisionRecordError` is not `ErrSketchRefused`, so `sketchFinish` returns it. The next finished transition then finds the entry already a Sketch, and `MakeSketch` writes nothing and returns nil.
- **F2:** The legacy guard refuses a discussed task until the next write migrates it. A depended-on discussed task stays refused. A grep for other uses of the old status found none.
- **F3, F5, F7:** The fallback row, the replay row and the depended-on discussed-task assertion each follow the review's proposal. The tier 0 and tier 1 gate runs passed.

Tool calls made: none. My context already held every changed file and the gate results.
