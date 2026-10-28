```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
1. Code-side push rejected more than once while origin lacks nothing, then lands with no record and no mail: pinned by `TestPushFailureClasses/APushThatLostARaceIsRetriedUntilItLandsWithNoRecord/Code` (`pushclasses_integration_test.go`). It asserts the origin tip, no record, no mail and `sleeps == 1`.
2. Records push whose replay retry is rejected again lands on a later attempt: pinned by the same test, `/Records` row.
3. Code-side rejection with origin ahead is recorded at once, with no sleep: pinned by `RejectedBecauseTheOtherMachinePushedFirst`. It asserts `*PushFailedError` of class `not_fast_forward` and `sleeps == 0`.
4. Persisting rejection waits out the backoff, then writes one record and one mail (records side): pinned by `ARecordsRetryWhoseSecondPushIsRejectedAgainWritesANotFastForwardRecord`. It asserts `sleeps >= len(PushBackoffDelays())`, one record and one mail.
5. Persisting rejection on the code side or the bolt side writes one record and one mail: none.
   - Code side: the declined-always case in `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide` asserts only `IsPushRejected`, an unmoved bare and no local movement. It does not assert the record count, the mail count or the sleep count.
   - Bolt side: `boltpullfirst_integration_test.go` only gained a no-op sleep.
6. The bolt side lands on a later attempt: none; only the records and code rows exist. Bolt reaches the loop through `retryBoltPush`, but nothing pins that wiring.
7. Loop stops at once, with no sleep, on a replay conflict, anchor stop, auth refusal or transport failure: none.
   - `rejectionRetryKind` has no case table and no sleep-count assertion for these kinds.
   - Existing tests, such as the anchor-stop one, would still pass if these were retried, since they only get slower or end in the same error.
8. An unnamed error (a failed fetch) is retried, and the failure is still recorded as a rejection:
   - The anchored test asserts `IsPushRejected` and an unmoved bare, so the joined-rejection shape is pinned.
   - Nothing asserts the retry itself, because no sleep count is checked.
   - Nothing covers a transient fetch failure followed by a landing.
9. Loop bounded by the backoff length: only the lower bound (`>=`) is asserted. An unbounded or longer loop is not pinned from above.
10. `pushRebaseFreeRetryingOnce` removed, stale-tracking-ref retry folded in: its stale-ref case is covered only indirectly by the declined-once anchored case and the new Code row. I found no dedicated stale-ref test in the diff.
11. Way-forward wording: pinned by the `TestFailedPushRecord` rows.
    - Records with paths, prime records with paths, bolt with paths and records with no paths are all exact-string assertions.
    - No bolt-without-paths row, but it shares `recordsConflictWayForward`.
12. `doc.go` prose: exempt.

**Findings**
- F1, LOW, scope: no test pins the stop-at-once behaviour for non-retryable failures. This is the `ErrReplayConflict`/destructive-refusal, anchor stop, auth and transport cases in `rejectionRetryKind` / `settleRejection` (`pushclassified.go`). Add a `sleeps == 0` assertion to the existing anchor-stop and replay-conflict subtests, or a small table. Evidence: those subtests in `pushclasses_integration_test.go` use no counting backoff.
- F2, LOW, scope: no test for a persisting rejection on the code side (one record, one mail, whole backoff slept) or on the bolt side (`PushRecorded` retry). Evidence: `pushanchored_integration_test.go` ~L357-368 asserts no record count, and `boltpullfirst_integration_test.go:610` has a no-op sleep only.
- F3, LOW, scope: no test shows an unnamed error (a failed fetch) is retried, with a sleep count above 0 or a later landing. The anchored fetch-failure case checks only the final error and the unmoved bare (~L340-348). The `sleeps >= len(delays)` assertion pins no upper bound, so the loop's boundedness is untested.

Tool calls: none; the inherited context held the diff and test hunks.
