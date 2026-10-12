# Card 11 — pr-align-row

**Create:**
- `internal/loomshed#NamePRAlign`
- `internal/shedrecipe#prAlignEntry`
- `internal/shedrecipe#armPRAlign`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomshed#InterruptPolicies`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#simpleEntryCases`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#TestRecipe_PublishRoutesThroughPRGate`
- `internal/loomrecipe/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/landingshed#NewPRAlign`

**Intent:**
Put card 10's producer into loom's producer graph as D5's `PR-Align` row between `Publish` and `PR-Gate`; darn's recipe is not touched.

`loom-recipe.yaml`:
- `Publish`'s `on_done` becomes `PR-Align`.
- A new row `PR-Align` with `engine: PRAlign` and `on_done: PR-Gate` is added, with no `segment`, `on_stuck` or `on_reroute`. It never returns `Stuck`, and a row outside the `PR-Review` segment leaves `PR-Gate`'s bounce episode to `PR-Rework`'s `Done`.
- Its comment says the row spawns the alignment agent, ends `Done` without spawning when `PR-Gate` would pass without a pull request or alignment is off, takes no parent lock, and is re-entered with the new head after every `PR-Rework` and moved-parent reroute.
- The header's escalate-to-a-human paragraph says `PR-Align` is outside that set, since it always ends `Done`, as `Friction-Reflect` is.

Wiring:
- `loomshed` declares `NamePRAlign` beside `NamePRGate`. `InterruptPolicies` maps it to `InterruptPolicyReinvoke`, with the comment "re-entry reuses the live spawn or its verdict".
- `shedrecipe` registers `"PRAlign": prAlignEntry` in `registry`. `prAlignEntry` is `prGateEntry`'s twin over `landingshed.NewPRAlign(env.Landing)` (`PATTERN-shed-recipe-registry`), wrapping its error with the `shedrecipe: PRAlign: ` prefix, and the `entries_simple.go` header names it.

Tests:
- `TestNames` gains `PRAlign`.
- `minimalEntries` and `simpleEntryCases` gain a `PRAlign` row whose env runs `validLandingDeps` and then `armPRAlign`. `armPRAlign` is `armPRGate` plus the `Alignment` seams `NewPRAlign` requires.
- `shedrecipe`'s `TestCoverageGuard` sees `PRAlign` reached through loom's `RecipeEngines` with no change.
- `wantProducerTable` gains the `{NamePRAlign, "PRAlign", *landingshed.PRAlign}` row between `Publish` and `PR-Gate`.
- `TestRecipe_PublishRoutesThroughPRGate` asserts `Publish` hands to `PR-Align`, and `PR-Align` hands to `PR-Gate` with no `OnStuck` and no segment. Its doc comment says so.
- The darn recipe is unchanged, and `TestNewDarn_RowsMatchTheDarnTable` already pins its exact row list, so the acceptance criterion that darn has no `PR-Align` row holds under an existing test.
- `envkit.FullEnv` already fills the `Alignment` seams (card 10), so `loomrecipe`'s built recipe constructs the row.

`internal/loomrecipe/doc.go` and `docs/overview.md`'s landing-tail line read `Describe → Merge-In → Merge-Review → Publish → PR-Align → PR-Gate → Finalize`.
The `landingshed` module-table line names the four producers, and darn's tail line stays as it is.

**ImpactSummary:** One new recipe row and its registry entry; every loom run from this commit passes through `PR-Align`, which stays a no-op where alignment is off or no pull request is required, and darn is unchanged.
