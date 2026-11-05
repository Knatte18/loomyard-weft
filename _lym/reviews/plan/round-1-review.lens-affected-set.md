```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: for each change a card makes, the set of tests, callers, docs and fixtures that read or assert it, and the cards that cover that set.

## Per change: the set found, and the cards that cover it

- **`lym beam reopen` and the reopen request removed**
  - Set: `beamcli/cli.go` (Command Short at :35, reopen help at :654–666), `beamcli/land.go`, `beamcli/deps.go`, `beamcli/doc.go`, `beamcli/land_test.go`, `loomcli/landingdeps.go:385` (`pullRequestDepsGuardedBy`), `loomcli/wiring_beam_test.go#TestReadReopenRequest`, `pawlengine/{beamawait,landrequest,doc}.go`, `pawlengine/world_test.go`, `hubgeom/dobbygeom.go`, `testkit/envkit/envkit.go:232`, `pawl-recipe.yaml`, `docs/overview.md`, `loomrecipe/doc.go:43–46`.
  - Covered by cards 5, 6 and 18, except `loomrecipe/doc.go:45` (missed).
- **Hold boundary moves from Parent-Align to Finalize**
  - Set: `boardengine/runlock.go` and `boardengine/doc.go:258,265`, `battencli/beamfork.go:48`, the `pawl-recipe.yaml` header, `loomrecipe/doc.go:46`, `battencli/dobbyreads.go:214` (`runTakesNoSlot` doc), `dobbyengine/doc.go:119` (NoSlot paragraph), `loomrecipe#TestPawlRecipe_HoldLiftsOnlyBeforeParentAlign`.
  - Covered by cards 5 and 7, except `dobbyreads.go:214`, `dobbyengine/doc.go:119` and `loomrecipe/doc.go:46` (missed).
- **The parent is no longer brought in by a Parent-Align row at landing (Finalize merges it in, and ready requires merge-parent first)**
  - Set: `beamcli/cli.go:55–61` (`mergeParentCmd` Long: "the beam's own landing brings the parent in at Parent-Align whether or not one ran"), `beamcli/doc.go:92`.
  - No card covers either.
- **The BeamDescribe row and producer removed**
  - Set: `shedrecipe/{registry,recipe,entries_describe,doc}.go`, `shedrecipe/conformance_test.go#minimalEntries`, `loomcli/{wiring,beamwiring,doc}.go`, seam markers in loomcli, battencli and envkit tests, `loomrecipe/pawlrecipe_test.go#wantPawlRows`, `landingshed/{beamdescribe,doc}.go`, `loomshed/loomshed.go#NameBeamDescribe`, `testkit/envkit/envkit.go:366`, `loomcli/pawlwiring.go:1` (file header: "the wiring only a pawl run needs beyond the BeamDescribe row").
  - Covered by cards 4, 5, 6 and 12, except the `pawlwiring.go` header (missed).
- **`pullrequest.API` gains `RequiredChecks`**
  - Set: `Client`, `landingshed#fakePullRequestAPI`, `pairteardown#closeRecordingAPI`. No other implementer exists (envkit imports `pullrequest` but implements no API).
  - Covered by card 3.
- **`ResolvePullRequest` and `resolveWithParentRule` re-signed**
  - Set: `pullrequestmode.go`, `finalize_pr.go:175`, `prreview.go:98`, `pullrequestmode_test.go`.
  - Covered by card 11.
- **`beamForkPrecondition` re-signed**
  - Set: `beamfork.go:40`, `beamfork_test.go:100`.
  - Covered by card 7.
- **`beamUpRefusal` re-signed**
  - Set: `orchcli/start.go:473`, `orchcli/cli_test.go:192`.
  - Covered by card 17.
- **`judgeLandOwners` re-signed**
  - Set: `land`, and `reopen` (which card 6 deletes).
  - Covered by card 13.
- **`WaitingTask.ReadyHold` added**
  - Set: `battencli/overviewdobby.go#dobbyTaskWait` is the only reader of `Ready()`.
  - Covered by card 7.
- **New fields on `pawlengine.Deps` and `landingshed.PullRequestDeps`**
  - Set: the reflective seam tests in loomcli, battencli and envkit, `TestLandingDeps_EveryFieldPopulated`, `intentionallyNil`.
  - Covered by cards 5, 10 and 11.
  - No reflective seam markers exist for `sheddriver#Deps`, `dobbyengine#Deps`, `beamcli#landDeps` or `shedbuild#ShedPaths`, so nothing else is forced there.
- **The interrupt-policy table gains the pawl rows**
  - Set: `loomshed/interruptpolicy_test.go`, `loomrecipe/interruptpolicy_meta_test.go`.
  - Covered by card 9.
