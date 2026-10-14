# Card 4 — loom config on the shared budget family, zero rounds settable per run

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#MigrateConfig`
- `internal/loomengine#ValidateLocalValues`
- `internal/loomengine#Config.Caps` -> `func (cfg Config) Caps(p Perch) reviewbudget.Caps`
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomcli#perchMaxBounces`
- `internal/loomcli#perchHardCap`
- `internal/loomcli#perchHardMaxBounces`
- `internal/loomcli#loomCLI.reviewBudget`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#TestWire_MergeReviewSeams`
- `internal/configreg#Module`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/loomengine#PerchCaps`
- `internal/loomengine#CapRoute`

**Create:**
- `plan:internal/loomengine#reviewPerches` -> `var reviewPerches = []Perch{PerchDiscussion, PerchPlan, PerchWebster, PerchMerge}`
- `plan:internal/loomengine#TestMigrateConfig` -> `func TestMigrateConfig(t *testing.T)`

**Uses:**
- `plan:internal/reviewbudget#Categories`
- `plan:internal/reviewbudget#Resolve`
- `plan:internal/reviewbudget#Route`
- `plan:internal/reviewbudget#CheckResolved`
- `plan:internal/reviewbudget#CheckLocal`
- `plan:internal/reviewbudget#Values`

**Intent:**
The loom half of D1 and D2 as the record's added entries settle them: loom.yaml's budget keys come from `internal/reviewbudget`, every per-perch budget takes 0 as zero rounds and null as "take `review_max_bounces`", the run-wide `review_max_bounces` accepts 0, `skip_merge_review` is retired for `merge_max_bounces: 0`, and a `run_settings` budget below its bound is refused at `lyx board upsert`.
Card 2 already gives the recipe its zero-round meaning; this card stops translating loom.yaml into it.
An operator `lyx config loom --set` of a null budget to an integer, or of a budget back to null, reads back as written through card 3's yamlengine rule, which this card's three null template leaves rely on.
What it lets through: a task can switch off any loom review, Discussion-Review and Plan-Review included, through its own `run_settings` or local layer, and no hub guard bounds that, as the settled design's point (8) and "Perch budgets carry no guard" accept; the zero-round Bouncer still runs its approve and commit seams, and every perch with a budget above 0 stays bound by its hard cap and the Bouncer's soft-cap rule.

`internal/loomengine`:

- `Config`: `DiscussionMaxBounces`, `PlanMaxBounces`, `WebsterMaxBounces` and `MergeMaxBounces` become `*int`, nil meaning take `ReviewMaxBounces`; `SkipMergeReview` is removed; the field comments say so.
- `template.yaml`: `discussion_max_bounces`, `plan_max_bounces` and `webster_max_bounces` become `~` with the comment "null takes review_max_bounces; 0 runs no review round: the perch's Bouncer settles approved at once"; `merge_max_bounces: 1` keeps its value and gains the same 0 sentence; `review_max_bounces`'s comment says 0 gives every perch without its own budget zero rounds; `skip_merge_review` is removed.
- `ConfigSpec` takes every budget, hard-cap and auto-accept key from `reviewbudget.Categories` over `reviewPerches`' names and drops `skip_merge_review`; every other entry stays, so the auto-accept keys stay guard and the budget keys stay none.
- `Config.Caps` returns `reviewbudget.Resolve(string(p), cfg.ReviewMaxBounces, cfg.ReviewHardMaxBounces, own soft, own hard)`; `PerchCaps` and `CapRoute` are deleted, their one outside caller being `perchHardCap` below, and `caps.go`'s file comment says the file names the loom perches and resolves their caps through `reviewbudget`.
- `LoadConfig` keeps its own `review_circling_checkpoint` below-1 refusal and replaces the `review_max_bounces` below-1 check, the per-perch negative checks, the run-wide hard-cap check, the per-perch hard-cap checks and the hard-below-soft loop with one `reviewbudget.CheckResolved(dirs, spec, values)` over the spec it loaded with and the resolved values of every perch in `reviewPerches`.
  The hard-cap and hard-below-soft messages keep today's text, since `reviewbudget` carries it; the soft-cap messages change to the "0 for no review round" wording card 1 fixes.
  The file comment of `config.go` and the doc comment of `LoadConfig` say the same: `review_max_bounces` and the per-perch budgets refused below 0, null taking the run-wide budget, 0 giving zero rounds, and `skip_merge_review` gone.
