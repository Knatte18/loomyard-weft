```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the fixer's edits to cards 1 and 2 still agree with the rest of the plan (overview and cards 3–4).

**LOW** — card 1, new subtest `AComputedScopeTakesNoBaseFromTheGreenRecord`; card 3's list of subtests whose exit changes.
- **Summary:** Card 3 makes every map run exit non-zero when a measurement fails. It changes the expected exit of the existing whole-module map call, but it does not name the new whole-module map call card 1 added.
- **Evidence:**
  - Card 3 line 108: "The first `--all` call's subtest wants exit 1, with `outcomes.TestOutcomesFails` and `alpha.TestAlphaHangs` in `failed_tests`… and broken's failed build (D7)."
  - The new card 1 subtest runs `test --wait --map` with no scope flag on the prime. That call "builds every package `--all` builds", so it measures the same failing `outcomes` and `alpha` tests and the `broken` build.
  - Card 1 does not say what exit code the new subtest expects, and card 3 does not update it. A reader of card 3 can't tell whether the new subtest asserts exit 0, which would break at card 3's gate, or whether it asserts no exit code at all.
  - Fix: have card 1's subtest assert only the built set. Alternatively, add the subtest to card 3's exit-1 row.

No other disagreements:
- Card 3 line 121 relies on "no green record yet". The new subtest removes the green record and restores tier0's command when it ends, so the two agree.
- Card 3 line 84 and overview line 28 say map mode never reaches the green record, which matches the rewritten line 114.
- Card 2's Intent and Edit list already include `internal/testcover/doc.go`, so the added `ProfiledFuncs` sentence has a file to land in.
- Card 1's Docs entry for `internal/gateslot/doc.go` now names the `Pool.TryAcquire` sentence. Card 3 also edits that file, but only its map-build bounds lines, so the two edits don't collide.

Tool calls:
- One grep of `00-overview.md`, `03-map-run-tier-bounds.md` and `04-map-run-clean-checks.md` for green, D7, gateslot, testcover, TryAcquire and ProfiledFuncs mentions. My context held only cards 1 and 2, not the overview or cards 3–4.
