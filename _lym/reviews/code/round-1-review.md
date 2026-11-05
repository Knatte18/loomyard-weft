---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/store.go:783
    summary: A plain promote of a note that carries the sketch label with its record file missing is refused with a way forward that names the same plain promote, and the doc comments that claim it helps are now wrong.
    fix: In `Store.promoteTo` (store.go:783-786), also give recipe raddle on an empty recipe when the discussed note carries the `SketchType` label, so the condition becomes `recipe == "" && current.Discussed && (recordHeld || slices.Contains(current.Labels, SketchType))`; a labelled note is a Sketch, the label rule cannot see the record file, and `--recipe raddle` already promotes such a note, so the plain promote then matches the refusal's claim. Add a `TestDiscussedPromotion` row in sketch_test.go with a Sketch whose record file is removed (a new setup that runs `sketch` then removes the record file, since `recordMissing` starts from `stripped` and drops the label) and a plain promote, wanting kind task, recipe raddle and the label kept. Update the `promoteTo` doc comment (store.go:764-767) and the sentence at boardengine/doc.go:327 so they state that a discussed note carrying the label takes raddle too. If the fixer prefers to keep the promote's recipe rule as is, it must instead make the label rule's discussed-prior refusal (labels.go:66-67) name `--recipe raddle` for a note whose record is missing, correct the `checkSketchLabel` comment (labels.go:52), and pin that refusal with a row.
---

### [MEDIUM] Plain promote of a labelled note with a missing record dead-ends on its own way forward

**Location:** `internal/boardengine/store.go:783`, `internal/boardengine/labels.go:66`, `internal/boardengine/doc.go:327`

**Issue:** `promoteTo` gives recipe raddle to a plain promote only when `recordHeld` is true, meaning the record file exists.
A discussed note that still carries the `sketch` label but whose record file is missing, such as one left by a rename that moved the entry but not its record, keeps recipe empty and the label.
`checkSketchLabel` then refuses it (`labels.go:57-67`), and its discussed-prior branch names `lym board promote '{"slug":"<slug>"}'` "which gives it recipe raddle", the command the operator just ran.
Before the diff, `promote --recipe raddle` succeeded for this note, because the label rule accepts raddle and the missing record skips the promotion rule, so the old way forward was reachable and the new one is not.
`doc.go:327` still says a note whose record file is missing is led to the plain promote, and the `checkSketchLabel` comment says the plain promote gives raddle; both are false for this case.
No test row covers it: both missing-record rows start from `stripped`, which has no label (`sketch_test.go:215-218`, `293`).
The lens reports of code-correctness, code-affected-set and code-tests each raised this same defect, and the test gap is folded into this finding.

## Rejected

- Plain promote through the CLI (`lym board promote S3`) lacks a CLI-level test: rejected, the handle lookup in `resolveLookup` is unchanged and the engine row `PlainPromoteOfASketchByHandleTakesRaddle` pins the behavior.
- The refusals name the JSON-slug form where the board entry wrote `lym board promote <slug>`: rejected, the entry's `<slug>` is a placeholder, the verb rejects a bare slug as invalid JSON, and the Wanted list asks only that the plain promote be named.
- No `Pattern-Departure:` trailer and no PATTERN change: rejected as a finding, the directive declares no PATTERN change and the diff touches no PATTERN rule.
