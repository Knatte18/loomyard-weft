# Card 16 — pawl-recipe-rows

**Create:**
- `plan:internal/shedrecipe#beamUpEntry` -> `func beamUpEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#beamAwaitEntry` -> `func beamAwaitEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#beamDownEntry` -> `func beamDownEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestPawlEntries_RefuseNilSeams` -> `func TestPawlEntries_RefuseNilSeams(t *testing.T)`
- `plan:internal/loomrecipe#TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign` -> `func TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign(t *testing.T)`

**Edit:**
- `contracts/recipes/pawl-recipe.yaml`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestNames`
- `internal/loomrecipe#TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`
- `plan:internal/loomrecipe#NewPawl`
- `plan:internal/loomrecipe#PawlRouting`
- `internal/loomrecipe#TestNewBeam`
- `internal/loomrecipe#TestBeamRouting`
- `internal/loomrecipe#TestRecipe_StructuralCheckHasNoFindings`
- `internal/loomrecipe#TestShippedRecipes_FanSettingsAgreeWithinSegment`
- `internal/loomrecipe#TestShippedRecipes_LanguageSkillsOnCodeRounds`
- `internal/loomcli#TestRecipeRouting`
- `internal/runprofile/template.yaml`
- `internal/runprofile_test#TestTemplateLadder`
- `internal/shedrecipe/doc.go`
- `internal/loomrecipe/doc.go`

**Intent:**
D8's rows, in order: Beam-Up, Beam-Await, Parent-Align, the Code-Review perch (Code-Bouncer, Code-Burler-Review, Code-Burler-Fix), Beam-Describe, Finalize, Beam-Down.

- `pawl-recipe.yaml`: `entry: Beam-Up`, `terminals: [Beam-Down]`; Beam-Up (`BeamUp`) → Beam-Await (`BeamAwait`) → Parent-Align (`ParentAlign`, loom's row with its merge fixer, no config) → Code-Bouncer; the three perch rows are darn's, with the same names, segment, `run_subdir`, `role_prefix`, rubric, fan and auto-accept settings and loom.yaml's `code_*` keys, their answer key the project entry snapshot `_lyx/discussion/board-entry.md` alone, in `artifact_paths` and the review row's `fasit`, and Code-Bouncer's `on_done` Beam-Describe; Beam-Describe → Finalize, which declares no `on_reroute`; Finalize → Beam-Down, whose empty `on_done` ends the run quietly.
  The perch carries no `carry_over`: a pawl run has no decision record to file still-open findings into, so they stay in the round's review files, and Beam-Describe carries no entry gate, since it writes from the entry and the commits whatever the budget.
  The header comment states the rows, the hold boundary and why Finalize never reroutes.
- `shedrecipe`'s one `Constructor` map gains `BeamUp`, `BeamAwait` and `BeamDown` (`PATTERN-shed-recipe-registry`); each entry rejects every config key and builds its `pawlengine` producer over `Env.Pawl`, a new `pawlengine.Deps` field, the producer's own constructor refusing a nil seam naming the field.
- `NewPawl` reads the Code-Review segment's budget from `env.Segments` and applies it as `NewDarn` does, stamping `zero_rounds` on Code-Bouncer at a budget of 0, refusing a missing segment entry naming the loom wiring's segment table; `PawlRouting(budgets)`, whose `budgets` card 6 already passes from `loomCLI.recipeRouting`, now finds the Code-Review segment and projects it as `DarnRouting` does.
- `profiles.yaml`'s template gains a `pawl` block per profile carrying the perch's `code_max_bounces` and `code_hard_max_bounces`, the same values as the profile's `darn` block, and `defaults` names `pawl: light`.

Tests: `TestNewBeam` asserts the nine rows in order with their engines and the routing's edges, and that a Code-Review budget of 0 settles Code-Bouncer at once; `TestBeamRouting` asserts the budget applied and a missing budget refused; `TestRecipeRouting` asserts a pawl run's routing carries the budget.
`TestRecipe_StructuralCheckHasNoFindings`, `TestShippedRecipes_FanSettingsAgreeWithinSegment` and `TestShippedRecipes_LanguageSkillsOnCodeRounds` cover the pawl recipe beside loom's and darn's.
`TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign` pins `boardengine.RunHolds` against the recipe: a beam project at `running · Worktree-Create` (`battenrecipe.NameWorktreeCreate`) and at every pawl row before Parent-Align reads no hold, and at Parent-Align and every row after it reads held, so a renamed row breaks the test.
`TestPawlEntries_RefuseNilSeams` builds each entry with a zero `Env.Pawl`, refused naming the first nil seam; the unknown-key refusal is `TestEveryEntry_RejectsUnknownConfigKey`'s, which reaches the three entries once `minimalEntries` gives each a valid input over a filled `Env.Pawl`; `TestNames` lists `BeamUp`, `BeamAwait` and `BeamDown`; `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet` expects them beside `DarnWrite` and `BeamDescribe`; the cross-consumer coverage guard sees the three engines used by the pawl recipe.
`TestTemplateLadder` asserts the `pawl` block of each profile and the default.
`shedrecipe/doc.go` and `loomrecipe/doc.go` describe the entries and the recipe.

**ImpactSummary:** the pawl recipe grows from two rows to nine over existing engines plus the three new ones; `NewPawl` and `PawlRouting` keep card 6's signatures, and the profile template gains a block no existing reader rejects.

**Commit:** 16: The pawl recipe: Beam-Up, Beam-Await, Parent-Align, the Code-Review perch, Beam-Describe, Finalize, Beam-Down
