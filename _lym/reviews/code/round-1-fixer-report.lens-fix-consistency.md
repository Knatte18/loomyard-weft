```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for consistency with its siblings and twins (fix-consistency lens).

**Findings**

1. **LOW** — `contracts/recipes/tapestry-recipe.yaml:58-59` (the reworded perch-exit comment).
   - The twin fixture `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:56-57` still carries the old text: "perch exits to Describe without Drift-Verify, which the orch moves back with `lym loom goto --to Drift-Verify`".
   - The fixer left it as it was. The recipe's header comment is the only place that text lives besides this fixture, so the two now disagree.
   - HEAD commit 9019c68b says Drift-Recheck was added to this fixture, so the fixture is kept in step with the recipe at least for its rows.
   - Whether a test compares header comments, I did not check. A reader who diffs the fixture against the recipe would notice the drift.
   - Fix: reword the fixture's comment to match, or confirm no test or reader needs the fixture's header to match the recipe's.

Checked and consistent:

- **`darn-recipe.yaml`:** its Drift-Recheck comments (lines 25-26, 224-229) already say the Stuck routes to Drift-Verify. It has no perch-exit-to-Describe sentence, so F6 has no twin there.
- **F4:** `reportengine/doc.go` lines 130 and 132 now match the `timingspans.go` maps (`NameDriftRecheck` is `PhaseWebster` at line 102 and `PhaseReview` at line 120).
- **F2:** the new `doc.go:140` sentence agrees with `recheck.go:36` and `recheck.go:164`, which both say lapse-all.
- **F5:** no other copy of the "Drift-Baseline, Drift-Await and Drift-Verify" template wording turned up under `contracts`, `internal` or `docs`.

**Calls made (2 of 6)**

1. `grep` for the old wordings ("Drift-Baseline, Drift-Await and Drift-Verify", "exits to Describe", "goto --to Drift-Verify") across `contracts`, `internal` and `docs`, plus `grep` on `darn-recipe.yaml` for Drift-Verify. My context lacked any search for twins of the reworded texts.
2. `grep` for "every acceptance" across `internal/driftengine` and `docs`. My context lacked the other places that state the lapse-all behaviour.
