# Batten, loom verbs and the report admit jacquard where it walks tapestry's path, and cost cells count a discussed tapestry run as jacquard

`jacquard-recipe-lists` · Task · feature, batten, loom, report, tools

Decision record: [Sketch](decision-jacquard-recipe-lists.md)

Depends on: [`jacquard-recipe`](design-jacquard-recipe.md)

Filed from the Sketch `recipe-build-from-sketch`.

Takes: jacquard-recipe's shedrun.RecipeJacquard, loomshed.NameRecordSeed and the Jacquard block row. D8 whole but loomrecipe's lists, which jacquard-recipe adds: every non-test closed recipe list naming shedrun.RecipeTapestry for a path a jacquard run walks names jacquard too, found by grepping for it outside tests: batten's admitted child recipes and their refusal text (internal/battencli/wire.go childRecipeSupported and unsupportedChildRecipeRefusal), loomcli's lists (arbiterprompt.go, arm.go, peers.go, wiring.go), tools/tokencount/shed.go, and the report's selected run recipes (internal/reportengine/runs.go); the report's jacquard phase table is tapestry's without its Discussion families, and both tables put the Jacquard block row and Record-Seed in Preflight's phase; a default that falls back to tapestry when a run has no seed (sharedbootstrap.go resolveSeedRecipe) stays tapestry. D9 whole: internal/reportengine's cost cells key a tapestry run whose stored_record is true under jacquard, and only the cell key changes.
Delivers: Batten admits jacquard as a child recipe; lym report selects jacquard runs with their phase table; a tapestry run with stored_record true lands in the jacquard cost cell and the tapestry cell with stored_record true is left empty.
Exports: No new symbol; batten's admitted child recipes include jacquard, which jacquard-start-refusal's batten tests rely on.

The part changes these packages:

- internal/battencli (about 20 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/reportengine (about 190 changed lines)
- tools/tokencount (about 5 changed lines)
