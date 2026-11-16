```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each change a card makes, I list what reads or asserts it and which cards cover that.

**Per-change list**
1. Card 1 moves the plan-conflict record path. Readers and writers:
   - `websterengine` `Run` (runlevel.go:403) and `FixPlanSpawnOrAttach` (planfixer.go:104,145); `webstercli` beginbatch.go:116,180,214; `shedadapters` webster.go:111.
   - Tests: `seedConflictRecord`, `TestRun_RefusesBeforeSpawn`, `TestRun_EntryHousekeeping`, `newFixPlanFixture`, `TestFixPlanSpawnOrAttach`, `TestBeginBatchCmd_PlanConflict`, `TestFixPlanCmd`, `TestWebsterProducer_OutcomeStuck`, `seedWebsterConflictRecord`, `TestRecipe_WebsterReroutesToPlanReview`.
   - Text: tapestry-recipe.yaml:41,260; runplan/doc.go; runplan/geometry.go:33; websterengine/doc.go:828; webster-spec (card 26).
   - Also affected and not listed by card 1: `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:364`, the file header at `internal/runplan/conflict.go:2`, and the test-only allowlist entry at `cmd/lym/testonlycode_linux_test.go:61`.
   - Covered by card 1 (and card 26), partly.
2. Card 4 makes the Webster done-card and begun-card readers delegates. Callers: loomshed/gates.go:254-276, loomcli/renumber.go:102. Signatures don't change. Covered by cards 4, 19 and 20.
3. Card 17 re-signs `loomshed.NewWebsterProducer`. Callers: entries_simple.go:177 and the tests `TestWebsterProducer_Call`, `_CommitsTheRunRecordOnDoneOnly`, `_PassesStuckReasonThrough`, `TestBatcherRows_StuckSurfacesTheBatcherError` (gatefindings_test.go:142), `TestGateRows_StopsNameAWayForward` (gates_test.go:61,150), `TestCancellation_RealProducersReturnErrorNotStuck`, entries_implementer_test.go:35 and shape_test.go:100. Card 17 covers all of them.
4. Card 18 adds `Env` fields. The `Env` fixtures and seam-marked tests are listed in card 18; nothing missing seen.
5. Card 19 changes `WebsterRecordNote`'s text. Asserted by publishfailure_test.go:50 (`TestWebsterRecordNote`). Other tests (entries_bouncer_test.go:79, bouncerprompt_test.go:144, rubric_test.go:138, contracts/stencils/rubric_test.go:79) plant their own note string and don't read the function. Covered by card 19.
6. Card 20 changes the begun-bound text and the lease wording. Readers: renumber_test.go:334,336, loomcli/doc.go:344, docs/overview.md:1025-1026, renumber.go:32,91,122. Covered by card 20.
7. Card 22 deletes `ArchivedWebsterDirs`. Readers: wiring.go:932, generation_test.go, interruptpolicy_test.go:49, loomshed/doc.go:290. Covered by card 22. The mention in docs/benchmarks/test-redundancy.md is a measurement report, so exempt.
8. Card 22 adds a second slot to the rework archive. Readers: planwrite.go:150 and the parts rotator, which share the seam. Covered by card 22.
9. Card 23 changes the merge-in warning text. Only merge_verbs.go reads it, plus the scenario test. Covered by card 23.
10. Card 24 re-signs `arbiterRecipeDescription`. Callers: arbiter.go:164 and recipekeyed_test.go:70. Covered by card 24.
11. Card 25 branches `fixerTimeout`. Only wire.go reads it, plus its test; the comment at gateslot/template.yaml:3 stays true. Covered by card 25.
12. Cards 3, 5 and 9–14 move implementer state into `_lym/weaver/`. Every reader of Webster's state should now read the active implementer's. Cards 19–25 cover D13's list. Also affected and not covered: `internal/failuretrace/trace.go:332` through `internal/loomcli/landingdeps.go:82`.

**Findings**

1. **MEDIUM — decision.** Location: card 1 (shared-plan-conflict-path), plus the decision record's D13 reader list.
   - Summary: under Weaver, the failure tracer that Parent-Align and Finalize use still loads Webster's state only. No card routes it through the seam.
   - Evidence: `internal/failuretrace/trace.go:332` calls `websterengine.LoadState(websterengine.Dir(anchorPath), …)`. `Trace` errors on a nil state (trace.go:51, "no webster state to trace against"). `internal/loomcli/landingdeps.go:82` wires it in. `internal/landingshed/verifygate.go:104-107` turns the error into a Warn.
   - Effect: under Weaver, every landing verify-failure record silently loses its card attribution (`Cards`, `LaterRange`, `Untraced`). The cards that cover D13's readers (19–25) don't list `failuretrace`.

2. **MEDIUM — consistency.** Location: card 1.
   - Summary: card 1 changes the Plan-Burler-Review instructions in `tapestry-recipe.yaml` but leaves out the fixture the motif block test compares against.
   - Evidence: `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:364` still names `.lym/webster/plan-conflict.md`. `internal/loomrecipe/blockrecipes_test.go:34-69` checks the fixture's rows against the expanded tapestry recipe with `reflect.DeepEqual`.
   - Effect: that test fails until the fixture is updated.

3. **MEDIUM — consistency.** Location: card 1.
   - Summary: the new `runplan.ConflictRecordRel` has no production caller. Its only caller, `websterengine.ConflictRecordRel`, is already allowlisted as test-only.
   - Evidence: `cmd/lym/testonlycode_linux_test.go:61` has `{Key: "internal/websterengine.ConflictRecordRel", Why: "oracle … in internal/loomrecipe's recipe test"}`. The recipe text is static YAML, and card 1 says `websterengine.ConflictRecordRel` returns `runplan.ConflictRecordRel()`.
   - Effect: the dead-code guard reports `runplan.ConflictRecordRel` as unreachable. Card 1 doesn't list `cmd/lym/testonlycode_linux_test.go`.

4. **LOW — consistency.** Location: card 1.
   - Summary: `runplan/conflict.go`'s file header still places the record in the implementer's scratch directory. Card 1 rewords only the `Geometry.ScratchDir` comment and `runplan/doc.go`.
   - Evidence: `internal/runplan/conflict.go:2` reads "the never-tracked plan-conflict record that refusal leaves in the implementer's scratch directory".

Tool calls:
1. A grep for the changed strings and the re-signed or deleted symbols across internal, cmd, docs, pattern and contracts — the manifest had the definitions, not every reader of the text.
2. Mapped caller line numbers to their enclosing test functions, and found the failure tracer and the motif fixture's reader — needed to check card coverage by test name.
3. A grep for the failure tracer's callers and its use of state — one hop past the manifest.
4. Read how `Trace` handles a nil state, how the landing gate handles the error, and how the motif fixture is compared — to confirm both effects.
5. Read the motif test's row comparison and the landing trace's warn path — to confirm the fixture's config text is compared and the trace error is silent.
