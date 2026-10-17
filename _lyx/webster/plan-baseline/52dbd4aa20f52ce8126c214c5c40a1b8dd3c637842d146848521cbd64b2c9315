# Card 6 — review-rows-read-segments

**Create:**
- `plan:internal/shedrecipe#rowSegment` -> `func rowSegment(entry string, row RowIdentity, env Env) (SegmentSettings, error)`
- `plan:internal/shedrecipe#settingRef` -> `func settingRef(entry string, row RowIdentity, cfg Config, key string, env Env) (ref settingReference, named bool, err error)`
- `plan:internal/shedrecipe#settingReference` -> `type settingReference struct`
- `plan:internal/shedrecipe#fanSetting` -> `func fanSetting(entry string, row RowIdentity, cfg Config, key string, env Env) (fans []string, ref settingReference, named bool, err error)`
- `plan:internal/shedrecipe#boolSetting` -> `func boolSetting(entry string, row RowIdentity, cfg Config, key string, env Env) (bool, error)`
- `plan:internal/shedrecipe#fakeSetting` -> `func fakeSetting(values map[string]any) func(module, key string) (any, error)`
- `plan:internal/loomrecipe#fanSettingMismatches` -> `func fanSettingMismatches(r shedbuild.Recipe) []string`
- `plan:internal/loomrecipe#TestShippedRecipes_FanSettingsAgreeWithinSegment` -> `func TestShippedRecipes_FanSettingsAgreeWithinSegment(t *testing.T)`

**Edit:**
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#newTestEnv`
- `internal/shedrecipe#TestBouncerEntry_ConstructionFailures`
- `internal/shedrecipe#TestBouncerEntry_EnvJudgeFallback`
- `internal/shedrecipe#TestBouncerEntry_ClusterExcludes`
- `internal/shedrecipe#TestBouncerEntry_BlockingEscalatesAndColorSegment`
- `internal/shedrecipe#TestBurlerHalfEntries_EnvFallback`
- `internal/shedrecipe#TestBurlerReviewEntry_ClusterFan`
- `internal/shedrecipe#TestBurlerFixEntry_GateConfig`
- `internal/shedrecipe#TestBurlerHalfEntries_ConstructionFailures`
- `internal/shedbuild#TestBuild_EveryRegisteredEngineBuilds`
- `internal/shedadapters#BurlerFixProducer.Call`
- `internal/shedadapters#TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview`
- `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents`
- `internal/loomrecipe#newEscalationShed`
- `internal/loomrecipe#TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/shedrecipe/doc.go`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/shedrecipe#burlerRowModels`

**Uses:**
- `internal/testkit/envkit#TemplateSetting`
- `internal/shedrecipe#reviewRow`
- `internal/shedrecipe#testReviewSegment`
- `internal/shedrecipe#SegmentSettings`
- `internal/shedrecipe#captureReview`
- `internal/shedrecipe#captureFix`
- `internal/shedrecipe#minimalBouncerConfig`
- `internal/shedrecipe#minimalReviewConfig`
- `internal/shedrecipe#minimalFixConfig`
- `internal/shedrecipe#assertErrContains`

**Intent:** The Bouncer, BurlerReview and BurlerFix entries read their segment's `SegmentSettings` and the settings their row names, and the embedded recipes name those settings (D3, D5, D6, D7).
After this card no entry reads `ReviewModels`, `RowReviewModels`, `RowClusterFans`, `RowFixFans`, `RowModelKeys`, `RowAutoAccept`, `RowHardMaxBounces` or `RowBouncerModels`; card 8 deletes them.

Segment (D3): `rowSegment` refuses a row whose `row.Segment` is empty, and a segment with no `env.Segments` entry, each naming the entry, the row and the segment, e.g. `shedrecipe: Bouncer: row "X" has no segment; way forward: put the row in a review segment with segment: in the recipe`, and `shedrecipe: Bouncer: row "X": segment "S" has no Env.Segments entry; way forward: wire segment "S"'s settings in the caller's Env.Segments`.
All three entries call it; the run-wide `ReviewModels` fallback goes (`burlerRowModels` is deleted with its two callers on this card).

