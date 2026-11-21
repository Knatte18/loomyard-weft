```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**

- **`batcher.retiredKeyError` way forward (hand delete or reconcile):** `TestActive` row `retiredKeyInRepoDefaultNamesDefaultFileAndFabricReconcile` and `assertRetiredKey` pin the `or run "lym ... reconcile"` clauses. The hand-delete clause is not asserted, but the refusal is reached. Pinned.
- **`runprofile` refusal of a recipe held only under its former name (`Settings`, `Default`):** `TestFormerRecipeNameRefusal`. Pinned.
- **`runprofile/template.yaml` plan keys under `tapestry:`:** `TestTemplateLadder`, which reads the module per segment. Pinned.
- **`PrepareRunSettings` refusing `loom.plan_max_bounces` (the key move is gone):** `internal/configreg/runsettings_test.go:48`, row "a loom plan_max_bounces is refused naming the key…". Pinned.
- **`PrepareRunSettings` refusing the old python switch and the retired shuttle claude binary key:** the rewritten rows in `runsettings_test.go`. Pinned.
- **`shedbuild.Parse` refusing a top-level `retired:` map as an unknown key:** `TestParse_ValueAndErrorCases` row "a retired map is refused as an unknown key". Pinned.
- **`shedrun.runSegment` with no `self` fallback:** `TestRunDirResolution` row "a stale self dir does not redirect self or the slug". Pinned.
- **`shuttleengine.isTerminalOutcome` and `dispositionCandidate` (`asking` gone, empty Outcome never attachable, `stopped` terminal):** `TestAttach_OutcomeDisposition` row `omitted_outcome_is_never_attachable` and the `TestAttach_UntrackedTerminalRecord_*` loops over the terminal values including `runOutcomeStopped`. Pinned.
- **`shuttleengine.Runner.reconstruct` (state copy, pid save, `PromptOffset` replay):** `TestAttach_HeldRuns` rows. The surviving mutant at `attach.go:316` only flips a Warn log. Pinned; the survivor is no finding.
- **`tendercli` pause entries without `AdoptCaller` or `OwnerCaller`:** `TestPause_AnAlreadyPausedSlugAnswersAlreadyPausedAndKeepsItsEntry`, `TestPausePair_PausesTheSlugOnceUnderTheDefaultCaller` and `TestPauseIntentEntries`. Pinned.
- **`fabricengine` `excludeLinesToStrip` with the anchored pattern only:** `TestExcludeLinesToStrip` (`unwire_exclude_test.go`). Pinned.
- **`fabricengine.namesWiredInSiblingWorktrees` (same-anchor compare and link check):** none. The mutants at `junction.go:632` and `:637` survive.
- **`fabricengine.moveWeftContent` committing a staged-deletion-only name (the fix in `467b8093`):** none found.
- **`fabricengine.seedWeftArtifactExcludes` idempotent seeding (`.weft/` and `.lym/` only):** `TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent` pins the entry set. Dropping the `present[...]` mark survives, so idempotence is effectively unpinned.
- **`configreg.Modules` appending the shuttle engines' modules:** none. The mutant at `configreg.go:174` survives.
- **`configsync.ReconcileHubWideAt`'s seed-only branch passing `templateBasePresent && !fileAbsent`:** none. The mutant at `configsync.go:482` survives.
- **`fabriccli.runReconcile`'s `warp_binding_detail` envelope field once the backfill is gone:** none. The mutant at `fabric.go:742` survives.
- **`shedadapters.RecordCirclingDecision` creating the decision with `automatic` false:** `TestRecordCirclingDecision_WritesPendingFile` does not catch the flag; the mutant at `circling.go:74` survives.
- **`verifytree` kinds, a removed constant only:** `TestLandingFailureRecord` round-trips with `testFailureKind`. No behaviour added.

**Findings**

1. **MEDIUM, design:** `internal/fabricengine/anchormove.go` `moveWeftContent`, the `stagedOnly` branch.
   - **Summary:** the fix in `467b8093` (commit a weft name whose only change is a staged deletion, so the records do not stay staged) has no test.
   - **Evidence:** the commit's test changes add only fixture adjustments (`anchorchange_integration_test.go`, `livestate_manifest_selftest_test.go`, `reconcile_migrates_markers_integration_test.go`). No anchor-move test stages a deletion and asserts the move's commit carries it and leaves the index clean.
   - **Fix:** add a `TestAnchorChange` row that tracks only run records under `_lym`, stages their deletion and asserts a clean `git diff --cached` after the move.
2. **MEDIUM, design:** `internal/fabricengine/junction.go:632` and `:637`, `namesWiredInSiblingWorktrees`.
   - **Summary:** the rewritten loop is unasserted; both negation mutants survive. These are the `AnchorRel` compare, now `continue` on a different anchor, and the `linkErr == nil && isLink` check.
   - **Evidence:** `unwire_exclude_test.go:21-65` builds `siblingWiring{sameAnchor: …}` by hand and never runs the walk.
   - **Fix:** add rows that unwire with a sibling linking the name at the same anchor (the line is kept) and at another anchor (the line is stripped).
3. **MEDIUM, design:** `internal/configreg/configreg.go:174`, the append of `shuttleengines.ConfigModules()` in `Modules`.
   - **Summary:** dropping it survives every covering test, so no test asserts that the engine modules (claude, codex) are registered.
   - **Evidence:** the mutation report's survivor at `configreg.go:174`, possible killers including `TestModules_SeedOnlyAndHubWideFlags`.
   - **Fix:** assert in the `Modules` test that each `shuttleengines.ConfigModules()` name is present.
4. **LOW, design:** `internal/fabricengine/recordsgit.go:92`, `seedWeftArtifactExcludes`.
   - **Summary:** dropping the `present[...] = true` mark survives, so a second seeding that appends duplicate lines goes unnoticed, even though the covering test's name claims idempotence.
   - **Evidence:** the line read at `recordsgit.go:90-92`, and the `Loosened-Check:` trailer on `TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent` in `467b8093`.
   - **Fix:** seed twice and assert the exact exclude content.
5. **LOW, design:** `internal/configsync/configsync.go:482`, the seed-only branch's `templateBasePresent && !fileAbsent`.
   - **Summary:** the clause is unasserted, so a seed-only file seeded from a prime copy while a template base exists is not distinguished.
   - **Evidence:** the mutation survivor at `configsync.go:482`, and the code read at `configsync.go:470-485`.
   - **Fix:** add a `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` row with the hub file absent, a prime copy present and a template base present.
6. **LOW, design:** `internal/fabriccli/fabric.go:742`, `runReconcile`'s `warp_binding_detail`.
   - **Summary:** negating the condition survives, so the envelope's detail field is unasserted now that the backfill is gone.
   - **Evidence:** `fabric.go:742-744`; the kept test `TestCommand_ReconcileLeavesAMissingWarpBindingUnrecorded` asserts the skipped binding but not the detail.
   - **Fix:** assert `warp_binding_detail` present for the unbound hub and absent otherwise.
7. **LOW, design:** `internal/shedadapters/circling.go:74`, `RecordCirclingDecision`'s `createCirclingDecision(…, false)`.
   - **Summary:** flipping `automatic` survives, so the written file's operator or automatic marking is unasserted.
   - **Evidence:** the call at `circling.go:74` and the signature at `circling.go:97-106`, plus `TestRecordCirclingDecision_WritesPendingFile`, which is said to pin the file content.
   - **Fix:** assert the rendered file carries no automatic marker.

**Tool calls**

1. Read the mutation-survivor lines (`attach.go`, `junction.go`, `recordsgit.go`, `configsync.go`, `configreg.go`, `fabric.go`, `circling.go`) and `467b8093`'s fabricengine stat. The context held only the report, not the code.
2. Read the `467b8093` diff of `anchormove.go` and its tests, the `createCirclingDecision` signature, and the `configreg.go` and `junction.go` base diffs. These were needed to tell whether each survivor sits on a changed line, and what the fix touched.
3. Grepped the fabricengine tests for staged-deletion and same-anchor coverage, and read the two fixture diffs of `467b8093`, to find a test pinning the fix and the walk.
4. Read the `configreg/runsettings_test.go` diff and grepped for the `loom.plan_max_bounces` refusal row, to confirm a row pins the key-move removal's new refusal.
