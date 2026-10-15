# Card 9 — recipe-code-rounds

**Create:**
- `internal/loomrecipe/languageskills_test.go`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`

**Uses:**
- `internal/shedbuild#Parse`

**Intent:**
Mark the code rounds: the `Webster-Burler-Review`, `Webster-Burler-Fix`, `Merge-Burler-Review` and `Merge-Burler-Fix` rows of the loom recipe and the `Darn-Burler-Review` and `Darn-Burler-Fix` rows of the darn recipe each gain `language-skills: true` in their `profile` map, with a comment that the round reviews or fixes code, so its seats load the hub's language skills after their own (D5).
The Discussion and Plan review rows and their fix rows stay without the key: they review prose, not code.

`internal/loomrecipe/languageskills_test.go` (package `loomrecipe`) adds `TestShippedRecipes_LanguageSkillsOnCodeRounds`, which parses `recipes.LoomRecipe` and `recipes.DarnRecipe` with `shedbuild.Parse` and, per recipe, collects every row whose `profile` map carries `language-skills`;
it asserts each such value is the boolean `true` and that the collected row names equal exactly the four loom rows and the two darn rows above, so a code row losing the key or a Discussion or Plan row gaining it fails.
Building each recipe's Burler rows through the card 8 mappers is already covered by the package's recipe build tests, which construct every shipped row.

**ImpactSummary:** six recipe rows gain one profile key that the card 8 mappers accept; no routing, segment or gate changes.
