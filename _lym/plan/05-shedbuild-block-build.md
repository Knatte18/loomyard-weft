# Card 5 — shedbuild-block-build

**Edit:**
- `internal/shedbuild#Build`
- `internal/shedbuild#RoutingOf`
- `internal/shedbuild#NewShed`
- `internal/shedbuild/doc.go`
- `contracts/specs/shed-recipe-spec.md`
- `internal/shedbuild#TestBuild_Errors`
- `internal/shedbuild#TestBuild_EveryRegisteredEngineBuilds`
- `internal/shedbuild#TestBuild_EntryGate`
- `internal/shedbuild#TestBuild_RowsCarryRoutingInRecipeOrder`
- `internal/shedbuild#TestRoutingOf_MatchesNewShedRowForRow`
- `internal/shedbuild#TestRoutingOf_ParseErrorSurfaces`

**Create:**
- `internal/shedbuild#RoutingFrom`

**Uses:**
- `internal/shedrecipe#LookupBlockGate`
- `internal/shedrecipe#BlockGateNames`
- `internal/shedrecipe#EngineBlock`
- `internal/shedbuild#Expand`

**Intent:** Build an expanded recipe's block rows and project their routing (D3, D5, D11).

`Build`:
- first refuses a row that carries `Block` while its engine is not `Block`, a block row that was never expanded: `shedbuild: producer %d %q: block row is not expanded; way forward: expand the recipe with shedbuild.Expand before building it`.
  The check runs ahead of the loom contract check, so an unexpanded loom recipe gets this way forward and not a contract clause; reached by building a parsed recipe holding a block row;
- builds a row whose `Engine` is `shedrecipe.EngineBlock` through the registry like any row, and gives it the entry gate `shedrecipe.LookupBlockGate(row.BlockGate)` builds over `env`;
  a `block_gate` naming no check is refused as `shedbuild: producer %d %q: <LookupBlockGate's refusal>`, which names the shipped checks;
- copies `row.SkipTo` and `row.BlockPath` onto every `ProducerDef` as `SkipTo` and `Block`.

`RoutingFrom` projects an already-expanded recipe's routing as `RoutingOf` does today, each row's `SkipTo` and `Block` included; `RoutingOf` parses, refuses a parsed recipe holding a block row with the same not-expanded refusal, and delegates to `RoutingFrom`.
`routing.go`'s file comment names both functions.
`NewShed`'s doc states that it builds a recipe without expanding it, so a recipe holding a block row goes through `Expand` and `NewShedFrom`.

`doc.go` states that `Build` and `RoutingFrom` take the expanded recipe and that a block row's gate is the named check's.
`shed-recipe-spec.md`'s exported-function paragraph names `RoutingFrom` and says the same of `Build` and `RoutingFrom`.

Tests, tier0:
- `TestBuild_Errors` gains the unknown `block_gate` row, asserting the row's name, the way forward and `always` in the error, and the unexpanded block row, built with `Contract: "loom"` so the row asserts the not-expanded refusal wins over the contract check.
- `TestBuild_EveryRegisteredEngineBuilds`'s one-row recipe sets `BlockGate: shedrecipe.BlockGateAlways` on the row whose engine is `shedrecipe.EngineBlock`, so the every-engine build still builds Block.
- `TestBuild_EntryGate` gains a row: an expanded block row with `block_gate: always` builds a gate that holds.
- `TestBuild_RowsCarryRoutingInRecipeOrder` sets `SkipTo` and `BlockPath` on a row and asserts both reach the definition.
- `TestRoutingOf_MatchesNewShedRowForRow` gains a second recipe, a parent with one block over a map resolver, run through `Expand`, then `NewShedFrom` and `RoutingFrom`, comparing `SkipTo` and `Block` row for row beside today's fields.
- `TestRoutingOf_ParseErrorSurfaces` gains a recipe holding a block row, refused naming `shedbuild.Expand`.

**ImpactSummary:** `Build` gains a block-row branch, an unexpanded-row refusal and two copied fields, and `RoutingOf` delegates to a new projection; no shipped recipe holds a block row yet, so every existing build and projection is unchanged.
