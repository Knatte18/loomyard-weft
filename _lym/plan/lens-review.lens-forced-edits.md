```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens on the plan in `_lym/plan/`: what each card's change forces outside the members it names. I found 13 defects: no BLOCKING, 4 MEDIUM and 9 LOW. Every member glyph the cards list resolves through `lym quarry`. No re-sign arrow is missing, and every `Create` handle's unit matches the package it is placed in.

1. **MEDIUM** — Card 9, checklist block marks: the change makes `internal/loomcli/doc.go` wrong, but the card lists that file only under `Uses`.
   - `internal/loomcli/doc.go:58-59` describes the checklist frame and its outcome words: "reads `bounced` and an awaiting one reads `escalated`, in this window only".
   - Card 9 adds the `entered` word for a block row's Done and the "`<name> in <block path>`" form on row and visit lines, so that paragraph no longer describes the frame.
   - Fix: move `internal/loomcli/doc.go` to `Edit` on card 9 and add both facts beside the `bounced`/`escalated` sentence.

2. **MEDIUM** — Card 1, history entry `block` field (D6): the status-file contract spec is not updated.
   - `contracts/specs/loom-status-spec.md:54` documents the optional keys of a history entry (`routed_to`, `reroute_capped`, …).
   - Card 1 adds `block` (`block,omitempty`) to `HistoryEntry`, and no card lists the spec.
   - Fix: list `//contracts/specs/loom-status-spec.md` under `Extends` on card 1 and add the `block` key with its meaning: omitted outside a block, a `/`-joined path otherwise.

3. **MEDIUM** — Card 7, arras in reports: `reportengine`'s package doc becomes false, and the card lists it only under `Uses`.
   - `internal/reportengine/doc.go:115` says "seeded to a recipe other than tapestry or darn, is left out silently".
   - The file comment at `internal/reportengine/runs.go:1` says "a selected recipe, tapestry or darn".
   - Card 7 adds arras to `runRecipes` and `recipePhaseTables`.
   - Fix: move `internal/reportengine/doc.go` to `Edit` on card 7, and add the `runs.go` file comment (its file self glyph or the `runRecipes` member) to the card.

4. **MEDIUM** — Card 1, `Goto`/`Relaunch` read `Block` and the skip target: the request types' field docs are now wrong.
   - `GotoRequest`'s and `RelaunchRequest`'s docs (`internal/shedengine/goto.go:42` and `:170`) both say "Producers needs only Name, Segment, OnDone and OnStuck, so a Routing.Producers projection works".
   - Card 1 makes `Goto`/`Relaunch` read `Block` from the target's definition, and makes `gotoAdmitted` read `SkipTo`.
   - The `Routing` type doc (`internal/shedengine/progress.go:3-5`) lists the fields read in the same way.
   - Fix: add `internal/shedengine#GotoRequest`, `internal/shedengine#RelaunchRequest` and `internal/shedengine#Routing` to card 1's `Edit` list and update the field lists.

5. **LOW** — Card 9: two new members have no `Create` handle.
   - The Intent adds the constant `checklistEnteredWord` and "one helper beside `checklistEntryName`".
   - The spec requires every new member to be declared as a `plan:` handle.
   - Fix: add `plan:internal/loomcli#checklistEnteredWord` -> `const checklistEnteredWord = "entered"`, and a handle with a head for the helper.

6. **LOW** — Card 7: tokencount's doc and generated sentences go stale.
   - `tools/tokencount/doc.go:22` says a run is finished when its seed and status "name the tapestry or the darn recipe".
   - `recipeList` (`tools/tokencount/shed.go:144`) joins `countedRecipes` with the conjunction between every pair. With arras added, `main.go:294-301` print "tapestry and darn and arras" and "tapestry or darn or arras".
   - Fix: list `tools/tokencount/doc.go` and `tools/tokencount#recipeList` on card 7, and render a three-item list properly.

7. **LOW** — Card 7: battenshed's error doc names four admitted recipes.
   - `internal/battenshed/deps.go:25` says `lym loom start` "serves the tapestry, darn, pawl and sampler recipes". That is the doc of `ErrUnsupportedChildRecipe`, whose refusal card 7 widens to arras.
   - Fix: add the variable's glyph to card 7 and add arras to the list.

8. **LOW** — Card 7: a third comment in `battencli/wire.go` stays stale.
   - Card 7 fixes the "four" in `childRecipeSupported`'s doc.
   - `childSeedParams`'s doc (`internal/battencli/wire.go:546`) says "tapestry, darn, sampler and beam declare one each". The function follows `childRecipeSupported`, so arras now gets the param too.
   - Fix: add `internal/battencli#childSeedParams` to card 7.

9. **LOW** — Card 7: the board help text lists recipe names and leaves arras out.
   - `internal/boardcli/cli.go:497` documents the recipe field as `"darn" or "tapestry", or default_recipe`.
   - arras is now a legal recipe value for a board entry, and no card lists this help text.

10. **LOW** — Card 6: three loomrecipe doc comments become stale.
    - `RowNames` (`internal/loomrecipe/names.go:57`) and `AmendmentTarget` (`names.go:85`) list the served recipes by name and say "in file order". After card 6 they serve arras and return expanded order. Neither member is on card 6.
    - The file comments of `bouncerrows.go:1` and `finalizerow.go:1` sit outside the member spans card 6 edits.
    - Fix: add `internal/loomrecipe#RowNames` and `internal/loomrecipe#AmendmentTarget` to card 6.

11. **LOW** — Card 2: the registry file's header comment still names one closed table.
    - `internal/shedrecipe/registry.go:1-4` says "The one table outside it is the closed implementer table". That header sits outside the `registry` var span card 2 edits.
    - The Intent fixes only "the registry's own doc comment".
    - Fix: also list the file's header (its file self glyph is refused next to a member glyph, so name it in the Intent).

12. **LOW** — Card 5: two `shedbuild` docs no longer match.
    - `NewShed` (`internal/shedbuild/newshed.go:52-78`) still documents parse-then-build, but a recipe holding a block row now fails with "not expanded".
    - `routing.go`'s file comment says it declares `RoutingOf` alone.
    - Neither `NewShed` nor the `routing.go` header is listed.

13. **LOW** — gate slot doc and PR-Review stencil comment: both list the recipes that run PR-Review, and arras also holds a PR-Review row.
    - `internal/gateslot/doc.go:112` lists "tapestry, darn, pawl, sampler  PR-Review".
    - The comment in `contracts/stencils/landing/landing-template-pr-review.md:4` says "for the tapestry, darn, pawl and sampler recipes alike".
    - No card lists either.

Tool calls beyond what my context held:
- `lym quarry resolve` over every glyph the cards list, to confirm each resolves; the exploration had not resolved them.
- `lym quarry resolve` on the doubtful glyphs, plus a grep for `checklistVisitsOf`/`RecipeRow{` callers in tests and for `TestShed_Validate`, to check card 9's re-sign callers and card 1's validate test.
- A grep for prose and assertions that enumerate recipe names, to find docs and texts that arras falsifies.
- Read `childSeedParams`, other non-test `RecipeSampler`/`RecipeDarn` sites and tokencount's `recipeList` users, to check card 7's coverage of per-recipe params and sentences.
- Read `loomcli/settledguard.go`'s `segmentWriter` and loomrecipe's row coverage guard, to check whether expanded arras routing or `NameMotif` forces edits there; neither does.
- A grep of tokencount docs, the board help text, `recipeLabel` and stencils for arras/motif, to confirm the stale texts.
