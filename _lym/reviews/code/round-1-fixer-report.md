# Fixer-report, code round 1

## Changes

- **F1, fixed as proposed.**
  - `internal/boardengine/store.go`, `promoteTo`: a plain promote gives recipe raddle to a discussed note when its record is stored or it carries the `sketch` label (`recipe == "" && current.Discussed && (recordHeld || slices.Contains(current.Labels, SketchType))`).
    The label restore stays keyed on `recordHeld`, since a labelled note needs none.
  - `internal/boardengine/sketch_test.go`: new `TestDiscussedPromotion` row `PlainPromoteOfASketchWithAMissingRecordFileTakesRaddle`, with a setup that runs `sketch` and then removes the record file; it wants kind task, recipe raddle, the label kept, the flag kept and no record.
    The row fails with the old condition, because `checkSketchLabel` refuses a labelled note with an empty recipe.
  - Doc comments of `promoteTo` (`store.go`), `Board.PromoteTo` (`board.go`), the `TestDiscussedPromotion` comment and the sentence at `doc.go:327` state that a discussed note carrying the label takes raddle too.
  - Commits: `boardengine: fix F1 — plain promote of a labelled Sketch with a missing record takes raddle`, and `boardengine: fix F1 — widen the PromoteTo, promoteTo and test doc comments to the labelled-note case`.
  - Gate: `lym gate test ./internal/boardengine` passed after each commit.

## Self-check

- fix-code-correctness, LOW, `board.go` `PromoteTo` doc comment stale: fixed as proposed.
- fix-consistency: no findings; its note on the same `PromoteTo` comment is covered by the line above.
- fix-proposal, LOW, a labelled note with no record is now released silently: disputed, every fix worse.
- fix-affected-set, LOW, `TestDiscussedPromotion` comment says a missing-record note promotes as any note: fixed as proposed.
- fix-affected-set, LOW, `promoteTo` comment on a nil `recordStored`: fixed as proposed, it now says only the label then gives raddle.

## Disputed

- **Fork finding of lens fix-proposal, ground: every fix worse.**
  The premise holds: a Sketch with a lost record file is released without the operator being told.
  The one other change, naming `--recipe raddle` in the refusal, leaves `lym board promote` a dead end for a labelled note, the defect F1 names.
  The silent release is not new: `promote --recipe raddle` admitted this note before the diff.
  The lens itself rates it an open question and not a defect.

## Deferred

Nothing deferred.
