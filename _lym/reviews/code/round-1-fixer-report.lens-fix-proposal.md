```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that were fixed as proposed (F1, F2, F4, F5, F6, F8, F9, F10, with F11 resting on F1).

**LOW**
- **Changed text:** `TestFinalize_GatesWithNoLockAndLocksToLand` in `internal/landingshed/finalize_unlocked_test.go`, F9's skip-keyed `onVerify` and its `wantLockAtVerify` of `[false]` and `[false, false]`.
- **Summary:** counting only the verifies that ran means these rows no longer pin that the locked pass still calls the gate over its merged tree after a lock-free pass.
- **Case:** change `landUnderLock` so that it runs `mergeParent` but skips `gateAndFix` when `gatedTip != ""`.
  - The unmoved, moved and behind-upstream rows still pass, because a skipped verify and a verify never called both record nothing. Before this fix, the `true` entry failed on that change.
  - The spent-bound row still passes, because its locked pass has `gatedTip == ""`.
  - `TestFinalize_QueuesAtTheFrontAfterALostRace` still passes: its `fronts` come from the lock waits that `onVerify` and `ParentHoldsTaskWork` contend, and the two locked acquires are contended by the lock-free verifies.
- **Why it matters:** the `ErrMergeInRequired` retry under that same pass would then land a code-side parent commit without a gate. This was found by reading the tests, not by running the mutation. I did not check whether another test in the package runs that retry with `FinalizeLostRaces > 0`; the ones visible in context run at bound 0.
- **Possible assertion:** besides the verifies that ran, check that the unmoved row's gate was asked once with the lock held (a skipped answer counts).

I built no breaking case for the other proposals (F1, F2, F4, F5, F6, F8, F10, F11).

Tool calls made: none.
