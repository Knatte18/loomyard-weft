# Card 4 — envkit-template-setting

**Create:**
- `internal/testkit/envkit#TemplateSetting`

**Edit:**
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`

**Uses:**
- `internal/configreg#Lookup`
- `internal/shedrecipe#SegmentSettings`

**Intent:** Give the shared test kit a template-backed setting lookup and fill `FullEnv`'s two new `Env` fields, so packages built on the kit keep today's budgets once the entries and `loomrecipe` read the segment settings (cards 6 and 7).

- `TemplateSetting` answers any registered module's template key from that module's embedded template (`configreg.Lookup`, the template decoded with `yaml.v3`), and errors naming the module for an unregistered module and naming the key for a key the template lacks, so a new recipe `_setting` key needs no kit change.
- `FullEnv` sets `Setting: TemplateSetting` and fills `Segments` with an entry for every review segment of the embedded recipes (`Discussion-Review`, `Plan-Review`, `Webster-Review`, `Merge-Review`, `Darn-Review`), each with empty models, `MaxBounces` equal to the kit's current run-wide budget (3, and 1 for `Darn-Review`, today's `DarnReviewMaxBounces`), `MaxBouncesKey` `review_max_bounces` and `ConfigFile` `loom.yaml` or `darn.yaml`, so packages built on the kit keep today's budgets.
- `TestFullEnv_FillsEverySeam` gains subtests: `TemplateSetting("loom", "plan_auto_accept")` answers `false`, an unregistered module and an unknown key each error naming the module or key, and `FullEnv(t).Segments` holds an entry for every review segment of the embedded loom and darn recipes (rows parsed with `shedbuild.Parse` over `recipes.LoomRecipe` and `recipes.DarnRecipe`, a segment counting as a review segment when it holds a `Bouncer` row).

`nilLegal` keeps the two entries card 3 gave it: `FullEnv` now fills both fields, and their nil stays legal for callers such as batten's wiring.

**ImpactSummary:** One new exported kit function and two more FullEnv fills; every package built on FullEnv sees the new fields, which no entry reads yet.

**Commit:** 4: envkit: add TemplateSetting and fill FullEnv's Segments and Setting