- `ValidateLocalValues` decodes the document a second time into a `map[string]any` and calls `reviewbudget.CheckLocal` with `reviewPerches`' names after its own driver and tackler checks, so a `run_settings` budget below its bound is refused at upsert, D2's one change to loom's behaviour; its doc comment names the budget keys.
- `MigrateConfig` keeps its floor raise and gains a one-time rewrite keyed on the old schema, which a full loom.yaml written by a binary before this change always carries: a document whose top level holds `skip_merge_review`.
  For such a document only, it rewrites each of `discussion_max_bounces`, `plan_max_bounces`, `webster_max_bounces` and `merge_max_bounces` whose value is the integer 0 to null, with the line `<key>: 0 rewritten to null, which takes review_max_bounces`;
  then, when `skip_merge_review` is true, it sets `merge_max_bounces` to 0, with the line `skip_merge_review: true rewritten to merge_max_bounces: 0`;
  then it removes `skip_merge_review`, with the line `skip_merge_review: removed`.
  So the hub's default file and a full worktree file keep their meaning after the binary change through `hubreconcile.Ensure`'s reconcile, which runs the hook, the route `PATTERN-binary-change-reconcile` allows; a second run finds no `skip_merge_review` and changes nothing more.
  A sparse layer file or a `run_settings` document carries no `skip_merge_review`, since the local layer refused that guard key and a strip drops keys equal to the layers below, so its 0 is left as the new zero-rounds meaning the settled design gives `run_settings`.
  This departs from the settled design's point (4), which rewrites a per-perch 0 to null in worktree files too: a sparse worktree-layer file holding `<perch>_max_bounces: 0`, written under the old meaning to inherit over a hub value above 0, keeps its 0 and gives that perch zero rounds after the binary change.
  The hook sees only a document's bytes, never its layer, and such a file's bytes match a `run_settings` or local document whose 0 must keep its new meaning, so the hook cannot tell them apart; passing the layer would change `configreg.Module.Migrate`'s signature for every module to serve one transition.
  The exposure is bounded to task worktrees alive at the binary change whose worktree file is sparse and holds a per-perch 0; a full worktree file holds `skip_merge_review` and is rewritten.
  It edits `yaml.Node`s as `batcher.MigrateConfig` does, keeps every comment and other key, and returns an unparseable document as an error marked `configengine.ErrInvalid`.
- `reviewPerches` lists the four perches in one place for `ConfigSpec`, `LoadConfig` and `ValidateLocalValues`.

`internal/loomcli`:

- `perchMaxBounces` puts a perch's entry in the map exactly when its `*int` is non-nil, 0 included, and no longer reads `SkipMergeReview`.
- `perchHardCap` names its loom route through `reviewbudget.Route(dirs, loomengine.ConfigSpec(), caps.HardKey)`, and its darn branch through `reviewbudget.Route(dirs, darnengine.ConfigSpec(), "review_hard_max_bounces")` in place of the fixed prime command, which gives the same text for the hub file; card 5 opens darn's local layer to the key, from which the local route follows.
- `perchHardMaxBounces` reads `cfg.Caps(perch).Hard` from the new `reviewbudget.Caps`, its comment unchanged in meaning.
- `TestWire_ReviewKeysReachTheEnv` seeds `merge_max_bounces: "0"` in place of `skip_merge_review: "true"` and still expects no Discussion-Bouncer entry, from the template's null, and `Merge-Bouncer: 0`.
- `TestWire_MergeReviewSeams` seeds `merge_max_bounces: "0"` in place of `skip_merge_review: "true"` and keeps its `RowMaxBounces` assertion.
- `loomCLI.reviewBudget` takes `c.cfg` as loaded when `c.cfg.ReviewHardMaxBounces > 0`, which `LoadConfig` guarantees is at least 1, in place of `c.cfg.ReviewMaxBounces > 0`, since a loaded run-wide `review_max_bounces` may now be 0; its doc comment no longer names `skip_merge_review`.

