# Card 4 — nest-loom-tapestry

**Create:**
- `internal/configengine#NestPaths`

**Edit:**
- `internal/loomengine#`
- `internal/reviewbudget#`
- `internal/loomcli#`
- `internal/shedrecipe#`
- `internal/shedbuild#`
- `internal/loomrecipe#`
- `internal/runprofile#`
- `internal/configengine#KeyAtLine`
- `internal/configreg#CheckSettingKey`
- `internal/configreg#SettingLookup`
- `internal/configreg#TestSettingLookup`
- `internal/configreg#writeLoomDefault`
- `internal/configreg#writeTapestryDefault`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configreg#TestLayerRunSettings`
- `internal/configreg#TestProfilesCheckLoad`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestModules_LocalLayerRefusals`
- `internal/configreg#TestLookup`
- `internal/beamcli#mergeParent`
- `internal/beamcli#TestMergeParent`
- `internal/beamcli#TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig`
- `internal/hardencli#commitSandboxConfig`
- `internal/configcli#TestDispatchSet`
- `internal/configcli#TestDispatchSetProfiles`
- `internal/configcli#TestDispatchLayers`
- `internal/configcli#TestPrintResolved`
- `internal/configcli#TestPrint`
- `internal/configcli#TestEditOne`
- `internal/configcli#TestConfigOverRealHub`
- `internal/configcli#TestConfigReconcileInGitRepo`
- `internal/configsync#TestReconcileAll`
- `internal/configsync#TestReconcileAll_SparseLayers`
- `internal/configsync#TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate`
- `internal/configsync#TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`
- `internal/hubreconcile_test#TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`
- `internal/hubreconcile_test#TestEnsure_UnpushedHubWideCommitOverMovedUpstreamLeavesStampAbsent`
- `internal/battencli#`
- `internal/boardcli#`
- `internal/webstercli#`
- `internal/testkit/envkit#`
- `contracts/recipes/motif-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/sampler-recipe.yaml`
- `contracts/recipes/tapestry-recipe.yaml`
- `contracts/recipes/arras-recipe.yaml`

**Uses:**
- `internal/configengine/doc.go`
- `internal/beamcli/doc.go`
- `internal/hardencli/doc.go`

**Intent:** Nest `loom.yaml` and `tapestry.yaml` one level per area (D1) and move every reader and writer to the nested paths (D8, D9).
The move is a rename only: every key keeps its value, default, comment meaning and category, and every fallback keeps its meaning.
It is one card because the templates, their decoders, the recipes' setting references and the shipped profiles must change in one commit:
a recipe or profile still naming a flat key fails recipe build or the profiles check against the nested template.

**Templates (D1).** `internal/loomengine/template.yaml` and `internal/loomengine/tapestry.yaml` take D1's table exactly:
`discussion` → `discussion.model`; `discussion_<rest>` → `discussion.<rest>`; `code_<rest>` → `code.<rest>`; `split_<rest>` → `split.<rest>`;
`raddle` → `raddle.model`, `raddle_auto_approve` → `raddle.auto_approve`; `friction` → `friction.model`, `friction_timeout_min` → `friction.timeout_min`;
`pr_review_<rest>` → `pr.review_<rest>`; `review_<rest>` → `review.<rest>`;
`reviewer_model`, `fixer_model`, `judge`, `seed`, `assist_reviewer_timeout_min` → `review.<same>`;
tapestry `plan` → `plan.model`, `plan_<rest>` → `plan.<rest>`, `parts` → `parts.model`, `parts_<rest>` → `parts.<rest>`, `part_writer` → `parts.writer`.
`subagent_model`, `driver`, `step_idle_timeout_min`, `notify_parent_on` and tapestry's `implementer` stay top-level.
Each area is one block mapping; within it the keys keep their present order.
Each comment is rewritten as prose, not by find and replace, so every key it names is the full dotted name (D3), such as "empty takes `loom.review.reviewer_model`" and "`loom.discussion.judge` and `loom.code.judge` override it per segment, and `tapestry.plan.judge` for the plan segment";
the `<segment>_reviewer_model` pattern becomes `loom.<segment>.reviewer_model`.

