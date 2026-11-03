---
format: 5
approved: false
language: go
---

# Plan: a shed block runs a whole sub-recipe inline

A new recipe row kind, the block, names another recipe whose rows are inlined into the parent at build time and entered as one step through a block gate check lym ships.
The engine learns a skip target and a block path, the registry gains the `Block` producer and the closed `BlockGates` table, shedbuild parses block rows and expands them, and loomrecipe reads every embedded recipe expanded through one table.
The block is proven on `motif`, a block source holding tapestry's Discussion rows, and `arras`, a seedable loom recipe running `motif` through the `Motif` block before tapestry's rows from Drift-Baseline on; `arras` takes tapestry's profile block through `profile_recipe`, and the status checklist marks blocks.
Tapestry's recipe file is unchanged.

## Card Index

1 — shedengine-block-routing — skip target and block path on rows and history entries, read by the skip, goto admission and history writes
2 — shedrecipe-block-registry — the Block producer and the closed BlockGates table with always, and the PATTERN entry
3 — shedbuild-block-row-parse — block row and profile_recipe keys in the recipe format, with their parse refusals
4 — shedbuild-expand — Expand inlines each block's sub-recipe into its parent
5 — shedbuild-block-build — Build builds expanded block rows with their gate, and RoutingFrom projects them
6 — loom-recipes-motif-arras — the motif and arras recipes, and every loom accessor reading the expanded recipe
7 — arras-seedable — arras accepted and wired wherever tapestry is by name
8 — profile-recipe — ProfileRecipe resolves the profile block's recipe for Seed-Child
9 — checklist-block-marks — the status checklist and terminal view read a block row's Done as entered and name an inlined row's block

## Shared Decisions

### Decision: names

- **Decision:** The Discussion-only block source is `motif` (`shedrun.RecipeMotif`), the new recipe `arras` (`shedrun.RecipeArras`), its block row `Motif` (`loomshed.NameMotif`), the engine `Block` (`shedrecipe.EngineBlock`) and the one check `always` (`shedrecipe.BlockGateAlways`), each one named constant.
- **Rationale:** the decision record's added-after-Discussion entry reads every `cartoon` as `motif`, so the record's `Cartoon` row is `Motif`.
- **Applies to:** cards 2, 6, 7, 9

### Decision: field names

- **Decision:** `shedbuild.Row` carries `Block`, `BlockGate` (YAML keys) and `SkipTo`, `BlockPath` (`yaml:"-"`, set only by `Expand`); `shedengine.ProducerDef` carries `SkipTo` and `Block`; `shedengine.HistoryEntry` carries `Block` (`block,omitempty`); `loomrecipe.RecipeRow` carries `BlockPath`, `SkipTo` and `IsBlock`.
  A block path joins the enclosing block rows' names, outermost first, with `/`.
- **Rationale:** one spelling per fact across the four layers keeps the routing projection, the build and the status reader in step.
- **Applies to:** cards 1, 3, 4, 5, 6, 9

### Decision: the block gate table

- **Decision:** the closed table is a value of the named map type `shedrecipe.BlockGates`, unexported as `blockGates` and reached only through `LookupBlockGate` and `BlockGateNames`, as the registry is reached through `Lookup` and `Names`.
- **Rationale:** the declared PATTERN text names one `Constructor` or `BlockGates` map; the type carries the name while the table stays closed.
- **Applies to:** cards 2, 5

### Decision: one resolver for every embedded recipe

- **Decision:** loomrecipe tells `embeddedRecipeSource` as the resolver when it expands any embedded recipe, pawl included; battenrecipe's loader projects the batten recipe through `RoutingOf` and tells none.
- **Rationale:** D11 makes loomrecipe's table the one expansion point, and `Expand` returns a recipe holding no block row unchanged, so telling the resolver to the pawl recipe changes nothing while a second, resolver-less parse path would be the disagreement D11 rules out.
- **Applies to:** card 6

### Decision: sketch in docs

- **Decision:** docs and help text this plan writes for motif and arras say motif is the recipe that makes a sketch, the decision record; no code identifier, file name or verb of the decision record is renamed.
- **Rationale:** the operator's added-after-Discussion entry naming the artifacts.
- **Applies to:** cards 6, 7

## PATTERN changes

- `PATTERN-shed-recipe-registry` — changes: Adding producer, recipe row or block gate: one `Constructor` or `BlockGates` map in `internal/shedrecipe`, bar `Implementer`'s; a row's value looked up by key, a side action a `Command` row, a row off by one config key its `entry_gate`, one whose off reads more its own call.
