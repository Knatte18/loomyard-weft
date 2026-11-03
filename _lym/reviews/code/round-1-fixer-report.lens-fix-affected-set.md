```yaml
lens: fix-affected-set
outcome: reported
```

Findings for lens fix-affected-set. F1, F2 and F3 are not sets of sites, so they are unchecked.

Unchecked findings: F1 (test content), F2 (a single doc paragraph), F3 (one test row).

**LOW. `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:10` and `:56-57`: the fixture header repeats the stale text F5 and F6 fixed.**
- Line 10 reads "The three Drift rows carry entry_gate: drift.enabled".
- Lines 56-57 read "perch exits to Describe without Drift-Verify, which the orch moves back with `lym loom goto --to Drift-Verify`".
- The same fixture's row `Drift-Recheck` at line 577 and its `on_done: Drift-Recheck` at line 455 show the perch now exits to Drift-Recheck.
- The fix changed the twin of line 56-57 only in `contracts/recipes/tapestry-recipe.yaml`.
- This is a frozen pre-Motif fixture copy, and its header comments are probably not load-bearing. If the fixture is meant to mirror the recipe, its comments are now stale.
- The commit `9019c68b` did touch this fixture.

**No further findings.**
- F4: the repo-wide search found no other Drift-Verify phase or row list that omits Drift-Recheck. The `tapestry-recipe.yaml:30` and `darn-recipe.yaml:22` "carry no on_stuck" lists are correct without it, because the review says Drift-Recheck carries an `on_stuck`.
- F5: no other live site names three enabled rows. The only hit is the fixture line above.
- F6: `contracts/recipes/darn-recipe.yaml` already describes Drift-Recheck after the perch, with no stale perch-exit wording.

Tool calls made:
- One grep across `.go`, `.md` and `.yaml` for Drift-Verify lists, three-row lists and perch-exit wording, to cover the F4, F5 and F6 set searches. My context held only the review's named sites.
- One read of `contracts/recipes/darn-recipe.yaml` lines 1-40, plus a grep for Drift-Recheck in the darn recipe and the fixture, to check the darn header and confirm the fixture's rows. My context did not hold that file.
