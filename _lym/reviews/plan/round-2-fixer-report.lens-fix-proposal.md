```yaml
lens: fix-proposal
outcome: reported
```

I checked the reviewer's proposals the fixer carried out as written (F1, F2, F4), looking for cases where the proposal itself is wrong. I found none.

- **F1:** `upsertPayload` (`internal/boardcli/cli.go:459`) is built from `boardengine.UpsertPayloadKeys()`. So the only hand-written copy of the keys is the `upsert` `Use` token (`cli.go:479`), and card 1 now edits it.
  - `upsert-batch` and `merge` spell `&lt;task&gt;`/`[&lt;task&gt;]`, which `parsePayloadValue` maps to `upsertPayload`, so neither needs a token edit.
  - The `rename` help stays in `Command`, so card 5 still lists it beside `storeVerbs`.
- **F2:** I tried to build a recipe that gets past `resolveReviewTarget` but makes the accessors fail, and could not.
  - `serviceOfRecipe` calls `IsLoom`, which fails only when `parseEmbeddedRecipe` fails (`recipekeys.go:35`, `names.go:116`).
  - `preRun` takes its recipe from the same `seedRecipe(target)` (`decision.go:867`).
  - Card 6 has both accessors parse through that same `parseEmbeddedRecipe`, with an empty name read as tapestry in both places.
  - So a recipe that was not refused always parses, and dropping the error is safe.
- **F4:** `rename.go`'s header (lines 1–3) only says a refused rename writes nothing, and that stays true after card 2. A saved rename whose record move fails is not a refusal, so leaving the header alone breaks nothing.

no findings

- Call 1: read `decision.go` 836–900 and `arm.go` 135–185, to check where `preRun` gets its recipe and how `serviceOfRecipe` classifies it. My context had neither.
- Call 2: read `IsLoom`, `parseEmbeddedRecipe`, card 6's Intent and `seedRecipe`, to check whether the accessors can fail on a recipe that `resolveReviewTarget` accepts. My context did not have them.
- Call 3: grepped `upsertPayload` and `on_land` in boardcli, to check whether the declared upsert keys are a second hand-written list that card 1 also has to edit. My context did not show it.
- Call 4: read `rename.go`'s file header, to check that dropping the header edit leaves no false sentence. My context did not show the header.
