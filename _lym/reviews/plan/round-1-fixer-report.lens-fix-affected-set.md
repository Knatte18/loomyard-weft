```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I reran the set search behind each set-based finding and checked that the fix covers every member.

Unchecked findings: none of the set-based ones. F1, F6, F10, F11, F16 and F20 were each rerun. The other findings are not about a set, so this lens does not cover them.

**LOW** — card 13's `StatusDraft` sentence (F11)
- **Summary:** the fix updates only the `StatusDraft` const's comment. Three other places say the same thing ("every new task takes it") and are left stale: the file header in the same file, and two glossary rows.
- **Evidence:** `internal/boardengine/draft.go:1` reads "the draft status every new task takes"; `docs/research/weaving-glossary.md:98` (drafter row) and `:142` (draft row) both say "every new task takes it". Card 13 targets only `internal/boardengine#StatusDraft`. Card 27 edits the glossary, but only its raddle, shed and reserved-names rows.

Sets that the fix covers fully:
- **F1:** no `SketchRecipe` is left in any card. `RecipeRaddle` is declared once (card 13:4) and only used by cards 23 and 24.
- **F10:** `internal/loomcli/doc.go` is now named only by card 23 (Extends) and card 25 (Uses). Card 25 comes after card 23, so no `uses-later-target` applies.
- **F20:** `names.go` lists the recipe names in three comments: lines 63, 77 and 101 (`RowNames`, `Rows`, `AmendmentTarget`). Card 23 now names all three.
- **F16:** every `pawlengine.Deps` `Notify` that has a body is covered: `pawlwiring.go:46` and `:137` (`pawlDeps`), `world_test.go:180`, `fixture_test.go` (`validPawlDeps`), `envkit.go:374` (`FullEnv`), and `beamawait.go:372` (`awaitRun.notify`). The sites that only set it nil or check that it is present (`beamup_test.go:193`, `entries_simple_test.go:123`, `entries_pawl.go:34`, `beamawait.go:49`) do not depend on the signature.
- **F6:** both directive tests, `TestStencils_OpeningStencilsRenderParentDirective` and `TestStencils_OpeningStencilsRenderEditDirective`, iterate over `roleOpeningStencils`, so the fix's wording covers them.

Outside this lens: `contracts/stencils/parentdirective_test.go:159` `TestStencils_NoUnboundedAskOperator` scans registered stencils. Card 21's raddle stencil puts the proposal to the operator, and I did not check it against that test.

Tool calls:
1. Grepped the plan for `SketchRecipe`/`RecipeRaddle` and for cards naming `loomcli/doc.go`, to rerun F1 and F10 against the edited cards.
2. Grepped Go comments for `shedrun.RecipeMotif`, to rerun F20's set of recipe-name comment lists.
3. Grepped the pawl `Notify` and `pawlengine.Deps{` sites, to rerun F16's set of callers and fakes.
4. Grepped for "every new task" and the test functions in `parentdirective_test.go`, to rerun F11's and F6's sets.
