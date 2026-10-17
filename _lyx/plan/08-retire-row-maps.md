# Card 8 — retire-row-maps

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#newTestEnv`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#wireDarnReview`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_MergeReviewSeams`
- `internal/loomrecipe#gatedRowFanViolations`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `docs/research/haiku-judge.md`

**Delete:**
- `internal/shedrecipe#RowModelKeys`
- `internal/shedrecipe#bouncerRowModels`
- `internal/loomcli#reviewRowModelKeys`
- `internal/loomcli#perchMaxBounces`
- `internal/loomcli#perchHardMaxBounces`
- `internal/loomcli#perchAutoAccept`

**Intent:** Remove the per-row maps the earlier cards left unread, so a new plain value needs only a template key, its `ConfigSpec` category, a recipe `_setting` key and the reading entry's code.

`Env` loses `ReviewModels`, `RowReviewModels`, `RowClusterFans`, `RowFixFans`, `RowModelKeys`, `ReviewMaxBounces`, `RowMaxBounces`, `RowAutoAccept`, `RowHardMaxBounces`, `RowBouncerModels` and `DarnReviewMaxBounces`, and their doc comments; `ReviewTimeout` keeps its own comment, reworded to stand alone.
The `RowModelKeys` type goes, and `recipe.go`'s file header stops naming it.

Fills removed:
- `loomCLI.wire` drops every fill of those fields; `reviewRowModelKeys`, `perchMaxBounces`, `perchHardMaxBounces` and `perchAutoAccept` have no reader left and are deleted, their roles held by `loomSegmentPerches`, `loomSegmentSettings` and `loomSegmentBudgets`.
  `loomPerches` stays: `perchHardCap` in `circling.go` keys it by Bouncer row name to find a CIRCLING verb's hard-cap source.
- `wireDarn` drops the `DarnReviewMaxBounces` and `RowHardMaxBounces` writes, and `wireDarnReview` drops its per-row map writes, keeping only the `Darn-Review` `Segments` entry; their doc comments stop naming `env.ReviewModels` and the maps.
- `envkit.FullEnv` drops `RowBouncerModels`, `ReviewMaxBounces` and `DarnReviewMaxBounces`, and `nilLegal` drops the eight `Row*` entries.
- `shedrecipe`'s `newTestEnv` drops `RowBouncerModels`, so `bouncerRowModels` is deleted.
- `docs/research/haiku-judge.md`'s sentence saying loomcli's wire threads each segment's seed and judge pair to its Bouncer row through `shedrecipe.Env.RowBouncerModels` names `shedrecipe.Env.Segments`, the segment's `Bouncer` field, instead.
- `gatedRowFanViolations`'s doc comment describes its `fans` map as a row-name to fan-list map of the values the rows' fan settings would answer, not `Env.RowClusterFans`.

Tests:
- `TestWire_DefaultConfig` drops its `c.env.ReviewModels` assertion in favour of each segment's `Segments[...].Models`, and its "merge review stays on by default" subtest reads `c.env.Segments["Merge-Review"].MaxBounces` (non-zero under the template).
- `TestWire_ReviewKeysReachTheEnv` and `TestWire_DarnRecipe` drop the per-row map assertions card 5 duplicated in `Segments`, and their per-row fan and auto-accept expectations go: card 6's `TestShippedRecipes_FanSettingsAgreeWithinSegment` pins which key each row reads, and card 1's `TestSettingLookup` pins what a key answers.
  In `TestWire_ReviewKeysReachTheEnv` the `fanned` wiring's per-row fan, `RowModelKeys` and `RowReviewModels` expectations go; its `Segments` assertions from card 5 stay, since they alone pin a segment's own model keys.
  Each test keeps one `Setting` read of its own module's seeded fan key (`fanned.env.Setting` for `loom.plan_reviewer_fan`, from card 5, and the "set fans and models" case's `c.env.Setting` for `darn.darn_fixer_fan`, also from card 5), which proves the wiring builds the lookup over the pair's `Dirs`, the board included for darn.
- `TestWire_MergeReviewSeams` reads `c.env.Segments["Merge-Review"].MaxBounces == 0` under `merge_max_bounces: 0`.

**ImpactSummary:** Eleven Env fields and one type removed with four unread loomcli helpers; their fills in loomcli and the kits and four loomcli wiring tests change, and the full suite's other users already moved on cards 3 to 7.

**Commit:** 8: shedrecipe: retire the per-row review maps from Env
