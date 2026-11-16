# Card 1 — shared-plan-conflict-path

**Create:**
- `plan:internal/lymdirs#ImplementerDirName` -> `const ImplementerDirName = "implementer"`
- `plan:internal/runplan#ConflictDir` -> `func ConflictDir(anchorRoot string) string`
- `plan:internal/websterengine#Geometry.ConflictDir` -> `func (g Geometry) ConflictDir() string`

**Edit:**
- `internal/runplan#Geometry`
- `internal/runplan/doc.go`
- `internal/runplan/conflict.go`
- `internal/websterengine#ConflictRecordRel`
- `internal/websterengine#ScratchDir`
- `internal/websterengine#Geometry`
- `internal/websterengine#Run`
- `internal/websterengine#FixPlanSpawnOrAttach`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI.beginBatchCmd`
- `internal/shedadapters#WebsterProducer.Call`
- `contracts/recipes/tapestry-recipe.yaml`
- `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml`
- `internal/websterengine_test#seedConflictRecord`
- `internal/websterengine_test#requireNoConflictRecord`
- `internal/websterengine_test#TestRun_EntryHousekeeping`
- `internal/websterengine_test#TestRun_RefusesBeforeSpawn`
- `internal/websterengine_test#newFixPlanFixture`
- `internal/websterengine_test#TestFixPlanSpawnOrAttach`
- `internal/webstercli#TestBeginBatchCmd_PlanConflict`
- `internal/webstercli#TestFixPlanCmd`
- `internal/shedadapters#seedWebsterConflictRecord`
- `internal/shedadapters#websterDepsWithScratch`
- `internal/shedadapters#TestWebsterProducer_OutcomeStuck`
- `internal/shedadapters#TestWebsterProducer_ConflictRerouteCapThenRepeatHalts`
- `internal/shedadapters#TestWebsterProducer_OtherEngineErrors`
- `internal/shedadapters#TestWebsterProducer_CancelledDuringRun_StuckPausedAndErrorBecomeContextError`

**Uses:**
- `internal/webstercli/doc.go`
- `internal/loomrecipe#TestRecipe_WebsterReroutesToPlanReview`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Move the plan-conflict record and the last-routed conflict out of Webster's scratch directory onto one implementer-neutral path both implementers write (D13), so a reroute to Plan-Burler-Review sees a conflict whichever implementer refused.

- `lymdirs.ImplementerDirName` names the `implementer` segment, the one declarer of it.
- `runplan.ConflictDir(anchorRoot)` returns `<anchorRoot>/.lym/implementer`, joined from `lymdirs.DotLymDirName` and `lymdirs.ImplementerDirName`.
  The existing record functions (`ConflictRecordPath`, `WriteConflictRecord`, `ClaimConflictRoute` and the rest) keep their `dir` parameter; callers now pass `ConflictDir`.
  `WriteConflictRecord` creates the directory first, since no lock file of either implementer sits in it.
- `websterengine.Geometry.ConflictDir()` returns `runplan.ConflictDir(g.AnchorRoot)`.
  `ConflictRecordRel` returns `.lym/implementer/plan-conflict.md`, forward-slashed for recipe text, joining `lymdirs.DotLymDirName`, `lymdirs.ImplementerDirName` and `runplan.ConflictRecordName` with `path.Join` itself, so the allowlisted test-only oracle stays the one reader of the relative path and no unreachable `runplan` function is added.
  `Run`'s entry removal of the record, `FixPlanSpawnOrAttach`'s read, begin-batch's write and removals in `webstercli`, and `WebsterProducer.Call`'s claim all pass `Geom.ConflictDir()` in place of `Geom.ScratchDir`.
- `runplan.Geometry.ScratchDir`'s comment and `runplan/doc.go` stop placing the record in the implementer's scratch directory and name `ConflictDir`;
  `websterengine/doc.go`'s "The plan-conflict record" section names `.lym/implementer/`.
- `conflict.go`'s file header and `ConflictRecordName`'s comment place the record and the last-routed conflict in the implementer-neutral conflict directory, not the implementer's scratch directory.
  `websterengine.ScratchDir`'s comment and the `Geometry.ScratchDir` field's comment no longer list the plan-conflict record or the last-routed conflict.
- The tapestry recipe's Plan-Burler-Review instructions name `.lym/implementer/plan-conflict.md` and say "the implementer's begin refused the approved plan", not Webster's begin-batch.
  `TestRecipe_WebsterReroutesToPlanReview` already asserts the instructions name `ConflictRecordRel()`, so it now pins the new path.
  The motif fixture `tapestry-rows-before-motif.yaml`, which the block-recipe test compares with the expanded recipe, carries the same new Plan-Burler-Review text.
- The listed tests and helpers seed, read and assert the record at `Geom.ConflictDir()`; a test geometry that sets only `ScratchDir` also sets `AnchorRoot` to the same temp root.

Webster's behaviour is otherwise unchanged; the existing rows of the listed tests are its coverage.

**ImpactSummary:** Relocates one never-tracked record for every reader and writer (webster engine, its CLI, the shed adapter and the recipe text); no state or schema change.
