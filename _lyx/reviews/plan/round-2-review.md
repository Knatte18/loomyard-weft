---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/05-retire-cross-card-containment.md
    summary: Card 5 does not enumerate the checkGlyphMalformed doc comment in planparser validate.go that still names containment-unit-overlap
---

### [LOW] Card 5 leaves one retired-ID comment unenumerated

**Location:** `_lyx/plan/05-retire-cross-card-containment.md`, Intent bullet on `validate.go`.
**Issue:** D2 and the acceptance criteria require that no spec, doc or test still names `containment-unit-overlap` as a live check.
Card 5 lists `internal/planparser/validate.go` as a file target and names two edits in it: `validate` stops calling `syntacticContainment`, and the header list drops the ID.
`internal/planparser/validate.go` also carries a third mention, in the doc comment of `checkGlyphMalformed` ("containment-unit-overlap skips it the same way"), which the Intent does not list.
Every other doc comment naming the retired tiers is enumerated (`resolvePass`, `ValidateDispatch`, `resolveTargets`, `cardOwnHandles`, `BeginBatch`, the `shape.go` ledger), so an implementer working from the Intent's list leaves this one behind.
**Fix:** Add a bullet to card 5's Intent: `checkGlyphMalformed`'s doc comment stops listing `containment-unit-overlap` among the checks that skip a malformed glyph.

## Focus verdicts

1. **`shapes#Generic` rows: consistent, not a defect.**
   Card 1 states that `GenericMethod`'s receiver names `Generic` outside the type's span, and lists `shapes#Generic.GenericMethod` beside `shapes#Generic` on the Delete leg.
   Card 2's removal code deletes the type and its method together.
   Card 10 puts `shapes#Generic.GenericMethod` beside the arrow leg on the same card.
   Card 11 states the outcome for both legs and ties it to the glyph that cards 1 and 10 list.
   The other fixture rows reference no other member (the fixture description says `GenericMethod` is the only one), so caller coverage changes no other row's outcome.
2. **Plan-gate-only checks: each states its dispatch bound, not a defect.**
   Card 8 (`redundant-file-target`) names the pairing that appears after binding and bounds it as plan noise the plan gate already refused.
   Card 10 (`resign-interface-method`) names the interface-method arrow that first appears after the plan gate and bounds it by the arrow's single mechanical effect and webster's per-card build and test gate.
   Card 11 (`caller-uncovered`) names the uncovered caller that first appears after the plan gate and bounds it by the same gate.
3. **Comments naming the retired tiers in `internal/planglyph/handle_test.go`: assigned once, not a defect.**
   The file holds two, both inside `TestBindHandles_RenameNewSideHandleBinds` (its doc comment and an inline comment).
   Card 6 assigns both through that member and says so; card 5 does not touch the file.
   The tests' `handle-unreferenced` code mentions are card 6's as well.

## Other checks that found nothing

- **Target shape against the deployed gate:** no Go file is named as a path on one card while a member of it is a target on another; the one cross-form pair (`answerSymbols` in `deleteorder.go`, a file target on card 11) appears only under `Uses` on cards 8 and 10, which the cross-card containment tier does not index.
- **Premises of the fix cards and the deletions:** `renameDeclSource` derives from `Symbol.Signature` as cards 3 and 4 describe, `declaredHandles` has no reader besides the `handle-unreferenced` half, and the two deleted `containment.go` files declare no helper used elsewhere.
- **External test unit spelling:** quarry names the external test unit `<dir>_test`, so the `shapes_test#TestExternal` row is correctly spelled.
- **`## verify:`:** matches the order the plan stencil states, and `./...` covers every targeted package.
