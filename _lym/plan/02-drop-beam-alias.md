# Card 2 — drop-beam-alias

**Edit:**
- `internal/shedrun#decodeSeed`
- `internal/shedrun#TestParseSeed`
- `internal/shedrun#TestValidateRecipe`

**Delete:**
- `internal/shedrun#legacyRecipeBeam`

**Uses:**
- `internal/shedcli#TestArmFromSeed_RefusalEnvelopeKinds`

**Intent:** Remove the legacy recipe name `beam` (D11). `decodeSeed` stops rewriting a recorded `beam` to `pawl` and decodes the recipe as written, still accepting unknown keys (PATTERN-tolerant-records); `legacyRecipeBeam` goes.

A seed recording `beam` then reads as recipe `beam`, a name no recipe table holds: `lym shed`'s recipe lookup refuses it as an unknown recipe naming the available recipes, as it refuses any unknown name, and `armFromSeed` closes that refusal with the generic way forward, which `TestArmFromSeed_RefusalEnvelopeKinds`' existing "unknown recipe on step" and "unknown recipe on status" rows already pin.
`TestParseSeed`'s "former beam recipe reads as pawl" subtest becomes "a beam seed keeps the recipe it records" (`Recipe == "beam"`, no error), the row that fails if the alias survives; `TestValidateRecipe`'s "former beam name" row stays refused.
`lym loom`'s refusal of a `beam` seed as a non-loom recipe arrives with card 19.

**ImpactSummary:** One decode branch and its constant go; only a seed recording `beam` changes reading, and no live seed records it.
