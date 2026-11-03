# Card 8 — profile-recipe

**Create:**
- `plan:internal/loomrecipe#ProfileRecipe` -> `func ProfileRecipe(recipe string) (string, error)`

**Edit:**
- `internal/battencli#prepareRunSettings`
- `internal/battencli#TestPrepareRunSettings`
- `internal/loomrecipe#TestRecipeKeys`

**Extends:**
- `internal/loomrecipe/doc.go`

**Intent:** Let a task's profile block come from the recipe its `profile_recipe` names (D8).

`loomrecipe.ProfileRecipe` is the one accessor every reader of a task's profile block resolves the recipe name through: it returns the named recipe's `profile_recipe`, or the recipe's own name when the key is empty; an empty name reads as tapestry, and a name that selects no embedded recipe is refused as `parseEmbeddedRecipe` refuses it.
`prepareRunSettings`, the one production reader of a profile block by recipe, resolves `settings.Recipe` through it before `profiles.Default` and `profiles.Settings`.
When `ProfileRecipe` refuses, `prepareRunSettings` keeps `settings.Recipe` as the task named it.
A task whose recipe names no embedded recipe and that names no profile therefore passes `prepareRunSettings` with its settings alone,
and Seed-Child's `WriteSeed` seam then refuses it through `shedrun.ValidateRecipe`, wrapped in `battenshed.ErrUnknownRecipe`, naming the available recipes, as it does today.
A task that also names a profile is still refused by `profiles.Settings`, as it is today, because the profile holds no block for that name.
Bound: `profile_recipe` only chooses which recipe's profile block and default profile a task's settings come from; it changes no row or config module, and only a recipe's top-level key sets it.

`doc.go` gains a sentence on `ProfileRecipe`.

Tests, tier0:
- `TestRecipeKeys` gains rows: `ProfileRecipe("arras")` is `tapestry`, `ProfileRecipe("tapestry")` and `ProfileRecipe("")` are `tapestry`, `ProfileRecipe("darn")` is `darn`, and an unknown name is refused.
- `TestPrepareRunSettings` gains rows, none over a profiles file with an `arras` block:
  - with no profiles file, an arras task naming no profile takes the template's tapestry default profile's tapestry block, the same documents a tapestry task gets;
  - over the `profiles` fixture, an arras task naming `light` takes `light`'s tapestry block;
  - a task whose recipe names no embedded recipe and no profile writes the task's settings alone, with no refusal.

**ImpactSummary:** One accessor in loomrecipe and one resolve step in Seed-Child's prepare; every existing recipe resolves to its own name, so their tasks' profiles are unchanged.
