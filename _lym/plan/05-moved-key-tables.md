# Card 5 — moved-key-tables

**Create:**
- `plan:internal/loomengine#loomMovedKeys` -> `func loomMovedKeys() map[string]string`
- `plan:internal/loomengine#tapestryMovedKeys` -> `func tapestryMovedKeys() map[string]string`
- `plan:internal/configreg#refuseMovedKeys` -> `func refuseMovedKeys(module Module, keys map[string]any) error`

**Edit:**
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#TapestryConfigSpec`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestLoadTapestryConfig`
- `internal/configreg#PrepareRunSettings`
- `internal/configreg#CheckSettingKey`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configreg#TestProfilesCheckLoad`
- `internal/configreg#TestSettingLookup`
- `internal/configcli#TestDispatchSet`
- `internal/boardcli_test#stepRunSettings`
- `internal/battenshed#TestSeedChild_RunSettings`
- `internal/shedbuild#TestBuild_Errors`

**Extends:**
- `internal/configreg/doc.go`

**Intent:** Fill loom's and tapestry's move tables from D1 and refuse an old name at the sites configengine does not own (D4).

`loomMovedKeys()` maps every old loom key of D1's table to its new path: `discussion` → `discussion.model`, each `discussion_<rest>` → `discussion.<rest>`, each `code_<rest>` → `code.<rest>`, each `split_<rest>` → `split.<rest>`, `raddle` → `raddle.model`, `raddle_auto_approve` → `raddle.auto_approve`, `friction` → `friction.model`, `friction_timeout_min` → `friction.timeout_min`, each `pr_review_<rest>` → `pr.review_<rest>`, each `review_<rest>` → `review.<rest>`, and `reviewer_model`, `fixer_model`, `judge`, `seed`, `assist_reviewer_timeout_min` → `review.<same>`.
`tapestryMovedKeys()` maps tapestry's: `plan` → `plan.model`, each `plan_<rest>` → `plan.<rest>`, `parts` → `parts.model`, each `parts_<rest>` → `parts.<rest>`, `part_writer` → `parts.writer`.
Each is written as an explicit literal map, one entry per old key, its new paths built from card 4's path constants where one exists, so a later template key never joins a table by a prefix rule.
`ConfigSpec()` and `TapestryConfigSpec()` set `MovedKeys` from them, so configengine's checks of card 2 reach every local, worktree and hub base file and every `lym config loom --set` and `lym config tapestry --set`.
A loom old key that is also in `tapestryPlanKeys`, such as `plan_max_bounces` in a loom file, stays a retired loom key: loom's table holds no `plan*` key, and the loom spec still refuses it naming `tapestry.plan.max_bounces` (card 4).

`PrepareRunSettings` calls `refuseMovedKeys` beside `refuseRetiredKeys`: the first top-level key of a module's object, in name order, that the module's `MovedKeys` names is refused, marked `ErrInvalid`, as `key <module>.<old> was moved to <module>.<new>`, with no way forward clause of its own.
`refuseMovedKeys` applies card 2's matching rule to the decoded object: an old key equal to its new path's first segment, `discussion`, is refused only when its value is not a `map[string]any`, so `{"discussion": {...}}` is the new area and passes on to the local rule.
Its callers print their own way forward once: Seed-Child's wrapper names `lym board upsert` and `lym config profiles`, and `lym board upsert` and the profile check refuse the write, which the operator repeats with the new name.
`CheckSettingKey` refuses a key the module's `MovedKeys` names, before its template check, with `setting key <module>.<old> was moved; way forward: name it <module>.<new>`, so a recipe reference such as an entry gate's `tapestry.plan_split` fails at recipe build naming `tapestry.plan.split`.

Reaching tests, all rows of existing tests:
- `TestLoadConfig_Loads` (tier0): for every `loomMovedKeys` entry, the loom template holds the new path and no top-level old key; `TestLoadTapestryConfig` asserts the same of `tapestryMovedKeys` (D1).
- `TestLoadConfig_Refuses` (tier0), whose table gains a `layer` field choosing the local file, the worktree file or the hub base file the row's document is written to:
  an old key `code_max_bounces: 2` in each of the three is refused naming `loom.code_max_bounces`, `loom.code.max_bounces` and that layer's reconcile command;
  a local file setting `discussion: {reviewer_fan: ""}` is refused as a Guard key by its new path, and one setting `discussion: {timeout_min: 60}` loads;
  a local `loom.yaml` holding `plan_max_bounces` is still refused naming `tapestry.plan.max_bounces`.
- `TestLoadTapestryConfig` (tier0): a local `tapestry.yaml` holding `part_writer` is refused naming `tapestry.parts.writer`.
- `TestPrepareRunSettings` (tier0): `{"loom": {"code_max_bounces": 2}}` is refused naming both full names and holding no `way forward:`; `{"loom": {"discussion": {"timeout_min": 60}}}` is admitted.
- `TestProfilesCheckLoad` (tier0): a profile block `{"loom": {"code_max_bounces": 2}}` is refused naming the profile, the recipe and both full names.
- `stepRunSettings` (`lym board upsert`'s run-settings step, at its file's existing tier): an upsert whose `run_settings` hold `{"loom": {"code_max_bounces": 2}}` is refused naming both full names and writes nothing.
- `TestSeedChild_RunSettings` (tier0): a row whose fake `PrepareRunSettings` returns an `ErrInvalidRunSettings` carrying the moved-key text stands Stuck with a reason holding exactly one `way forward:`.
- `TestBuild_Errors` (tier0): a recipe whose entry gate reads `tapestry.plan_split` fails its build naming `tapestry.plan.split`.
- `TestSettingLookup` (tier0): `CheckSettingKey("loom", "discussion_reviewer_fan")` is refused naming `loom.discussion.reviewer_fan`.
- `TestDispatchSet` (tier0): `lym config loom --set code_max_bounces=2` is refused naming `--set code.max_bounces=2` and writes nothing; `lym config loom --unset code_max_bounces` removes the old key from a file that holds it.

`configreg/doc.go` gains a sentence that `PrepareRunSettings` refuses a moved key by both full names.

**ImpactSummary:** Turns on D4's refusal for every loom and tapestry old name in every config layer, `--set`, run_settings, profiles and recipe references; no other module has a move table, so their loads are unchanged.
