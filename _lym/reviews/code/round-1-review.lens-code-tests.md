```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none.

1. **Motif takes its own run-profile block and the `motif` default (#58).**
   - Pinned by `TestPrepareRunSettings` (`internal/battencli/runsettings_test.go`), three rows. They cover the default profile's motif block, `light`'s motif block (never its tapestry block), and a profile with no motif block refused at Seed-Child.
   - Pinned by `TestTemplateLadder` (`internal/runprofile/runprofile_test.go`) for each ladder profile's motif block and `Default("motif") == "solid"`.
   - Pinned by `TestLoad` for `defaults.motif`.
   - The lacking-block reports are pinned by `TestLackingTemplateBlocks` (`migrate_test.go`), `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate` and `TestConfigOverRealHub`.
   - The `boardcli` profiles verb is pinned by `stepProfiles`.

2. **`profile_recipe`, `ProfileRecipe` and `Recipe.ProfileRecipe` removed.**
   - Removed tests covered only the removed behaviour: `parse_test.go`'s decode row, `recipekeys_test.go`'s `wantProfileRecipe` and `ProfileRecipe` accessor checks, and the `like` indirection in `runsettings_test.go`.
   - `Parse` rejecting an old `profile_recipe: x` as an unknown key is not pinned by a row of its own. It is the generic unknown-key strictness. No finding.

3. **A finished Finalize-less run writes `discussed` (#53).**
   - Pinned by `TestWiredCommitStatus_WritesFinishedStatus` (`internal/loomcli/commitstatus_integration_test.go`), row `FinalizeLessRunMarksTheEntryDiscussed`.
   - Pinned by `TestRecipeKeyedWiring` (`internal/loomcli/recipekeyed_test.go`), `wantFinished` for Motif.

4. **`ComputeLayers` puts a `discussed`-status task in layer `Discussed`, and a dependent weighs it as ready.**
   - Pinned by the new `TestComputeLayers` row in `internal/boardengine/layer_test.go`.
   - **Finding 1, LOW, class `consistency`.** Nothing pins that a task with `Discussed: true` and `draft` status stays `Draft`. The decision record's expectation, and the commit message's "a tapestry run that reused a stored record and was then cancelled still reads as `draft`", rest on the layer keying off status alone. The new row sets status and flag together, so a regression to a flag-based check would pass. Location: `layer_test.go` (new row), `layer.go:46-49`. A second row with the flag and `draft` status would close it.

5. **README Discussed subsection, order before Draft, and the `discussed` code span.**
   - Pinned by `TestRenderDiscussedSection` (`render_test.go`).
   - **Finding 2, LOW, class `consistency`.** `doc.go:271` claims the Discussed table carries the Recipe and After columns. The test asserts only the heading, intro line, order and title cell, so a missing Recipe or After column on the Discussed table would not fail. Location: `render_test.go` new test, `doc.go:271-275`.

6. **`lym board list` layer `Discussed` and the `--text` `[discussed]` suffix.**
   - The layer is pinned only through `ComputeLayers` (item 4).
   - **Finding 3, LOW, class `consistency`.** No test pins `[discussed]` in the `--text` listing (`internal/boardcli/text.go:53-55` prints any status) or the JSON `layer` for a discussed entry in `boardcli`. The code is generic over status. The record names the listing as an expected outcome, so a one-row assertion would pin it. This is low risk.

7. **`Board.SetRunStatus` writes nothing when the entry already carries the status.**
   - **Finding 4, MEDIUM, class `design`.** No test pins this. `TestSetRunStatus` (`internal/boardengine/board_test.go:715-757`) is unchanged by the diff. Its rows are no status, done entry and absent slug, so removing the new `task.Status != nil && *task.Status == status` clause would pass. The commit message states the behaviour: "so a repeated transition makes no Bolt commit". Location: `board.go:196`, `board_test.go:715`. The fix is a row that seeds an entry with the status, calls `SetRunStatus` with the same status, and asserts no commit or no board-file change. `skipGit` is on in this test, so a byte or revision check is needed.

8. **Batten's seam writes `<state> · <row>` for rows before Run-Shed only.**
   - Pinned by `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed`. It covers Worktree-Create, Seed-Child (blocked) and Open-IDE written, and Run-Shed, Land and Worktree-Teardown not written.
   - A nil `SetBoardStatus` is covered by the existing `TestNewCommitStatusSeam`, whose deps set no board seam.
   - **Finding 5, LOW, class `consistency`.** A failing `SetBoardStatus` only warns and the commit still proceeds. This is stated in the `commitstatus.go` header and `cli.go`, but no row has the stub return an error and assert the commit and spawn still ran.
   - **Finding 6, LOW, class `consistency`.** `battenCommitStatusDeps`'s real `SetBoardStatus` (`commitstatus.go:138-144`) has no test. It calls `OpenHub` and `SetRunStatus(runID, …)`, relying on a batten run id being the task slug. The loom analogue is covered by `internal/loomcli/commitstatus_integration_test.go`.
   - `RowsBeforeRunShed` has no direct test. It is exercised through the seam test, which lists rows by name, so it is covered indirectly. Its panic branches are internal-invariant guards and need no test.

9. **Start write: a step over a status naming no row writes the first row as `current_producer` and tells `CommitStatus` before the call (#49).**
   - Pinned by `TestStep_StartWriteNamesTheFirstRow` (`internal/shedengine/run_commitstatus_test.go`). It checks the status during the call (A, running, no history) and the calls `[A, B]`.
   - **Finding 7, LOW, class `consistency`.** The branch where the first row is entry-gated is not pinned. After `enter` skips the gated first row and the status is re-read, `started` is recomputed at `run.go:299`. No test shows the start write does not fire again or misfire for a gated first row. The test seeds `commonSeed("")` with an ungated first row.
   - **Finding 8, LOW, class `consistency`.** The resume case still takes the old write. A seed that already names the row and has state `running` writes nothing extra, which is the "ordinary running-to-running path never fires" claim in the `run.go:314-316` comment. It is presumably covered by the existing `CommitStatus` call-sequence tests. The diff changes no existing test for the extra start call. If the existing call-sequence tests seed an empty `current_producer`, their expectations would have shifted. The verify gate is the arbiter of that. No finding unless it fails.

10. **`lym loom status` names the producer in flight (#49).**
    - The diff leaves `internal/loomcli` status code untouched, so the status file is the source. The only pin is item 9's `duringCall` assertion on the status file. No loomcli-level test shows `lym loom status` printing the producer for a fresh run. This follows from the status-file test and is not a finding on its own.

11. **Docs and comments** (`doc.go` files, `shed-recipe-spec.md`, `docs/overview.md`, `SKILL.md`, `motif-recipe.yaml`) are static prose and exempt.

12. **Test-economy check on added tests.** None duplicates an existing test.
    - The new `TestNewCommitStatusSeam_WritesBoardStatusBeforeRunShed` covers a behaviour the existing seam test does not.
    - The `TestStep_*` test is distinct from `TestRun_CommitStatusReceivesTransitionProducerAndState` because it asserts the status during the call.
    - The removed `ProfileRecipe` and `like` tests covered only removed behaviour.
    - No redundancy finding.

Tool calls (1 of 6): `sed -n 710,770p internal/boardengine/board_test.go` plus `git log` on it, because my context did not show `TestSetRunStatus`'s rows or whether the diff touched them.
