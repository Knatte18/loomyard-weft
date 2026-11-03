# Lens review

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged onto the first line naming them.

- coverage 1 + guard-reach 1, MEDIUM — fixed differently: the proposal offered `composeActivity` or `RenderStatusHuman`; `composeActivity` was not changed, since the engine holds no routing to tell a block row from another row; card 9 now changes `RenderStatusHuman` (the `lym loom status` terminal view) through a new `blockAwareLast`, printing `→ entered` for a block row's Done and `in <block>` on the `now` and `last` lines, with `TestRenderStatusHuman` rows.
- coverage 2, MEDIUM — fixed as proposed: card 6's `TestBlockRecipes` compares `New(arras)` with `Routing(arras)` row for row on `SkipTo` and `Block`, and `TestRecipeKeys` gains `HasEngine` and `EngineRow` rows on arras.
- coverage 3, MEDIUM — fixed differently: the proposal named `TestRecipeKeyedWiring`, which builds no segment settings; card 7 adds an arras pass to `TestWire_DefaultConfig`'s budget block, which reaches `loomSegmentPerches`' discussion perch mapping through `loomSegmentBudgets` and the wired env (`internal/loomcli/wiring_test.go:448-480`).
- coverage 4 + row-validity 7, LOW — fixed as proposed: card 4's `TestExpand` has one refusal row per case, each run-only key and each edge key its own row.
- coverage 5 + guard-reach 3, LOW — fixed differently: no row for an empty name was added; card 2's `LookupBlockGate` now refuses any name the table lacks, with no separate empty-name branch, since Parse already refuses a block row with no `block_gate`.
- coverage 6, LOW — fixed as proposed: card 1's `TestGoto_BlockedRunAdmitsOnlyEarlierRows` gains a row admitting a goto onto an inlined row of a passed-over block.
- coverage 7, LOW — fixed as proposed: the overview's Shared Decisions record why loomrecipe tells its one resolver to every embedded recipe, pawl included.
- coverage 8, LOW — fixed as proposed: `TestExpand` gains a row whose sub-recipe's `terminals` name a foreign row, keeping the parent's.
- coverage 9, LOW — fixed as proposed: `TestBlockRecipes` asserts the built arras Discussion ring's `OnStuck` chain, equal `Segment` and `MaxBounces`, and `Block`.
- way-forward 1 + guard-reach 2, MEDIUM — fixed as proposed: card 8's `prepareRunSettings` keeps the task's recipe when `ProfileRecipe` refuses, so an unknown recipe with no profile named still meets the seed write's `shedrun.ValidateRecipe` refusal, wrapped in `battenshed.ErrUnknownRecipe`; the planned refusal row became a no-refusal row.
- way-forward 2, LOW — fixed as proposed: card 5's not-expanded check runs ahead of the loom contract check, and its `TestBuild_Errors` row builds with `Contract: "loom"`.
- way-forward 3, LOW — fixed as proposed: card 4 states each `Expand` refusal's text and way forward, and its rows assert them.
- way-forward 4, LOW — fixed as proposed: card 1 states the `SkipTo` validate rule is a fail-loud wiring guard like validate's other rules.
- guard-reach 4, LOW — fixed as proposed: card 5's `RoutingOf` refuses a parsed recipe holding a block row with the not-expanded refusal, with a `TestRoutingOf_ParseErrorSurfaces` row.
- row-validity 1, MEDIUM — fixed as proposed: `TestGoto_BlockedRunAdmitsOnlyEarlierRows` gains a `producers` field defaulting to `tailProducers`, now under card 1's `Uses`.
- row-validity 2, MEDIUM — fixed as proposed: `TestStep_Block` is dropped; card 1 keeps only the skip-target and block-path rows, in `TestStep_EntryGate` and `TestStep_RerouteMovesToTargetAndEndsTargetSegmentEpisode`, and names the existing tests that carry the block's goto, re-launch and Stuck behaviour.
- row-validity 3, MEDIUM — fixed as proposed: card 8's default-profile row runs with no profiles file, over the template's `tapestry: solid` default, so it fails without the resolve.
- row-validity 4, LOW — fixed as proposed: the Block producer is a row of `TestSimpleEntries_HappyPath`; `TestBlockEntry` became `TestBlockGates`, covering only the gate table, and the unknown-gate refusal is asserted once, through Build.
- row-validity 5, LOW — fixed as proposed: card 9 drops the passed-over `Motif` row.
- row-validity 6, LOW — fixed as proposed: `fakeGate` is under card 1's `Uses`.
- forced-edits 1, MEDIUM — fixed as proposed: `internal/loomcli/doc.go` moved to card 9's `Edit`, with the `entered` word and the `in <block path>` form.
- forced-edits 2, MEDIUM — fixed as proposed: card 1 extends `contracts/specs/loom-status-spec.md` with the `block` key.
- forced-edits 3, MEDIUM — fixed as proposed: card 7 edits `internal/reportengine/doc.go` and names `runs.go`'s file comment in its Intent.
- forced-edits 4, MEDIUM — fixed as proposed: card 1 edits `GotoRequest`, `RelaunchRequest` and `Routing` docs.
- forced-edits 5, LOW — fixed as proposed: card 9 declares `checklistEnteredWord` and `checklistBlockName` handles.
- forced-edits 6, LOW — fixed as proposed: card 7 edits `recipeList` and `tools/tokencount/main.go`, whose package comment holds the stale sentence (the package has no doc.go).
- forced-edits 7, LOW — fixed as proposed: card 7 edits `internal/battenshed#ErrUnsupportedChildRecipe`.
- forced-edits 8, LOW — fixed as proposed: card 7 edits `internal/battencli#childSeedParams`.
- forced-edits 9, LOW — fixed as proposed: card 7 edits the board recipe help text in `internal/boardcli#storeVerbs`.
- forced-edits 10, LOW — fixed as proposed: card 6 edits `RowNames` and `AmendmentTarget` and names the two file comments in its Intent.
- forced-edits 11, LOW — fixed as proposed: card 2's Intent names `registry.go`'s file header.
- forced-edits 12, LOW — fixed as proposed: card 5 edits `NewShed` and names `routing.go`'s file comment.
- forced-edits 13, LOW — fixed as proposed: card 7 edits `internal/gateslot/doc.go` and the PR-Review stencil's comment.

Beyond the findings, card 7 now also names arras in `arbiterRecipeDescription` (D10, a by-name tapestry label), and cards 6 and 7 write the parent's later decision that motif makes a sketch, the decision record.