- **A recipe-declared retired-row map**
  - Set: the existing retired-row data `loomrecipe.RetiredRows` (`recipekeys.go:156`) and its readers: `loomcli/arm.go:288` (`renameRetiredRows`, which renames at arm time through `shedengine.RenameProducers`), `loomcli/peers.go:88–90`, `reportengine/timingspans.go:136`, `loomrecipe/discussionapproval.go:63`, and the docs `loomrecipe/doc.go:39,51,55` and `loomcli/doc.go:11`.
  - No card covers this set.

## Findings

1. **MEDIUM · design · cards 1, 5 (retired-row map)**
   - **Summary:** The plan adds a second retired-row mechanism and reconciles it with none of the existing one or its readers.
   - **Detail:**
     - The plan adds `shedbuild.Recipe.Retired` (a YAML `retired:` map), `shedengine.Shed.Retired` and `loomrecipe.RetiredRowNames`.
     - `loomrecipe.RetiredRows`, applied by `loomcli.renameRetiredRows` at arm, already carries each recipe's retired rows.
     - No card names or reconciles `RetiredRows` or its readers: `arm.go:288`, `peers.go:88`, `discussionapproval.go:63`, `timingspans.go:136`.
     - The docs `loomrecipe/doc.go:39,51` and `loomcli/doc.go:11` still describe `RetiredRows` as the recipe's retired-row data.
     - The overview's Shared Decisions and D13's premise ("no row-migration mechanism exists") miss this set.
   - **Evidence:** `internal/loomrecipe/recipekeys.go:154-156`, `internal/loomcli/arm.go:288`, `internal/loomcli/darnwiring.go:176-178`, `internal/loomrecipe/doc.go:39`, `internal/loomcli/doc.go:11`.

2. **MEDIUM · consistency · method, the hold-boundary and Parent-Align-landing text**
   - **Summary:** Cards 5 and 7 cover the boundary's code and part of its text, but text that still states the old boundary and Parent-Align's merge-in is missed.
   - **Members missed:**
     - `internal/battencli/dobbyreads.go:214` (`runTakesNoSlot` doc: "takes a slot from Parent-Align on"). No card targets it.
     - `internal/dobbyengine/doc.go:119` ("it takes one from Parent-Align on"). Cards 7 and 16 list the file, but no Intent names this paragraph.
     - `internal/loomrecipe/doc.go:43,46` (pawl's row list; "A run at Parent-Align or any row after it holds its beam"). Card 5 lists the file, but its Intent does not name this text.
     - `internal/beamcli/cli.go:55-61` (`mergeParentCmd` Long: "the beam's own landing brings the parent in at Parent-Align whether or not one ran"). No card targets `beamcli#mergeParentCmd`. This now contradicts D3's refusal 3 and the help text D14 asks for.
     - `internal/beamcli/doc.go:92` (the same claim). The file is listed by several cards, but no Intent names this text.

3. **MEDIUM · consistency · method, the `lym beam reopen` text set**
   - **Summary:** Card 6 removes the verb's text from beamcli, pawlengine, loomcli and `docs/overview.md`, but one reader is missed.
   - **Detail:**
     - `internal/loomrecipe/doc.go:45` still says "Its one reroute edge takes PR-Review back to Beam-Await, which PR-Review follows when `lym beam reopen` asked…".
     - Neither card 6 nor card 18 targets the file, and card 5's Intent does not name it.
     - Card 18 asserts "no text names `lym beam reopen` after card 6", against D14's acceptance criterion.
   - **Evidence:** `internal/loomrecipe/doc.go:45`.

4. **LOW · consistency · card 5 or 12, `internal/loomcli/pawlwiring.go` file header**
   - **Summary:** The header still describes the file as "the wiring only a pawl run needs beyond the BeamDescribe row".
   - **Detail:** Card 5 removes that row's wiring and card 12 deletes `beamwiring.go`. Cards 5 and 10 target only `loomcli#pawlDeps`, never the file's self glyph.
   - **Evidence:** `internal/loomcli/pawlwiring.go:1`.

## Tool calls

1. Grep for `beam reopen` across repo text, `pullrequest.API` implementers, and landing-hold text: the manifest held no repo-wide reader search for these strings.
2. Grep for `//lym:seam` markers of the changed structs, the function holding `landingdeps.go:385`, and `operatorwait.go`: reflective seam tests and that function were not in the manifest.
3. `//lym:seam` markers with context, plus files naming Beam-Describe: to pair each marker with its test, and to find remaining readers of the removed row.
4. Grep in `loomrecipe/doc.go` and `loomcli/doc.go`, callers of `beamUpRefusal`, readers of `WaitingTask`, and pawl row names in the beam, batten, dobby, orch, hubgeom and board packages: the manifest lacked these readers.
5. `beamcli/cli.go` lines 1–64 and `battencli/dobbyreads.go` lines 205–222: to identify which command and which doc own the stale Parent-Align lines.
6. Grep for `loomrecipe.RetiredRows` and its readers: `loomrecipe/doc.go:39` revealed an existing retired-row mechanism the manifest had not read.
