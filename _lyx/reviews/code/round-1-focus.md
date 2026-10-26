---
round: 1
focus:
  - "internal/planparser/fabricref.go: does `matcher.MatchPath(id)` on a bare card id match exactly what the audit's barePattern matches in `--card <id>`, so every id the audit would flag is refused and every id it would not flag passes? Check against `internal/fabricengine/refscanner.go` and the `FabricReferenceMatcher` contract."
  - "The new finding in `CheckCardFabricReference`: does it keep the naming-free seam (no weft naming imported into planparser), name the card and matched text, and carry a way forward a test reaches, per the decision record's Scope and Constraints?"
  - "Existing `card-fabric-reference` outcomes: does adding the per-card id finding change the finding order, count or Card field of any existing row, or of a caller that consumes these findings (plan index format and rework validation, `lyx loom validate-plan`)?"
  - "Test rows in `fabricref_test.go`: do the two new rows cover the acceptance criteria (suffix-ending slug fails with the rename way forward, mid-name `weft` passes) without duplicating an existing row, and is the fabric suffix derived rather than hard-coded?"
  - "Docs in the same diff: do the `card-fabric-reference` entry in `contracts/specs/loom-plan-spec.md`, the `internal/planparser/doc.go` text and the function's doc comment match what the code does, and does the diff leave `internal/fabricengine/refscanner.go` untouched as the decision record's Scope requires?"
---

Seed focus from the board entry, the decision record and the single-commit diff over the review range.
