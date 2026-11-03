```yaml
lens: repo-invariants
outcome: reported
```

Task: run the repo-invariants lens over cards 1–9 (test seams, doc text the cards make false, card order).

**Per-card invariants**

- **Card 1 (shedengine).**
  - Seam: held. No `//lym:seam` or `NumField` test reflects over `ProducerDef` or `HistoryEntry`. The JSON-key tests (`historyfold_test.go`, `gateattempts_test.go`, `run_routing_test.go`) are not reflective.
  - Docs: held. `internal/shedengine/doc.go` is under Edit. Its lines 113–117, 142 and 74 describe skip and goto routing by OnDone, and card 1 rewrites them. `loom-status-spec.md` is under Extends and only gains a line.
  - Order: held.
- **Card 2 (shedrecipe).**
  - Seam: held.
  - Docs: held. The `registry` doc and `doc.go` are covered. In `pattern/PATTERN-shed-recipe-registry.md`, "the one exception is Implementer's entry" is about producers, so it stays true.
  - Order: held. At card 2, `TestBuild_EveryRegisteredEngineBuilds` builds `Block` with an empty Config through `blockEntry`, and that passes. The coverage-guard allowlist gains `Block`.
- **Card 3 (shedbuild parse).**
  - Seam: held.
  - Docs: broken. `contracts/specs/shed-recipe-spec.md` is listed by no card (finding 2).
  - Order: held.
- **Card 4 (Expand).**
  - Seam: held.
  - Docs: broken. The same recipe-format spec has no expansion text (finding 2).
  - Order: held.
- **Card 5 (Build, RoutingFrom).**
  - Seam: held.
  - Docs: held. `doc.go`, `routing.go` and the `NewShed` doc are covered.
  - Order: broken. `TestBuild_EveryRegisteredEngineBuilds` fails at this card (finding 1).
- **Card 6 (loomrecipe).**
  - Seam: held. `RecipeRow` has no reflective test.
  - Imports: held. The import allowlist in `seam_enforcement_test.go` is unchanged, since `shedbuild`, `loomshed` and `contracts/recipes` are already imported.
  - Order: held. The allowlist drops `Block`, which `RecipeEngines` now reaches through arras.
- **Card 7 (arras seedable).**
  - Seam: held.
  - Docs: partly broken. `internal/battenshed/doc.go:137` lists recipes and is only under Uses (finding 3).
  - Order: held. `NameMotif` and `RecipeArras` come from card 6.
- **Card 8 (ProfileRecipe).**
  - Seam, docs and order: held. battencli already imports loomrecipe (`overview.go:22`).
- **Card 9 (checklist).**
  - Seam, docs and order: held. `loomcli/doc.go` is under Edit and `shedverbs/doc.go` describes no `last` or `now` line format.
- **New test tiers:** held. Every new test is a function in an existing package, declared tier0.

**Findings**

1. **MEDIUM, consistency, card 5 (`05-shedbuild-block-build.md`): card 5 fails its own gate.**
   - `TestBuild_EveryRegisteredEngineBuilds` builds every name in `shedrecipe.Names()` from a one-row `Recipe{Producers: []Row{{Name: "Row-"+name, Engine: name, …}}}` with no `BlockGate` (`internal/shedbuild/build_engines_test.go:89-128`).
   - Card 5's new branch gives an `EngineBlock` row the gate from `shedrecipe.LookupBlockGate(row.BlockGate)`. Card 2 makes that refuse any name the table lacks, so the empty name is refused.
   - The `Block` subtest therefore fails from card 5 on. Card 5 does not list the test.
   - Fix: list `internal/shedbuild#TestBuild_EveryRegisteredEngineBuilds` under Edit and set `BlockGate: shedrecipe.BlockGateAlways` for the Block engine there.
2. **MEDIUM, consistency, cards 3 and 4: the recipe-format spec goes stale and no card lists it.**
   - `contracts/specs/shed-recipe-spec.md:89-98` names the format's top-level keys and its row keys: "Each row carries `name`, `engine`, `config`, `on_done`, `on_stuck`, `on_reroute`, `segment`, and `max_bounces`". It also states strictness on unknown keys.
   - Card 3 adds `block`, `block_gate` and a top-level `profile_recipe`, and lets a block row carry no `engine`. Card 4 adds inline expansion.
   - No card names the spec (grep of `_lym/plan/*.md` finds no hit), so the spec contradicts the format the plan ships.
   - Fix: card 3 lists the spec under Edit for the row keys, and card 4 extends it with the expansion rules.
3. **LOW, consistency, card 7 (`07-arras-seedable.md`): a battenshed doc sentence leaves out arras.**
   - `internal/battenshed/doc.go:137` reads "a tapestry, darn or sampler fixer's escalate halt, which the arbiter judges first, is marked and skipped too".
   - Card 7 makes arras a child recipe that has the same fixers, and lists `internal/battenshed/doc.go` only under Uses. The sentence now leaves out arras.
   - Fix: move it under Edit and name arras, or word the sentence by row rather than by recipe name.

**Tool calls**

1. Grep for `lym:seam` and `NumField` or `reflect.TypeOf` tests over the changed structs — the manifest held no seam tests.
2. Grep for history JSON-key pins and the doc text of loomshed, shedverbs, battenshed, boardcli and shedengine — the manifest held only the shedrecipe and shedbuild doc greps.
3. Read `internal/shedbuild/build_engines_test.go` and listed `contracts/specs` — needed the every-engine build test and the recipe-format spec, which were not read.
4. Grep `contracts/specs/shed-recipe-spec.md` for its key lists, the plan for references to it, and battenshed for recipe-name sites — to confirm the stale text and that no card covers it.
5. Grep the loomrecipe and shedbuild seam and overlay guard tests for import allowlists and Parse use — to check card order against the import-allowlist scans.
