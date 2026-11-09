```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test |
|---|---|
| `arras` rows, order, `Motif` `SkipTo`, routing, `Raddle-Burler-Fix` `raddle` gate, absent `carry_over`/`approve_seam`/`commit_seam`/`subject` | `TestArrasRecipe` (`arrasrecipe_test.go:36`) |
| `arras` accepted by the loader and the structural check | `TestRecipe_StructuralCheckHasNoFindings` row, `New(arras)` in `TestArrasRecipe` |
| Entry numbers | `TestEntryNumbers` row |
| Row-name order | `TestRowNames_FollowEachRecipesRowOrder` row |
| `ValidateRecipe("arras")` accepted, `RecipeNames` | `seed_test.go` (reversed row; `want` list) |
| `shedcli` arming table | `TestRecipes_KeySet`, `TestRecipes_Entries` |
| `battencli` admitted child and refusal text | `TestChildRecipeSupported`, `TestWire_WriteSeedRefuses…` |
| Run profile `defaults.arras` | `runprofile_test.go:48`, `boardcli/verbs_test.go:912`, `configsync_test.go` |
| `HoldsSplitPerch(arras)` | `TestHoldsSplitPerch` |
| Discussion approval row | `TestDiscussionApprovalRowsCarryApproveSeam` and `TestArrasRecipe` |
| `escalate_decisions` / `decisions_escalated` rows | `TestDecisionSwitches_SetTogetherOnLoomsDiscussionRowsOnly` |
| Fan settings agree within the segment | `TestShippedRecipes_FanSettingsAgreeWithinSegment` |
| Interrupt policies for the three new rows | `TestInterruptPolicies_MatchAssembledRows` |
| Arbiter description | `TestRecipeKeyedWiring` `Arras` row |
| `finishesAsSketch` false for arras | `TestRecipeKeyedWiring` |
| Re-arm on a fresh entry after a Raddle-File return | `TestArrasRun_SplitPerchReviewsEveryProposalBeforeFiling`, second case |
| Perch reviews the proposal before filing, first pass | Same test, first case |
| `Raddle-Review` segment settings (models, `split_max_bounces`, hard cap, fixer key `raddle`) | `TestWire_ReviewKeysReachTheEnv` row, `TestWire_DefaultConfig` |
| `split_*` classification, `split_auto_accept` as a guard key | `configreg_test.go:143` and `:312` |
| Rubric registered, embedded, markers allowed | `TestLoomRubrics` row |

**Findings**

1. **MEDIUM, design. Run-level coverage is missing for D2's unsettled-goto behaviour and D4/D5/D7.**
   - The new tests drive only the settled re-arm path. `arrasrecipe_test.go:172` has no case for a goto into `Raddle-Write` from an unsettled perch awaiting its parent.
   - The same file has no case for an auto-accept with only `mechanical` open entries, or for `goto` setting the proposal aside.
   - It has none for one-task filing and none for `decision add --by operator` during `Raddle-Write` reaching every part.
   - It has none for `raddle-approve` refusing in an arras run's perch mode.
   - The existing perch-mode tests (`raddlewrite_test.go`, `raddleverbs_test.go`) use fixture recipes. Only the `HoldsSplitPerch(arras)` detection ties them to arras.
   - Evidence: the full `arrasrecipe_test.go`, and the grep showing `RaddleModePerch` only in loomshed/loomcli tests.
2. **LOW, design. `split_judge` has no non-default row.**
   - `ResolveJudge` now resolves `split_judge` through `resolveSegmentBouncerModels("split", …)`. No test sets a non-empty `split_judge`.
   - The grep for `SplitJudge` and `split_judge` finds none in test files.
   - `TestWire_ReviewKeysReachTheEnv` expects the Discussion judge and seed, which come from the default fallback.
   - The wiring row `Bouncer: …wantBouncerModels["Discussion-Review"]` is the only coverage.
3. **LOW, design. The `LoadConfig` bounds and model/fan validation for `split_*` keys have no reaching test.**
   - The new `PerchSplit` entries in `PerchSoft`/`PerchHard` (`config.go:1399`) have no negative-budget row.
   - The same goes for `ValidateLocalValues`, `segmentModelKeys` and `fanKeys` on the split keys.
   - The grep for `split_max_bounces`/`split_hard`/`split_reviewer_model`/`split_fixer_model` hits only `configreg_test.go:315` and `wiring_test.go:988`, so no test feeds a bad `split_*` value.
   - If a generic perch-table test covers these, it is outside what I could see, and the finding would drop.
4. **LOW, design. An empty `split_fixer_model` with `cfg.Raddle == ""` has no row.**
   - `ResolveReview`'s branch at `review.go:794` is pinned only for the set case: `FixKey: loomKey("raddle")` in `TestWire_ReviewKeysReachTheEnv`.
   - The summary's claim that a bare `Config` keeps `fixer_model` is untested.
5. **LOW, consistency. `BouncerRunSubdir` for `Raddle-Bouncer` (`split`) is not asserted in `TestBouncerRunSubdir`.**
   - The loop in `bouncerrows.go` now also reads the arras recipe.
   - The test at `bouncerrows_test.go:200` was not changed by the diff.
   - The run-level test only checks that `<RunRoot>/split/round-1-review.md` exists.
6. **LOW, design (test economy). `TestArrasRecipe` repeats assertions that other tests already pin.**
   - The `slices.Equal(names, arrasRowNames)` check repeats `TestRowNames_FollowEachRecipesRowOrder`'s arras row.
   - The `HoldsSplitPerch(arras)` assertion (`arrasrecipe_test.go:83`) repeats `TestHoldsSplitPerch`.
   - The `DiscussionApprovalRow(arras)` assertion (`:86`) repeats `TestDiscussionApprovalRowsCarryApproveSeam`.
   - The new `arrasRowNames` slice is also a hand-typed duplicate of `TestEntryNumbers`' arras row.
7. **LOW, design. `runprofile/template.yaml`'s four new `arras` blocks have no assertion.**
   - They carry `split_*` values.
   - `runprofile_test.go` changes only `wantDefaults`, so a wrong key or value inside an `arras` block would pass.
   - `template.yaml` is Go-read config and so is not exempt as static prose. The same applies to the `template.yaml` defaults for `split_auto_accept: false` and `split_max_bounces: 2`.
   - `split_max_bounces: 2` is pinned only by `templateConfig` (`config_test.go:135`). The `false` default is pinned only by the zero value in `templateConfig`.

**Tool calls**
1. One `Bash` call, three greps. It read `BouncerRunSubdir` tests, `split_*`/`PerchSplit`/`SplitJudge` in tests, and `arras` in `runprofile` and `boardcli` tests. My context lacked these test files, which were not in the diff or the manifest.
