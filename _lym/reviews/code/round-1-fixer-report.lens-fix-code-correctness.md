```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code through the correctness lens, working from the context I hold.

**No findings.**

The changed paths I ran step by step:
- **`concludeMergeSides`**: with the weft arm gone, a record this binary writes (`WeftOutcome` `up_to_date`) concludes only the warp side. The warp arm still uses `logger`, `rec` and `saveMergeState`, so nothing is left dangling.
- **`sideRecordedMergeGone` and `sideConcludeAlreadyLanded`**: with the empty-source exemption gone, a warp record with an empty `WarpSource` now errors with an invalid SHA, which `TestMergeCrucible_ContinueRefusesAttemptThatNeverReachedBothSides` hit. The fix sets `WarpSource` there. The weft side is skipped by its `up_to_date` outcome before the source is read.
- **`recordedAnchorOnWeft`**:
  - A remote carrying the records branch reads `FETCH_HEAD`.
  - A namespaced remote without it returns not found.
  - Otherwise it reads `HEAD`.
  - `warpBranch` is still used, and the vet and tier-1 gate runs passed.
- **Stale-anchor refusal in `clone.go`**: `warpBranch` is in scope and the format arguments match the verbs. `TestCloneHub_StaleFabricAnchorHardErrors` now applies the remedy and asserts the retried clone adopts `backend`.
- **Drift partial scope**:
  - `checkScope` refuses `partial` through the generic message, which names `full`.
  - `Config.ScopeWasPartial` and the `logger` import are gone, and nothing else in the tree referenced them.
  - Stored `ScopeRecord` handling in `verdict.go` and `check.go` is untouched.
- **`retiredKeyError` in batcher**: `templateNamesProfile` decodes the template's `profiles` map. A template-named profile (`wide`) keeps the reconcile text and `tuned` or `mine` get the hand delete only. Both `assertRetiredKey` callers pass the matching flag.
- **loom plan keys**: they are retired with category `guard` and the tapestry way forward, and `TestPrepareRunSettings` asserts the text through `wantWayForwardOf`.

Whether each added test fails with its defect in place:
- **`TestAdd_AtAnotherAnchorCommitsTheDeletionOfTheOnlyTrackedRunRecords`**: the fixture leaves only run records tracked under `_lym`, so `_lym` is untracked at the move. Without `stagedOnly`, the commit pathspec omits `_lym`, the deletion stays staged, and both the diff and `diff --cached` assertions fail.
- **`TestUnwireJunctions_ExcludeLineFollowsTheSiblingWalk`**:
  - Negating the same-anchor compare makes the prime skipped in case 1, so the line is stripped and the test fails.
  - Negating the link check makes the plain directory count as wired in case 3, so the line is kept and the test fails.
- **`TestEnsure_FailedWalkAfterAReinterpretationLeavesNoMarkerAndTheNextWalkCommitsItOnce`**: the hook fires after the marker and base are written. A rollback that left the marker would fail the stat assertion, and a duplicate commit would fail the one-commit count.

Out of lens: the review's F11 sub-items at `configsync.go:482`, `recordsgit.go:92` and `circling.go:74` are not addressed in the change set.

No further tool calls were made; the context already held every changed file and the gate results.
