# Relevant files

- `contracts/specs/implementer-spec.md` — pins the implementer row's in, out and outcomes Weaver must meet
- `contracts/specs/webster-spec.md` — Webster's run directory, verify gate and summary, the model Weaver mirrors
- `internal/shedrecipe/entries_implementer.go` `implementerEntry`, `implementers` — the closed implementer table Weaver joins
- `contracts/recipes/tapestry-recipe.yaml` — the Webster row, its edges and its run-level gates entry
- `internal/loomshed/webster.go` `NewWebsterProducer` — how the implementer row's producer is built and resolves the batcher
- `internal/loomcli/arbiterprompt.go` `arbiterRecipeDescription` — the shared tapestry description the arbiter sees
- `pattern/PATTERN-shed-recipe-registry.md` — the Implementer exception to the registry map
- `pattern/PATTERN-shed-driver.md` — the stop chain and the shared-machinery rule
- `pattern/PATTERN-shuttle-provider-seam.md` — model choice mapping and the shuttle stop verb
