# Fixer report — plan, round 1

## Fixed

- **F1 (BLOCKING):** premise confirmed: `func (g *Generic[E]) GenericMethod() E` names `Generic` outside the type's resolved span, which D5's own-package token rule counts as a reference.
  Fixed at the source so every card stays green, following D5's cover rule (the member glyph whose span holds the occurrence):
  - Card 1 (`01-glyphchain-plan-gate.md`): the false "No fixture member references another" sentence now names `GenericMethod`'s receiver as the one reference, and states that every leg deleting or re-signing `shapes#Generic` lists `shapes#Generic.GenericMethod` on the same card; the Delete leg says the `shapes#Generic` row deletes the type and its method together.
  - Card 2 (`02-glyphchain-delta-legs.md`): the Delete leg's code change for that row removes the type and its method together, so the integration leg compiles and done-checks clean.
  - Card 10 (`10-resign-head-match.md`): the arrow leg for `shapes#Generic` lists `shapes#Generic.GenericMethod` beside it on the same card.
  - Card 11 (`11-caller-coverage.md`): a row-list entry states that the `shapes#Generic` Delete and arrow legs stay clean unchanged, covered by that method glyph.
- **F2 (LOW):** card 10 now states what dispatch lets through for `resign-interface-method` (an arrow on an interface method first appearing after the plan gate, whose head can clear `resign-head-mismatch`) and the bound (the arrow only marks caller coverage, which dispatch does not run, and the per-card build and test gate).
  Card 11's "What this lets through, bounded" list gains the dispatch skip of `caller-uncovered`, why it stays out of `ValidateDispatch`, and the per-card build and test gate as its bound.
- **F3 (NIT):** premise confirmed at `internal/planglyph/handle_test.go:573` and `:603`.
  Card 6's `TestBindHandles_*` bullet now owns rewording both comments in `TestBindHandles_RenameNewSideHandleBinds` so they stop naming "both containment tiers".

`lyx loom validate-plan` over the edited plan returns `ok: true` with informational `create-new-unit` findings only.

## Disputed

None.

## Deferred

None.
