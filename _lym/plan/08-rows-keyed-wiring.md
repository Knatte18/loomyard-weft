# Card 8 — rows-keyed-wiring

**Create:**
- `internal/shedbuild#RowEntryGate`
- `internal/loomrecipe#CodePerchGate`
- `internal/loomcli#loomCLI.wireTreeScope`
- `internal/loomcli#discussionRecordCheck`

**Edit:**
- `internal/loomcli#codeBudgetZero` -> `func codeBudgetZero(l *lymcwd.Location, recipe string) func() (bool, error)`
- `internal/loomcli#rerouteReviewed` -> `func rerouteReviewed(l *lymcwd.Location, recipe string) func() (bool, error)`
- `internal/loomcli#landingDeps`
- `internal/loomcli#loomCLI.wireDarn` -> `func (c *loomCLI) wireDarn(location *lymcwd.Location, stencilsDir, specsDir, frictionDir string, registry modelspec.Registry) error`
- `internal/loomcli#darnDeps` -> `func darnDeps(l *lymcwd.Location, runID, recipe string) loomshed.DarnDeps`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestRerouteReviewed_ReadsTheCodeBudgetFromLoomYaml`
- `internal/loomcli#TestDarnDeps_LatestOutcomeNamesTheRowsFindingsFile`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli/doc.go`
- `internal/loomrecipe#TestSamplerRecipe`
- `internal/landingshed#Deps`

**Extends:**
- `internal/shedbuild/doc.go`
- `internal/loomrecipe/doc.go`

**Uses:**
- `internal/loomcli#darnTreeScope`
- `internal/loomcli#runRecordOf`
- `internal/loomcli#recordHistory`
- `internal/loomcli#hubConfigWayForward`
- `internal/shedbuild#entryGateOf`
- `internal/loomrecipe#testEnv`
- `internal/landingshed/doc.go`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:**
D3 and the last bullet of D6: the loom wiring keys darn's verify, budget and record seams on the recipe's rows and keys instead of the DarnWrite engine, and every reading of whether the Code-Review perch is off follows the perch rows' `entry_gate` as well as the budget.
`shedbuild.RowEntryGate` parses `row`'s entry_gate checks and builds them as `Build` does (the existing `parseEntryGate` and `entryGateOf`), returning nil for an ungated row; `shedbuild/doc.go` names it.
`loomrecipe.CodePerchGate` returns `RowEntryGate` of the named recipe's Code-Review segment Bouncer row, nil for a recipe with no such row or an ungated one (tapestry, darn, pawl), and refuses an unknown recipe; `loomrecipe/doc.go` names it.
`codeBudgetZero(l, recipe)` reads loom.yaml's Code-Perch soft cap as before and also reports the perch off when `CodePerchGate(recipe, …)`, built over an env whose `Setting` is `configreg.SettingLookup(fabricengine.ConfigDirs(l))` and `SettingKnown` is `configreg.CheckSettingKey`, does not hold, read at call time; `rerouteReviewed(l, recipe)` negates it.
`landingDeps` passes its `recipe` to both, so a Finalize reroute on a sampler run with `code_review` false and a non-zero budget reads the perch off and halts for the orch instead of rerouting past skipped rows (D3 acceptance).
The `CodeReviewOff` and `RerouteReviewed` field docs of landingshed's `Deps` say the perch reads as off when its budget is 0 or when the Code-Review rows' entry gate leaves them out.
`TestRerouteReviewed_ReadsTheCodeBudgetFromLoomYaml` runs its rows on the darn recipe and gains sampler rows over a hub board holding a `sampler.yaml`: `code_review` false with `code_max_bounces` 2 reads no review and budget zero, `code_review` true with 2 reads a review, and `code_review` true with 0 reads none.
`TestSamplerRecipe` gains rows that evaluate `CodePerchGate(sampler, …)`: it does not hold with `env.Setting` answering `sampler.code_review` false and holds with true; that `CodePerchGate(darn, …)` is nil; and that an unknown recipe name is refused.
`TestRerouteReviewed_ReadsTheCodeBudgetFromLoomYaml` also gains a sampler row whose local `sampler.yaml` sets `code_review: "yes"`, a non-bool, where both seams return the gate's evaluation error.
`wire` stops keying on the DarnWrite engine for everything: for a recipe whose `verify_source` is `tree_scope` (darn and sampler) it calls `wireTreeScope`, which loads the hub-wide darn.yaml strictly with the editor way forward for a malformed tree key and `hubConfigWayForward` otherwise, and sets `VerifyCommand`, `VerifyMergeBase` (the task merge base), `VerifyTreeScope` and `DarnVerifyAttempts`; for a recipe with a DarnWrite row it then calls `wireDarn`, which keeps only `Env.Darn` and `Env.DarnTable` and no longer re-sets `SegmentBounces`, since `wire` already sets it for every recipe from the recipe's own routing.
`discussionRecordCheck(l, recipe)` is the record check `darnDeps` built inline, `loomshed.ValidateDiscussion` over the record, support log and `recordHistory(l, runRecordOf(l, recipe))`, so a sampler run's check reads its own run's record history; `darnDeps` takes the recipe and uses it.
`TestDarnDeps_LatestOutcomeNamesTheRowsFindingsFile` passes `shedrun.RecipeDarn`; `TestWire_DarnRecipe` keeps its assertions, which still hold.
`loomcli/doc.go`'s wiring sentence says the tree-scope verify seams follow `verify_source: tree_scope`, the Darn-Write seams a Darn-Write row, and the perch-off reading the Code-Review rows' entry gate.

**ImpactSummary:** two landing seams gain a recipe argument: `codeBudgetZero` has two production callers (`landingDeps` and `rerouteReviewed`), `rerouteReviewed` one (`landingDeps`), and one test covers both; `darnDeps` gains a recipe argument and `wireDarn` loses its `budgets` parameter; `shedbuild.RowEntryGate` and `loomrecipe.CodePerchGate` are new exported functions; darn's wiring splits into two keyed parts with the same seams set for a darn seed, and tapestry and pawl wiring is unchanged.

**Commit:** 8: loom wiring keys the tree-scope and perch-off readings on the recipe's rows
