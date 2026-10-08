# Card 6 — retire-handle-unreferenced

**Edit:**
- `internal/planparser/validate.go`
- `internal/planparser/handle.go`
- `internal/planparser/validate_test.go`
- `internal/planparser/handle_test.go`
- `internal/planglyph#TestBindHandles_MatchedHandleRewritesDeclaringAndReferencingCard`
- `internal/planglyph#TestBindHandles_RenameNewSideHandleBinds`
- `internal/planglyph/glyphchain_test.go`
- `internal/planglyph/glyphchain_integration_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Intent:**
Retire `handle-unreferenced` (D4), so a card declares every new member it creates as a `plan:` handle whether or not a later card references it.

- `checkHandleConsistency` in `validate.go` drops its third half and the doc sentence on why that half read Create declarations alone; the file's header list drops the ID.
- `handle.go` deletes `declaredHandles`, its only reader gone, and its header stops naming a "declared" predicate.
- `TestValidate_HandleConsistency` drops the `handle-unreferenced` subtest and the ID from its clean-case loop; `TestHandleIndexes` keeps only its `referencedHandles` assertions.
- The two `TestBindHandles_*` tests drop the ID from the findings they tolerate after the rewrite, and `TestBindHandles_RenameNewSideHandleBinds`' doc comment and inline comment stop naming "both containment tiers", which card 5 retired.
- The spec drops the `handle-unreferenced` row, renumbers the list and fixes every `row N` cross-reference, and rewrites the Plan: handles section's paragraphs on the handle checks so they name `handle-dangling`, `handle-collision` and `handle-malformed` with no count and no `handle-unreferenced`.
  The stencil sentence requiring a later reference is card 12's.

Flip the D1 unreferenced-handle row: a Create handle no other card references validates clean, and its integration leg binds it at record-batch and passes its done-check.

What this lets through: a declared handle no other card uses.
What bounds it: the handle is still bound at record-batch, where a card whose delta matches fewer handles than it owns is `bind-count-mismatch`, and done-checked, where a Create target that still does not resolve is `create-not-done`, so a handle the card fails to build is still caught.
`handle-dangling`, `handle-collision` and `handle-malformed` stay.

**ImpactSummary:** Removes one pure check ID and its helper; plans that were refused for a declared-but-unused handle now validate, and no caller outside planparser and planglyph's tests reads the ID.
