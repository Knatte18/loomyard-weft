```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F4–F14 code and tests as the fixer changed them.

**Findings**

1. LOW, `testmappackage.go`, `runFirstKeptEntry` and its call in `mapPackage`.
   - **Summary:** the new probe branch has no test, so deleting it breaks nothing.
   - **Evidence:**
     - The only new rows that reach the self-check have a passing queued test, so `firstProfile(runs)` is never nil there.
     - No stand-in test makes every queued test fail, skip or time out on a non-whole job.
   - The code itself is correct on a concrete input:
     - Take a map with one passing entry `TestA` and a queue of one failing `TestB`.
     - The probe runs `TestA` and appends its run.
     - `selfCheck` then excludes `TestA` through `ran`.
     - `fillMapTests` replaces `TestA` with the fresh run.
   - One side effect: a flaky probe failure downgrades `TestA` to an `OutcomeFailed` entry, and the self-check is then skipped anyway. This is acceptable.

2. LOW, `testmappackage.go`, `r.measures` moved after the skip checks.
   - **Summary:** no row asserts that a package skipped after its tests reports no measure.
   - **Evidence:**
     - The new `testMapVerbCounts` check runs only after a call with no skips.
     - The existing skip subtests (`ASkipAfterTheTestsCountsAgainstTheCap`) decode neither `packages` nor `plan`.
     - Restoring the early append would pass every test.

I found no wrong result in the changed code. Each point below holds on a concrete input:
- **`PlanUpkeep`:** the `declared` filter drops an entry for an undeclared test and sets `Changed`. The `Failure` clear runs only when `Failure.Commit != in.Commit`, and `planPackage` already returns early for a failure at HEAD, so nothing is lost.
- **Audit:** a zero `MeasuredAt` sorts first, because `time.Time.Compare` puts the zero time earliest. No caller read the dropped `BuiltAt` fallback.
- **Defect in place:** each added test fails with its defect restored.
  - `ARemovedTestDropsItsEntry`: the stale entry would stay.
  - `AFailureAtAnEarlierCommitIsCleared`: `Map.Failure` would stay set.
  - Ghost-entry row: a ghost entry seeded from the old map would stay with its old commit.
  - `Zzz` row: with the `lostFunc` branch removed, `SelfCheckMismatches` would be 0.
  - No-slot row: if the plan took a slot, the call would hit the one-minute bound.
  - Cancelled row: a `Failed` entry would be written.
  - `TestDropsPlain` spawn entry: if `Spawn` were ignored, `TestDropsLate` would become contained.
- **Not a regression guard:** the beta `Failure` row only pins current behaviour. `mappedPackages` never reads `Failure`, so the row passes without any defect. This is not a defect in the change.

**Tool calls:** none made. My context already held every changed file, and the parent's tier 0 and tier 1 gate runs passed.
