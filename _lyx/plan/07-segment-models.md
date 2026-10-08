# Card 7 — segment-models

**Edit:**
- `internal/loomengine/config.go`
- `internal/loomengine/template.yaml`
- `internal/loomengine/config_test.go`
- `internal/loomengine/review.go`
- `internal/loomengine/review_test.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_burler.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/testkit/envkit/envkit.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_test.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Intent:**
Let each review segment set its own reviewer and fixer models (D4), falling back to the run-wide `review` and `fix` lists.

- `loomengine.Config` gains six `ModelSpecList` keys: `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix`, `webster_review`, `webster_fix`.
  `template.yaml` carries each as `""`, with a comment saying empty takes the run-wide `review` or `fix` list and a set value (one model-spec, or a per-round list whose last entry serves every later round) replaces it for that segment.
- `LoadConfig` treats an empty list and a list of one empty string as unset; every entry of a set list is validated through `modelspec.Parse` at load, refused naming the key and the entry index with the existing model-spec list way forward.
  The `config.go` header lists the new keys among those it validates.
- `ResolveReview` resolves the three segments' `burlerengine.RoundModels` into `ReviewSettings` (`Discussion`, `Plan`, `Webster`), each list falling back independently to the resolved run-wide `review` or `fix` list, every entry resolved through the registry at run start as today.
- `shedrecipe.Env` gains `RowReviewModels map[string]burlerengine.RoundModels`, keyed by BurlerRound row name, a run-wide value like `SegmentBounces`; `burlerRoundEntry` passes the entry for its own row name to `BurlerDeps.Models` and falls back to `Env.ReviewModels` when the map has none.
- `envkit`'s `nilLegal` gains `RowReviewModels` beside `SegmentBounces`, since a nil map is the documented fallback to `Env.ReviewModels`; `envkit.NilSeams` therefore skips it and `battencli`'s `TestWire_EverySeamFilled`, whose `wire` leaves it nil, keeps passing.
- `loomcli`'s `wire` fills the map for `loomshed.NameDiscussionBurler`, `loomshed.NamePlanBurler` and `loomshed.NameWebsterBurler` from `ReviewSettings`.
- The judge model stays the run-wide `judge` key, and no budget or fixer start becomes per segment.
- `refusal-spec.md`'s model-spec-list config row names the six keys; `docs/overview.md`'s loom config line lists them.

Tests:

- `config_test.go`: a malformed entry in each new key is refused at load naming the key, and the template's empty values load clean; extend the existing refusal and load tables.
- `review_test.go`: an unset segment key takes the run-wide list, a set one replaces it, review and fix falling back independently.
- `entries_burler_test.go`: a row whose name has a map entry picks its models per round from it, and a row without one takes `Env.ReviewModels`; extend the existing Env-fallback test.
- `wiring_test.go`: a `loom.yaml` setting `plan_review` reaches only the Plan-Burler row's map entry.

**ImpactSummary:** Adds optional loom.yaml keys and an Env map; with every new key empty, each BurlerRound row runs on exactly the models it runs on today.