**Decoders.** `loomengine.Config` decodes each area as a nested struct (one per area: discussion, code, split, raddle, friction, pr, review), and `TapestryConfig` its plan and parts areas the same way.
Every reader of a field moves to the nested field, in loomengine and every package that reads `Config` or `TapestryConfig`;
`Config.WithTapestry` copies the nested plan and parts structs.
`LoadTapestryConfig`'s run-wide read decodes `review.max_bounces`, `review.hard_max_bounces` and `review.amend_max_bounces`.
A reflective test in loomengine that walks `Config` or `TapestryConfig`, such as `TestConfigTemplate_ContainsEveryConfigYAMLTag`, carries one `//lym:seam` line per new nested area struct it walks.
`LoadConfig`'s and `LoadTapestryConfig`'s validation tables name the nested paths; each key path a message or reader cites is one named constant in loomengine (D3), the constants replacing the inline literals, such as `codeMaxBouncesKey = "code.max_bounces"` built from the area and leaf constants.
`configengine.KeyAtLine` returns the dotted path of the deepest mapping key whose entry spans the line, not only the top-level key, so a model-spec or fan shape error names `code.reviewer_model`; its two loomengine callers and its tapestry callers pass it on unchanged.

**Specs and open maps (D9).** `ConfigSpec().Categories`, `ConfigOpenMaps()`, `TapestryConfigSpec().Categories` and `TapestryConfigOpenMaps()` name the new dotted paths, each with its old key's category; `LocalDenied` follows, since it is derived.
`code.reviewer_fan` stays the one reviewer fan path open to the local layer.
The loom spec's `RetiredKeys` keep `fan_review`, `pr_review`, the shed-only keys and `tapestryPlanKeys` under their old flat names, and `tapestryPlanKeyWayForward` becomes a per-key text naming the key's new full tapestry name, such as `set it under the tapestry module as tapestry.plan.max_bounces instead: ...` (D4 last bullet).
Two loom spec rules need care: a retired loom key `plan` covers `plan.<x>` by prefix in `retiredCovering`, which never collides since loom holds no `plan` area; and `pr_review`, a retired loom key, is an exact top-level name that the new `pr` area never equals.

**Budgets (D8).** `reviewbudget.SoftKey`, `HardKey` and `AutoAcceptKey` return `<perch>.max_bounces`, `<perch>.hard_max_bounces` and `<perch>.auto_accept`;
`RunSoftKey`, `RunHardKey` and `RunAmendKey` become `review.max_bounces`, `review.hard_max_bounces` and `review.amend_max_bounces`.
`Route` finds the layer holding a dotted path by reading each layer's leaf paths, highest first, instead of `Stack.Origins`' top-level map, and keeps its CLI form, such as `lym config loom --set code.max_bounces=<n>`.
`CheckLocal` reads each bound's value at its dotted path in the decoded document.
`loomengine.Perch`'s doc names the area, not a key prefix.

**Segment resolution (D8).** `loomengine`'s per-segment model, fan, judge and seed resolution reads `<segment>.<key>` and falls back to `review.<key>`;
the split fixer's fallback reads `raddle.model`; every `shuttleengine.ModelKey` names its nested path (`discussion.model`, `raddle.model`, `plan.model`, `parts.model`, `parts.writer`, `parts.chair`).
The autonomy keys (`pr.review_ask_parent`, `discussion.interactive`, `raddle.auto_approve`) and loomcli's ask-parent and raddle setting constants name the nested paths.
`loomengine.FlagCodeAutoAccept` reads `code.auto_accept` from the nested `code` mapping; a file without it, or without the area, is flagged as before.

**Setting references (D8).** A recipe setting reference splits at its first dot into module and dotted path.
`shedbuild`'s entry-gate operand pattern admits a dotted path after the module (`tapestry.plan.split`).
`configreg.CheckSettingKey` checks the dotted path against the module's template with `nodeAt`-style descent, and `configreg.SettingLookup` returns the value at the dotted path of the module's snapshot; a flat key is a one-segment path, so other modules' references are unchanged.
`shedrecipe`'s hard-coded reads become `tapestry` `plan.lenses` and `parts.max`.
The recipes' references move: `loom.discussion.reviewer_fan`, `loom.discussion.fixer_fan`, `loom.discussion.auto_accept`, `loom.discussion.gate_fixer`, `loom.code.reviewer_fan`, `loom.code.fixer_fan`, `loom.code.auto_accept`, `loom.split.reviewer_fan`, `loom.split.fixer_fan`, `loom.split.auto_accept`, `tapestry.plan.reviewer_fan`, `tapestry.plan.fixer_fan`, `tapestry.plan.auto_accept`, `tapestry.plan.gate_fixer`, `tapestry.plan.split`; `tapestry.implementer` is unchanged.
The recipes' own comments naming those keys name them in full.

