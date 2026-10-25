```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, I traced what reads or asserts it and which cards cover that set.

**Per change: the set found and the cards that cover it**

1. **`internal/alignment` and `internal/parentreview` imports go.** Callers: loomcli `wiring.go`, `alignmentwiring*.go`, `decision.go`, `review*.go`, `reviewstatus*.go`, three parentreview tests; reportengine `wakes.go`, `runs.go`, `timingstops.go` and seam test; orchengine `prompt.go` and seam test; shedrecipe `recipe.go`, `entries_gate*.go`, `entries_discussionwrite_test.go` and seam test; battencli `wire.go`; loomrecipe `fixture_test.go`.
   - Covered by cards 1, 2, 3, 4 and 5.
   - Not covered: the doc list in `pattern/PATTERN-cwd-resolution.md:29` ("Bound packages"), which still names `parentreview`.
2. **The `loomengine` parent-review accessors and `DiscussionWriterRole` go.** Callers: loomcli `wiring.go`, `review.go`, `reviewstatus.go`, `alignmentwiring.go`; reportengine `rounds.go`, `wakes.go`, `timingwaits.go`, `fake_test.go`, `timing_test.go`; battencli `wire.go`; loomengine `review_test.go`, `discussiontable_test.go`.
   - Covered by cards 2, 3, 4, 5 and 6. Complete.
3. **`Env.ParentReview` goes.** Users: loomcli `wiring.go:673`; shedrecipe `entries_gate.go` and `entries_discussionwrite.go` with their tests; loomrecipe `shape_test.go` (`testEnv`) and `fixture_test.go`; envkit `nilLegal`.
   - Covered by card 3. Complete.
4. **The alignment wiring goes.**
   - Card 3 covers `alignmentwiring.go`, including `hubBoardTasks`.
   - Not covered: the status commit pathspec in `internal/loomcli/wiring.go:113` still stages `shedrun.AlignmentDir`, and `TestStatusCommitPathspec` (`wiring_commitstatus_test.go:161`) still asserts it.
5. **The four `loom.yaml` keys are retired.** Readers: `template.yaml:43-47`; `config.go`; `config_test.go`; `driver.go` and `driver_test.go`; `configreg_test.go:85`; `runsettings_test.go:217-233`; the loomcli seeds `wiring_test.go:68,96`; the webstercli seed `wiring_test.go:83`.
   - Covered by card 6. Complete.
6. **`lyx loom review` and the review stencils go.** Text naming them: `loom-template-discussion.md:143`, `loom-recipe.yaml:104`, `orch-template-role.md:44-46,167,216`, `docs/overview.md:259-260`.
   - Covered by cards 3, 4, 5, 9 and 18. Complete.
7. **The new `decision` escalation cause.** Readers: `escalation.go`; `circling.go`; `arbiterstop.go:77`; `loomcli/circling.go`; `bouncer-template-escalation.md:12-14`; the hard-cap line in `orch-template-role.md:62`.
   - Covered by cards 12, 14 and 18. Complete.
8. **The `goal-scope` and `board-fit` lens texts and the `review-discussion` fan.** Readers: `internal/lenses/config_test.go`, and `internal/loomengine/discussiontable_test.go:413-459`, which uses only the name.
   - Covered by card 7. Complete.
9. **`Merge-In` becomes `Parent-Align`: Go identifiers.** `NameMergeIn`, `landingshed.MergeIn`, `NewMergeIn` and `FixerModeMergeIn` are used across loomrecipe, loomcli, landingshed and reportengine `timingspans.go`.
   - Covered by card 16's member list plus its catch-all line ("remaining test members that name ... take the new names"). Complete for identifiers.
10. **The gate key `merge-in` and the row name as string literals in tests and fixtures.**
    - Card 16 covers only envkit `LandingDeps` and `fixedGateLists`, and gateslot.
    - Missed:
      - `internal/landingshed/verifygate_test.go:129` (`fixedGateTierLists` keyed `"merge-in"`), read by `mergein_test.go:543`.
      - `internal/loomcli/landingdeps_verify_test.go:53` (`TestLandingDeps_VerifyCommandReadsPlanAtCallTime` calls `VerifyCommand("merge-in")`).
      - `internal/landingshed/fixerrun_test.go:87,95` (`fixerTestRow = "Merge-In"`, gate key `"merge-in"`).
      - `internal/landingshed/fixerattempts_test.go:19,48`.
      - The gate.yaml fixtures in `internal/webstercli/verify_integration_test.go:51` (`verifyGateConfig`) and `internal/websterengine/runlevel_git_test.go:70` (`gateScenario.restart`).
      - Row-name literals in `internal/loomcli/finalizeguard_test.go:47`, `landingdeps_test.go:157-158`, `settledguard_test.go:27-28`, `reviewstatus_test.go:436,451`.
11. **The `merge_fix_attempts` key is renamed.** Readers: landingshed `config.go`, `template.yaml`, the `config_test` fixtures, `MergeIn.align`.
    - Covered by card 15. Complete.

**Findings**

1. **MEDIUM**, class `design`, card 16 (`16-parent-align-row.md`, Gate key and the closing catch-all bullet): the rename lists Go identifiers but not the string literals of the old gate key and row name in test fixtures.
   - Missed: `fixedGateTierLists` (`internal/landingshed/verifygate_test.go:129`, read with `"merge-in"` at `mergein_test.go:543`); `TestLandingDeps_VerifyCommandReadsPlanAtCallTime` (`internal/loomcli/landingdeps_verify_test.go:53`); `fixerTestRow` and its gate key (`internal/landingshed/fixerrun_test.go:87,95`); `verifyGateConfig` (`internal/webstercli/verify_integration_test.go:51`); `gateScenario.restart` (`internal/websterengine/runlevel_git_test.go:70`).
   - A ParentAlign fixture keyed `parent-align` that reads `fixedGateTierLists["merge-in"]` misses its tier list.
   - Fix: list these members on card 16, with the literals renamed to `parent-align` and `Parent-Align`.
2. **LOW**, class `consistency`, `pattern/PATTERN-cwd-resolution.md:29`: the bound-package list still names `parentreview` once card 5 deletes the package, and no card edits the file. Fix: add the file to card 17's Prosa targets, or to card 5.
3. **LOW**, class `scope`, `internal/loomcli/wiring.go:113` and `internal/loomcli/wiring_commitstatus_test.go:161` (`TestStatusCommitPathspec`): the status commit pathspec still stages the alignment attempt directory after D1 removes everything that writes it, and its test still asserts that. No card covers either, so the leftover wiring stays. Fix: drop the clause and its test row on card 3, or record why archived directories still need staging.

Tool calls:
1. Grepped docs, yaml and scripts for the removed verbs, keys and stencils, to find readers outside the Go code.
2. Grepped `docs/overview.md`, the cause lists in docs, and tests naming the lenses, to find readers of the new cause and the lens names.
3. Read `overview.md:569`, `discussiontable_test.go`, `loom-status-spec.md` and the escalation stencil's cause list, to confirm cause and lens readers.
4. Grepped loomrecipe, loomcli and landingshed test files for Merge-In identifiers and literals, to find the members card 16 does not list.
5. Read `finalizeguard_test.go`, `landingdeps_verify_test.go`, the `fixedGateTierLists` users and the `fixer_test.go` enclosure, to judge whether the missed literals break.
