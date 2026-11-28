# Jacquard recipe holds tapestry's rows after Discussion, tapestry becomes the Motif and Jacquard blocks, and a Record-Seed row seeds the stored record

`jacquard-recipe` · Task · feature, shed, loom, testkit · landing · Finalize

Decision record: [Sketch](decision-jacquard-recipe.md)

Filed from the Sketch `recipe-build-from-sketch`.

Takes: D1 whole: contracts/recipes/jacquard-recipe.yaml holding Preflight, Record-Seed and every tapestry row from Drift-Baseline to Friction-Reflect, moved with routing, segments, gates and comments, never copied, with jacquard's top-level keys; shedrun's recipe-name constant jacquard in the closed vocabulary (RecipeNames, OfferedRecipeNames, ValidateRecipe) and shedcli's recipe table kept in step with it; the embed in contracts/recipes/recipes.go; internal/loomrecipe's embedded-recipe map and the loomrecipe recipe lists jacquard's own rows need to run alone and as a block (RecipeEngines, bouncerrows.go, finalizerow.go). D2 whole: tapestry-recipe.yaml as the Motif block row (block_gate undiscussed, on_done Jacquard) and the Jacquard block row (block_gate always, no on_done), entry Motif, row names unchanged so an in-flight tapestry run resumes by row name; the tapestry and motif recipe headers rewritten for the move and naming jacquard, which is D10's recipe-header step. D3 whole: shedbuild.Expand admits a last outermost block row with empty on_done and block_gate always, refuses a last one with another gate with the way forward "set on_done, or use block_gate always", and refuses every other outermost empty on_done as today. D4's engine and contract steps: the RecordSeed registry entry in internal/shedrecipe, the RecordSeed producer in internal/loomshed (Record-Write's stored-only mode, writing only a missing record or support log and committing the pair, Stuck with nothing written when the record is missing and no stored record exists, its reason naming the tapestry goto, the jacquard batten sequence and the jacquard re-seed with the seed's path, driver and parent filled in as disagreeingSeedRefusal does), its Env wiring in internal/loomcli/wiring.go and testkit/envkit, the widened loom contract in internal/shedbuild/loomcontract.go (at most one record row, at most one seed row, at least one, seed after record on the on_done path, the seed read as the record row when no record row exists), and loomrecipe.RecordEngine returning RecordSeed for a recipe with only a seed row. Per the added entry, no plan or build recipe, pair handover or Worktree-Teardown skip.
Delivers: A jacquard recipe that builds as a loom recipe from Preflight to Friction-Reflect; tapestry expanded as Motif's rows followed by Jacquard's with the same edges; a tapestry run of an undiscussed entry walking Motif, Discussion, Preflight, Record-Seed (writing nothing) and Drift-Baseline; Record-Seed seeding, keeping or Stuck per D4; the loom contract building tapestry, jacquard, darn and motif under the widened clauses.
Exports: Fixed for later parts: shedrun.RecipeJacquard ("jacquard"); the engine name RecordSeed in shedrecipe's registry and beside LoomRecordEngines; loomshed.NameRecordSeed ("Record-Seed"); the block row name Jacquard in tapestry-recipe.yaml; loomrecipe.RecordEngine returning DiscussionWrite for tapestry and RecordSeed for jacquard; loomrecipe.EngineRow unchanged in signature.

The part changes these packages:

- contracts/recipes (about 490 changed lines)
- internal/shedrun (about 20 changed lines)
- internal/shedcli (about 5 changed lines)
- internal/shedrecipe (about 110 changed lines)
- internal/shedbuild (about 190 changed lines)
- internal/loomshed (about 270 changed lines)
- internal/loomrecipe (about 190 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/testkit/envkit (about 10 changed lines)
