# Card 2 — drop-beam-alias

**Edit:**
- `internal/shedrun#decodeSeed`
- `internal/shedrun#TestParseSeed`
- `internal/shedrun#TestValidateRecipe`
- `internal/shedcli#armFromSeed`
- `internal/shedcli#TestArmFromSeed_RefusalEnvelopeKinds`

**Delete:**
- `internal/shedrun#legacyRecipeBeam`

**Uses:**
- `internal/shedcli/doc.go`

**Intent:** Remove the legacy recipe name `beam` (D11). `decodeSeed` stops rewriting a recorded `beam` to `pawl` and decodes the recipe as written, still accepting unknown keys (PATTERN-tolerant-records); `legacyRecipeBeam` goes.

A seed recording `beam` then reads as recipe `beam`, a name no recipe table holds: `lym shed`'s recipe lookup refuses it as an unknown recipe naming the available recipes, as it refuses any unknown name. As the Added after Discussion entry "A beam seed names no recipe this lym knows" requires, the refusal names the re-seed way forward: when lookup of the seed's recipe fails, `armFromSeed` appends `; way forward: remove the run's seed.json and seed the run again with "lym shed seed <run-id> --recipe <recipe>"`, the `lym shed` form of the re-seed route card 19 gives `lym loom`, so `step`, `status` and a looping step carry it in place of the generic way forward. Every row of `TestArmFromSeed_RefusalEnvelopeKinds` that seeds an unknown recipe, "unknown recipe on step", "unknown recipe on status" and each `loop:` row seeding `bogus-recipe`, asserts that text as its suffix and counts one way-forward clause. The kind-less branch of `TestArmFromSeed_RefusalEnvelopeKinds`, the `wantKind == ""` early return, also asserts `wantSuffix` on the envelope's error text and counts its `way forward:` clauses against `wantWayForwardCount` when the row sets them, and the "unknown recipe on status" row sets `wantSuffix` to the re-seed text and `wantWayForwardCount` to 1, so the status path checks both too.
`TestParseSeed`'s "former beam recipe reads as pawl" subtest becomes "a beam seed keeps the recipe it records" (`Recipe == "beam"`, no error), the row that fails if the alias survives; `TestValidateRecipe`'s "former beam name" row stays refused.
`lym loom`'s refusal of a `beam` seed as naming no recipe this lym knows, with its re-seed way forward, arrives with card 19.

**ImpactSummary:** One decode branch and its constant go; only a seed recording `beam` changes reading, and no live seed records it. Every `lym shed` refusal of a seed naming an unknown recipe now names the re-seed way forward in place of the generic one.
