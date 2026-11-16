```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the range, I listed what reads or describes it outside the touched lines and checked whether the diff leaves it correct.

**Per change, the set found:**
- **`testcover.Dir(boardDir)` became `Dir(root)`, and its doc changed.** Callers are `redundancy.go:282`, `testmap.go:311,313`, `redundant_integration_test.go:38`, `redundancy_integration_test.go:118`, and `testmap_integration_test.go:668,671,760,1341,1864,1904`. All are still correct: only the parameter name and the doc changed. `testcover/doc.go` and `lymdirs/dirs.go` were updated in the diff. Covered.
- **`hubgeom.PrimeWorktreePath` was removed.** No remaining caller in Go or markdown outside `_lym/reviews`. Covered.
- **`testMapLocation` now returns three values, and `coverageMapsDir` is new.** Callers are `runTestMap` (`testmap.go:243`) and `runRedundant` (`redundant.go:93`). Both were updated. Covered.
- **The `testmap` refusal outside a hub now reads "from a worktree of a hub", and the non-prime refusal is gone.** The `OutsideEveryHub` row in `TestGateTestMap` was updated, and the `OutsideThePrime` and `FromAPairsRecordsSibling` rows were removed. No other site asserts the old text. Covered.
- **`redundant`'s way forward now reads "from this worktree".** `redundant_integration_test.go:105-109` matches only on substrings like "has no coverage map" and "way forward", so it is unaffected. Covered.
- **Behaviour change: `testmap` runs in any worktree and keeps a non-prime worktree's maps under its anchor's `.lym`.** Places that describe it:
  - `docs/overview.md:303,934-935`: updated.
  - `internal/gatecli/doc.go` (Coverage maps section and the `redundant` section): updated.
  - `internal/gatecli/testmapplan.go:1`: updated.
  - `README.md:77`: neutral, still correct.
  - `internal/dobbyengine/doc.go:503`: neutral, correct.
  - `internal/gateslot/doc.go:67,171-175`: neutral, still correct, because the shared maps are still read by the redundancy report step from every worktree.
  - `pattern/PATTERN-test-speed.md:221`: **stale**.
  - `pattern/PATTERN-test-speed.md:290`: **stale**.
  - `docs/research/speed/*`: measurement reports, exempt.
- **The build lock moves with `mapsDir`.** `testMapBuildLockPath` and `lockPackage` (`testmap.go:598-610`) and `waitBuildLock` (`testmapwait.go:22`) all take `r.mapsDir`. `lock.TrackedFileLockPath` puts the lock next to the map, because there is no `_lym` ancestor. This matches `PATTERN-lym-files`, since the map is not a tracked file. Covered.
- **Declared PATTERN changes:** the directive declares none, so there is nothing to check.
- **Commit messages:** the four commits carry no `Pattern-Departure:` trailer, and there is nothing a trailer would need to excuse. Covered.

**Findings:**

1. MEDIUM, consistency, `pattern/PATTERN-test-speed.md:221` and `:290`. The PATTERN background still describes the old behaviour that only the prime builds maps.
   - Line 221 says `lym gate testmap` "rebuilds the per-package coverage maps from the hub's prime worktree".
   - Line 290 says a call "plans every map at the prime's HEAD through the coverage selector".
   - After the diff, both are false for a non-prime call: it builds at its own worktree's HEAD into its own `.lym/testcover` (`internal/gatecli/testmap.go:301-313`, `internal/gatecli/doc.go:996-1001`).
   - The diff updated `docs/overview.md` and the gatecli doc but left this rule document wrong.
   - Fix: reword both lines to "at the HEAD of the worktree it runs in", naming the shared maps from the prime and a worktree's own maps elsewhere. If this repository requires a declared PATTERN change before a `pattern/` background is edited, defer the edit to the run's parent instead.

Tool calls:
1. Grep for `testcover.Dir(` callers and stale prime wording in gatecli and testcover. The context only had the grep for the `mapsDir` field, not the `Dir` callers in tests.
2. Read `redundant_integration_test.go:95-135` and grepped for "from the hub's prime worktree". The context did not show the refusal rows that might assert the changed way-forward text.
3. Grep for every mention of "lym gate testmap" outside the diff. This was needed to find docs that describe the changed behaviour.
4. Read `README.md:74-82`, `gateslot/doc.go:64-70,169-180`, `PATTERN-test-speed.md:284-296` and the redundancy-step lines of `gatecli/doc.go`. These were the hits from call 3, which the context had not read.