`internal/configreg`:

- `Module`'s `Migrate` doc comment widens to admit rewriting a key's retired meaning into its replacement, as loom's 0-to-null and `skip_merge_review` rewrites do, beside retiring a key and raising one to its floor.
- `TestModules_EveryTemplateKeyClassified`'s loom `LocalDenied` list drops `skip_merge_review`.
- `TestPrepareRunSettings` gains one acceptance and one refusal for loom over the real registry, the wiring of the hook; the bound matrix is `TestCheckLocal`'s: `{"loom": {"plan_max_bounces": 0}}` comes back unchanged as its document, so a `run_settings` 0 keeps its zero-rounds meaning through the migrate hook, and `{"loom": {"review_hard_max_bounces": 0}}` is refused as a `*RunSettingsError` for module loom, matching `configengine.ErrInvalid` and naming the key and its bound.

`contracts/specs/refusal-spec.md`: the "config: review key below 1" row becomes "review key out of range" for `review_circling_checkpoint` below 1 and `review_max_bounces` or a per-perch budget below 0, its way forward naming 0 for no review round and null for the run-wide budget; the "config: hard cap out of range" row's way forward names the layer routes as today; the "Seed-Child: run settings invalid" row adds a review budget key below its bound, refused at `lyx board upsert` too.
`docs/overview.md`'s loom config paragraph says the per-perch budgets, `merge_max_bounces` added to its list, take null for `review_max_bounces` and 0 for no review round, in place of "where 0 takes `review_max_bounces`".

Tests:
`templateConfig` holds nil for the three null budgets, a pointer to 1 for `MergeMaxBounces`, and no `SkipMergeReview`.
`TestLoadConfig_Loads`: the per-perch row takes pointers, the merge row seeds `merge_max_bounces: "0"` without `skip_merge_review` and expects a pointer to 0, and two rows are added, `plan_max_bounces: 0` loading as a pointer to 0 and `review_max_bounces: 0` loading with every null perch resolving to 0 through `Caps`.
`TestLoadConfig_Refuses`: the `review_max_bounces` 0 row becomes -1 with the new wording, a per-perch -1 row asserts the null-or-0 wording, the hard-cap rows keep their text.
`TestMigrateConfig` is a table over `MigrateConfig`: a hand-written old-schema fragment holding `skip_merge_review: false`, `plan_max_bounces: 0` and `webster_max_bounces: 2` comes back with `plan_max_bounces` null, `webster_max_bounces: 2`, no `skip_merge_review` and exactly the lines `plan_max_bounces: 0 rewritten to null, which takes review_max_bounces` and `skip_merge_review: removed`; one with `skip_merge_review: true` and `merge_max_bounces: 1` comes back with `merge_max_bounces: 0`; a document without `skip_merge_review` holding `plan_max_bounces: 0` comes back unchanged with no line; running the hook over its own output changes nothing; a below-floor `step_idle_timeout_min` is still raised; an unparseable document is `configengine.ErrInvalid`.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/loomengine` from 57 to 58 for `TestMigrateConfig`.

**ImpactSummary:** loom.yaml's budget keys, template, categories, load checks, migration and local-value check move onto reviewbudget with zero-round and null semantics, and skip_merge_review is retired; loomcli's per-perch map, hard-cap route and loaded-config test, configreg's Migrate contract and its pinned key lists, and the refusal spec and overview follow.

**Commit:** 4: loom config: zero-round perches per run, null inherits, skip_merge_review retired
