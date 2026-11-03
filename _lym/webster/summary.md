# Shed block rows: motif and arras recipes, profile_recipe and block-aware status

All four batches (cards 1–9) recorded `done`.
No batch needed recovery, and no digest listed renames or departures.

## Batch 01, shedengine-block-routing (cards 1–4)

- Card 1: skip target and block path on rows and history entries.
- Card 2: the `Block` producer and the closed `BlockGates` table with `always`.
- Card 3: block row and `profile_recipe` keys with their parse refusals.
- Card 4: `Expand` inlines each block's sub-recipe.
- Head `38ce2996`.
- Deviations outside the batch targets:
  - `internal/shedbuild/expand.go` and `expand_test.go`
  - `internal/shedbuild/parse.go` and `parse_test.go`
  - `internal/shedengine/entry_test.go`
  - `internal/shedrecipe/entries_block.go`
  - `internal/shedrecipe/entries_simple_test.go`

## Batch 05, shedbuild-block-build (cards 5–6)

- Card 5: `Build` builds expanded block rows with their gate, and `RoutingFrom` projects them.
- Card 6: the `motif` and `arras` recipes, and every loom accessor reading the expanded recipe.
- Head `01922415`.
- Deviations: `internal/loomrecipe/loomrecipe.go` and `internal/shedbuild/build.go`.

## Batch 07, arras-seedable (card 7)

- `arras` is accepted and wired wherever `tapestry` is by name.
- Head `566b3092`.
- Deviations:
  - `internal/boardengine/task_test.go`
  - `internal/loomcli/arbiterprompt.go`
  - `internal/reportengine/timingspans.go`
  - `tools/tokencount/time_integration_test.go`

## Batch 08, profile-recipe (cards 8–9)

- Card 8: `ProfileRecipe` resolves the profile block's recipe for Seed-Child.
- Card 9: the status checklist and terminal view read a block row's Done as entered and name an inlined row's block.
- Head `566b5e15`.
- Deviations:
  - `internal/loomcli/checklist_test.go`
  - `internal/loomrecipe/recipekeys_test.go`
  - `internal/shedverbs/human.go` and `human_test.go`

## Scope guard

The glyph scope guard reported only the deviations above, all informational.
No digest carried warnings or `Pattern-Departure:` trailers.

## Verify gate findings

Attempt 1 of 2 failed on the package `github.com/Knatte18/loomyard/cmd/lym`.
The findings file recorded no card touching a failing package.
Its full log is `.lym/verify/verify-2.log`.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `d903e307cfa1af6575c8ddcb14881ff0d6ad7881` fix: keep lym board upsert's Long under the 2000-byte cap
