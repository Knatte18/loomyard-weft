# Card 31 — beam-recipe

**Create:**
- `contracts/recipes/beam-recipe.yaml`
- `plan:contracts/recipes#BeamRecipe` -> `var BeamRecipe []byte`
- `plan:internal/shedrun#RecipeBeam` -> `const RecipeBeam = "beam"`
- `plan:internal/shedrecipe#beamDescribeEntry` -> `func beamDescribeEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/loomrecipe#NewBeam` -> `func NewBeam(env shedrecipe.Env, paths shedbuild.ShedPaths) (*shedengine.Shed, error)`
- `plan:internal/loomrecipe#BeamRouting` -> `func BeamRouting() (shedengine.Routing, error)`
- `plan:internal/loomrecipe#TestNewBeam` -> `func TestNewBeam(t *testing.T)`

**Edit:**
- `internal/shedrun#RecipeNames`
- `internal/shedrun#TestRecipeNames`
- `internal/shedrun#TestValidateRecipe`
- `internal/shedcli#recipes`
- `internal/shedcli#TestRecipes_KeySet`
- `internal/shedcli#TestRecipes_Entries`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#TestNames`
- `internal/loomrecipe#RecipeEngines`
- `internal/loomrecipe#parseEmbeddedRecipe`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder`

**Extends:**
- `internal/shedrecipe/doc.go`
- `internal/loomrecipe/doc.go`

**Uses:**
- `plan:internal/landingshed#NewBeamDescribe`
- `plan:internal/landingshed#BeamDescribeDeps`

**Intent:**
D11's landing recipe `beam`, registered as PATTERN-shed-recipe-registry requires.
`contracts/recipes/beam-recipe.yaml` has two rows and no review segment: `Beam-Describe` (engine `BeamDescribe`, `on_done: Finalize`) and `Finalize` (engine `Finalize`, `on_done: ""`, no `on_reroute`), entry `Beam-Describe`, terminal `Finalize`; its header comment says the Finalize never reroutes (card 29) and why, and that a halt goes to the orch.
`recipes.BeamRecipe` embeds it; `shedrun.RecipeBeam` joins the closed recipe vocabulary (`RecipeNames`, `ValidateRecipe`), and `shedcli`'s `recipes` table arms it through `loomcli.ArmAt` with loom's verbs and bootstrap verb, since loom's bootstrap runs it (card 32).
The registry gains `BeamDescribe`, `beamDescribeEntry`, which rejects every config key and builds `landingshed.NewBeamDescribe` from `Env.BeamDescribe`, a new `Env` field of type `landingshed.BeamDescribeDeps`, refusing a zero value through `requireSeam`.
`loomrecipe.NewBeam` parses `BeamRecipe` and builds it against env with the status-path checks `New` makes; `BeamRouting` projects its routing; `RecipeEngines` and `parseEmbeddedRecipe` (behind `Rows` and `RowNames`) include it, so the cross-consumer coverage guard reaches `BeamDescribe`.
`TestNewBeam` builds the shed from a filled env and asserts its two rows, their edges and that Finalize has no reroute edge, and that a missing `BeamDescribe` seam is refused; `TestRecipeNames`, `TestValidateRecipe`, `TestRecipes_KeySet` and `TestRecipes_Entries` add `beam`; `TestNames` lists `BeamDescribe`; `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` and `TestRowNames_FollowEachRecipesRowOrder` add the beam recipe's engines and rows.

**ImpactSummary:** The closed recipe vocabulary, the shed arming table and the engine registry each gain one entry; loom and darn recipes are untouched.