**Writers (D8).** `configengine.NestPaths` turns a map of dotted paths to values into the nested object, so `{"code.max_bounces": 2}` becomes `{"code": {"max_bounces": 2}}`; two paths sharing an area share one mapping.
`beamcli`'s `mergeParent` builds its loom run settings through it from `reviewbudget.SoftKey(perch)` and `HardKey(perch)`, so the filed task carries `{"loom": {"code": {"max_bounces": N, "hard_max_bounces": M}}}`.
`hardencli`'s `commitSandboxConfig` sets `friction.model`.
`internal/runprofile/template.yaml`'s blocks nest the same way, such as `loom: {code: {max_bounces: 1, hard_max_bounces: 2}}` and `tapestry: {plan: {max_bounces: 2, hard_max_bounces: 3}}`, and its header names `loom.raddle.auto_approve` and `loom.code.reviewer_model`; `runprofile`'s doc examples follow.
`boardcli`'s help sample names `{"tapestry": {"plan": {"max_bounces": 1}}}`.

**Tests.** Every test fixture or assertion that writes or reads a flat loom or tapestry key moves to the nested path, in the packages and members this card lists; a fixture's value and what the test asserts stay as they were.
`TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig` writes its hub `loom.yaml` as `code: {max_bounces: 7, hard_max_bounces: 9}`.
The shipped recipes' builds stay pinned by the existing `loomrecipe` recipe tests, which build each shipped recipe and so reach the moved references.
The acceptance rows this card adds, as rows of existing tests:
- `TestLoadConfig_Loads` (tier0): the loaded template defaults sit at the nested paths.
  Its table gains a `hubBase` field that writes the row's document as a hub base `loom.yaml` under a board dir beside a worktree anchor; a hub base setting `discussion: {review_ask_parent: true}` loads that value (D1, D9).
  Rows loading `code: {max_bounces: 2}` and, separately, only `review: {max_bounces: 4}` assert `Config.Caps` for the discussion, code and split perches: the perch's own value when set, else `review.max_bounces` (D8).
- `TestLoadTapestryConfig` (tier0): the loaded tapestry template defaults sit at the `plan.*` and `parts.*` paths, and a row setting `plan: {max_bounces: 2}` beside a loom `review: {max_bounces: 4}` gives `Caps(PerchPlan)` 2 through `WithTapestry`, and without the plan value 4 (D8).
- `TestLoadConfig_Refuses` (tier0): a local file setting `discussion: {advisors: []}` is refused by its new path as before; a hub base with `code: {reviewer_model: {x: 1}}` is refused naming `code.reviewer_model`, the deepest key `KeyAtLine` now returns.
- `TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied` loom and tapestry rows move to the nested paths, such as `code.auto_accept` and `plan.lenses`, and `TestModules_LocalLayerRefusals` runs over the nested paths.
- `reviewbudget`'s `TestCheckResolved` and `TestCheckLocal` (tier0): the budget, hard-cap and fallback keys they name and read are the dotted paths, `code.max_bounces` falling back to `review.max_bounces`.
- `loomengine`'s `TestResolveReview` (tier0): a segment with an empty `code.reviewer_model` takes `review.reviewer_model`.
- `TestSettingLookup` (tier0): `loom`, `split.reviewer_fan` resolves the nested value and `CheckSettingKey` admits it; `CheckSettingKey("loom", "subagent_model.x")`, a path through a scalar, is refused as no key of the module.
- `TestMergeParent` (tier0): the filed task's `run_settings` is `{"loom": {"code": {"max_bounces": N, "hard_max_bounces": M}}, ...}`, and passing it through `configreg.PrepareRunSettings`, the step Seed-Child runs, admits it and yields a loom document whose `code.max_bounces` is N and `code.hard_max_bounces` is M.
- `loomengine`'s `TestFlagCodeAutoAccept` (tier0): a file setting `code: {auto_accept: false}` is left unflagged.

**ImpactSummary:** Renames every loom and tapestry key path across loomengine, reviewbudget, the recipe and setting plumbing, the shipped recipes and profiles, two writers and the tests of a dozen packages; behaviour and values are unchanged, and no other module's key moves (D2).
