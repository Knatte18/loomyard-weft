# shedrecipe capabilities: segment settings and a told setting lookup

All six batches recorded `done`, with no `departures` reported.

- **01 configreg-setting-lookup** (head `55330c0aa`): added `configreg.SettingLookup`. Scope guard deviation: `internal/configreg/settinglookup_test.go`.
- **02 constructor-row-identity** (head `8181cba7f`): `Constructor` takes a `RowIdentity` with the row's name and segment. No deviations.
- **03 shedrecipe-segment-settings** (cards 3, 4, 5; head `3686a6095`): added `SegmentSettings`, `Env.Segments` and `Env.Setting`, `envkit.TemplateSetting`, and the loom and darn wiring fills. Scope guard deviation: `internal/testkit/envkit/envkit.go`.
- **06 review-rows-read-segments** (head `06e175002`): the review entries read their segment and named settings, with build-time refusals, and both recipes name the settings. Scope guard deviations: `internal/loomrecipe/fansettings_test.go`, `internal/loomrecipe/gatequiescence_test.go`, `internal/shedrecipe/entries_settings.go`.
- **07 budgets-by-segment** (head `ff7b74fb4`): `New`, `NewDarn`, `Routing` and `DarnRouting` take budgets keyed by segment. Scope guard deviations: `internal/loomrecipe/darnrecipe.go`, `darnrecipe_test.go`, `loomrecipe.go`, `reviewbudget_test.go`.
- **08 retire-row-maps** (head `4acb9b30a`): removed the unread per-row `Env` maps, `RowModelKeys` and their wiring helpers. No deviations.

Deviations are informational only; no batch was stuck or recovered.
The plan-level verify gate runs after this turn.
