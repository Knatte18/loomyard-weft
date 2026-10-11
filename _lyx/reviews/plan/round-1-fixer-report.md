# Plan-Review round 1 — fixer report

Every finding's premise was checked against the code and the decision record, and all seven hold.
The split for F5 renumbered the cards: the old cards 3–7 are now 3 and 5–8, and the new card 4 is `owned-handles`.
Card numbers below are the new ones.
`lyx loom validate-plan` passes after the fixes, with only `create-new-unit` informational findings left.

## Changes

- **F1, fixed differently** (`08-begin-batch-builds-pack.md`).
  The premise holds: `BeginBatch` would refuse on a failed fresh `ParsePlan`, which D5 does not license ("the batch itself is still valid").
  The parse error now degrades and never refuses.
  `buildKickstartPack` logs at Warn and builds the pack from `deps.Plan` and `batch.Cards`, it returns no error (its declaration head drops the `error` result), and `BeginBatch` gains no refusal, so the ImpactSummary and the D9 line's "in every case" hold.
  Not done as proposed: listing every member glyph of `deps.Plan`'s batch cards as unresolved would make `internal/websterengine` select member glyphs itself.
  The overview's Shared Decisions place member-glyph selection in `internal/planglyph` behind `planindex.Index`, and the record's `PATTERN-glyph-conversion-chokepoint` constraint sends it through `glyph.Parse`.
  The fallback passes the in-memory cards to `index.KickstartPack` instead, so member glyphs still resolve and only a canonicalized handle can keep its draft spelling.
  For reachability, `fakeIndex` gains a `ValidateDispatch` override (a new `Create:` declaration) behind a `breakPlanAfterValidate` field, which forwards and then corrupts `00-overview.md`.
  A new `TestBeginBatch_KickstartPack` row reaches the branch through that field.
- **F2, fixed as proposed** (`08-begin-batch-builds-pack.md`).
  Each row now writes its own plan with `plankit.Write`, parses it into `fx.Deps.Plan`, points `fx.Deps.Geom.PlanDir` at it and sets the fingerprint with `mustFingerprint`.
  The switch-on row's `Edit` member glyph has its declaration written with `writeWorktreeFile`.
  The `language: none` row writes the same plan under `language: none`.
  The canonical-spelling row declares the draft handle without `seedRewritingPlanDir`'s blocking card 2.
  `seedRewritingPlanDir` leaves `Uses:`, and `mustFingerprint` and `writeWorktreeFile` join it.
- **F3, fixed as proposed** (`05-bind-trace.md`): the `bindErr` variant.
  `fakeIndex.bindErr` is copied into the `fakeDelta` that `fakeIndex.Delta` returns, and `fakeDelta.BindHandles` returns it.
  The read-only-directory sentence is gone.
  `fakeIndex.Delta`, `fakeDelta` and `fakeDelta.BindHandles` join `Edit:`.
- **F4, fixed as proposed** (`07-pack-stencils-and-render.md`).
  `renderFork` carries the re-sign arrow with the trailing `pack *planindex.Pack`.
  `TestRenderPrompts_FrictionDirective` joins `Edit:`, with a test bullet saying it passes a nil pack.
- **F5, fixed with the proposed split, placed after the outcome card instead of ahead of it** (new `04-owned-handles.md`, `03-bind-outcomes.md`).
  Ahead of the outcome card, the split fails `delete-before-reference`: `validate-plan` reported that card 3 deletes `internal/planglyph#cardOwnHandles` while the outcome card's `Edit:` code still names it at `internal/planglyph/handle.go:390`, `:426`, `:440` and `internal/planglyph/handle_test.go:664`.
  Its way forward is to move the delete after the editing card.
  So card 3 now holds only the outcome types, the `Delta.BindHandles` result change and its implementers and tests, and lists outcomes per handle `cardOwnHandles` lists.
  Card 4 creates `planparser.OwnedHandles`, deletes `cardOwnHandles`, and edits `BindHandles` (no arrow), `planglyph/doc.go` and `TestBindHandles_RenameFileSidePairIsNotAHandle`.
  Card 5, the first `internal/websterengine` caller, still comes after it.
  Prose card-number mentions were updated by hand after the renumbering.
- **F6, fixed as proposed, first option** (`07-pack-stencils-and-render.md`).
  The `noMemberGlyphsLine` const is gone.
  A fourth marker-free stencil, `webster-fragment-pack-empty.md` with `WebsterFragmentPackEmpty`, holds the D8 line, and `renderPackSection` reads it through `stencilstore`.
  It gets its own `wordingClaims` and `TestStencils_TopLevelMarkers` rows.
  The ImpactSummary now says four stencils.
  The precedent-only `noPrecedingBatchDigest` leaves `Uses:`.
- **F7, fixed as proposed, first option** (`07-pack-stencils-and-render.md`).
  The pack fragment carries its own `{{.specs_dir}}/loom/loom-plan-spec.md` pointer, as the record's producer-pointer constraint words it.
  `RenderForkPrompt` fills it with `stencil.Fill` before filling `locate`.
  The `TestStencils_TopLevelMarkers` row reads `specs_dir` for that fragment, and its claims row pins the pointer.

## Disputed

None.

## Deferred

None.
