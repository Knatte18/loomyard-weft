# Card 10 — loom-key-messages

**Edit:**
- `internal/loomengine#`
- `internal/reviewbudget#CheckResolved`
- `internal/reviewbudget#CheckLocal` -> `func CheckLocal(module string, set map[string]any, perches []string) error`
- `internal/reviewbudget#TestCheckResolved`
- `internal/reviewbudget#TestCheckLocal`
- `internal/loomcli#`
- `internal/loomrecipe#`
- `internal/loomshed#`
- `internal/burlerengine#`
- `internal/shedadapters#`
- `internal/configengine#ValidateLocal`
- `internal/configengine#ValidateMachine`
- `internal/configengine#warnUnknownKeys`
- `internal/configengine_test#TestValidateLocal`
- `internal/configreg#refuseRetiredKeys`
- `internal/configreg#refuseHubOnlyKeys`
- `internal/configreg#CheckSettingKey`
- `internal/configreg#TestPrepareRunSettings`
- `internal/shedbuild#TestBuild_Errors`
- `internal/envconfig_test#TestLoadConfig_StacksHubAndMachineName`
- `internal/gateslot#TestLoadConfig`

**Uses:**
- `internal/configengine#KeyName`
- `internal/configengine/doc.go`
- `internal/configreg/doc.go`
- `internal/reviewbudget/doc.go`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Every Go message that names a loom or tapestry key, and every message configengine and configreg render for any module's key, names it in full through `configengine.KeyName` (D3).

- `loomengine`: every `loom config key %q` and `tapestry config key %q` refusal renders `KeyName(<module constant>, <path constant>)`, such as `loom.code.reviewer_model`, from card 4's named path constants; a message that also names another key, such as `PartsChairTimeout`'s and `CapAssistTimeout`'s, names it in full too (`tapestry.plan.timeout_min`, `loom.review.timeout_min`), and a CLI command it prints keeps the CLI's form, `lym config loom --set code.max_bounces=2`.
  The review, fixer and fan key strings loomengine and loomcli hand the burler and Bouncer rows, such as `FixFanKey` and `ReviewKey`, are the full names, so `burlerengine`'s fork refusal and `shedadapters`' unrecorded-fan text name `loom.discussion.fixer_fan`.
- `reviewbudget`: `CheckResolved`'s refusals name the owner module's key in full (`loom.code.max_bounces`, `loom.review.hard_max_bounces`, `tapestry.plan.max_bounces`) and `CheckLocal`'s refusals name the module the caller passes; `Route` keeps its CLI form.
  `CheckLocal` takes the module's name so it can render the full name, and its two callers, loomengine's `ValidateLocalValues` and `ValidateTapestryLocalValues`, pass `loom` and `tapestry` from their module constants.
- `loomcli`, `loomrecipe` and `loomshed`: every message naming a loom or tapestry key, such as `rownotice`'s `notify_parent_on`, `loomrecipe`'s budget text and `loomshed`'s raddle refusal, names it in full from a named constant.
- `burlerengine` and `shedadapters`: the messages above, and comments naming a loom key, such as `shedadapters`' `review_amend_max_bounces`, name it in full.
- `configengine`: `ValidateLocal`'s and `ValidateMachine`'s refusals name `KeyName(spec.Name, path)` instead of the bare path, and `warnUnknownKeys`' warning lists the keys in full; this covers every module's local and machine refusals.
- `configreg`: `refuseRetiredKeys`' and `refuseHubOnlyKeys`' refusals, and `CheckSettingKey`'s, name the key in full.
Every test asserting one of these texts is a target of this card: the members listed, and the tests inside the packages listed, `burlerengine`'s `TestEngine_Fix_Outcomes` and `shedadapters`' burler-fix test included.
The acceptance rows: `TestPrepareRunSettings` (tier0) asserts a run-settings refusal names `loom.<area>.<key>`, `TestValidateLocal` (tier0) a local-layer refusal, and `reviewbudget`'s `TestCheckResolved` (tier0) a budget refusal, each rendered through `KeyName`.

**ImpactSummary:** Reworded refusal and warning texts across loomengine, reviewbudget, loomcli, loomrecipe, loomshed, burlerengine, shedadapters, configengine and configreg, one re-signed reviewbudget function with two loomengine callers, and the asserting tests of shedbuild, envconfig and gateslot; values and decisions are unchanged.
