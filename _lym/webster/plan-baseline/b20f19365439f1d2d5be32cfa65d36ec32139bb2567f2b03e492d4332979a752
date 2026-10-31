# Card 3 — loom-contract-check

**Edit:**
- `internal/shedbuild#Recipe`
- `internal/shedbuild#Build`
- `internal/shedbuild#TestParse_ValueAndErrorCases`
- `internal/shedbuild/doc.go`
- `internal/shedrecipe/doc.go`
- `contracts/recipes/tapestry-recipe.yaml`
- `contracts/specs/shed-recipe-spec.md`

**Create:**
- `plan:internal/shedrecipe#LoomRecordEngines` -> `var LoomRecordEngines = []string{"DiscussionWrite", "RecordWrite"}`
- `plan:internal/shedrecipe#LoomBuildEngines` -> `var LoomBuildEngines = []string{"Implementer", "DarnWrite"}`
- `plan:internal/shedrecipe#LoomFinalizeEngine` -> `const LoomFinalizeEngine = "Finalize"`
- `plan:internal/shedbuild#checkLoomContract` -> `func checkLoomContract(r Recipe) error`
- `plan:internal/shedbuild#TestBuild_LoomContract` -> `func TestBuild_LoomContract(t *testing.T)`

**Intent:** A recipe declares itself a loom with a top-level `contract: loom` key, and Shed checks the contract when it builds the recipe (D1).

`Recipe` gains `Contract string \`yaml:"contract"\`` and `VerifySource string \`yaml:"verify_source"\``, decoded only by `Parse` (PATTERN-sole-parsers); Parse checks only the shape: a `contract` other than empty or `loom`, and a `verify_source` other than empty, `plan` or `tree_scope`, are refused naming the key, the legal values and the way forward "set it to one of them or remove it".

`Build` calls `checkLoomContract` first when `r.Contract == "loom"`, before any registry lookup, and returns its refusal wrapped `shedbuild:`.
`checkLoomContract` reads row engines, edges and the recipe's keys only, the engine sets coming from `shedrecipe.LoomRecordEngines`, `LoomBuildEngines` and `LoomFinalizeEngine`, declared beside the registry so a new loom recipe needs no change to the check (PATTERN-shed-recipe-registry). Each refusal names the broken clause and its way forward (PATTERN-refusal-way-forward), in this order:
1. exactly one record row (engine in `LoomRecordEngines`): zero or two refused, "way forward: give the recipe exactly one record row, engine DiscussionWrite or RecordWrite";
2. exactly one build row (engine in `LoomBuildEngines`), refused likewise naming Implementer and DarnWrite;
3. the build row lies after the record row on the `on_done` path walked from `entry` (follow `on_done` from the entry row until a terminal, an empty `on_done` or a revisited row; the record row must be met before the build row), else "way forward: place the build row after the record row on the on_done path from the entry";
4. a `Finalize` row exists and every row reached by following `on_done` from it is in `terminals`, else "way forward: end the recipe's on_done path at Finalize followed only by terminals";
5. `verify_source` is set, else "way forward: set verify_source to plan or tree_scope".
A recipe without `contract: loom` is not checked and is no loom.
The build segment is the rows a variant owns that turn the record into the change (Added after Discussion: "The contract names a build segment, not a writer"); the check holds the one row of the segment whose engine is a build engine, and no identifier or message calls it a writer.

`tapestry-recipe.yaml` gains `contract: loom` and `verify_source: plan` at the top level, beside `amendment_target`, with a header line saying so. Darn declares the contract in card 11, once it has its record row; pawl and batten never do.

Tests: `TestParse_ValueAndErrorCases` gains rows decoding `contract` and `verify_source` into the fields and refusing an unknown value of each. `TestBuild_LoomContract` is table-driven over inline recipes that declare `contract: loom` with Stub-engine filler rows and the named engines: a valid recipe makes `checkLoomContract` return nil, called directly since the test sits in the package; and `Build` refuses, with the `shedbuild:` prefix, one row each with no record row, two record rows, no build row, two build rows, the build row before the record row on the on_done path, no Finalize row, a Finalize whose on_done successor is not a terminal, and no verify_source, each asserting the clause and its `way forward:` text; a recipe without `contract` holding none of those rows makes `checkLoomContract` unreached, so `Build` returns no contract refusal for it. Loading the shipped tapestry recipe through `loomrecipe.New` stays covered by `internal/loomrecipe#TestNew_ShapeMatchesRecipe`.

`shed-recipe-spec.md` documents the top-level `contract` and `verify_source` keys and the five clauses; `internal/shedbuild/doc.go` documents the check in Build; `internal/shedrecipe/doc.go` names the three engine sets beside the registry.

**ImpactSummary:** Adds two decoded keys and a Build-time check that fires only for recipes declaring contract: loom; tapestry is the one such recipe after this card, and no other recipe's build changes.
