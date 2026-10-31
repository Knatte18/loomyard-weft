```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the affected-set searches behind F3, F1 and F2 and check that the fix covers every member.

Findings I left unchecked:
- **F1:** I did not rerun a search for other callers of `reviewbudget.Route` and `reviewbudget.CheckResolved`.
- **F2:** I counted no new search. The fixer's own grep already found every row seeding `bogus-recipe`, and card 2's rewritten text covers all of them.
- **F3, card 6:** I did not read whether `TestResolveReviewAndJudge_MalformedSpec` (`internal/loomengine/review_test.go:309`, `:336`) or `TestPlanTable` (`internal/loomengine/plan_test.go:445-448`) asserts the module name. The first has `plan_judge` and `plan_seed` rows. The second's lens-fan error rows check `wantErrNames`. See finding 2.

Findings:

1. **MEDIUM — card 5's Edit list (the F3 fix) misses `internal/loomcli#seedGoDriverRun`.**
   - **Summary:** this smoke helper writes two of the seven removed keys into the local `loom.yaml`. After card 5, `ValidateLocal` refuses that file at load.
   - **Evidence:** `internal/loomcli/smoke_helpers_test.go:186-190` sets `localPath := configengine.LocalFile(loc.AnchorPath(), "loom")` and then `os.WriteFile(localPath, []byte("tackler_max_spawns: 0\narbiter_max_spawns: 0\n"), ...)`.
   - **Why it breaks:** card 5 makes each removed key a `RetiredKeys` entry with `CategoryGuard`, refused in a local file. So every tmux-tier smoke run seeded through this helper is refused at its first load.
   - **Coverage:** no card names `seedGoDriverRun`; it appears only in `_lym/plan/lens-review.lens-forced-edits.md`.
   - **Fix:** the helper writes the two keys into the pair's local `shed.yaml`. Add it to card 5's Edit list.

2. **LOW — card 6's Edit list (F3 fix) may miss two loomengine tests that move with its own changes (unverified).**
   - **`TestResolveReviewAndJudge_MalformedSpec`:** its rows "plan_judge" (`review_test.go:309`) and "plan_seed" (`:336`) test a refusal for the plan keys. Card 6 makes `loomSegmentModelSource("plan", …)` name module `tapestry`. If these rows assert the `loom` module or `loom.yaml`, they break.
   - **`TestPlanTable`:** its lens-fan rows (`plan_test.go:445-448`) reach the `twillLenses` errors, which card 6 changes from `loom config key` to `tapestry config key` for `plan_lenses`. The visible `wantErrNames` list only `plan_lenses`, the fan and `lenses.yaml`, so these rows likely survive.
   - **Coverage:** neither test is on any card.
   - **Evidence:** the hits come from a grep. I did not read the assertions; the tool-call cap stopped me.

Not a finding: `internal/runprofile/template.yaml:18-39` still names `loom: {plan_max_bounces, plan_hard_max_bounces}`. Card 6 states that `KeyMoves` translates these when read, with nothing rewritten, so this is intended.

Tool calls:
1. Grepped Go code and tests for the shed-key fields and keys. The context held only card 5's named targets, not a repository search.
2. Grepped the `RecipeOverride` callers and the shed-key writes in unlisted loomcli and battencli files, to check whether the hits fall outside card 5.
3. Found the enclosing functions of the `RecipeOverride` callers, plus the smoke helper's write target.
4. Named the smoke helper, checked whether any card covers it, and listed every file naming a key that moves to `tapestry`.
5. Showed the lines behind the unlisted hits for the keys that move to `tapestry`.
6. Named the tests behind the `review_test.go` and `plan_test.go` hits and checked card 6 for them.
