# Round 1 fixer-report

## Changes

All four findings fixed as proposed.

- F1: `TestDoneChecks_Verdicts` gained the row "Rename new side landed in the external test package" (`internal/planglyph/donecheck_resolve_test.go`).
- F2: `TestBindHandles_HandleBindsToTheOtherPackageOfItsDirectory` gained a delta field and the row "exact match wins over the other package".
  The new `TestBindHandles_OtherPackageMatchStillCountsTowardMismatch` table pins a counterpart match counting toward `bind-count-mismatch` beside an unmatched handle.
  Committed together with F3, whose double-bind row lives in the same table.
- F3: `BindHandles` tracks the glyphs claimed within a card (`internal/planglyph/handle.go`); a handle reaching a claimed glyph counts as unmatched.
  It refuses any claimed glyph, not only counterpart binds, so the outcome does not depend on handle order.
  The doc comment states the rule.
- F4: `internal/planparser/doc.go` and `contracts/specs/loom-plan-spec.md` now name `goPackageDir` and `GoPackageCounterpart` as the sanctioned adjustments.

- Gate repair: the F4 comment in `internal/planparser/doc.go` read as one sentence broken at a comma; it is now two sentences, one per line.

`lyx gate test ./internal/planglyph ./internal/planparser` passed before the final comment-only edit.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, LOW: the `BindHandles` doc comment omitted the claim rule and the `BindNoDeltaMatch` wording. Fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.
