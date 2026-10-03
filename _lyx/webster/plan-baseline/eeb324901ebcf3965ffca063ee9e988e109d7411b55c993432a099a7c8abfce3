# Card 3 — judge-model-key

**Edit:**
- `internal/loomengine/template.yaml`
- `internal/loomengine/config.go`
- `internal/loomengine/config_test.go`
- `internal/loomengine/review.go`
- `internal/loomengine/review_test.go`
- `internal/shedrecipe/recipe.go`
- `internal/shedrecipe/entries_bouncer.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/loomcli/wiring.go`
- `docs/overview.md`

**Intent:** Give the Bouncer its own `judge` model key (D8), so the narrowed sit-rep judge runs on a fast model instead of sharing the reviewers' `review` key.

`template.yaml` gains `judge: sonnet[medium]` beside `review`, with a comment naming it the model-spec for every Bouncer row's seed and judge passes.
`Config` gains `Judge string` (yaml `judge`), and `LoadConfig` validates it through `modelspec.Parse` as a required role key, exactly like `review`; update the file header's list of validated keys.
`review.go` gains `JudgeSettings` (Model, Effort, Version; no timeout, since `BouncerConfig` carries none) and `ResolveJudge(cfg Config, reg modelspec.Registry) (JudgeSettings, error)`, mirroring `ResolveReview`.

`shedrecipe.Env` gains `JudgeModel`, `JudgeEffort` and `JudgeVersion`, documented as the run-wide default every Bouncer row falls back to.
`bouncerEntry` falls back to them instead of the `Review*` fields;
a row's own `model`/`effort`/`version` keys still override.
`BurlerRound` rows keep reading `Review*`.
`loomcli/wiring.go` resolves `ResolveJudge` next to `ResolveReview` and fills the three `Judge*` fields.

Tests:
`config_test.go` — the well-formed fixture carries `judge` and loads it; a malformed `judge` value fails `LoadConfig` naming the key.
`review_test.go` — `ResolveJudge` resolves `sonnet[medium]` to the sonnet model with effort `medium`, and a malformed spec errors.
`entries_bouncer_test.go` — `TestBouncerEntry_EnvReviewFallback` becomes a judge fallback test: a Bouncer row with no model keys takes `Env.Judge*` while `Env.Review*` holds different values, and a row-level `model` still wins.

`docs/overview.md`'s loom config line lists the `judge` role key.

**ImpactSummary:** Every Bouncer row's seed and judge model switches from `review` to `judge`; an existing hub's `loom.yaml` without the key takes the template default in memory until `lyx config reconcile --apply` writes it.

**Commit:** 3: loom: Bouncer rows run on their own judge model key
