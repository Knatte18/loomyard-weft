```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff changes, find the test row that would fail if that behaviour broke.

**Behaviours and the tests that pin them:**

1. **The hour loop returns a held message at its hour with its last reported reason, and an unreported one with `bridge-unsettled`, when no take runs.**
   - Pinned by `TestStore_MachineOutbound/WithTheSchedulerStartedAndNoTakeAMessageIsReturnedAtItsHourWithItsLastReportedReasonOrBridgeUnsettled`.
   - Its assertions on the outcome, the two return reasons and the two notices are unchanged (`internal/creelengine/machinemail_test.go:615-629`).
   - Covered.

2. **The scheduler's first pass sees messages posted before `Start` and arms their hour wait.** The reorder puts this path under the subtest.
   - Pinned by the same subtest: `AwaitWaitsWhere(1, isHourWait)` at `machinemail_test.go:613`, then the return assertions.
   - If the first pass armed no hour wait, the subtest would time out. If it armed a wrong one, the messages would not return after `Advance(time.Hour)`.
   - Covered.

3. **A post after `Start` wakes the loop, which arms an hour wait.** The subtest pinned this before the reorder and no longer does.
   - Still pinned by `APostToAnotherMachineStartsNoHubWorkerAtTheSchedulersStartOrOnThePost` (`machinemail_test.go:631-642`). It starts the scheduler, posts, and waits for one hour wait. With no wake, the loop sits idle with nothing pending and arms none.
   - Covered.

4. **A message posted after `Start` comes back at its hour.** The subtest pinned this before the reorder and no longer does.
   - Still pinned by `TwoMessagesDueTogetherAreReturnedInOnePassAndTheLoopNeverWaitsUnderASecond` (`machinemail_test.go:643-657`). It posts after `e.start`, advances past the hour, and waits for `machine-returned` on both messages.
   - That subtest checks the return event, not the reason. The reason is the same `returnDueMachineLocked` code whether a message was posted before or after the start (`machinemail.go:98-111`), and behaviour 1 pins it. Nothing becomes uncovered.
   - Covered.

**Findings:** none.
- The reorder removes no coverage, as behaviours 3 and 4 show.
- No assertion was widened or removed. The 10-second deadlines in `awaitOutcome` and `AwaitWaitsWhere` are unchanged.
- The subtest still checks everything its name states: scheduler started, no take, returned at the hour, last reported reason or `bridge-unsettled`.

Tool calls: none. Every definition and sibling subtest I needed was already in my context.
