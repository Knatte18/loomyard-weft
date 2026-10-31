# Card 7 — recipe-key-accessors

**Create:**
- `internal/loomrecipe#IsLoom`
- `internal/loomrecipe#ServedByLoom`
- `internal/loomrecipe#HasEngine`
- `internal/loomrecipe#RecordEngine`
- `internal/loomrecipe#VerifySource`
- `internal/loomrecipe#RetiredRows`
- `internal/loomrecipe#TestRecipeKeys`

**Extends:**
- `internal/loomrecipe/doc.go`

**Uses:**
- `internal/loomrecipe#parseEmbeddedRecipe`
- `internal/shedrecipe#LoomRecordEngines`

**Intent:** Give every consumer one place to read a recipe's contract, keys and rows by recipe name, so later cards replace branches on `RecipeDarn`/`RecipeTapestry` with these reads (D2, D3, D8). Each parses the embedded recipe through `parseEmbeddedRecipe`, which refuses a name that selects no embedded recipe; an empty name reads as `tapestry`, as an unseeded run does everywhere.

- `IsLoom` is true when the recipe declares `contract: loom` (tapestry; darn from card 11).
- `ServedByLoom` is the one place the pawl exception is named (D2): it is `IsLoom` or the recipe is `shedrun.RecipePawl`, "the single non-loom recipe `lym loom` still serves until `pawl-loom-recipe` lands". It admits pawl alone; batten and any unknown name answer false (an unknown name with the refusal error from parsing).
- `HasEngine` reports whether some row of the recipe carries the engine, so a verb or wiring keys on rows (`PlanWrite`, `Implementer`, `RecordWrite`), never the name.
- `RecordEngine` returns the engine of the recipe's one row whose engine is in `shedrecipe.LoomRecordEngines`, empty when it has none (pawl, batten).
- `VerifySource` returns the top-level `verify_source`, empty when unset.
- `RetiredRows` returns each recipe's retired-row map as that recipe's data, read from a package-level table keyed by recipe name: tapestry's `Merge-In` → Parent-Align and `Loom-Preflight` → Tapestry-Preflight, darn's `Darn` → Darn-Write, and nil for pawl, batten and any other name. The existing `TapestryRetiredRows` and `DarnRetiredRows` stay until card 8 folds them into it.

`TestRecipeKeys` (tier0) is table-driven per recipe: tapestry is loom and served with record engine DiscussionWrite, verify source plan, an Implementer row and a PlanWrite row; pawl is served but not loom and has no record engine; batten is neither; `beam` and `nope` are refused by every accessor; `RetiredRows` of tapestry and darn equal today's two maps and of pawl is empty. The package doc names the accessors.

**ImpactSummary:** Adds read-only accessors over the embedded recipes; nothing calls them yet and no existing member changes.