Settings (D5, D6): a row config value that references a setting is a string `module.key` under a key ending in `_setting`.
`settingRef` reads `key` with `configString`; absent or empty means not named.
A named value is split at the first dot into `settingReference{Module, Key, Text}`, refusing no dot, an empty module or an empty key; then a nil `env.Setting` is refused; then the lookup is called and its error (unknown module, unknown key, unreadable file) is wrapped.
Every refusal but one names the entry, the row, the config key and the reference: a recipe value that is not a string keeps `configString`'s own text, which names only the key.
Every other refusal's way forward names the recipe key to fix, with two exceptions: a lookup error keeps the lookup's own text, and the nil-`Setting` refusal, a wiring fault no recipe edit cures, ends `way forward: fill Env.Setting in the caller that builds this recipe`.
Card 1's unregistered-module and unknown-key errors carry their own `way forward`.
An error propagated from `configengine.Load` or the snapshot decode stays as it is, because it is unreachable for the shipped recipes: for `loom` and `darn` the wiring's strict load of the same stack has already passed before the recipe builds.
`fanSetting` turns the value into a fan list as `burlerengine.FanList` reads YAML: a `string` is a one-entry list and a `[]any` of strings is the list (a lookup's YAML decode yields `[]any`, never `[]string`, so there is no `[]string` case); a null, an empty string and a list whose every entry is `""` count as absent (a nil list, `named` still true); any other shape (a number, a map, a list holding a non-string) is refused naming the row, key, reference and the shape.
`boolSetting` answers a `bool`, false for a null, and refuses any other shape.
`settingReference` renders the refusal citation `<key> in <module>.yaml`.
Every setting a row names is read at construction, so `loomrecipe.New` and `NewDarn` refuse before any row runs.
Bound: a `_setting` key lets a row read any template key of any registered config module, a guard-category key included, through `Env.Setting`; only rows of the recipes embedded in the binary name such keys, the lookup is read-only and answers only template keys of registered modules (card 1), a guard-category key never takes a local or `run_settings` value because `configengine.Load` refuses it, and `TestShippedRecipes_FanSettingsAgreeWithinSegment` pins each shipped row's exact reference, so a recipe edit that points a row at another key fails that test.

Entries:
- `bouncerEntry` recognises `reviewer_fan_setting` and `auto_accept_setting`; `ClusterFans` is the fan list when `hasFanName` holds (as with today's map), `AutoAccept` the bool, `Models` the segment's `Bouncer`, `HardCap` the segment's `HardMaxBounces`.
- `burlerReviewEntry` recognises `reviewer_fan_setting` and `fixer_fan_setting`; `Models` is the segment's `Models` and `RunOpts.ReviewKey` its `ReviewKey`.
  A row that names either setting records its round's fans whatever the value, exactly where a present map entry did, and a list naming a fan beside the recipe profile's `cluster-fan` or `fix-fan` stays a construction error; the conflict message names the reference instead of "the hub config through the wiring".
- `burlerFixEntry` recognises `fixer_fan_setting` and reads it at construction; when named, `RequiresFans` is true and `RunOpts.FixFanKey` is `<key> in <module>.yaml` (so loom's citation gains the file name it lacks today), and when not named the row reads no recorded fans and runs the single fixer; `Models` is the segment's `Models` and `FixKey` its `FixKey`.
  The `<key> in <module>.yaml` form also reaches `burlerengine`'s twill engine refusal through `RunOpts.FixFanKey`, which the darn wiring already feeds in that form, so that refusal's text needs no change.
- `BurlerFixProducer.Call`'s unrecorded-round refusal (a `RequiresFans` row over a round whose usage record holds no recorded fan names) names the fixer fan's source from `p.opts.FixFanKey`, e.g. `round 2's usage record holds no recorded fan names (<err>); the row's fixer fan comes from discussion_fixer_fan in loom.yaml; way forward: …`, keeping its way forward to the review row as the message's end.
  This is the refusal the fix row raises on an unrecorded fan round, so it cites `<key> in <module>.yaml` for loom and darn alike.
Each entry's doc comment is updated for these sources, and each `configRejectUnknown` list gains its new keys.

Recipes:
- each loom review segment's Bouncer and BurlerReview rows name `reviewer_fan_setting: loom.<perch>_reviewer_fan`, its BurlerReview and BurlerFix rows name `fixer_fan_setting: loom.<perch>_fixer_fan`, and its Bouncer names `auto_accept_setting: loom.<perch>_auto_accept`, `<perch>` being `discussion`, `plan`, `webster` or `merge`;
- Darn-Review's rows name `darn.darn_reviewer_fan` and `darn.darn_fixer_fan` the same way and no auto-accept setting, so Darn-Bouncer's auto-accept stays false;
- the "No cluster-fan key" comments name the `_setting` keys instead.
The row set each segment reads is exactly today's wiring, so a loom and a darn run resolve the same fans and auto-accept flags per row.

D7 test: `fanSettingMismatches` returns one message per violation over a parsed recipe, grouping rows by segment among segments holding a `Bouncer` row: the rows naming `reviewer_fan_setting` name one reference, likewise `fixer_fan_setting`; the BurlerReview row names `reviewer_fan_setting` exactly when the Bouncer does; the BurlerFix row names `fixer_fan_setting` exactly when the BurlerReview row names it or sets a profile `fix-fan`.
`TestShippedRecipes_FanSettingsAgreeWithinSegment` asserts no mismatch over `recipes.LoomRecipe` and `recipes.DarnRecipe`, and pins each shipped segment's references to today's wiring: each loom segment's rows name exactly `loom.<perch>_reviewer_fan`, `loom.<perch>_fixer_fan` and `loom.<perch>_auto_accept` for its own perch (`Discussion-Review` → `discussion` and so on), Darn-Review's name exactly `darn.darn_reviewer_fan` and `darn.darn_fixer_fan`, and Darn-Bouncer names no `auto_accept_setting`.
It also runs hand-authored three-row recipes as rows: `reviewer_fan_setting` references differing between Bouncer and review row (1 message), `fixer_fan_setting` references differing between review and fix row (1), review row naming `reviewer_fan_setting` without the Bouncer (1), Bouncer naming it without the review row (1), fix row missing `fixer_fan_setting` beside a review row naming it (1), fix row missing it beside a review profile `fix-fan` (1), fix row naming it with neither (1), and an agreeing segment (0).
It carries one `//testtiming:keep` reason naming both of its purposes: the pin over the shipped recipes and the self-check of the hand-authored cases.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/loomrecipe` row goes from 56 to 57.

Shared fixture: `newTestEnv` sets `Segments: map[string]SegmentSettings{testReviewSegment: {}}`; `fakeSetting(values)` returns a lookup answering `values["module.key"]` and erroring, naming the reference, for a reference it lacks.

Tests (each building row asserting the exact value; a settings-refusal row asserting the error's row, key, reference and way forward; a segment-refusal row the error's entry, row, segment and way forward; the lookup-error row the row, key and reference, `fakeSetting`'s text carrying no way forward; and the `configString` row as named below):
- `TestBouncerEntry_ConstructionFailures`: the `NoRowBouncerModelsEntry` row becomes a row with an empty segment and a row whose segment has no entry; new rows reach each settings refusal with every upstream check passing: `reviewer_fan_setting` of `"loomfan"` (no dot), `".x"` and `"loom."`; a recipe value of `3` for `reviewer_fan_setting`, refused by `configString` as not a string, whose row asserts only the key name `reviewer_fan_setting` and `must be a string`, as `configString` words it, since a direct entry call carries no row name and the error holds no reference or way forward; a named setting with `env.Setting` nil, whose row asserts the way forward `fill Env.Setting in the caller that builds this recipe`; a lookup error from `fakeSetting` (unknown reference); a fan value of `3` and of `[]any{"a", 1}`; an `auto_accept_setting` value of `"yes"`.
One row sets `reviewer_fan_setting: ""` with `env.Setting` nil and builds: an empty reference counts as not named, so no lookup is needed.
- `TestBouncerEntry_EnvJudgeFallback`: the seed and judge triples come from `env.Segments[testReviewSegment].Bouncer`.
- `TestBouncerEntry_ClusterExcludes`: the fan rows set `reviewer_fan_setting: loom.fan` with `fakeSetting` answering the old map's lists, plus a null-valued row behaving like today's all-empty list.
- `TestBouncerEntry_BlockingEscalatesAndColorSegment`: auto-accept comes from `auto_accept_setting` through `fakeSetting` (true), the hard cap from the segment's `HardMaxBounces`; a new row with a null auto-accept value and no `blocking_escalates` runs the soft-cap rule (Stuck, where the true row with the same keys is Done), proving null is false.
- `TestBurlerHalfEntries_EnvFallback`: models come from `env.Segments[testReviewSegment].Models`; the `RowMapEntryReplacesEnvModelsForItsOwnRowOnly` row becomes one asserting a row of another segment (`reviewRow` with a second `Segments` entry) takes that segment's models; the timeout rows stay.
- `TestBurlerReviewEntry_ClusterFan`: the wiring-list rows name the two settings through `fakeSetting`; a row whose lookup answers the scalar string `"env-fan"` runs that fan as a one-entry list; rows for a null, an empty string and an all-empty list each record a solo round (`wantRecorded`, empty fans); the profile-conflict rows name the reference; the `RowModelKeys` subtest reads `ReviewKey` from the segment.
- `TestBurlerFixEntry_GateConfig`: the keys subtest asserts `FixKey` from the segment and `FixFanKey == "discussion_fixer_fan in loom.yaml"` when the row names `fixer_fan_setting: loom.discussion_fixer_fan`, and a row naming no setting has an empty `FixFanKey`.
  A new subtest lays out round 1's review and acceptance record inline, with no usage record, and calls the built fix producer: a row naming `fixer_fan_setting` whose value is null is refused as unrecorded with an error containing `discussion_fixer_fan in loom.yaml`, so a named setting requires the recorded round whatever its value, while a row naming no setting runs the single fixer (one Fix call, `profile.FixFan == ""`).
- `TestBurlerHalfEntries_ConstructionFailures`: for each half, one segment refusal (the review half an empty segment, the fix half a segment with no entry) and an unknown reference of its fan setting, so the fix row's `fixer_fan_setting` is read and refused at build; the malformed-reference and nil-`Setting` refusals live in the shared `settingRef`, which the Bouncer rows reach.
- `TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview`: its unrecorded-round rows set `spec.opts.FixFanKey` to `discussion_fixer_fan in loom.yaml` and assert the message contains `the row's fixer fan comes from discussion_fixer_fan in loom.yaml` and still ends with the way forward to the review row.
- `TestNoGatedRowAuthorizesForkSubagents`: the shipped-recipe rows derive which rows take a fan from the rows that name `reviewer_fan_setting` or `fixer_fan_setting`, keeping the per-perch fan names the test assigns, so a gated row that names a fan setting cannot escape the check by being missing from a hand-kept row list.
- `TestBuild_EveryRegisteredEngineBuilds`: the Bouncer, BurlerReview and BurlerFix rows carry `Segment: "Plan-Review"`, which `envkit.FullEnv` wires, in place of the `RowBouncerModels` override.
- `newEscalationShed`, `TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe` and `TestEscalation_PerchBudgetOneWithAutoAccept` set the hard cap on `env.Segments` entries and the Webster auto-accept by wrapping `env.Setting` around `envkit.TemplateSetting` to answer `loom.webster_auto_accept` true.

`internal/shedrecipe/doc.go` gains a `# Segment settings and row settings` section stating D1–D6 as built: `Env.Segments` keyed by segment, the `RowIdentity` segment, the `module.key` reference form under `_setting` keys, the value shapes, the build-time refusals and the lookup bound.
`PATTERN.md` gains, in the `Shed and loom` section after `PATTERN-shed-recipe-registry`, the declared entry line verbatim:

```markdown
- `PATTERN-recipe-setting-lookup` — Giving a shed recipe row a per-row or per-segment config value: a plain value is named `module.key` and read through `Env.Setting`, a resolved one of a review segment's row is a `SegmentSettings` field; never its own `Env` field.
```

`PATTERN.md` ends at no more than 15564 bytes (`pattern.HeadroomBytes`), so the card also shortens existing entry lines by at least 264 bytes in total, keeping the declared entry line verbatim and dropping only words the entries' linked background files already say.
`lyx loom lint-pattern` confirms the result.

**ImpactSummary:** Three review entries switch sources and gain build-time refusals, both embedded recipes gain setting keys, and shedadapters' unrecorded-round refusal gains the setting's citation; shedrecipe, shedadapters, shedbuild and loomrecipe tests follow, and the old per-row Env maps lose their last reader.

**Commit:** 6: shedrecipe: review rows read their segment's settings and their named settings
