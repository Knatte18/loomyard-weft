# Card 6 — landing-batch-max-key

**Create:**
- `plan:internal/landingshed#batchMaxKey` -> `const batchMaxKey = "batch_max"`
- `plan:internal/landingshed#Config.BatchSize` -> `func (c Config) BatchSize() int`
- `plan:internal/landingshed#TestConfig_BatchSize` -> `func TestConfig_BatchSize(t *testing.T)`

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed#ConfigSpec`
- `internal/landingshed#LoadConfig`
- `internal/landingshed/template.yaml`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/landingshed#TestLoadConfig_InvalidKeys`
- `internal/landingshed#TestFinalize_OwnerChecks`
- `internal/loomcli#parentFixture.finalize`
- `contracts/recipes/hardener-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/tapestry-recipe.yaml`
- `contracts/recipes/sampler-recipe.yaml`
- `docs/overview.md`

**Delete:**
- `internal/landingshed#finalizeLostRacesKey`

**Uses:**
- `internal/landingshed#Config.IdleRelease`
- `internal/loomcli/doc.go`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Add `landing.batch_max` and remove `landing.finalize_lost_races` (D9).

- `Config` gains `BatchMax int `yaml:"batch_max"``, the most entries one batch takes, and loses `FinalizeLostRaces`.
- `Config.BatchSize()` returns `BatchMax`, and 1 for a value below 1, an absent key included; it mirrors `Config.IdleRelease`.
  `LoadConfig` refuses no value of the key, so its doc drops "and a lost-race bound below 0" and the `FinalizeLostRaces < 0` refusal goes.
- `template.yaml` replaces the `finalize_lost_races` line with `batch_max: 4  # most runs one landing batch on a parent takes, gated with one gate run; 1 lands every run alone under the parent lock; a value below 1 reads as 1`.
- `ConfigSpec`'s categories drop `finalizeLostRacesKey` and its comment, and add `batchMaxKey: configengine.CategoryCrossPair`, since the batch size bounds how long one leader holds the parent lock others wait on.
  `RetiredKeys` gains `"finalize_lost_races": configengine.CategoryCrossPair`, so the local and machine layers refuse it and a default or worktree layer still setting it loads with configengine's unknown-key warning; the key's meaning does not change, so no reinterpretation ships (`PATTERN-config-layers`).
- `finalizeLostRacesKey` is deleted from `keynames.go`, its only readers being `ConfigSpec` and `LoadConfig`, and `batchMaxKey` is added there.
- The recipe comments naming `finalize_lost_races` (`hardener-recipe.yaml`, `darn-recipe.yaml`, `tapestry-recipe.yaml`, `sampler-recipe.yaml`) and `docs/overview.md`'s two sentences on the lost-race bound are reworded to the queue: a run at Finalize merges the parent with no lock, then waits in the parent lock's queue, and the lock's holder gates and lands up to `landing.batch_max` queued runs together.
  The search that found them is `grep -rn "finalize_lost_races\|FinalizeLostRaces\|lost race" --include=*.go --include=*.yaml --include=*.md` outside `.lym` and `_lym`.
- Callers of the removed field: `TestFinalize_OwnerChecks`'s transport rows drop their `lostRaces` column and its assignment; `parentFixture.finalize` in `internal/loomcli` drops `FinalizeLostRaces: 1`.
  Card 5 already removed the finalize.go readers.

Tests, tier0 in `package landingshed`, adding about 0.2 seconds to one tier0 run:
`TestLoadConfig_TemplateDefaults` asserts `cfg.BatchMax == 4`, adds `finalize_lost_races` to its retired-key list and to the seeded default-layer file, and asserts that file still loads with the template's values.
It gains a row that seeds `finalize_lost_races: 2` in the local layer, `.lym/config/landing.local.yaml`, and asserts `LoadConfig` refuses it naming the key, which only the `RetiredKeys` entry's `CategoryCrossPair` makes the local layer do.
The default layer's unknown-key warning itself is configengine's, pinned by `internal/configengine`'s layer tests, so this row pins only the entry that routes the key there.
`TestLoadConfig_InvalidKeys` drops its "finalize lost races below 0" row.
`TestConfig_BatchSize` rows: 4 reads 4, 1 reads 1, 0 reads 1, -3 reads 1.

**ImpactSummary:** Removes one Config field read by two tests and one loomcli fixture, adds one key; recipe and overview comments change wording only.
