```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it:**

1. Merge and gate with no lock, then land under the lock on a parent that has not moved: `TestFinalize_GatesWithNoLockAndLocksToLand`, row "an unmoved parent lands on the tree gated with no lock".
2. A parent that moved before the lock frees the lock, and the merge and gate run again with no lock: same test, row "a parent that moved before the lock…".
3. A spent lost-race bound merges and gates under the lock: same test, row "a spent lost-race bound…".
4. A bound of 0 keeps the old behaviour: same test, row "a bound of 0…". All earlier Finalize tests also run at bound 0, because their fixture leaves `FinalizeLostRaces` unset.
5. A parent that already holds the work runs no lock-free pass: same test, row "a parent already holding the work…".
6. The resolver gets the tip in the lock-free merge, and "" in the locked merge: same test, through `wantTips`.
7. The lock is released after landing: same test, through the final `lockHeld()` assertion.
8. `parentMergeTip` picks the tip for each relation and for the push skip, takes the lock only for the local read, and returns the lock-wait reasons: `TestMergeIn_MergeTip` and `TestMergeIn_UnreadableLockRecordHaltsNamingTheReleaseVerb`. Both drive it through ParentAlign, which shares the function.
9. Finalize's lock-free pass followed by the locked fast-forward of a parent behind its upstream, then the match on the gated tip: none.
10. A gate failure in the lock-free pass runs the fixer and fix review there, `freeLock` frees nothing, and the run then lands: none.
11. A conflict in the lock-free merge goes through the conflict session: none.
12. The merge record from the lock-free pass carries `MergedTip` = tip without `ReadParentTips(false)` (the `judgeParentMerge` tip branch): none.
13. `gateUnlocked` halts on a failed read of whether the parent holds the work: none.
14. `gateUnlocked` returns an error from `parentMergeTip`, or halts on its lock reason: none at the Finalize level.
15. Step 1b4b fails on an error from `HeadSHA`: none.
16. The locked pass after a lost race queues at the front: none.
17. The locked pass after a lock-free gate skips the verify of the already verified tree: none. The fake gate verifies every time, so the test shows a verify under the lock (`wantLockAtVerify` `[false, true]`).
18. `keepParentHead` moved into `mergeAndGate`, keeping the base of the landed diff: covered at bound 0 only, by the landed-size tests at `finalize_test.go:603-680`.
19. A negative `finalize_lost_races` is refused at load: `TestLoadConfig_InvalidKeys`, row "finalize lost races below 0".
20. The log line when the bound is spent: log only, so no test is needed.

**Findings:**

- **MEDIUM** · design · `internal/landingshed/finalize_unlocked_test.go:427-429`
  No test covers the lock-free pass with an upstream the parent is behind. A wrong order between the head compare in step 1b4b and `syncParent`'s fast-forward would pass every row.
  Every row stubs `ReadParentTips` with `Relation: ParentEqual`, so the fast-forward at `finalize.go:605` never runs before the compare at `finalize.go:442-451`.

- **MEDIUM** · design · `internal/landingshed/finalize_unlocked_test.go:415-419`
  No test covers a gate failure, the fixer or fix review, or a conflict in the lock-free pass. The new `freeLock` rule, that a process holding no lock frees nothing (`finalize.go:545-552`), is never exercised there, and neither is the land that follows.
  Every row lands on a gate that passes (`requireLanded`). `TestFinalize_GateAndFixer` (`finalize_gate_test.go:311`) runs at bound 0, because the fixture at `finalize_gate_test.go:46-83` leaves `FinalizeLostRaces` unset.

- **LOW** · design · `internal/landingshed/finalize.go:334-357` and `finalize.go:442-446`
  The new error paths have no test: the owner halt on a failed holds-read in `gateUnlocked`, the error and lock-reason ends of `parentMergeTip` in Finalize, and the `fail` on a `HeadSHA` error in step 1b4b.
  `finalize_unlocked_test.go:430` stubs `ParentHoldsTaskWork` to succeed, and no row makes `HeadSHA` fail.

- **LOW** · design · `internal/landingshed/finalize.go:1354-1362`
  No test pins that the lock-free merge writes the merge record's `MergedTip` as the tip it merged. The fixer reads that field at `finalize.go:1141`.
  `finalize_unlocked_test.go` asserts only the resolver's tips, never the record.

- **LOW** · design · `internal/landingshed/finalize.go:310` (`front = true` on a lost race)
  The summary claims "After a lost race the next locked pass queues at the front", and no row asserts it.
  `_lym/landing/summary.md:34`; the row "a parent that moved…" at `finalize_unlocked_test.go:416` checks only the lock and the tips.

- **LOW** · design · `internal/landingshed/finalize_unlocked_test.go:415`
  No test shows that the locked pass after a lock-free gate skips the verify. That skip is what holds the lock "for the checks and the landing alone" (`finalize.go:194-195`).
  The fake gate counts a verify under the lock (`wantLockAtVerify` `[false, true]`), so a locked merge that changed the tree and re-ran the gate under the lock would pass.

Tool calls:
1. Listed the Test functions in the Finalize and ParentAlign test files, and which tests use a lock store. The manifest read no test inventory beyond the fixture.
2. Grepped the Finalize tests for lock-freed and front-queue assertions, and read the opening rows of `TestMergeIn_MergeTip`. Needed to decide what existing tests cover the new `freeLock` rule and the shared `parentMergeTip`.
