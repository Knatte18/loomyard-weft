# Card 12 — Wire `review_stalemate_rounds` and the target snapshot into the rows

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#TestBouncerEntry_CirclingReasonSlugAndBudget`
- `internal/shedrecipe#TestBurlerReviewEntry_ProfileMapping`
- `internal/loomcli#loomCLI.wire`
- `internal/battencli#battenCLI.wire`
- `internal/testkit/envkit#FullEnv`
- `docs/overview.md`

**Uses:**
- `internal/fabricengine#Open`
- `internal/shedadapters#TargetSnapshot`
- `internal/reviewbudget#Route`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/shedrecipe/doc.go`
- `internal/loomcli/doc.go`

**Intent:**
D5 and D6's settings and seams reach the review, fix and Bouncer rows of every review segment.

Config (`PATTERN-config-layers`):
- `Config` gains `ReviewStalemateRounds int` (yaml `review_stalemate_rounds`).
- `template.yaml` sets it to 2 beside `review_circling_checkpoint`, with a comment: the unchanged review rounds in a row that escalate a stalemate, 0 turning the check off.
- `ConfigSpec` classifies it `CategoryNone`: its value only moves an escalation earlier or later and lets no gate pass.
- `LoadConfig` refuses a negative value: `loom config key "review_stalemate_rounds": must not be negative, got <n>; set it to 0 to turn the check off or to a positive integer; way forward: <route>`, where the route is `reviewbudget.Route(dirs, ConfigSpec(), "review_stalemate_rounds")`, so a value set in the local layer or a task's `run_settings` is changed where it lives.
- `docs/overview.md`'s review budget keys line names it, and its paragraph on what a review segment escalates for names the stalemate escalation beside the circling finding and the hard cap.

`shedrecipe.Env` gains `ReviewStalemateRounds int`, `ReviewTreeID func() (string, error)` and `ReviewTreeChanges func(fromID, toID string) ([]string, error)`.
`loomCLI.wire` sets the first from the loom config and the two closures from `fabricengine.Open(location)` opened on each call, then `WorktreeTreeID` and `ChangedPathsBetween`, the way `codeSubject` reaches the fabric.
`battenCLI.wire` fills the two closures the same way over its own location, and `envkit.FullEnv` fills them with stubs, so the reflective seam tests that require every `Env` seam filled (`TestWire_EverySeamFilled` in loomcli and battencli, `TestWire_DarnRecipe`, `TestWire_SamplerRecipe` and `TestFullEnv_FillsEverySeam`) hold.

A shared `TargetSnapshot` per segment, built in the entries:
`WorktreeRoot` is `Env.WorktreeRoot`, `CacheDir` is the anchor's `.lym` joined with `lymdirs.ReviewSnapshotDirName` and the row's `run_subdir`, and the two source closures are the Env closures.
- `burlerReviewEntry` fills `Scope` and `Paths` from its profile's fix scope and target paths and sets `BurlerHalfDeps.Snapshot` and `DecisionRecordPath` from `Env.DecisionRecordPath`.
- `burlerFixEntry` needs neither; it records `decisions_routed` alone.
- `bouncerEntry` sets `BouncerConfig.StalemateRounds` from `Env.ReviewStalemateRounds` and `BouncerConfig.Snapshot` to a `TargetSnapshot` with the roots and closures, whose scope and paths the Bouncer takes from each round's record.
A Code-Review target with no paths hashes the whole working tree in source scope, so its empty `Paths` is legal; an overlay target with no paths hashes nothing and reads as unchanged only when the fixer-report also passes, which Card 10 requires.

Tests, tier0:
- `TestLoadConfig_Loads` gains `ReviewStalemateRounds: 2` in its template defaults and a row loading 0; `TestLoadConfig_Refuses` gains a row refusing -1 whose message ends with the worktree-layer route.
- `TestBouncerEntry_CirclingReasonSlugAndBudget` gains a row driving the built Bouncer's `Call`: an Env with `ReviewStalemateRounds` 2 and a `ReviewTreeID` that answers one fixed id, over two judged CONTINUE rounds laid out with equal-id snapshot records and passing fixer-reports, awaits with a Reason naming cause `stalemate`; the same layout with `ReviewStalemateRounds` 0 does not.
- `TestBurlerReviewEntry_ProfileMapping` gains a row calling the built review producer once over a `shedfake.BurlerHalfRunner` with an overlay target and asserting `round-1-snapshot.json` holds a `target_id` and the Env decision record's added entries.
`TestModules_EveryTemplateKeyClassified` and `TestConfigTemplate_ContainsEveryConfigYAMLTag` hold the key's classification and template line unchanged.

**ImpactSummary:** One new loom key and three Env fields, filled by the loom and batten wiring and the envkit fixture; every review segment's rows get the seams, and a stack without the key loads the template default of 2.
