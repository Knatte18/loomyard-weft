```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → pinning test**

1. The `undiscussed` gate holds, passes over the block, or returns the seam's error. It is pinned by `internal/shedrecipe/entries_block_test.go`, "undiscussed holds unless the seam seeded the stored record" (three rows).
2. The `undiscussed` gate refuses an Env with a nil seam. It is pinned by the "undiscussed refuses an Env with no SeedDiscussedRecord seam" subtest in the same file.
3. `BlockGateNames` lists `undiscussed`. It is pinned by "the table lists its names sorted" in the same file.
4. `seedDiscussedRecord` seeds and commits the record and the support log for a discussed entry, and writes nothing for an undiscussed or absent entry. It is pinned by `TestSeedDiscussedRecord_Real` (3 rows, tier 1, `internal/loomcli/discussionrecord_integration_test.go:99`).
5. `seedDiscussedRecord` with a record already on disk, a support log already present, or a flagged entry whose record file is missing: **none**.
   - `DecisionRecord` reports not found for the missing-file case, which would run Discussion.
   - The "support log written only when missing" branch has no row.
   - The record-already-on-disk overwrite is the case the summary flags for hand-checking.
6. `seedDiscussedRecord` returning a board-open, write or commit error: **none**.
   - The gate-level test only injects a seam error.
   - Nothing pins that `seedDiscussedRecord` returns the error rather than `false, nil`.
7. `Env.SeedDiscussedRecord` is wired into `wire`, so a real tapestry run reaches the seam: **none**.
   - The `envkit.FullEnv` stub returns false.
   - `battencli/wire_seams_test.go` only lists the field as loom-only.
   - `TestMotifRecipeRunsAlone` uses `testEnv`, and I did not check whether it supplies the seam.
   - If `wiring.go:882` were dropped, `New(tapestry)` would fail on the nil-seam refusal only if the loomrecipe tests build tapestry with a real or stub Env. Judge this as a weak pin.
8. Tapestry's expanded rows and top-level keys equal the pre-change snapshot. They are pinned by `TestTapestryMotifBlock`, "rows" and "top-level keys" (`internal/loomrecipe/blockrecipes_test.go`). The rows subtest compares the full `Row` structs including config and `BlockGate: undiscussed`, so it is not a weak pin.
9. The motif recipe runs alone with the Discussion rows only and Bouncer `OnDone` empty. It is pinned by `TestMotifRecipeRunsAlone`. The loom contract's record-only form is pinned by `TestBuild_LoomContract`, "a record-only recipe… passes", through `New(motif)`.
10. A recipe with only a build row, or only a Finalize row, is still refused. This is **none** for the new `recordOnly` boundary.
    - The existing contract refusal rows may cover missing-build and missing-finalize cases.
    - No row was added for the half-present case the summary says is refused as before.
    - I did not verify that the old rows hit the `recordOnly` boundary.
11. `Expand` ignores a block recipe's `contract`, `verify_source`, `amendment_target` and `profile_recipe`. It is pinned by the rewritten `TestExpand` subtest "the parent's terminals and top-level keys stay the run's", which asserts all four and replaces the four removed refusal rows. The removed refusals were deleted without a counter-test, which is acceptable.
12. A finished run of a recipe with no Finalize row writes the draft status.
    - `newCommitStatusSeam` writing `FinishedStatus` is pinned by `TestNewCommitStatusSeam_FinishedStatus`.
    - The recipe-to-status mapping is pinned by the `finishedBoardStatus` column in `recipekeyed_test.go:44,70`.
    - The wiring of `c.commitStatusDeps` into both call sites (`wiring.go:307,326`) is pinned by **none**. Reverting a call site to `loomCommitStatusDeps` would fail no test.
13. The arbiter description for a recipe with no Finalize row is pinned by the motif row in `recipekeyed_test.go:44` (fragment "a motif run, a task's Discussion").
14. The passed-approval test finds Bouncer's approval inside the Motif block. The Motif row is pinned as "before the segment" in `TestPastDiscussionApproval`.
    - Rows after the segment for tapestry were already covered.
    - The test does not assert that `DiscussionApprovalRow(tapestry)` is `Discussion-Bouncer` found through the block. `discussionapproval_test.go:13` pins the approval row each recipe answers, so this is covered if its rows include tapestry.
15. `RecipeNames` and `ValidateRecipe` accept motif and refuse the former arras name. They are pinned by `seed_test.go` (`TestValidateRecipe`, `TestRecipeNames`). The shed arming table is pinned by `shedcli/table_test.go:40`.
16. Seed-Child and Run-Shed admit motif. They are pinned by `battencli/wire_test.go:457,468,517` and `runsettings_test.go:35,41`.
17. The board's task-recipe refusal text names motif. It is pinned by `boardengine/task_test.go:195`. The `lym board upsert` help text change has no test, and as static prose it is exempt.
18. Run reports and tokencount drop arras. They are pinned by the edits in `reportengine/runs_test.go`, `timing_test.go` and the tokencount tests. The `Motif` row under the discussion phase (`timingspans.go`) is pinned by **none** that I could see in the diff stat: only a `timing_test.go` change of 4 lines.
19. Status panes number tapestry's rows one higher. This is pinned by `TestEntryNumbers`, with the motif run's own numbering.
20. The `RecipeEngines` set no longer holds `Block`. It is pinned by `TestRecipeEngines_ReportsExactlyLoomsOwnEngineSet`. The tapestry recipe still holds the Block engine through its Motif row, so the expected list there deserves a check, but the test derives the set from the recipes.

**Findings**

- MEDIUM, design: `seedDiscussedRecord` has no test for a board-read failure or a write/commit failure returning the error, nor for a flagged entry with a missing record file. These are the failure paths the focus names.
  - Location: `internal/loomcli/discussionrecord.go:203-229`.
  - Test evidence: `internal/loomcli/discussionrecord_integration_test.go:101-169` has three rows (discussed, undiscussed, absent) and no error row.
- LOW, design: the record-already-on-disk and support-log-already-present branches of `seedDiscussedRecord` (`discussionrecord.go:215-224`) have no row. The summary's hand-check item relies on that overwrite.
  - Test evidence: `discussionrecord_integration_test.go:101-169` seeds into a fresh pair only.
- LOW, design: switching the two `newCommitStatusSeam` call sites to `c.commitStatusDeps(location)` is unpinned. A regression to `loomCommitStatusDeps` leaves a motif run's entry in the run lock and lets dobby restart it, and no test fails.
  - Location: `internal/loomcli/wiring.go:307` and `:326`.
  - Test evidence: the new tests call `newCommitStatusSeam` and `finishedBoardStatus` directly (`wiring_commitstatus_test.go:134-160`, `recipekeyed_test.go:70`).
- LOW, design: `Env.SeedDiscussedRecord` being wired in `wire` at `wiring.go:882` has no test. The shared Env stub returns false, so a dropped line is caught only if some test builds tapestry with the real `wire`.
  - Test evidence: `envkit.go:427`.
- LOW, design: the half-present boundary of the record-only contract (a build row without a Finalize row, or the reverse) has no new row at `recordOnly` (`internal/shedbuild/loomcontract.go`). The new test adds only the both-absent case.
  - Test evidence: `build_test.go:248-257`.

I found no duplicated or redundant new tests. The `Rows` and `top-level keys` subtests of `TestTapestryMotifBlock` replace the old `TestBlockRecipes` coverage one for one.

**Tool calls**
1. Bash grep and diff for `lands no code`, `finishedBoardStatus`, `recordOnly` and `PastDiscussionApproval` in tests, plus the test diffs of `discussionapproval_test.go`, `seed_test.go`, `task_test.go` and `recipe_test.go`: my context lacked those test files.
2. Bash grep for `RecipeMotif` and `arbiterRecipeDescription` in tests: my context lacked the other tests that name motif.
